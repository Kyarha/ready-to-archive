---
name: ready-to-archive
description: >-
  Audit the current session before it gets archived and give one clear GO / NO-GO on
  whether it is safe to archive. Use when the user invokes /ready-to-archive, or asks
  "can I archive this?", "is it safe to archive", "ready to archive?", "is everything
  recorded", "did we save everything", or "anything left before I archive". Runs a
  grounded three-pillar check — RECORDED (every decision, conclusion, and consequential
  change is written where the project keeps it, not only in this chat), COMMITTED (the
  git working tree is clean and pushed if the work must live remotely), and LANDED
  (branch work is merged to main where it had to be, or the open PR is named) —
  verifying each against git and the filesystem, NOT the chat recollection. Defaults to
  caution: anything it cannot positively verify is reported as a gap, never waved
  through. Ends with an explicit verdict and, when there are gaps, offers to close them
  — but never commits, pushes, merges, or records on the user's behalf without an
  explicit go-ahead.
---

# Ready to archive?

**One job: tell the user, honestly, whether this session is safe to archive — so nothing
of consequence is lost when it disappears.**

The user is about to archive the session. Once it is archived, whatever lived *only in this
conversation* is gone. This skill is the last gate: it answers **"can I safely archive?"**
with a clear **GO** or **NO-GO**, backed by evidence, not optimism.

---

## The prime directive — verify, don't recall

**This skill exists because a session was once archived with things unrecorded — the check
trusted the conversation instead of the repo.** Do not repeat that.

- **Every "yes, that's saved" must point at something you actually observed this turn** — a
  file you just read, or a `git` command you just ran. Your recollection of having "written
  it down earlier" is *not* evidence; the context may have been summarized, or the write may
  have failed, or it went somewhere that won't survive.
- **Default to NO-GO.** A clean GO requires positive evidence on all three pillars. Silence
  is not evidence. If you cannot verify something, it is a **gap**, not a pass.
- **Name the boundary.** You can only vouch for what is observable from here — this repo, the
  user's memory files, and git. Work done in other sessions or external systems is outside
  your window; say so rather than implying you checked it.

If you honor nothing else here, honor this section.

---

## The three pillars (the audit)

Run all three. Each is a titled bullet in the output with its evidence or its gap.

### Pillar 1 — RECORDED (nothing of consequence lives only in this chat)

First, recall what this session actually produced: **decisions** made, **conclusions /
findings** reached, and **modifications** to files or plans. Then, for *each* one, verify it
is written somewhere durable — by reading the target, not by trusting memory.

- **Decisions** (a hard-to-reverse choice, a "we'll do X not Y") → the project's decision
  record. Look for where *this* project puts them and confirm the decision is there:
  jig ADRs (`docs/decisions/adr-*.md`), jig inbox (`docs/inbox.md`), studio `decisions/log.md`,
  or a plain `DECISIONS.md` / `CHANGELOG.md`. A real decision that isn't written down is a gap.
- **Conclusions / findings** (a root cause identified, a benchmark result, "the answer is X")
  → these are the easiest to lose because they feel "obvious" in the moment and live only in
  chat. Confirm each consequential finding is captured — in a bug doc, an ADR's context, the
  spec's clarifications, a findings/notes file, or the user's memory. If it only exists in
  this conversation, it is a gap.
- **Modifications that need a paper trail beyond the code itself** → a jig slice implemented
  but its `STATUS` not moved (e.g. still `IN_PROGRESS` instead of `REVIEWED`/`DONE`), an ADR
  left `Proposed` after being accepted in chat, a `README` / `CLAUDE.md` / spec that the change
  made stale. The diff being committed is Pillar 2; here you ask whether the *surrounding
  record* was updated to match.
- **Cross-session facts & user feedback** → the user's auto-memory
  (`~/.claude/.../memory/`). If the user stated a new preference, gave a correction, or
  established a project fact this session and it is not saved there, that is a gap — hand off
  to `jig:memory-sync` in a jig project, or write the memory file directly.

If you genuinely cannot reconstruct everything decided this session (context was summarized),
**say so** and ask the user to glance at the specific area — do not paper over it with a
confident "all recorded."

### Pillar 2 — COMMITTED (the working tree is clean)

- **Is this even a git repo?** `git rev-parse --is-inside-work-tree`. If not, "committed"
  does not apply — say so plainly, and lean harder on Pillar 1 (there is no safety net).
- **Clean tree?** `git status --porcelain`. Any modified or staged files = uncommitted work.
  Distinguish real work from scratch/build artifacts, but **list untracked files** — a new
  file of consequence that was never `git add`ed is the classic silent loss.
- **Pushed, if it must live remotely?** If the branch has an upstream
  (`git log --oneline @{u}..HEAD`), unpushed commits are a gap when the work needs to survive
  off this machine. Local-only commits are fine if the user works locally — flag, don't assume.
- **Stashes?** `git stash list`. A stash that looks like unfinished work is worth surfacing.

### Pillar 3 — LANDED (on main where it had to go)

- **What branch are we on?** `git branch --show-current`.
- **Was this work meant to land on main?** If so, check whether it actually did:
  `git log --oneline main..HEAD` (commits on the branch but not on main) and, if a remote
  exists, compare against `origin/main`. Detect the default branch name — it may be `main`
  or `master`.
- **Three honest outcomes**, not one forced answer:
  - *Merged* — the branch's commits are on main → landed, GO on this pillar.
  - *Open PR* — name it; landing is pending review, which may be exactly right. Not a gap,
    but say it plainly so the user knows it isn't on main yet.
  - *Stranded* — commits sit on a feature branch with no PR and no merge, and the work was
    meant to land → gap. Hand off to `jig:slice-land` (jig projects) or offer to open the PR.
- **Do not force everything onto main.** Some work legitimately stays on a branch. Surface the
  state; let the user decide whether it "had to" land.

---

## The output shape

Lead with the verdict. Then the evidence, then any gaps, then the closing commitment.

### 1. The verdict line (always, first)

One line, unmissable:

- ✅ **SAFE TO ARCHIVE** — everything of consequence is recorded, committed, and landed where
  it had to be.
- ⚠️ **NOT YET** — *N* thing(s) still need attention before archiving (listed below).
- ❓ **CAN'T FULLY VERIFY** — I couldn't confirm *X* from here; check it before you archive.

### 2. What I checked (always)

The three pillars, each a titled bullet with the **evidence** — the command output or file
you actually looked at:

- **Recorded** — the ADR / memory / status update you confirmed exists (or the gap).
- **Committed** — what `git status` showed, verbatim-ish (`clean`, or the file list).
- **Landed** — the branch, and its merge/PR state against main.

### 3. Still open (only when there are gaps)

One titled bullet per gap: **what** is unrecorded / uncommitted / unstranded, and **where it
needs to go**. This is the user's to-do list for reaching GO — keep it a scannable checklist,
never a paragraph.

### 4. The verdict, restated + the offer (always)

Close by restating GO / NO-GO in one line. If NO-GO, give the **shortest path to GO**, and
offer to help close the gaps — but every consequential step waits for the user's explicit yes:

> *Two gaps: the root-cause finding isn't in the bug doc, and the fix is committed on
> `fix/parser` but not merged. Want me to (a) write the finding into docs/bugs/041, and (b)
> hand off to jig:slice-land to merge — or will you take it from here?*

When it is a genuine **GO**, say so plainly and stop. Don't manufacture doubt to seem
thorough — a clean session deserves a clean green light.

---

## What this skill does NOT do on its own

Recording, committing, pushing, merging, and landing are **the user's calls** — this skill
diagnoses and reports; it does not act unless asked.

- It **never** commits, pushes, merges, or opens a PR without an explicit go-ahead.
- Recording a missed finding or memory is low-risk and may be *offered* eagerly — but still
  only done on the user's yes.
- The verdict is advisory. The user archives; the skill just makes sure they do it
  with eyes open. **A narrow "yes, fix that one gap" is not a blanket yes** to commit-and-push
  everything.

---

## Judgment

- **Grounded, never guessed.** Every pillar's verdict traces to a command you ran or a file
  you read *this turn*. "I believe it was saved" is a NO-GO, not a GO.
- **Conservative by design.** When torn between GO and NOT-YET, pick NOT-YET and name the
  doubt. The cost of a false GO (lost work) dwarfs the cost of a false NOT-YET (a second look).
- **One clear verdict, not a shrug.** The user asked a yes/no question. Answer it — then show
  the evidence. Don't bury the verdict under the checklist.
- **Adapt to the project.** Git pillars are moot in a non-code / non-git folder; the Recorded
  pillar carries the weight there (studio decisions log, memory, the project's own docs). In a
  jig/servo/shaper repo, lean on the artifacts those workflows already maintain.
- **Honest about your window.** You vouch for what's observable from here. If something
  important happened outside it, say "I can't see that from this session — please confirm."
