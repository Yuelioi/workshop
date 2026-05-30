---
name: landing
description: Use when explicitly invoking the flightdeck landing ritual — classifies new knowledge from the session, updates cockpit.md, blocks on hanging tasks, runs a lightweight stray/orphan workspace smoke-check, optionally commits. Triggered by `/flightdeck:landing`.
disable-model-invocation: true
---

# Flightdeck Landing

User-triggered explicit landing ritual. Thin entry-point that runs the [exit-ritual.md](../workflow/exit-ritual.md) decision tree as a one-command slash. Use for:

- Wrapping up a session cleanly before context compression.
- Natural pause point (ship complete / brainstorm done) — closing checks before moving on.
- Re-running mid-session to enforce the "no junk kneeboard / no hanging safety-review" discipline.

## Run this checklist

The full rules + rationale live in [exit-ritual.md](../workflow/exit-ritual.md). Skeleton:

1. **Resolve hanging tasks first** — incomplete safety-review dispositions and stale `kneeboard/` files block clean exit. See [exit-ritual.md § Hanging tasks](../workflow/exit-ritual.md#hanging-tasks--block-session-exit). If one is genuinely blocking, list it and pause for the user before running steps 2–7.
2. **Classify new knowledge** — apply heuristics (a)–(h), first-match wins. See [exit-ritual.md § Classification heuristics](../workflow/exit-ritual.md#classification-heuristics). No new knowledge is a valid outcome — don't manufacture a classification just to complete landing.
3. **Update `cockpit.md`** — only bump `Last updated` on the 4 sanctioned triggers. See [exit-ritual.md § Board update](../workflow/exit-ritual.md#board-update--what-changes). Then run the **Length check** (below) right away, before steps 4–7 — so the trim is reflected before AGENTS.md regen and commit, not after.
4. **Apply lifecycle transitions** — for each spec/flight-plan touched, decide state. See [workflow/SKILL.md § Lifecycle of specs and plans](../workflow/SKILL.md#lifecycle-of-specs-and-plans).
5. **Regenerate `AGENTS.md` if the source changed** — if any field AGENTS.md renders changed this session (`cockpit.md`: `Last updated` / `Active focus` / `Next session` / `Hanging tasks`; `manifest.md`: `In flight`), run `/flightdeck:emit-agents-md` so the cross-tool bridge file stays current. Judge "changed" against the file's state at session start, not an empty baseline. See [emit-agents-md SKILL.md](../emit-agents-md/SKILL.md).
6. **Workspace smoke-check (lightweight, non-blocking)** — before committing, scan for files this session added/left in `flightdeck/` that would drift the workspace (use `git status --short` to spot what's new or modified). Report, do not block:
   - **Stray root file**: any `.md` directly under `flightdeck/` that is not an entry file (`cockpit.md` / `manifest.md` / `logbook.md` / `INDEX.md`) → flag "stray root file; classify into a folder or remove".
   - **Orphan / unreachable**: any non-entry `.md` not reachable from an entry (`cockpit.md` / `INDEX.md` / `manifest.md` / a bundle `README.md` — bundle leaves are reachable via the README's `reading_order`) → flag "orphan; link from an entry or remove". Skip `landed/` and `kneeboard/`.
   - **Missing frontmatter**: a new flat file in `checklists/` or `incident-reports/` (not a bundle leaf) lacking `when_to_read` / `applies_to` / `last_updated` → flag.
   This is a smoke-check at write-time, not the full audit. For the complete sweep (bundle contracts, manifest↔folder mismatch, stale Blockers, AGENTS.md drift, etc.) run `/flightdeck:walkaround`. Surface any hit **before** the commit prompt so junk isn't committed; the user decides whether to fix now or proceed.
7. **Commit (if user wants)** — ask before; default to `checklists/commits.md` style if it exists; otherwise terse imperative subject + reasoning in body.

## Length check (runs right after step 3)

If `flightdeck/cockpit.md` > 80 lines: propose a trim. The fix is to **move** the overflow — historical / contextual sections that belong in `logbook.md` or `manifest.md` — not to delete it; confirm with the user before removing anything from cockpit. If `logbook.md` > 300 lines via `Recently finished`, cap to 5 entries with ≤ 3-line summaries; older bodies live in git log / archived flight-plans.

## Output format

```
Hanging tasks: none / [resolved X / blocking on Y]
New knowledge classified:
  - incident-reports/ +1: <file>
  - checklists/ +0 (no triggers)
  - (etc.)
Cockpit updated:
  - Last updated: [yes/no, reason]
  - Next session: [refreshed / unchanged]
Logbook updated:
  - Recently finished: [+1 entry / unchanged]
Lifecycle transitions: [list / none]
Workspace smoke-check: clean / [stray: X | orphan: Y | missing-frontmatter: Z]  (run /flightdeck:walkaround for full audit)

Commit now? (Y/n)
```

## Red flags

If you find yourself doing any of these, STOP and re-read [exit-ritual.md § Red flags](../workflow/exit-ritual.md#red-flags--stop):

- Brainstorming where every knowledge item belongs (heuristics catch 90%; default-brainstorm is the failure mode)
- Saving session logs / debug dumps to `flightdeck/` — transient byproducts, not knowledge; DO NOT WRITE
- Bumping `Last updated` after a typo fix or pure exploration
- Leaving `kneeboard/` files for "next session" (flightdeck rule: kneeboard survives one session by definition)
