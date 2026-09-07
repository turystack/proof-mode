# Gates — the ladder

**Concept.** Eight rungs, in order, short-circuiting at the first red one. The
order is not taste: each rung is cheaper than the next and, when it fails, makes
the ones below it meaningless.

**Rules defined here:** `DLV-8` · `DLV-9` — the law is the *Invariants* table
below; every ❌ item cites the id it violates.

## Invariants

| ID | Law | Class | Gate |
|---|---|---|---|
| DLV-8 | The ladder runs in order and stops at the first red rung; a later rung never reports on code an earlier one rejected. | constitutional | `test:ladder-order` |
| DLV-9 | A rung is never bypassed to unblock. No ignore comment, no disabled rule, no narrowed glob, no `--no-verify`. | constitutional | `gate:no-bypass` |

## The ladder

| # | Rung | Command | Proves |
|---|---|---|---|
| 1 | Format | `biome format --write . && git diff --exit-code` | the gate is reading code someone formatted |
| 2 | Lint | `biome check .` | layer direction, the stack lints, the GritQL rules |
| 3 | Typecheck | `tsc --noEmit` | the contract holds at the type level |
| 4 | Structure | `turystack-proof structure` | placement, barrels, route shape, generated artifacts untouched |
| 5 | Test | `vitest run` | behavior, at unit and integration level |
| 6 | Coverage | `vitest run --coverage` | both floors: the codebase, and the lines this task changed |
| 7 | End to end | `vitest run --config vitest.e2e.config.ts` | the wiring, against real infrastructure |
| 8 | Visual evidence | `playwright test --grep @evidence` | the screens exist, in every required state |

Rung 1 is first for a reason that looks pedantic and is not: a formatting diff
appearing *after* the gate ran means the gate ran on code nobody had formatted,
and every diff below it is noise.

## Why the order short-circuits

```text
typecheck red  →  the tests below it are running against code that does not
                  compile in the shape the contract says it has
test red       →  coverage of a failing suite is a number about nothing
e2e red        →  the screenshots would capture a broken screen
```

A ladder that runs everything regardless produces a wall of failures whose only
real cause is the first one. Stopping is what makes the output readable.

## What each rung cannot do

Knowing the limits is what keeps the green honest:

- **Lint** sees one file at a time. It catches an import direction; it cannot
  see that two modules together form a cycle.
- **Structure** sees the tree and the declarations. It catches a verb as a path
  segment; it cannot tell that a plural noun is a nominalized action.
- **Test** proves what someone wrote a test for. Coverage says how much code ran,
  never whether the assertions were worth making.
- **E2E** proves the wiring on the paths it exercises.
- **Visual** proves a capture exists and matches its baseline; whether the
  screen is *right* is `manual`.

Each of those limits is why `manual` bindings exist, and why a report that
prints its automated share is more trustworthy than one that implies total
coverage.

## Bypassing

`DLV-9` is absolute, and the reasoning is short: a bypass is invisible three
weeks later, when the rule it silenced is the one that would have caught the
incident.

When a rung is genuinely wrong — a false positive, a rule that does not fit a
legitimate case — the fix is at the rule, not at the call site:

```text
❌  // biome-ignore lint/...: unblocking
❌  narrowing the glob so the file stops being linted
❌  git commit --no-verify

✅  fix the rule in @turystack/*-config, with a fixture proving both sides:
    one file that must fail, one that must pass
✅  if the law is wrong, change the law — in the skill, with its id
```

The fixture pair is the part people skip. A rule written from a single example
catches that example; the negative fixture is what stops it from silently
catching nothing after a refactor.

## Reading a gate report

The report prints, per rung: the command, the result, and the state. Two things
are worth checking beyond the colour:

- **The command is the real one.** A rung that reports green while running
  `--passWithNoTests` on a project with no tests is a rung reporting on nothing.
- **The evidence count.** `3 of 3 captures` is a gate; `captures: optional` is a
  placeholder wearing a gate's colour.

## Never do

- Reordering the ladder so a cheap rung runs after an expensive one (`DLV-8`).
- Any form of bypass (`DLV-9`).
- Treating a `manual` binding as automatically satisfied because the automated
  rungs are green.
- Letting a rung run with a flag that makes it unable to fail.
