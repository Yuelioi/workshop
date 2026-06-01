# Cockpit — flightdeck (the flightdeck project itself)

**Last updated**: 2026-06-01 by 月离 (reverted own workbench from abandoned 2.0 to shipped 1.2; finalizing release)
**Active focus**: flightdeck 1.2 — explicit `status` frontmatter + per-folder/root `INDEX.md` (refinement of 1.1.x). Shipped on this branch; finalizing release + dogfooding.

## Next session

1. Finalize the 1.2 release per [checklists/version-bump.md](checklists/version-bump.md).
2. Dogfood 1.2 on real projects — log friction, classify at landing.
3. Behaviorally verify Codex / Cursor / Gemini manifests against the archived release-gate scenarios (see [checklists/version-bump.md](checklists/version-bump.md) verification section for smoke-check reference).
4. Reassess deferred folders — see [sketches/v1x-deferred-ideas.md](sketches/v1x-deferred-ideas.md).

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
