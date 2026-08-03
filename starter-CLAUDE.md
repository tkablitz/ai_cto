<!--
  HOW TO USE THIS FILE
  1. Copy it into your repository root as CLAUDE.md (Claude Code) or AGENTS.md (most others).
  2. Fill in every <bracketed placeholder>. The Commands section matters most — get it right first.
  3. Delete any section that doesn't apply. A short accurate file beats a long aspirational one.
  4. Delete this comment block.

  KEEP IT SHORT. This file is loaded into context on every single turn, so every line you add is
  a line you pay for continuously. Aim for under ~100 lines. If a rule only matters occasionally,
  it belongs in a skill, not here.

  KEEP IT TRUE. A stale rules file is worse than none — it is confidently wrong and it is read
  first. Re-check it whenever build commands, architecture, or team boundaries change.
-->

# <Project name>

<One or two sentences: what this system does and who uses it.>

## Commands

```bash
<build>              # e.g. dotnet build MySolution.sln  /  npm run build
<test>               # e.g. dotnet test  /  npm test
<test one file>      # how to run a single test file or filter — used constantly
<run locally>        # e.g. npm run dev
<lint / format>
<deploy>             # or: deploys happen via <pipeline>, never by hand
```

<Anything non-obvious about the toolchain: required runtime versions, env vars, a service that
must be running first, a step that only works from a particular directory.>

## Architecture

<A short map. Where the entry points are, what the main layers/modules are, where config and
secrets come from. Enough that a competent stranger knows which file to open — not a full tour.>

## Ownership boundary

We change only: `<repos, directories, resource groups, tags>`.

Everything else is **READ-ONLY** — investigate freely and raise evidence-cited issues, but never
edit it. When in doubt, raise it, don't change it.

<Delete this section if you own everything you can reach.>

## Engineering non-negotiables

- **TDD.** Failing test first. The implementer never edits tests — if a test is wrong, that's a
  separate, deliberate change. Every defect found anywhere becomes a regression test *before* its
  fix merges.
- **Gates, not vigilance.** CI runs everything on every push. A red gate means no ship, and is a
  signal to strengthen tests — never to retry until green.
- **Standing tests over one-offs.** Every proof becomes a re-runnable script, test, or gate. If it
  was demonstrated once by hand, it wasn't demonstrated.
- **Everything as code.** Infrastructure in IaC, deployed only by pipeline. No console/portal
  changes to resources we own. Unavoidable manual steps are documented in-repo as exact commands.
- **Secretless.** Federated identity for pipelines, managed identity + vault at runtime, a local
  secrets store for development. Zero credentials in source.
- **Least privilege.** Never run scripts under an owner/admin identity. Use a scoped service
  identity; if none exists, ask for one rather than borrowing elevated access.
- **Fresh dependencies.** Latest stable runtimes and libraries. EOL warnings are action items.

## Definition of Done

**Complete** = runs in the named environment · enforced with no bypass path · tested including
failure cases with retained results · reproducible from source control · observable and documented.

Every "Complete" carries its evidence — the command, link, or run ID that proves it.
**"In QA" is not done.** Quality ownership doesn't transfer at a gate.

## Working agreements

- **Verify before asserting.** Never claim a defect, or recommend a fix, from a screenshot or from
  memory. Confirm against ground truth — API, logs, code, a live probe — first.
- **Verify provenance.** Confirm an output came from the source you think it did. Re-run a
  subagent's decisive check yourself; never ship on its report alone. Check what's actually
  deployed (version, hash, process start time) before drawing conclusions from it.
- **Fix the class, not the instance.** After any fix, grep for every other consumer of the same
  source or pattern before calling it done. Before writing "X can no longer happen," run the sweep
  that makes it true — or narrow the claim to what was actually verified.
- **Plan first.** brainstorm → written spec → written plan → build → review → ship. I approve the
  plan before the build starts.
- **Default to subagents** for plan execution and multi-task work. The test-writer works from the
  spec and must not read the implementation; the implementer never edits tests; the reviewer is
  told to find problems, not to summarize.
- **Nothing leaves this repo under my name without my actual review.** Committing drafts here is
  fine; publishing externally is not.
- **Report honestly.** If tests fail, show the output. If a step was skipped, say so. Don't present
  in-flight work as finished.

## Environment

- Shell: `<PowerShell / bash / zsh>` — give me commands in the form I actually run.
- OS: `<Windows / macOS / Linux>`
- <Anything else that trips up generated commands: path quirks, proxies, a corporate CA.>

---

*Practices above are condensed from `AI-DEV-BEST-PRACTICES.md`, which explains the reasoning and
the failures behind each one. Process enforcement comes from the Superpowers skill library —
<https://github.com/obra/superpowers>.*

<!-- SPDX-License-Identifier: MIT · © 2026 Torsten Kablitz · https://github.com/tkablitz/ai_cto -->
