# Board — workshop (the workshop project itself)

**Last updated**: 2026-05-26 by 月离 (entered dogfood mode — push v0.6/v0.7/v0.8 tags; gather operational signals before v1.0 freeze decision)
**Active focus**: Dogfood v0.6–v0.8 (≥ 1 week) — cross-session / cross-tool / deliberate violations to surface friction; targets: is board.md the single recovery entry? does wip TTL compress entropy? do scar→playbook gates false-positive?

## Next session

1. Day-to-day dogfood: use workshop normally on real projects. Log friction in `wip/` then classify at session-exit per the v0.6+v0.8 discipline gates.
2. End-of-dogfood-week trigger (~2026-06-02): aggregate signals — read `scars/` and any `/workshop:doctor` reports captured. Note false-positive scar→playbook promotion prompts, stale-wip slip-throughs, AGENTS.md drift.
3. v1.0 entry decision: enter as-roadmapped, scope-adjust per dogfood findings, or defer further. Re-read v1.0 section of [specs/2026-05-25-v0.6-to-v1.0-roadmap.md](specs/2026-05-25-v0.6-to-v1.0-roadmap.md) with the week's signals in hand.

## In flight (only artifacts whose state diverges from folder location)

<!-- All current specs sit at implicit ⚪ pending. No divergent states. -->

| Artifact | State | Owner / Reason | Refs |
| --- | --- | --- | --- |
| _none_ | | | |

**Status legend**: 🔵 awaiting review · 🔴 blocked · 🗑️ scrapped
(⚪ pending / 🟡 in progress / ✅ done are implicit from location.)

## Blockers

- (none) — v0.7 deferred Codex / Cursor / Gemini behavioral verification to community PRs via [.github/PULL_REQUEST_TEMPLATE/manifest-verification.md](../.github/PULL_REQUEST_TEMPLATE/manifest-verification.md). Not blocking v0.8.

## Deferred

- v1.x: MCP server exposing `workshop/`
- v1.x: Boomerang-style subagent template under `plans/`
- v1.x: Continuance benchmark
- v1.x: Spec compression / retrospective synthesis

## Recently finished (cap 5, FIFO)

- 2026-05-26: v0.8.0 — lifecycle deepening: `/workshop:doctor` audit skill, multi-criterion scar→playbook promotion gate, wip Pre-write checklist, OpenSpec delta markers, Recently finished auto-trim enforcement (commit `325dcf1`).
- 2026-05-26: v0.7.0 — cross-tool reach: `/workshop:emit-agents-md` slash skill + AGENTS.md dogfood, Cursor MDC fields, manifest schema verification, capability × tool matrix, README "Why not AGENTS.md?" section (commit `23685f7`).
- 2026-05-25: v0.6.0 — cleanup cut: SKILL.md de-dup, frontmatter hard-fail, Mermaid lifecycle diagram, wip TTL gate, plan-checkbox doc clarification (commit `e9e0c85`).
- 2026-05-25: v0.5.0 — explicit `/workshop:session-enter` and `/workshop:session-exit` slash skills (commit `e926dec`)
- 2026-05-25: v0.4.0 — dogfood refinements: proactive scar resurfacing, semi-implicit lifecycle, `Last updated` trigger pinning (commit `ae876ee`)

## Hanging tasks

- (none)

---

**Board hygiene** (skill: workshop-workflow):
- 300 lines hard ceiling. Aim for < 200.
- `In flight` only lists artifacts with explicit `state:` frontmatter (the divergent ones). Implicit ⚪/🟡/✅ state is inferred from folder location.
- `Recently finished` cap 5 entries FIFO. Per-entry summary ≤ 3 lines. Longer content → link to commit / archived plan.
- `Last updated` bumps ONLY when: Next session changes / Active focus shifts / Recently finished gains an entry / major task completes. Not on typo fixes, grep, or routine commits.

## Note on dogfooding

This `workshop/` is the workshop project's own workbench. Scars / playbooks here document **maintaining the workshop tool**, not using it on other projects. All `applies_to:` frontmatter must target workshop-project paths (`skills/`, `hooks/`, `scaffolds/`, etc.) — never `applies_to: general`.

Users running workshop in their own projects see their own `workshop/`, not this one.
