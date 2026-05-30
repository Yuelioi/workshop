# Folder semantics

Reference for every `flightdeck/` subdirectory: what it holds, naming convention, lifecycle, and links to related folders.

## Minimal vs full setup

Not every project needs every folder. Start with just `cockpit.md`. Add folders **when the need appears**, not preemptively.

| Setup | What exists |
| --- | --- |
| **Minimal** | `flightdeck/cockpit.md` only |
| **+ knowledge** | add `incident-reports/` when first lesson is worth keeping |
| **+ design** | add `specs/` when first design doc is worth writing |
| **+ planning** | add `flight-plans/` when first multi-step implementation is broken down |
| **+ procedures** | add `checklists/` when a multi-step process is run a second time |
| **Full** | all 11 folders + 3 entry files below |

Premature folder creation is an anti-pattern. Empty directories signal "this should be filled" and create pressure to write low-signal content.

## Routing model

**Flightdeck is graph-routed, not filesystem-routed.** A file is "active" only if it is reachable from some entry — `cockpit.md`, `INDEX.md`, `manifest.md`, or any bundle `README.md`. **A file not reachable from any entry effectively does not exist** (no session will ever read it). This is the single most important property to preserve: a stray, well-written file that nothing links to is invisible, and AI can rarely detect that on its own. `walkaround` audits reachability.

Reachability edges are markdown links from an entry **plus** each bundle README's `reading_order` entries (a leaf listed in its README's `reading_order` is reachable — that list *is* the router, so leaves need not also be linked in prose to count as reached).

Discovery is recursive frontmatter grep (descends into subfolders) + this reachability graph. Custom folders / root files are allowed — flightdeck favors extensible conventions over a locked taxonomy — **but they must be reachable from an entry**, or `walkaround` flags them as orphans.

## Which folder? (decision table)

When unsure where something goes, classify by lifecycle:

| Kind | Lifecycle | Goes in |
| --- | --- | --- |
| Uncommitted idea | not started | `sketches/` |
| A design to implement | one-shot; archived after shipping | `specs/` |
| Long-lived operational reference / standard / checklist | evergreen | `checklists/` |
| Imported external material (others' code / RFCs / articles) | reference | `charts/` |

The common mistake is filing an evergreen reference under `specs/`. A spec is a *design you intend to build and then archive*; an evergreen standard you consult repeatedly is a `checklists/` reference. (`checklists/` = authored operational reference; `charts/` = imported external material — keep that split clear instead of adding a `references/` folder.)

## The 11 folders + 3 entry files

```
flightdeck/
├── INDEX.md            # Quick lookup of subdir purposes + key files
├── cockpit.md          # Must-read every session entry (≤80 lines)
├── manifest.md         # On-demand: In flight + Blockers
├── logbook.md          # Rarely read: Recently finished + Deferred
│
├── specs/              # Design docs (one design per file)
├── flight-plans/       # Implementation plans (one plan per file)
│
├── checklists/         # Operational reference (checklists, conventions, standards; may be bundles)
├── incident-reports/   # Lessons learned (mistakes worth not repeating)
├── charts/             # External material (competitor code, RFCs, articles)
│
├── sketches/           # Long-term ideas (not started; awaiting trigger)
├── safety-reviews/     # External AI / reviewer feedback (raw + disposition)
├── kneeboard/          # Session-scratch (short-lived, prune each landing)
│
└── landed/             # Archive umbrella (replaces */finish/)
    ├── flight-plans/   # Archived after the plan is fully executed
    └── specs/          # Archived after the design is shipped
```

## Entry files

### `cockpit.md` — must-read every session

**The only required file.** Read first, updated last. Hard ceiling: **80 lines**.

Contains:
- `Last updated: YYYY-MM-DD by <who> (<one-line summary>)`
- `Active focus: <current main thread>`
- "Next session" section: 1-5 concrete items
- "Hanging tasks" section: open items blocking clean landing

**Why split from the old board.md**: `board.md` carried 8 responsibilities ranging from "must read every session" to "almost never read", forcing a 300-line ceiling to fight growth pressure. The split separates by read-time. cockpit.md carries only the operational state needed at every session start.

The 80-line ceiling is not arbitrary aesthetics — it is cognitive-load engineering for the human + AI both reading cockpit on every session start. Treat the ceiling as a load-bearing design constraint, not a style guide.

Authority: project state lives here. If `cockpit.md` and an old spec disagree, cockpit wins.

Update rule: only "user-perceivable semantic progress" — not activity logs. Pure exploration / grep / typo-fix does not update.

### `manifest.md` — on-demand state tracker

Read when picking up flagged artifacts — not every session. No hard ceiling (content is structurally bounded by the state-divergence rule).

Contains:
- "In flight" table: **only** artifacts whose `state:` diverges from their folder location (`state: blocked`, `state: awaiting-review`, `state: scrapped`). Implicit-state artifacts (pending spec, in-progress flight-plan) need no row — their folder is enough.
- "Blockers" section: items waiting on external decision / answer.

A 20-task project with 18 flight-plans in `flight-plans/` does not need 18 manifest rows — they're all implicit 🟡.

### `logbook.md` — rarely read history

Read for retrospectives, release-notes prep, or "what shipped last quarter". Not a session-start file.

Contains:
- "Recently finished" section: FIFO cap 5. When adding a new entry, drop the oldest. Date-based caps drift — fixed count is stable. Per-entry summary ≤ 3 lines; longer bodies belong in the archived flight-plan file.
- "Deferred" section: items intentionally postponed, with link to original source.

**The three entry files are peers in the [authority order](SKILL.md#authority-order-when-sources-disagree)** — they describe different facets of project state (cockpit = what to do, manifest = what's open, logbook = what happened) and don't compete with each other.

## Folder details

### `specs/` — design docs

One `.md` per design topic. Hand-write, or pipe in a brainstorming/spec-writing skill's output if you use one.

Naming: `YYYY-MM-DD-<feature>-design.md`

After the spec ships → `mv specs/foo.md landed/specs/foo.md`. Archived specs lose to current state in [authority order](SKILL.md#authority-order-when-sources-disagree).

### `flight-plans/` — implementation plans

One `.md` per multi-step task. Hand-write, or use a planning skill if you have one.

Naming: `YYYY-MM-DD-<feature>-plan.md`

After execution complete → `mv flight-plans/foo.md landed/flight-plans/foo.md`.

**Checkbox convention is not load-bearing for flightdeck**: a flight-plan can use `- [ ]` task lists if the author finds them useful, but flightdeck does not track checkbox state. **Progress lives in `cockpit.md`** (`In flight` lifecycle state + `logbook.md` `Recently finished` entries) and the commit log — not in plan-internal checkboxes. Real-world usage shows checkboxes routinely go un-flipped without harming plan quality; treat them as optional notation, not a status mechanism. Prefer `## Phase N: <name>` headers + prose for structure.

### `checklists/` — procedures

Authored **operational reference**: reusable checklists, conventions, and reference standards worth consulting more than once. (Format ranges from a command sequence + checklist to a multi-page standard. For external/imported reference material, use `charts/` instead.)

Naming: `<topic>.md` (no date prefix — checklists are stable resources). A multi-file topic uses a bundle: `checklists/<topic>/` (see [Bundles](#bundles-multi-file-topics)).

Examples: `verify.md` (test before commit), `re-fixture.md` (regenerate test fixtures), `release.md`.

Promotion rule: a process becomes a checklist the **second** time you run it. First time = ad-hoc. Second time = pattern.

**Frontmatter required**: `when_to_read` (one-line trigger) + `applies_to` (short keyword tags) + `last_updated` (YYYY-MM-DD). Optional: `skip_when` (one-line "when NOT to read this" — negative routing to cut "maybe relevant" token waste). Same pattern as skill SKILL.md metadata — lets AI grep for relevance + judge staleness without loading the body. A checklist with no `when_to_read` is invisible to skill routing. See [templates.md#checklist](templates.md#checklist).

### `incident-reports/` — lessons learned

Mistakes worth not repeating. Format strictly enforces useful root cause (template in [templates.md](templates.md#incident-report)).

Naming: `<topic>.md` (no date prefix — incident reports are reference, not log)

Recurrence rule: same incident happens again → **append `## [Case N]`** to existing file. Do not create a new file. Repeated recurrence (≥3 times or single severe case) → promote one-liner to your project agent rules.

**Frontmatter required**: `when_to_read` (one-line trigger) + `applies_to` (short keyword tags) + `last_updated` (bump on Case append or status flip). Lets AI grep for relevance + judge staleness instead of reading every incident report at session start (token waste). An incident report with no `when_to_read` is invisible to skill routing.

### `charts/` — external material

External docs, competitor source code, RFCs, blog posts, etc. — kept here so the team has a single place for "where do I find that thing".

Naming: `<source>-<topic>.md` (e.g. `boltframe-shape-layer.md`, `rfc-6749.md`)

Lifecycle: prune when the external source becomes irrelevant. Do not delete reflexively — keep if you still might consult it.

### `sketches/` — long-term ideas

Unstarted ideas. Either grow into a spec (move to `specs/`) or sit. No status tracking.

Naming: `<topic>.md` (no date prefix — ideas are timeless until acted on)

Distinguish from `kneeboard/`: sketches are **unstarted**. `kneeboard/` is **in-progress and abandoned at session end**.

### `safety-reviews/` — external review feedback

Raw feedback from reviewers (other AIs, colleagues) + your **disposition** (adopt / reject / defer).

Naming: `YYYY-MM-DD-<spec-or-topic>-<reviewer>.md`

Disposition rule: no safety-review can exist in `safety-reviews/` without a disposition section. If disposition is incomplete, add a hanging task to `cockpit.md` ("finish disposition of `<file>`") and do not close the session.

### `kneeboard/` — session scratch

Short-lived scratch files: copy-pasted error output you'll refer to in 5 minutes, draft text you're shaping. Lives **one session**.

Naming: free-form. Date prefix optional.

**Exit cleanup rule**: at every landing, any `kneeboard/` file older than one session must be either classified into another folder or deleted. **No kneeboard files survive overnight without an explicit decision.**

This is the most-violated rule. Default to deletion. The cost of deleting a useful note is far smaller than the cost of `kneeboard/` slowly turning into a junk drawer.

**Enforcement (v0.6+)**: kneeboard files require a `last_touched: YYYY-MM-DD` frontmatter field, and stale entries trigger a landing-blocking hard gate. See [templates.md#kneeboard](templates.md#kneeboard) for the full rules.

### `landed/` — archive umbrella

Top-level archive for completed work. Replaces the old `*/finish/` pattern (where each active folder shadowed its own archive). `landed/` is a clean umbrella with typed subdirectories:

- `landed/flight-plans/` — plans archived after full execution.
- `landed/specs/` — specs archived after the design ships.

Archived files under `landed/` lose to current state in [authority order](SKILL.md#authority-order-when-sources-disagree).

### `INDEX.md` — quick lookup

A scannable index of the flightdeck, especially of `incident-reports/` and `checklists/` (resource directories). One line per file with a hook.

Maintenance: AI maintains automated sections marked with `<!-- AUTO-START -->` ... `<!-- AUTO-END -->`. Outside the markers is hand-curated. See [templates.md](templates.md#indexmd).

## Future expansion slots (DO NOT CREATE PREEMPTIVELY)

These are placeholder concepts. Create them only when the project's actual usage demands them:

- `decisions/` — Architecture Decision Records (ADRs). Useful when a project has ≥ 3 cross-spec decisions worth tracing back. Until then, decisions live in spec / cockpit / commit messages.
- `experiments/` — long-running data probes worth referencing across sessions (e.g., "the byte-level study of how AE rejects this header"). Until then, throwaway probes live in `tmp/` at the project root.

If you find yourself wanting one of these, note the need in `cockpit.md` and discuss before creating.

## Bundles (multi-file topics)

When one topic needs several files (a multi-chapter reference, a large checklist), make a **bundle**: a subfolder containing a `README.md` router plus detail files.

```
checklists/release-process/
├── README.md        # bundle contract + router (carries frontmatter)
├── 01-prepare.md    # leaf
├── 02-build.md      # leaf
└── 03-publish.md    # leaf
```

- **The README is the bundle contract**, not just an entry. Frontmatter:
  ```yaml
  ---
  bundle: true
  when_to_read: <one-line trigger>
  applies_to: [<short tag>, ...]
  reading_order: [01-prepare.md, 02-build.md, 03-publish.md]
  last_updated: YYYY-MM-DD
  # optional: skip_when, scope, non_goals
  ---
  ```
  The body should state **purpose / scope / non-goals / reading order**, or the bundle decays into "a folder with many `.md` files".
- **`reading_order` is the routing edge to the leaves**: it is what makes leaves reachable from the README (and therefore from the entry graph). A leaf present in the directory but absent from `reading_order` is an orphan — walkaround flags it.
- **Leaf** = any file in the bundle other than the README. Rules:
  - Leaves MUST NOT carry routing frontmatter (`when_to_read` / `applies_to`) — otherwise a recursive grep matches them directly and breaks the single-entry guarantee. The README is the only routing surface.
  - Leaves **inherit** the README's routing semantics unless explicitly overridden.
  - Freshness lives on the bundle: the README's `last_updated` is authoritative. Leaf `last_updated` is optional (per-leaf timestamps invite metadata decay).
  - Leaf-to-leaf links use relative paths (the dangling-reference audit catches breakage on rename/move).
- **No nested bundles**: one routing boundary per bundle.

Discovery still uses recursive frontmatter grep; the bundle contract leaves room for a future explicit routing index without locking today's convention out.

### Structured data (optional)

A bundle may hold `.csv` / `.json` for bulk lookup data — but ONLY alongside a `README.md` router that says "query on demand, do not read in full". Structured files must never be the primary entry (that would hurt AI readability + git reviewability).

## Naming convention table

| Folder | Filename pattern | Reason |
| --- | --- | --- |
| `specs/` | `YYYY-MM-DD-<feature>-design.md` | Date helps order by recency; designs are time-bound |
| `flight-plans/` | `YYYY-MM-DD-<feature>-plan.md` | Same as specs |
| `checklists/` | `<topic>.md` | Stable resource — date noise hurts |
| `incident-reports/` | `<topic>.md` | Stable resource — date noise hurts |
| `sketches/` | `<topic>.md` | Ideas are timeless |
| `safety-reviews/` | `YYYY-MM-DD-<spec>-<reviewer>.md` | Date + reviewer identify uniqueness |
| `charts/` | `<source>-<topic>.md` | External source is the key identifier |
| `kneeboard/` | free-form | Short-lived, naming overhead unjustified |

## Anti-patterns

| Anti-pattern | Why it fails | Fix |
| --- | --- | --- |
| Empty subdirs created to "establish the convention" | Pressure to fill → low-signal writes | Minimal setup; add folders on demand |
| `sketches/` used as `kneeboard/` | Half-finished work never gets pruned | `kneeboard/` is one session; `sketches/` is unstarted |
| Same fact duplicated across folders | Drift → trust collapses | One authoritative source; others link |
| Incident report files named `2026-05-23-bug.md` | Date noise; impossible to find by topic | Use `<topic>.md` |
| `tmp/` placed inside `flightdeck/` | Junk gets committed | `tmp/` lives at project root, gitignored |
