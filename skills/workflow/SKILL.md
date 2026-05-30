---
name: workflow
description: Use when a project has a flightdeck/ directory, when starting one, or when AI session context needs to survive across sessions
---

# workflow

## Core principle

`flightdeck/` is a directory convention organized by **when you read what** — a persistent workbench for AI-assisted coding. Write strictly: only content that changes future behavior, influences decisions, or gets referenced repeatedly.

## First-time setup (no `flightdeck/` exists)

If invoked in a project without a `flightdeck/` directory — typically because the user typed `/flightdeck:preflight` to bootstrap:

1. Ask: **"No `flightdeck/` here. Create one? (minimal: just `cockpit.md`)"**. Wait for confirmation.
2. If yes, short interview:
   - "Active focus — current main thread (5–15 words)?"
   - "First 'next session' item — one concrete action?"
3. Create `flightdeck/cockpit.md` from this template, substituting `<...>` with the answers + today's date:

   ```markdown
   # Cockpit — <project name>

   **Last updated**: <YYYY-MM-DD> by <user>
   **Active focus**: <from interview>

   ## Next session

   1. <from interview>

   ## Hanging tasks

   - (none)
   ```
4. Do NOT pre-create `incident-reports/`, `checklists/`, `specs/`, etc. — flightdeck's principle is "add folders when the need appears, not preemptively." `cockpit.md` alone is the minimal contract.
5. From the next session onward (Claude Code), the SessionStart hook auto-loads this skill whenever `flightdeck/` is detected — no need to re-invoke the slash.

Then proceed with the Entry checklist below.

## Entry checklist (run at session start)

1. Read `flightdeck/cockpit.md` — focus on `Last updated` and the "next session" section.
2. Reconcile against repo state. **Mismatch → ask user before acting.**
   - `Active focus` matches `git branch --show-current`?
   - The first "next session" item is reflected in `git status`?
   - Any `git stash list` entries not mentioned in cockpit?
   - Is `Last updated` more than ~14 days behind the most recent commit? → cockpit may be stale.
3. All reconciled → execute the first "next session" item.

**Fallback when `Next session` is empty**: search in this order, present candidates to user (do not auto-start):
1. `flight-plans/` (excluding `landed/`) — already broken down, immediately executable.
2. `specs/` (excluding `landed/`) — designed but no plan; ask "write plan now or execute directly?"
3. `sketches/` — unstarted ideas; ask which (if any) to promote.

Active flight-plans/specs should be in manifest's `In flight` table — if fallback finds something that isn't, that's a manifest sync bug; flag to user.

## During — scenario triggers

| Scenario | Read first |
| --- | --- |
| Looking for next task | `cockpit.md` |
| Unsure about architecture | `specs/` |
| Running tests / preparing commit | `checklists/` |
| Strange behavior / deja-vu bug | `incident-reports/` |
| Need outside perspective | `charts/` + `safety-reviews/` |
| Designing new feature | `specs/` |
| Breaking work into tasks | `flight-plans/` |

**How to pick the right incident report / checklist**: don't read every file. Both folders use frontmatter (`when_to_read` + `applies_to` + `last_updated`) — grep the metadata, only load full files whose triggers match the current task. Use `last_updated` to judge staleness. An optional `skip_when` field (negative routing — "when NOT to read this") lets a file pre-empt a false match; absent is fine.

### Frontmatter requirements (hard-fail)

Incident reports and checklists **MUST** carry frontmatter with `when_to_read`, `applies_to`, and `last_updated`. On a missing field: STOP, report the file path + missing fields to user, offer (a) add now or (b) delete the file. Silent skip = files invisible while their authors believe they're active. That is the worst failure mode for an advice system.

### Proactive incident report resurfacing

Before starting a task whose description / file paths overlap with an incident report's `applies_to` tags, surface it: "this touches `[tags]`, overlapping with [incident-reports/X.md](incident-reports/X.md) — worth a read first?". Do NOT auto-increment `[Case N]` — that only happens on a real, user-confirmed recurrence.

## Authority order (when sources disagree)

Project agent rules > `cockpit.md` (≡ `manifest.md` ≡ `logbook.md`) > active `flight-plans/` > active `specs/` > `checklists/` > `incident-reports/` > `landed/`

The three new files are **peers** (they describe different facets: cockpit = what to do, manifest = what's open, logbook = what happened), so they share one rung. In practice, only `cockpit.md` carries authority over operational state. `manifest.md` is an index of state divergences. `logbook.md` is immutable-ish history. The peer grouping reflects that they don't compete with each other.

> **"Project agent rules"** = your project's top-level AI instructions file — whatever your AI tool reads on every session.

## Design philosophy

> **Semantic clarity outranks thematic consistency.**

When naming or structuring decisions trigger a conflict between "fits the aviation metaphor" and "reads correctly", clarity wins. The flightdeck metaphor is used because it sharpens operational intent — *not* as a theme to be applied uniformly. Two folders (`specs/`, `sketches/`) intentionally use neutral names because no aviation equivalent improves them. Future concepts face the same test.

Reject:
- aviation roleplay / sci-fi theming / meme interfaces / gamified agent cosplay
- "cute but unclear" terms (e.g., `/stuck → /request-vector` was rejected during rebrand — `/stuck` already reads correctly)
- forcing every new term into the metaphor

## Lifecycle of specs and plans

Specs / flight-plans pass through 6 states. **Default state is inferred from file location** — flightdeck only asks you to maintain explicit state when the truth diverges from location.

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

    note right of Pending: ⚪ specs/*.md (not in landed/)
    note right of InProgress: 🟡 flight-plans/*.md (not in landed/)
    note right of AwaitingReview: 🔵 state: awaiting-review
    note right of Done: ✅ landed/*/*.md
    note right of Blocked: 🔴 state: blocked
    note right of Scrapped: 🗑️ state: scrapped
```

**Implicit (no annotation needed)**:
- ⚪ `specs/*.md` not in `landed/` — pending
- 🟡 `flight-plans/*.md` not in `landed/` — in progress
- ✅ files under `landed/` — done

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

**Manifest `In flight` table shows ONLY rows where state diverges from location** — i.e., rows with an explicit `state:` value. Implicit-state artifacts don't need a row; their folder location is enough.

**Review owner — name one when state goes `awaiting-review`**:
- **User test** — needs human validation. Add hanging task.
- **Self review** — AI runs structured review (spawn `code-reviewer` subagent or re-read diff with fresh eyes).
- **External AI critique** — paste diff to another LLM; save to `safety-reviews/` with disposition.

A `state: awaiting-review` artifact without a named owner is a hanging task — blocks exit.

**Short-circuit (spec → direct execute, no plan)** is fine when spec is small. Big multi-phase specs always pass through a plan.

## Exit ritual

90% of exits are obvious — classify and write directly. Only truly ambiguous items invoke brainstorming.

Heuristics (first match wins):
- Bug + root cause → `incident-reports/`
- "Every time we do X, follow these steps" → `checklists/`
- One-off log / debug session → DO NOT WRITE
- Spans multiple folders, no clear primary → brainstorm with user

After classifying: update `cockpit.md` (`Last updated` + "next session") → commit.

Details: [exit-ritual.md](exit-ritual.md).

## Write gate

`flightdeck/` records only content that **changes future behavior, influences decisions, or gets referenced repeatedly**. Session byproducts, debug logs, and chat play-by-plays do not qualify. Gate strictly.

## Folder map

11 folders + 3 entry files, all optional except `cockpit.md`. See [folder-semantics.md](folder-semantics.md) for the list and naming conventions.

Minimal: just `flightdeck/cockpit.md`. Add folders as the need appears.

**Which folder?** Classify by lifecycle: uncommitted idea → `sketches/`; a design to build then archive → `specs/`; evergreen operational reference / standard / checklist → `checklists/`; imported external material → `charts/`. The common mistake is filing an evergreen reference under `specs/` — a spec is a design you ship and archive, not a standing reference.

**Routing is graph-based, not filesystem-based.** A file is "active" only if reachable from an entry (`cockpit.md`, `INDEX.md`, `manifest.md`, or a bundle `README.md`). **A file nothing links to effectively does not exist** — no session reads it. Custom folders / root files are allowed but MUST be reachable from an entry, or they are orphans.

**Bundles** — when one topic needs several files, make a subfolder with a `README.md` router carrying `bundle: true` + `reading_order` + routing frontmatter (`when_to_read` / `applies_to` / `last_updated`). Detail leaves carry NO routing fields and inherit the README's; the README's `reading_order` is the routing edge that makes leaves reachable. One routing boundary per bundle (no nesting). Detail in [folder-semantics.md](folder-semantics.md#bundles-multi-file-topics).

## Templates

See [templates.md](templates.md) for `incident-report` / `checklist` / `sketch` / `safety-review` / `kneeboard` / `cockpit.md` / `manifest.md` / `logbook.md` / `INDEX.md` templates.

## Relation to project agent rules

| | Project agent rules | `flightdeck/` |
| --- | --- | --- |
| Loaded | every session | on demand |
| Contains | rules + trigger table + style | state + knowledge + history |
| When in conflict | wins | yields |

## Incident report promotion gates

Multi-criterion gate evaluated by `landing`. An incident report reaches the **checklist promotion gate** when ALL three hold:
1. `[Case N] count ≥ 3` in the incident report file.
2. Cases recurred across **≥ 2 distinct sessions** (same-session triple-hits don't count).
3. Remediation pattern is **stable across cases** (the "next time avoid X" rule reads similarly across all cases — not 3 unrelated fixes papering over one symptom).

When the gate fires, `landing` prompts: "Promote `incident-reports/X.md` to `checklists/X.md`?". User confirms — promotion is **never automatic**.

A separate **project-rules upgrade gate** fires when a promoted incident report continues to recur after promotion. Then add a one-liner to project agent rules and mark the incident report `Status: upgraded → project rules`. Do not delete the incident report.

## Common mistakes — STOP and reclassify

| Mistake | Fix |
| --- | --- |
| Same fact in cockpit + incident-report + spec | One authoritative source; others link via `[name](incident-reports/X.md)` |
| `sketches/` used as `kneeboard/` (piles up) | `kneeboard/` is short-lived; `sketches/` is unstarted ideas |
| `kneeboard/` file without `last_touched:` frontmatter | Required field. Add it or delete the file. |
| `kneeboard/` files older than current session | Stale kneeboard blocks landing. Classify, delete, or add `defer_reason:`. See [templates.md § kneeboard](templates.md#kneeboard). |
| `incident-reports/` writes "forgot / careless" | Root cause must be a wrong assumption / wrong model / wrong process. |
| `safety-reviews/` paste-only, no disposition | Disposition required (adopt / reject / defer). No disposition = hanging task. |
| Brainstorming where every knowledge item belongs | Heuristics catch 90%. Default-brainstorm is the failure mode. |
| `tmp/` written into flightdeck | `tmp/` lives at project root, gitignored. |
| Cockpit > 80 lines | Trim immediately — move historical / contextual content to `logbook.md` or `manifest.md`. |
| `Recently finished` > 5 entries | Landing MUST auto-trim. If you see > 5, the trim step was skipped — drop oldest until count = 5. |
| Bumping `Last updated` on every commit / typo / grep | Signal pollution. Only bump on 4 triggers in exit-ritual.md `Board update`. |
| Incident report / checklist without required frontmatter | STOP, report file path + missing fields. Add or delete before proceeding. |
| Incident report / checklist with `last_updated` > 1 year in a fast-moving project | Likely stale advice. Bump after re-verifying or flip to `status: obsolete`. |
| "Save in case it's useful later" | No. Gate strictly. |
| "I'll fill the safety-review disposition next session" | No. Hanging task now. |

## Cross-references

The flightdeck convention describes WHAT to write and WHERE; the tool that produces the content is up to you (hand-write, use any AI skill, or ad-hoc LLM).

**Optional companions** (Claude Code with the `superpowers` plugin installed):
- `superpowers:brainstorming` → produces well-structured specs that fit `specs/`.
- `superpowers:writing-plans` → produces task lists that fit `flight-plans/`. Plan files use `- [ ]` checkboxes for executing-plans tracking; **flightdeck does not require flipping these** — progress lives in `cockpit.md` + commit log.

These are convenient but **not required** — flightdeck accepts content from any source.

The `flightdeck/` directory structure is **tool-agnostic** — any AI assistant can follow these conventions via project-level instructions. See `adapters/` for per-tool install paths.
