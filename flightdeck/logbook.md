# Logbook — flightdeck history

> Rarely read. Open when writing release notes, doing retrospectives, or auditing what was done across past sessions.

## Recently finished (cap 5, FIFO)

- 2026-05-28: v1.0.0 — flightdeck rebrand + board.md decomposition (cockpit/manifest/logbook), `*/finish/` → `landed/` umbrella, skill modules renamed, governing principle "semantic clarity outranks thematic consistency" surfaced into SKILL.md.
- 2026-05-26: v0.8.0 — lifecycle deepening: `/workshop:doctor` audit skill, multi-criterion scar→playbook promotion gate, wip Pre-write checklist, OpenSpec delta markers, Recently finished auto-trim enforcement (commit `325dcf1`).
- 2026-05-26: v0.7.0 — cross-tool reach: `/workshop:emit-agents-md` slash skill + AGENTS.md dogfood, Cursor MDC fields, manifest schema verification, capability × tool matrix, README "Why not AGENTS.md?" section (commit `23685f7`).
- 2026-05-25: v0.6.0 — cleanup cut: SKILL.md de-dup, frontmatter hard-fail, Mermaid lifecycle diagram, wip TTL gate, plan-checkbox doc clarification (commit `e9e0c85`).
- 2026-05-25: v0.5.0 — explicit `/workshop:session-enter` and `/workshop:session-exit` slash skills (commit `e926dec`)

## Deferred

- v1.x: MCP server exposing `flightdeck/`
- v1.x: Boomerang-style subagent template under `flight-plans/`
- v1.x: Continuance benchmark
- v1.x: Spec compression / retrospective synthesis
- v1.1+: optional folders (`briefing/`, `blackbox/`, `crew-handover/`, `experiments/`) — deferred from v1.0 rebrand, revisit when usage demands

---

**Logbook hygiene**:
- `Recently finished` cap 5 entries FIFO. Per-entry summary ≤ 3 lines. Longer content → link to commit / archived plan.
- `Deferred` items live here as long as they remain deferred. Promote out when actively worked on.
- No date-bumping rule — logbook is append-mostly history.
