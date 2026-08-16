# Trip Time-Overlap Detection for a Captain

> **Spec Task 10 · Day-1 checklist #11** · Smart Line — Block C (Algorithmic Problem Solving)
> Solution: `src/challenges/trip-overlap.js` · Tests: `tests/day1/trip-overlap.test.js`
> Reused by: Trip Conflict Detector (`src/challenges/trip-conflict-detector/`)

---

## Problem statement

Operations reports captains appearing to be assigned to two trips whose times overlap.
Given one captain's trips with start/end times, determine whether any two trips overlap
and return the overlapping pairs.

**Input:** `trips: [{ tripId, start, end }]` — up to 5,000 trips per captain per query.

**Output:** list of overlapping trip-ID pairs, e.g. `[['A', 'B'], ...]`.

**Example:** Trip A 07:00–08:00, Trip B 07:30–08:15 → overlap.

## Boundary decision (the touch case) — half-open intervals

Intervals are treated as **half-open `[start, end)`**:

```
overlap ⟺ max(startA, startB) < min(endA, endB)
```

A trip ending exactly when another starts (**A ends 08:00, B starts 08:00**) is
**NOT** a conflict.

**Defense:**
1. **Semantics** — a conflict is a period where one captain must be in two places at
   once; touching intervals share *zero duration*, so no such period exists.
2. **Convention** — half-open intervals are the standard scheduling convention: an
   instant is never double-counted, identical intervals overlap, adjacent ones don't.
3. **Consistency** — the same rule drives Task 10, the weekly Trip Conflict Detector,
   and FR-03: one rule, one implementation.
4. **Escape hatch** — if operations needs turnaround time, a configurable
   `minGapMinutes` parameter (conflict #4) layers on without changing this rule.

Zero-duration trips (`start === end`) are the empty interval: they overlap nothing
and are treated as suspicious data (skipped by the sweep).

## Solution

```js
export function findOverlappingPairs(trips) {
  const sorted = [...trips].sort((a, b) => {
    if (a.start !== b.start) return a.start - b.start;
    if (a.end !== b.end) return a.end - b.end;
    return String(a.tripId).localeCompare(String(b.tripId));
  });

  const result = [];
  const active = []; // processed trips with end > current.start, in start order

  for (const trip of sorted) {
    // Zero-duration trips are the EMPTY interval: they overlap nothing.
    if (trip.end <= trip.start) continue;

    // Expire trips that end at or before the current trip starts.
    for (let i = active.length - 1; i >= 0; i--) {
      if (active[i].end <= trip.start) active.splice(i, 1);
    }

    // Remaining active trips satisfy startActive <= trip.start < endActive;
    // they overlap `trip` iff startActive < trip.end (half-open rule).
    for (const a of active) {
      if (a.start < trip.end) result.push([a.tripId, trip.tripId]);
    }

    active.push(trip);
  }

  return result;
}
```

## Complexity

- **Time:** O(n log n + k) — the sort dominates; `k` = number of overlapping pairs
  reported. Each trip enters and leaves the active set once, and each reported pair
  costs O(1). The algorithm **never** does pairwise comparisons of non-overlapping
  trips (the naive O(n²) is avoided).
- **Worst-case note:** `k` itself can be O(n²) when *all* trips mutually overlap —
  that is the size of the answer, unavoidable regardless of algorithm.
- **Space:** O(n) for the sorted copy + active set.

## Edge cases

| Case | Behavior | Test |
|---|---|---|
| Touch (A ends 08:00, B starts 08:00) | **Not** a conflict (half-open) | `trip-overlap.test.js` |
| Partial overlap | Reported | ✓ |
| Nested intervals | Reported (outer–inner) | ✓ |
| Identical intervals | Reported | ✓ |
| Zero-duration trip | Overlaps nothing (empty interval) | ✓ |
| Chain (A–B and B–C, not A–C) | Only the real pairs | ✓ |
| Empty / single-trip list | `[]` | ✓ |
| Shuffled input | Same output (deterministic sort) | ✓ |

## CLI usage

```bash
node src/challenges/trip-overlap.js --sample                  # demo
node src/challenges/trip-overlap.js input.json               # {"trips": [{"tripId","start","end"}]}
```

Times accept `"HH:MM"` or numeric (minutes). Sample run:

```json
{
  "boundaryRule": "half-open intervals [start,end): touch is NOT a conflict",
  "tripsChecked": 4,
  "overlappingPairs": 2,
  "pairs": [
    { "tripA": "A", "tripB": "B", "rangeA": "07:00-08:00", "rangeB": "07:30-08:15" },
    { "tripA": "B", "tripB": "C", "rangeA": "07:30-08:15", "rangeB": "08:00-09:00" }
  ]
}
```

> Note: C touches A (08:00/08:00 → not flagged) but overlaps B (08:00–08:15 shared →
> flagged). Exactly the half-open semantics.

## Verification

```text
# tests 11
# pass  11
# fail   0
```

Run: `npm test` (all Day-1 suites) or `node --test tests/day1/trip-overlap.test.js`.

## Traceability

- **FR-03** (captain conflict check at publish) in `docs/requirements/functional.md`.
- **Assumption A-03** (half-open, no buffer) in `docs/requirements/assumptions.md`.
- **Conflict #4** (buffer as configurable parameter) in `docs/requirements/conflicts.md`.
- Reused unchanged by the **Trip Conflict Detector** practical challenge.
- End-of-day interview question 3 uses this exact defense.
