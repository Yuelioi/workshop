---
name: preflight
description: Use when explicitly invoking the flightdeck entry ritual — reconciles cockpit.md against repo state, runs staleness check, and reports the first "next session" item. Triggered by `/flightdeck:preflight`.
disable-model-invocation: true
---

# Flightdeck Preflight

User-triggered explicit entry ritual. Same logic as the [flightdeck-workflow](../flightdeck-workflow/SKILL.md) skill's Entry Checklist section, exposed as a one-command slash for cases where:

- A long session has gone off the rails and you want to re-anchor on the cockpit.
- The auto-loaded `flightdeck-workflow` didn't fire (e.g. fresh `flightdeck/` mid-session).
- The user wants a clean, predictable starting point before delegating to other skills.

## Run this checklist exactly

1. **Read `flightdeck/cockpit.md`** — focus on `Last updated` and the "Next session" section.

2. **Reconcile against repo state.** Run these checks in parallel:
   - `git branch --show-current` — matches `Active focus`?
   - `git status --short` — does the first "Next session" item show up as in-progress files?
   - `git stash list` — any entries not mentioned in cockpit?
   - `git log -1 --format=%cs` — is `Last updated` more than ~14 days behind the most recent commit?

3. **Surface stale `kneeboard/` files.** For each file in `flightdeck/kneeboard/` whose `last_touched:` frontmatter predates the current session's start (or is missing): report it to the user with the file path. Do NOT auto-delete or auto-classify. The user resolves at landing; this step exists so they enter the session aware of pending hygiene debt.

4. **Mismatch handling** — **always ask the user before acting**:
   - If branch differs: "Cockpit says focus is X but branch is Y — which is current?"
   - If `git status` is clean but cockpit says in-progress: "Cockpit flags 'X in progress' but tree is clean — did it ship?"
   - If stash exists not in cockpit: "Stash entry from <date> not on cockpit — pick up, drop, or note?"
   - If cockpit > 14 days stale: "Cockpit last updated <date>, most recent commit <date>. Cockpit may be stale — refresh first?"

5. **All reconciled → execute the first "Next session" item.**
   State the item back to the user in one sentence ("Executing: [item description]"), then proceed.

## Fallback when "Next session" is empty

Don't auto-start anything. Search in order, present candidates to user:

1. `flightdeck/flight-plans/` (excluding `landed/`) — already broken down, immediately executable.
2. `flightdeck/specs/` (excluding `landed/`) — designed but no plan; ask "write plan now or execute directly?"
3. `flightdeck/sketches/` — unstarted ideas; ask which (if any) to promote.

Actively-implementing artifacts SHOULD already be in manifest's `In flight` table. If fallback finds something not on the manifest, flag as a manifest sync bug.

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
- Don't bump `Last updated` from this skill — entry doesn't modify cockpit.
- Don't load every incident-report/checklist upfront — they're routed by `applies_to` metadata when relevant tasks come up.
- Don't grep the codebase looking for "things to do" — cockpit.md is authoritative.

For deeper background on the flightdeck convention (folder semantics, scenario triggers during the session, write gate), see [flightdeck-workflow/SKILL.md](../flightdeck-workflow/SKILL.md).
