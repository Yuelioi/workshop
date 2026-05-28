# Cockpit — flightdeck (the flightdeck project itself)

**Last updated**: 2026-05-28 by 月离 (shipped v1.0 — flightdeck rebrand + board decomposition)
**Active focus**: Post-v1.0 dogfood under new names — watch for manifest.md naming friction, cockpit.md ceiling pressure, metaphor lock-in temptations on any new concept added.

## Next session

1. Dogfood flightdeck on real projects. Log friction in `kneeboard/` then classify at landing per the existing discipline gates.
2. Re-emit AGENTS.md across active projects using `/flightdeck:emit-agents-md`.
3. Reassess [v1.1+ deferred folders](landed/specs/2026-05-28-flightdeck-rebrand-design.md#out-of-scope-deferred-to-v11) (briefing/, blackbox/, crew-handover/, experiments/) after 1-2 weeks of real use.

## Hanging tasks

- (none)

## Note on dogfooding

This `flightdeck/` is the flightdeck project's own workbench. Incident-reports / checklists here document **maintaining the flightdeck tool**, not using it on other projects. All `applies_to:` frontmatter must target flightdeck-project paths (`skills/`, `hooks/`, `scaffolds/`, etc.) — never `applies_to: general`.

Users running flightdeck in their own projects see their own `flightdeck/`, not this one.

---

**Cockpit hygiene** (skill: flightdeck-workflow):
- **80 lines hard ceiling.** Cockpit is intentionally ephemeral and operational. Historical / archival content does not live here — it goes to logbook.md.
- `Last updated` bumps ONLY when: Next session changes / Active focus shifts / a major task completes. Not on typo fixes, grep, or routine commits.
- If something feels permanent or referential, it does not belong in cockpit — move to logbook.md or out entirely.
