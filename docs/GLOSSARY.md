# Glossary

This glossary defines key terms as used **within this system only**.
Common language meanings do not apply if they conflict with definitions here.

---

### Attendance
A dated operational record representing whether a doctor was expected on a
specific day and what actually occurred.

Attendance is an **event**, not a property.

---

### Attendance Status
The categorical outcome recorded in an Attendance record.

Valid values:
- **Present** — Expected and attended
- **Late** — Expected and attended late
- **Absent** — Expected and did not attend
- **OFF** — Not expected on that date

---

### OFF
A planned, auditable state indicating a doctor was **not expected** to attend
on a specific clinic day.

OFF is:
- Date-specific
- Operational
- Capacity-driven

OFF is **not** absence and carries no negative metric weight.

---

### Expected
A doctor is considered expected on a date if:
- The clinic day is active
- The doctor is scheduled
- No OFF record exists for that date

---

### Replacement
A doctor attending in place of another doctor on a clinic day.

Replacement:
- Consumes clinic capacity
- Is governed by strict eligibility rules
- Does not modify absence metrics

---

### Replacement Eligibility
The set of enforced rules determining whether a doctor may act as a replacement
on a specific date.

Eligibility is enforced **before selection**, not after.

---

### Deterministic Key
A primary key derived from known values (e.g., DoctorID + Date) rather than
generated randomly.

Used to enforce structural uniqueness.

---

### Ledger (Append-Only)
A table where records are never edited or deleted.
Corrections are performed by adding new records.

Used to preserve auditability and historical accuracy.

---

### AdminStatus
The administrative state of a doctor (e.g., Active, Leave).

AdminStatus:
- Is not edited directly
- Is derived from Status Change History

---

### Status Change History
An append-only ledger recording administrative status transitions of doctors.

Provides authority traceability.

---

### Coordinator
An operational role responsible for attendance marking and enforcing daily
clinic rules within defined time windows.

---

### AdminResident
A supervisory role with extended authority, including corrections and OFF
cancellation.

---

### Clinic Day
A date on which a clinic is operationally active and attendance is valid.

Clinic Days define temporal legitimacy.

---

### Metric (35-Day)
A rolling performance indicator calculated from Attendance data over the
previous 35 days.

Metrics are derived, not stored.

---

### Eligibility Rule
A rule that determines whether an action or choice is allowed **before** it is
presented to the user.

Used to prevent invalid actions proactively.

---

### Silent Overwrite
Any modification of historical data that leaves no explicit audit trail.

Explicitly prohibited.

---

### Governance System
A system whose primary function is to enforce rules, preserve integrity, and
reduce disputes—not merely to collect data.

---
