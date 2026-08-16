# Smart Line — Clarification Questions for the Product Owner (Task 02)

> **Purpose:** Surface every business rule the Morning Brief left ambiguous **before** any
> schema, API, or algorithm is committed to. Each question is answerable by a
> non-technical stakeholder in one sentence, and each is followed by *why the answer
> changes the design*.
>
> 18 questions across the 5 required domain groups. IDs (Q-xx) are referenced by
> `assumptions.md` (Task 03) and the baseline (Task 14).

---

## A. Employees

**Q-01. Can one employee be picked up from more than one location (e.g., home and work site)?**
*Design impact:* Decides whether the Employee↔Pickup Location link is 1:1 or N:M, which changes the assignment algorithm and the database model (Day 3).

**Q-02. Can an employee belong to two branches at the same time (e.g., works at two plants)?**
*Design impact:* If yes, trip assignment, reporting, and cost allocation can no longer assume a single branch context — routing and billing queries change shape.

**Q-03. Who is allowed to assign an employee to a trip — the company manager, the captain, dispatch, or any combination?**
*Design impact:* Determines authorization rules and which actor owns the "assigned but not boarded" state, affecting both functional requirements and the use-case model (Day 2).

**Q-04. What happens to an employee who is not assigned to any trip on a working day — is that allowed or is it an error to flag?**
*Design impact:* Defines whether "unassigned employee" is a normal state (no alert) or a KPI alert, which changes notification and reporting rules.

## B. Trips

**Q-05. Can a single trip serve employees from more than one branch?**
*Design impact:* If yes, trips cannot be assumed to belong to exactly one branch — cost allocation, route ownership, and capacity checks must handle cross-branch passengers.

**Q-06. If two of a captain's trips touch exactly at the same minute (one ends 08:00, the next starts 08:00), is that acceptable or must there be a buffer?**
*Design impact:* Fixes the overlap boundary semantics used by Task 10 and the Trip Conflict Detector; a buffer changes which pairs are flagged as conflicts.

**Q-07. Can a trip be split into multiple runs of the same route on the same day, and is a "trip" then one run or the whole day?**
*Design impact:* Defines the granularity of the Trip entity, which cascades into attendance aggregation, conflict detection, and wallet charging.

**Q-08. Who may cancel or reschedule a trip after it is published — and must affected employees be notified?**
*Design impact:* Sets the state machine of a trip (draft/published/cancelled/done) and triggers notification requirements — a core functional requirement.

## C. Vehicles

**Q-09. Is a vehicle always assigned to exactly one trip at a time, or can it be reassigned mid-trip if a bus breaks down?**
*Design impact:* Reassignment mid-trip means the Trip→Vehicle link is temporal (start/end of assignment), not a simple FK — changes the data model and the conflict rules.

**Q-10. When counting seats, does a duplicate employee entry in the assignment list count as one seat or two?**
*Design impact:* Directly defines Task 09's deduplication behavior — "capacity" must match the physical truth (one person = one seat).

**Q-11. Who decides a vehicle is unavailable (maintenance, breakdown) — the captain, the manager, or the admin, and how is that recorded?**
*Design impact:* Vehicle availability becomes a gating rule for trip assignment (scope edge case), requiring a status field plus an audit trail.

## D. Wallet

**Q-12. Is the wallet prepaid (employee tops up, rides are deducted) or postpaid (company is billed and the wallet is a balance sheet)?**
*Design impact:* Determines whether wallet transactions face double-spend risk (prepaid) or invoicing logic (postpaid) — a fundamental consistency requirement.

**Q-13. Are employees charged for no-show (assigned but not boarded) or only for attended trips?**
*Design impact:* Binds wallet charging to attendance truth, which makes attendance idempotency (Task 13) a money-critical correctness issue.

**Q-14. Can a wallet balance go negative, or must every charge fail if the balance is insufficient?**
*Design impact:* Defines the atomicity/consistency rule for wallet debits and the error handling the employee app must surface.

## E. Complaints

**Q-15. What is the expected resolution target for a complaint (e.g., same day, 48 hours), and who owns the deadline?**
*Design impact:* Turns complaints into a measurable SLA with escalation rules, driving the state machine and notification design.

**Q-16. Can a complaint be raised about a trip the employee was not assigned to (e.g., witnessed an incident)?**
*Design impact:* Relaxes the complaint↔trip reference from "assigned trips only", which affects validation rules and the call center's lookup workflow.

**Q-17. Can a company manager raise complaints on behalf of their employees, or only employees themselves?**
*Design impact:* Adds a second reporter actor to complaints, which changes permissions, UI, and the escalation path to the admin.

**Q-18. Should complaints be anonymous by default, or must the reporter always be identified?**
*Design impact:* Anonymous reporting changes audit requirements, contact-back flows, and possibly compliance constraints on data retention.

---

## Cross-cutting questions deliberately deferred (not part of the 18)

- Timezone convention for trip times (assumption in `assumptions.md`).
- Notification channels and opt-out rules (assumption; refined in Day 2 use cases).
- Data retention period for attendance/complaints (assumption; NFR security section).
