# Test plan (moved)

Moved to [`flightdeck/landed/specs/2026-05-23-v1.0-release-gate.md`](flightdeck/landed/specs/2026-05-23-v1.0-release-gate.md) on 2026-05-25 — the flightdeck project now dogfoods its own `flightdeck/`.

This stub remains so links from `README.md` / `README.zh.md` / `CHANGELOG.md` still resolve. Edit the spec, not this file.

## 1.2 test points

- **status frontmatter**: every artifact carries a valid `status`; `walkaround` flags a missing one (CRITICAL) and an out-of-range value (WARNING).
- **INDEX consistency**: each folder's `INDEX.md` (plus root `flightdeck/INDEX.md`) matches the files on disk; `landing` regenerates only the changed folders' INDEX.
- **Migration**: a synthetic 1.1.x layout (`manifest.md` / `logbook.md` / `kneeboard/` / `flight-plans/` / `incident-reports/` / `safety-reviews/`) triggers the 1.1.x→1.2 detection in `preflight`/`workflow`; an idempotent re-run skips already-migrated artifacts.
- **walkaround**: the 10 audits surface illegal status, INDEX↔folder drift, missing `superseded_by`, orphan plan (INFO), and legacy 1.x paths.
- **rules.md**: `git: false` makes `landing` skip the commit step and append `landed/HISTORY.md`; `disabled_folders` suppresses orphan flags; `disabled_gates` skips the `debrief-disposition` gate.
- **emit-agents-md**: renders Current focus / Next session / Hanging tasks from `cockpit.md`.
