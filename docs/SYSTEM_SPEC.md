# System Specification (Authoritative)

This document is the human-readable, authoritative specification of the OPD
Attendance & Governance System.

It defines:
- Core entities
- Operational semantics
- Non-negotiable rules
- Enforcement philosophy

This document takes precedence over:
- UI behavior
- User assumptions
- Verbal instructions
- Informal practices

If a behavior contradicts this specification, the behavior is incorrect.

---

## Canonical Specification

# OPD Attendance & Governance System — System Specification (Authoritative)

## Scope
This specification defines a production-grade AppSheet governance system for outpatient clinic operations in a hospital environment, covering:
- Doctor roster and administrative status
- Clinic day scheduling and capacity legitimacy
- Daily attendance marking (Present / Late / Absent / OFF)
- Replacement assignment with strict eligibility (including OFF and Special Clinics exceptions)
- Auditability via append-only ledgers
- 35-day performance metrics derived from Attendance only

This system is live and operational. **Data integrity, auditability, and operational clarity override UI convenience at all times.**

---

## Non-Negotiable Principles
1. **Data rules > UI convenience**
2. **Attendance is the single source of truth** for daily expectation vs reality
3. **Ledger tables are append-only** (no silent edits, no manual deletes)
4. **OFF is expectation logic, not absence**
5. **Eligibility > validation errors** (hide invalid options; do not rely on warnings)
6. **No silent overwrites** of attendance reality
7. **No ambiguous states**
8. **No bots / no background automation**
9. All writes must be **attributable** (who, when)

---

## Authoritative Tables

### 1) Doctors (Master Table)
**Key:** `DoctorID`  
**Purpose:** Defines eligibility, scheduling assignment, clinic type, and derived performance metrics.

**Core columns**
- `DoctorID` (Key)
- `Name`
- `AssignedDay` (Enum: Saturday → Thursday) — may be blank for Special Clinics doctors
- `ClinicType` (Enum: General Clinics, Special Clinics)
- `AdminStatus` (Enum: Active, Leave, Clearance, Special) — **read-only**
- `Expected_35D`
- `Attended_35D` (Present ONLY)
- `Late_35D` (Late ONLY)
- `Missed_35D` (Absent ONLY)
- Additional admin/descriptive columns as needed

**Invariants**
- `AdminStatus` is never edited directly.
- Status changes are expressed only through Status Change History and projected back to Doctors.

---

### 2) Attendance (Operational Truth Table)
**Key:** `AttendanceID` — deterministic  
**Purpose:** The single operational truth for each doctor on each date.

**Deterministic key**
- `AttendanceID = DoctorID-YYYYMMDD`

**Core columns**
- `AttendanceID` (Key)
- `DoctorID` (Ref → Doctors)
- `Date`
- `AttendanceStatus` (Enum: Present, Late, Absent, OFF)
- `ReplacementBy` (Ref → Doctors)
- `LateReason`
- `TimeMarked`
- `MarkedBy`

**Invariants**
- Exactly one Attendance row per doctor per date is allowed.
- Attendance rows are created via governed actions (buttons) and/or tightly permissioned edits—never ad-hoc free editing.
- OFF cannot overwrite Present/Late/Absent.

---

### 3) Status Change History (Append-Only Ledger)
**Key:** `StatusChangeID`  
**Purpose:** Auditable administrative status authority for doctors.

**Core columns**
- `StatusChangeID` (Key)
- `DoctorID` (Ref → Doctors)
- `OldStatus`
- `NewStatus`
- `Proof` (Image)
- `Notes`
- `ChangedBy`
- `ChangedAt`
- `IsActive` (TRUE only for the latest row per doctor)

**Invariants**
- Rows are never edited or deleted manually.
- Corrections are implemented by adding new rows.
- Exactly one active row per doctor.

---

### 4) Behavior Cases
**Key:** `CaseID`  
**Purpose:** Document behavior incidents separately from attendance facts.

**Core columns**
- `CaseID` (Key)
- `DoctorID` (Ref → Doctors)
- `Date`
- `CaseType`
- `Description`
- `ReportedBy`
- `ActionTaken`
- `LinkedAttendanceID` (optional)

**Invariants**
- Behavior Cases do not mutate Attendance or metrics directly.

---

### 5) Users
**Key:** `UserID` or `Email`  
**Purpose:** Role and access control; ensures write accountability.

**Core columns**
- `UserID/Email`
- `Role` (AdminResident, Coordinator, Viewer, etc.)
- `IsActive`

**Invariants**
- All governed writes capture the user identity (e.g., MarkedBy/ChangedBy).

---

### 6) Clinic Days (Time–Date Control Table)
**Key:** `ClinicDayID` (or Date+ClinicType composite)  
**Purpose:** Defines operational legitimacy of dates and optional capacity context.

**Core columns**
- `ClinicDayID`
- `Date`
- `DayName`
- `ClinicType`
- `IsActive`
- Optional: `DeskCapacity` and notes

**Invariants**
- Attendance creation is valid only against active Clinic Days (as enforced by actions).

---

## Attendance Semantics (Strict)

| Status   | Meaning                          | Expected Today | Counts in Expected_35D | Counts in Missed_35D |
|----------|----------------------------------|----------------|------------------------|----------------------|
| Present  | Expected, attended normally       | Yes            | Yes                    | No                   |
| Late     | Expected, attended late           | Yes            | Yes                    | No                   |
| Absent   | Expected, did not attend          | Yes            | Yes                    | Yes                  |
| OFF      | Not expected today (planned)      | No             | No                     | No                   |

OFF is not Absent. OFF is a planned not-expected state.

---

## OFF Design (Critical)
OFF is implemented as:
- A real Attendance row with `AttendanceStatus = OFF`
- Created via a governed action (button), not ad-hoc editing
- Auditable and date-specific
- Excluded from negative metrics
- Removes doctor from daily expected workflow

**OFF must never overwrite Present/Late/Absent.**

---

## Workflow Visibility (Data-First)
Doctors who are OFF today must not enter the attendance flow.

**Expected-today slice must:**
- Include only Active doctors
- Match AssignedDay to today (for General Clinics)
- Exclude any doctor with an OFF row today

---

## Attendance Action Governance
All operational behavior must be enforced via:
- Slices
- Valid_If
- Only_If
- Actions

**Never via “Show_If hacks” that only hide UI without enforcing data rules.**

---

## Replacement Rules (Strict + Exceptions)

### Baseline Replacement Rules (Always enforced)
Replacement is:
- Not allowed if AttendanceStatus = Absent
- Not allowed for self
- Not allowed if replacement doctor is not Active
- Additional frequency limits may be enforced if defined

### Same-Day Restriction (Core)
Doctors scheduled on the same AssignedDay (General Clinics) cannot replace each other.

### OFF Exception (Core)
A doctor who is OFF today:
- CAN act as a replacement
- EVEN IF they share the same AssignedDay
Reason: OFF doctors are not consuming clinic capacity.

### Special Clinics Exception (Core)
If replacement doctor has `ClinicType = Special Clinics`, AssignedDay restrictions do not apply.

**Replacement eligibility must be implemented as eligibility filtering** (Valid_If) so ineligible doctors are not selectable.

---

## Metrics (Non-Negotiable)
Metrics are derived strictly from Attendance:

- Expected_35D counts only: Present, Late, Absent
- Attended_35D counts only: Present
- Late_35D counts only: Late
- Missed_35D counts only: Absent
- OFF is excluded entirely
- Replacement has no effect on Missed_35D

---

## Roles & Authority

### AdminResident
- Full access, 24/7
- Can mark OFF
- Can cancel OFF (explicit correction)
- Can correct attendance using governed mechanisms
- Can manage status changes via the ledger

### Coordinator
- Can mark Present / Late (time-bound)
- Can mark OFF (24/7 if policy requires)
- Must enforce rules with no exceptions
- No ability to bypass eligibility constraints

---

## Red Lines (Explicitly prohibited)
- Renaming doctors to encode logic
- Bypassing Attendance rows
- Collapsing OFF into Absent
- Breaking metric definitions
- Silent overwrites or manual metric edits
- Manual edits/deletes in ledger tables
- Background automation that changes attendance state

---

## Auditability Standard
Every operational change must be explainable with:
- What happened (Attendance row)
- Who did it (MarkedBy/ChangedBy)
- When it happened (TimeMarked/ChangedAt)
- Why it happened (context fields where relevant)
