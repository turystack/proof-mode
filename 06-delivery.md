# Delivery — the report and the handoff

**Concept.** A task is delivered when one artifact says so: the gate report.
Not a message, not a checklist in a pull request description, not a verbal "it
is done" — one page that a reader can open and disagree with.

**Rules defined here:** `DLV-14` · `DLV-15` · `DLV-16` — the law is the
*Invariants* table below; every ❌ item cites the id it violates.

## Invariants

| ID | Law | Class | Gate |
|---|---|---|---|
| DLV-14 | A task is delivered as one report, whose verdict is computed from the gates and the evidence it holds — never asserted. | constitutional | `gate:report-emitted` |
| DLV-15 | Contract changes and migrations appear in the report, with breaking changes marked. What others depend on is never delivered silently. | constitutional | `gate:contract-delta` |
| DLV-16 | Every id the task claims is named in the report beside the test that proved it, and the report is attached to the board task that asked for it. A report nothing points at is evidence nobody finds again. | constitutional | `gate:rule-id-coverage` |

## What the report contains

The runner emits `gate-report.json` and renders it as a page. The blocks, in
order, and what each one is there to stop:

| Block | Stops |
|---|---|
| Verdict + gate ladder | "it works on my machine" |
| What changed | green whose scope nobody checked |
| What this was built from | work built over a missing input (`DLV-1`) |
| Law coverage | a `manual` binding read as automatically satisfied |
| Contract delta | a breaking change discovered by a consumer (`DLV-15`) |
| Backend evidence | a spec nobody tested; a migration nobody described |
| Frontend evidence | a happy-path screenshot standing in for five states |
| Raw output | a green summary that the tools never actually printed |

The raw block matters more than it looks. A report is a summary, and a summary
is a place where a number can be wrong; keeping the verbatim output means the
green can always be checked against what the tools said.

## The payload

One shape, always. The runner writes `gate-report.json` and injects it into the
page; both carry the same object, and the page computes its verdict from it
rather than reading one.

Writing this down is not documentation for its own sake. The checks that read
the report and the template that renders it were built from two different ideas
of this object, and three of the checks spent their life reporting `skipped`
against a report full of what they were looking for — a silence that reads
exactly like "not applicable". A shape nobody wrote down is a shape everyone
guesses.

```jsonc
{
  "schema": "turystack.gate-report/1",

  // Which task, and from which commit. A report whose commit is not the one
  // under review describes a different task.
  // `covers` is the ids this task claims — from the board task it closes,
  // which `board` points back at. They are what an audit reads.
  "task":     { "id", "title", "board", "covers", "branch", "baseline",
                "commit", "author", "finishedAt", "durationMs" },

  // Which runner, and which version of the law it ran under. Green under one
  // version of a skill is not evidence under another.
  "provenance": { "runner", "skills": [{ "name", "version" }] },

  // The eight rungs, in order. A rung below a red one is absent, not "skipped
  // because it passed" — see 04-gates.md.
  "ladder":   [{ "id", "name", "command", "state", "result", "note" }],

  // Scope. `untested` is the number that makes a green coverage total honest.
  "changed":  { "files", "added", "removed", "untested",
                "entries": [{ "path", "added", "removed", "covered" }] },

  // The five inputs of 01-context.md, each resolved to something openable.
  "context":  [{ "kind", "title", "source", "resolvedBy" }],

  // Every law this change touched, and what proved it. `detector` is the
  // proof: `biome:…`, `grit:…`, `gate:…`, `test:…` or `manual`.
  // A `manual` binding carries `reviewer` and a `note` saying what they
  // checked — not what they concluded (DLV-13).
  "law":      { "total", "touched", "automated", "manual", "violations",
                "bindings": [{ "id", "law", "detector", "state",
                               "reviewer?", "note?" }] },

  // What others now depend on. `breaking: true` is allowed; invisible is not.
  "contractDelta": [{ "kind", "change", "name", "breaking", "note" }],

  "backend":  { "coverage", "specs", "endpoints", "migrations", "docsShot" },
  "frontend": { "coverage", "specs", "outcomes", "accessibility", "bundle",
                "screens" },

  // Verbatim tool output. A summary is where a number can be wrong.
  "raw":      { "biome", "vitest", "coverage" }
}
```

Two sub-shapes carry most of the weight:

```jsonc
// backend.specs[] and frontend.specs[] — the spec↔test link of DLV-10.
// A spec with no `test` is a requirement nobody proved, and one whose `spec`
// opens with no id is a requirement no audit can find (DLV-16).
{ "spec", "test", "level", "state" }

// frontend.screens[] — the proposed design beside the shipped screen.
// A `required` state with no image fails the delivery (DLV-12); it is never
// rendered as an empty placeholder.
{ "name", "route", "proposed",
  "required": ["success", "empty", "denied"],
  "captures": [{ "state", "image" }],
  "checks":   [{ "text", "ref", "state" }],
  "tests":    [{ "test", "state" }] }
```

The canonical example is the payload inside
`@turystack/proof-mode-gates` › `templates/report.html`. It is not a sample kept
beside the code — the runner's own tests read it as their fixture, so the shape
documented here, the shape the checks read and the shape the page renders are
one thing.

## The verdict is computed

`DLV-14` is what separates this report from a template someone fills in. The
page derives its own verdict:

```text
verdict = every rung green
          AND every required capture present
          AND every touched manual binding signed
```

So a delivery cannot be declared green by writing `"verdict": "pass"` in the
payload. If three required captures are missing, the page says so — in the
chip, and in a banner naming which ones. A report that renders a placeholder
instead of failing is a report that lies politely.

## The ids, and the board

`DLV-16` is what makes a report auditable six months later, and it has two
halves that fail separately.

**The ids.** A request arrives as a sentence and the project's spec turns it
into ids; the report is where those ids meet the tests that proved them. So a
spec entry opens with the id it satisfies:

```text
❌  "A cancelled order cannot be cancelled again"
✅  "AC-2 · A cancelled order cannot be cancelled again"
```

The difference is not cosmetic. The first form produces a report that lists
twenty green tests and cannot answer "was `AC-2` implemented?" — which is the
only question an audit ever asks. The second form answers it without opening a
file.

**The board.** The report is written to a path the board task points at, and the
task's status becomes `done` because that report exists. Two things stop being
possible: a task marked done whose gates nobody ran, and a report that exists in
a run's output and nowhere anyone will look.

```text
board task T-4  covers AC-1, AC-2, AC-4
                report reports/T-4/report.html
                status done — because the report is there
```

The project's spec skill owns the board's shape and the rule that a status
without a report is a claim; this section owns the other end of it — that the
report says which task it closes, and which ids it proved.

## The contract delta

`DLV-15` exists because the cross-stack seams are where silent breakage lives.
Every one of these is something another person now depends on:

| Kind | Question the reader has |
|---|---|
| endpoint | new, changed or gone? |
| error code | can a consumer branch on it? (`ARC-ERR-2`) |
| permission | published from the catalogue? (`ARC-SEC-12`) |
| event | absolute state, with its identifier? (`ARC-IDM-5`) |
| field | nullable, so old consumers keep parsing? (`ARC-CTR-6`) |

A breaking change is not forbidden — it is required to be **visible**, with the
rollout that keeps both versions alive while a consumer remains.

## The handoff

Three things travel together, and the last one is the one people forget:

1. **The report**, as the page and as `gate-report.json` for CI.
2. **The branch**, at the commit the report was generated from — a report whose
   commit is not the one under review describes a different task.
3. **The open questions**, if any: assumptions recorded in the context block
   (`01-context.md`), and anything a person still has to decide.

A delivery with an open assumption is still a delivery, as long as the
assumption is on the page. A delivery with a hidden one is a defect with a
green report attached.

## Never do

- Declaring done in a message instead of a report (`DLV-14`).
- Writing the verdict into the payload rather than letting it be computed
  (`DLV-14`).
- Shipping a new permission, error code or event without it appearing in the
  contract delta (`DLV-15`).
- A spec entry in the report that cites no id, leaving an audit nothing to
  match against the project's spec (`DLV-16`).
- Marking a board task done with no report attached to it (`DLV-16`).
- Handing over a report generated from a different commit than the one under
  review.
- Leaving an assumption in the code and out of the page.
