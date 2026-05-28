---
name: landing
description: Use when explicitly invoking the flightdeck landing ritual — classifies new knowledge from the session, updates cockpit.md, blocks on hanging tasks, optionally commits. Triggered by `/flightdeck:landing`.
disable-model-invocation: true
---

# Flightdeck Landing

User-triggered explicit landing ritual. Thin entry-point that runs the [exit-ritual.md](../flightdeck-workflow/exit-ritual.md) decision tree as a one-command slash. Use for:

- Wrapping up a session cleanly before context compression.
- Natural pause point (ship complete / brainstorm done) — closing checks before moving on.
- Re-running mid-session to enforce the "no junk wip / no hanging critique" discipline.

## Run this checklist

The full rules + rationale live in [exit-ritual.md](../flightdeck-workflow/exit-ritual.md). Skeleton:

1. **Resolve hanging tasks first** — incomplete critique dispositions and stale `kneeboard/` files block clean exit. See [exit-ritual.md § Hanging tasks](../flightdeck-workflow/exit-ritual.md#hanging-tasks--block-session-exit).
2. **Classify new knowledge** — apply heuristics (a)–(h), first-match wins. See [exit-ritual.md § Classification heuristics](../flightdeck-workflow/exit-ritual.md#classification-heuristics).
3. **Update `cockpit.md`** — only bump `Last updated` on the 4 sanctioned triggers. See [exit-ritual.md § Board update](../flightdeck-workflow/exit-ritual.md#board-update--what-changes).
4. **Apply lifecycle transitions** — for each spec/flight-plan touched, decide state. See [flightdeck-workflow/SKILL.md § Lifecycle of specs and plans](../flightdeck-workflow/SKILL.md#lifecycle-of-specs-and-plans).
5. **Regenerate `AGENTS.md` if `cockpit.md` changed** — if any of `Last updated`, `Active focus`, `Next session`, `In flight`, or `Hanging tasks` were updated this session, run `/flightdeck:emit-agents-md` so the cross-tool bridge file stays current. See [emit-agents-md SKILL.md](../emit-agents-md/SKILL.md).
6. **Commit (if user wants)** — ask before; default to `checklists/commits.md` style if it exists; otherwise terse imperative subject + reasoning in body.

## Length check before exit

If `flightdeck/cockpit.md` > 80 lines: trim immediately. Most common cause = historical / contextual sections that belong in `logbook.md` or `manifest.md`. If `logbook.md` > 300 lines via `Recently finished`, cap to 5 entries with ≤ 3-line summaries; older bodies live in git log / archived flight-plans.

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

Commit now? (Y/n)
```

## Red flags

If you find yourself doing any of these, STOP and re-read [exit-ritual.md § Red flags](../flightdeck-workflow/exit-ritual.md#red-flags--stop):

- Brainstorming where every knowledge item belongs (heuristics catch 90%; default-brainstorm is the failure mode)
- Saving session logs / debug dumps to `flightdeck/` (gate (g) — DO NOT WRITE)
- Bumping `Last updated` after a typo fix or pure exploration
- Leaving `kneeboard/` files for "next session" (flightdeck rule: kneeboard survives one session by definition)
