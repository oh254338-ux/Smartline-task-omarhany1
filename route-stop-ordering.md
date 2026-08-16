# Route Stop Ordering

> **Spec Task 12 · Day-1 checklist #13** · Smart Line — Block C (Algorithmic Problem Solving)
> Solution: `src/challenges/route-ordering.js` · Tests: `tests/day1/route-ordering.test.js`

---

## Problem statement

A route has multiple stops; the order matters for total trip duration and employee
wait times. Given a starting depot and a set of stop coordinates, produce a stop order
that minimizes total travel distance.

This is the **Traveling Salesman Problem** on a plane — NP-hard. A practical heuristic
is the correct engineering call here; the task explicitly does not require the optimal tour.

**Input:** `depot: {lat, lng}`, `stops: [{id, lat, lng}]` (up to 25 stops).

**Output:** ordered list of stop IDs + total estimated distance.

## Solution — Nearest-Neighbor heuristic

```js
export function orderRouteStops(depot, stops) {
  const remaining = stops.map((s) => ({ ...s }));
  const order = [];
  let total = 0;
  let current = depot;

  while (remaining.length > 0) {
    // pick the nearest unvisited stop (ties broken by id → deterministic)
    let best = 0;
    let bestDist = Infinity;
    for (let i = 0; i < remaining.length; i++) {
      const d = haversineMeters(current.lat, current.lng, remaining[i].lat, remaining[i].lng);
      if (d < bestDist || (d === bestDist && remaining[i].id < remaining[best].id)) {
        best = i;
        bestDist = d;
      }
    }
    const chosen = remaining.splice(best, 1)[0];
    order.push(chosen.id);
    total += bestDist;
    current = chosen;
  }

  return { order, totalDistanceMeters: total };
}
```

Starting at the depot, repeatedly visit the **nearest unvisited stop** until all are
visited. Total distance = depot→first + between stops. Distances via haversine.

## Why nearest-neighbor — heuristic justification

| Approach | Cost at n = 25 | Verdict |
|---|---|---|
| Exact TSP (Held–Karp DP) | O(n²·2ⁿ) ≈ 25² × 33.5M ≈ **2×10¹⁰ ops** | Far beyond the 1-second budget |
| Brute-force permutations | O(n!) | Hopeless |
| **Nearest-neighbor (chosen)** | **O(n²) = 625 evaluations → microseconds** | Correct engineering call |

- NN typically lands within ~10–25% of the optimal tour on Euclidean instances —
  well within operational tolerance for a bus route.
- Worst case can be arbitrarily bad in theory, but at ≤ 25 stops the practical
  quality is good; a **2-opt local-search pass** (also O(n²)) is the documented
  follow-up if tour quality ever matters more.

## Complexity

- **Time:** O(n²) — 625 distance evaluations at n = 25, far under 1 second.
- **Space:** O(n) for the remaining-stops list.
- **Determinism:** ties broken by stop id → output independent of input order.

## Edge cases

| Case | Behavior | Test |
|---|---|---|
| Two stops at identical coordinates | Distance 0 between them; deterministic id tie-break | `route-ordering.test.js` |
| Single stop | `order: [stopId]`, total = depot→stop | ✓ |
| Straight-line stops | NN produces the natural linear order | ✓ |
| Empty stops | Empty order, 0 distance | ✓ |
| Scattered stops | Nearest-first, all stops covered (permutation) | ✓ |
| 25 stops | Verified well under 1 s | ✓ |

## CLI usage

```bash
node src/challenges/route-ordering.js --sample                  # demo
node src/challenges/route-ordering.js input.json               # {"depot": {...}, "stops": [...]}
```

Sample run (depot + 4 gates):

```json
{
  "heuristic": "nearest-neighbor (greedy) — O(n²)",
  "stopCount": 4,
  "order": ["gate-west", "gate-east", "gate-north", "plant-2"],
  "totalDistanceMeters": 1436
}
```

## Verification

```text
# tests 7
# pass  7
# fail  0
```

Run: `npm test` (all Day-1 suites) or `node --test tests/day1/route-ordering.test.js`.

## Traceability

- Implements the "route stop order" concept from `glossary.md` (Route visits Pickup
  Locations in a defined order).
- **Scope decision X-03** (`docs/requirements/scope.md`): fleet-wide route-optimization
  AI is out of scope; this per-route heuristic is the in-scope baseline solver.
- End-of-day interview daily-gate question: defend why O(n²) NN beats exact TSP here.
