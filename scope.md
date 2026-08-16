# Smart Line — Scope Boundary Definition (Task 08)

> **Purpose:** Draw a clear, defensible line between what this assessment's system
> must do (v1) and what is explicitly deferred. Every exclusion is acknowledged as
> future work — not ignored — with a justification.

---

## In scope (v1) — 7 items

| # | In-scope item | Justification |
|---|---|---|
| S-01 | **Company/Branch/Employee/Route/Trip management** (CRUD + state machine) | The core business objects; nothing else is buildable without them (Tasks 01, 05). |
| S-02 | **Trip scheduling with capacity validation & captain conflict detection** | A company manager's #1 pain ("over-assignment happens"); FR-02/FR-03, Tasks 09–10, Challenge. |
| S-03 | **Employee assignment to trips (incl. walk-on approval)** | The assignment flow that attendance depends on (FR-05…FR-08). |
| S-04 | **Attendance capture (offline-first, idempotent) + per-company daily view** | The "did they actually board?" promise; FR-09…FR-12, Task 13. |
| S-05 | **Wallet: prepaid employee wallet, atomic debit of boarded rides, balance view** | Money correctness is core; NFR-11, A-06. |
| S-06 | **Complaints: intake, triage, escalation (Call Center → Admin), status tracking** | The customer feedback loop the platform markets (Task 02 group E). |
| S-07 | **Notifications: trip changes, attendance confirmation, complaint updates** | Without delivery, none of the above reaches users (glossary #13). |
| S-08 | **Vehicle availability status affecting trip assignment (maintenance flag)** | *Scope edge case:* looks like a "nice-to-have" but is required for data integrity — assigning a vehicle under maintenance would produce invalid trips and poison attendance/wallet data. In-scope as a minimal `available / unavailable(reason, until)` status gate. |

## Out of scope (v1) — 6 items (acknowledged as future work)

| # | Out-of-scope item | Justification | Future work path |
|---|---|---|---|
| X-01 | **Fuel tracking & consumption analytics** | No decision in v1 depends on fuel data; it would add sensors/input burden with no consuming requirement. | Day 6+ "Ops Analytics" epic; new entity `FuelLog`. |
| X-02 | **Driver payroll / driver settlement** | Captains are company employees with external payroll; platform has no payroll authority or contract data. | Day 6+ integration with HR/payroll providers. |
| X-03 | **Route optimization AI (auto-optimizing routes fleet-wide)** | Task 12 covers a *per-route ordering heuristic*; fleet-wide multi-route optimization is a research-grade feature with no v1 stakeholder requirement. | Day 7+ ML spike; reuses `route-ordering.js` as the baseline solver. |
| X-04 | **In-app chat / social features between employees** | Not requested by any actor; adds moderation and privacy surface. | Day 6+ optional; separate product decision. |
| X-05 | **B2C ride-hailing (individual riders booking single rides)** | Smart Line is company-contract transportation; consumer booking changes the entire commercial model. | Separate product line; explicitly out of platform v1. |
| X-06 | **Multi-language/multi-currency billing engine** | Company billing is invoicing-level (Day 4+); per-currency ledger is not required by any stated rule. | Day 6+ with finance team input. |

## Boundary rules

1. **Data integrity wins over feature scope:** any "nice-to-have" that, if absent, allows invalid data (e.g., maintenance status, X-01's counterpart) is pulled into scope — see S-08.
2. **No entity outside the 14 in `glossary.md`** enters v1 without a new stakeholder requirement (fuel, payroll, and consumer booking would each add entities — deferred).
3. **Heuristics over optimum:** in-scope algorithms are bounded, deterministic heuristics (Task 12 NN, Task 11 union-find) — the "optimal" versions are explicitly out of scope (X-03).
4. **Defensibility:** every exclusion above is justified by *no consuming requirement in v1*, which is the standard asked for in the end-of-day interview.
