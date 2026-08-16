# `<Company>` — Definition of Done

**Audience:** development team + project/product management.
**Purpose:** one shared meaning for the word "Completed," so status tables, standups, and
architecture documents say the same thing to everyone.
**Created by Claude `<model>`, reviewed by `<owner>` · `<date>`**

## 1. The core rule

**A claim of "Completed" must come with evidence a reviewer can check.** If it can't be
demonstrated on request — a command, a test run, a URL, a pipeline link — it isn't done, it's
intended.

## 2. Status vocabulary (use these five words, only)

| Status | Means | Test |
|---|---|---|
| **Designed** | Documented approach, agreed | Design doc reviewed |
| **Prototype** | Works in a demo/sandbox setting; not enforced, not repeatable, or not in the target environment | You can demo it, but you couldn't put a customer on it |
| **In progress** | Being built; parts verifiably work | Partial evidence exists |
| **Complete** | Meets all five criteria in §3, in the environment named | Evidence linked per §4 |
| **Verified** | Complete + independently re-checked by someone who didn't build it | Second signature / verification note |

Hard rules: **name the environment** ("Complete (staging)" and "Complete (production)" are
different claims; no environment = Prototype until shown otherwise). **A demo is a Prototype.**

## 3. The five criteria for "Complete"

1. **Runs in the environment it claims** — not "would work if deployed."
2. **Enforced, not optional** — no bypass path. Ask: *what happens to a caller who ignores
   this feature?* A security control with a side-door is decoration.
3. **Tested, including failure cases, with results retained** — automated tests/probes exist,
   pass, and output is linked. Happy-path manual clicks don't qualify.
4. **Reproducible from source control** — config/infra in the repo, deployed by pipeline.
   Portal-only work fails this criterion by definition.
5. **Observable and documented** — logs/metrics wired; secrets vaulted; the doc states
   environment, tier, and known gaps.

Criterion 4 caps everything: while a thing is portal-only, nothing built on it rises past
Prototype, no matter how well it demos.

## 4. Claim-with-evidence table format

| Item | Status (env) | Evidence |
|---|---|---|
| `<feature>` | Complete (staging) | <command a reviewer can run read-only, with expected output / pipeline run ID / probe URL> |

## 5. For the PM — running a status review

1. Ask for the **evidence cell**, not the status word: "Completed — evidence?"
2. Spot-check **one** claim per review by running the command or opening the link.
3. No evidence → relabel **Prototype** and move on. Prototype is a location, not a criticism.
4. Police the two recurring failure patterns: **aspirational-as-current** (target architecture
   in present tense) and **demo-as-done** (sandbox walkthrough labeled Completed).
5. A milestone is met when its rows are **Complete with evidence** — not when the demo went well.

<!-- SPDX-License-Identifier: MIT · rev 2026-08-03 · © 2026 Torsten Kablitz · https://github.com/tkablitz/ai_cto -->
