# Build — slices and the gate loop

**Concept.** Implementation is a loop, not a phase. One slice, gated; then the
next. The alternative — write everything, gate once — is not faster, it only
moves all the failures to the end and mixes them together.

**Rules defined here:** `DLV-6` · `DLV-7` — the law is the *Invariants* table
below; every ❌ item cites the id it violates.

## Invariants

| ID | Law | Class | Gate |
|---|---|---|---|
| DLV-6 | Every slice is gated as it lands. The ladder is never deferred to the end of the task. | constitutional | `manual` |
| DLV-7 | Red is fixed before the next slice starts. Failures are never accumulated to be dealt with later. | constitutional | `manual` |

## The loop

```text
        ┌──────────────────────────────────────────┐
        │  write the smallest provable slice       │
        └───────────────────┬──────────────────────┘
                            ▼
        ┌──────────────────────────────────────────┐
        │  turystack-proof                          │
        └───────────────────┬──────────────────────┘
              red ──────────┴────────── green
               │                          │
        fix now, in this slice      next slice
```

There is no third branch. "Green except for one lint I will clean up later" is
red, and it is the branch that ends with a task whose last hour is spent on
problems from its first.

## Why gating per slice is cheaper

Three effects compound, and none of them is about discipline:

- **Attribution.** A rung that goes red on a fifty-line slice names the change
  that broke it. The same rung on a thousand-line task names nothing, and the
  next twenty minutes are bisecting.
- **Cost of the fix.** A layer violation caught in the slice that introduced it
  is a moved import. Caught three slices later, it has consumers.
- **Design feedback.** Gates encode the law. A rung that keeps going red on the
  same slice is usually not a nagging lint — it is the architecture saying the
  cut was wrong, and it is saying it while the cut is still cheap to change.

## Which rungs to run per slice

The full ladder is in `04-gates.md`. Per slice, run what the slice can affect —
and run the full ladder before the task is declared finished:

| Slice touches | Run at minimum |
|---|---|
| any code | `format`, `lint`, `typecheck` |
| a file's placement, a route, a barrel, a generated artifact | `+ structure` |
| behavior | `+ test` |
| a use-case, controller, handler or migration | `+ e2e` |
| a screen | `+ visual` |

`coverage` is worth running per slice too, on the diff floor: it is the rung
that tells you a slice shipped code nothing exercises, which is exactly the
moment to write that test rather than three slices later.

## Working with a red rung

Read the rung, not the symptom:

- **`lint` red on a layer rule** — the import direction is wrong. Moving the
  file is usually the fix; adding an exception never is.
- **`structure` red** — something is in the wrong place, or a generated artifact
  was edited. Regenerate rather than hand-fix (`ARC-CTR-4`).
- **`test` red on a test you did not touch** — the slice changed behavior
  something else depended on. That is the gate doing its job; decide whether the
  dependency or the change is wrong.
- **`coverage` red on the diff floor while the total floor is green** — the
  slice added code nothing exercises. The total number was never going to catch
  it.
- **`visual` red** — a required capture is missing. That is the rung refusing to
  let evidence be optional.

## Never do

- Deferring the ladder to the end of the task (`DLV-6`).
- Starting a slice on top of a red one (`DLV-7`).
- Disabling a rule, adding an ignore comment or narrowing a glob to make a rung
  pass (`DLV-9` in `04-gates.md`).
- Committing a slice whose tests you did not run because "it is only a rename".
- Letting a slice grow until a red gate can no longer say which change caused
  it.
