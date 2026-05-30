# Templates

Reusable file templates for `flightdeck/` files. Each template has a strict structure — deviation typically means the file should live in a different folder or be deleted.

---

## incident-report

```markdown
---
status: active   # active | obsolete | superseded
since: YYYY-MM-DD
last_updated: YYYY-MM-DD   # bump on every [Case N] append or status flip
when_to_read: <one-line trigger — "when AI is about to do X, check this incident report first">
applies_to: [<keyword>, <keyword>, ...]   # short tags AI can grep
# Optional — Cursor MDC interop (only useful if scoping by file paths matters):
# globs: "src/parser/**/*.go, src/lexer/**/*.go"   # comma-separated patterns
# alwaysApply: false                                # default false; true loads on every Cursor session
---

# <one-line topic>

**Symptom**: How the user / test / build actually observed it. Error text verbatim.

**Root cause** (FORBIDDEN: "forgot", "careless", "didn't notice" — must be a wrong assumption / wrong model / wrong process):
I assumed X, but in reality Y.

**Lesson**: The specific next-time action. Not "be careful". Concrete behavior or check.

---

## [Case 2]   ← Appended on recurrence. DO NOT create a new file.

**Symptom**: ...
**Root cause**: ...
**Lesson**: ...
```

### Rules

- **One file per topic.** Recurrences append `## [Case N]`.
- **Forbidden root causes**: "forgot", "careless", "didn't notice", "rushed". These hide the real model error.
- **Status field**:
  - `active` — still applies to the current codebase
  - `obsolete` — the underlying constraint no longer exists (framework upgraded, code removed). Keep the file as history but mark.
  - `superseded` — folded into your project agent rules. Note the upgrade: `status: superseded → project-rules §<section>`. Do not delete.
- **Promotion path**: incident reports promote in two stages — first to `checklists/` (after a 3-criterion gate at `landing`), then to project agent rules (only if the checklist is also ignored and the incident continues to recur). Full gate criteria + workflow in [workflow/SKILL.md § Incident report promotion gates](../workflow/SKILL.md#incident-report-promotion-gates).
- **Frontmatter `when_to_read` + `applies_to` are REQUIRED** (not optional). An incident report without them fails the workflow routing check and is reported as a hanging task. They let AI grep for relevance without loading the full file — same pattern as skill SKILL.md `description`. Examples:
  - `when_to_read: "before designing a recursive parser"` / `applies_to: [parser, recursion, stack-depth]`
  - `when_to_read: "before adding a new migration"` / `applies_to: [migration, schema, postgres]`
  - Keep tags **short and concrete** — `[parser, recursion]` beats `[code-quality, architecture]`. Generic tags don't help AI choose.
- **Frontmatter `last_updated`**: bump on every meaningful change (Case append / status flip / advice rewrite). Lets AI judge staleness: a `last_updated` 2 years ago about a removed module is probably obsolete — promote to `status: obsolete` or delete. Lets users sort by recency when triaging.
- **Optional Cursor MDC interop fields** (`globs:` + `alwaysApply:`): include these if you want non-Claude AI tools that read `.cursor/rules/*.mdc`-style frontmatter to scope-load the incident report by file path. Flightdeck's own routing uses `when_to_read` + `applies_to` — these MDC fields are purely a bridge for tools that don't honor flightdeck's SessionStart hook. Skip if not relevant to your toolchain.

---

## checklist

```markdown
---
last_updated: YYYY-MM-DD   # bump every time the checklist content actually changes
when_to_read: <one-line trigger — "before doing X, follow these steps">
applies_to: [<keyword>, <keyword>, ...]   # short tags AI can grep
# Optional — Cursor MDC interop (only useful if scoping by file paths matters):
# globs: "release/**, Makefile, scripts/release/**"   # comma-separated patterns
# alwaysApply: false                                  # default false; true loads on every Cursor session
---

# <topic> checklist

## When to follow this

<2-3 line description of the situation this checklist handles>

## Steps

1. <command or check>
2. <command or check>
3. ...

## Verification

- <how to confirm each step worked>

## Common pitfalls

- <known trap and how to avoid it>
```

### Rules

- **One file per topic** (e.g. `verify.md`, `release.md`, `re-fixture.md`).
- **Frontmatter `when_to_read` + `applies_to` are REQUIRED** (not optional). A checklist without them fails the workflow routing check — same hard-fail rule as incident reports. See `workflow/SKILL.md § Frontmatter requirements`.
- **Frontmatter `last_updated`**: bump every time the checklist content actually changes (not for typo fixes). Lets AI / users judge staleness: a build checklist last touched 2 years ago in a fast-moving project is suspect.
- **Promotion rule**: a process becomes a checklist **on the second occurrence**. First time is ad-hoc; second time is the pattern worth recording.
- **No date prefix** — checklists are stable resources, not log entries.
- **Optional Cursor MDC interop fields** (`globs:` + `alwaysApply:`): same as incident reports — include if you want tools reading `.cursor/rules/*.mdc`-style frontmatter to scope-load the checklist by file path. Flightdeck routing uses `when_to_read` + `applies_to`; the MDC fields are an interop bridge for non-Claude tools.

---

## sketch

```markdown
# <idea title>

One-line gist of the idea.

**Trigger**: Why this came up now / what pain it solves.
**Related**: Other spec / plan / incident report this connects to (optional).
**Revisit when**: The condition under which this is worth re-evaluating (optional).
```

### Rules

- No status field — sketches are either acted on (move to `specs/`) or sit.
- If the sketch has been sitting > 6 months and no "revisit when" condition has triggered, consider deletion. An idea that never finds its moment is not high-signal.

---

## kneeboard

```markdown
---
last_touched: YYYY-MM-DD   # ISO date the file was last meaningfully edited
---

# <one-line scratch topic>

<Free-form scratch — debug notes, partial drafts, intermediate thinking.
Whatever the session needs that doesn't yet fit a permanent home.>
```

### Rules

- **`last_touched` is REQUIRED.** Update it every time you meaningfully edit the file. `preflight` reads this to flag stale kneeboard files.
- **Kneeboard file survives one session.** A `last_touched` predating the current session's start means the file is stale. `preflight` surfaces stale kneeboard files; `landing` BLOCKS until each stale kneeboard file is classified, deleted, or has a written defer reason.
- **No filename convention required.** Use whatever scratch identifier helps (`debug-notes.md`, `gpt-paste`, `temp-spec-draft`). Short.
- **Defer reason example**: if you genuinely need a kneeboard file carried across sessions, add a `defer_reason:` frontmatter field with a 1-line justification (e.g. `defer_reason: investigation paused on user vacation; resume 2026-06-15`). The defer reason makes the carry explicit and surfaceable.

### Pre-write checklist (hard gate)

Before creating any new `kneeboard/` file, the author (human or AI) must answer:

1. **Will this content survive past this session?**
   - **Yes** → STOP. kneeboard is the wrong destination. The right home is probably `incident-reports/` (bug + root cause), `checklists/` (repeated procedure), `specs/` (design decision), `sketches/` (long-term idea), or `safety-reviews/` (external feedback). Classify directly; do NOT create the kneeboard file.
   - **No** → proceed to (2).

2. **What's the destination if it does survive past this session unexpectedly?** Pick one:
   - **"delete at landing"** — confirm intent. If this is genuine scratch, fine. The kneeboard file will be classified-or-deleted at landing per the v0.6 TTL hard gate.
   - **Named graduation target** — e.g., `defer_reason: this will graduate to incident-reports/ if a root cause emerges`. The defer reason makes intent visible across sessions.

If you cannot answer (1) without ambiguity, OR cannot answer (2) with a concrete choice: do **NOT** create the kneeboard file. Either classify the content directly now, or hold the thought without committing it to disk. kneeboard is short-lived scratch; ambiguity defeats that purpose. The checklist is prose discipline, not programmatic enforcement — but it exists so the author can't claim they didn't know.

---

## safety-review

```markdown
# <spec or topic> — <reviewer> review

**Date**: YYYY-MM-DD
**Reviewer**: <name / model>
**Reviewed**: [link to the spec or artifact](../specs/YYYY-MM-DD-foo-design.md)

## Raw feedback

<Reviewer's full text — pasted verbatim. If summarized from a conversation, mark "(paraphrased by user)" at top.>

## Disposition (per review point — every point must carry a tag)

1. **[adopt]** <review point in 1 line> — <what specifically to change in the spec / code>
2. **[reject]** <review point in 1 line> — <one-line why>
3. **[defer]** <review point in 1 line> — <link to follow-up cockpit item / sketch / new spec>
4. **[adopt]** <next point> — <change>
   ...

(Every numbered point ends with `[adopt]` / `[reject]` / `[defer]`. No bare points.)
```

### Rules

- **Every review point carries exactly one of `[adopt]` / `[reject]` / `[defer]`.** No bullet may exist without a tag. A bullet without a tag = the file is incomplete.
- **Completion gate**: a safety-review file is "complete" iff every numbered point has a tag. Incomplete file = exit-blocking hanging task in `cockpit.md` ("finish disposition of `safety-reviews/<file>`"). The hanging task does not clear until the file is complete.
- **`[defer]` must point somewhere**: link to a `cockpit.md` future-work item or a sketch. Defer without a target is silently lost — that's the failure mode the tag exists to prevent.
- **Long reviews (>1000 words)**: keep raw verbatim. Disposition section may grow long too — that's fine. The constraint is per-point tagging, not brevity.
- **Splitting one review point into multiple**: allowed (and encouraged when one bullet bundles `[adopt]` + `[defer]` halves). Just enumerate each as its own line.

---

## cockpit.md

```markdown
# Cockpit — <project>

**Last updated**: YYYY-MM-DD by <who> (<one-line state summary>)
**Active focus**: <current main thread, 5–15 words>

## Next session

1. <first concrete action — must be executable by reading-cockpit-only>
2. <second>
3. ...

## Hanging tasks

- [ ] Finish disposition of [safety-reviews/...](safety-reviews/...)
- [ ] Classify or delete [kneeboard/...](kneeboard/...)
```

### Rules

- **Length cap: 80 lines hard ceiling.** Past 80, cockpit has become a dump — trim immediately. Historical / contextual content belongs in `logbook.md` or `manifest.md`, not cockpit.
- **`Active focus` is current state**, not history. Update it as the focus shifts.
- **Hanging tasks block landing**: a session cannot be closed cleanly while a hanging task is open. Either resolve, or explicitly defer with a date.
- **No metric tracking duplicated elsewhere** (test pass counts, build status). Single authoritative source — cockpit references via link.

## manifest.md

```markdown
# Manifest — <project>

## In flight

| Artifact | State | Owner / Reason | Refs |
| --- | --- | --- | --- |
| _none_ | | | |

## Blockers

- <items waiting on external decision / answer>
```

### Rules

- **Shows only divergent state.** Artifacts at their implicit location-state (pending spec, in-progress flight-plan) need no row here. Only `state: blocked`, `state: awaiting-review`, or `state: scrapped` artifacts appear.
- **No hard ceiling.** Content is structurally bounded by the state-divergence rule.

## logbook.md

```markdown
# Logbook — <project>

## Recently finished

- <newest first; ~3-line summary per entry; link to commit / PR>
- <entry 2>
- <entry 3>
- <entry 4>
- <entry 5>
- ... (cap at 5 entries — when adding new, drop oldest)

## Deferred

- <items intentionally postponed; link to original source>
```

### Rules

- **`Recently finished` cap: 5 entries fixed (not by date).** Date-based caps drift — during atomic-PR weeks a "last 2 weeks" rule retains 30+ commits and bloats logbook past 500 lines. Fixed count is stable. When adding a new entry, drop the oldest. Anything older lives in `git log`.
- **Per-entry summary in `Recently finished` ≤ 3 lines.** If you want to write more, the artifact (commit message / archived flight-plan file) is the right place — link to it instead.

---

## INDEX.md

```markdown
# flightdeck/ index

Last regenerated: YYYY-MM-DD

## Reading order for a new contributor

1. [`cockpit.md`](cockpit.md) — current state and next actions
2. [`specs/`](specs/) — design context
3. [`incident-reports/`](incident-reports/) — known traps
4. [`checklists/`](checklists/) — how to do common operations

## Resource directories

### incident-reports/

<!-- AUTO-START: incident-reports -->
- [incident-topic-A](incident-reports/topic-a.md) — one-line hook
- [incident-topic-B](incident-reports/topic-b.md) — one-line hook
<!-- AUTO-END: incident-reports -->

### checklists/

<!-- AUTO-START: checklists -->
- [verify](checklists/verify.md) — pre-commit verification
- [re-fixture](checklists/re-fixture.md) — regenerate test fixtures
<!-- AUTO-END: checklists -->

### charts/

<!-- AUTO-START: charts -->
- [boltframe-shape-layer](charts/boltframe-shape-layer.md) — competitor parser
<!-- AUTO-END: charts -->

## Curated notes

Anything outside `<!-- AUTO-* -->` markers is hand-curated. AI must not modify it.
```

### Rules

- **AUTO sections**: AI rewrites between `<!-- AUTO-START: <name> -->` and `<!-- AUTO-END: <name> -->` markers. The AI must regenerate the entries from the actual files in that subdirectory.
- **Outside markers is hand-curated**: AI must not touch this content. The reading-order and curated-notes sections belong to the human.
- **Tool-agnostic protocol**: the AUTO marker is a documented convention. Any AI assistant supporting `flightdeck/` should honor it.
- INDEX.md is **optional**. Add it only when the flightdeck has enough `incident-reports/` and `checklists/` that scanning is no longer easy.

---

## Spec evolution markers (optional convention)

When amending a long-lived spec — especially **backlog specs** in `specs/` that gain items over multiple sessions, or **specs revised after safety-review disposition** (e.g., the roadmap revision after a GPT review) — mark new / modified / removed items with prefix tags so the change history is grep-able and merge-friendly:

- **`ADDED:`** — new item or section.
- **`MODIFIED:`** — existing item changed. Note the old + new state inline if the change isn't self-evident.
- **`REMOVED:`** — item dropped. Strike-through (`~~text~~`) or comment-out rather than deleting outright, so the audit trail survives.

Example from a revised backlog spec:

```
- ADDED: B7 — cache layer with TTL on read-heavy endpoints.
- MODIFIED: B3 — switched from polling to webhook (was: 5s poll loop).
- REMOVED: ~~B5 server-side rendering~~ (rejected after benchmarks; rejected approach noted in commit log).
```

### Rules

- **Optional.** Small one-shot specs (single-session, no safety-review round) don't need delta markers. The cost of adding them outweighs the benefit at that scale.
- **Apply only to substantive changes.** Typo fixes don't earn a marker; an item's scope shifting does.
- **REMOVED keeps history.** Strike-through preserves the audit trail; outright deletion makes it impossible to see "what we considered and rejected". The audit trail is the whole point.
- **Markers compose with `state:` frontmatter.** A spec can be `state: blocked` AND have an `ADDED:` line in its body. State applies to the artifact; markers apply to items within.

---

## Cross-folder reference syntax

When one file references another, use a markdown link with a one-word hook:

```markdown
Known trap: [v2-aelayer structure](incident-reports/v2-aelayer-structure.md)
Procedure: [verify before commit](checklists/verify.md)
Decision: [why we chose splice over rewrite](landed/specs/2025-12-01-write-strategy-design.md)
```

Why this matters:
- The reader (human or AI) can jump straight to the source of truth.
- Single authoritative location — no duplication.
- When the linked file moves, the broken link is visible and fixable.

**Forbidden**: pasting facts inline that exist elsewhere ("we use splice not rewrite because..."). Link, do not copy.
