# Plan — specs before code

**Concept.** The plan turns a request into two lists: **what the task must
satisfy** (specs) and **which laws it will touch** (ids). Both are written
before implementation, because both are what the delivery will later have to
prove — and a spec invented after the code exists is a description of the code,
not a requirement of it.

**Rules defined here:** `DLV-4` · `DLV-5` · `DLV-17` — the law is the
*Invariants* table below; every ❌ item cites the id it violates.

## Invariants

| ID | Law | Class | Gate |
|---|---|---|---|
| DLV-4 | The specs the task must satisfy are written before implementation starts, in the user's language, each one testable. | constitutional | `manual` |
| DLV-5 | The laws the task will touch are named up front, so a `manual` binding is discovered at planning time and not at delivery time. | constitutional | `manual` |
| DLV-17 | When the task comes from a board, its approved cases **are** the spec list: every one is proved by a test and appears in the report. The plan adds to that list, and never silently replaces it. | constitutional | `gate:case-test-link` |

## What a spec looks like

A spec states an observable outcome. It is not a task, and it is not an
implementation note:

```text
❌  "add a cancel button"                       a task, proves nothing
❌  "call the cancel endpoint on click"         an implementation note
✅  "A cancelled order cannot be cancelled again"
✅  "Cancelling releases every reserved shipment"
✅  "Without the permission the action stays visible, inert, with its reason"
```

The test is blunt: **can this sentence fail?** "Add a button" cannot fail — a
button either exists or the task was not done. "A cancelled order cannot be
cancelled again" can fail, which is why it is worth writing and worth testing.

Every spec on this list ends up in the delivery report next to the test that
proves it (`DLV-10`). That is the reason to write them now: the list is the
contract for what "done" means, agreed before anyone is invested in code.

## When the task already has its cases

A task taken from a board arrives with a list a person has already read and
added to — `turystack-harness` writes it at task creation and stops until
somebody approves it. `DLV-17` says what this step does with that list: it is
the spec list, not an input to a new one.

```text
board task T-4
  TC-1  AC-1  unit       a paid order becomes cancelled and releases its shipments
  TC-2  AC-2  unit       cancelling twice changes nothing, and the refusal says why
  TC-4  —     integration  two cancellations racing leave one cancellation, one refusal

plan
  every TC above, each mapped to the slice that will prove it
  + whatever this planning found that the list did not have
```

The plan is expected to **grow** the list — planning is where the fourth
unhappy path shows up. What it may not do is drop an entry: a case a person
added and a case an agent proposed are the same kind of promise once approved,
and the one most likely to be quietly abandoned is the one that was hardest to
write a test for. That is the one that was worth the most.

If a case turns out to be wrong, it goes back to the person who approved it.
That is a conversation, not an edit.

## Covering the unhappy paths

Most missing specs are on the same four branches, and the constitution already
names them. Walk them explicitly:

| Branch | Ask | Law |
|---|---|---|
| The read has no data, partial data, or fails | what does the surface show in each? | `ARC-ERR-8` |
| The actor is not allowed | inert with a reason, or a denied surface? | `ARC-ERR-9` |
| The write reaches other entities | what does the confirmation have to show first? | `ARC-CON-11` |
| The message arrives twice, or late | does the effect stay correct? | `ARC-IDM-1`, `ARC-IDM-3` |

A task that produces four specs and skips these produces four tests and a
support ticket.

## Naming the laws up front

Read the routing table in `SKILL.md`, open the sections the task touches, and
list the ids you already know apply. This costs minutes and buys two things:

- **`manual` bindings surface now.** If the task touches `ARC-CON-11`, someone
  will have to sign that the blast radius is right. Knowing it at planning time
  means the reviewer is lined up; discovering it at delivery time means waiting.
- **The build knows what it is aiming at.** A slice written against a named law
  is a slice that already knows what its test looks like.

## Slicing

The plan ends with the task cut into slices. A slice is the **smallest change
that can be proven on its own** — it compiles, its tests pass, and the gate
ladder runs green on it.

```text
task: cancel an order from the table

slice 1   entity: the transition and its guard        unit tests
slice 2   use-case + endpoint                         integration + e2e
slice 3   table action + permission gate              component tests
slice 4   confirmation with blast radius              component + e2e
slice 5   deep link for the open row                  route test
```

Two properties make a cut good: each slice is **independently green**, and the
order **front-loads risk** — the thing most likely to be wrong (the domain
rule) is proven first, while there is still time to change the plan.

Cutting by layer for its own sake is not the goal; cutting so each piece can be
proven is.

## Never do

- Writing code first and deriving the specs from it afterwards (`DLV-4`).
- Replacing a board task's approved cases with a fresh list of your own
  (`DLV-17`).
- Quietly dropping the case that turned out to be hard to prove (`DLV-17`).
- A spec that cannot fail (`DLV-4`).
- Planning only the happy path and treating the other four outcomes as details
  (`ARC-ERR-8`).
- Discovering at delivery time that a law needed a human signature (`DLV-5`).
- A slice so large that a red gate cannot say which change caused it.
