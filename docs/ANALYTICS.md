# Governance Analytics (Read-Only)

This document defines allowed analytical views.
All analytics are observational only and must not trigger actions.

---

## Allowed Views

### OFF Frequency
- OFF count per doctor
- OFF count per clinic day
- OFF trends over time

Purpose:
- Capacity planning
- Policy review

---

### Replacement Patterns
- Replacement frequency by doctor
- Same-day replacement attempts (blocked vs allowed)

Purpose:
- Detect strain points
- Validate rules effectiveness

---

### Repeated Absence Clusters
- Consecutive Absent occurrences
- Absent frequency within 35-day window

Purpose:
- Advisory input only
- No automatic penalties

---

## Explicit Constraints

Analytics must NOT:
- Modify attendance
- Trigger status changes
- Auto-generate OFF
- Score or rank doctors

All outputs are advisory.
