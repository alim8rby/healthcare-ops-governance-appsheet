# Decision Log

This document records **irreversible or high-impact design decisions** made in
the OPD Attendance & Governance System.

The purpose is not historical narration, but:
- Explainability under audit
- Protection against future “simplification” attempts
- Preservation of governance intent as the system evolves

If a decision is logged here, it is considered **architecturally intentional**.

---

## D-001 — Attendance as the Single Source of Truth
**Decision**  
All operational reality regarding doctor presence, absence, expectation, and
OFF states is represented exclusively in the Attendance table.

**Rejected Alternatives**
- Inferring attendance from logs or actions
- Flags on Doctors table
- Derived attendance states

**Rationale**
Attendance is an event, not a property.
Events must be explicit, dated, and auditable.

**Risk Prevented**
- Silent rewrites
- Retroactive reinterpretation of history
- Conflicting “truths” across tables

---

## D-002 — Deterministic AttendanceID (DoctorID-YYYYMMDD)
**Decision**  
Attendance records use a deterministic primary key.

**Rejected Alternatives**
- Auto-generated IDs
- Composite uniqueness rules without deterministic keys

**Rationale**
Deterministic keys enforce uniqueness structurally, not procedurally.

**Risk Prevented**
- Duplicate attendance rows
- Coordinator-dependent correctness
- Race-condition edge cases

---

## D-003 — OFF Implemented as a Real Attendance Row
**Decision**  
OFF is a first-class Attendance record with AttendanceStatus = OFF.

**Rejected Alternatives**
- Boolean OFF flag on Doctors
- UI-only hiding
- Derived “not expected” logic

**Rationale**
OFF must be date-specific, auditable, and metric-safe.

**Risk Prevented**
- Conflation of OFF with Absent
- Retroactive sanitization of attendance
- Informal capacity manipulation

---

## D-004 — Append-Only Status Change History
**Decision**  
Doctor administrative status changes are recorded in an append-only ledger.

**Rejected Alternatives**
- Direct editing of Doctors.AdminStatus
- Overwriting previous status values

**Rationale**
Administrative authority must be reconstructible over time.

**Risk Prevented**
- Untraceable administrative decisions
- Disputes without evidence
- Retroactive justification

---

## D-005 — Eligibility Rules Over Validation Errors
**Decision**  
Invalid actions are prevented by filtering eligibility, not by rejecting input.

**Rejected Alternatives**
- Allow selection then show error
- Coordinator warnings without enforcement

**Rationale**
Human operators under pressure make predictable mistakes.
Systems must fail *closed*, not *loud*.

**Risk Prevented**
- Repeated coordinator errors
- Escalations caused by UI friction
- Inconsistent enforcement

---

## D-006 — Replacement Capacity Protection
**Decision**  
Same-day replacement between doctors is prohibited unless the replacement doctor
has an OFF record for that date.

**Rejected Alternatives**
- Allowing same-day replacements unconditionally
- Manual approval workflows

**Rationale**
Replacement consumes clinic capacity.
Capacity must be accounted for explicitly.

**Risk Prevented**
- Double-booking clinic resources
- Hidden workload inflation
- Post-hoc disputes

---

## D-007 — Special Clinics Exempt from AssignedDay Constraints
**Decision**  
Doctors classified as Special Clinics are exempt from General Clinic
AssignedDay restrictions for replacement purposes.

**Rationale**
Special Clinics operate under a different scheduling ontology.

**Risk Prevented**
- Forcing artificial constraints
- Encoding logic into names or exceptions

---

## D-008 — No Background Automation
**Decision**  
All state changes are user-initiated and explicit.

**Rejected Alternatives**
- Scheduled jobs
- Bots auto-creating attendance
- Auto-cleanup scripts

**Rationale**
Implicit automation reduces explainability and increases mistrust in hostile
institutional environments.

**Risk Prevented**
- “Who changed this?” incidents
- Loss of human accountability
- Political resistance to the system

---

## D-009 — Metrics Are Derived, Never Written
**Decision**  
35-day metrics are computed exclusively from Attendance data.

**Rejected Alternatives**
- Storing counters
- Manual adjustments
- Hybrid models

**Rationale**
Metrics must be reproducible from raw events.

**Risk Prevented**
- Manual manipulation
- Inconsistent recalculation
- Loss of audit credibility

---

## D-010 — Governance Over Convenience
**Decision**  
All design choices favor integrity, auditability, and clarity over speed or UX
comfort.

**Rationale**
This system operates in environments where convenience is often weaponized
against accountability.

**Risk Prevented**
- Gradual erosion of safeguards
- “Temporary exceptions” becoming permanent
- System capture by informal practices
