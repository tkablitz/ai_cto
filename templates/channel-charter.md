# Channel charter — `<short-name>`

A **channel** is a session that outlives the task it started on. This file is what a channel fills in
before it writes anything, and what the other channels read to find out it exists. One per channel,
committed where every channel can see it.

Copy it, fill it, register it. If a row cannot be answered yet, write **`undecided`** — a blank row
and an unasked question look identical, and only one of them is safe.

---

## 1. Identity

| Field | Value |
| --- | --- |
| **Short name** | How other channels address this one. Lowercase, no spaces. **A greenfield channel names its root context; a fork names its lineage** — the roster's job is telling channels apart, and the first channel for a new context has no lineage to name itself for. Then read §3 before minting an address from it: a name that is fine on a private roster is not automatically fine in public commit metadata |
| **Purpose** | One sentence. What this channel is *for*, not what it did last |
| **Parent** | The short name it was forked from, or `none`. **A fork is a new channel** |
| **Started** | `YYYY-MM-DD` |
| **Machine** | Which workstation or VM |

## 2. What it owns

| Field | Value |
| --- | --- |
| **Working directory** | The one directory this channel writes in. **Not shared with any other channel** |
| **Memory store** | Derived from the working directory, so this is a consequence of the row above rather than a separate choice. Record the resolved key — **measured after the fact, never predicted**. A store may not exist until the channel writes its first memory |
| **Repositories it may write to** | Explicit list, or **`none`**. `none` is a normal answer for a channel that produces drafts, reviews, or outbound work |
| **Read-only clones it keeps** | Path, and what it is a clone of. Must sit **outside every channel's directory** |
| **Handoffs land in** | Path and repository. If that repository is private, say so — it decides where cross-channel items have to go instead |

> **A charter row is a claim, not a measurement.** One roster entry asserted its directories were
> "shared with no other channel" while a forked channel had been living in one of them for a week.
> Nothing contradicted it, because the sharing had no symptom. **Re-verify the rows in §2 at kickoff
> rather than trusting the file that exists to make them true.**

## 3. Commit identity

**Only fill this in if the channel actually commits.** A channel that pushes nothing needs no identity,
and inventing one puts a name in permanent metadata for no benefit — commit email is not removable
without rewriting history the remote already holds.

| Field | Value |
| --- | --- |
| **Author/committer address** | The address this channel's commits carry, or **`n/a — commits nothing`** |
| **How it is selected** | The mechanism, not the value: a repo-local `user.email`, a `gitdir:`-keyed conditional include, or a machine-level default |

> **Check the keying, not the value.** A conditional include keyed on the *remote URL*
> (`hasconfig:remote.*.url`) gives the **repository** an identity, not the channel — so on a machine
> hosting two channels, whichever one commits to a given repo silently gets the same address, and the
> record attributes the work to the wrong channel. **Only `gitdir:` keying, or a repo-local
> `user.email` in a clone one channel owns, actually follows the channel.** Verify by running
> `git config --show-origin --get user.email` *in the channel's own directory* and reading which file
> answered. Flags precede keys: `git config user.email --get` *sets* `user.email` to the literal
> `--get`, exits 0, and prints nothing.
>
> **An allowlist on the domain will not catch a wrong channel.** A gate accepting `*@example.com`
> accepts every suffix under it. It catches a foreign domain reaching a personal repo — worth having,
> and not the same thing. The check that catches the other case compares against *this charter*.
>
> **The decision this row records is made when the address is minted, not when a commit is pushed.**
> A channel that will ever author commits in a **public** repository must not be given an address
> containing a client, employer or organization name — because by push time the address exists, the
> commits exist, and the only remaining question is whether to publish them. A push-time allowlist is
> a useful backstop and it is answering a narrower question than this one: *is this address
> registered*, not *should this name exist*. Keep both, and do not mistake the second for the first.
>
> This is not hypothetical. One project published a client-derived address into a public repository's
> permanent metadata by following its own documented procedure — the domain was fine, the name sat
> before the `@`, and every check it had was written against the domain. **Rewriting history does not
> unpublish it**, so the decision recorded in this row is the last point at which the outcome is still
> available.
>
> **Give the machine-level fallback its own address, distinct from every channel's.** Then that
> address appearing on a commit is a signal — some channel committed from a clone with no repo-local
> identity — rather than a silent misattribution to whichever channel the include happened to name.

## 4. Registered

| Field | Value |
| --- | --- |
| **Announced in** | Link to the shared coordination record where the other channels can see this entry |
| **Last kickoff** | `YYYY-MM-DD` — last time this channel ran. **Its age carries no inference**: a channel nobody invoked owes nothing |

> **Registration and self-description are different acts with different owners.** Adding a channel to
> the canonical list is the owner's call. The charter is the channel's own account of itself, and
> nobody else should write it — a charter authored on a channel's behalf is a set of assertions about
> a context its author cannot see.

---

## Splitting a channel out of a directory it has been sharing

The retrofit. Do it **before** the fork writes anything else. The order below is not cosmetic: steps
2 and 5 lose information if they run late, and step 6 cannot be done by the channel that needs it.

1. **Back up the whole store first.** No version control, no history, no recorded author — the backup
   is the only undo, and everything after this point is destructive. Verify it by hashing every file,
   not by counting them: a file that copied empty makes the count look right.

2. **Inventory the store and attribute every file — by content, not by metadata.** Where a store
   stamps an origin id, that stamp records *which process wrote the file*, and a split needs to know
   *which channel owns it*. Those diverge the instant a fork happens, retroactively, for everything
   written before it. In one real split a file of plainly outbound material carried the parent's
   stamp — correctly, because it predated the fork by a day. **Deciding from stamps would have been
   confidently wrong, which is worse than having no stamps at all.**

3. **Expect fewer joint files than you think, and check whether "joint" is really a link.** In one
   split, ten files divided six/four with **none** genuinely shared — the one file that looked joint
   merely *referenced* a memory owned by the other channel. A one-way dependency is not co-ownership,
   and mistaking the two is the first reasoning anyone reaches for.

4. **Reword the first-person memories before moving anything.** Any file saying *this session*, *I
   own*, *my repo* is **already false** for one of the two readers and has been since it was written.
   Reword to name the channel — by its directory, which is the one fact that differs. Do this first:
   after the move you have lost which channel it was true for.

   **The exposure is not symmetric.** It is not "both channels are at risk"; it is *whoever wrote
   memories about itself*. Memories asserting facts about the work are harmless to the other reader.

5. **Move the files, prune the index rather than replacing it, then rewrite the links that now cross
   out.** The index is loaded every session and is never leftover; a rebuilt one loses what it was
   never told to carry. The link step is the one people skip, and it matters because **an unresolved
   cross-reference is legitimate** — it marks something worth writing later. Across a store boundary
   the identical syntax means the target exists, elsewhere, unreachable. The notation cannot
   distinguish *not written yet* from *not visible from here*.

   Check the **direction** before assuming the cost is shared: in one split all four crossing
   references ran the same way, so one channel could leave without touching a file and the other
   could not.

   Restating a fact inline beats leaving a dangling reference, and it **re-creates the duplication
   the split exists to remove**. Take the trade deliberately and write down that you took it; two
   copies of a fact will drift and nothing will tell you when.

6. **Neither channel can finish this alone, so do it as copy → verify → delete.** A channel must not
   delete from a store it does not own. The departing channel copies out and verifies; the owning
   channel deletes the originals only after confirming each file exists in the destination *and* in
   the backup. Between those two steps the fact lives in two places and can drift — keep the window
   short and state that it is open.

7. **Decide the commit identity — §3 above — and check the keying rather than the value.**

8. **Restart the moved channel in its new directory, with a written state summary as its first
   input.** The old session's context does not follow it and should not: **a summary written
   deliberately beats a context inherited accidentally**, which is this whole section in miniature.
   Write the summary *before* stopping the old session, and write it clean of names the directory may
   later have to publish — a file written clean needs no retrofit.

9. **Register the new charter, and amend the parent's.** The parent still claims a directory it no
   longer solely occupies until somebody edits it.

**Verify by measurement, not inspection.** After the split the two stores share no filename, each
index lists exactly its own files, and every remaining cross-reference resolves locally. All three
are one command each. None of them is what you will feel like doing at the end of a retrofit.

**And time-stamp what you verified.** In one split, "all files copied byte-identical" and "three files
differ" were both true and reported hours apart — the copy was clean, and the links were rewritten
afterwards. **A verification run at the wrong moment tells the wrong story even when everyone involved
is honest and correct.**

<!-- SPDX-License-Identifier: MIT · rev 2026-08-27.2 · © 2026 Torsten Kablitz · https://github.com/tkablitz/ai_cto -->
