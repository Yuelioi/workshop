# Cockpit — flightdeck (the flightdeck project itself)

**Last updated**: 2026-06-02 by 月离 (v2.0.0 shipped — single explicit /preflight entry; workflow skill + hook removed)
**Active focus**: flightdeck 2.0 shipped — single explicit `/flightdeck:preflight` entry (init-or-read); `workflow` skill + SessionStart hook deleted; protocol folded into `preflight/`. Layout still 1.2.
**Layout**: 1.2

## Next session

1. Dogfood 2.0 single-entry on real projects — confirm `/preflight` init-or-read and no auto-load surprises; classify friction at landing.
2. Behaviorally verify Codex / Cursor / Gemini manifests against the archived release-gate scenarios (see [checklists/version-bump.md](checklists/version-bump.md) verification section for smoke-check reference).
3. Reassess deferred folders — see [sketches/v1x-deferred-ideas.md](sketches/v1x-deferred-ideas.md).

## Hanging tasks

- (none)

## Note on dogfooding

This `flightdeck/` is the flightdeck project's own workbench. `incidents/` / `checklists/` here document **maintaining the flightdeck tool**, not using it on other projects. All `applies_to:` frontmatter must target flightdeck-project paths (`skills/`, `hooks/`, `scaffolds/`, etc.) — never `applies_to: general`.

Users running flightdeck in their own projects see their own `flightdeck/`, not this one.

---

**Cockpit hygiene** (skill: workflow):
- **80 lines hard ceiling.** Cockpit is operational, not archival. History lives in `git log` / `landed/HISTORY.md`.
- `Last updated` bumps ONLY when: Next session changes / Active focus shifts / a major task completes / an artifact lands.
- Finished items leave `Next session`; they are not logged in cockpit.
- Artifact state is tracked via `status:` frontmatter + the folder `INDEX.md` files, not here.
