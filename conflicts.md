# Smart Line — Requirement Conflict Analysis (Task 07)

> **Purpose:** Identify requirements written so far that genuinely pull against each
> other, choose a resolution for each, and state the trade-off explicitly. All
> resolutions are technically feasible, not wishful.

---

## Conflict 1 — Real-time attendance updates vs. reliable capture in low-connectivity zones

**The conflict:** NFR-02 demands real-time position/attendance visibility (<10 s E2E), but NFR-07 demands the app work fully offline in industrial zones. A device that is offline *cannot* stream in real time; a design that forces real-time streaming breaks in low connectivity; a design that forces offline-first cannot guarantee real-time dashboards.

**Resolution:** **Offline-first capture with event sourcing and eventual sync.**
- All attendance events are written locally, stamped with a client-generated event ID, and queued; sync happens opportunistically (on connectivity, throttled).
- Dashboards show the **latest synced state** and mark it with a *"last seen / stale since"* indicator so managers know the freshness of what they see.
- The system does not attempt real-time guarantees for offline devices; it guarantees **no loss and no duplication** (Task 13 semantics).

**Trade-off stated:** Managers trade *guaranteed freshness* for *guaranteed correctness*: during connectivity gaps, the dashboard may lag (minutes, not seconds), but the final attendance truth is always complete and never double-counted. Real-time live tracking remains available only for online devices.

## Conflict 2 — Strict wallet consistency vs. low API latency

**The conflict:** NFR-11 demands atomic, idempotent, never-overdraft wallet operations; NFR-01 demands 95% of requests under 300 ms. Strict distributed consistency (synchronous quorum writes, locks) costs latency; permissive reads risk showing stale balances.

**Resolution:** **Single-writer ledger partition + client idempotency keys + read-your-writes.**
- Wallet transactions are routed to a shard keyed by employee ID (single-writer per wallet), executed against a local ledger with a unique idempotency key per charge — atomic, serialized, and cheap.
- Balance reads serve from the same shard with read-your-writes consistency (the employee always sees their own latest balance; cross-shard aggregate reports can be eventually consistent).

**Trade-off stated:** Cross-wallet operations (e.g., company-wide billing rollups) accept eventual consistency and slight lag, while per-wallet operations keep strict atomicity at latency well under the 300 ms budget. The cost is a hot-partition risk for very active wallets and a more complex shard-routing layer.

## Conflict 3 — Scale/data growth vs. privacy retention & encryption cost

**The conflict:** NFR-04 expects 500M+ attendance events with queries within 300 ms; NFR-09/NFR-10 require full encryption at rest and bounded retention (2 years, then purge). Encrypting and retaining everything forever would blow both the storage budget and query latency.

**Resolution:** **Tiered storage with time-based partitioning and a documented retention pipeline.**
- Hot data (current month) lives in the primary store, indexed and fully encrypted.
- Cold data (older) moves to partitioned/archived storage with a defined query path, and is purged/anonymized exactly at the 2-year boundary per NFR-10.
- Retention is a scheduled, audited job — not an afterthought.

**Trade-off stated:** Historical queries become slower or restricted to the archive path (cold reads may take seconds, outside NFR-01), and a purge job introduces a small operational risk of over-deletion — mitigated by a 30-day "soft-delete" quarantine window before permanent erasure.

## Conflict 4 — Captain no-buffer scheduling (A-03) vs. operational punctuality expectations

**The conflict:** Assumption A-03 treats back-to-back trips (08:00/08:00) as non-conflicting, but operations staff report late arrivals — a zero-buffer schedule is mathematically valid but operationally fragile.

**Resolution:** Keep the **algorithmic rule** (half-open intervals, no overlap) as the correctness baseline, and expose a **configurable `minGapMinutes`** parameter (default 0 for v1) that operations can raise per branch/company without changing the algorithm.

**Trade-off stated:** With a buffer configured, more pairs are flagged as conflicts (fewer trips schedulable per captain per day) in exchange for realistic turnaround time. The default (0) maximizes scheduling flexibility but pushes punctuality risk to operations.

---

## Conflict resolution summary

| # | Requirements in tension | Resolution | Loses | Gains |
|---|---|---|---|---|
| 1 | NFR-02 vs NFR-07 | Offline-first + freshness indicator | Guaranteed real-time freshness | No-loss, no-dup attendance everywhere |
| 2 | NFR-11 vs NFR-01 | Single-writer shard + idempotency keys | Cross-wallet read freshness | Strict per-wallet atomicity at low latency |
| 3 | NFR-04 vs NFR-09/10 | Tiered storage + retention pipeline | Cold-query latency | Bounded cost, privacy compliance |
| 4 | A-03 vs operations | Configurable min-gap parameter | Schedule density when buffer > 0 | Operational reliability on demand |
