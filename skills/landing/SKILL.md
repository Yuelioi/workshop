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

The full rules + rationale live in [exit-ritual.md](../workshop-workflow/exit-ritual.md). Skeleton:

1. **Resolve hanging tasks first** — incomplete critique dispositions and stale `wip/` files block clean exit. See [exit-ritual.md § Hanging tasks](../workshop-workflow/exit-ritual.md#hanging-tasks--block-session-exit).
2. **Classify new knowledge** — apply heuristics (a)–(h), first-match wins. See [exit-ritual.md § Classification heuristics](../workshop-workflow/exit-ritual.md#classification-heuristics).
3. **Update `board.md`** — only bump `Last updated` on the 4 sanctioned triggers. See [exit-ritual.md § Board update](../workshop-workflow/exit-ritual.md#board-update--what-changes).
4. **Apply lifecycle transitions** — for each spec/plan touched, decide state. See [workshop-workflow/SKILL.md § Lifecycle of specs and plans](../workshop-workflow/SKILL.md#lifecycle-of-specs-and-plans).
5. **Regenerate `AGENTS.md` if `board.md` changed** — if any of `Last updated`, `Active focus`, `Next session`, `In flight`, or `Hanging tasks` were updated this session, run `/workshop:emit-agents-md` so the cross-tool bridge file stays current. See [emit-agents-md SKILL.md](../emit-agents-md/SKILL.md).
6. **Commit (if user wants)** — ask before; default to `playbooks/commits.md` style if it exists; otherwise terse imperative subject + reasoning in body.

## Length check before exit

If `workshop/board.md` > 300 lines: trim. Most common cause = `Recently finished` accumulated long per-entry summaries. Cap to 5 entries with ≤ 3-line summaries; older bodies live in git log / archived plans.

## Output format

```
Hanging tasks: none / [resolved X / blocking on Y]
New knowledge classified:
  - scars/ +1: <file>
  - playbooks/ +0 (no triggers)
  - (etc.)
Board updated:
  - Last updated: [yes/no, reason]
  - Next session: [refreshed / unchanged]
  - Recently finished: [+1 entry / unchanged]
Lifecycle transitions: [list / none]

Commit now? (Y/n)
```

## Red flags

If you find yourself doing any of these, STOP and re-read [exit-ritual.md § Red flags](../workshop-workflow/exit-ritual.md#red-flags--stop):

- Brainstorming where every knowledge item belongs (heuristics catch 90%; default-brainstorm is the failure mode)
- Saving session logs / debug dumps to `workshop/` (gate (g) — DO NOT WRITE)
- Bumping `Last updated` after a typo fix or pure exploration
- Leaving `wip/` files for "next session" (workshop rule: wip survives one session by definition)
