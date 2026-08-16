# Smart Line — Requirements Baseline (Task 14)

> **Single source of truth going into Day 2 (SRS / use cases).** Consolidates
> Tasks 1–8. Day 2 tasks must be performable using only this document.
>
> Status: v1 baseline, Day 1. All assumptions are explicit; open questions are
> listed at the end and must be answered by the Product Owner before Day 3 ERD
> freeze.

---

## Table of contents

1. [Scope in one paragraph](#1-scope-in-one-paragraph)
2. [Glossary (13 entities)](#2-glossary)
3. [Actors](#3-actors)
4. [Assumptions](#4-assumptions)
5. [Functional requirements](#5-functional-requirements)
6. [Non-functional requirements](#6-non-functional-requirements)
7. [Requirement conflicts & resolutions](#7-requirement-conflicts--resolutions)
8. [Scope boundary (in/out)](#8-scope-boundary)
9. [Open questions for the Product Owner](#9-open-questions-for-the-product-owner)

---

## 1. Scope in one paragraph

Smart Line operates company-contract employee transportation: companies with
branches register employees, routes, vehicles, and captains; trips are scheduled
instances of routes with capacity and conflict validation; attendance is captured
offline-first and idempotently; employees hold prepaid wallets charged for
boarded rides; complaints flow from employees through the call center to admins;
notifications keep every actor informed. v1 excludes fuel tracking, driver
payroll, fleet-wide route-optimization AI, B2C ride-hailing, and multi-currency
billing (see §8).

## 2. Glossary

Full definitions with relationships: see [`glossary.md`](glossary.md).
The 13 required entities — Company, Branch, Employee, Pickup Location, Captain,
Vehicle, Route, Trip, Attendance, Wallet, Complaint, Call Center, Notification —
plus Admin (the brief lists 14 names while saying 13; all 14 are covered).

Core invariants:
- A company can have multiple branches; an employee belongs to a company and
  usually a branch; a route is tied to a branch.
- A trip is a concrete instance of a route (date/time + vehicle + captain +
  assigned employees).
- Attendance belongs to an employee + trip; an employee may only be marked
  boarded on a trip they were assigned to.
- A captain's trips must not share positive duration (half-open rule).

## 3. Actors

Full capabilities: see [`actors.md`](actors.md). Summary:

| Actor | Role | Key goals |
|---|---|---|
| Company Manager | Company-side operator | create trips, assign employees, see morning attendance, manage complaints/billing |
| Employee | Transported person | view trips/pickups, mark boarded, wallet, raise complaints, notifications |
| Captain | Driver | view schedule, confirm boarding, report issues |
| Call Center Agent | Smart Line ops | trip lookup, complaint intake/triage/escalation |
| Admin | Platform operator | onboard companies, registries, resolve escalated complaints |
| *(supporting)* Notification Service | delivery system | deliver messages, report delivery status |

Role-combination rule: a person may hold Employee + Company Manager roles;
authorization is per-use-case role, and self-resolution of one's own complaint is
blocked.

## 4. Assumptions

Full risks & traceability: see [`assumptions.md`](assumptions.md). Summary:

| # | Assumption | If wrong, the impact is… |
|---|---|---|
| A-01 | One active pickup location per employee | N:M pickup model; per-trip pickup selection |
| A-02 | One branch per employee | Cross-branch cost allocation; reporting changes |
| A-03 | Half-open intervals, no buffer (touch ≠ conflict) | Configurable min-gap parameter, not algorithm change |
| A-04 | One vehicle per trip, no mid-trip swap in v1 | Temporal vehicle assignment; extra state machine |
| A-05 | Capacity counts unique employees | Over/under-booking risk |
| A-06 | Prepaid wallet; charge only boarded rides; no negative balance | Postpaid invoicing redesign |
| A-07 | Times stored as UTC instants; input is branch-local wall-clock | Only conversion layer changes |
| A-08 | Attendance dedup by eventId; ties broken by eventId | Server-side clock reconciliation needed |

## 5. Functional requirements

Full text with edge cases: see [`functional.md`](functional.md). The 12 FRs cover:

- **Trip creation (FR-01…04):** create from route template; capacity validation
  at publish; captain conflict check at publish; trip state machine.
- **Assignment (FR-05…08):** assign with capacity guard; idempotent uniqueness
  per trip; double-assignment warning; unassign (blocked after boarding).
- **Attendance (FR-09…12):** boarded only for assigned (walk-on approval path);
  captain confirmation; duplicate-event suppression; per-company daily view.

## 6. Non-functional requirements

Full numbers: see [`non-functional.md`](non-functional.md). Summary:

| Cat | NFR | Target |
|---|---|---|
| Performance | API latency | 95% < 300 ms, 99% < 1 s @ reference load |
| Performance | Live position | E2E < 10 s (99th pct) |
| Scalability | Morning peak | 5,000 RPS sustained 2 h |
| Scalability | Data volume | 50M trips / 500M attendance events within latency |
| Scalability | Conflict batch | 500 captains × 50 trips < 1 s |
| Availability | Uptime | core 99.9%, wallet 99.95% |
| Availability | Offline capture | no loss, no duplicates on sync |
| Security | AuthZ | RBAC on every request, lockout after 5 fails/15 min |
| Security | Data protection | AES-256 at rest, TLS 1.2+, append-only wallet audit |
| Security | Retention | 2-year retention, then anonymize/purge |
| Reliability | Wallet consistency | atomic + idempotent, never overdraft |
| Reliability | Event idempotency | replay-safe ingestion |

## 7. Requirement conflicts & resolutions

Full analysis: see [`conflicts.md`](conflicts.md). Resolutions:

1. **Real-time vs offline attendance** → offline-first + freshness indicator.
2. **Wallet consistency vs latency** → single-writer shard + idempotency keys.
3. **Scale vs privacy/retention** → tiered storage + retention pipeline.
4. **No-buffer scheduling vs punctuality** → configurable `minGapMinutes`.

## 8. Scope boundary

Full justifications: see [`scope.md`](scope.md).

**In scope (v1):** company/branch/employee/route/trip management; scheduling with
capacity + conflict validation; employee assignment (incl. walk-on approval);
offline-first idempotent attendance + daily view; prepaid wallet; complaints
(triage/escalation); notifications; vehicle availability gate (data integrity).

**Out of scope (acknowledged future work):** fuel tracking; driver payroll;
fleet-wide route optimization AI; in-app social chat; B2C ride-hailing;
multi-currency billing.

## 9. Open questions for the Product Owner

These are the questions from `clarification-questions.md` whose answers the
assumptions (A-01…A-08) provisionally fill. They must be confirmed before the
Day 3 ERD freeze:

| Q | Open question | Provisional answer (assumption) |
|---|---|---|
| Q-01 | One or many pickup locations per employee? | One active (A-01) |
| Q-02 | Employee in multiple branches? | No (A-02) |
| Q-03 | Who assigns employees to trips? | Company Manager (FR-05) |
| Q-04 | Is an unassigned employee an error? | No — valid state, manager visibility only |
| Q-05 | Cross-branch trips allowed? | No in v1 (branch-scoped routes) |
| Q-06 | Buffer between a captain's trips? | 0 min default (A-03, conflict #4) |
| Q-07 | Trip = one run or whole day? | One run (glossary) |
| Q-08 | Who may cancel/reschedule? | Manager/Admin; notify affected (FR-04) |
| Q-09 | Vehicle reassignment mid-trip? | No in v1 (A-04) |
| Q-10 | Duplicate IDs count as seats? | No — unique employees (A-05) |
| Q-11 | Who sets vehicle availability? | Admin registry; captain reports (actors) |
| Q-12 | Prepaid or postpaid wallet? | Prepaid (A-06) |
| Q-13 | Charge for no-shows? | No — boarded only (A-06, FR-10) |
| Q-14 | Negative balance allowed? | No (A-06) |
| Q-15 | Complaint resolution SLA? | Open — needs PO decision (drives NFR) |
| Q-16 | Complaints about non-assigned trips? | Open — see conflict #1 appendix |
| Q-17 | Managers raise complaints for employees? | Open — actor model impact |
| Q-18 | Anonymous complaints? | Open — affects retention/audit |

**Also open:** the "13 vs 14 entities" count discrepancy in the brief (resolved
pragmatically by covering all 14).
