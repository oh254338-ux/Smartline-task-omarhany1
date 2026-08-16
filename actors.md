# Smart Line — Actor Identification (Task 04)

> **Purpose:** List every actor who interacts with the system and their primary goals.
> Capabilities are phrased as **user goals** (not implementation details) so this
> document feeds directly into Day 2's Use Case Diagram.
>
> Five primary actors + one secondary system actor (the Notification Service is a
> dependency, not a person, so it is listed as a supporting actor).

---

## 1. Company Manager

The operational owner of a company's transportation on the platform. They are accountable for "who actually boarded this morning".

**Capabilities (goals):**
1. View my company's employees and their assigned branches.
2. Create and publish trips for my company's routes, assigning employees and vehicles.
3. See morning attendance — which of my employees boarded which trip today.
4. Monitor and resolve complaints raised about my company's trips.
5. View my company's wallet/billing status and usage reports.

## 2. Employee

The transported person — the service consumer. Their interaction is mostly mobile: know where to go, get on the bus, know what they're charged.

**Capabilities (goals):**
1. View my assigned trips and pickup locations for today (and upcoming days).
2. Mark myself as boarded on a trip (with reliable offline/retry behavior).
3. View my wallet balance and per-ride charges.
4. Raise a complaint about a trip, and track its status.
5. Receive notifications about trip changes, attendance confirmation, and complaint updates.

## 3. Captain

The driver. Their day is a sequence of trips with fixed windows; they are the on-ground source of truth for what actually happened.

**Capabilities (goals):**
1. View my scheduled trips for today/week with times, route stops, and pickup points.
2. View the list of employees assigned to my current trip (to check who boards).
3. Confirm/deny boarding for employees on my trip (captain-side attendance).
4. Report trip issues (delay, breakdown, incident) to the call center.
5. Receive notifications of trip changes or cancellations affecting my schedule.

## 4. Call Center Agent

Smart Line's operations front line — handles live inquiries and complaint intake while trips are running.

**Capabilities (goals):**
1. Look up a trip and see its live status (on time, delayed, cancelled) for a caller.
2. Log a new complaint from an employee or captain and assign a category/priority.
3. Update a complaint's status and resolution notes.
4. Escalate unresolved or high-priority complaints to an admin.
5. Contact a captain mid-trip (via the platform's contact channel) to verify incidents.

## 5. Admin

Platform-level operator with cross-company oversight and registry ownership.

**Capabilities (goals):**
1. Onboard and manage companies, branches, and their contracts.
2. Maintain the vehicle registry (capacity, availability, maintenance status) and captain registry (licenses, assignments).
3. Resolve escalated complaints and enforce SLAs across all companies.
4. Repair data issues (cancel trips, correct attendance) with a full audit trail.
5. Configure platform-wide rules (zones, capacity limits, notification templates).

## 6. Supporting actor — Notification Service

Not a person, but an external system dependency that delivers messages on the platform's behalf.

**Capabilities:**
1. Deliver push/SMS/in-app messages to employees, captains, and company managers.
2. Confirm delivery success/failure back to the platform (for retry and SLA tracking).

---

## Role-combination edge case

> *A user who is both an Employee and, separately, has a Complaint escalated to them as a manager.*

Handled by **role-based access**, not by user identity: the same person holds two
roles (Employee + Company Manager). Each role grants its own goal set:
- As **Employee**: they see their own trips/wallet and raise complaints.
- As **Company Manager**: they see company-wide data and receive escalated complaints.

Design rule: every API/use case is authorized against the **role required by that use
case**, and a user's role list is explicit in the session. A complaint they raised as an
employee must never self-assign to them as the resolving manager — escalation targets a
role, and a self-conflict check prevents the same person resolving their own complaint
(flagged for Day 2 use-case validation).

## Actor → entity mapping (traceability)

| Actor | Primary entities they touch |
|---|---|
| Company Manager | Company, Branch, Employee, Route, Trip, Attendance, Complaint, Wallet |
| Employee | Trip, Attendance, Wallet, Complaint, Pickup Location |
| Captain | Trip, Attendance, Vehicle, Complaint (report) |
| Call Center Agent | Trip, Complaint, Notification |
| Admin | Company, Branch, Vehicle, Captain, Complaint, Trip, Attendance |
