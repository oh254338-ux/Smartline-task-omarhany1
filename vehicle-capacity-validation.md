# Vehicle Capacity Validation

> **Spec Task 09 · Day-1 checklist #10** · Smart Line — Block C (Algorithmic Problem Solving)
> Solution: `src/challenges/vehicle-capacity.js` · Tests: `tests/day1/vehicle-capacity.test.js`

---

## Problem statement

Vehicles have fixed seat capacities. Dispatch sometimes over-assigns employees to a
route without realizing it. Given a vehicle's capacity and a list of trips, each with
a list of assigned employee IDs, determine which trips (if any) exceed capacity.

**Input:** `capacity: int`, `trips: [{ tripId, employeeIds: [] }]` — up to 10,000 trips per call.

**Output:** list of violating trips with the overflow count.

**Example:** capacity = 14, trip with 15 employee IDs → overflow of 1.

## Solution

```js
export function findCapacityViolations(capacity, trips) {
  if (!Number.isInteger(capacity) || capacity < 0) {
    throw new Error(`capacity must be a non-negative integer, got ${capacity}`);
  }
  if (!Array.isArray(trips)) throw new Error('trips must be an array');

  const violations = [];

  for (const trip of trips) {
    const ids = Array.isArray(trip.employeeIds) ? trip.employeeIds : [];
    // Deduplicate: a Set is O(1) per insert, so the whole trip is O(|employeeIds|).
    const unique = new Set(ids);
    const uniqueAssigned = unique.size;
    const overflow = uniqueAssigned - capacity;
    if (overflow > 0) {
      violations.push({ tripId: trip.tripId, uniqueAssigned, capacity, overflow });
    }
  }

  // Deterministic output ordering regardless of input order.
  violations.sort((a, b) => String(a.tripId).localeCompare(String(b.tripId)));
  return violations;
}
```

## Design decisions

| Decision | Rationale |
|---|---|
| **Duplicate IDs count once** | One physical person occupies one seat (assumption A-05). A repeated ID in the assignment list is a data-entry duplication, not a second passenger. |
| `overflow = uniqueAssigned − capacity`, reported only when > 0 | The output contains only actionable violations; `capacity 14 / 15 assigned` → `overflow: 1`. |
| Capacity 0 is valid input | Any assigned employee (≥1 unique) violates; an empty list never does. |
| Sorted output by `tripId` | Deterministic regardless of input order — stable for tests and diffing. |
| Input validation | Negative or fractional capacity is rejected with a clear error. |

## Complexity

- **Time:** O(n) over the **total number of employee assignments** — every ID is
  inserted into a `Set` exactly once (O(1) per insert).
- **Space:** O(m), where m = total assignments (the per-trip Sets, bounded by input size).

## Edge cases

| Case | Behavior | Test |
|---|---|---|
| Duplicate employee IDs within a trip | Counted once → 6 raw entries with 3 unique at capacity 3 = OK | `vehicle-capacity.test.js` |
| Empty employee list | Never violates, any capacity | ✓ |
| Capacity 0 | Any unique employee → overflow = uniqueAssigned | ✓ |
| Exactly at capacity | Not flagged | ✓ |
| Empty trips list | Returns `[]` | ✓ |
| Negative / fractional capacity | Throws | ✓ |
| 10,000 trips | Completes quickly (verified < 2 s) | ✓ |

## CLI usage

```bash
node src/challenges/vehicle-capacity.js --sample                  # demo with sample trips
node src/challenges/vehicle-capacity.js input.json               # {"capacity": 14, "trips": [...]}
```

Sample run (capacity = 14, 5 trips):

```json
{
  "capacity": 14,
  "tripsChecked": 5,
  "violatingTrips": 2,
  "violations": [
    { "tripId": "t1", "uniqueAssigned": 15, "capacity": 14, "overflow": 1 },
    { "tripId": "t4", "uniqueAssigned": 18, "capacity": 14, "overflow": 4 }
  ]
}
```

## Verification

```text
# tests 9
# pass  9
# fail  0
```

Run: `npm test` (all Day-1 suites) or `node --test tests/day1/vehicle-capacity.test.js`.

## Traceability

- **FR-02** (capacity validation at publish) and **FR-05** (assignment capacity guard)
  in `docs/requirements/functional.md`.
- **Assumption A-05** (capacity counts unique employees) in `docs/requirements/assumptions.md`.
- Feeds the Day-4 assignment API dedup rule and Day-5 dispatch validation service.
