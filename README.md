<div align="center">

# 📦 ready-to-archive

### The last gate before you archive a session — so nothing of consequence gets lost.

*One clear GO / NO-GO, backed by evidence, not optimism.*

[![version](https://img.shields.io/badge/version-0.1.0-blue)](https://github.com/Kyarha/ready-to-archive/releases)
[![license](https://img.shields.io/badge/license-MIT-green)](LICENSE)
[![Claude Code plugin](https://img.shields.io/badge/Claude%20Code-plugin-8A63D2)](https://docs.claude.com/en/docs/claude-code)
[![skills](https://img.shields.io/badge/skills-1-orange)](#whats-inside)

</div>

---

You finish a session and go to archive it. But once it's archived, whatever lived **only in
that conversation** is gone — a root cause you figured out, a decision you made, a file you
never committed. **ready-to-archive** is the check you run first. It reads the repo, your
memory, and git — and tells you, plainly, whether it's safe.

It was built for one reason: **a session once got archived with things unrecorded**, because
the check trusted the conversation instead of the repo. This skill doesn't. Every "yes, that's
saved" points at a file it actually read or a `git` command it actually ran — and anything it
**can't** verify is reported as a gap, never waved through.

## 🚀 Quick start

1. **Install** — download the latest release zip and add it as a plugin in Claude Code.
2. **Ask, before you archive:**
   > *"Ready to archive?"*  ·  *"Can I safely archive this?"*  ·  or run `/ready-to-archive`

You get back a verdict — ✅ SAFE, ⚠️ NOT YET, or ❓ CAN'T FULLY VERIFY — with the evidence.

## 🧭 What's inside

One skill: **`/ready-to-archive`**. It runs a grounded **three-pillar** audit:

| Pillar | The question | Where it looks |
| --- | --- | --- |
| **📝 Recorded** | Does every decision, conclusion, and consequential change live somewhere durable — not only in this chat? | decision log / ADRs, spec `STATUS`, inbox, `README` / `CLAUDE.md`, your memory files |
| **💾 Committed** | Is the git working tree clean — nothing modified, staged, or untracked-of-consequence — and pushed if it must live remotely? | `git status`, upstream, stashes |
| **🚀 Landed** | Did branch work reach main where it had to, or is the open PR named? | branch vs. `main`, remote, open PRs |

## 🔒 It diagnoses — you decide

The verdict is **advisory**, and the skill acts only when you ask it to:

- It **never** commits, pushes, merges, or opens a PR without your explicit go-ahead.
- When there are gaps, it offers the **shortest path to GO** and hands off to the skill that
  owns the work (`jig:slice-land` to merge, `jig:memory-sync` to save a learning).
- A narrow *"yes, fix that one gap"* is never a blanket yes to commit-and-push everything.

## 🛡️ Why it defaults to caution

A false **GO** loses work; a false **NOT YET** just costs a second look. So when it's torn,
it picks NOT YET and names the doubt. When it genuinely can't reconstruct what happened this
session (context was summarized), it says so and asks you to glance — rather than papering
over it with a confident green light.

---

<div align="center">
<sub>MIT · made so a session never gets archived with something important still in the chat.</sub>
</div>
