# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.0.0] — 2026-05-28

**Project renamed from `workshop` to `flightdeck`.** Aviation framing for operational discipline, continuity, and reliability. Single breaking-change window — post-v1.0 is additive-only.

See [MIGRATION.md](MIGRATION.md) for upgrade steps. Design rationale: [flightdeck/landed/specs/2026-05-28-flightdeck-rebrand-design.md](flightdeck/landed/specs/2026-05-28-flightdeck-rebrand-design.md).

### Renamed

- **Project**: workshop → flightdeck (plugin name, marketplace identifier, repo URL, install commands)
- **Folders**: `plans/` → `flight-plans/`, `playbooks/` → `checklists/`, `scars/` → `incident-reports/`, `reference/` → `charts/`, `critiques/` → `safety-reviews/`, `wip/` → `kneeboard/`. `specs/` and `sketches/` retained (no aviation equivalent improves them).
- **Skill modules**: `workshop-workflow/` → `flightdeck-workflow/`, `session-enter/` → `preflight/`, `session-exit/` → `landing/`, `doctor/` → `walkaround/`. `emit-agents-md/` unchanged.
- **Slash commands**: `/workshop:session-enter` → `/flightdeck:preflight`, `/workshop:session-exit` → `/flightdeck:landing`, `/workshop:doctor` → `/flightdeck:walkaround`, `/workshop:emit-agents-md` → `/flightdeck:emit-agents-md`.

### Decomposed

- **`board.md` split into three files** separated by read-time:
  - `cockpit.md` — must-read every session entry (Active focus, Next session, Hanging tasks). **80-line hard ceiling.**
  - `manifest.md` — on-demand (In flight artifacts, Blockers). No ceiling.
  - `logbook.md` — rarely read (Recently finished FIFO 5, Deferred). Append-mostly history.

### Restructured

- **`*/finish/` archive subdirs promoted to top-level `landed/` umbrella**. Now `landed/flight-plans/` and `landed/specs/`. Eliminates the "active folder shadowing its own archive" inelegance.

### Surfaced

- **Governing principle** lifted from rebrand spec into `flightdeck-workflow/SKILL.md` and `AGENTS.md`: **"Semantic clarity outranks thematic consistency."** The metaphor is a tool, not a theme. Resist metaphor lock-in on future concepts.

### Repository

- GitHub repo renamed `Yuelioi/workshop` → `Yuelioi/flightdeck`. Auto-redirect in place. Plugin marketplace identifier updated across all 4 platforms.
- VERSION: 0.8.1 → 1.0.0.

### Deferred to v1.1+

These were considered for v1.0 but punted to keep scope contained. Revisit when real usage demands each:

- `briefing/` (domain context / glossary)
- `blackbox/` (raw session-log persistence)
- `crew-handover/` (human ↔ human / cross-AI handoff)
- `experiments/` (long-running probes — already a `future expansion slot`)
- Automated migration skill (manual `MIGRATION.md` was sufficient for user base = 1)

## [0.8.1] — 2026-05-26

Patch release: README clarity + bootstrap UX + content cleanup. No protocol changes.

### Added

- **Bootstrap behavior in `workshop-workflow` skill**: when invoked in a project without `workshop/`, the skill now asks to create one, runs a short Active focus / Next session interview, and writes `workshop/board.md` directly. No more `install.sh --scaffold=minimal` round-trip required for Claude Code users.
- **Slash commands table** in README (EN + ZH) — all 5 commands listed with auto-load behavior and one-line purpose. Replaces the scattered "Force-invoke" + "Explicit slash commands (v0.5.0+)" sections.

### Changed

- **README "Day 1" section** simplified to `/workshop:workshop-workflow` flow. The clone + `install.sh` path remains documented as the fallback for non-Claude tools.
- **`workshop-workflow` skill** decoupled from `superpowers` plugin. Cross-references reframed as "Optional companions" — workshop is self-contained and accepts content from any source. Scenario triggers, exit-ritual heuristics, and folder-semantics no longer prescribe `superpowers:*` skills.
- **Trims**: SKILL.md dropped redundant prose Transitions table (Mermaid covers it), "Why semi-implicit" paragraph, "Backlog specs" edge case, "Proactive scar resurfacing" explanation. Common mistakes merged with Red flags. Net auto-load token cost down ~20%.
- **CHANGELOG** compressed: v0.3–v0.5 entries reduced to 1–2 lines each; "Known limitations" sections folded; per-commit citations dropped.
- **Archived plan** `workshop/plans/finish/2026-05-25-v0.6-cleanup.md` compressed from 789 → 39 lines (step-by-step Edit prescriptions removed; outcome summary retained).

### Fixed

- **README.md** had a duplicate `🇨🇳 中文用户` Chinese-link line inside the `## Install` section (original top-of-file placement + middle convenience link both rendered). Removed the duplicate.
- **`exit-ritual.md`** "Red flags" and "Common rationalizations" tables had significant overlap with `SKILL.md` Common mistakes; the two exit-ritual tables removed (one cross-link added instead).

## [0.8.0] — 2026-05-26

Lifecycle deepening. State-machine depth and protocol-drift surfacing — QoL polish landed before the v1.0 format freeze.

### Added

- **`/workshop:doctor` slash skill** (`skills/doctor/SKILL.md`) — audits a workshop for protocol drift across 8 categories: scars/playbooks frontmatter, stale wip, dangling internal references, orphan scars, board ↔ folder lifecycle mismatch, stale Blockers entries, Recently finished length, AGENTS.md regeneration drift. Severity: CRITICAL / WARNING / INFO. Never auto-fixes — surfaces drift; author decides.
- **OpenSpec-style spec evolution markers** (`ADDED:` / `MODIFIED:` / `REMOVED:`) — optional convention documented in `skills/workshop-workflow/templates.md` for long-lived backlog specs.
- **wip Pre-write checklist** in `skills/workshop-workflow/templates.md` — two-question hard gate before creating any new `wip/` file. Prose discipline, not programmatic enforcement.

### Changed

- **Scar promotion gate** is now multi-criterion: `[Case N] count ≥ 3` AND `≥ 2 distinct sessions` AND remediation pattern stable across cases. Two-stage path: first to `playbooks/`, then to project rules only if the playbook keeps getting ignored.
- **`Recently finished` auto-trim** is enforced, not advisory. `exit-ritual.md` wording tightened to "MUST enforce" / "not author-discretion".
- **session-exit Step 5** ("Check scar→playbook promotion gate (wrap-up)") added after the commit decision.

## [0.7.0] — 2026-05-26

Cross-tool reach. Workshop is portable beyond Claude Code — AGENTS.md emitter bridges to Codex CLI / Copilot / Cursor / Windsurf / Continue / Cody.

### Added

- **`/workshop:emit-agents-md` slash skill** (`skills/emit-agents-md/SKILL.md`) — regenerates `AGENTS.md` at repo root from `workshop/board.md` between fenced markers (`<!-- BEGIN: workshop -->` / `<!-- END: workshop -->`). Hand-authored content outside markers is preserved. Relative links from `board.md` get prefixed with `workshop/`.
- **`AGENTS.md` at repo root** — dogfood output of the emitter. Bridges to the cross-tool standard.
- **Optional Cursor MDC frontmatter fields** (`globs:` + `alwaysApply:`) on scar/playbook templates.
- **Capability × tool compatibility matrix** in `workshop/specs/2026-05-23-v1.0-release-gate.md`.
- **Community PR template** at `.github/PULL_REQUEST_TEMPLATE/manifest-verification.md`.
- **README "Why not just AGENTS.md?" section** (EN + ZH).

### Changed

- **`session-exit/SKILL.md`** gains step 5 (regenerate `AGENTS.md` if `board.md` changed); old Commit step becomes step 6.

### Known limitations

- Behavioral verification on Codex CLI / Cursor / Gemini CLI is out of scope; community PRs invited.
- `.codex-plugin/plugin.json` is functionally inert (Codex CLI has no plugin manifest format); workshop reaches Codex via emitted `AGENTS.md`.

## [0.6.0] — 2026-05-25

Internal consistency cut. No new user-facing features; deduplicates doctrine, hardens enforcement.

### Changed

- **De-duplicated `skills/session-exit/SKILL.md` against `exit-ritual.md`** — lifecycle table + (a)–(h) classification + `Last updated` triggers now live in one place; `session-exit/SKILL.md` is a thin entry-point.
- **Hard-fail on missing scars/playbooks frontmatter.** `when_to_read` + `applies_to` + `last_updated` are REQUIRED; workshop STOPs and reports missing fields instead of silent-skipping.
- **`wip/` TTL hard gate.** Wip files require `last_touched:` frontmatter; `session-enter` surfaces stale wip; `session-exit` BLOCKS until classified, deleted, or explicitly deferred with `defer_reason:`.

### Added

- **Mermaid lifecycle state diagram** in `skills/workshop-workflow/SKILL.md`.

### Migration from v0.5.x

- Add `when_to_read` + `applies_to` + `last_updated` to existing scars / playbooks (or delete the file). Otherwise v0.6+ STOPs and reports them.
- Add `last_touched:` to existing `wip/` files. Otherwise they count as stale and block session-exit.

## [0.5.0] — 2026-05-25

Two explicit slash-command skills exposing the workshop entry / exit rituals as one-command triggers: `/workshop:session-enter` (re-anchor mid-session) and `/workshop:session-exit` (clean wraps). Both `disable-model-invocation: true` — fire only on explicit slash.

## [0.4.0] — 2026-05-25

Dogfood refinements after 1 week of real use: `last_updated` frontmatter on scars/playbooks; proactive scar resurfacing when task overlaps with scar `applies_to` tags; per-review-point critique disposition tags; semi-implicit lifecycle (location = source of truth, frontmatter for divergent states); `Last updated` bump triggers pinned to 4 events.

## [0.3.0] — 2026-05-25

SessionStart hook auto-injects `workshop-workflow` when project has `workshop/`. Six-state lifecycle machine added. Board hygiene: 300-line hard ceiling, `Recently finished` capped at 5 (FIFO), per-entry summary ≤ 3 lines. Scars / playbooks require `when_to_read` + `applies_to` frontmatter.

## [0.2.0] — 2026-05-23

Multi-AI manifests: Claude Code plugin + self-hosted marketplace (tested); Codex CLI / Cursor / Gemini CLI manifests (untested). Chinese README mirror. Skill content made fully tool-neutral.

## [0.1.0] — 2026-05-23

Initial scaffold: `workshop-workflow` skill, adapters, scaffold templates, installers, RED-phase test plan.
