# Requirement Decomposition Notes — Digital Reference Draft (Section 4 / Task 3)

> ⚠️ **This is a REFERENCE DRAFT, not the submission.** The task requires a
> physical hand-written sheet, photographed as `day1-planning-sheet.jpg`.
> Copy the content below onto paper (abbreviate freely — short bullet lines),
> write your name + "Task 3 / Planning" + today's date on the sheet, photograph
> it, and commit. Content is consistent with `docs/requirements/assumptions.md`
> and `docs/requirements/baseline.md`.

---

## 1. Problem understanding (write 3–4 lines)

- Smart Line = company-contract employee transportation (NOT consumer ride-hailing).
- Trips = concrete instances of routes: vehicle + captain + assigned employees + time window.
- Core promise: "which of my employees actually boarded this morning?" → trip scheduling,
  capacity & conflict validation, offline-first attendance, wallet, complaints.
- Platform ops layer: call center + notifications + admin registry.

## 2. Assumptions (8, abbreviated — full text in assumptions.md A-01…A-08)

| # | One-line assumption | Risk if wrong |
|---|---|---|
| A-01 | One active pickup location per employee | N:M pickup model; per-trip pickup selection |
| A-02 | One branch per employee | Cross-branch cost allocation + reporting break |
| A-03 | Half-open intervals; touch (08:00/08:00) ≠ conflict | Under-reported conflicts → add minGapMinutes param |
| A-04 | One vehicle per trip, no mid-trip swap in v1 | Temporal vehicle assignment + state machine |
| A-05 | Capacity counts UNIQUE employees | Overbooking if duplicates are real bookings |
| A-06 | Prepaid wallet; charge boarded rides only; no negative balance | Postpaid invoicing redesign |
| A-07 | Store UTC instants; input = branch-local wall-clock | Only conversion layer changes |
| A-08 | Attendance dedup by eventId; ties broken by eventId | Server-side clock reconciliation needed |

## 3. Open questions for the Product Owner (top 4)

1. **Q-01** Can an employee have more than one pickup location? → changes 1:1 vs N:M + clustering.
2. **Q-02** Can an employee belong to two branches? → breaks branch-scoped reporting/billing.
3. **Q-06** Is a buffer needed between a captain's trips? → sets conflict-detection strictness.
4. **Q-15** Complaint resolution SLA & who owns the deadline? → drives escalation + NFR.
   *(also open: Q-16…Q-18 — complaints about unassigned trips, manager-raised, anonymity)*

## 4. First-pass entity list (14)

Company · Branch · Employee · Pickup Location · Captain · Vehicle · Route · Trip ·
Attendance · Wallet · Complaint · Call Center · Notification · Admin
*(brief lists 13 names but enumerates 14 — flagged, all covered)*

## 5. Sketch of decomposition (optional quick diagram)

```
Company ──► Branch ──► Route ──► Trip ──► Attendance
              │          │         │
              ├─► Employee ◄───────┘        Wallet ◄── (charged on boarded)
              └─► Pickup Location ─► stops on Route
Captain ──► Trip   (no time overlap)
Vehicle ──► Trip   (capacity gate)
Complaint ◄─ Employee ─► Call Center ─► Admin
Notification ──► everyone (events)
```

## Handwriting tips

- Use short bullet lines; abbreviations OK (Emp, Cap, Veh, Att, WC).
- Draw the A-xx table as a 3-column table; keep one line per assumption.
- Keep the sketch small (5–6 boxes) — legibility beats completeness.
- Put name + "Task 3 / Planning" + date at the top corner.
