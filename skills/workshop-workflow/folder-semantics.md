# Folder semantics

Reference for every `workshop/` subdirectory: what it holds, naming convention, lifecycle, and links to related folders.

## Minimal vs full setup

Not every project needs every folder. Start with just `board.md`. Add folders **when the need appears**, not preemptively.

| Setup | What exists |
| --- | --- |
| **Minimal** | `workshop/board.md` only |
| **+ knowledge** | add `scars/` when first lesson is worth keeping |
| **+ design** | add `specs/` when first design doc is worth writing |
| **+ planning** | add `plans/` when first multi-step implementation is broken down |
| **+ procedures** | add `playbooks/` when a multi-step process is run a second time |
| **Full** | all 10 folders below |

Premature folder creation is an anti-pattern. Empty directories signal "this should be filled" and create pressure to write low-signal content.

## The 10 folders

```
workshop/
├── INDEX.md            # Quick lookup of subdir purposes + key files
├── board.md            # Current state — the only file that must exist
│
├── specs/              # Design docs (one design per file)
│   └── finish/         # Archived after the design is shipped
├── plans/              # Implementation plans (one plan per file)
│   └── finish/         # Archived after the plan is fully executed
│
├── playbooks/          # Procedures (commands + checklists + conventions)
├── scars/              # Lessons learned (mistakes worth not repeating)
├── reference/          # External material (competitor code, RFCs, articles)
│
├── sketches/           # Long-term ideas (not started; awaiting trigger)
├── critiques/          # External AI / reviewer feedback (raw + disposition)
└── wip/                # Session-scratch (short-lived, prune each exit)
```

## Folder details

### `board.md` — the current state

**The only required file.** Read first, updated last. Contains:
- `Last updated: YYYY-MM-DD by <who> (<one-line summary>)`
- `Active focus: <current main thread>`
- "Next session" section: 1-5 concrete items
- "In flight" / "Recently archived" (optional, last ~2 weeks)

Authority: project state lives here. If `board.md` and an old spec disagree, board wins.

Update rule: only "user-perceivable semantic progress" — not activity logs. Pure exploration / grep / typo-fix does not update.

### `specs/` — design docs

One `.md` per design topic. Hand-write, or pipe in a brainstorming/spec-writing skill's output if you use one.

Naming: `YYYY-MM-DD-<feature>-design.md`

After the spec ships → `mv specs/foo.md specs/finish/foo.md`. Archived specs lose to current state in [authority order](SKILL.md#authority-order-when-sources-disagree).

### `plans/` — implementation plans

One `.md` per multi-step task. Hand-write, or use a planning skill if you have one.

Naming: `YYYY-MM-DD-<feature>-plan.md`

After execution complete → `mv plans/foo.md plans/finish/foo.md`.

**Checkbox convention is not load-bearing for workshop**: a plan can use `- [ ]` task lists if the author finds them useful, but workshop does not track checkbox state. **Progress lives in `board.md`** (`In flight` lifecycle state + `Recently finished` entries) and the commit log — not in plan-internal checkboxes. Real-world usage shows checkboxes routinely go un-flipped without harming plan quality; treat them as optional notation, not a status mechanism. Prefer `## Phase N: <name>` headers + prose for structure.

### `playbooks/` — procedures

Multi-step processes worth running a second time. Format: command sequence + checklist + convention notes.

Naming: `<topic>.md` (no date prefix — playbooks are stable resources)

Examples: `verify.md` (test before commit), `re-fixture.md` (regenerate test fixtures), `release.md`.

Promotion rule: a process becomes a playbook the **second** time you run it. First time = ad-hoc. Second time = pattern.

**Frontmatter required**: `when_to_read` (one-line trigger) + `applies_to` (short keyword tags) + `last_updated` (YYYY-MM-DD). Same pattern as skill SKILL.md metadata — lets AI grep for relevance + judge staleness without loading the body. A playbook with no `when_to_read` is invisible to skill routing. See [templates.md#playbook](templates.md#playbook).

### `scars/` — lessons learned

Mistakes worth not repeating. Format strictly enforces useful root cause (template in [templates.md](templates.md#scar)).

Naming: `<topic>.md` (no date prefix — scars are reference, not log)

Recurrence rule: same scar happens again → **append `## [Case N]`** to existing file. Do not create a new file. Repeated recurrence (≥3 times or single severe case) → promote one-liner to your project agent rules.

**Frontmatter required**: `when_to_read` (one-line trigger) + `applies_to` (short keyword tags) + `last_updated` (bump on Case append or status flip). Lets AI grep for relevance + judge staleness instead of reading every scar file at session start (token waste). A scar with no `when_to_read` is invisible to skill routing.

### `reference/` — external material

External docs, competitor source code, RFCs, blog posts, etc. — kept here so the team has a single place for "where do I find that thing".

Naming: `<source>-<topic>.md` (e.g. `boltframe-shape-layer.md`, `rfc-6749.md`)

Lifecycle: prune when the external source becomes irrelevant. Do not delete reflexively — keep if you still might consult it.

### `sketches/` — long-term ideas

Unstarted ideas. Either grow into a spec (move to `specs/`) or sit. No status tracking.

Naming: `<topic>.md` (no date prefix — ideas are timeless until acted on)

Distinguish from `wip/`: sketches are **unstarted**. `wip/` is **in-progress and abandoned at session end**.

### `critiques/` — external review feedback

Raw feedback from reviewers (other AIs, colleagues) + your **disposition** (adopt / reject / defer).

Naming: `YYYY-MM-DD-<spec-or-topic>-<reviewer>.md`

Disposition rule: no critique can exist in `critiques/` without a disposition section. If disposition is incomplete, add a hanging task to `board.md` ("finish disposition of `<file>`") and do not close the session.

### `wip/` — session scratch

Short-lived scratch files: copy-pasted error output you'll refer to in 5 minutes, draft text you're shaping. Lives **one session**.

Naming: free-form. Date prefix optional.

**Exit cleanup rule**: at every session exit, any `wip/` file older than one session must be either classified into another folder or deleted. **No wip files survive overnight without an explicit decision.**

This is the most-violated rule. Default to deletion. The cost of deleting a useful note is far smaller than the cost of `wip/` slowly turning into a junk drawer.

**Enforcement (v0.6+)**: wip files require a `last_touched: YYYY-MM-DD` frontmatter field, and stale entries trigger an exit-blocking hard gate. See [templates.md#wip](templates.md#wip) for the full rules.

### `INDEX.md` — quick lookup

A scannable index of the workshop, especially of `scars/` and `playbooks/` (resource directories). One line per file with a hook.

Maintenance: AI maintains automated sections marked with `<!-- AUTO-START -->` ... `<!-- AUTO-END -->`. Outside the markers is hand-curated. See [templates.md](templates.md#indexmd).

## Future expansion slots (DO NOT CREATE PREEMPTIVELY)

These are placeholder concepts. Create them only when the project's actual usage demands them:

- `decisions/` — Architecture Decision Records (ADRs). Useful when a project has ≥ 3 cross-spec decisions worth tracing back. Until then, decisions live in spec / board / commit messages.
- `experiments/` — long-running data probes worth referencing across sessions (e.g., "the byte-level study of how AE rejects this header"). Until then, throwaway probes live in `tmp/` at the project root.

If you find yourself wanting one of these, note the need in `board.md` and discuss before creating.

## Naming convention table

| Folder | Filename pattern | Reason |
| --- | --- | --- |
| `specs/` | `YYYY-MM-DD-<feature>-design.md` | Date helps order by recency; designs are time-bound |
| `plans/` | `YYYY-MM-DD-<feature>-plan.md` | Same as specs |
| `playbooks/` | `<topic>.md` | Stable resource — date noise hurts |
| `scars/` | `<topic>.md` | Stable resource — date noise hurts |
| `sketches/` | `<topic>.md` | Ideas are timeless |
| `critiques/` | `YYYY-MM-DD-<spec>-<reviewer>.md` | Date + reviewer identify uniqueness |
| `reference/` | `<source>-<topic>.md` | External source is the key identifier |
| `wip/` | free-form | Short-lived, naming overhead unjustified |

## Anti-patterns

| Anti-pattern | Why it fails | Fix |
| --- | --- | --- |
| Empty subdirs created to "establish the convention" | Pressure to fill → low-signal writes | Minimal setup; add folders on demand |
| `sketches/` used as `wip/` | Half-finished work never gets pruned | `wip/` is one session; `sketches/` is unstarted |
| Same fact duplicated across folders | Drift → trust collapses | One authoritative source; others link |
| Scar files named `2026-05-23-bug.md` | Date noise; impossible to find by topic | Use `<topic>.md` |
| `tmp/` placed inside `workshop/` | Junk gets committed | `tmp/` lives at project root, gitignored |
