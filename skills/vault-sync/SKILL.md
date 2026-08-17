---
name: vault-sync
description: Promote durable findings from a repo's session-notes/ (see the checkpoint skill) into Hugo's Obsidian vault (~/hfluhr_vault). Manual/occasional only — never self-triggered. Use when the user asks to sync session notes to the vault, catch the vault up, or promote recent findings.
---

# Vault sync

`session-notes/` in a given repo (rescued by the `checkpoint` skill) is the
fast, low-friction capture layer — one file per session thread, git-tracked,
code-backed. The vault (`~/hfluhr_vault`) is the durable, curated layer Hugo
actually revisits, and it already has its own two-tier structure that this
skill feeds into rather than replaces:

- `Phd/daily/YYYY-MM-DD.md` — the fleeting/session-log layer. Dated, narrative,
  low-friction. E.g. the 2026-08-13 daily note has a "GLMsingle locked on reward
  feedback" heading with a six-item follow-up list promoted from a
  `learning-habits-analysis` session — that's the reference example for what a
  promoted entry looks like.
- `Phd/Projects/<ProjectName>/<area>/*.md` — the durable, permanent-note layer
  **for the specific project the repo belongs to**. E.g. for
  `learning-habits-analysis`, multivariate work belongs under
  `Projects/Learning-Habits/Imaging/Multivariate/` (`RSA.md` already lives
  there and is linked from the daily note as `[[RSA]]`); behavioral-modeling
  findings under `Behavioral Modeling/`; QC findings under `Data/`. This is
  **not** the same as the vault's `Phd/topics/` folder, which holds
  cross-project concept notes (RL, ANNs, habits) — don't put project-specific
  findings there. The repo name and the vault's project folder name won't
  always match exactly (`learning-habits-analysis` → `Learning-Habits`) — find
  the right one via `[[INDEX]]` or by asking, don't guess from string similarity.

A promotable finding usually belongs in **both**: a dated pointer in the daily
note (what happened, when) and, if it's the kind of thing worth knowing six
months from now independent of when it happened, folded into the relevant
permanent note under that project's folder. Not everything needs the second
half — use judgment, and ask if it's unclear which existing note (or whether a
new one) is the right home.

This is **never** self-triggered — only run it when explicitly asked. It writes
into Hugo's personal vault, outside the source repo, and "durable enough to
promote" is a judgment call each time.

## The vault has its own CLAUDE.md — read it and follow it

`~/hfluhr_vault/Phd/CLAUDE.md` governs everything written into the vault.
Before writing anything, if you haven't already in this session:

- **Read `[[INDEX]]` first** to orient — this is a hard rule in that CLAUDE.md,
  not optional here.
- Use Obsidian **wiki-links** (`[[Note name]]`) for cross-references, never
  markdown links `[text](path)`.
- **Log every edit** to `[[Claude-edits-log]]` under a `## [[YYYY-MM-DD]]`
  heading — creating, editing, or appending to any vault file all count.
- No hallucinated content — a promoted entry must trace back to something
  actually stated in the source session note, not an inference or embellishment.

## Procedure

**1. Identify the source repo and its matching vault project folder.**

Confirm which repo's `session-notes/` is being synced, and which
`Phd/Projects/<ProjectName>/` folder it maps to (via `[[INDEX]]`, or ask if new).

**2. Find session notes that haven't been promoted yet.**

List `session-notes/*.md` in the source repo. For each, check whether it's
already referenced from the vault: grep the daily notes and the matching
`Phd/Projects/<ProjectName>/` subfolder for the session-note's filename or an
equivalent backlink/description. If the user gives a date range or says "since
we last did this," scope to that instead of re-scanning everything.

**3. For each unpromoted note, draft condensed entries — not a copy.**

Short: the finding(s) that matter going forward, the number, one line of
consequence — not the derivation (that stays in the repo) and not the full
narrative. Always include the path back to the source session note (e.g.
`` `<repo>/session-notes/2026-08-13_...md` ``) so the vault entry can always be
traced to its rerunnable derivation, not just trusted.

**4. Propose placement, don't default silently.**

State which daily note and which (if any) permanent note under
`Phd/Projects/<ProjectName>/` you'd write to, and why. If nothing existing fits
and a new permanent note seems warranted, say so explicitly rather than creating
one unasked.

**5. Show the draft before writing, then follow through on the vault's own rules.**

Confirm the condensed version captures what matters before writing it — this is
vault content Hugo will revisit for months. After writing, log the edit to
`[[Claude-edits-log]]` as that vault's CLAUDE.md requires, and update `[[INDEX]]`
if a new note was created.
