# Roadmap (Integrity-First)

This roadmap defines **allowed evolution paths** for the OPD Attendance &
Governance System.

Any future change must:
- Preserve auditability
- Preserve semantic meaning
- Preserve enforcement at the data level

Convenience-driven changes are explicitly out of scope.

---

## Guiding Principle

> The system may evolve, but its guarantees must not.

If a proposed feature weakens:
- determinism
- traceability
- eligibility enforcement
- separation of concerns

it must be rejected.

---

## Phase 1 — Consolidation (Current)

**Goal:** Stabilize governance, eliminate informal practices.

Focus areas:
- Documentation completeness
- Coordinator rule adherence
- OFF usage normalization
- Replacement abuse prevention

Explicitly **not** included:
- UI redesigns
- Automation
- Analytics expansion

---

## Phase 2 — Observability (Safe Enhancements)

**Goal:** Improve visibility without changing behavior.

Allowed:
- Read-only dashboards
- Audit views (OFF history, replacements, corrections)
- Pattern summaries (weekly/monthly)

Constraints:
- No derived data becomes authoritative
- No write paths added
- No metric redefinitions

---

## Phase 3 — Policy Support (Decision Assistance)

**Goal:** Support management decisions without automating them.

Allowed:
- Flags for repeated Absent patterns
- OFF frequency summaries
- Replacement frequency summaries

Constraints:
- No automatic penalties
- No automated status changes
- No auto-generated attendance

All outputs must be advisory only.

---

## Phase 4 — Controlled Expansion

**Goal:** Extend the system to additional clinics or departments.

Allowed:
- New Clinic Types
- Additional Clinic Days
- Additional coordinator roles

Constraints:
- Core semantics unchanged
- Attendance remains the single source of truth
- OFF remains a first-class state

---

## Explicitly Rejected (Out of Scope)

The following will **not** be implemented:

- Background jobs or bots
- Auto-attendance marking
- Auto-OFF assignment
- Manual metric overrides
- Soft deletes of historical data
- UI-only rule enforcement
- Encoding logic into names or labels

---

## Change Acceptance Criteria

Any proposed change must document:
1. Which rule(s) it affects
2. Which safeguards remain intact
3. How auditability is preserved
4. How disputes remain reconstructible
5. A rollback path

If any of the above is missing, the change must not proceed.

---

## Long-Term Vision (Without Compromise)

The system aims to:
- Reduce operational friction
- Reduce disputes
- Protect management decisions
- Normalize rule-based governance

It does **not** aim to:
- Replace human judgment
- Police behavior automatically
- Reform institutional culture

Governance succeeds when rules enforce themselves quietly.
