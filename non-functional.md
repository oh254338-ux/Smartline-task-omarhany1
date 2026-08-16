# Smart Line — Non-Functional Requirements Draft (Task 06)

> **Purpose:** Measurable NFRs for a platform that must grow to **1,000+ companies,
> millions of trip records, and real-time driver tracking** while running in
> low-connectivity industrial zones. Every target is concrete and testable.
>
> **Scale baseline used for these numbers:**
> - 1,000 companies × 20 branches = 20,000 branches
> - 20,000 branches × 300 employees = 6M employees
> - 1M trips/month average → 12M+ trip records/year, plus attendance events
> - Morning peak: the 06:00–09:00 window concentrates most trip activity

---

## 1. Performance

**NFR-01 — API latency.**
The system shall serve 95% of interactive API requests (trip views, assignment, attendance queries) in **under 300 ms** at the reference load, with a 99th percentile cap of **1 s**.

**NFR-02 — Real-time trip position.**
The system shall make a captain's live position visible to company managers and the call center with **end-to-end latency under 10 seconds** from device to dashboard (99th percentile) during normal connectivity.

## 2. Scalability

**NFR-03 — Throughput at morning peak.**
The system shall sustain **5,000 requests/second** sustained for 2 hours during the morning peak without breaching NFR-01 (measured at the API gateway).

**NFR-04 — Data volume.**
The system shall support **50M trip records** and **500M attendance events** with query performance within NFR-01 (requires partitioning/archival strategy for historical data).

**NFR-05 — Captain/trip scale for conflict checks.**
The conflict-detection and capacity-validation checks shall complete in **under 1 second** for a full-week batch of **500 captains × 50 trips** (25,000 trips) — the Task 13/Challenge scale — and scale linearly per captain thereafter.

## 3. Availability

**NFR-06 — Service availability.**
The core services (trip management, attendance ingestion, wallet) shall achieve **99.9% monthly availability** (≤ ~43 min/month downtime), with wallet operations at **99.95%**.

**NFR-07 — Offline tolerance.**
The mobile app shall allow captains and employees to **capture boarding events while fully offline** (industrial zones), queue them locally, and sync automatically when connectivity returns **without data loss and without duplicates** (idempotent replay — Task 13 semantics).

## 4. Security

**NFR-08 — Authentication & authorization.**
The system shall enforce role-based access (Employee / Captain / Company Manager / Call Center Agent / Admin) on **every** API request, with failed attempts logged and rate-limited (e.g., lockout after 5 failed logins in 15 minutes).

**NFR-09 — Data protection at rest and in transit.**
The system shall encrypt all personal data (employee PII, wallet balances, complaint content) **at rest** (AES-256) and **in transit** (TLS 1.2+); wallet transaction records shall be immutable and append-only with a full audit trail.

**NFR-10 — Data retention & privacy.**
The system shall retain attendance and complaint records for a **defined period (baseline: 2 years)**, after which they shall be anonymized or purged per a documented schedule, with complaint data subject to reporter anonymity rules (Q-18).

## 5. Reliability (data integrity)

**NFR-11 — Wallet consistency.**
Wallet debit/credit operations shall be **atomic and idempotent**: a retried charge shall never double-deduct, and a charge that would overdraw the balance shall fail without partial effect (ties to assumption A-06).

**NFR-12 — Event idempotency.**
The attendance ingestion pipeline shall process **duplicate events idempotently** (by event ID) so that replays and retries never change the derived state — verified by replaying the same event set twice and asserting identical output (Task 13 acceptance).

---

## Category coverage & conflict awareness

| Category | NFRs | Conflict pairing |
|---|---|---|
| Performance | NFR-01, NFR-02 | NFR-02 vs NFR-07 (real-time vs offline) → resolved in `conflicts.md` |
| Scalability | NFR-03, NFR-04, NFR-05 | NFR-04 vs NFR-09 (scale vs retention) → archival design |
| Availability | NFR-06, NFR-07 | NFR-01 vs NFR-07 (latency vs offline queues) → resolved in `conflicts.md` |
| Security | NFR-08, NFR-09, NFR-10 | NFR-09 vs NFR-04 (encryption cost vs throughput) → hardware-accelerated TLS |
| Reliability | NFR-11, NFR-12 | NFR-11 vs NFR-01 (strict consistency vs latency) → resolved in `conflicts.md` |

> The consistency/latency tension for wallet transactions (NFR-11 vs NFR-01) is the
> trade-off examined in `conflicts.md` and in the end-of-day interview (question 6).
