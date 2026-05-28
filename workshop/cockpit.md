# Cockpit — flightdeck (the flightdeck project itself)

**Last updated**: 2026-05-26 by 月离 (entered dogfood mode — push v0.6/v0.7/v0.8 tags; gather operational signals before v1.0 freeze decision)
**Active focus**: Dogfood v0.6–v0.8 (≥ 1 week) — cross-session / cross-tool / deliberate violations to surface friction; targets: is board.md the single recovery entry? does wip TTL compress entropy? do scar→playbook gates false-positive?

## Next session

1. Day-to-day dogfood: use workshop normally on real projects. Log friction in `wip/` then classify at session-exit per the v0.6+v0.8 discipline gates.
2. End-of-dogfood-week trigger (~2026-06-02): aggregate signals — read `scars/` and any `/workshop:doctor` reports captured. Note false-positive scar→playbook promotion prompts, stale-wip slip-throughs, AGENTS.md drift.
3. v1.0 entry decision: enter as-roadmapped, scope-adjust per dogfood findings, or defer further. Re-read v1.0 section of [specs/2026-05-25-v0.6-to-v1.0-roadmap.md](specs/2026-05-25-v0.6-to-v1.0-roadmap.md) with the week's signals in hand.

## Hanging tasks

- (none)

## Note on dogfooding

This `workshop/` is the workshop project's own workbench. Scars / playbooks here document **maintaining the workshop tool**, not using it on other projects. All `applies_to:` frontmatter must target workshop-project paths (`skills/`, `hooks/`, `scaffolds/`, etc.) — never `applies_to: general`.

Users running workshop in their own projects see their own `workshop/`, not this one.

---

**Cockpit hygiene** (skill: flightdeck-workflow):
- **80 lines hard ceiling.** Cockpit is intentionally ephemeral and operational. Historical / archival content does not live here — it goes to logbook.md.
- `Last updated` bumps ONLY when: Next session changes / Active focus shifts / a major task completes. Not on typo fixes, grep, or routine commits.
- If something feels permanent or referential, it does not belong in cockpit — move to logbook.md or out entirely.
