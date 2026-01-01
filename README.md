# OPD Attendance & Governance System (AppSheet)

This repository documents a production-grade outpatient clinic administration
system used to manage doctor attendance, OFF scheduling, replacements, and
performance metrics in a real hospital environment.

The system is designed under high-friction institutional constraints and
prioritizes:

- Data integrity over convenience
- Auditability over speed
- Governance over UI flexibility
- Explicit rules over personal judgment

This is not a demo project. The system is live and used operationally.

## Core Concepts
- Attendance is the single source of truth
- OFF is a first-class, auditable attendance state
- Ledger tables are append-only
- Eligibility rules prevent invalid actions before they happen
- No silent overwrites, no background automation

## Repository Scope
This repository contains:
- System architecture and data semantics
- Business and operational rules (with AppSheet expressions)
- Design decisions and their rationale
- Operational rollout guidance

It intentionally does NOT contain:
- Patient data
- Real staff names or identifiers
- Hospital or ministry-specific information
- Screenshots or exports from production

## Navigation
- `docs/ARCHITECTURE.md` — system design and philosophy
- `docs/RULES.md` — rules as language + expressions
- `docs/DECISION_LOG.md` — why key decisions were made
- `docs/OPERATIONS.md` — how the system is run safely
- `docs/ROADMAP.md` — future work without breaking safeguards

## Status
Deployed in a live OPD setting and under continuous refinement.

## License
MIT License
