# Smart Line — Stated Assumptions & Risks (Task 03)

> **Purpose:** For 8 of the 18 questions in `clarification-questions.md`, state the
> assumption the design proceeds with today, the risk if it is wrong, and the design
> decision it will be traceable to on Day 3–4.
>
> All assumptions are mutually consistent: together they define the baseline system
> documented in `baseline.md` (Task 14).

---

## A-01 — One primary pickup location per employee (Q-01)

**Assumption:** An employee has exactly **one active pickup location** (their home or designated point), though the data model stores a history of changes.

**Risk if wrong:** If employees truly need multiple simultaneous pickup points, the Employee↔Pickup Location link becomes N:M, assignment must choose *per trip* which point applies, and the proximity-grouping algorithm (Task 11) must pick the right point per employee.

**Traceability:** Day 3 ERD — Employee and Pickup Location join table design; Day 4 — assignment API payload (`pickupLocationId` per assignment).

## A-02 — One branch per employee (Q-02)

**Assumption:** An employee belongs to **exactly one branch** at any point in time (transfers are modeled as a branch change with history).

**Risk if wrong:** Multi-branch employees break the "trip belongs to one branch" invariant — cross-branch trips would need per-passenger cost allocation and branch-based reporting would double-count. The overlap and capacity algorithms are unaffected, but queries and billing change.

**Traceability:** Day 3 ERD — Branch foreign key on Employee with effective-dated history; Day 2 — use case "view my company's employees" scoping.

## A-03 — Captain trip times use half-open intervals with NO buffer (Q-06)

**Assumption:** A captain's trips are considered conflicting **only if their time intervals share a positive duration** — intervals are half-open `[start, end)`, so a trip ending exactly when another starts (08:00/08:00) is **not** a conflict.

**Risk if wrong:** If operations actually requires a turnaround buffer, the conflict detector under-reports conflicts by the buffer amount and the captain may be late in practice; the fix is a single configurable `minGapMinutes` parameter, not an algorithm rewrite.

**Traceability:** Task 10 implementation + Trip Conflict Detector (`trip-overlap.js`, `trip-conflict-detector/`); Day 5 — scheduling service.

## A-04 — One vehicle per trip, no mid-trip reassignment in v1 (Q-09)

**Assumption:** A trip has exactly **one vehicle from publish to completion**; breakdowns are handled by cancelling/replacing the trip, not by swapping the vehicle mid-trip.

**Risk if wrong:** Mid-trip reassignment requires a temporal Trip→Vehicle assignment (start/end time), complicates conflict detection (vehicle conflicts, not just captain conflicts), and adds a "swap" event to the trip state machine.

**Traceability:** Day 3 ERD — Trip.VehicleId FK (simple, no temporal table); Day 4 — trip update API constraints; scope decision in `scope.md`.

## A-05 — Capacity counts unique employees (Q-10)

**Assumption:** When validating capacity (Task 09), **each employee counts once**, even if their ID appears multiple times in the assignment list (a data-entry duplication). Overflow = `uniqueEmployeeCount − capacity`.

**Risk if wrong:** If duplicate IDs are real bookings (e.g., the same person booked twice for two seats — unusual for individual riders), capacity validation would under-count and vehicles would be overbooked in practice.

**Traceability:** `vehicle-capacity.js` implementation; Day 4 — assignment API dedup rule; Day 5 — dispatch validation service.

## A-06 — Wallet is prepaid, per-employee, and cannot go negative (Q-12, Q-14)

**Assumption:** Each employee has a **prepaid wallet**; ride charges are **deducted only for attended (boarded) trips**; a charge that would make the balance negative **fails atomically**.

**Risk if wrong:** If the model is postpaid/company-billed, the "insufficient funds" failure path disappears and the consistency requirement shifts from atomic debits to invoice reconciliation — a significant NFR and schema change.

**Traceability:** Day 3 ERD — Wallet + Transaction tables with balance invariant; Day 5 — wallet service with idempotency keys (mirrors Task 13 idempotency thinking); NFR in `non-functional.md`.

## A-07 — All times are stored in the company's local timezone (Q-timezone)

**Assumption:** Trip `start`/`end` are interpreted in **the branch's local timezone** and converted to UTC instants for comparison; midnight-spanning trips are represented as end-after-start UTC instants.

**Risk if wrong:** If a single platform-wide timezone (or per-employee timezones) is required, conflict detection across timezone boundaries changes — but the comparison must still happen on absolute instants, so only the conversion layer changes, not the algorithm.

**Traceability:** Trip Conflict Detector (`trip-conflict-detector/`) epoch conversion; Day 4 — trip schema timezone field; NFR correctness.

## A-08 — Duplicate attendance events are deduplicated by eventId, and ties are broken deterministically (Q-13 context)

**Assumption:** Attendance events carry a unique `eventId`; duplicates (retries) are ignored; the final status per employee is the event with the **latest timestamp**, and on identical timestamps the event with the **larger eventId** wins (deterministic, order-independent).

**Risk if wrong:** If devices can emit identical timestamps for genuinely different states (e.g., "boarded" then "left" within the same second), a deterministic tie-break may pick the wrong final state; production mitigation is server-side clock reconciliation, which is out of v1 scope.

**Traceability:** `attendance-aggregation.js` implementation; Day 5/6 — event ingestion service (idempotency recurs as a production concern).

---

## Consistency check

| Assumption | Conflicts with | Reconciliation |
|---|---|---|
| A-03 (no buffer) | A-06 (charge only boarded) | Independent concerns: timing vs money; no interaction. |
| A-04 (no reassignment) | A-01 (pickup per employee) | Independent dimensions (vehicle vs passenger); no interaction. |
| A-07 (local tz) | A-03 (touch boundary) | Both defined on absolute UTC instants after conversion; consistent. |
