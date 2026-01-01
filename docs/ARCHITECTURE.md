## Core Data Model

The system is built around a small number of authoritative tables with
clear ownership and non-overlapping responsibilities.
Each table exists to answer one class of operational question only.

---

### 1) Doctors (Master Table)
**Role:** Authoritative registry of doctors eligible to participate in OPD clinics.  
**Key:** `DoctorID`

**Purpose**
- Defines *who* the doctor is in the system
- Defines *when* they are expected (AssignedDay)
- Holds derived performance metrics
- Acts as a reference point for all operational tables

**Key fields**
- `DoctorID` (Primary Key)
- `Name`
- `AssignedDay` (Enum: Saturday → Thursday)
- `ClinicType` (Enum: General Clinics, Special Clinics)
- `AdminStatus` (Active, Leave, Clearance, Special) **read-only**
- Rolling 35-day metrics:
  - `Expected_35D`
  - `Attended_35D`
  - `Late_35D`
  - `Missed_35D`

**Invariants**
- `AdminStatus` is never edited directly.
- Any change in administrative state is reflected via the Status Change History ledger.
- Metrics are *derived*, never manually edited.

---

### 2) Attendance (Operational Truth Table)
**Role:** The single source of truth for daily expectation vs. attendance reality.  
**Key:** `AttendanceID` (deterministic)

**Purpose**
- Records whether a doctor was expected on a specific date
- Records what actually happened
- Serves as the foundation for metrics, disputes, and audits

**Deterministic key**
- `AttendanceID = DoctorID-YYYYMMDD`
- Guarantees **exactly one row per doctor per date**

**Key fields**
- `AttendanceID`
- `DoctorID` (Ref → Doctors)
- `Date`
- `AttendanceStatus` (Present, Late, Absent, OFF)
- `ReplacementBy` (Ref → Doctors)
- `LateReason`
- `TimeMarked`
- `MarkedBy`

**AttendanceStatus semantics (strict)**
- **Present:** Expected today, attended normally
- **Late:** Expected today, attended late
- **Absent:** Expected today, did not attend
- **OFF:** Not expected today (planned capacity / operational decision)

**Invariants**
- Attendance rows are created via governed actions only.
- No silent overwrites.
- OFF cannot overwrite Present / Late / Absent.
- Replacement has no effect on Missed metrics.

---

### 3) Status Change History (Audit Ledger)
**Role:** Append-only ledger of administrative status changes.  
**Key:** `StatusChangeID`

**Purpose**
- Provides a complete, tamper-resistant history of doctor status changes
- Protects management decisions during disputes or audits

**Key fields**
- `StatusChangeID`
- `DoctorID` (Ref → Doctors)
- `OldStatus`
- `NewStatus`
- `Proof` (Image)
- `Notes`
- `ChangedBy`
- `ChangedAt`
- `IsActive` (TRUE only for latest status)

**Invariants**
- Rows are never edited or deleted manually.
- Corrections are implemented by appending new rows.
- Exactly one row per doctor has `IsActive = TRUE`.

---

### 4) Behavior Cases
**Role:** Documentation of behavioral or disciplinary incidents related to clinic operations.  
**Key:** `CaseID`

**Purpose**
- Separates **behavioral governance** from attendance facts
- Prevents mixing discipline with operational data
- Supports escalation, review, and pattern recognition

**Key fields**
- `CaseID`
- `DoctorID` (Ref → Doctors)
- `Date`
- `CaseType` (Enum: Misconduct, Repeated Absence, Conflict, Other)
- `Description`
- `ReportedBy`
- `ActionTaken`
- `LinkedAttendanceID` (optional)

**Invariants**
- Behavior Cases do not alter Attendance or metrics directly.
- They provide **context**, not punishment logic.
- Decisions based on behavior must still be reflected through formal status changes or rules.

---

### 5) Users
**Role:** Access control and accountability layer.  
**Key:** `UserID` or `Email`

**Purpose**
- Defines *who* is interacting with the system
- Governs permissions and action visibility
- Ensures every write action is attributable

**Key fields**
- `UserID / Email`
- `Role` (AdminResident, Coordinator, Viewer, etc.)
- `IsActive`
- Optional scope fields (clinic, department)

**Invariants**
- All write actions record `MarkedBy = USEREMAIL()`.
- Role-based access is enforced through:
  - Action visibility
  - Only_If conditions
  - Slice access

---

### 6) Clinic Days (Time–Date Control Table)
**Role:** Operational calendar defining valid clinic days and capacity context.  
**Key:** `ClinicDayID` or composite (Date + Clinic)

**Purpose**
- Defines when clinics are officially open
- Anchors attendance expectations to real operational days
- Prevents attendance marking on invalid dates

**Key fields**
- `ClinicDayID`
- `Date`
- `DayName`
- `ClinicType`
- `IsActive`
- Optional capacity descriptors

**Invariants**
- Attendance is only valid against active Clinic Days.
- OFF decisions are contextualized by clinic capacity, not personal preference.

---

## Separation of Concerns (Why This Matters)

- **Doctors** define eligibility and assignment
- **Attendance** defines daily operational reality
- **Status Change History** defines administrative authority
- **Behavior Cases** define conduct, not attendance
- **Users** define accountability
- **Clinic Days** define temporal legitimacy

No table is overloaded.
No responsibility is ambiguous.
This separation is what allows the system to remain auditable, explainable,
and resistant to informal overrides.


## Data Relationships & Referential Integrity

This system is intentionally relational and constraint-driven.
Relationships are explicit, directional, and enforced to prevent ambiguity,
duplicate authority, or silent state corruption.

---

### Entity Relationship Overview (Logical)

Users ───────────────┐ │ ▼ Attendance  ◀────────── Doctors │                  ▲ │                  │ ▼                  │ Behavior Cases               │ │ Status Change History ────────────────┘
Clinic Days ─────────▶ Attendance

---

### Relationship Semantics (Authoritative)

#### Doctors ↔ Attendance (1 → Many)
- A single doctor can have many Attendance records.
- A single Attendance record **must** reference exactly one Doctor.
- Attendance cannot exist without a valid Doctor reference.

**Constraint**
- Referential integrity is enforced via `Ref → Doctors`.
- Deleting or altering Doctors does **not** cascade into Attendance.

**Rationale**
Attendance is immutable historical fact; it must not be orphaned or rewritten.

---

#### Doctors ↔ Status Change History (1 → Many)
- A doctor can have multiple historical status changes.
- Exactly **one** status row per doctor is active at any time (`IsActive = TRUE`).

**Constraint**
- No updates or deletes; append-only.
- Doctors.AdminStatus is a **projection**, not a source of truth.

**Rationale**
This guarantees explainability of administrative decisions under audit.

---

#### Doctors ↔ Behavior Cases (1 → Many)
- Behavioral or disciplinary events are linked to a doctor.
- Behavior Cases may optionally reference Attendance for context.

**Constraint**
- Behavior Cases **do not** mutate Attendance or metrics.
- They are descriptive, not operational.

**Rationale**
Separating conduct from attendance prevents subjective punishment logic
from contaminating operational truth.

---

#### Users ↔ Attendance / Status / Behavior (1 → Many)
- Users represent actors in the system.
- Every operational write is attributable to a User.

**Constraint**
- `MarkedBy`, `ChangedBy`, and `ReportedBy` fields are mandatory on writes.
- Role-based access gates actions, not data visibility alone.

**Rationale**
Accountability is enforced structurally, not procedurally.

---

#### Clinic Days ↔ Attendance (1 → Many)
- Each Clinic Day defines whether attendance is operationally valid.
- Attendance records must align with active Clinic Days.

**Constraint**
- Attendance creation is blocked if Clinic Day is inactive.
- OFF decisions are contextualized by Clinic Day capacity.

**Rationale**
Time is a first-class constraint. Attendance outside valid clinic days
is structurally invalid.

---

### Replacement Relationship (Self-Referential, Guarded)

#### Attendance ↔ Doctors (ReplacementBy)
- `ReplacementBy` is a self-referential relationship back to Doctors.
- This relationship is **highly constrained** by eligibility rules.

**Constraints**
- Replacement is not allowed if AttendanceStatus = Absent.
- Replacement cannot reference self.
- Replacement doctor must be Active.
- Same-day replacement is blocked unless replacement doctor has OFF for that date.
- Special Clinics doctors bypass AssignedDay constraints by design.

**Rationale**
Replacement consumes capacity. Eligibility rules ensure capacity
is not double-booked and responsibility remains traceable.

---

### Cardinality Summary

| Relationship                         | Cardinality | Enforced By              |
|-------------------------------------|-------------|--------------------------|
| Doctors → Attendance                | 1 → Many    | Ref + Deterministic Key |
| Doctors → Status Change History     | 1 → Many    | Append-only ledger      |
| Doctors → Behavior Cases            | 1 → Many    | Ref                     |
| Users → Operational Tables          | 1 → Many    | MarkedBy / Role gating  |
| Clinic Days → Attendance            | 1 → Many    | Action preconditions    |
| Doctors → Attendance (Replacement)  | Conditional | Valid_If eligibility    |

---

### Data Ownership & Authority Boundaries

| Table                | Owns Truth About                |
|----------------------|---------------------------------|
| Doctors              | Eligibility & assignment         |
| Attendance            | Daily operational reality        |
| Status Change History| Administrative authority         |
| Behavior Cases        | Conduct & escalation context     |
| Users                | Accountability                   |
| Clinic Days           | Temporal legitimacy              |

No table overlaps authority.
No table infers another’s responsibility.
This is deliberate.

---

### Failure Mode Containment

The relational design ensures:
- A bad user action cannot silently corrupt history
- A wrong assumption cannot propagate across tables
- Disputes can always be reconstructed from data alone

This is a governance system, not a CRUD application.
