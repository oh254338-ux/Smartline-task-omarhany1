# Smart Line — Functional Requirements Draft (Task 05)

> **Purpose:** 12 testable functional requirements covering **trip creation, employee
> assignment, and attendance tracking** — the three flows company managers care about
> most ("which of my employees actually boarded this morning").
>
> Every requirement is written as `The system shall ...` and is individually testable.
> No implementation detail (no table names, no endpoint names). Edge cases (walk-on
> boarding, double assignment) are handled explicitly.

---

## Trip creation

**FR-01 — Trip creation from route template.**
The system shall allow an authorized company manager to create a trip by selecting an existing route of their branch, a date, a time window, a vehicle, and a captain.

**FR-02 — Capacity validation at publish.**
The system shall prevent publishing a trip whose assigned employee count exceeds the selected vehicle's seat capacity, and shall report the overflow count to the publisher.

**FR-03 — Captain conflict check at publish.**
The system shall prevent publishing a trip if its time window overlaps any other trip already assigned to the same captain (per the half-open interval rule in `assumptions.md` A-03).

**FR-04 — Trip state transitions.**
The system shall track each trip through the states *draft → published → in-progress → completed* (and *cancelled*), and shall allow only the authorized actor to perform each transition (e.g., only the company manager or admin may cancel a published trip).

## Employee assignment

**FR-05 — Assign employee to trip.**
The system shall allow an authorized manager to assign an employee to a published trip, provided the employee belongs to the trip's branch (or is covered by a cross-branch rule) and the trip remains within vehicle capacity after the assignment.

**FR-06 — Assignment uniqueness per trip.**
The system shall ensure an employee is listed at most once per trip, even if the assignment is submitted multiple times (duplicate submission must be idempotent).

**FR-07 — Double-assignment visibility.**
The system shall warn the manager when an employee is assigned to two trips whose time windows overlap on the same day, and shall require explicit confirmation before both assignments can coexist.

**FR-08 — Unassign from trip.**
The system shall allow an authorized manager to remove an employee from a published trip, and shall free the seat for re-assignment, provided no attendance record exists for that employee on that trip.

## Attendance tracking

**FR-09 — Boarded marking for assigned employees only.**
The system shall allow an employee to be marked as *boarded* on a trip only if they are assigned to that trip; any attempt to mark an unassigned employee (walk-on) shall be rejected, except when an authorized captain or manager explicitly records a walk-on approval.

**FR-10 — Captain confirmation of boarding.**
The system shall allow the captain of a trip to confirm or deny the boarding status of each assigned employee during the trip window.

**FR-11 — Duplicate attendance event suppression.**
The system shall accept duplicate boarding events (e.g., mobile retries) without changing the final attendance state, using a unique event identifier so each physical boarding is counted exactly once.

**FR-12 — Attendance visibility per company.**
The system shall present each company manager with a daily attendance view — for each of their trips, which assigned employees boarded, did not board, or were unassigned — and the view shall reflect the latest event state even when events arrive out of order.

---

## Edge cases explicitly covered

| Edge case | Handling |
|---|---|
| Walk-on (employee boards a trip they were not assigned to) | Rejected by default (FR-09); explicit captain/manager approval path exists and is audit-logged. |
| Employee assigned to two overlapping trips the same day | Warning + explicit confirmation (FR-07); the system still tracks both, but the manager owns the decision. |
| Duplicate boarding events / out-of-order events | Idempotent by event ID; latest timestamp wins (FR-11, FR-12) — mirrors Task 13. |
| Trip at capacity | Publishing and assignment blocked with overflow count (FR-02, FR-05). |
| Unassign after boarding | Blocked (FR-08) — attendance truth outranks assignment cleanup. |

## Testability note

Each FR maps to at least one acceptance test in `tests/day1/` context (FR-02 ↔
vehicle capacity tests; FR-03 ↔ trip overlap tests; FR-11 ↔ attendance aggregation
tests). FRs with no algorithmic counterpart (FR-01, 04, 05, 08, 10, 12) are specified
for Day 2 use cases and Day 4 API contract tests.
