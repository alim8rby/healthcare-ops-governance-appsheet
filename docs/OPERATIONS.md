# Operations Guide (Production Runbook)

This document defines how the OPD Attendance & Governance System is introduced,
operated, and defended in a live hospital environment.

This is not a user manual.
This is a control and survival document.

---

## 1. Operating Assumptions

The system is deployed in an environment where:
- Manual practices are entrenched
- Informal exceptions are normalized
- Digitization is often perceived as a threat
- Accountability is resisted if framed incorrectly

Therefore:
- The system must protect management first
- Rules must be enforced structurally, not socially
- Disputes must be resolvable from data alone

---

## 2. Rollout Strategy (Non-Negotiable)

### 2.1 Language Framing
Never describe the system as:
- “Digitization”
- “Transparency”
- “Monitoring”

Always describe it as:
- Organization
- Documentation
- Dispute reduction
- Decision support
- Protection of management decisions

---

### 2.2 Rollout Order
Rollout must follow this sequence:

1. Hospital Manager (private walkthrough)
2. General Secretariat / senior authority (if applicable)
3. Coordinators
4. Doctors

**Never** deploy bottom-up.

---

### 2.3 Initial Scope
- Start with one clinic day or unit
- Enforce rules strictly from day one
- No “trial exceptions”

Early consistency prevents later political rollback.

---

## 3. Coordinator Operations

### 3.1 Core Responsibilities
Coordinators are responsible for:
- Correct attendance marking
- Correct OFF usage when desk capacity is exceeded
- Enforcing one clinic per doctor per week (including OFF)
- Enforcing replacement eligibility without exceptions

### 3.2 What Coordinators Are NOT Allowed to Do
- Override system rules verbally
- Mark attendance outside governed actions
- Invent informal exceptions
- Backfill attendance without justification

---

## 4. OFF Operational Policy

### 4.1 When OFF Is Used
OFF is used when:
- Clinic desk capacity is exceeded
- Operational constraints require reducing expected doctors
- Planning requires removing a doctor from expectation

OFF is **not** a punishment.

---

### 4.2 OFF Is Mandatory, Not Optional
When capacity cannot accommodate all doctors:
- OFF must be applied
- Personal preference is irrelevant
- Verbal agreements are invalid

---

### 4.3 OFF Accountability
Every OFF action records:
- Who marked it
- When it was marked

This protects coordinators and management alike.

---

## 5. Replacement Handling

### 5.1 Replacement Is Capacity Consumption
Replacement is treated as consumption of clinic capacity.

### 5.2 Replacement Enforcement
Replacement must obey:
- Active status
- Not self
- Not Absent
- Same-day restriction unless OFF exception applies
- Special Clinics exemption where defined

No verbal approvals override eligibility rules.

---

## 6. Incident Handling & Disputes

### 6.1 General Principle
When disputes occur:
- Refer to Attendance as the source of truth
- Refer to Status Change History for authority
- Refer to timestamps and users, not narratives

---

### 6.2 Common Scenarios

#### “Doctor says they were present”
Response:
- Check Attendance record
- If absent, system reflects expected vs reality
- No retroactive correction without formal process

#### “Coordinator marked wrong status”
Response:
- Correction must leave audit trail
- No silent edits
- Use governed correction actions only

#### “Why was I OFF?”
Response:
- OFF is an operational decision
- Capacity-based, not personal
- Recorded and auditable

---

## 7. Administrative Corrections

### 7.1 Attendance Corrections
- Only AdminResident can perform corrections
- Corrections must be explicit and logged

### 7.2 Status Corrections
- Never overwrite AdminStatus
- Append a new Status Change History row

---

## 8. Monitoring & Review

### 8.1 Weekly Review (Recommended)
Review:
- OFF frequency
- Replacement patterns
- Repeated Absent cases
- Behavior Case accumulation

This is for **pattern detection**, not punishment.

---

## 9. Failure Modes & Responses

| Failure Mode | System Response |
|--------------|----------------|
| Duplicate attendance attempt | Blocked structurally |
| Same-day replacement abuse | Blocked by eligibility |
| Retroactive OFF | Blocked |
| Manual override attempt | No write surface |
| Dispute escalation | Reconstruct from data |

---

## 10. Operational Red Lines

The following must **never** be done:
- Manual database edits
- Backdated attendance without record
- Metric manipulation
- Informal “just this once” exceptions

Once exceptions are normalized, governance collapses.

---

## 11. Success Criteria

The system is considered successful when:
- Disputes are resolved by data, not arguments
- Coordinators stop improvising
- Management stops firefighting
- Rules enforce themselves
