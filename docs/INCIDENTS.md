# Incident Playbooks

This document defines how operational incidents are handled without violating
system governance or auditability.

---

## Incident 1 — Doctor disputes attendance status

**Claim**
> “أنا كنت موجود”

**Resolution Path**
1. Check Attendance record
2. Check TimeMarked and MarkedBy
3. Attendance record is authoritative

**Rule**
- No retroactive correction without governed action
- Verbal claims do not override records

---

## Incident 2 — Coordinator marked wrong status

**Resolution Path**
1. Identify incorrect Attendance row
2. AdminResident performs correction via governed action
3. Audit trail remains intact

**Rule**
- No silent edits
- No delete-and-recreate patterns

---

## Incident 3 — Doctor disputes OFF assignment

**Resolution Path**
1. Check OFF Attendance record
2. Verify TimeMarked and capacity context
3. Explain OFF as operational, not personal

**Rule**
- OFF decisions are capacity-based
- Personal preference is irrelevant

---

## Incident 4 — Replacement eligibility dispute

**Resolution Path**
1. Review eligibility rules
2. Verify OFF exception or Special Clinics status
3. Decision is rule-based, not negotiable

---

## Incident 5 — Request for exception

**Resolution Path**
- Decline politely
- Reference system rules
- No ad-hoc overrides

**Rule**
Once exceptions are normalized, governance collapses.
