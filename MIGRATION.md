# Migration: workshop → flightdeck (v0.8.1 → v1.0.0)

This document records the v1.0 rebrand for the maintainer's reference. Read this if you cloned a pre-v1.0 workshop directory and need to update it manually, or if you're trying to reconcile old commits / docs against new paths.

## What changed

### Top-level

- Project name: **workshop** → **flightdeck**
- GitHub repo: `Yuelioi/workshop` → `Yuelioi/flightdeck` (auto-redirect in place)
- Plugin name and marketplace identifier: workshop → flightdeck
- VERSION: 0.8.1 → 1.0.0

### Directory rename

- `workshop/` → `flightdeck/`

### Folders inside the working dir

| Old | New |
| --- | --- |
| `plans/` | `flight-plans/` |
| `playbooks/` | `checklists/` |
| `scars/` | `incident-reports/` |
| `reference/` | `charts/` |
| `critiques/` | `safety-reviews/` |
| `wip/` | `kneeboard/` |
| `*/finish/` | `landed/*/` (promoted from nested archive to umbrella) |
| `specs/`, `sketches/` | unchanged |

### Entry-point file decomposition

- `board.md` split into three files (separated by read-time):
  - `cockpit.md` — Active focus + Next session + Hanging tasks. **80-line ceiling.**
  - `manifest.md` — In flight + Blockers.
  - `logbook.md` — Recently finished + Deferred.

### Slash skills

- `/workshop:session-enter` → `/flightdeck:preflight`
- `/workshop:session-exit` → `/flightdeck:landing`
- `/workshop:doctor` → `/flightdeck:walkaround`
- `/workshop:emit-agents-md` → `/flightdeck:emit-agents-md` (unchanged function, namespace updated)

## Migrating a pre-v1.0 workshop directory

For each project that has a `workshop/` directory you want to update:

1. **Rename top-level**: `git mv workshop flightdeck`
2. **Rename inner folders** per the table above:
   ```bash
   cd flightdeck
   git mv plans flight-plans
   git mv playbooks checklists      # if it exists
   git mv scars incident-reports    # if it exists
   git mv reference charts          # if it exists
   git mv critiques safety-reviews  # if it exists
   git mv wip kneeboard             # if it exists
   ```
3. **Promote archives to landed/ umbrella**:
   ```bash
   mkdir -p landed/flight-plans landed/specs
   # If you had plans/finish/* :
   git mv flight-plans/finish/* landed/flight-plans/
   rmdir flight-plans/finish
   # If you had specs/finish/* :
   git mv specs/finish/* landed/specs/
   rmdir specs/finish
   ```
4. **Split `board.md`** into three files. Read your current `flightdeck/board.md` and partition its sections by the read-time mapping:
   - **cockpit.md**: `Last updated` + `Active focus` + `## Next session` + `## Hanging tasks` (≤ 80 lines)
   - **manifest.md**: `## In flight` (with status legend) + `## Blockers`
   - **logbook.md**: `## Recently finished` + `## Deferred`
   - Delete `board.md` after.
   - See `flightdeck/landed/specs/2026-05-28-flightdeck-rebrand-design.md § board.md decomposition` for templates.
5. **Update AGENTS.md / GEMINI.md / CLAUDE.md** at project root: replace `workshop/` paths with `flightdeck/`, replace `board.md` with `cockpit.md`/`manifest.md` per context.
6. **Re-emit AGENTS.md** with `/flightdeck:emit-agents-md` if you use the auto-generated form. The fenced block markers change from `<!-- BEGIN: workshop -->` to `<!-- BEGIN: flightdeck -->`.
7. **Reinstall the plugin**: uninstall the workshop plugin, install flightdeck:
   ```text
   /plugin uninstall workshop
   /plugin marketplace add Yuelioi/flightdeck
   /plugin install flightdeck@flightdeck-marketplace
   ```

## Why the rename

See [flightdeck/landed/specs/2026-05-28-flightdeck-rebrand-design.md](flightdeck/landed/specs/2026-05-28-flightdeck-rebrand-design.md) for the design rationale.

Short version: `workshop` framed the project as a maker space / sandbox. `flightdeck` frames it as an operational protocol — closer to aviation operations than to a workshop, which is what the project actually does: session lifecycle, checklists, incident tracking, handoffs, controlled autonomy.

The decomposition of `board.md` into cockpit / manifest / logbook is the only conceptual change. Everything else is renaming.
