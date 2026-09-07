# Context — what the task is built on

**Concept.** Before any code, the task has five inputs. Each one either resolves
to a location someone can open, or it is missing — and a missing input is a
question for a person, not a gap to fill with a reasonable guess.

**Rules defined here:** `DLV-1` · `DLV-2` · `DLV-3` — the law is the
*Invariants* table below; every ❌ item cites the id it violates.

## Invariants

| ID | Law | Class | Gate |
|---|---|---|---|
| DLV-1 | Every one of the five inputs resolves to a real location before implementation starts. | constitutional | `gate:context-resolved` |
| DLV-2 | A missing input stops the part of the task that depends on it. It is never invented, defaulted or deferred silently. | constitutional | `manual` |
| DLV-3 | The resolved locations are recorded in the delivery report, not held in memory. | constitutional | `gate:context-recorded` |

## The five inputs

| Input | Answers | Lives in | Section |
|---|---|---|---|
| **Capability spec** | what should happen, including the unhappy paths | `<project>-spec` | `01-definition.md` |
| **Domain model** | which entities exist, their states and legal transitions | `<project>-spec` | `02-domains.md` |
| **Design** | what the screen looks like, per required state | `<project>-uiux` | `05-assets.md` + `assets/` |
| **UI/UX rules** | how this product words, spaces and behaves | `<project>-uiux` | `01-brand.md`, `02-layout.md`, `03-copy.md` |
| **API contract** | the shape the two sides agreed on | the generated SDK | never hand-written |

### Finding this project's skills

They carry the project's name, so nothing resolves them from a constant. The
CLI writes the answer when it materializes them:

```json
// .claude/turystack.json  (and .codex/turystack.json)
{ "project": "acme", "skills": { "spec": "acme-spec", "uiux": "acme-uiux" } }
```

Read that first. It sits beside `skills/` rather than inside a skill, because a
law skill is rewritten on every install and would lose it.

When the manifest is absent — an older project, a manual install — fall back to
the shape: the project's own skills are the folders in `.claude/skills/` whose
names end in `-spec` and `-uiux`. Everything else there is law, installed under
its canonical `turystack-*` name and never edited by the project.

Both are materialized once from `@turystack/spec-template` and
`@turystack/uiux-template` and never overwritten, so their content is the
project's from the first day.

A backend-only task still needs the first two and the fifth. A frontend-only
task needs all five. A refactor needs the first two, because "behavior
unchanged" is a claim about a behavior somebody wrote down.

## What resolution means

An input is resolved when you can point at **the thing itself**, not at its
category:

```text
❌  "the spec says orders can be cancelled"
✅  acme-spec › 01-definition.md › Cancel an order — with its unhappy paths

❌  "there's a design for it"
✅  acme-uiux › assets/orders-table.png, indexed in 05-assets.md, exported 2026-08-12

❌  "the endpoint exists"
✅  apps/admin/src/~sdk/orders — cancelOrder, generated from the current contract
```

There is a third failure mode, and it is the most common: the section exists
and still carries its marker.

```markdown
<!-- turystack:unfilled -->
```

That is not resolved. It is the project saying, in a form a machine can read,
that nobody decided this yet — so `turystack-proof` reports it as a missing
input rather than leaving it to whoever notices first.

The difference matters because the second form can be checked by someone else,
and because half of all "missing spec" discoveries happen at exactly this step:
the file exists, and it does not answer the question the task asks.

## When something is missing

Missing is normal. Stopping the whole task is usually wrong. Split it:

```text
task: "cancel an order from the table"

resolved     domain model, API contract, UI/UX rules
missing      what happens when the order is already cancelled
             (spec is silent)

→ everything that does not depend on that answer proceeds:
  the table action, the permission gate, the deep link, their tests

→ the branch that does depend on it stops, and the question goes out:
  "already-cancelled order: hide the action, show it inert, or let it
   fail with a message? The other two produce different tests."
```

Three properties make the question useful rather than a blocker:

- **It is specific.** Not "what about edge cases" — the exact branch, with the
  options, and what each one changes.
- **It carries the cost.** Naming what each answer implies is what lets a
  non-engineer answer it quickly.
- **It does not stall the rest.** The unblocked work continues, so the answer
  arrives into a task that is already mostly done.

## Assumptions, when they are unavoidable

Sometimes nobody is available and the work must continue. Then the assumption
becomes a first-class artifact, not a silent decision:

- it is written into the delivery report, in the context block;
- the code path it produced is marked, so it can be found again;
- it is phrased as a decision someone can reverse in one place.

An assumption you can find later is a risk. An assumption spread through the
code by memory is a defect waiting for the person who wrote it to leave.

## Never do

- Starting implementation with an input unresolved because "it will be obvious
  from the code" (`DLV-1`).
- Filling a silent spec with the most reasonable behavior and moving on
  (`DLV-2`).
- Reading the design from a screenshot in a chat message rather than from
  `<project>-uiux` › `assets/` (`DLV-1`).
- Treating a section that still carries its unfilled marker as resolved
  (`DLV-1`).
- Hand-writing a type, a permission id or a payload shape because the contract
  has not been generated yet — that is `ARC-CTR-5`, and it is a blocker.
- Resolving the context, then not recording it, so the next person repeats the
  same archaeology (`DLV-3`).
