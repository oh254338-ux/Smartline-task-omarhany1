# Nearest Pickup Point Grouping

> **Spec Task 11 · Day-1 checklist #12** · Smart Line — Block C (Algorithmic Problem Solving)
> Solution: `src/challenges/pickup-clustering.js` · Tests: `tests/day1/pickup-clustering.test.js`

---

## Problem statement

Dispatch wants to auto-suggest which employees could share a route based on proximity
of their pickup points. Given pickup points with coordinates and a maximum walking
distance, group employees into clusters where **every pair in a cluster is within
max distance of at least one other member** — chained proximity (transitivity through
the "within maxDistance" graph).

**Input:** `points: [{ employeeId, lat, lng }]` (up to 2,000 points), `maxDistanceMeters: number`.

**Output:** list of clusters (arrays of employee IDs).

**Example:** 3 points close together + 1 far away → 2 clusters.

## Solution

```js
import { haversineMeters } from './pickup-clustering.js'; // exported, R = 6371000 m

class UnionFind {
  // path compression + union by size → O(n · α(n)) ≈ O(n)
}

export function clusterPickupPoints(points, maxDistanceMeters) {
  const uf = new UnionFind(points.length);

  // O(n²) pairwise distance computation — acceptable at n ≤ 2,000.
  for (let i = 0; i < points.length; i++) {
    for (let j = i + 1; j < points.length; j++) {
      const d = haversineMeters(points[i].lat, points[i].lng, points[j].lat, points[j].lng);
      if (d <= maxDistanceMeters) uf.union(i, j);   // ≤ : exactly-at-max-distance unions
    }
  }

  // Group indices by root → clusters; ids sorted, clusters ordered by min id.
  // (full implementation in src/challenges/pickup-clustering.js)
}
```

**Algorithm:** union-find over the points. If the haversine distance between two
points is `<= maxDistanceMeters`, union them. Connected components are the clusters;
an isolated point forms a singleton cluster (the outlier case).

**Haversine:** `d = 2R · atan2(√a, √(1−a))` with `a = sin²(Δφ/2) + cos φ₁ cos φ₂ sin²(Δλ/2)`, `R = 6,371,000 m`.

## Complexity

- **Distance computation:** O(n²) haversine evaluations — at n = 2,000 that is ~2M
  operations, well under a second.
- **Union-find:** O(n · α(n)) ≈ O(n) with path compression + union by size.
- **Total:** O(n²) time, O(n) space.

## Scaling beyond 2,000 points (to 100,000)

O(n²) at n = 100,000 is ~5×10⁹ evaluations — too slow. Options, in preference order:

1. **Spatial grid / uniform hashing (recommended):** bucket points into cells of side
   `maxDistance`; a point only compares against its own cell + the 8 neighbouring
   cells. Expected O(n) comparisons (worst case O(n²) only when all points collapse
   into one cell — which is the "everyone is one cluster" pathological case where the
   answer is trivial). Union-find on top stays O(n · α(n)).
2. **k-d tree / quadtree range queries:** O(n log n) worst case; more code, floating-point care.
3. **HNSW / approximate:** overkill — we want exact clustering.

Recommendation: grid hashing — simple, exact, cache-friendly.

## Edge cases

| Case | Behavior | Test |
|---|---|---|
| All points identical | Distance 0 → one cluster (even with maxDistance 0) | `pickup-clustering.test.js` |
| Single outlier | Singleton cluster | ✓ |
| Exactly at max distance | Unions (rule is `<=`) — verified both sides of the boundary | ✓ |
| Chained proximity | e1–e2 and e2–e3 edges connect e1 to e3 even if e1–e3 > max | ✓ |
| Empty points | `[]` | ✓ |
| 2,000 points | Completes quickly (verified < 3 s) | ✓ |

## CLI usage

```bash
node src/challenges/pickup-clustering.js --sample                  # demo
node src/challenges/pickup-clustering.js input.json               # {"maxDistanceMeters": 200, "points": [...]}
```

Sample run (max distance 200 m, 4 points):

```json
{
  "maxDistanceMeters": 200,
  "pointsChecked": 4,
  "clusterCount": 2,
  "clusters": [
    ["e1", "e2", "e3"],
    ["e4"]
  ]
}
```

## Verification

```text
# tests 10
# pass  10
# fail   0
```

Run: `npm test` (all Day-1 suites) or `node --test tests/day1/pickup-clustering.test.js`.

## Traceability

- Feeds the "suggest which employees share a route" dispatch flow (Task 12 route
  ordering consumes its clusters).
- **Assumption A-01** (pickup location per employee) in `docs/requirements/assumptions.md`.
- End-of-day interview question 5 uses the 100k-point scaling discussion above.
