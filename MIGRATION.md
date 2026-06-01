# Migration

This document records breaking migrations for the maintainer's reference.

## 1.1.x → 1.2

1.2 keeps the 1.x worldview (sketch / spec / plan / incident / checklist / chart / debrief) and adds two things: **explicit `status` metadata** on every artifact, and a **per-folder + root `INDEX.md`** derived index. `preflight`/`workflow` detect the old layout (any of `manifest.md`, `logbook.md`, `kneeboard/`, `flight-plans/`, `incident-reports/`, `safety-reviews/`) and offer this **interactive, non-silent, idempotent** migration (each step skips if already done; unknown fields preserved):

1. **`manifest.md`** → fold non-trivial rows into `cockpit.md` (Active focus / Next session / Hanging tasks) or drop; delete `manifest.md`.
2. **`logbook.md`** → import `Recently finished` into `landed/HISTORY.md` (newest first); move `Deferred` to `sketches/` or `cockpit.Next session`; delete `logbook.md`.
3. **`kneeboard/`** → classify each file into a folder or delete; remove the empty dir. Session scratch now lives in project-root `tmp/` (gitignored).
4. **`flight-plans/` → `plans/`**. Each file: add `status:`; optionally add `implements: specs/<x>.md`.
5. **`incident-reports/` → `incidents/`**. Add `status:` (keep `when_to_read`/`applies_to`/`last_updated`).
6. **`safety-reviews/` → `debriefs/`**. Add `status:` + `reviewed: specs/<x>.md` + `last_updated` (no `when_to_read`/`applies_to`).
7. **`specs/` stays.** Add `status:` to each file.
8. **`checklists/` / `charts/` stay.** Add `status:` (knowledge folders keep their routing fields).
9. **`sketches/` stays.** Add `status: active` (or `scrapped`).
10. **Build `INDEX.md`** for every artifact folder + a root `flightdeck/INDEX.md` (the `<!-- AUTO -->` region derived from each file's frontmatter).
11. **cockpit**: drop any `## In flight` / `## Blockers`; pure focus (Active focus / Next session / Hanging tasks).
12. **Optional:** create `rules.md` for toggles.

### Old → new mapping

| 1.1.x | 1.2 |
|---|---|
| `manifest.md` | folded into `cockpit.md` (or dropped) |
| `logbook.md` | `landed/HISTORY.md` (+ `sketches/` for Deferred) |
| `kneeboard/` | removed; scratch → project-root `tmp/` |
| `flight-plans/` | `plans/` (+ `status`, + optional `implements`) |
| `incident-reports/` | `incidents/` (+ `status`) |
| `safety-reviews/` | `debriefs/` (+ `status`, + `reviewed`) |
| `specs/` | `specs/` (+ `status`) |
| `checklists/` / `charts/` / `sketches/` | unchanged paths (+ `status`) |
| location-implicit state | explicit `status:` field |
| (no index) | per-folder `INDEX.md` + root `flightdeck/INDEX.md` |
