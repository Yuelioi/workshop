---
name: flightdeck-workflow
description: Use when a project has a flightdeck/ directory, when starting one, or when AI session context needs to survive across sessions
---

# flightdeck-workflow

## Core principle

`workshop/` is a directory convention organized by **when you read what** — a persistent workbench for AI-assisted coding. Write strictly: only content that changes future behavior, influences decisions, or gets referenced repeatedly.

## First-time setup (no `workshop/` exists)

If invoked in a project without a `workshop/` directory — typically because the user typed `/workshop:workshop-workflow` to bootstrap:

1. Ask: **"No `workshop/` here. Create one? (minimal: just `board.md`)"**. Wait for confirmation.
2. If yes, short interview:
   - "Active focus — current main thread (5–15 words)?"
   - "First 'next session' item — one concrete action?"
3. Create `workshop/board.md` from this template, substituting `<...>` with the answers + today's date:

   ```markdown
   # Board — <project name>

   **Last updated**: <YYYY-MM-DD> by <user>
   **Active focus**: <from interview>

   ## Next session

   1. <from interview>

   ## In flight (only artifacts whose state diverges from folder location)

   | Artifact | State | Owner / Reason | Refs |
   | --- | --- | --- | --- |
   | _none_ | | | |

   ## Recently finished (cap 5, FIFO)

   - (none)

   ## Hanging tasks

   - (none)
   ```
4. Do NOT pre-create `scars/`, `playbooks/`, `specs/`, etc. — workshop's principle is "add folders when the need appears, not preemptively." `board.md` alone is the minimal contract.
5. From the next session onward (Claude Code), the SessionStart hook auto-loads this skill whenever `workshop/` is detected — no need to re-invoke the slash.

Then proceed with the Entry checklist below.

## Entry checklist (run at session start)

1. Read `workshop/board.md` — focus on `Last updated` and the "next session" section.
2. Reconcile against repo state. **Mismatch → ask user before acting.**
   - `Active focus` matches `git branch --show-current`?
   - The first "next session" item is reflected in `git status`?
   - Any `git stash list` entries not mentioned in board?
   - Is `Last updated` more than ~14 days behind the most recent commit? → board may be stale.
3. All reconciled → execute the first "next session" item.

**Fallback when `Next session` is empty**: search in this order, present candidates to user (do not auto-start):
1. `plans/` (excluding `finish/`) — already broken down, immediately executable.
2. `specs/` (excluding `finish/`) — designed but no plan; ask "write plan now or execute directly?"
3. `sketches/` — unstarted ideas; ask which (if any) to promote.

Active plans/specs should be in board's `In flight` table — if fallback finds something that isn't, that's a board sync bug; flag to user.

## During — scenario triggers

| Scenario | Read first |
| --- | --- |
| Looking for next task | `board.md` |
| Unsure about architecture | `specs/` |
| Running tests / preparing commit | `playbooks/` |
| Strange behavior / deja-vu bug | `scars/` |
| Need outside perspective | `reference/` + `critiques/` |
| Designing new feature | `specs/` |
| Breaking work into tasks | `plans/` |

**How to pick the right scar / playbook**: don't read every file. Both folders use frontmatter (`when_to_read` + `applies_to` + `last_updated`) — grep the metadata, only load full files whose triggers match the current task. Use `last_updated` to judge staleness.

### Frontmatter requirements (hard-fail)

Scars and playbooks **MUST** carry frontmatter with `when_to_read`, `applies_to`, and `last_updated`. On a missing field: STOP, report the file path + missing fields to user, offer (a) add now or (b) delete the file. Silent skip = files invisible while their authors believe they're active. That is the worst failure mode for an advice system.

### Proactive scar resurfacing

Before starting a task whose description / file paths overlap with a scar's `applies_to` tags, surface the scar: "this touches `[tags]`, overlapping with [scars/X.md](scars/X.md) — worth a read first?". Do NOT auto-increment `[Case N]` — that only happens on a real, user-confirmed recurrence.

## Authority order (when sources disagree)

Project agent rules > `board.md` > active `plans/` > active `specs/` > `playbooks/` > `scars/` > archived (`*/finish/`)

> **"Project agent rules"** = your project's top-level AI instructions file — whatever your AI tool reads on every session.

## Lifecycle of specs and plans

Specs / plans pass through 6 states. **Default state is inferred from file location** — workshop only asks you to maintain explicit state when the truth diverges from location.

```mermaid
stateDiagram-v2
    [*] --> Pending: new spec written
    Pending --> InProgress: plan written
    Pending --> Done: short-circuit (small spec, direct execute)
    InProgress --> AwaitingReview: impl done, review pending
    AwaitingReview --> Done: review passed
    Pending --> Blocked: external dep
    InProgress --> Blocked: external dep
    AwaitingReview --> Blocked: review external
    Blocked --> InProgress: unblocked
    Blocked --> AwaitingReview: unblocked
    Pending --> Scrapped: abandoned
    InProgress --> Scrapped: abandoned
    AwaitingReview --> Scrapped: rejected
    Blocked --> Scrapped: abandoned
    Done --> [*]
    Scrapped --> [*]

    note right of Pending: ⚪ specs/*.md (not in finish/)
    note right of InProgress: 🟡 plans/*.md (not in finish/)
    note right of AwaitingReview: 🔵 state: awaiting-review
    note right of Done: ✅ */finish/*.md
    note right of Blocked: 🔴 state: blocked
    note right of Scrapped: 🗑️ state: scrapped
```

**Implicit (no annotation needed)**:
- ⚪ `specs/*.md` not in `finish/` — pending
- 🟡 `plans/*.md` not in `finish/` — in progress
- ✅ files under `*/finish/` — done

**Explicit** — set frontmatter `state:` only for divergent states:

```yaml
---
state: blocked          # waiting on external decision / input
# or
state: awaiting-review  # implementation done, review not yet passed
# or
state: scrapped         # abandoned; will not ship
---
```

**Board `In flight` table shows ONLY rows where state diverges from location** — i.e., rows with an explicit `state:` value. Implicit-state artifacts don't need a row; their folder location is enough.

**Review owner — name one when state goes `awaiting-review`**:
- **User test** — needs human validation. Add hanging task.
- **Self review** — AI runs structured review (spawn `code-reviewer` subagent or re-read diff with fresh eyes).
- **External AI critique** — paste diff to another LLM; save to `critiques/` with disposition.

A `state: awaiting-review` artifact without a named owner is a hanging task — blocks exit.

**Short-circuit (spec → direct execute, no plan)** is fine when spec is small. Big multi-phase specs always pass through a plan.

## Exit ritual

90% of exits are obvious — classify and write directly. Only truly ambiguous items invoke brainstorming.

Heuristics (first match wins):
- Bug + root cause → `scars/`
- "Every time we do X, follow these steps" → `playbooks/`
- One-off log / debug session → DO NOT WRITE
- Spans multiple folders, no clear primary → brainstorm with user

After classifying: update `board.md` (`Last updated` + "next session") → commit.

Details: [exit-ritual.md](exit-ritual.md).

## Write gate

`workshop/` records only content that **changes future behavior, influences decisions, or gets referenced repeatedly**. Session byproducts, debug logs, and chat play-by-plays do not qualify. Gate strictly.

## Folder map

10 folders, all optional except `board.md`. See [folder-semantics.md](folder-semantics.md) for the list and naming conventions.

Minimal: just `workshop/board.md`. Add folders as the need appears.

## Templates

See [templates.md](templates.md) for `scar` / `playbook` / `sketch` / `critique` / `wip` / `board` / `INDEX.md` templates.

## Relation to project agent rules

| | Project agent rules | `workshop/` |
| --- | --- | --- |
| Loaded | every session | on demand |
| Contains | rules + trigger table + style | state + knowledge + history |
| When in conflict | wins | yields |

## Scar promotion gates

Multi-criterion gate evaluated by `session-exit`. A scar reaches the **playbook promotion gate** when ALL three hold:
1. `[Case N] count ≥ 3` in the scar file.
2. Cases recurred across **≥ 2 distinct sessions** (same-session triple-hits don't count).
3. Remediation pattern is **stable across cases** (the "next time avoid X" rule reads similarly across all cases — not 3 unrelated fixes papering over one symptom).

When the gate fires, `session-exit` prompts: "Promote `scars/X.md` to `playbooks/X.md`?". User confirms — promotion is **never automatic**.

A separate **project-rules upgrade gate** fires when a promoted scar continues to recur after promotion. Then add a one-liner to project agent rules and mark the scar `Status: upgraded → project rules`. Do not delete the scar.

## Common mistakes — STOP and reclassify

| Mistake | Fix |
| --- | --- |
| Same fact in board + scar + spec | One authoritative source; others link via `[name](scars/X.md)` |
| `sketches/` used as `wip/` (piles up) | `wip/` is short-lived; `sketches/` is unstarted ideas |
| `wip/` file without `last_touched:` frontmatter | Required field. Add it or delete the file. |
| `wip/` files older than current session | Stale wip blocks session-exit. Classify, delete, or add `defer_reason:`. See [templates.md § wip](templates.md#wip). |
| `scars/` writes "forgot / careless" | Root cause must be a wrong assumption / wrong model / wrong process. |
| `critiques/` paste-only, no disposition | Disposition required (adopt / reject / defer). No disposition = hanging task. |
| Brainstorming where every knowledge item belongs | Heuristics catch 90%. Default-brainstorm is the failure mode. |
| `tmp/` written into workshop | `tmp/` lives at project root, gitignored. |
| Board > 300 lines | Trim `Recently finished` to 5 entries, ≤ 3-line summaries. |
| `Recently finished` > 5 entries | Session-exit MUST auto-trim. If you see > 5, the trim step was skipped — drop oldest until count = 5. |
| Bumping `Last updated` on every commit / typo / grep | Signal pollution. Only bump on 4 triggers in exit-ritual.md `Board update`. |
| Scar / playbook without required frontmatter | STOP, report file path + missing fields. Add or delete before proceeding. |
| Scar / playbook with `last_updated` > 1 year in a fast-moving project | Likely stale advice. Bump after re-verifying or flip to `status: obsolete`. |
| "Save in case it's useful later" | No. Gate strictly. |
| "I'll fill the critique disposition next session" | No. Hanging task now. |

## Cross-references

The workshop convention describes WHAT to write and WHERE; the tool that produces the content is up to you (hand-write, use any AI skill, or ad-hoc LLM).

**Optional companions** (Claude Code with the `superpowers` plugin installed):
- `superpowers:brainstorming` → produces well-structured specs that fit `specs/`.
- `superpowers:writing-plans` → produces task lists that fit `plans/`. Plan files use `- [ ]` checkboxes for executing-plans tracking; **workshop does not require flipping these** — progress lives in `board.md` + commit log.

These are convenient but **not required** — workshop accepts content from any source.

The `workshop/` directory structure is **tool-agnostic** — any AI assistant can follow these conventions via project-level instructions. See `adapters/` for per-tool install paths.
