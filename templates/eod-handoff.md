# EOD Handoff — `<date>`

## Today in one paragraph
<What shipped, what it proves, where prod stands (version/tag), anything in flight.>

## Shipped (with evidence)
1. `<Deliverable>` → <file/commit/version/pipeline-run>. <One line on what the gate proved.>
2. …

## Kickoff findings (`<date>`)
- <Canary/probe results vs the last run; drift in the client's system; roster/access changes.>

## Open items → NEXT SESSION
1. **Next kickoff.** Watch: <specific metrics/trends with this run's values, to compare against>.
2. <Next epic / next concrete step, with pointers to its spec/plan.>
3. **Backlog (non-blocking):** <lettered/numbered small items with one-line context each.>

## Cross-channel

Delete this section if you are the only agent working on this material. If you are not, it is the
half of the handoff that a file cannot deliver — see §4.3.

- **Posted to the shared record:** `<what other channels need to know, and where you put it>`
- **Waiting on:** `<channel or person>` for `<what>` — recorded at `<where they will see it>`
- **Shared state I wrote to today:** `<trees, branches, stores others also use — or "none">`

## State at close
- Prod `<version>` healthy, tagged. `<branch>` == `<remote>`, tree clean (<known untracked>).
- Tags: `<range>`. Suites: <counts per project>.
- Roadmap tracker updated. Memories current: <files touched>. 
- Op notes: <sharp-edged environment facts learned today, one line each>.

<!-- SPDX-License-Identifier: MIT · rev 2026-08-18 · © 2026 Torsten Kablitz · https://github.com/tkablitz/ai_cto -->
