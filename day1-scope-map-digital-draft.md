# Scope Map — Digital Reference Draft (Task 15)

> ⚠️ **This is a REFERENCE DRAFT, not the submission.** The task requires a
> hand-drawn box-and-line map, photographed as `day1-scope-map.jpg`. Use the
> layout below as your hand-drawing guide. The content matches the entities in
> `glossary.md` and the actors in `actors.md`.

```
                    ┌─────────────────────────────────────────────┐
                    │           SMART LINE PLATFORM               │
                    │              (system boundary)              │
                    │                                             │
  ┌──────────────┐  │  ┌───────────────────────────────────────┐  │
  │ COMPANY      │◄─┼──┤  TRIP MANAGEMENT                      │  │
  │ MANAGER      │  │  │  • Routes → Trips                     │  │
  └──────────────┘  │  │  • Capacity & conflict validation     │  │
                    │  │  • Assignment / attendance            │  │
  ┌──────────────┐  │  └───────────────┬───────────────────────┘  │
  │ EMPLOYEE     │◄─┼──┐              │                          │
  └──────────────┘  │  │              ▼                          │
                    │  │  ┌───────────────────────┐              │
  ┌──────────────┐  │  │  │  WALLET               │              │
  │ CAPTAIN      │◄─┼──┼──┤  • Balances            │              │
  └──────────────┘  │  │  │  • Atomic debits       │              │
                    │  │  └───────────────────────┘              │
  ┌──────────────┐  │  │                                         │
  │ CALL CENTER  │◄─┼──┼──┐  ┌───────────────────────┐           │
  │ AGENT        │  │  │  ├──│  COMPLAINTS           │           │
  └──────────────┘  │  │  │  │  • Intake → triage    │           │
                    │  │  │  │  • Escalation         │           │
  ┌──────────────┐  │  │  │  └──────────┬────────────┘           │
  │ ADMIN        │◄─┼──┼──┼─────────────┤                        │
  └──────────────┘  │  │  │             ▼                        │
                    │  │  │  ┌───────────────────────┐           │
                    │  │  └──│  NOTIFICATIONS        │           │
                    │  │     │  • Push / SMS / in-app│           │
                    │  │     └───────────────────────┘           │
                    │  │                                         │
                    │  │  ┌───────────────────────┐              │
                    │  └──│  REGISTRIES           │              │
                    │     │  • Vehicles, Captains │              │
                    │     └───────────────────────┘              │
                    │                                             │
                    └─────────────────────────────────────────────┘
```

## Hand-drawing guide (what each box/line means)

| Element | Meaning |
|---|---|
| Outer rectangle | System boundary — actors live OUTSIDE, subsystems INSIDE. |
| Actor boxes (left) | The 5 actors from `actors.md`; each is a person/role, not data. |
| Subsystem boxes (inside) | The major v1 subsystems: Trip Management (incl. attendance), Wallet, Complaints, Notifications, Registries, Admin. |
| Lines actor→subsystem | Which subsystems each actor touches (e.g., Employee → Trip Management for boarding, Wallet for balance, Complaints to raise, Notifications to receive). |
| Dotted relationship | Wallet debits are triggered by attendance events from Trip Management (the money/data coupling). |

## Layout rules to reproduce

1. Actors on the left, outside the boundary; every actor connected to ≥ 1 subsystem.
2. Trip Management is the hub (top center) — most actors touch it.
3. Wallet connects to Trip Management (attendance-driven charging).
4. Complaints connects to Call Center Agent (intake) and Admin (escalation).
5. Admin also owns Registries (vehicles/captains) and company onboarding.
6. Write your name + "Task 15 / Scope Map" + date on the sheet before photographing.
