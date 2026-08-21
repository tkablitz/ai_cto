# ai_cto — repository rules

Public repository: <https://github.com/tkablitz/ai_cto>

## ⛔ This repository is PUBLIC

Everything committed here is world-readable, permanently, and is forked, cached, and indexed
within minutes of a push. Treat every commit as irreversible publication.

**Never commit, in files, commit messages, branch names, or issues:**

- Client, employer, customer, or partner names — past or present
- Colleague or personnel names, or anything about their performance
- Product names, internal system names, repo names, or work-item IDs from any engagement
- Cloud resource names, subscription/tenant IDs, endpoints, or hostnames
- Local filesystem paths (`C:\Users\...`, home directories)
- Anything from a legal, IP, security, or commercial-sensitivity context
- Metrics or incidents that could identify the system they came from

The guides deliberately describe real failures with real numbers. That is fine **because the
systems are unnamed and unnameable from the text.** Keep it that way: when adding a new lesson,
write the mechanism and the consequence, never the context that identifies where it happened.

**A pre-push hook enforces this.** It lives in `.githooks/pre-push`, and it is a *hook* rather
than CI on purpose: GitHub Actions runs after the push, and by then the blob is public. For an
irreversible failure, a check that runs afterwards is a notification, not a gate.

Hooks are not cloned — git will not run code that arrived over the wire — so **activate it once
per clone**, and give it the name list it cannot carry itself:

```powershell
git config core.hooksPath .githooks
# then write the real names into .git/leak-patterns, one regex per line —
# untracked, because the list itself is the thing that must never be published
```

If that list is missing, the hook **fails** rather than passing quietly. A scan with nothing to
scan for returns clean and looks identical to a real pass; see §2.2 of the guide.

It checks every tree being pushed for excluded names and local paths, checks author and committer
identity, and checks that every file is named in `LICENSE` and carries its SPDX notice. It does
not replace judgment about what belongs here — it catches the cases where judgment was already
exercised and then forgotten.

`git push --no-verify` bypasses it, as it bypasses any hook. That is git's design and cannot be
prevented from inside the hook.

**To prove the gate is real, use a throwaway branch — never `main`.** The hook scans the whole range
being pushed, not just the tip, so a violation committed and then *reverted* still reds it: the
revert adds a commit, it does not remove one, and the branch stays unpushable until the history is
rewritten. Verified 2026-08-21 — a reverted violation, gone from the tip with a clean working tree,
still blocked with exit 1. So commit the violation on a throwaway branch, hand the hook its stdin
directly instead of pushing, and delete the branch after:

```bash
printf 'refs/heads/main %s refs/heads/main %s\n' "$(git rev-parse HEAD)" "$(git rev-parse origin/main)" \
  | sh .githooks/pre-push origin "$(git remote get-url origin)"
```

**Save the output — it is the only evidence that survives.** A CI gate leaves a run record attached
to a commit that exists on no branch afterwards; a local hook leaves nothing at all. Delete the
branch and the proof goes with it unless the text was captured. *(The procedure was earned by a
history-auditing CI job elsewhere that poisoned its own `main` proving its red the usual way — break
it, revert, expect green. The missing run record is the part specific to a hook.)*

**Scan the history, not just the working tree.** Deleting a file in a later commit does not
unpublish it — if the commit that *added* it gets pushed, the blob is public forever and
`git show <sha>:<path>` retrieves it. A clean `git status` proves nothing about what a push
publishes. Before pushing new commits, check every tree they contain, and squash rather than
push a history that ever held excluded material:

```powershell
git log --stat origin/main..HEAD          # every path touched by unpushed commits
git log --format='%an <%ae>' origin/main..HEAD | Sort-Object -Unique   # published identities
```

Commit metadata is published too. Author and committer email land in the permanent record, so
they must not carry an employer or client domain.

## What this repository is

A licensed package teaching AI-assisted software development with DevOps and TDD as the enforced
foundations. One guide, a drop-in rules file, five templates. See `README.md`.

- `AI-DEV-BEST-PRACTICES.md` — the guide. The primary artifact.
- `starter-CLAUDE.md` — the template readers copy into *their* repos. Not this file.
- `templates/` — scaffolds referenced by the guide.

**`PLAYBOOK.md` is deliberately NOT here.** The engagement methodology — governance over teams you
don't own, architecture review structure, leadership communication — is retained as commercial
material and lives outside this repository. Do not add it, and do not reconstruct its content here.
If a contribution starts drifting toward "how to run the engagement," that belongs in the private
playbook, not in this repo.

## Licensing — do not change casually

Dual-licensed, and the split is deliberate:

- **Guides** — CC BY-NC 4.0 plus an internal-organizational-use grant. Commercial rights are
  retained by the author.
- **`starter-CLAUDE.md` + `templates/`** — MIT, so adopters can paste them in without friction.

Every file must be named under one of the two sections in `LICENSE`. A file that falls through the
gap is unlicensed by default. **When adding a file, add it to `LICENSE` in the same commit** — and
verify mechanically, not by eye:

```bash
for f in $(git ls-files | grep -v '^LICENSE$'); do grep -qF "$f" LICENSE || echo "UNLICENSED: $f"; done
```

The license terms also appear in `README.md` and in the guide footer. **All three must agree** —
individual files get forwarded without the repo, so each carries its own notice.

The MIT files carry that notice as a one-line SPDX comment at the end, because the README tells
readers to copy them out and a file that travels without its terms is a file nobody's legal team
will clear. **Repository infrastructure carries no notice** — today `CLAUDE.md`, `.gitattributes`,
and `.githooks/` — because none of it is meant to leave: it is this repo's own rules and plumbing,
not material a reader copies out. Stated as a category rather than a list, so adding plumbing does
not require editing this sentence. Everything a reader *is* told to copy needs a notice.
**A new template needs both its `LICENSE` section 2 entry and its SPDX line** — verify
mechanically:

```bash
for f in starter-CLAUDE.md templates/*.md; do grep -q SPDX "$f" || echo "NO NOTICE: $f"; done
```

> **Deferred — a guard test for `.gitattributes`.** The obvious guard ("no tracked file contains
> CRLF") passes vacuously on a Linux runner whether or not the file exists, so a real one needs a
> mechanism assertion alongside it — §2.2 of the guide. This repo has no CI to run either in.
> **Adopt when** this repo grows a pipeline for any reason; the guard is a few lines and belongs in
> that same commit. Until then the deletion risk is carried by review, not by a gate — which is
> vigilance, and named as such rather than pretended otherwise.

## Editorial standards

- **The guides are reviewed artifacts.** Substantive changes get the owner's review before push.
  Attribution lines name the model that actually did the work and assert a review that happened.
- **Claims cite mechanisms, not authority.** Every rule in the guides earns its place with a
  concrete failure. Don't add advice that lacks one.
- **No dangling references.** If a guide cites a template, that template ships here. Verify.
- **Prose paragraphs are single unbroken lines** where a renderer treats newlines literally;
  otherwise wrap at ~100 characters, matching the existing files.
- Commands appear in both shells only where the syntax genuinely differs. Identical commands get
  one block, tagged `powershell`.
