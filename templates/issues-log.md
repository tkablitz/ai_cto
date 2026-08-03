# <System> Issues & Observations

Living doc tracking bugs, contract ambiguities, and future requirements observed by
<your validation tool / review process> while integrating against <system> at <host>.

**Status legend:**
- `Open` — awaiting investigation / decision
- `Resolved (client-side)` — fixed in our tooling; their contract unchanged
- `Resolved (system)` — fixed by the owning team
- `Deferred` — acknowledged, not blocking, to be addressed later

Rules of the log: issues are numbered and never deleted; updates append dated notes; every
claim carries evidence (verbatim failure signature, log excerpt, or reproduction command);
every issue ends with a specific **ask** for the owning team; client-side mitigations are
recorded but never substitute for the system fix.

---

## Issue #1 — <one-line title>

- **Date observed:** <date>
- **Endpoint/Component:** <what>
- **Status:** Open
- **Failure signature:**
  ```
  <verbatim error / response body / log lines>
  ```
- **Observed impact:** <what broke, at what scale, measured how — cite the run/campaign>
- **Root cause (if known):** <file:line or behavior analysis; distinguish confirmed from suspected>
- **Ask for <owning team>:** <numbered, specific, smallest-viable actions>
- **Client-side mitigation (if any):** <what we changed in our tooling; note it does not close the issue>

---

## Future Requirement — <heads-up title>

- **Status:** Deferred — heads-up for <team>
- **Context:** <what's coming that makes this matter>
- **Ask:** <options, ranked>

<!-- SPDX-License-Identifier: MIT · © 2026 Torsten Kablitz · https://github.com/tkablitz/ai_cto -->
