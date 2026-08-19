# AI-Assisted Development: Best Practices for Real Systems

A portable field guide for teams adopting AI coding assistants on production software.
Everything here was learned by shipping — versioned releases, gated continuous deployment,
~1,800 automated tests, restore-from-code proven — with an AI writing most of the code and a
human owning the outcome.

**The foundations are DevOps and TDD, and they are not negotiable.** Every other practice in this
guide exists to keep those two honest when a machine is producing code faster than anyone can read it.

> **How to use this.** Read Part 0 and Part 1 before you configure anything. Part 6 is the
> copy-paste setup. Parts 2–5 are what you come back to when something bites you. Part 7 is the
> catalogue of ways tooling lies to you.

---

## Part 0 — The thesis

**AI raises the volume of code. It does not raise the quality of code.** A capable model produces
plausible, well-structured, confidently-explained work at a rate no human review process was
designed to absorb. That is the whole problem in one sentence.

Three consequences follow, and they drive everything below:

1. **Velocity without mechanical gates automates incident production.** If your safety depends on
   someone noticing, you have no safety. It has to be a pipeline that says no.
2. **The reviewer is now the bottleneck, so spend the review budget where it pays.** Reading every
   line is neither possible nor useful. Reading the *tests* — adversarially — is.
3. **"It's green" is the most dangerous sentence in AI-assisted development.** See Part 2. This is
   the section that will save you the most pain, and it is the least intuitive.

The counter-intuitive result of running this way: you go **faster** with more discipline, not less.
Gates remove the human approval queue. Tests remove the fear of shipping. The teams that get burned
by AI coding are the ones that kept the old ceremony and dropped the engineering.

---

## Part 1 — The non-negotiable foundations

Each principle below names **the mechanism that enforces it**. A principle with no enforcement
mechanism is decoration — an AI will agree with it warmly and then route around it.

### 1.1 TDD, with teeth

- **Failing test first.** Write the test, watch it fail, then implement until green.
- **The implementer never edits the tests.** If a test is wrong, that is a separate, deliberate,
  reviewed change — not a step in getting to green. This single rule prevents most test-gaming.
- **Every defect found anywhere — dev, staging, production, a user report — becomes a failing
  regression test before its fix merges.** No exceptions, no "it's obvious."
- **The suite is your deploy-readiness evidence.** It is not a chore attached to the work. It *is*
  the artifact that lets you ship without a human approver.

> **The AI-specific trap: plan-supplied tests get no independent signal.**
> When the AI writes both the tests and the implementation into one plan, the TDD loop verifies
> nothing — the suite can only ever be as good as its test block was, and both halves share the
> same misunderstanding. **Review AI-written tests at least as adversarially as AI-written code.**
> Better: have a *separate* agent write tests from the spec, without reading the implementation.

*Enforcement:* CI runs the suite on every push; the role separation in Part 5 keeps the test-writer
and the implementer apart; the pre-merge checklist in the Appendix asks "which test fails without
this fix?"

**The *T* is generic, and this is where TDD grows rather than gets replaced.** Test-first is the
foundation; unit tests are only its first layer. Contract, integration, property-based, mutation,
behavioral, guard, canary and restore-from-code proofs are all *tests* in the same sense — each
answers one question the others are blind to, and each belongs at a different point in the lifecycle
because cost and feedback latency differ. `templates/validation-strategy.md` lays the layers out
with their blind spots stated, and Part 2 is the argument for why the blind spots matter more than
the counts.

### 1.2 Gates, not vigilance

- CI runs everything on every push. Not nightly. Not on demand.
- Deploys pass **automated release gates** — smoke plus behavioral proofs — on a staging surface
  before promotion. Red gate means no ship, mechanically.
- **A red gate is a signal to strengthen tests, not to retry until green.** "Re-run it" is the
  single most corrosive habit available to a team with a flaky gate.
- **A gate that fires when nothing is wrong is how that habit forms.** Flakiness is the familiar
  cause. The worse one is a check that is *deterministically* wrong on the healthy path, because
  reliable noise is easier to learn to ignore than intermittent noise. One migration gate asked
  whether any local branch held a commit no remote had — the right question, which under a
  squash-merge workflow answers *yes* on every healthy clone forever, since squashing gives the
  merged commit a new SHA. Every alarm it raised would have been false, and the habit of clearing
  those without looking is precisely what let a genuinely stranded branch sit unnoticed for six
  days. The fix is rarely to delete the check, which was asking something worth asking; it is a
  second, cheaper tier that separates the benign answer from the real one, so the first tier's
  alarms become cheap to clear rather than expensive to investigate.

> **Before adopting a check, run it against a known-good state.** If it fires, you have not built a
> gate — you have built a training exercise in ignoring gates.
- **No manual approval gates in the pipeline.** Merge to trunk, green CI + CD, it ships. Safety
  lives in the depth of the gates, not in a human clicking approve. This is the DevOps Handbook's
  First Way: build quality in, prefer peer review over change-approval boards.
- **Every gate must be runnable on the platforms your developers actually use.** A gate only the
  runner can execute turns a five-second local check into a push-and-wait cycle, and it teaches
  people to push in order to find out. See Part 7 for how this hides.

> **Scope this honestly.** No-approvals CD is correct when *you own the blast radius* — internal
> tools, your own services, systems whose failure costs you and not a customer. For a
> customer-facing product with real money or safety exposure, keep the gates automated and add the
> promotion control your risk actually justifies. The principle that survives either way: **the
> gate is mechanical; the judgment went into the gate, not into the moment of shipping.**

*Enforcement:* the pipeline. If your pipeline can be bypassed, that's the first thing to fix.

### 1.3 Standing tests over one-offs

Every proof becomes a **re-runnable artifact** — a committed script, a test, a pipeline gate.

**If you demonstrated something once by hand, you have not demonstrated it.** A manual walkthrough
says nothing about the code six weeks from now, and six weeks from now is exactly when you'll need
to know. When you catch yourself doing a one-off validation, ask whether it should be promoted to a
standing gate — the answer is usually yes, and it usually costs ten extra minutes.

This compounds: the accumulated gate suite *is* the evidence that makes no-approval CD safe.

**And it applies to artifacts, not only to code.** Ask what a repository *asserts*, not what it
executes. A docs, config, or infrastructure repo asserts that its files are licensed, that the
references in its prose resolve, that the commands it documents still run, and — if it is public —
that its contents are safe to publish. Every one of those can quietly become false. None of them
has a compiler to notice, which is precisely why nobody thinks to test a repository that holds no
code: the invariants are real and the violations are silent.

The tell is a checklist. **If a repo's rules file documents commands to run before a push, those
commands are its test suite** — written down, un-run, and dependent on somebody remembering. One
such repo had accumulated ten, executed by hand and retyped slightly differently each time. A
retyped check is a fresh opportunity to author it wrong, and nothing guarantees the mistake fails
safe rather than reporting a clean result it never earned.

Two bounds keep this from becoming ceremony:

- **Gate only the assertions whose failure is expensive to discover late.** A slow or noisy gate
  gets bypassed, and a bypassed gate is worse than none, because it still reads as protection.
- **Where the failure is irreversible, the gate must run before the irreversible step.** For a
  public repository that means a local pre-push hook, not CI. A pipeline runs *after* the push,
  and for a public repo the push *is* the publication — including a branch push. A leaked name is
  fetchable and forkable before the first workflow starts. CI is the right tool for everything you
  can fix afterwards, and the wrong one for anything you cannot.

### 1.4 Everything as code

- Infrastructure in IaC, deployed **only** by pipeline. Portal/console changes to owned resources
  are prohibited — they are undocumented, unreviewable, and invisible to the next rebuild.
- **Prove restore-from-code by actually doing it**: tear down, rebuild from the repo, through the
  full gate, at least once. An untested disaster-recovery story is a wish.
- Config that genuinely cannot live in code (a console-side toggle, a directory role grant) is
  documented in the repo as an explicit, copy-pasteable manual step. Not a memory. Not a ticket.

### 1.5 Secretless by default

- Workload identity / OIDC federation for pipelines. Managed identity plus vault references at
  runtime. Local development uses a user-secrets store, never a checked-in file.
- Zero credentials in source, **enforced by a scanning gate** — not by discipline.
- On any leak: rotate immediately and treat version-control history as compromised.

### 1.6 Least privilege — including for the AI

**Never let an automated script run under your owner/admin identity.** Use a purpose-scoped service
principal or role. If no correctly-scoped identity exists, create one — do not lend your own access
as a shortcut.

> Learned expensively: a cleanup routine scoped to `--mine` was run locally under an owner
> identity. Under that identity, "mine" meant *everything that owner had ever created*, and it
> permanently deleted ~31 archived run artifacts from a shared store with no snapshots. The same
> script was safe in CI, where it ran as a narrowly-scoped test identity whose "mine" was three
> throwaway records.

**Before running anything that deletes or mutates shared state, check the blast radius of the
identity you are holding**, not just the correctness of the command. And prefer scripts that delete
by explicit ID over scripts that delete by scope filter — the footgun is the filter.

### 1.7 Visibility is a core DevOps principle

Surface results in the platform's **native views**: test results in the CI UI, coverage, deployment
history per environment, analytics. Publish on failure as well as success — a red build with no
per-test breakdown wastes the most valuable signal you get all week.

**Treat "it runs but you can't see it" as a defect.** Work that isn't visible can't inform decisions
or build confidence, and confidence is the currency that buys you the no-approvals pipeline.

### 1.8 Definition of Done, with evidence

Adopt a shared vocabulary early and make everyone — human and AI — use it.

Five statuses: **Designed · Prototype · In progress · Complete · Verified.**

Five criteria for **Complete**: runs in the named environment · enforced with no bypass path ·
tested including failure cases with retained results · reproducible from source control ·
observable and documented.

**Claim-with-evidence:** every "Complete" carries the command, link, or run ID that proves it. The
entire review technique compresses to two words: *"Completed — evidence?"*

Two failure patterns to police relentlessly, both of which AI output produces naturally:
- **Aspirational-as-current** — the target state written in the present tense.
- **Demo-as-done** — a sandbox walkthrough labeled Complete.

And one rule about handoffs: **"in QA" is not done.** Quality ownership does not transfer at a gate.
Everyone writes code and everyone tests; you own your deliverable's quality through production.
Don't let a progress metric count in-flight work as complete to make a bar look better.

### 1.9 Keep dependencies fresh

Run the latest stable runtimes and libraries. Treat end-of-life warnings as **action items, not
noise**. When a dependency bumps, re-run the gates and redeploy even with no functional change —
the no-approvals pipeline makes this nearly free, and small continuous updates beat a big, risky,
catch-up migration every time.

### 1.10 Stateless by default

Design every service to be as stateless as reasonable so it can run as multiple instances. State
belongs in external stores, never trapped in instance-local memory or disk that can't be
reconstructed. If you must ship single-instance, **document that scale-out is unsafe** so nobody
flips the switch in a console and quietly breaks production.

### 1.11 Buy over build — but YAGNI governs

Use a managed service when it replaces code you'd otherwise write **for a need that exists today**.
Never adopt a service because it exists.

This one is easy to over-apply, and AI assistants over-apply it enthusiastically — ask for an
architecture and you'll get a thirteen-layer reference diagram. The test per component: *does this
service replace code we would otherwise write, for a need we have now?* If no, defer it with a
written **pull-trigger** ("adopt when X") so the decision is revisitable instead of relitigated.

### 1.12 A missing tool is a question, not a constraint

An agent reached for a standard JSON tool, found it absent, and hand-rolled the manipulation instead.
Another reached for a scripting runtime, found it missing, and took a longer path. Both produced
working results. Neither mentioned it. Installing either tool would have taken the owner a minute,
and he would have said yes.

**The instinct was right and the silent redirect was wrong**, but the interesting part is *why* this
happens to agents specifically and not to the developers they work alongside. A developer who hits a
missing tool feels something — tedium — and that feeling is what produces the install. It is not a
cost-benefit calculation; it is that hand-rolling the thing is *annoying*. **An agent feels none of
it.** Writing sixty lines costs it exactly what writing three costs it, so the pressure that normally
drives tooling investment is the one signal the agent structurally cannot generate. Left alone, the
environment never improves and the codebase quietly fills with workarounds nobody chose on the merits.

**The second cost is that the workaround is invisible in the artifact.** A reviewer sees sixty lines
of string manipulation and cannot see that a tool was missing. The alternative was never recorded, so
nobody can ask why it was not taken. The code does not carry the constraint that shaped it.

> **When you reach for a tool that isn't there, say so.** First check whether it exists under another
> name — CLIs often embed the thing you want, and a runtime you already have may cover it. If it is
> genuinely absent and the workaround costs more than a few lines or would recur, **ask**, naming
> what you wanted it for, what going without costs, and what adding it costs: provenance, licence,
> and attack surface are the owner's call, not the agent's. **Never install it yourself**, especially
> on a machine that touches someone else's code. Meanwhile keep working and mark the fallback
> provisional; stop only if shipping the fallback is worse than waiting.

**Record the answer where the next session will find it — refusals as much as approvals.** A "denied,
because X" entry costs one line and stops three agents rediscovering the same dead end. Otherwise
every session on that machine pays the tax again, independently, and none of them knows the others
did.

**And if you do route around a missing tool without asking, say so in the handoff.** That is the
minimum: the decision becomes visible to someone who can overrule it.

---

## Part 2 — Why a green suite is weak evidence

This is the most important section in this guide.

**In one day, nine defects survived a green suite of ~1,000 tests. In four of them, the test itself
was protecting the bug.** Not missing — *protecting*. The suite ran, passed, and actively encoded
the broken behavior as correct.

Running the suite cannot find this class of failure. Only a specific kind of reading can.

### 2.1 The five failure shapes

**1. The test asserts the defect is correct.** A test named for a fallback asserted that the silent
fallback was the desired behavior. The fix had to *invert* that test, not delete it. Whenever a fix
requires changing an existing test, stop and ask which of the two was wrong.

**2. The assertion is weaker than the name.** A test named as the guard for the headline bug
asserted only `latitude != 0`. A regression that returned the same wrong coordinate for every
record passed cleanly. **Ask of every test: does the assertion match what the name claims?**

**3. The fixture cannot express the failure.** The deepest one. Every staged search returned the
same result regardless of the search radius, so "results actually within reach" and "results that
exist somewhere" were indistinguishable in every test that existed. That blind spot hid a critical
defect for weeks: a component was classifying availability from data its callers could never use,
and confidently reporting healthy. **Ask what your fixtures make it impossible to observe.**

**4. The check's pass condition is satisfied by the bug.** An end-to-end verification script
asserted "no errors · both sources seen · counter monotonic." It passed all day while the feature
was entirely dead: the client was rejecting *every* frame, and a counter fed by nothing else is
monotonic *because* of that. Rejecting everything is trivially monotonic.

> **The question that catches this: would this check still pass if the feature were entirely
> absent?** If yes, it is not a check.

**5. The predicate matches a shape, and something else has that shape.** A helper ended in
`!error.response && !!error.request`, with a test named "network error (redirect-to-login) is an
auth failure." A request timeout has that identical shape. A month later, when the service got
slow, every timed-out poll was classified as an expired session and the page reloaded — wiping the
user's work. Green throughout, because the test only ever constructed the case the author had in
mind.

> **A shape-matching predicate will eventually match something that shares the shape and nothing
> else.** Ask: *what else in this system produces this shape?* If the answer is "nothing today,"
> write the exclusion list anyway — "today" is the part that expires. Prefer naming the cases you
> are ruling *out* over widening the positive test.

### 2.2 Four more that look like passes

**A conditional gate can be skipped many times in a row, and its skips look like passes.** A
post-deploy assertion only executed when a live workload existed at deploy time. Three consecutive
green deploys had none, so it silently never ran — and a change shipped through all three without
that path executing once. The gate wasn't flaky or wrong; nobody checked that its *precondition* was
ever met.

> **Ask of any gate: has it actually RUN recently, or has it only not-failed?** Grep the build log
> for the gate's own output line, not for the build result. When you control the precondition,
> arrange it deliberately.

**A whole suite can be green and the feature still dead.** 1,209 backend tests, 630 UI tests and two
mutation checks all passed against a fix, and a 60-second manual run then showed the bug intact.
Nothing asserted what the system reported *after* a session left the registry — the entire run-end
transition had no coverage at all.

> **Ask of any change: is there a transition here that only a live run crosses?** Those are where
> defects hide, because they are the states your fixtures can't easily construct.

**An assertion can be unobservable where it runs, and pass for that reason.** A repository pinned
line endings with a `.gitattributes` rule and guarded it with a test asserting the property: *no
tracked file has CRLF*. On the Linux CI runner that test passes whether or not the rule exists —
nothing on that platform produces CRLF to begin with. Delete `.gitattributes` and the guard stays
green forever, on the only machine that runs it. This is the fourth failure shape wearing platform
clothing: *would this check still pass if the feature were entirely absent?* On Linux, yes.

> **When a property is only observable on some platforms, assert the mechanism as well** — that the
> rule guaranteeing the property exists and says what you think it says. The property assertion
> catches a real violation where one can occur; the mechanism assertion fails everywhere, including
> on the runner that cannot see the property. Neither is sufficient alone.

**A check can return success for the command while the thing it checked failed.** An automated CI
verification reported a red run as green. The flag carrying the run's conclusion into the exit code
had been omitted, so the exit code described the watch rather than the run — and watching a failed
run is a successful watch. Piping the command discards the exit code regardless, since a pipeline
reports its last stage. Both forms print that the run failed, and return zero. Then the obvious
repair inherits the defect: reading the run's conclusion instead reports an *in-progress* run as
passing, because a run still executing has no conclusion at all.

> **Ask of any check: what question did it actually answer?** "The command succeeded" and "the thing
> it inspected passed" are different questions that agree just often enough to be trusted. Read the
> state you care about, require that state to be terminal before reading it, and treat not-yet-known
> as a third outcome — a two-state check cannot report a three-state world.

### 2.3 The review technique

When reviewing a batch of tests — yours or the AI's — spend the effort on these five questions:

1. Does each test's assertion match what its **name** claims?
2. What can the **fixtures** not express?
3. Which tests would **still pass** against the reintroduced bug?
4. Would this check still pass if the feature were **entirely absent**?
5. What **else** produces the shape this predicate matches?

And one habit that is worth more than all five: **make every new assertion fail once, on purpose.**
Break the thing it guards, watch it go red, then fix it. Ten seconds. It is the only way to know
your test is connected to anything.

### 2.4 Mocks carry minimal state

*"Test connecting to the database, not the database itself."*

A mock **emits contract shapes on demand** — a deterministic 409 on the Nth call, a fault knob, an
error trigger — so your client's code paths become exercisable. It must **never model the domain
that decides when those shapes occur**. No inventory model, no arbitration logic, no simulated
economy.

**Why:** every behavior you replicate in a mock is a second implementation of somebody else's system,
and it will drift. Drift produces false confidence — green against your mock's idea of the API, then
surprised by the real one. Behavioral fidelity gets validated against the real thing, in a real
environment. When tempted to add state to a test double, ask: *am I validating our connection code,
or re-implementing their system?*

### 2.5 "Isn't this just mutation testing?"

It is the first question this part attracts, and it deserves a straight answer: **partly, and the
part it doesn't cover is the dangerous part.**

Mutation testing alters your source — flips a conditional, moves a boundary, deletes a statement —
and checks whether any test notices. A surviving mutant is code whose behavior can change with the
suite still green. That is §2.3's habit — *make every new assertion fail once, on purpose* —
mechanized and applied to every assertion continuously instead of when someone remembers. By §1.2's
logic that makes it a gate rather than vigilance, which is a straight upgrade. **Adopt it.**

Against the five shapes it splits cleanly, and the split is the useful part:

| Shape | Mutation testing |
|---|---|
| 1. Test asserts the defect is correct | **Confident false green** — mutate the fallback, the test goes red, mutant killed |
| 2. Assertion weaker than its name | **Catches it** — mutate to another wrong value, the lazy assertion keeps passing |
| 3. Fixture cannot express the failure | **Signals, misdiagnoses** — the mutant survives, but the fix looks like "add a test" |
| 4. Pass condition satisfied by the bug | **Catches it** — mutate the feature away, the check stays green |
| 5. Predicate matches a shape something else has | **Confident false green** — every mutant dies, score is clean |

Shapes 2 and 4 are its home ground; it finds those more reliably than a reviewer will. Shape 3 it
surfaces but points the wrong way, and a stubborn survivor in a hard-to-test region tends to get
waived as an equivalent mutant.

Shapes 1 and 5 are where it fails, and it fails *confidently* — producing a better score than a
weaker suite would. The reason is structural:

> **Mutation testing asks whether your tests are sensitive to changes in the code you wrote. It
> cannot ask whether the code you wrote was the right code.** Where the tests faithfully encode a
> misunderstood requirement, every mutant dies. A high score on a wrong specification is confident,
> well-measured, and wrong.

That is also why it is not a substitute for deriving tests from a specification. It runs after the
implementation exists and takes it as the reference, so it can automate the *sensitivity* guarantee
but never the *specification* one. Spec-driven development has the mirror weakness — a gap in the
requirements is invisible to tests and implementation alike, because both were derived from it.
Neither technique rescues the other; they fail in opposite directions, which is exactly why you run
both.

Treat surviving mutants as a review queue rather than a number to raise. The triage is the value.
Layer placement, thresholds and blind spots for this and every other validation type are laid out in
`templates/validation-strategy.md`.

---

## Part 3 — Working agreements with the AI

These are the "rules" people ask for. Each is a durable instruction, phrased so it survives being
pasted into a `CLAUDE.md` / `AGENTS.md` / system prompt.

### 3.1 Only touch what you own

> Define the boundary explicitly and mechanically: these repos, these resources, this tag. Anything
> outside it is **read-only**. Investigate freely, raise evidence-cited issues, never fix it
> yourself.

This matters more with AI than without, because nothing technical stops a cross-boundary edit — the
model has your credentials and no instinct about org politics. Make the boundary a tag or a naming
convention so it is checkable, and check before every create/modify/**delete**.

This is also what makes your findings welcome rather than threatening: a team that knows you never
touch their code reads your issues as help.

### 3.2 Grant autonomy explicitly, and scope it

Vague autonomy produces either paralysis or overreach. State it plainly, e.g.:

> "Within [boundary], decide what to fix and ship it — root-cause, failing test first, fix, sweep,
> merge, deploy, report. Don't stop to ask permission and don't stop at the diagnosis. Outside
> [boundary], raise it."

Then **name what the grant does not cover**: anything published externally under your name, anything
requiring privileges the AI shouldn't hold, destructive operations on shared state. And require a
report either way — **autonomy is about not blocking, not about going quiet.**

### 3.3 Verify before asserting

> Never claim a defect, or recommend a fix, from an eyeballed screenshot, a memory, or a document's
> claim. Confirm against ground truth — API, logs, code, a live probe — first.

A real one: a run summary was read off a screenshot as ending *before* it started. The owner
approved a fix on that reading. The actual value was fine — a `2` misread as a `1` at low
resolution — and a full spec/TDD/deploy cycle was burned on a phantom. Low-resolution digits are
ambiguous; a thirty-second ground-truth check beats a phantom-bug chase.

If ground truth isn't reachable, say *"it looks like X — let me confirm before we act"* rather than
recommending anything.

### 3.4 Verify provenance before trusting output

The failure mode isn't "the output was wrong." It's **"the output was plausible, well-formed, from
the wrong source, and I acted on it."**

- **Subagent reports:** re-run the decisive check yourself. Never ship on an agent's "440 tests
  green."
- **Anything served or deployed:** confirm the *identity* of what's live — bundle hash, `/version`,
  process start time — before concluding anything from what you see. Several rounds were once spent
  analyzing a week-old bundle and nearly reported as current behavior.
- **"Complete" from an async or multi-worker source:** match on identity (run ID, epoch, endpoint
  set), not on "it stopped saying running." A stale worker looks identical to a finished one.
- **Two sources agreeing is not corroboration when they share a scope.** Two independent counters
  agreed *exactly* — and were both per-segment, on a run that had crossed a segment boundary. They
  agreed because they shared a blind spot. The real number was double. **Ask what scope a figure
  describes before treating a second figure as confirming it.**
- **Parallel agents that might touch the same files:** isolate them (git worktrees) or serialize
  them. "They won't overlap" is not a substitute for isolation; two agents once clobbered each
  other's tests after exactly that reasoning.

**The tell, every time: an artifact that is plausible and suspiciously fast.** Check provenance
before you believe it.

### 3.5 Fix the class, not the reported instance

When a defect is reported, fix the category — then **grep for every other consumer of the same
source** and fix or explicitly clear them.

Three times this bit, in three different domains:
- A display bound to a stale default was fixed in the component that was pointed at; a second
  component reading the same source shipped the same wrong value for two more days.
- Two deployment pipelines resolved their target with `[?starts_with(name,'app-')] | [0]`. A new
  resource sharing that prefix made the deploy target a coin flip. The sweep found **eight more call
  sites** — including a teardown script that would have deleted the wrong environment, printed
  success, and exited 0.
- A clock-skew guard was added in one place with a comment explaining exactly why. The same
  subtraction existed elsewhere, unguarded, and rendered a negative axis label in production hours
  later. **A guard is a class of fix — its rationale is the search query.**

Three derived rules:

- **Check both directions of every guard.** A naming guard that stops the lab touching production
  does nothing about production resolving the lab.
- **Kill `| [0]`.** Prefer "exactly one match or throw," so a future sibling is a loud failure rather
  than a silent change of target.
- **"X can no longer happen" is a claim about the whole repository, and the grep that proves it is
  part of the work.** Consolidating N copies into one definition is only a class fix if you go find
  N. If you won't run the sweep, downgrade the claim to what you actually did.

Also: when accepting a known degradation, **describe what the user will SEE**, not just what the
state is. "The non-owner reports no progress object" was recorded as a minor degradation; in
practice it rendered as a progress bar strobing on and off once per second — worse than no bar at
all. If you can't picture it, that's the signal to go look.

### 3.6 Instrument, don't speculate

When a UI symptom resists code-reading, **instrument the live browser and have the user reproduce
it.** Code-reading produced two confident, wrong hypotheses; one passive recorder settled it in a
single run — and simultaneously disproved an apparent "request storm" that turned out to be a
logging artifact.

Inject something *passive* (an observer parked on a global), never a monkey-patch that mutates app
state. Prefer the platform's own performance entries over a tool's timestamps — batched calls read
as a storm in one and as normal cadence in the other. And if the finding is durable, graduate the
probe into a committed script.

### 3.7 Scope precision in claims that leave the room

**A factually-true claim phrased too broadly reads as false.** "Production has no X deployment at
all" was anchored in a true finding (no X *API* in production) but scoped so broadly it implied the
platform didn't exist — and a senior stakeholder pushed back on exactly that line, spending
credibility the correct underlying finding needed.

- Name the **subject** of every negative claim: "the X API has no production deployment," never
  "production has no X."
- Before any executive review, sweep for **no / none / never / at all** and check each one's scope.
- Pair the gap with what *does* exist. Precision and crediting real progress are the same edit.

### 3.8 Review before publishing anything external

Committing a draft to your own repo is fine autonomously — that *is* the review surface. The
boundary is **external visibility**: a wiki page, a document sent onward, anything carrying
"reviewed by <a human's name>."

**An attribution line asserting review is a precondition, not a caption.** Draft → present → publish
on explicit approval. "Add it to the wiki today" authorizes the *plan*, not skipping the review.

Two mechanical details worth pre-agreeing: mirrors of already-reviewed repo docs can have standing
re-sync authorization (the commit was the reviewed artifact); and the attribution must name the
model **actually running**, not the one copied forward from the last document.

### 3.9 Report honestly

If tests fail, say so and show the output. If a step was skipped, say that. When something is done
and verified, say it plainly without hedging. Do not narrate progress that hasn't happened, and
never present in-flight work as finished.

---

## Part 4 — The operating loop

### 4.1 Cadence

Organize work as **Phase → Epic → Day**: a phase is a theme with an exit gate; an epic is sized to
roughly one working day. Every epic runs the same loop:

> **brainstorm → written spec → written plan → build → review → ship → end-of-day handoff**

**Plan first, build second.** The AI writes the brainstorm, spec, and plan as files; the human
approves each before build starts. This is where AI-assisted development earns its speed — errors
caught at plan time cost minutes, and the same errors caught after a build cost a day. The single
highest-leverage habit in this whole guide is *reading the plan carefully*.

**One roadmap document per phase** is the source of truth, with a status tracker updated at every
close. When a decision gets made mid-phase about future work, write it into the roadmap under that
epic **immediately** — otherwise the future session rediscovers it from scratch.

### 4.2 Session kickoff

**The unit is the session, not the calendar day.** Sessions span days, other people ship while you
are not running, and a session that has been idle for three days needs this *more* than one that ran
yesterday, not less. Start every session by:

1. **Reading the shared coordination record** — if more than one agent operates on shared material.
   Another channel may have deleted a branch, moved a tree, changed a rule you follow, or left
   something addressed to you. Do this *before* pulling, so the pull confirms what you were told
   rather than surprising you with it.
2. **Pulling every repo** — yours and the ones you only read. Including the repo that holds your
   working agreements, which is the one nobody thinks to pull because it isn't the code.
3. **Re-probing the deployed state.** Never trust yesterday's notes about today's production. Ask
   the live system what version it is.
4. **Running the standing canaries** — the small automated proof that the critical path still works.
5. **Reporting drift somewhere durable** before starting the day's work — on the shared record, not
   only to whoever happens to be in the room. If nothing changed, say so explicitly: a kickoff that
   found nothing and a kickoff nobody ran are otherwise identical silence.

Formalize this as a committed runbook with a state file for day-over-day deltas, so it's mechanical
rather than remembered.

**A record you read once a day is a snapshot of something that changes hourly.** Five comments landed
on a shared coordination record within 33 minutes of one channel's daily read, one of them addressed
to that channel. The read establishes what changed overnight; it says nothing about what is true when
you act four hours later. **Re-read before acting on anything cross-channel, and again at close.**

**And a synchronized check across actors on their own clocks needs a stated window.** A "did everyone
check in today?" test, evaluated at one instant, marked a channel absent that had checked in 27
minutes earlier — the check measured arrival order and reported it as compliance. Absence at an
instant is not absence in a day. Name the cutoff, or you publish your own timing as someone else's
behaviour.

**But a check-in record shows when someone last ran — never whether they should have.** The
temptation once the record exists is to read a stale entry as a lapse. Resist it: in most multi-agent
setups, presence is only owed by an agent something invoked, nobody works every day, and a secondary
agent's resting state is idle for days at a stretch while a primary runs several times in one. **A
stated window does not fix this** — it is the same assumption wearing a timestamp, since a window only
detects a miss if presence was owed in it. Read the field as *last time this ran*, draw no inference
from its age, and get liveness signals from whoever owns the work rather than from silence.

### 4.3 End-of-day handoff

One file per working session: what shipped (with versions and run IDs), what's open, and **exact resume
pointers**. The next session starts by reading it. This is what makes multi-day AI work coherent —
context windows end, and the handoff is what survives.

**Push before you write it.** A handoff describing work that exists in one working directory is a
promise, not a record. A crash recovery on one machine restored a snapshot that turned out to be a
day old rather than minutes old, and took more than a day of work with it — work the handoff
faithfully described and could not restore. Un-pushed work is not work, and an AI-assisted day
produces enough of it to make that expensive.

**A handoff that names someone else's action has to reach them.** One file per session is the right shape
for the next session in *your* seat — but a file is not a channel. If the handoff says "waiting on X"
and X is another agent or another person, that item belongs where **they** look, with the file
carrying a pointer to it. Otherwise the handoff is a diary: accurate, durable, and read by nobody who
could act on it. Symmetrically, whatever you wrote to shared state today — a branch you deleted, a
tree you moved, a rule you changed — goes there too, because the next channel's kickoff (§4.2, step
one) is reading that record and not your file.

**And the record is what propagates a decision — not the human everyone reports to.** Once several
agents run under one owner, the tempting shortcut is for that person to paste each decision into the
others. It fails three ways. **It goes stale in transit:** a relayed verification instruction carried
a file's line count that had already changed, and would have produced a false failure on the machine
that received it. **It loses its qualifiers:** a briefing that prescribed a path *with an escape
hatch* arrived without the hatch, so a prescription became an assertion about an environment nobody
had checked — and two of its claims were wrong. **And it hides readership:** routing through a person
tells you a message was sent, never who read it, so an agent that disagreed is indistinguishable from
one that never saw it. Put the change in the record and in the commit, and let each agent collect it
when it next runs. **Keep the exception explicit** — if something genuinely cannot wait for another
agent's next start, say so and say why, and let a human decide to interrupt. Naming it as an
escalation is what keeps escalation rare enough to mean something.

> **Then give the record a catch-up read.** If sessions can be idle for days, the returning one
> should not have to reconstruct a week from a long thread. A short dated log — one line per decision
> with the commit that carries it, newest first — costs a line per change and turns "what did I
> miss?" into a bounded read. Keep it a summary that points at commits rather than a second copy of
> the reasoning, or it becomes one more thing that goes stale.

### 4.4 Memory discipline

If your tool has persistent memory, curate it:

- Durable **preferences and corrections** become memory files immediately, each with **why** and
  **how to apply** — the "why" is what makes the rule survive a situation its author didn't foresee.
- **Project state lives in files in the repo** (roadmap, handoffs), not only in memory.
- Memory is **curated, not append-only**: update, merge, delete. A wrong memory is worse than no
  memory, because it's confidently retrieved.
- **Memories are point-in-time observations.** Anything citing a file, function, or flag needs
  re-verification before you act on it.
- **A memory store shared by two agents cannot hold the words "this channel".** Memory is usually
  keyed to the working directory, so two sessions operating on one workspace share one store — and
  every first-person memory in it is false for whichever session reads it second, while being loaded
  into that session's context automatically and silently. Worse, a memory store has no history, no
  recorded author, and no merge step where a collision could surface: the second writer wins and the
  first version is simply gone. The only signal that exists is a tool refusing a stale write, and
  that fires only when the same session happened to read the file earlier — a fresh session writes
  straight over. **The collision rules that apply to code apply harder here, not more loosely**,
  because none of version control's safety net is underneath it. Give each agent its own working
  directory and it gets its own store; separating shared facts from identity facts by convention is
  the fallback, and it is remembered rather than enforced.
- **A memory that travels must not carry a copy of state that lives somewhere else.** Paths are the
  obvious case: one that is right on one machine is wrong on every other, so "correcting" it only
  moves which machine is broken. An *enumeration* fails identically and is easier to miss — a memory
  listing the files a shared directory holds was accurate for exactly one day, until the directory
  gained one. Both fail silently rather than visibly, and a seeded memory is precisely the artifact a
  session trusts *instead of* going to look. Store the identity that travels — a repository URL, a
  package name — and let the reader resolve the rest. Keep only what the source cannot convey: that
  two runbooks run in a given order is real information; which files exist is not.

### 4.5 Full-stack restart per run

When starting a local end-to-end run, **stop and start every service as one unit** — never reuse
processes from the previous run. Interdependent in-memory state silently interferes: a stale
runtime credential once survived an environment switch and produced auth failures that looked
exactly like an expired secret, costing hours of investigation on the wrong hypothesis.

### 4.6 Version and tag everything

Trunk-based development: short-lived branches, merge fast, release from trunk.

- A version carried in the app and **surfaced at runtime** (`/version`, a health field, a UI
  footer), so a live instance can tell you exactly which build it is.
- The CD pipeline **auto-tags** the released commit on successful promotion, so every deploy maps
  to an immutable tag. Never a manual step.

This is what makes "which version introduced this?" and "roll back to the last good one" into
ten-second operations.

### 4.7 Keep the README current

If the README is shared, treat it as a standing per-epic deliverable, and **write for a generic
reader** — not for the person who owns the project. Avoid anything that assumes the reader's
identity or privileges; use neutral placeholders in examples. Give commands in both shells **only
where the syntax genuinely differs**; two identical blocks are noise.

---

## Part 5 — Agents, subagents, and models

This is the concrete answer to *"what agents do you use?"*

### 5.1 Default to subagent-driven execution

For any implementation plan or multi-task work — especially under an autonomy grant — dispatch
subagents rather than executing inline. Inline is the exception.

**Why:** subagents protect the main context window (which is the scarce resource in a long session)
and give **fresh reasoning per task**, uncontaminated by the conversation that produced the plan.
When in doubt, use subagents. Genuinely trivial work — a single file, a single edit, no tests — is
still fine inline.

### 5.2 Role separation is the point

The value isn't parallelism, it's **independence**:

| Role | Mandate | Critical constraint |
|---|---|---|
| **Planner** | Turn the spec into ordered, verifiable tasks | Writes files, not code |
| **Test-writer** | Write failing tests **from the spec** | Must not read the implementation |
| **Implementer** | Make the tests pass | **Never edits tests** |
| **Reviewer** | Find bugs and spec violations | Fresh context; instructed to *find problems*, not summarize |

That test-writer constraint is doing enormous work. A test written from the implementation asserts
what the code does; a test written from the spec asserts what the code *should* do. Only the second
one can fail usefully.

The reviewer's instruction matters just as much: an agent asked to "review this" produces a
flattering summary. An agent asked to "find the bugs, find where this violates the spec, assume
something is wrong" produces findings.

> You do not have to build this yourself. The **Superpowers** skill library (§6.4) ships this exact
> role separation — `subagent-driven-development`, `requesting-code-review`, `using-git-worktrees` —
> and triggers it automatically. Install it before you hand-roll an agent workflow.

### 5.3 Two review passes find different bugs

Run **spec compliance** and **code quality** as separate passes. They genuinely surface different
defects. Then run a **holistic pre-ship review** across the whole change — it catches what per-task
reviews structurally cannot, because each per-task reviewer only ever saw its own slice.

**Adversarial review earns its keep.** In one release it caught a fault-handling path that shipped
dead (the code compared against a status string that never appears on disk, so a crashed worker
would have reported healthy forever — the exact scenario the fix existed for), a state flap, and two
vacuous tests. All after a green suite.

### 5.4 Model allocation

Standardize on one strong model for the session and its subagents. **Upscale individual subagents to
your most capable model only when both conditions hold:**

1. The task is genuinely **hard** — stronger reasoning changes the *result*, not just the polish.
2. It is **expensive to get wrong**.

Importance alone isn't enough; a critical-but-mechanical task runs fine on the baseline. Good
candidates for the upscale: adversarial review, deep root-causing of a confusing failure, ambiguous
many-file integration the plan couldn't fully specify. Poor candidates: implementation from a good
plan (a good plan makes coding mechanical), plumbing, docs, mechanical refactors, exploration.

**Announce each upscale with a reason.** Cost should be visible and vetoable.

One boundary worth knowing: you typically control the *subagent* model but not the main session's.
When the hard thinking lands in the main loop, either flag it so a human can switch models, or
delegate that specific reasoning to an upscaled subagent. Don't quietly under-power it.

### 5.5 Keep a proving ground

Maintain a **lab** — a pipeline, environment, or sandbox that is entirely yours to break. Prototype
risky pipeline, permission, and platform changes there *before* they touch the real deploy path.

One such lab paid for itself before its first successful run: six systemic discoveries plus a timing
measurement that rewrote the team's operational rules. Guardrails that make a lab safe: it touches
only prefixed resources, it fails closed, and it refuses to fabricate a measurement.

### 5.6 Human validation gates for UX

For user-facing changes, have the owner review hands-on from a running build **before merge**. This
is a deliberate, scoped exception to no-approvals CD — visual and interaction quality is the one
thing your gate suite genuinely cannot assess.

Related, and easy to miss: **the UI must feel alive.** A frozen frame is indistinguishable from a
crash. Anything derivable locally — above all the clock — should render continuously and never block
on the network. But keep **liveness** and **data freshness** separate: the frame keeps moving while
each datum honestly states its own age. Never fabricate samples to look busy.

### 5.7 Known AI traps to police — including in your own output

- Hallucinated APIs, packages, and flags
- Plausible-but-wrong logic that reads beautifully
- Tutorial-shaped security (parsing a token is not validating it)
- Stale-version patterns from training data
- Test-gaming — weakening an assertion to reach green
- Confident agreement with a wrong premise you supplied
- Aspirational-as-current documentation

### 5.8 Running more than one session

A second session for overflow is genuinely useful — one channel is prime, the other picks up what
prime is too busy for. Two hazards, neither where people look for them.

**Branch discipline buys nothing, because the conflict is on the filesystem rather than in git.** Two
sessions sharing one working directory edit the same files with no lock, no warning, and no merge
step where a conflict could surface. Git protects *history*; it has no opinion about a working tree
two writers hold open. One session's uncommitted edit vanishes under the other's write and neither
observes anything unusual.

- **Separate working directories, not separate branches** — a worktree or a second clone per session.
- If they must share a tree, **only one writes.** The other reads, reports, and hands changes to the
  writer. Read-only is a real role, not a demotion.
- **Ground-truth at every kickoff.** The sibling channel may have shipped while you weren't looking.

**An approval request must name what it will change.** A human running several sessions holds one
piece of context — *which channel am I in* — and will use it to interpret any request that doesn't
supply its own. An approval raised inside one project's session, for a change to a different
repository under different governance, gets read against the wrong target and granted. Nothing about
the request looks wrong; it simply never says where it lands.

> **Name the repository, the branch, and the governance in the request itself**, not in the
> surrounding conversation. The reviewer's channel is not a substitute for the target, and it
> diverges from it exactly when the request is unusual — which is when approval matters most.

---

## Part 6 — Setting up the environment

### 6.1 The four layers

| Layer | Where | What belongs there |
|---|---|---|
| **Global rules** | User-level config (`~/.claude/CLAUDE.md`) | How you work, everywhere. Shell preferences, engineering defaults. |
| **Project rules** | `CLAUDE.md` / `AGENTS.md` in each repo | Architecture, commands, boundaries, conventions for *this* codebase. |
| **Memory** | Tool's per-project memory directory | Durable preferences, corrections, hard-won facts. One fact per file. |
| **Project state** | Files in the repo (roadmap, handoffs, specs) | Anything the team needs, or that must survive the tool. |

The layering rule: **if it would matter to a new human engineer, it goes in the repo.** Memory is for
how *you* want to work, not for how the system works.

### 6.2 Bootstrap a new project

```powershell
# 1. Generate the project rules file from the codebase (in each repo)
/init

# 2. Create the memory directory for this workspace, if your tool uses one
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\projects\<workspace-key>\memory"
```

```bash
# Same, on macOS/Linux
mkdir -p ~/.claude/projects/<workspace-key>/memory
```

Then seed three things before writing any code:

1. **The boundary** — what you own, what is read-only.
2. **The engineering defaults** — Part 1 of this guide, pasted or referenced.
3. **A Definition of Done** — §1.8, as a file in the repo, so the vocabulary is shared.

### 6.3 A starter rules file

A complete, fill-in-the-blanks version ships next to this guide as **`starter-CLAUDE.md`** — copy
that file into your repo rather than retyping the block below. Two things it stresses that are easy
to miss: **keep it short**, because it is loaded into context on every turn and you pay for every
line continuously; and **keep it true**, because a stale rules file is read first and is
confidently wrong.

**Keeping it true means running the commands before you write them down.** One rules file
documented three ways to run a single test. All three failed: one named a test file for a feature
that had never been built, one a test name that had never existed, and one a script no package
declared. The broken lines were the copy-pasteable ones, so they were the first thing every new
session ran — the file's most-used content was its least-true, and each session burned time
rediscovering that before it could start. Re-check them whenever the tree moves underneath them.
A rules file is the one document that gets read *before* anything is in a position to correct it.

The shape:

```markdown
## Ownership boundary
We change only: [repos, resource groups, tags]. Everything else is READ-ONLY —
investigate and raise evidence-cited issues, never fix it directly.

## Engineering non-negotiables
- TDD: failing test first; the implementer never edits tests; every defect found
  anywhere becomes a regression test before its fix merges.
- CI runs everything on every push. A red gate means no ship, and is a signal to
  strengthen tests — never to retry until green.
- Every proof becomes a re-runnable script/test/gate. No one-off validations.
- Infrastructure is code, deployed only by pipeline. No console changes.
- Secretless: federated identity for pipelines, managed identity + vault at
  runtime, secret-scanning gate. Never run scripts under an owner/admin identity.
- Latest stable runtimes and libraries. EOL warnings are action items.

## Definition of Done
Complete = runs in the named environment · enforced with no bypass · tested
including failure cases with retained results · reproducible from source ·
observable and documented. Every "Complete" carries its evidence.
"In QA" is not done.

## Working agreements
- Verify before asserting: never claim a defect from a screenshot or from memory —
  confirm against the API, the logs, or the code first.
- Verify provenance: confirm an output came from the source you think it did.
  Re-run a subagent's decisive check yourself.
- Fix the class, not the instance: after any fix, grep for every other consumer
  of the same source before calling it done.
- Default to subagent-driven execution. Test-writer works from the spec; the
  implementer never edits tests; the reviewer is told to find problems.
- Plan first: brainstorm → spec → plan → build → review → ship. The human
  approves the plan before the build starts.
- Nothing leaves this repo under my name without my actual review.
- Report honestly: if tests fail, show the output; if a step was skipped, say so.

## Environment
- Shell: [PowerShell / bash]. Give me commands in the form I actually run.
- Commands: [build] [test] [run] [deploy]
```

### 6.4 Skills and plugins

A **skill** packages a repeatable procedure so the AI runs *your* process instead of improvising
one. Skills are how the principles in this guide stop being advice and start being behavior — they
fire automatically when their trigger matches, before the model has a chance to freelance.

#### Don't start from scratch — install Superpowers

Most of the process discipline in Parts 1, 4 and 5 already exists as a maintained skill library:

> **Superpowers** — <https://github.com/obra/superpowers>
> by Jesse Vincent (obra) · marketplace page: <https://claude.com/plugins/superpowers>

```powershell
/plugin install superpowers@claude-plugins-official
```

The skills trigger on their own, so there is nothing to remember once it is installed. It is the
single highest-leverage thing you can do on day one — you inherit a worked implementation of
brainstorm → spec → plan → TDD → subagent execution → review instead of writing it yourself.

**What it ships, mapped to this guide** (v5.0.7 at time of writing — check the repo for the current
set):

| Skill | Enforces |
|---|---|
| `test-driven-development` | §1.1 — red/green TDD, before any implementation code |
| `systematic-debugging` | §3.3, New defect drill — root-cause *before* proposing a fix |
| `verification-before-completion` | §1.8, §3.9 — run the command and read the output before claiming anything passes. "Evidence before assertions, always" |
| `brainstorming` | §4.1 — required before creative work; draws out intent and requirements first |
| `writing-plans` | §4.1 — spec → written plan, before touching code |
| `executing-plans` | §4.1 — run a written plan in a fresh session with review checkpoints |
| `subagent-driven-development` | §5.1, §5.2 — role-separated execution of a plan's independent tasks |
| `dispatching-parallel-agents` | §5.1 — fan out 2+ genuinely independent tasks |
| `using-git-worktrees` | §3.4 — isolate parallel agents so they can't clobber each other |
| `requesting-code-review` | §5.3 — review before merge; ships a `code-reviewer` agent |
| `receiving-code-review` | §5.7 — demands technical rigor on feedback rather than performative agreement. This is the direct antidote to the *confident agreement* trap |
| `finishing-a-development-branch` | §4.6 — structured merge / PR / cleanup at completion |
| `writing-skills` | Authoring and testing your own (below) |
| `using-superpowers` | The bootstrap — teaches the agent to look for a skill before responding |

It also adds `/brainstorm`, `/write-plan`, and `/execute-plan` as slash commands.

Two things worth knowing before you adopt it in an enterprise setting: it is a **community project**
distributed through the official marketplace, not an Anthropic-authored artifact — read the skills
before you trust them, which is cheap since they're plain markdown in a public repo. And it is
**opinionated by design**: it will stop your agent from jumping straight to code. That is the point,
and it is also the thing people try to turn off in week one. Don't.

#### Then write your own

Superpowers covers the general engineering workflow. It cannot know your release checklist, your
incident drill, your document conventions, or your review rubric. **Write a skill for anything
you've explained twice.**

The shape that works: **a trigger description precise enough that it fires on the right tasks and
not the wrong ones**, plus steps written as instructions rather than prose. Keep them in version
control alongside the code they describe, and use `writing-skills` to author and test them — a skill
that never triggers is indistinguishable from one that doesn't exist.

---

## Part 7 — The catalogue of ways tooling lies to you

Every one of these cost real hours. They are not exotic — they are the *normal* texture of
automating against real platforms, and the generalized rule matters more than the specific case.

**A tool that returns empty instead of erroring turns a loop blind.**
A `python` invocation that was actually a Windows Store alias stub printed an install hint and
produced *no output on the pipe*. The polling loop parsing through it never saw its terminal
condition, ran to its full timeout, and **exited 0** — 45 minutes of blindness that looked like a
successful watch. The build had in fact succeeded 25 minutes in.
→ **Any watcher must fail loudly when its parse yields nothing.** Silence and not-yet are different
states. And note that checking the binary "exists" proves nothing — the alias stub is found too.

**List APIs sort in ways that hide exactly what you're looking for.**
A builds API defaulted to ordering by *finish time*, so in-progress runs — which have no finish time
— sorted last. A "watch for the new build" loop polling the first row could never see a running
build. This manufactured two separate false "the trigger didn't fire" alarms, one of which caused a
duplicate deploy.
→ **Poll by ID, which is ordering-independent.** Use ordered queries only for the initial discovery
of the ID. Knowing about the trap isn't enough; bake the fix into every monitoring script.

**Control-plane operations are asynchronous, and often invisible.**
A restart command returned in ~10 seconds; the actual worker recycle landed **3.5–5 minutes later**,
and in all seven measured restarts produced *no observable downtime at all* from outside.
→ **Never infer that an operation completed from availability.** Use a monotonic identity signal —
process start time, build ID, an epoch field on a health endpoint. "It's up" tells you nothing about
*which* instance is up.

**A proof scoped to staging can recycle production.**
A failover proof scale-bounced the *shared* hosting plan — which hosted both the staging slot and
production. A gate that read as staging-only forced every production workload to fail over
mid-deploy. It went unnoticed for months because production was usually idle at deploy time.
→ **Trace what your gates actually touch, not what they're named after.** And put a `try/finally`
around every cleanup loop; the same incident leaked two live workloads because a mid-loop throw
abandoned the rest.

**A library's reported URL is not the interface it bound.**
A mock server reported `.Url = http://localhost:{port}` while actually binding `0.0.0.0` and `::` —
tripping a firewall consent dialog on every new test binary path.
→ **Read the OS listener table, not the library's own string.** Pin test listeners to loopback and
port 0 explicitly; it also kills port-collision flakes.

**Your machine's clock can be wrong, and everything local inherits the error.**
A workstation clock ran ~1 day 15 hours behind. `Get-Date`, every git commit timestamp, and every
file mtime shared that one bad clock — so they could only ever agree with *each other*, and
date-based reasoning about the repository went briefly incoherent (commits appeared older than the
files depending on them).
→ **Cross-check any date you write into an artifact against a non-local source** — an HTTP `Date`
header is authoritative and takes one command. The tell that something is wrong: files or filenames
dated in the future.

**Never relay binary content through a model-mediated parameter.**
Uploading a 30 KB document through a tool that took base64 content *as an inline parameter* produced
a corrupt file both times — the model transcribes every byte, and loses roughly 0.25% of characters.
Error positions differed between attempts, so no amount of retrying converges.
→ **Binaries move by file path or by the user.** Always verify size or hash against the local file
before declaring an upload done.

**A process's environment is a snapshot, not a view of the system.**
A session captured `PATH` at startup. A tool installed *during* that session was genuinely present on
disk and genuinely absent from the environment the session could see, so every check reported "not
installed" — correctly, about a stale copy of the world. Three separate diagnostics agreed, which
read as convergent evidence; all three consulted the same inherited environment, so they shared one
blind spot.
→ **Ask the source, not the inherited copy** — the package manager or the filesystem, never `PATH`.
And **agreement between checks is evidence only if the checks are independent.** Three signals
derived from one snapshot are one signal reported three times.

**A test framework that isn't installed on the CI agent gates nothing.**
A whole category of script tests sat in the repo, ran locally, and were silently absent from CI.
→ **Confirm each test type actually executes in the pipeline** — find its output in the build log,
by name.

**A gate that cannot be run locally is half a gate.**
A formatting check passed on the Linux runner and failed on all 94 files on every Windows machine:
Git for Windows checks files out as CRLF and the formatter defaulted to LF. Nobody could satisfy
the gate before pushing, so formatting breaks were discoverable only after they were already in the
history. It is the mirror image of the entry above, and harder to spot, because the gate *was*
running — just nowhere a developer could reach it.
→ **Confirm every gate runs on the platforms developers actually use, not only on the runner.**
Cross-platform defaults are the usual culprit: line endings, path separators, case sensitivity,
locale.

The through-line for all of them: **confirm the mechanism, don't infer it from a symptom.** Every
one of these was a case where a plausible signal (exit 0, an empty list, "the site responds", a
green build) was accepted as proof of a mechanism nobody had actually checked.

---

## Appendix — Working checklists

### New defect drill
1. Reproduce it and capture the exact signature verbatim.
2. Write the **failing test first**. Watch it fail for the right reason.
3. Root-cause it — no fix before the mechanism is understood.
4. Fix.
5. **Sweep**: grep for every other consumer of the same source, pattern, or quantity.
6. Ask whether an existing test should have caught it. If one *passed* against this bug, that test
   is also a defect.
7. Merge, ship, report — including what the sweep found.

### Pre-ship review
- [ ] Does every new test's assertion match its name?
- [ ] Would any new check still pass if the feature were entirely absent?
- [ ] Has each gate in this path actually *run* recently, or only not-failed?
- [ ] Is there a state transition here that only a live run crosses? Is it covered?
- [ ] Did any existing test have to change? Which of the two was wrong?
- [ ] For every "X can no longer happen" in the spec or commit message — was the sweep run?
- [ ] For any accepted degradation — what will the user actually *see*?

### Definition of Done
- [ ] Runs in the named environment
- [ ] Enforced with no bypass path
- [ ] Tested including failure cases, with retained results
- [ ] Reproducible from source control
- [ ] Observable and documented
- [ ] Evidence attached — the command, link, or run ID that proves it

### Session kickoff
- [ ] Read the shared coordination record — before pulling, if others work on this material
- [ ] Pull every repo, including read-only ones
- [ ] Re-probe deployed versions from the live systems
- [ ] Run the standing canaries
- [ ] Report drift — or state explicitly that nothing changed

---

## If you only take four things

1. **Put the safety in the pipeline, not in a person.** Mechanical gates are what let you accept
   AI-speed output without accepting AI-speed incidents.
2. **Read the tests adversarially.** A green suite is weak evidence, and the specific way it lies to
   you is Part 2. Make every new assertion fail once, on purpose.
3. **Separate who writes the tests from who writes the code** — even when both are the same model.
   Plan-supplied tests give no independent signal.
4. **Verify, don't infer.** Ground truth over screenshots, provenance over plausibility, mechanism
   over symptom. The most expensive mistakes in this guide all started with a plausible signal
   nobody checked.

---

*Created by Claude Opus 5, reviewed by Torsten Kablitz · 2026-08-03. Distilled from years of DevOps
and TDD practice across production systems; no proprietary or client-identifying material.*

*Home: <https://github.com/tkablitz/ai_cto>*

*© 2026 Torsten Kablitz. Licensed under
[CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/), plus an explicit grant for internal
use inside your own organization — commercial or not. Read it, share it, build your own standards
and training from it. Reselling it, bundling it into a paid product, or delivering paid services
whose substance is this material needs a separate license. The `starter-CLAUDE.md` and `templates/`
files are MIT, so you can paste those into your repositories with no strings. Full terms in
`LICENSE`.*

*Shipped alongside this guide: `README.md` (start here), `starter-CLAUDE.md` (drop-in project rules
file), and `templates/` — the scaffolds referenced throughout, listed with their purposes in
`README.md`.*
