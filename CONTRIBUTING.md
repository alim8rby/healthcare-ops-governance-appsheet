# Contributing Guidelines

This project documents a live governance system.
Contributions are welcome only if they preserve its guarantees.

---

## Non-Negotiable Principles

Any contribution must preserve:

- Data integrity
- Auditability
- Deterministic behavior
- Explicit rule enforcement
- Separation of concerns

Convenience, speed, or UI simplicity are not valid justifications
for weakening safeguards.

---

## Acceptable Contributions

- Clarifying documentation
- Improved rule explanations
- Additional decision log entries
- New rules that preserve existing semantics
- Safer enforcement mechanisms

---

## Prohibited Contributions

The following will be rejected:

- Encoding logic into names or labels
- Bypassing Attendance rows
- Collapsing OFF into Absent
- Weakening eligibility rules
- Introducing background automation
- Manual metric overrides
- Silent edits or soft deletes

---

## Change Requirements

Every proposed change must include:

1. The rule(s) affected
2. The risk it addresses
3. The safeguards it preserves
4. Any new enforcement surfaces
5. A rollback strategy

Changes without governance justification will not be merged.

---

## Review Philosophy

This is not a feature-driven project.
It is a governance system.

Stability and explainability override novelty.
