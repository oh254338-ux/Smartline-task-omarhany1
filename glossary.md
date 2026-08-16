# Smart Line — Business Domain Glossary (Task 01)

> **Purpose:** Name and relate every core business concept of the Smart Line platform
> so that requirements, ERD (Day 3) and API design (Day 4) speak one language.
> **Note on count:** The brief asks for "13 entities" but lists **14** names. All 14 are
> covered below; the count discrepancy is logged as an open question in the baseline.

---

## 1. Company

**Definition:** A client organization (e.g., a factory, hospital, or office complex) that contracts Smart Line to transport its workforce. The company owns the commercial relationship: billing, service levels, and the set of employees entitled to transportation.

**Relationships:**
- A Company **has one or more Branches** (business rule: a company can have multiple branches); branches inherit the company's contract.
- A Company **employs Employees** — an employee belongs to a company, and usually to one branch.
- A Company's needs (capacity, zones) **shape the Routes** its branches operate.
- A Company is **managed by Company Manager(s)** (actor) and **administered by the platform Admin**.

## 2. Branch

**Definition:** A physical location or organizational unit of a Company (e.g., a plant, a depot, a campus). Branches are the operational anchor for routing and dispatch: routes are tied to a branch, and most employees report to one.

**Relationships:**
- A Branch **belongs to one Company** (1:N).
- A Branch **owns Routes** (business rule: a route is tied to a branch) and **serves Employees** assigned to it.
- A Branch **defines Pickup Locations** within its operating area (stops on its routes).
- Edge case: an employee with no branch assigned, and a branch with no active routes yet, must both be valid states.

## 3. Employee

**Definition:** A person transported by Smart Line — the end consumer of the service. Each employee belongs to a company (and usually a branch), may hold a wallet, gets assigned to trips, and generates attendance and complaint records.

**Relationships:**
- An Employee **belongs to a Company** and **usually a Branch** (may be unassigned or multi-branch — open question).
- An Employee **is assigned to Trips** and **uses a Wallet** for trip-related balances/payments.
- An Employee **has one or more Pickup Locations** (open question) and **produces Attendance** records and **raises Complaints**.
- An Employee is **notified** of trip/attendance changes.

## 4. Pickup Location

**Definition:** A georeferenced point (lat/lng, plus a human-readable name) where employees board or alight. Pickup locations anchor route stops and enable proximity-based grouping of employees onto shared routes.

**Relationships:**
- A Pickup Location **belongs to a Branch** (or is shared across companies — open question) and **is visited by Routes** as a stop.
- Employees **are linked to one or more Pickup Locations**; proximity between pickup points is what the system uses to suggest shared routes (Task 11 clustering logic).
- A Pickup Location **is used by Trips** when a route instance passes through it.

## 5. Captain

**Definition:** The driver/operator of a vehicle on a trip. Captains are the people who physically carry the trip out; they are assigned to trips, appear in attendance/conflict checks, and are contactable through the call center.

**Relationships:**
- A Captain **drives Trips** (a captain may have many trips; two trips may not overlap in time — Task 10 / Trip Conflict Detector).
- A Captain **operates Vehicles** (one vehicle per trip at a time; vehicle reassignment mid-trip is an open question).
- A Captain **reports to the Branch** whose routes they run and **receives Notifications** (trip assignments, changes).

## 6. Vehicle

**Definition:** A physical bus/van with a fixed seat capacity. Vehicles are the capacity constraint of the whole system: dispatch must not over-assign employees beyond capacity (Task 09).

**Relationships:**
- A Vehicle **is assigned to Trips** (one trip at a time under normal scheduling) and **belongs to a Branch/Company** fleet.
- A Vehicle's **capacity limits how many Employees** can be assigned to a trip.
- Vehicle **maintenance/availability status** gates trip assignment (scope decision — treated as in-scope for data integrity).

## 7. Route

**Definition:** A reusable pattern of travel — a sequence of stops (pickup locations) with an intended direction and schedule template, tied to a branch. Routes are the "template"; trips are the "instances".

**Relationships:**
- A Route **belongs to a Branch** (business rule) and **visits Pickup Locations** in a defined order (Task 12 ordering problem).
- A Route **is instantiated as Trips** on given dates/times with a captain and vehicle.
- A Route **serves the Employees** whose pickup points lie on it.

## 8. Trip

**Definition:** A concrete, scheduled instance of a route: a vehicle + captain + set of assigned employees + a date/time window between pickup and drop-off. Trips are the unit of scheduling, conflict detection, attendance, and complaint reference.

**Relationships:**
- A Trip **instantiates a Route** and **uses a Vehicle** driven by a **Captain**.
- A Trip **carries assigned Employees** (walk-ons are the edge case) and **generates Attendance** events for each boarded employee.
- A Trip **is the subject of Complaints** and **is the scope unit** for the capacity (09), overlap (10) and attendance (13) algorithms.

## 9. Attendance

**Definition:** The record of whether an employee actually boarded a trip on a given day — the operational truth the company pays for. Captured via mobile events that may be duplicated or arrive out of order (Task 13).

**Relationships:**
- Attendance **belongs to an Employee** and **references a Trip**; an employee may only be marked boarded on a trip they were assigned to (business rule).
- Attendance **drives Company Manager visibility** ("who actually boarded this morning") and **feeds the Wallet** if trip charges are per-ride.
- Attendance events are **idempotent by eventId** — duplicates must not inflate counts.

## 10. Wallet

**Definition:** A per-employee (and possibly per-company) balance ledger that records prepaid funds or per-ride charges related to transportation usage. Wallet transactions demand strong consistency (no double-spend, no lost updates).

**Relationships:**
- A Wallet **belongs to an Employee** (1:1) and **records Transactions** for trips taken/attendance confirmed.
- Wallet charges **reference Trips/Attendance** (only boarded rides are charged — open question) and **reconcile with the Company's** billing contract.
- Wallet operations **conflict with low-latency requirements** (consistency vs latency trade-off, Task 07).

## 11. Complaint

**Definition:** A formal issue raised by an employee (or company manager) about a trip — late arrival, missed pickup, captain behavior, vehicle condition. Complaints are logged, triaged, escalated, and resolved; they are the voice-of-customer loop of the platform.

**Relationships:**
- A Complaint **is raised by an Employee** (reporter) and **references a Trip/Route** (subject).
- A Complaint **is handled by the Call Center** (triage/handling) and **may be escalated to an Admin** or company manager.
- Complaint resolution **may trigger Notifications** to the reporter.

## 12. Call Center

**Definition:** The operational function (team of agents) that handles phone/chat contact from employees and captains: trip status inquiries, complaints intake, and emergency coordination during trips.

**Relationships:**
- The Call Center **receives and triages Complaints** and **answers Employee/Captain inquiries** about Trips.
- Call Center agents **view Trip status in real time** (needs live trip data) and **escalate to Admins** when needed.
- Call Center activity **generates Notifications** (resolution updates to reporters).

## 13. Notification

**Definition:** The outbound communication channel (push/SMS/in-app) that informs employees, captains, and company managers about trip assignments, schedule changes, attendance confirmation, wallet events, and complaint status.

**Relationships:**
- Notifications **target Employees, Captains, and Company Managers** based on their role and the triggering event (Trip change, Attendance, Wallet, Complaint).
- Notifications **are triggered by system events** across entities (trip cancellation, complaint resolution, wallet top-up).
- Notification delivery **depends on connectivity** (low-connectivity industrial zones — Task 07 conflict).

## 14. Admin

**Definition:** The platform-side operator (Smart Line staff) who oversees the whole ecosystem: onboarding companies/branches, managing vehicles/captains registry, resolving escalated complaints, and configuring system-wide rules (capacities, zones, billing).

**Relationships:**
- An Admin **onboards and manages Companies/Branches** and **maintains the Vehicle and Captain registries**.
- An Admin **receives escalated Complaints** and **can override/repair Trip and Attendance data** (audit-trailed).
- Admin actions **affect all entities** — the admin is the system-level actor outside any single company.

---

## Entity relationship summary (first pass)

| Entity | Parent/owner | Children / dependents | Key constraint |
|---|---|---|---|
| Company | — | Branch, Employee | 1:N branches |
| Branch | Company | Route, Pickup Location, Employee | route tied to branch |
| Employee | Company / Branch | Attendance, Complaint, Wallet | belongs to company |
| Pickup Location | Branch | (linked to) Employee, Route stop | geo point |
| Captain | Branch/Company | Trip | no overlapping trips |
| Vehicle | Branch/Company | Trip | fixed capacity |
| Route | Branch | Trip | template |
| Trip | Route | Attendance, Complaint | capacity + no overlap |
| Attendance | Trip + Employee | — | assigned-only boarding |
| Wallet | Employee | Transactions | consistency-critical |
| Complaint | Employee → Trip | Call Center, Admin | escalation chain |
| Call Center | Smart Line | Complaint handling | — |
| Notification | any event | Employee/Captain/Manager | delivery reliability |
| Admin | Smart Line | all entities | global oversight |

> This glossary is the vocabulary used by every other Day-1 document and the source
> of truth for Day 2 (use cases) and Day 3 (ERD).
