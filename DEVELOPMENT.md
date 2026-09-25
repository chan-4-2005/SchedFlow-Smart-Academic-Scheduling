# SchedFlow — Development Plan

## 1. Development Principle

SchedFlow must be developed as a controlled pipeline rather than as a single AI application.

The implementation must preserve these boundaries:

```text
Natural Language
      ↓
Gemini
      ↓
Structured Constraints
      ↓
Pydantic
      ↓
Validation / Ambiguity
      ↓
Human Clarification
      ↓
OR-Tools
      ↓
Timetable
      ↓
Independent Verification
```

---

## 2. Development Priorities

The MVP is intentionally limited to a 4–6 hour implementation window.

Priority order:

1. Project foundation.
2. Formal constraint models.
3. Constraint extraction.
4. Validation.
5. Ambiguity handling.
6. Scheduling.
7. Independent verification.
8. Streamlit integration.
9. Testing.
10. Documentation and cleanup.

Do not implement low-priority infrastructure before the core pipeline works.

---

## 3. Phase 1 — Project Foundation

Create:

```text
app/
tests/
requirements.txt
.env.example
.gitignore
```

Configure:

* Python environment;
* dependencies;
* package structure;
* configuration management;
* Streamlit entry point;
* pytest.

The application should run before advanced functionality is added.

---

## 4. Phase 2 — Formal Constraint Model

Define Pydantic models representing the scheduling domain.

At minimum, model:

```text
Faculty
Course
Section
Room
TimeSlot
Constraint
TimetableAssignment
VerificationResult
```

The exact model design should support the MVP requirements without unnecessary abstraction.

The constraint model must support:

* hard;
* soft;
* conditional;
* ambiguous constraints.

Tests must verify valid and invalid model instances.

---

## 5. Phase 3 — Gemini Constraint Extraction

Implement:

```text
app/extraction/extractor.py
app/extraction/prompts.py
app/extraction/schemas.py
```

The extraction pipeline should:

1. accept natural-language requirements;
2. send a controlled prompt to Gemini;
3. request structured constraint output;
4. parse the result;
5. validate it using Pydantic;
6. return structured constraints.

The prompt must explicitly instruct Gemini that it is an interpreter of requirements, not a timetable generator.

The LLM must not receive permission to decide the final schedule.

---

## 6. Phase 4 — Validation and Ambiguity

Implement:

```text
schema_validator.py
semantic_validator.py
ambiguity_detector.py
```

### Schema validation

Use Pydantic.

### Semantic validation

Use deterministic Python logic.

### Ambiguity detection

Identify cases such as:

* undefined time expressions;
* unspecified entities;
* missing required information;
* unclear preference/requirement status;
* conflicting interpretations.

The system should expose ambiguity rather than silently guessing.

---

## 7. Phase 5 — Human Clarification

The Streamlit UI should allow a user to resolve supported ambiguities.

Example:

```text
Detected ambiguity:

"Monday morning"

Choose interpretation:

○ Periods 1–2
○ Periods 1–3
○ Custom
```

The clarification should update the formal constraint representation.

Do not ask the LLM to repeatedly guess the user's intent when a deterministic UI choice can resolve it.

---

## 8. Phase 6 — OR-Tools Scheduler

Implement:

```text
app/scheduling/model.py
app/scheduling/solver.py
```

The scheduler should:

1. receive validated formal constraints;
2. create scheduling variables;
3. define domains;
4. add hard constraints;
5. encode supported soft constraints;
6. solve using OR-Tools;
7. return a structured timetable or infeasibility result.

The scheduler must not contain natural-language interpretation logic.

---

## 9. Phase 7 — Independent Verification

Implement:

```text
app/verification/verifier.py
```

The verifier receives:

```text
Formal constraints
+
Generated timetable
```

It independently checks supported constraints.

The verifier should produce a structured result containing:

```text
valid
hard violations
soft violations
conflicts
warnings
```

The verifier should not simply trust the solver's success status.

---

## 10. Phase 8 — Streamlit Integration

The UI should expose the complete MVP workflow.

Suggested interface:

```text
SchedFlow
────────────────────────────────────

1. Scheduling Requirements

[ Natural-language input area ]

[Extract Constraints]

────────────────────────────────────

2. Extracted Constraints

Constraint table

Type | Category | Details | Status

────────────────────────────────────

3. Ambiguities

Detected issues

[Clarify]

────────────────────────────────────

4. Schedule

[Generate Timetable]

Timetable display

────────────────────────────────────

5. Verification

✓ No hard violations

or

⚠ Violations detected

Details...
```

Keep UI logic thin.

---

## 11. Phase 9 — Testing

Testing is required at every major layer.

### Extraction tests

Test:

* representative natural-language requirements;
* malformed LLM output;
* missing fields;
* invalid classifications.

LLM-dependent tests should be isolated from deterministic unit tests.

### Validation tests

Test:

* valid constraints;
* invalid periods;
* invalid entities;
* invalid durations;
* contradictory parameters;
* ambiguity detection.

### Scheduling tests

Test:

* simple feasible schedule;
* faculty conflict;
* room conflict;
* section conflict;
* availability restriction;
* consecutive periods;
* infeasible input.

### Verification tests

Test:

* valid timetable;
* faculty violation;
* room violation;
* section violation;
* availability violation;
* incomplete timetable.

---

## 12. Test Pyramid

Prefer:

```text
       ┌──────────────┐
       │ End-to-End   │
       └──────────────┘
      ┌────────────────┐
      │ Integration    │
      └────────────────┘
   ┌──────────────────────┐
   │ Deterministic Unit   │
   │ Tests                │
   └──────────────────────┘
```

Most tests should be deterministic unit tests.

Do not make the entire test suite dependent on live Gemini API calls.

---

## 13. Dependency Rules

Use only dependencies that directly support the MVP.

Primary dependencies:

* google-genai or the selected supported Gemini Python SDK;
* pydantic;
* ortools;
* streamlit;
* pytest;
* python-dotenv if needed.

Do not add:

* LangChain;
* agent frameworks;
* vector databases;
* RAG frameworks;
* unnecessary orchestration frameworks;
* production databases;

unless the architecture is explicitly revised.

---

## 14. Environment Variables

Secrets must be stored outside source control.

Example:

```text
GEMINI_API_KEY=your_key_here
```

`.env` must never be committed.

`.env.example` may contain variable names but no secrets.

---

## 15. Git Workflow

GitHub is the source of truth.

Recommended development flow:

```text
Make change
    ↓
Run tests
    ↓
Run application/check
    ↓
Review diff
    ↓
Commit
    ↓
Push
```

Commits should describe meaningful development stages.

Example:

```text
feat: add constraint models
feat: add Gemini constraint extraction
feat: add semantic validation
feat: add OR-Tools scheduler
feat: add independent verifier
feat: integrate Streamlit workflow
test: add scheduling and verification tests
```

---

## 16. Architectural Rules

The following rules are mandatory.

### Rule 1

The LLM does not generate timetables.

### Rule 2

Natural language never goes directly to OR-Tools.

### Rule 3

LLM output must pass structured validation.

### Rule 4

Material ambiguity must not be silently guessed.

### Rule 5

OR-Tools generates the timetable.

### Rule 6

The verifier independently checks the timetable.

### Rule 7

Streamlit must not contain core business logic.

### Rule 8

Secrets must never be committed.

### Rule 9

Core deterministic logic must be testable without the LLM.

### Rule 10

Do not expand MVP scope without explicit approval.

### Rule 11

Do not change the architecture silently.

### Rule 12

If implementation reveals that the documented architecture is insufficient, stop and report the proposed architectural change before making it.

---

## 17. Implementation Agent Rules

The coding agent must:

1. Read `PROJECT_SPEC.md`.
2. Read `ARCHITECTURE.md`.
3. Read `DEVELOPMENT.md`.
4. Follow the documented architecture.
5. Implement one major phase at a time.
6. Run tests after each phase.
7. Avoid unnecessary dependencies.
8. Avoid speculative features.
9. Keep interfaces between modules explicit.
10. Report assumptions and deviations.

The coding agent must not:

* replace OR-Tools with an LLM;
* create an agentic scheduling system;
* introduce RAG;
* create a production database;
* add authentication;
* silently change the project scope;
* bypass validation;
* bypass verification.

---

## 18. Definition of Done

A feature is considered complete only when:

* implementation exists;
* relevant tests exist;
* tests pass;
* integration does not break existing functionality;
* architecture boundaries remain intact;
* documentation remains accurate.

---

## 19. MVP Definition of Done

The MVP is complete when this complete flow works:

```text
User enters requirement
        ↓
Gemini extracts constraints
        ↓
Pydantic validates them
        ↓
Semantic validation runs
        ↓
Ambiguities are surfaced
        ↓
User clarifies when necessary
        ↓
Formal constraints reach OR-Tools
        ↓
OR-Tools generates timetable
        ↓
Independent verifier checks timetable
        ↓
Streamlit displays:
    - extracted constraints
    - ambiguities
    - timetable
    - conflicts
    - verification result
```

The MVP must demonstrate the architectural principle:

> AI interprets. Deterministic optimization schedules. Independent verification checks.

