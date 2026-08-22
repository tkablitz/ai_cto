# `<System>` — Validation Strategy

**Created by Claude `<model>` · last substantive review by `<owner>`, `<date>`**

> **TDD expanded, not replaced.** The *T* is generic. Test-driven means tests are foundational and
> written first — they are how the design gets specified, not how it gets checked afterwards. What
> follows adds layers to that foundation; none of them replaces it, and a team that adopts the later
> layers to avoid writing tests first has kept the cost and thrown away the benefit.

The organizing idea: **every validation layer answers exactly one question, and is blind to the
others.** Line coverage tells you how much code ran. This document tells you which *questions* have
been answered, and which are still open. A suite can be enormous and still leave the important
question unasked — see §2 of `AI-DEV-BEST-PRACTICES.md`.

Fill in the right-hand columns for your system. The blind spots are not fill-in — they are the same
everywhere, and they are why no single layer is sufficient.

---

## 1. The ladder

| Layer | Answers | Blind to | Runs at | Our tooling / threshold |
|---|---|---|---|---|
| **Unit** (test-first) | Does this unit do what the spec says? | Whether the spec was right; anything crossing a boundary | Authoring, every commit | `<framework>` |
| **Contract** | Does the interface match what consumers expect? | Whether either side does the right thing behind the interface | Every commit | `<tool>` |
| **Integration** | Do the parts actually talk? | Whether the whole produces the intended outcome | Every commit | `<framework>` |
| **Property-based** | Does the rule hold across inputs nobody thought to enumerate? | Rules you didn't state | Every commit, or nightly if slow | `<tool>` |
| **Mutation** | Are the tests load-bearing — would a reintroduced bug be caught? | Whether the code was right in the first place | Changed files per PR; full run nightly | `<tool>`, score ≥ `<n>`% |
| **End-to-end behavioral** | Does the feature work in a real environment? | Anything not on the exercised path | Pre-merge, and as a release gate | `<harness>` |
| **Release gate** | Is this build safe to promote? | Everything after promotion | Pre-deploy, on a staging surface | `<pipeline stage>` |
| **Guard** | Does the mechanism protecting an invariant still exist? | Violations the platform can't express | Every commit | `<test path>` |
| **Canary / probe** | Is it still working, right now, in production? | Anything not probed | Continuously, plus daily kickoff | `<probe>` |
| **Restore-from-code** | Can we rebuild this from source alone? | Data and anything created by hand | At least once per phase | `<teardown/rebuild job>` |

---

## 2. What each layer is for, and where it lies to you

Each entry states the blind spot concretely, because a layer trusted past its blind spot is worse
than one you don't have — it produces confidence that isn't earned.

### Unit — the foundation, written first

Write the failing test, then the implementation. The implementer never edits the tests; the
test-writer works from the specification, not from the code (§5.2).

**Blind spot:** a unit test derived from a wrong spec is a wrong test that passes. It also cannot see
anything that only happens across a boundary.

**Ours:** `<framework, layout, naming convention, how to run one test>`

### Contract — the interface holds

**Blind spot:** both sides can satisfy the contract and still be wrong, together, if the contract
describes the wrong interaction.

**Ours:** `<consumer/provider tooling, where contracts live, who owns them>`

### Integration — the parts connect

Mocks emit contract shapes on demand and must never model the domain that decides when those shapes
occur (§2.4). A mock that models the other system is a second implementation of it, and it will
drift.

**Blind spot:** green integration against your mock's idea of an API says nothing about the real one.

**Ours:** `<framework, which boundaries are mocked, which are real>`

### Property-based — the rule holds generally

The direct answer to §2.1's fifth failure shape: *what else produces the shape this predicate
matches?* Example-based tests only ever construct the case the author had in mind. Generated inputs
find the case they didn't.

**Blind spot:** it can only check properties you stated. An unstated invariant is unprotected, and
generators reflect the same assumptions as the tests.

**Ours:** `<tool, which invariants are expressed as properties, seed/replay policy>`

### Mutation — the tests actually bite

The mechanized form of the habit in §2.3: *make every new assertion fail once, on purpose.* Mutation
testing does it for every assertion, continuously, instead of relying on someone remembering — which
by §1.2 makes it a gate rather than vigilance, and therefore a genuine upgrade.

It is strongest exactly where human review is weakest:

- **assertions weaker than their names** — mutate to another wrong value, watch a lazy assertion
  keep passing
- **pass conditions the bug satisfies** — mutate the feature away and see the check stay green

**Blind spot, and it is a sharp one:** mutation testing asks whether your tests are sensitive to
changes in *the code you wrote*. It cannot ask whether the code you wrote was the right code. Where
a test faithfully encodes the wrong intent — §2.1's first and fifth shapes — every mutant dies and
the score is perfect. **A high mutation score on a misunderstood requirement is confident,
well-measured, wrong.**

It is also weakest where the fixture cannot express the failure (§2.1, shape three). The mutant
survives, so a signal does appear — but the natural fix is "write a test that kills it" using the
same blind fixture, and a stubborn survivor in a hard-to-test region gets waived as an equivalent
mutant. The signal shows up; the diagnosis does not.

Treat surviving mutants as a **review queue, not a number to raise.** The triage is the value; the
score is the by-product.

**Ours:** `<tool, scope per PR vs nightly, score threshold, who triages survivors, waiver policy>`

### End-to-end behavioral — the feature is alive

**Blind spot:** anything off the exercised path. Ask of every change: *is there a transition here
that only a live run crosses?* Those are where defects hide, because they are the states fixtures
cannot easily construct.

**Ours:** `<harness, which journeys, where it runs, runtime budget>`

### Release gate — safe to promote

Smoke plus behavioral proofs on a staging surface before promotion. Red means no ship, mechanically.
A red gate is a signal to strengthen tests, never to retry until green (§1.2).

**Blind spot:** everything after promotion. A gate proves the build was good at gate time.

**Ours:** `<stage name, proofs run, promotion rule, who can bypass and how that is recorded>`

### Guard — the mechanism still exists

Tests that assert a *protection* is still in place: a config that pins behavior, a rule that must not
be deleted, a permission that must stay narrow.

**Blind spot, and the reason this layer is listed separately:** a guard that asserts only the
*property* can pass vacuously on a platform where the property is unobservable — and it then keeps
passing after the mechanism is deleted. **Assert the mechanism as well as the property.** The
property assertion catches a real violation where one can occur; the mechanism assertion fails
everywhere, including on the runner that cannot see the property.

**Ours:** `<which invariants are guarded, and for each: the property assertion and the mechanism assertion>`

### Canary / probe — still working now

**Blind spot:** anything not probed, and any failure shorter than the probe interval.

**Ours:** `<probes, interval, where results surface, who is paged>`

### Restore-from-code — rebuildable

Prove it by actually doing it: tear down, rebuild from source through the full gate. If you have
demonstrated it once by hand, you have not demonstrated it (§1.3).

**Blind spot:** data, and anything ever created by hand that nobody wrote down.

**Ours:** `<job, cadence, date last proven, evidence link>`

---

## 3. Placement — why not everything everywhere

Cost and feedback latency decide placement, not thoroughness. A gate developers cannot run locally
is half a gate (§1.2), and a slow gate gets bypassed.

| SDLC point | Layers that run | Budget |
|---|---|---|
| Authoring (local) | Unit, contract, integration, guard | `<seconds>` |
| Pre-merge (CI) | The above + property-based, mutation on changed files, e2e subset | `<minutes>` |
| Nightly | Full mutation run, full property-based, full e2e | `<duration>` |
| Pre-deploy | Release gate: smoke + behavioral proofs on staging | `<minutes>` |
| Post-deploy | Canary, probes | continuous |
| Per phase | Restore-from-code | `<cadence>` |

**Every layer must be runnable on the platforms developers actually use.** Confirm it rather than
assume it — cross-platform defaults (line endings, path separators, case sensitivity, locale) are
the usual reason a gate passes on the runner and fails, or cannot run at all, on a laptop.

---

## 4. Adoption order

Do not adopt this table top to bottom on day one. Each later layer earns its place when the one
before it stops finding things.

1. **Unit, test-first, with the implementer barred from editing tests.** Non-negotiable and first.
2. **Integration + release gate.** Now a red gate means no ship.
3. **Guard tests** for any invariant a config file protects.
4. **E2E behavioral** for the journeys that would embarrass you.
5. **Mutation**, once the suite is large enough that nobody can tell by reading whether it bites.
6. **Property-based**, once a shape-matching predicate has burned you once.
7. **Canary and restore-from-code**, once anything is in production.

Anything deferred gets a written **pull-trigger** — *"adopt when …"* — so the decision stays
revisitable instead of relitigated (§1.11).

Deferred here: `<layer>` — adopt when `<trigger>`.

---

## 5. Review questions

Ask these of the strategy, not of individual tests. The per-test questions are in §2.3.

1. Which question is currently answered by **no** layer?
2. Which layer are we trusting past its blind spot?
3. Which gate has most recently **run**, as opposed to most recently not-failed?
4. If the feature under review were entirely absent, which layers would go red? If the answer is
   "none," the layers are measuring the wrong thing.
5. What did our last surviving-mutant triage conclude — and did anything change as a result?

---

## 6. Status

| Layer | Status | Evidence |
|---|---|---|
| `<layer>` | `<Designed / Prototype / In progress / Complete / Verified>` | `<command, run ID, or link a reviewer can check>` |

Statuses and the five criteria for **Complete** are defined in `templates/definition-of-done.md`.
Every "Complete" carries the evidence that proves it.

<!-- SPDX-License-Identifier: MIT · rev 2026-08-13 · © 2026 Torsten Kablitz · https://github.com/tkablitz/ai_cto -->
