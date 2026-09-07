# Evidence — what has to be shown

**Concept.** Evidence is what lets someone believe the task is done **without
redoing it**. Green gates are part of it and not all of it: they say nothing
broke, not that what was asked for was built.

**Rules defined here:** `DLV-10` · `DLV-11` · `DLV-12` · `DLV-13` — the law is
the *Invariants* table below; every ❌ item cites the id it violates.

## Invariants

| ID | Law | Class | Gate |
|---|---|---|---|
| DLV-10 | Every spec from the plan is tied to the test that proves it; no spec is unproven and no listed test is unexplained. | constitutional | `gate:spec-test-link` |
| DLV-11 | Coverage clears both floors: the codebase floor, and the higher floor on the lines this task changed. | constitutional | `test:coverage-floors` |
| DLV-12 | A required capture that is missing fails the delivery. It is never rendered as an empty placeholder. | constitutional | `gate:required-captures` |
| DLV-13 | Every `manual` binding the task touched is signed by a named person, with the reason recorded. | constitutional | `gate:manual-signed` |

## Both sides

| Evidence | Backend | Frontend |
|---|---|---|
| Specs ↔ tests | unit · integration · e2e | unit · component · e2e |
| Coverage | total floor **and** diff floor | total floor **and** diff floor |
| Contract | endpoints as published, in the docs | the generated SDK consumed, never hand-written |
| Data | migrations, with phase and reversibility | — |
| Screens | — | proposed × shipped, plus every required state |
| Accessibility | — | `axe` clean, violations by impact |
| Size | — | bundle against its budget |

The frontend column is longer because more of its correctness is visible rather
than assertable. The backend column is not shorter because it is easier — it is
shorter because its evidence is mostly tests, and tests are already listed.

## Why the diff floor exists

A codebase at 91% total coverage is compatible with a task that added four
hundred lines covered by nothing. The total moves by fractions; the number that
would have caught it is coverage restricted to the lines this task changed.

```text
total     91.4%  ✓ floor 85    the codebase is healthy
diff      96.2%  ✓ floor 90    what this task wrote is tested
```

The diff floor is **higher** on purpose. Old code has history and reasons; code
written today has neither, so it has no excuse.

## Captures

A screenshot of the happy path proves the happy path. The states that carry the
laws are the other ones:

| Required capture | Because |
|---|---|
| `success` | it is the design's claim, next to the design |
| `empty` | `ARC-ERR-8` — an empty read is a decided state, not a blank |
| `denied` | `ARC-ERR-9` — denial is stated, and only a capture shows it was not hidden |

`error` and `partial` join the list when the surface can reach them
meaningfully. What is *not* acceptable is a delivery where the only picture is
the one where everything worked.

`DLV-12` is the rule that gives this teeth: the report computes its verdict from
the captures it holds. A required one missing turns the verdict red — the same
principle as `ARC-TST-7`, applied to the delivery instead of the test suite.

## Signing a manual binding

A `manual` binding is not a formality and not an automatic pass. It has three
parts, and a signature missing any of them is not a signature:

```text
who      a named person, not "the team"
what     the specific claim they are standing behind
why      what they checked to believe it

✅  ARC-CON-11 · signed by joaogabriel
    "The confirm lists the 3 shipments the cancellation releases; I opened
     the impact read and compared it against the order's shipments."

❌  ARC-CON-11 · reviewed
```

The automated share of the law is printed in the report precisely so `manual`
never gets read as "covered". It is the part a machine could not check, which
makes it the part most worth reading.

## Assembling as you go

Evidence gathered at the end is evidence reconstructed, and reconstruction is
where "I am sure we tested that" comes from. Capture as each slice lands: the
spec↔test link when the test is written, the screenshot when the screen works,
the signature when the reviewer is already looking at it.

## Never do

- A spec with no test, or a test list nobody can map back to a spec (`DLV-10`).
- Reporting the total coverage and calling the task covered (`DLV-11`).
- Shipping with a required capture missing, placeholder rendered (`DLV-12`).
- Marking a `manual` binding as reviewed with no name and no reason (`DLV-13`).
- Screenshotting only the state where everything worked.
- Reconstructing the evidence after the fact, from memory.
