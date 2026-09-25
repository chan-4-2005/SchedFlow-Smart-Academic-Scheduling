# SchedFlow — Project Specification

## 1. Project Title

**SchedFlow: Smart Academic Scheduling**

## 2. Project Overview

SchedFlow is a human-in-the-loop academic scheduling system that investigates the conversion of natural-language university scheduling requirements into formal scheduling constraints.

SchedFlow is not primarily an AI timetable generator.

University timetable requirements are often expressed by humans in natural language. These requirements may describe faculty availability, room restrictions, class conflicts, consecutive laboratory periods, preferences, or last-minute changes. Natural-language requirements can be ambiguous, incomplete, conflicting, or incorrectly interpreted when converted into formal constraints.

SchedFlow addresses this problem through a controlled pipeline:

```text
Natural-language requirement
        ↓
AI constraint extraction
        ↓
Structured constraint representation
        ↓
Schema and semantic validation
        ↓
Ambiguity / conflict detection
        ↓
Human clarification when required
        ↓
Deterministic OR-Tools scheduling
        ↓
Independent schedule verification
        ↓
Timetable + explanations + conflict information
```

The central architectural principle is:

> The LLM interprets scheduling requirements. It does not generate the timetable.

OR-Tools is responsible for deterministic timetable generation, and an independent verifier checks the resulting timetable.

---

## 3. Problem Statement

University scheduling requirements are frequently communicated using natural language.

Examples include:

* "Dr. Rao is unavailable on Monday afternoon."
* "The physics laboratory requires two consecutive periods."
* "Professor Kumar prefers morning classes."
* "Room 201 cannot be used for laboratory sessions."
* "Section A and Section B cannot have classes at the same time."
* "This faculty member is unavailable during the first two periods on Friday."

The same requirement may be interpreted differently depending on context.

Potential problems include:

* ambiguous terminology;
* incomplete information;
* incorrect constraint classification;
* conflicting requirements;
* incorrect conversion of natural language into formal constraints;
* constraints being silently interpreted instead of clarified.

SchedFlow investigates whether explicitly detecting ambiguity and validating extracted constraints before deterministic scheduling can reduce such errors.

---

## 4. Research Direction

The primary research direction is:

> Investigate whether ambiguity-aware constraint extraction and independent verification can reduce errors when converting natural-language university scheduling requirements into formal scheduling constraints.

The MVP is an engineering prototype that establishes the pipeline required for later evaluation.

The architecture must therefore preserve the distinction between:

1. natural-language interpretation;
2. formal constraint representation;
3. validation;
4. human clarification;
5. deterministic scheduling;
6. independent verification.

---

## 5. MVP Objectives

The MVP must support:

1. Natural-language scheduling requirement input.
2. AI-based extraction of structured constraints.
3. Classification of constraints as:

   * hard;
   * soft;
   * conditional;
   * ambiguous.
4. Pydantic-based structured validation.
5. Semantic validation.
6. Ambiguity detection.
7. Human clarification of selected ambiguous requirements.
8. Deterministic timetable generation using OR-Tools.
9. Independent timetable verification.
10. Conflict and infeasibility reporting.
11. A simple Streamlit interface.
12. Automated tests using pytest.

---

## 6. MVP Workflow

A user provides one or more natural-language requirements.

Example:

> "Dr. Rao cannot teach Monday afternoon. The physics lab needs two consecutive periods and should preferably be scheduled in Lab 2."

SchedFlow processes the requirement through the following stages:

### Stage 1 — Extraction

Gemini converts the natural-language requirement into structured constraints.

### Stage 2 — Schema Validation

Pydantic checks whether the extracted data conforms to the expected data model.

### Stage 3 — Semantic Validation

Deterministic Python validation checks whether the extracted constraints make sense.

### Stage 4 — Ambiguity Detection

SchedFlow identifies requirements whose meaning cannot safely be determined.

### Stage 5 — Human Clarification

The user can clarify relevant ambiguous constraints.

### Stage 6 — Scheduling

Validated constraints are converted into an OR-Tools scheduling model.

### Stage 7 — Verification

The generated timetable is independently checked against the formal constraints.

### Stage 8 — Presentation

Streamlit displays:

* extracted constraints;
* ambiguity warnings;
* timetable;
* hard-constraint violations;
* soft-constraint violations;
* infeasibility information;
* verification status.

---

## 7. Constraint Categories

SchedFlow must distinguish at least four categories.

### Hard Constraint

A requirement that must be satisfied.

Example:

> "Professor Rao cannot teach Monday Period 3."

### Soft Constraint

A preference that should be satisfied where possible.

Example:

> "Professor Rao prefers morning classes."

### Conditional Constraint

A requirement that applies only when a specified condition is true.

Example:

> "If the physics lab is scheduled, it must use Lab 2."

### Ambiguous Constraint

A requirement whose meaning cannot be safely formalized without clarification.

Example:

> "Professor Rao is unavailable Monday morning."

If the system has not defined which periods constitute "morning", the requirement should be flagged for clarification rather than silently interpreted.

---

## 8. MVP Scheduling Model

The MVP should use a simplified academic scheduling model containing:

### Entities

* Faculty
* Course
* Student section
* Room
* Time slot

### Scheduling assignment

A class assignment should contain at minimum:

* course;
* section;
* faculty;
* room;
* day;
* period.

### Example scheduling constraints

The MVP should support representative constraints such as:

* faculty cannot teach two classes simultaneously;
* room cannot host two classes simultaneously;
* section cannot attend two classes simultaneously;
* faculty availability;
* room availability;
* room restrictions;
* required consecutive periods;
* hard preferences;
* soft preferences;
* basic conditional constraints where practical.

The implementation should remain intentionally small. Additional constraint types can be added later.

---

## 9. LLM Responsibility

Gemini is responsible only for natural-language interpretation.

Gemini may:

* identify entities;
* identify constraint types;
* classify constraints;
* extract parameters;
* identify ambiguity;
* provide confidence/interpretation metadata where useful.

Gemini must not:

* generate the final timetable;
* decide solver assignments;
* directly manipulate OR-Tools;
* bypass Pydantic validation;
* silently resolve material ambiguity.

---

## 10. Deterministic Scheduling Responsibility

OR-Tools is responsible for generating the timetable.

The solver receives validated formal constraints.

The solver must not receive raw natural-language requirements.

Given the same scheduling input and configuration, the system should aim for reproducible behavior.

---

## 11. Independent Verification

The timetable verifier must be logically separate from the scheduling model.

The verifier should independently inspect the generated timetable and report:

* hard-constraint violations;
* soft-constraint violations;
* room conflicts;
* faculty conflicts;
* section conflicts;
* availability violations;
* consecutive-period violations;
* missing assignments;
* other supported constraint violations.

The verifier must not simply return "valid" because OR-Tools reported a successful solve.

---

## 12. Ambiguity vs Infeasibility

SchedFlow must distinguish:

### Ambiguity

The requirement is unclear.

Example:

> "Professor Rao prefers morning."

The system may need clarification about what "morning" means.

### Infeasibility

The requirements are clear but cannot all be satisfied simultaneously.

Example:

Two classes require the same room and same time slot while neither can be moved.

These cases must produce different user-facing explanations.

---

## 13. MVP Non-Goals

The MVP will not implement:

* authentication;
* production database;
* mobile application;
* RAG;
* voice input;
* autonomous agents;
* notifications;
* complex deployment infrastructure;
* multi-agent workflows;
* advanced optimization research;
* large-scale university integration;
* production-grade user management.

These may be considered in future versions but must not expand the MVP unnecessarily.

---

## 14. Technology Stack

### Programming Language

Python

### LLM

Gemini API

### Structured Validation

Pydantic

### Scheduling

Google OR-Tools

### User Interface

Streamlit

### Testing

pytest

### Version Control

GitHub

### Development

VS Code

### Primary Coding Agent

Claude

### Architecture / Research / Review / QA

ChatGPT

---

## 15. Success Criteria for the MVP

The MVP is successful if a user can:

1. Enter natural-language academic scheduling requirements.
2. See the extracted structured constraints.
3. See whether each constraint is hard, soft, conditional, or ambiguous.
4. Receive an ambiguity warning when clarification is required.
5. Provide clarification.
6. Generate a timetable using OR-Tools.
7. View the timetable.
8. Run independent verification.
9. See any detected violations.
10. See a meaningful infeasibility/conflict explanation when scheduling cannot satisfy the requirements.
11. Run the automated test suite successfully.

---

## 16. Future Research Extension

The MVP should leave room for future evaluation of:

* extraction accuracy;
* constraint classification accuracy;
* ambiguity detection accuracy;
* semantic validation accuracy;
* clarification effectiveness;
* solver feasibility;
* verification accuracy;
* comparison of direct extraction versus ambiguity-aware extraction;
* error rates in natural-language-to-constraint conversion.

The MVP should therefore preserve intermediate representations instead of collapsing the entire workflow into a single LLM call.

