# Rules & Enforcement Specification

This document defines the operational rules governing the OPD Attendance &
Governance System.  
Rules are written in **plain language first**, then mapped to their **enforcement
surface** in AppSheet.

**Principle:**  
If a rule is not enforceable at the data level, it is not a real rule.

---

## 0. Rule Classification

Rules are grouped by intent:

- **Structural Rules** – protect data integrity
- **Semantic Rules** – define meaning of states
- **Visibility Rules** – control workflow exposure
- **Action Governance Rules** – control writes
- **Eligibility Rules** – constrain choices
- **Temporal Rules** – enforce time legitimacy
- **Metric Rules** – define counting logic
- **Authority Rules** – define who can do what

---

## 1. Structural Rules

### 1.1 One Attendance Row per Doctor per Date
**Rule**  
A doctor must have **exactly one** Attendance record per calendar date.

**Why**  
Prevents duplicates, silent overwrites, and retroactive manipulation.

**Enforcement**
- Deterministic primary key
- Action preconditions

**Authoritative Mechanism**
- `AttendanceID = DoctorID-YYYYMMDD`
- Attendance creation actions blocked if a row already exists for that date.

---

### 1.2 Deterministic AttendanceID
**Rule**  
Attendance IDs are derived, not user-generated.

**Why**  
Ensures uniqueness without reliance on user behavior.

**Enforcement**
- Key column formula

---

### 1.3 Append-Only Ledgers
**Rule**  
Ledger tables (Status Change History) are append-only.

**Why**  
Auditability and dispute reconstruction.

**Enforcement**
- No edit/delete permissions
- Corrections require new rows

---

## 2. Attendance Semantics Rules

### 2.1 AttendanceStatus Meaning (Strict)
| Status  | Meaning |
|--------|---------|
| Present | Expected and attended |
| Late | Expected and attended late |
| Absent | Expected and did not attend |
| OFF | **Not expected today** |

**Why**  
Metrics and accountability depend on semantic precision.

---

### 2.2 OFF Is Not Absent
**Rule**  
OFF represents a planned, non-expected state and must never be treated as absence.

**Why**  
Conflating OFF with Absent corrupts metrics and incentivizes manipulation.

**Enforcement**
- OFF excluded from Expected_35D and Missed_35D
- OFF handled as a distinct AttendanceStatus

---

### 2.3 OFF Cannot Overwrite Attendance
**Rule**  
OFF cannot replace Present, Late, or Absent.

**Why**  
Prevents retroactive sanitization of attendance history.

**Enforcement**
- OFF creation action blocked if Attendance row already exists.

---

## 3. Visibility & Workflow Rules

### 3.1 Today Expected Doctors Slice
**Rule**  
Doctors who are OFF today must not appear in today’s attendance workflow.

**Why**  
Prevents invalid actions and coordinator confusion.

**Enforcement**
- Slice excludes doctors with OFF Attendance for today.

---

### 3.2 Workflow Is Slice-Driven
**Rule**  
Users act only on data surfaced by slices, not raw tables.

**Why**  
Visibility is part of governance.

---

## 4. Action Governance Rules

### 4.1 Present / Late Creation
**Rule**
- Present/Late can only be marked if no Attendance row exists for today.

**Enforcement**
- Action `Only_If` conditions

---

### 4.2 OFF Creation
**Rule**
- OFF is created via a dedicated action.
- OFF is date-specific.
- OFF is auditable.

**Enforcement**
- Action writes Attendance row with:
  - Status = OFF
  - TimeMarked
  - MarkedBy

---

### 4.3 OFF Cancellation
**Rule**
- OFF can be cancelled only if AttendanceStatus = OFF.
- Admin-only.

**Why**
Prevents rewriting attendance facts.

---

## 5. Replacement Rules

### 5.1 Baseline Restrictions
Replacement is **not allowed** if:
- AttendanceStatus = Absent
- Replacement doctor is self
- Replacement doctor is not Active

---

### 5.2 Same-Day Restriction
**Rule**
Doctors assigned to the same clinic day cannot replace each other.

**Why**
Prevents double consumption of clinic capacity.

---

### 5.3 OFF Exception
**Rule**
A doctor who is OFF today may act as a replacement,
even if they share the same AssignedDay.

**Why**
OFF doctors are not consuming clinic capacity.

**Enforcement**
- Replacement eligibility requires OFF Attendance row for today.

---

### 5.4 Special Clinics Exception
**Rule**
Doctors whose ClinicType = Special Clinics are exempt from AssignedDay constraints.

**Why**
They are not part of the General Clinics scheduling model.

---

## 6. Temporal Rules

### 6.1 Clinic Day Validity
**Rule**
Attendance is valid only on active Clinic Days.

**Enforcement**
- Attendance actions reference Clinic Days table.

---

### 6.2 Time-Bound Attendance (Coordinator)
**Rule**
Coordinators can mark attendance only within allowed time windows.

**Enforcement**
- Action `Only_If` conditions

---

## 7. Metric Rules (35-Day Window)

### 7.1 Expected_35D
Counts:
- Present
- Late
- Absent

Excludes:
- OFF

---

### 7.2 Attended_35D
Counts:
- Present only

---

### 7.3 Late_35D
Counts:
- Late only

---

### 7.4 Missed_35D
Counts:
- Absent only

**Invariant**
Replacement has no effect on Missed_35D.

---

## 8. Role & Authority Rules

### 8.1 AdminResident
- Full access
- Can mark and cancel OFF
- Can edit AttendanceStatus
- 24/7 authority

---

### 8.2 Coordinator
- Can mark Present / Late / OFF
- Time-bound for attendance
- Cannot bypass rules

---

### 8.3 Accountability
**Rule**
Every operational write must be attributable.

**Enforcement**
- USEREMAIL() captured in MarkedBy / ChangedBy

---

## 9. Explicit Non-Rules (Out of Scope)

The system does **not**:
- Infer intent
- Automate punishments
- Apply disciplinary logic automatically
- Modify metrics manually
- Allow informal exceptions
