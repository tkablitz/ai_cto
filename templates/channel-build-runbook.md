# Building a new channel — the owner's half

> **This is a worked example, not a generic template.** It was written for, and run on, one specific
> stack — **Windows 11, PowerShell 7 with Git Bash alongside, Claude Code in the desktop app, Azure
> DevOps for the repository, the Azure CLI (`az`) for creating it, and GitHub for the public
> repository whose name-exclusion list it checks against.** The paths, the key-derivation rule, the
> memory-store location, the `az repos` commands, the `AZURE_CONFIG_DIR` fix, the `$env:` variables
> — all of that is that stack's, and will be wrong on yours in ways that mostly fail silently.
>
> **What transfers is the shape, not the commands:** describe a step before running it; a claim gate
> before every create; verify from outside rather than from the create's own output; measure a
> derived value rather than deriving it; make every check able to fail before trusting it; set the
> identity and the branch in the gap between clone and first commit; register with the disposition in
> the same act.
>
> **So the intended use is: hand this file to your own AI agent and have it rewrite the commands for
> your environment** — macOS or Linux, bash or zsh, GitHub or GitLab or Bitbucket, AWS or GCP, the
> CLI or a different desktop tool — and keep every check, gate, and *stop here* exactly where it is.
> A prompt that works: *"Adapt this runbook for `<OS>`, `<shell>`, `<repository host>`, `<cloud CLI>`
> and `<agent tool>`. Change the commands and the paths; do not remove or weaken any check, and
> where a check cannot be expressed on my stack, say so rather than dropping it."* Then run the
> result the way §0's first paragraph says — one step at a time, described before it runs — because
> the adaptation itself is exactly the kind of plausible, well-formed output whose errors show up
> only when a step is written down and turns out to be wrong.

A **channel** is a session that outlives the task it started on (guide §5.9). This runbook stands one
up from nothing: a working directory, a repository, a seeded memory store, and a registration the
other channels can read. It was extracted from two builds run a day apart on two machines, and every
step below either caught a real defect on one of them or exists because a step above it did.

**It covers only the owner's half.** A channel writes its own charter and drafts its own roster entry
— nobody else may, because a charter written on a channel's behalf is a set of assertions about a
context its author cannot see. The owner's half ends at the first prompt. What the channel does after
that is `templates/channel-charter.md`.

**Run it as a conversation, not a script.** Every step below is *described → approved → run →
reported*, in that order, one at a time. That is slower than a script and it is not a ceremony: on
the two builds this came from, **three defects were caught only because a step was written down
before it ran and the description turned out to be wrong** — a cloud CLI's default project, a claim
about how an empty repository clones, and a claim carried from one machine to the other without
re-measuring. A script that executes a phase end to end deletes the place those were caught. Whatever
you automate, keep the description step.

Every `<placeholder>` is yours to fill. Commands are PowerShell unless the syntax differs.

---

## 0. Verify the machine layer — nothing here is a look-and-see

"Once per machine" steps fail silently in both directions. Skipped, the channel runs without the
working agreements and nobody notices for weeks. Re-run, a seeding step overwrites a store that had
accrued a channel's own memories. **So a machine that has obviously been set up gets checked anyway**,
and every check below distinguishes *done* from *not done* rather than looking plausible.

```powershell
# Where is the shared playbook clone, according to memories already seeded on this box?
Get-ChildItem "$env:USERPROFILE\.claude\projects\*\memory\reference_playbook.md" |
  Select-String -Pattern '<playbook-repo-name>' | Select-Object -First 3 Path, Line

# Is it current, and has nobody committed in it?
git -C "<clone>" pull --ff-only
git -C "<clone>" rev-list --count origin/main..HEAD        # expect 0

# Which identity would a fresh clone get here? Read WHICH FILE answered, not just the value.
git config --show-origin --get user.email

# Which branch name would a fresh clone be born with?
git config --get init.defaultBranch

# If the target is a cloud repository host: which project would a bare create land in?
az devops configure --list        # or the equivalent for your host
```

**Run the identity check from outside every repository.** Inside a clone that has a repo-local
override it reports the override, not the fallback, and proves nothing about what a new clone will
inherit. One build ran it from the wrong directory and read the answer as settled.

**All four of those are one question: which machine-level defaults are right for one tier of work on
this box and wrong for another?** Treat it as a class, not a list, and expect a fifth:

| Default | Right for | Wrong for | How it fails |
|---|---|---|---|
| `user.email` global block | the account's own repos | anything a conditional include does not match | a commit carries the wrong identity, permanently |
| `init.defaultBranch` | one host's convention | the other host's | a fresh clone's unborn `HEAD` points at a branch nobody expects — **silent until the first push** |
| a cloud CLI's default project/org | the last thing you did there | this build | a bare `create` lands the repo somewhere nobody will look, **and succeeds** |
| a conditional include | the one remote pattern it names | every other remote | the machine's advertised fallback is true for one repository tier and false for another |

**And a fifth that is not a default but shares the shape: anything keyed to the OS user profile is
shared by every channel on the machine, whether or not it looks like it.** A cloud CLI's token cache
is per profile — one `login` by any channel silently re-identifies every other channel and the owner's
own shell, with no error and nothing on any record. The working tree and the memory store are the
same fact one system over. If a channel needs its own cloud identity, relocate the cache
(`AZURE_CONFIG_DIR` or your CLI's equivalent) in that channel's own settings; it applies from the
*next* session, and CLI extensions are per cache directory, so a fresh one has none.

Every command from here carries its target explicitly — `--org`, `--project`, a full remote URL —
because a default that has been watched being wrong is not a default any more.

## 1. Decide four things before anything is created

These are the owner's decisions, and the first one is made at minting time, not at push time.

**A. The short name.** Lowercase, no spaces. *A greenfield channel names its root context; a fork
names its lineage.* Then test it — and the address it implies — against the public repository's
name-exclusion list, **by running the pattern file, not by reading it**:

```bash
for n in <candidate> <candidate>-prime <other>; do
  printf '%s' "$n" | grep -qi -E -f <public-clone>/.git/leak-patterns \
    && echo "$n  MATCHES — cannot be minted" || echo "$n  clear"
done
```

An organization's full name will match. An **abbreviation of it will not** — and it is still
organization-derived; the scan cannot see it. Decide deliberately whether that is acceptable (one
owner ruled that an abbreviation identifying nothing to an outsider is fine), and record the ruling
where the next build finds it. **A channel's display name in your tool's UI may itself be on the
list even when its short name is not** — so the short name is the only token that ever travels into
a commit trailer, a PR body, or anything reaching a public repository.

**B. Where the repository lives.** This decides which identity tier the clone lands in (§0's fourth
row), which CLI creates it, and whether the platform's PR flow applies. Do not assume; measure it
with a throwaway:

```bash
mkdir probe && cd probe && git init -q && git remote add origin "<the exact remote URL you intend>"
git config --show-origin --get user.email      # THIS is what the channel would commit as
cd .. && rm -rf probe                          # never committed, never pushed
```

**C. The commit identity.** Three valid answers, and the template asks which:

| Answer | When | Cost |
|---|---|---|
| a channel-specific address, repo-local | the usual case | none — that is what the roster expects |
| **an address the channel shares** — the machine fallback, or the organization's own identity on organization work | company work on a company machine | **attribution is convention, not mechanism.** No configuration anyone inspects afterwards says which channel wrote a commit. Valid, *if written in the charter*; a gap if discovered |
| `n/a — commits nothing` | drafts, reviews, outbound | none |

If it is a channel-specific address, set it **in the gap between clone and first commit** — the
machine tier will otherwise supply something plausible and wrong, silently.

**D. Whether it may ever push to the public repository.** If not: its address is deliberately
absent from that repository's identity allowlist, and **the disposition is recorded in the same act
as registration** — because registering is what creates the drift row, and an unrecorded absence
surfaces the next morning as *"would be refused mid-push"* for a channel that was never going to
push. If the name is organization-derived the answer is *no*, and on a machine that has a clone of
the public repository that rule has no backstop: say so in the charter as vigilance.

## 2. Create — claim gates before every create, verify from outside after

```bash
# CLAIM GATE: does the name already exist? If this lists it, stop — it is somebody's.
az repos list --org <org-url> --project "<project>" --query "[].name" -o tsv

# Create, with every target explicit.
az repos create --name <repo> --org <org-url> --project "<project>" -o json

# VERIFY FROM THE API, not from the create command's own output.
az repos show --repository <repo> --org <org-url> --project "<project>" \
  --query "{name:name, defaultBranch:defaultBranch, remoteUrl:remoteUrl}" -o json
```

A create confirming its own success is a notification, not a check (guide §1.2). Substitute your
host's commands; keep the three-step shape.

```bash
# The directory must not already exist. `mkdir` without -p IS the claim gate.
mkdir <channel-dir>
git clone "<remoteUrl from the API>" <channel-dir>

cd <channel-dir>
git symbolic-ref --short HEAD          # NOT rev-parse --abbrev-ref: that exits 128 on an empty repo
git config --show-origin --get user.email
```

**An empty repository has no default branch, so a clone of one takes the local
`init.defaultBranch`** — whatever the host declares, and whatever you were told. One build was told
the host would supply `main`; the clone came up on `master`. Fix it now, while it is free:

```bash
git symbolic-ref HEAD refs/heads/main
git symbolic-ref --short HEAD          # read it back
```

After the first push this becomes a branch rename with a remote migration, which once stranded two
commits for six days.

## 3. Measure the memory key — never derive it

Your tool keys persistent memory to the working directory, through a transform that is
**non-injective** (separators and underscores both flatten to one character, so `my_project` and
`my-project` share a store) and **does not normalize case** (one machine holds `C--…` and `c--…`
stores side by side, for the same drive, and nothing in the path predicts which you get). A wrong
key writes eight files into a directory no session ever opens, and the channel runs unseeded with no
error. So the key is read back from what the harness created, never typed from the rule.

```powershell
# BEFORE opening the session — to a FILE, never a shell variable
Get-ChildItem "$env:USERPROFILE\.claude\projects" -Directory -Name |
  Sort-Object | Set-Content "$env:TEMP\stores-before.txt"
```

Now open a session with its working directory at `<channel-dir>` **and give it a prompt** — a
session that is never prompted creates nothing. Keep the prompt trivial; this session runs
unseeded, with none of the working agreements loaded:

> Print your current working directory, then stop. Do nothing else.

> **This session is a throwaway. Do not name it, pin it, or treat it as the channel.** It exists to
> make the harness create a directory. The channel's real first session is a *different* one, opened
> after seeding. On one build the owner renamed and pinned this session in the tool's session list —
> reasonably, it was the only one for that directory — and had to unpin and replace it. **The step
> reads like the beginning of the channel; it is the beginning of the measurement.**

**Open it in the surface the channel will actually run in.** If the desktop app and the CLI derive
keys differently, the seed lands where the real session never looks. The check below catches *"no
store was created"*; it cannot catch *"created under a different key than the real session uses."*

```powershell
$before = Get-Content "$env:TEMP\stores-before.txt"
$after  = Get-ChildItem "$env:USERPROFILE\.claude\projects" -Directory -Name
Compare-Object $before $after | Where-Object SideIndicator -eq '=>' |
  Select-Object -ExpandProperty InputObject
```

**Exactly one name, or stop.** Empty means nothing was created — prompt the session and retry. More
than one means you cannot tell which is yours — re-baseline. Do not filter by the project's name: the
filter is built from the same transform whose failure it exists to catch, and fails the same silent
way.

Then check what the new store **holds**, not whether it exists — the harness creates `memory/`
during that session, so by now it is there and empty, and a reader who treats presence as "something
accrued" skips a seeding that was needed:

```powershell
Get-ChildItem "$env:USERPROFILE\.claude\projects\<key>\memory" -File   # expect nothing
```

## 4. Seed — with checks that could have failed

Run from the root of the playbook clone, so the resolved path written into the seeds is this
machine's:

```powershell
$playbook = (Get-Location).Path
$mem = "$env:USERPROFILE\.claude\projects\<key-as-READ-BACK>\memory"

# How many SOURCE files carry the token? Must be NON-ZERO or the check after seeding proves nothing.
(Select-String -Path "$playbook\seed-memory\*" -Pattern '\{\{PLAYBOOK_ROOT\}\}').Count

Get-ChildItem "$playbook\seed-memory\*" -File | ForEach-Object {
  (Get-Content $_.FullName -Raw).Replace('{{PLAYBOOK_ROOT}}', $playbook) |
    Set-Content (Join-Path $mem $_.Name) -NoNewline
}

(Get-ChildItem $mem).Count                                              # expect the seed count
(Select-String -Path "$mem\*" -Pattern '\{\{PLAYBOOK_ROOT\}\}').Count   # expect 0
# The resolved path actually landed. Match the MACHINE'S PATH — never the repository's name,
# which every copy carries whether or not anything was substituted.
Select-String -Path "$mem\*" -Pattern ([regex]::Escape($playbook)) | Select-Object -Unique Filename
```

Three assertions, and each one exists because its predecessor was vacuous on a real build. *Absent
afterwards* passes if the sources were already substituted. *Present afterwards* passes if you match a
string the sources contain anyway. **The source count is what makes the leftover count mean
anything.** Expect source `N`, leftover `0`, and the resolved path in exactly those `N` files.

**Never re-run this against a store holding anything beyond the seed set.** It writes the index, and
the index is where a channel's own memories are listed.

## 5. The first prompt — and what it must and must not carry

Open a new session at `<channel-dir>` — this one is seeded. The prompt sends the channel to read the
playbook, its memory, and the coordination record; tells it its short name and its context; and
instructs it to measure its own key, write its charter from `templates/channel-charter.md`, and draft
its roster entry for you to register. Then the first goal, which is a written brainstorm — not a
plan, and not any cloud resource.

Two opposite-looking rules, both load-bearing:

> **The prompt must not carry *state* written somewhere the channel can read** — a pasted roster,
> a list of the other channels — because a pasted copy goes stale without a symptom.
>
> **The prompt must carry *decisions* written nowhere** — the identity trade from §1C, a policy the
> owner refined verbally during the build, a runbook the channel should read for its structure
> rather than its commands — because the prompt is their only carrier. One setup record went stale
> *during the build it described*; the channel's entry was correct on that point only because the
> refinement was typed into the prompt by hand.

## 6. Register — the owner's act, in one motion

The channel posts its draft entry. You add it to the canonical list, correct the machine row (it will
be wrong: one build found the machine hosting four channels where the row said two), **and record its
allowlist disposition in the same act** — §1D. Then re-run whatever drift check you keep and read
the result before believing it: on one build the check classified the new address correctly *and
explained the classification wrongly*, naming the local part when the domain had matched.

## Not on day one

**No cloud bootstrap, no pipeline.** If the channel needs cloud accounts, the runbooks for that open
with decisions that are cheap up front and a migration to retrofit — read them *before* the phase is
planned, not while it is executed. And read them for their decision structure if your cloud or host
differs from theirs; the commands will not transfer and the questions will.

---

**What this runbook does not do, deliberately.** It does not write the charter, draft the roster
entry, or name the other channels. It does not automate the description step. And it does not claim
to have found every machine default — the table in §0 is short because five have been measured, not
because only five exist. When you find the sixth, add it there rather than beside it.

<!-- SPDX-License-Identifier: MIT · rev 2026-09-07 · © 2026 Torsten Kablitz · https://github.com/tkablitz/ai_cto -->
