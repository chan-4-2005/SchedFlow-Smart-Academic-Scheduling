# SchedFlow — System Architecture

## 1. Architectural Principle

SchedFlow follows a strict separation between natural-language interpretation and deterministic scheduling.

```text
Natural Language
      ↓
LLM Constraint Extraction
      ↓
Structured Constraint Model
      ↓
Schema Validation
      ↓
Semantic Validation
      ↓
Ambiguity / Conflict Detection
      ↓
Human Clarification
      ↓
Formal Constraints
      ↓
OR-Tools Scheduler
      ↓
Generated Timetable
      ↓
Independent Verifier
      ↓
Timetable + Explanations + Violations
```

The LLM must never directly generate the timetable.

---

## 2. High-Level Architecture

```text
┌───────────────────────────────┐
│            USER               │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│        STREAMLIT UI            │
│                               │
│ Requirement Input             │
│ Constraint Review             │
│ Clarification                 │
│ Timetable Display             │
│ Verification Results          │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│      CONSTRAINT EXTRACTION    │
│                               │
│ Gemini API                    │
│ Prompt + Structured Output    │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│    STRUCTURED CONSTRAINTS     │
│                               │
│ Pydantic Models               │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│ VALIDATION + AMBIGUITY LAYER  │
│                               │
│ Schema Validation             │
│ Semantic Validation           │
│ Ambiguity Detection           │
│ Conflict Detection            │
└───────────────┬───────────────┘
                │
        ┌───────┴────────┐
        │                │
   Ambiguous          Valid
        │                │
        ▼                │
 Human Clarification    │
        │                │
        └───────┬────────┘
                ▼
┌───────────────────────────────┐
│       OR-TOOLS SCHEDULER     │
│                               │
│ Scheduling Variables          │
│ Hard Constraints              │
│ Soft Constraints              │
│ Objective                     │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│       GENERATED TIMETABLE     │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│     INDEPENDENT VERIFIER      │
│                               │
│ Constraint Checks             │
│ Conflict Detection            │
│ Completeness Checks           │
└───────────────┬───────────────┘
                │
                ▼
┌───────────────────────────────┐
│      RESULTS / EXPLANATION    │
└───────────────────────────────┘
```

---

## 3. Architectural Components

## 3.1 Streamlit Interface

Location:

```text
app/streamlit_app.py
```

Responsibilities:

* collect requirements;
* display extracted constraints;
* display ambiguity;
* collect clarification;
* trigger scheduling;
* display timetable;
* display verification results.

The UI must not contain core scheduling or validation logic.

---

## 3.2 Constraint Extraction

Location:

```text
app/extraction/
```

Responsibilities:

* construct extraction prompts;
* send natural-language requirements to Gemini;
* parse structured output;
* convert output into internal Pydantic models.

The extractor must not create timetable assignments.

Conceptual interface:

```text
NaturalLanguageRequirement
        ↓
ConstraintExtractor
        ↓
List[Constraint]
```

---

## 3.3 Constraint Schemas

Location:

```text
app/extraction/schemas.py
app/core/models.py
```

Pydantic models define the formal representation of extracted requirements.

The schema should represent concepts such as:

* constraint ID;
* constraint type;
* category;
* entities;
* parameters;
* condition;
* source text;
* ambiguity information;
* confidence where useful.

The schema must be explicit enough to be consumed by deterministic validation and scheduling code.

---

## 3.4 Validation Layer

Location:

```text
app/validation/
```

The validation layer contains three conceptual stages.

### Schema Validation

Checks data structure and field types using Pydantic.

### Semantic Validation

Checks logical validity.

Examples:

* unknown faculty;
* unknown room;
* invalid period;
* invalid duration;
* invalid day;
* impossible ranges;
* incompatible parameters.

### Ambiguity Detection

Identifies requirements that cannot be safely formalized.

Ambiguity detection should be deterministic where possible after extraction.

The system must not hide ambiguity simply because the LLM produced a valid JSON object.

---

## 3.5 Human Clarification

Human clarification is part of the core architecture.

When a requirement is ambiguous, the UI should present the issue and allow the user to provide clarification.

Example:

```text
Requirement:
"Professor Rao is unavailable Monday morning."

Issue:
"Morning" has not been mapped to a defined period range.

Clarification:
[ Select periods ]
```

The clarified result becomes part of the formal scheduling input.

---

## 3.6 Scheduling Layer

Location:

```text
app/scheduling/
```

Responsibilities:

* define scheduling variables;
* define domains;
* translate formal constraints into OR-Tools constraints;
* define objectives for soft constraints;
* invoke OR-Tools;
* return a structured timetable or infeasibility result.

The scheduler accepts formal data only.

It must never parse natural language.

---

## 3.7 Independent Verification

Location:

```text
app/verification/
```

The verifier receives:

```text
Generated Timetable
+
Formal Constraints
```

It independently checks whether the timetable satisfies the constraints.

The verifier should not rely solely on solver status.

Conceptual interface:

```text
Timetable
    +
Constraints
    ↓
Verifier
    ↓
VerificationResult
```

---

## 4. Core Data Flow

```text
User Input
   │
   ▼
Raw Requirement
   │
   ▼
Gemini
   │
   ▼
Extracted Constraint
   │
   ▼
Pydantic
   │
   ├── Invalid → Validation Error
   │
   ▼
Semantic Validation
   │
   ├── Invalid → Semantic Error
   │
   ▼
Ambiguity Detection
   │
   ├── Ambiguous → Human Clarification
   │
   ▼
Formal Constraint Set
   │
   ▼
OR-Tools
   │
   ├── Infeasible → Conflict/Infeasibility Report
   │
   ▼
Timetable
   │
   ▼
Independent Verifier
   │
   ├── Violations → Verification Report
   │
   ▼
Final Result
```

---

## 5. Constraint Representation

A constraint should conceptually contain:

```text
Constraint
├── id
├── type
├── category
├── source_text
├── entities
├── parameters
├── condition
├── ambiguity
└── confidence
```

The exact implementation schema may evolve as implementation reveals necessary fields, but changes must remain consistent with this architecture.

---

## 6. Constraint Categories

The architecture recognizes:

```text
HARD
SOFT
CONDITIONAL
AMBIGUOUS
```

These categories have different downstream behavior.

### HARD

Passed to the solver as mandatory.

### SOFT

Represented as an optimization preference where practical.

### CONDITIONAL

Translated into formal conditional logic.

### AMBIGUOUS

Must be clarified before being treated as a scheduling constraint when the ambiguity materially affects scheduling.

---

## 7. Solver Boundary

The following boundary is mandatory:

```text
LLM
 │
 │ structured constraints
 ▼
Validation
 │
 │ validated formal constraints
 ▼
OR-Tools
```

Never:

```text
LLM → OR-Tools using natural language
```

Never:

```text
LLM → final timetable
```

---

## 8. Verification Boundary

The solver and verifier are separate components.

```text
                 ┌─────────────┐
Formal Input ───►│ OR-Tools    │
                 └──────┬──────┘
                        │
                        ▼
                   Timetable
                        │
                        ▼
                 ┌─────────────┐
Formal Input ───►│ Verifier    │
                 └─────────────┘
```

The verifier independently evaluates the timetable against the formal constraints.

---

## 9. Infeasibility Handling

The scheduler must distinguish between:

```text
No valid schedule exists
```

and:

```text
The input is ambiguous or invalid
```

Examples:

```text
Ambiguous:
"Friday morning"

Invalid:
"Period 99"

Infeasible:
Two mandatory classes require the same faculty,
room, and time slot.
```

The UI must communicate these differently.

---

## 10. Proposed Repository Structure

```text
schedflow/
│
├── README.md
├── PROJECT_SPEC.md
├── ARCHITECTURE.md
├── DEVELOPMENT.md
├── requirements.txt
├── .env.example
├── .gitignore
│
├── app/
│   ├── __init__.py
│   ├── streamlit_app.py
│   │
│   ├── core/
│   │   ├── __init__.py
│   │   ├── models.py
│   │   └── config.py
│   │
│   ├── extraction/
│   │   ├── __init__.py
│   │   ├── extractor.py
│   │   ├── prompts.py
│   │   └── schemas.py
│   │
│   ├── validation/
│   │   ├── __init__.py
│   │   ├── schema_validator.py
│   │   ├── semantic_validator.py
│   │   └── ambiguity_detector.py
│   │
│   ├── scheduling/
│   │   ├── __init__.py
│   │   ├── model.py
│   │   └── solver.py
│   │
│   └── verification/
│       ├── __init__.py
│       └── verifier.py
│
└── tests/
    ├── __init__.py
    ├── test_extraction.py
    ├── test_validation.py
    ├── test_scheduling.py
    └── test_verification.py
```

The implementation agent may adjust the exact file organization when necessary, but the architectural module boundaries must remain.

---

## 11. Future Extension Points

The architecture intentionally leaves room for:

* experiment datasets;
* extraction evaluation;
* ambiguity detection metrics;
* alternative LLMs;
* constraint extraction benchmarking;
* logging of intermediate representations;
* schedule quality metrics;
* research comparison experiments.

These are not required for the MVP.

