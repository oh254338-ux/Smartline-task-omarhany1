# Attendance Aggregation Under Duplicate Events

> **Spec Task 13 · Day-1 checklist #14** · Smart Line — Block C (Algorithmic Problem Solving)
> Solution: `src/challenges/attendance-aggregation.js` · Tests: `tests/day1/attendance-aggregation.test.js`

---

## Problem statement

The mobile app sometimes sends duplicate "boarded" events due to retries on poor
connectivity, and events can arrive out of order. Given a raw stream of attendance
events for a trip, compute the final attendance state per employee.

**Input:** `events: [{ employeeId, status, timestamp, eventId }]` — up to 50,000 events
per trip in worst case (bulk backfill).

**Output:** map of `employeeId → finalStatus`.

**Rules:**
1. Duplicates by `eventId` are ignored (a retry is the same physical event).
2. Final status = the event with the **latest timestamp**.
3. Identical timestamps → **deterministic tie-break by eventId**.
4. Events with missing `employeeId` are unattributable → skipped.
5. Result is **idempotent and order-independent** for the same event set.

## Solution

```js
export function aggregateAttendance(events) {
  // Phase 1 — dedup by eventId (max-reduction on timestamp, order-independent).
  const byEventId = new Map();
  for (const ev of events) {
    if (ev == null) continue;
    const key = String(ev.eventId);
    const prev = byEventId.get(key);
    if (!prev || ev.timestamp > prev.timestamp) byEventId.set(key, ev);
  }

  // Phase 2 — per employee, keep max by (timestamp, eventId), order-independent.
  const byEmployee = new Map();
  for (const ev of byEventId.values()) {
    if (ev.employeeId == null) continue; // unattributable
    const prev = byEmployee.get(String(ev.employeeId));
    if (!prev) { byEmployee.set(String(ev.employeeId), ev); continue; }
    const better =
      ev.timestamp > prev.timestamp ||
      (ev.timestamp === prev.timestamp && compareEventIds(ev.eventId, prev.eventId) > 0);
    if (better) byEmployee.set(String(ev.employeeId), ev);
  }

  return Object.fromEntries([...byEmployee].map(([id, ev]) => [id, ev.status]));
}
```

## Why it is order-independent (the point of the task)

The computation is a **pure max-reduction**: for each key (eventId, then employeeId)
we keep the winner of a *total order that does not depend on arrival order*:

- among events with the same `eventId` → max by timestamp (same physical event);
- among events for the same employee → max by `(timestamp, eventId)`.

Re-running on the same event **set** — any order, any duplicates — yields the same
output. This is exactly the property the production ingestion pipeline needs on
Day 5/6 (idempotent replay).

**Tie-break caveat:** identical timestamp + different status → larger `eventId` wins.
Deterministic, but arbitrary; production mitigation is server-side clock
reconciliation.

**Clock skew:** timestamps are trusted as given. If device clocks skew badly,
"latest by timestamp" can be wrong; production should normalize ordering with server
receipt time and keep device timestamp as a field.

## Complexity

- **Time:** O(n) — two hash-map passes over the stream (better than the O(n log n)
  requirement).
- **Space:** O(n).

## Edge cases

| Case | Behavior | Test |
|---|---|---|
| Duplicate `eventId` (retry) | Counted once | `attendance-aggregation.test.js` |
| Out-of-order arrival | Latest timestamp wins regardless of arrival order | ✓ |
| Late event with EARLIER timestamp | Does not override the chronologically later one | ✓ |
| Identical timestamps, different statuses | Deterministic tie-break by eventId | ✓ |
| Missing `employeeId` | Skipped, never crashes | ✓ |
| Multi-employee stream | Each employee resolved independently | ✓ |
| Shuffled / replayed input | Identical output (idempotent) | ✓ |
| 100k events (bulk backfill) | Completes quickly (verified < 2 s) | ✓ |

## CLI usage

```bash
node src/challenges/attendance-aggregation.js --sample                  # demo
node src/challenges/attendance-aggregation.js input.json               # {"events": [...]}
```

Sample run (7 events → 3 employees):

```json
{
  "rule": "dedup by eventId; latest timestamp wins; ties broken by eventId; order-independent",
  "eventsReceived": 7,
  "attendance": { "e1": "absent", "e2": "boarded", "e3": "boarded" }
}
```

(`ev-1` duplicate retry ignored; `e1`'s later `absent` wins; `e2`'s later `boarded`
wins; the orphan event without `employeeId` dropped.)

## Verification

```text
# tests 9
# pass  9
# fail  0
```

Run: `npm test` (all Day-1 suites) or `node --test tests/day1/attendance-aggregation.test.js`.

## Traceability

- **FR-11 / FR-12** (duplicate suppression, latest-state view) in
  `docs/requirements/functional.md`.
- **NFR-12** (event idempotency, replay-safe ingestion) in
  `docs/requirements/non-functional.md`.
- **Assumption A-08** (dedup by eventId, tie-break by eventId) in
  `docs/requirements/assumptions.md`.
- The pattern recurs verbatim in Day 5/6 production ingestion (the report flags it
  as the key learning of the day).
