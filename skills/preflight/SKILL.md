---
name: preflight
description: Use when explicitly invoking the flightdeck entry ritual — reconciles cockpit.md against repo state, runs staleness check, and reports the first "next session" item. Triggered by `/flightdeck:preflight`.
disable-model-invocation: true
---

# Flightdeck Preflight

User-triggered explicit entry ritual. Same logic as the [flightdeck-workflow](../flightdeck-workflow/SKILL.md) skill's Entry Checklist section, exposed as a one-command slash for cases where:

- A long session has gone off the rails and you want to re-anchor on the board.
- The auto-loaded `workshop-workflow` didn't fire (e.g. fresh `workshop/` mid-session).
- The user wants a clean, predictable starting point before delegating to other skills.

## Run this checklist exactly

1. **Read `workshop/board.md`** — focus on `Last updated` and the "Next session" section.

2. **Reconcile against repo state.** Run these checks in parallel:
   - `git branch --show-current` — matches `Active focus`?
   - `git status --short` — does the first "Next session" item show up as in-progress files?
   - `git stash list` — any entries not mentioned in board?
   - `git log -1 --format=%cs` — is `Last updated` more than ~14 days behind the most recent commit?

3. **Surface stale `wip/` files.** For each file in `workshop/wip/` whose `last_touched:` frontmatter predates the current session's start (or is missing): report it to the user with the file path. Do NOT auto-delete or auto-classify. The user resolves at session-exit; this step exists so they enter the session aware of pending hygiene debt.

4. **Mismatch handling** — **always ask the user before acting**:
   - If branch differs: "Board says focus is X but branch is Y — which is current?"
   - If `git status` is clean but board says in-progress: "Board flags 'X in progress' but tree is clean — did it ship?"
   - If stash exists not in board: "Stash entry from <date> not on board — pick up, drop, or note?"
   - If board > 14 days stale: "Board last updated <date>, most recent commit <date>. Board may be stale — refresh first?"

5. **All reconciled → execute the first "Next session" item.**
   State the item back to the user in one sentence ("Executing: [item description]"), then proceed.

## Fallback when "Next session" is empty

Don't auto-start anything. Search in order, present candidates to user:

1. `workshop/plans/` (excluding `finish/`) — already broken down, immediately executable.
2. `workshop/specs/` (excluding `finish/`) — designed but no plan; ask "write plan now or execute directly?"
3. `workshop/sketches/` — unstarted ideas; ask which (if any) to promote.

Actively-implementing artifacts SHOULD already be in board's `In flight` table. If fallback finds something not on the board, flag as a board sync bug.

## Output format

Report concisely:

```
Board reconciled (Last updated: 2026-05-25; Active focus: <X>; tree clean)
Next session item #1: <item description>

Proceeding.
```

Or if blocked:

```
Reconcile flagged:
- <mismatch 1>
- <mismatch 2>

Resolve which?
```

## Don't do

- Don't auto-pick a fallback when `Next session` is empty — always ask.
- Don't bump `Last updated` from this skill — entry doesn't modify board.
- Don't load every scar/playbook upfront — they're routed by `applies_to` metadata when relevant tasks come up.
- Don't grep the codebase looking for "things to do" — the board is authoritative.

For deeper background on the workshop convention (folder semantics, scenario triggers during the session, write gate), see [workshop-workflow/SKILL.md](../workshop-workflow/SKILL.md).
