---
name: turystack-proof-mode
description: "Delivery harness for one Turystack implementation task — open it as soon as a task exists, before the architecture, backend, frontend or primitives skills, whenever someone says implement, build, add, create, fix, ship, migrate or refactor in a Turystack product. It runs the task; `turystack-harness` runs the session, decides whether the project can be coded in at all and hands the task over with its ids resolved — so open that one first, and this one the moment it points at work. It resolves the five inputs a task depends on (product spec, domain model, design, UI/UX rules, API contract) and HALTs rather than building over a missing one; routes to the pattern skills; runs the gate ladder — format, lint, typecheck, structure, test, coverage, e2e, visual — after every slice instead of once at the end; and closes with a delivery report where every acceptance criterion is tied to the test that proves it, both coverage floors clear, and each screen sits beside the design it had to match. Use it also when a task is finished and its evidence has to be assembled, or when you need to know whether something is actually done. The pattern skills answer how code is written; this one answers when, and what proves it."
---

# turystack-proof-mode

The delivery harness. It is the only skill that runs a task from end to end;
every other skill is something it opens on the way.

It is not where a session starts. `turystack-harness` decides whether the
project has the two skills every input here resolves against, and which task
this is — then hands it over, with its ids already named. This skill takes it
from there and does not decide it again.

## What this skill is for

A pattern skill answers *how is this written*. This one answers the questions
that come before and after:

```text
before   what is this task built on, and is any of it missing?
during   which laws apply, and is the code still green after each slice?
after    what proves it is done, and where is that evidence?
```

The question *before* those three — is there a spec and a design to build
against at all, and which task is this — belongs to `turystack-harness`.

Those three are where delivery actually fails. Code written against a
half-remembered spec passes every lint. A gate run once at the end finds
fifteen problems at the worst possible moment. A task declared finished with no
evidence is a task someone else has to re-verify.

## The loop

```text
  ┌── 1. resolve context ──────────────────────────────┐
  │      five inputs, each with a location             │
  │      one missing → stop and ask, never assume      │
  └────────────────────┬───────────────────────────────┘
                       ▼
  ┌── 2. plan ─────────────────────────────────────────┐
  │      specs listed before code; skills routed;      │
  │      the laws this task will touch, named          │
  └────────────────────┬───────────────────────────────┘
                       ▼
  ┌── 3. build in slices ──────────────────────────────┐
  │   ┌─► smallest slice that can be proven ──┐        │
  │   │   gate it                             │        │
  │   └── green? next slice : fix now ────────┘        │
  └────────────────────┬───────────────────────────────┘
                       ▼
  ┌── 4. assemble evidence ────────────────────────────┐
  │      specs↔tests, coverage, captures, deltas       │
  └────────────────────┬───────────────────────────────┘
                       ▼
  ┌── 5. deliver ──────────────────────────────────────┐
  │      one report; green or it is not delivered      │
  └────────────────────────────────────────────────────┘
```

Step 3 is a loop on purpose. Gates run **after every slice**, not once at the
end — a red gate on a slice you just wrote points at ten lines; the same gate
after a day of work points at a thousand.

## How to use

The task arrives as a sentence. *"deixa o operador cancelar um pedido."* That is
the input this harness takes — nobody asks for work in acceptance criteria, and
they should not have to. Step 1 is what turns the sentence into ids.

Each step below names its **goal**, what it **does**, and the conditions under
which it **HALTs**. A HALT is not a suggestion to be careful: it is a stop, with
the thing that has to happen before the step can be re-entered.

| # | Step | Goal | HALT when |
|---|---|---|---|
| 1 | `01-context.md` | resolve the sentence to ids and the five inputs | an input does not resolve, or the ask maps to no `AC-n` |
| 2 | `02-plan.md` | slices, each naming the `AC-n` / `UX-DR-n` it satisfies | a slice satisfies no criterion, or a criterion has no slice |
| 3 | `03-build.md` | one slice, gated as it lands | a rung is red — fix it in this slice, never the next |
| 4 | `04-gates.md` | read the red rung for what it actually proves | a rung is bypassed rather than fixed (`DLV-9`) |
| 5 | `05-evidence.md` | assemble proof as each slice lands | a required capture is missing, or a `manual` binding is unsigned |
| 6 | `06-delivery.md` | one report, verdict computed | a criterion has no test, or the report names no `AC-n` |
| 7 | `06-delivery.md` | the report is attached to the board task it closes | the task is marked done with no report on it |

**Step 1 in full, because it is the one that gets skipped.** Read the sentence.
Find the capability in `<project>-spec` it belongs to, and list the `AC-n` it
will satisfy. Find the surface in `<project>-uiux`, and list the `UX-DR-n`. From
here on the task is those ids — not the sentence, which nobody will remember the
same way in a week.

```text
"deixa o operador cancelar um pedido"
    → Capability: Cancel an order   → AC-1, AC-2, AC-3, AC-4
    → Surface: Orders table          → UX-DR-1, UX-DR-2, UX-DR-3
    → five inputs resolved, recorded in the report
```

If the sentence maps to no criterion, that is not a small gap to fill by
inference — it means the thing being asked for is not in the spec. HALT, and
ask. Writing the criterion takes minutes with the person who knows; inventing
it costs a release.

## Do not stop early

Run to completion in one execution. Not "significant progress", not "a good
stopping point", not "I will continue in the next session" — those end with a
task that has to be re-read from scratch by whoever picks it up, including you.

The only reasons to stop are the HALT conditions in the table above and the
user asking you to. Neither of them is "this feels like enough for now". A
delivery is decided by step 6, not by how much has been done.

## What each step may write

A step that writes outside its scope is a step whose output nobody can review
against what it was asked to do.

| Step | May write |
|---|---|
| 1 · Context | the report's `context` block. No source files. |
| 2 · Plan | the slice list and its `AC-n` / `UX-DR-n` mapping |
| 3 · Build | source, tests and fixtures for the current slice — nothing from a later one |
| 4 · Gates | the rule in `@turystack/*-config` when it is genuinely wrong, with a fixture proving both sides. Never an ignore comment |
| 5 · Evidence | captures, spec↔test links, `manual` signatures |
| 6 · Delivery | `gate-report.json` and its page |

Rules are never edited to make a rung pass (`DLV-9`), and the spec is never
edited to match what was built — if the spec is wrong, that is a HALT and a
conversation, not a quiet correction.

The pattern skills are opened *inside* steps 2 and 3, per the routing below.
This skill never restates their content; it decides when they are needed.

Every section opens with a **Rules defined here** line naming the ids it owns,
so you can confirm you opened the right file before reading it. A section
that says `none` states no law of its own — everything in it is cited.

## Where the task came from

This skill takes a task and delivers it. What decides that a task may exist at
all — whether the project's own skills are written, and which of them this
session is meant to touch — is `turystack-harness`, and it runs before this one
on every session.

Two consequences for step 1. The five inputs resolve against skills the harness
produced, so an unresolvable input is usually a bootstrap that has not happened
rather than a section someone forgot. And the task arrives with the ids it
claims already named, from the board: they are the input to step 1, not
something to re-derive from the sentence.

## Routing to the other skills

| The task touches | Open |
|---|---|
| Deciding whether the project is ready to be coded in at all | `turystack-harness` |
| Anything non-trivial, always first | `turystack-architecture-pattern` |
| An endpoint, use-case, entity, repository, handler, event, migration | `turystack-backend-pattern` |
| A screen, route, form, table, state, permission-gated action | `turystack-frontend-pattern` |
| A shared UI unit's contract, styles, composition or a11y | `turystack-frontend-primitives-pattern` |

The order matters: the constitution decides *what applies*, the stack skill
decides *how it is written*, the library documentation decides *how its API is
called*. Reading them in reverse produces code that matches an example and
breaks a law.

## The refusal rule

**A missing input stops the task.** Not a guess, not a placeholder, not "I will
assume the usual". If the product spec does not say what happens when the order
is already cancelled, that is a question for a person — and asking it costs
minutes, while inventing it costs a release.

This is the same law the constitution applies to contracts (`ARC-CTR-5`: a
missing symbol is a blocker, not a license to work around it), applied one level
up: a missing decision is a blocker, not a license to invent one.

What "stop" means in practice is in `01-context.md`; it is rarely stopping
everything — usually part of the task is unblocked and proceeds.

## Before you hand anything over

1. **Context.** Did every one of the five inputs resolve to a real location, and
   is each one recorded in the report? (`DLV-1`)
2. **Slices.** Was every slice gated as it landed, or did gates run once at the
   end? (`DLV-6`)
3. **Ladder.** Is every rung green — including `visual`, which fails on a
   missing required capture rather than rendering a placeholder? (`DLV-8`, `DLV-12`)
4. **Coverage.** Do both floors clear: the codebase floor and the higher floor
   on the lines this task changed? (`DLV-11`)
5. **Specs.** Is every spec from step 2 tied to a test that proves it, with no
   spec left unproven and no test left unexplained? (`DLV-10`)
6. **Manual law.** Was every `manual` binding the task touched actually signed
   by a person, with the reason recorded? (`DLV-13`)
7. **Deltas.** Are the contract changes and the migrations visible in the
   report, with breaking changes marked? (`DLV-15`)

`turystack-proof` prints this list with real pass/fail. The items whose binding
is `manual` are the ones it stops and asks you to sign.

## Ownership rule

This skill answers **what happens when, and what proves it**. The constitution
answers **which law applies**. The stack skills answer **how the code is
written**. `@turystack/proof-mode-gates` answers **what a check actually runs**. When they
disagree, the constitution wins first, then this skill's sequence, then the
runner's mechanics.
