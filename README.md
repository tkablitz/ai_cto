# AI-Assisted Development — starter package

Everything you need to run an AI coding assistant on real software without it quietly wrecking
things. One guide, a drop-in config file, five templates.

The foundations are **DevOps and TDD**. Every other practice here exists to keep those two honest
when a machine is producing code faster than anyone can read it.

---

## Start here (about 30 minutes)

**1 — Read the guide.** `AI-DEV-BEST-PRACTICES.md`. If you only have ten minutes, read Part 0, then
Part 2, then the four-item summary at the end. **Part 2 is the one that will save you the most
pain** — it's about why a green test suite is much weaker evidence than it feels like.

**2 — Install a skill library.** Skills make the AI follow a process instead of improvising one, and
they fire automatically. Don't build this yourself:

```powershell
/plugin install superpowers@claude-plugins-official
```

Source and docs: <https://github.com/obra/superpowers>. Guide §6.4 explains what's in it and maps
each skill to the practice it enforces.

**3 — Drop in the config.** Copy `starter-CLAUDE.md` into your repo as `CLAUDE.md` (or `AGENTS.md`,
depending on your tool), then fill in the bracketed parts — your build and test commands, your
ownership boundary. Instructions are in the file's header.

Or have your assistant do it. Open it in your repo and paste:

> Read `AI-DEV-BEST-PRACTICES.md` in full. Then draft a `CLAUDE.md` for this repository using
> `starter-CLAUDE.md` as the base, filled in with this codebase's actual build, test, and deploy
> commands. Show me the draft before writing the file.

**4 — Adopt a Definition of Done.** `templates/definition-of-done.md`, into your repo or wiki. This
is the cheapest high-value step in the whole package: it gives your team and your AI one shared
meaning for the word "complete," which is what stops status reports from drifting into fiction.

**5 — Only if you're leading the work:** set up a phase roadmap (`templates/phase-roadmap.md`) and,
if you integrate against systems you don't own, an issues log (`templates/issues-log.md`). Skip this
if you're here to write code — the first four steps are the whole job.

---

## What's in here

| File | What it's for |
|---|---|
| `AI-DEV-BEST-PRACTICES.md` | The guide. Read it. |
| `LICENSE` | CC BY-NC 4.0 for the guide, MIT for the templates — see below |
| `starter-CLAUDE.md` | Drop-in project rules file — rename to `CLAUDE.md` / `AGENTS.md` |
| `CLAUDE.md` | Rules for maintaining *this* repo. **Not** the file you copy — use `starter-CLAUDE.md`. Kept here as a worked example of §6.1 |
| `templates/definition-of-done.md` | Shared vocabulary for "complete" (guide §1.8) |
| `templates/eod-handoff.md` | End-of-day handoff, so work survives a context window (§4.3) |
| `templates/phase-roadmap.md` | Phase/epic planning doc, one per phase (§4.1) |
| `templates/issues-log.md` | Findings log for systems you don't own (§3.1) |
| `templates/architecture-review.md` | Structured review of someone else's design proposal |
| `templates/validation-strategy.md` | Which validation answers which question, and where each is blind (§1.1, §2.5) |

Every template uses `<angle-bracket placeholders>`. Fill them in and delete what you don't need.

---

## This is process, not a product

**Don't clone or fork this repo as the base for your own project.** There is nothing here to build
on — no application, no build, no CI config, no source tree. Cloning it gets you a git history and
a `CLAUDE.md` that belong to *this* repository rather than yours, and you'd spend your first hour
deleting things.

Start your project the normal way, with whatever scaffolding tool fits it. Then bring in only the
pieces you have a use for:

| Bring in | When |
|---|---|
| `starter-CLAUDE.md` → your `CLAUDE.md` | Day one, always. Highest-leverage file here |
| `templates/definition-of-done.md` | Day one. Cheapest way to stop "done" from drifting |
| `templates/eod-handoff.md` | Once work starts spanning more than a day |
| `templates/phase-roadmap.md`, `templates/issues-log.md` | Only if you're leading the work |
| `templates/architecture-review.md` | When someone hands you a design to review |
| `templates/validation-strategy.md` | Once the suite is big enough that nobody can tell by reading whether it bites |

**The guide is read, not copied.** Link to it from your repo if it helps; don't vendor it. The
licensing follows the same line, and it matters in a commercial setting: `starter-CLAUDE.md` and
everything in `templates/` are MIT, so they can live in a company codebase with no obligations,
while the guide is CC BY-NC. Copy the scaffolds freely — leave the guide here.

Take the rest when you hit the need for it, not before. That is §1.11 applied to this package.

**Telling a stale copy from a current one.** Once a scaffold is in your repo it stops receiving
updates — nothing here can reach it, and a copy that silently diverges is the problem this package
spends a whole section on. So each copyable file carries a `rev` date in the licence notice at the
end:

```html
<!-- SPDX-License-Identifier: MIT · rev 2026-08-15 · © 2026 Torsten Kablitz · … -->
```

Compare your copy's `rev` against the one here to see whether yours is behind, then diff if it is.
The date moves only when the file's substance changes, and a pre-push hook in this repo refuses a
change that edits a scaffold without moving its `rev` — because a version stamp maintained by hand
is exactly the sort of hand-maintained copy of state that §4.4 tells you not to trust.

---

## Read this before you adopt §1.2

The guide argues for **continuous deployment with no manual approval gates** — merge to trunk, green
pipeline, it ships. That is correct **when you own the blast radius**: internal tools, your own
services, systems whose failure costs you rather than a customer.

If you're shipping something with real money, safety, or regulatory exposure, keep the gates
automated and add whatever promotion control your actual risk justifies. The principle that holds
either way:

> **The gate is mechanical. The judgment goes into the gate, not into the moment of shipping.**

It's the one section a skim-reader can misapply badly. Everything else in the guide is safe to adopt
as written.

---

## License

Dual-licensed, so the parts you *read* and the parts you *paste* have different terms.

- **The guide** (`AI-DEV-BEST-PRACTICES.md` and this README) — **CC BY-NC 4.0**, plus an explicit
  grant for **internal use inside your own organization, commercial or not.** Read it, share it,
  and build your own internal standards and training from it. What you can't do is sell it, bundle
  it into a paid product, or deliver paid consulting or training whose substance is this material.
- **`starter-CLAUDE.md` and everything in `templates/`** — **MIT**, with no strings. These exist to
  be copied into your repo and edited beyond recognition, and a scaffold carrying licensing
  obligations into your codebase is one your legal team will just tell you not to use.

So: use it at work, freely. Teach your team from it. Just don't resell it.

Commercial licenses are available — see `LICENSE` for terms, attribution wording, and contact.

---

## A note on where this came from

Distilled from years of DevOps and TDD practice — the working agreements, code reviews, and
post-mortems behind production systems built this way: versioned releases, gated continuous
deployment, suites running to ~1,800 automated tests, disaster recovery proven by actually tearing a
system down and rebuilding it from source.

The specific numbers and failures in the guide are real. They are also **ordinary** — every one is a
way that any sufficiently complex system can break, which is exactly why they earn their place here
rather than being embarrassing. Process does not prevent failure. It makes failure an expected
result rather than a surprise, and turns an incident into a regression test instead of a bad week.

The systems these happened to are not named, and nothing here is confidential to any client. Use it,
adapt it, pass it to a colleague.

Two suggestions if you do pass it on. Keep Part 7 — the catalogue of ways tooling lies to you — even
though it looks like the most skippable section; it's the part people come back to. And fold your
own post-mortems into it as you accumulate them. A best-practices doc that never grows is a doc
nobody learned anything from.

Found this useful, or disagreed with something hard enough to want to argue about it? Open an issue
at <https://github.com/tkablitz/ai_cto>.
