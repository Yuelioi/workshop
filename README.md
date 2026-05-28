# flightdeck

🇨🇳 **中文用户**: see [README.zh.md](README.zh.md) for a Chinese version.

> An operational protocol for AI-assisted engineering sessions.

> **Renamed from `workshop` (≤ v0.8.1)** — the project's identity is now operational reliability, not "maker space". See [MIGRATION.md](MIGRATION.md) for the upgrade path, and the design rationale in [flightdeck/landed/specs/2026-05-28-flightdeck-rebrand-design.md](flightdeck/landed/specs/2026-05-28-flightdeck-rebrand-design.md).

Your AI assistant forgets everything between chats. `flightdeck` is a directory convention + skill that gives it operational discipline across sessions — so the next session knows what you were doing, why, and what to do next.

## What it is

A `flightdeck/` directory layout your AI reads and writes by convention:

```
flightdeck/
├── INDEX.md            # Quick lookup of subdir purposes + key files
├── cockpit.md          # Must-read every session entry (≤ 80 lines)
├── manifest.md         # On-demand: In flight + Blockers
├── logbook.md          # Rarely read: Recently finished + Deferred
│
├── specs/              # Design docs                    (when designing)
├── flight-plans/       # Implementation plans           (when breaking down work)
│
├── checklists/         # Repeatable procedures          (commands + checklists)
├── incident-reports/   # Lessons learned (no "forgot")  (recurring traps)
├── charts/             # External material              (RFCs, competitor code)
│
├── sketches/           # Long-term ideas                (unstarted)
├── safety-reviews/     # External review feedback       (raw + disposition)
├── kneeboard/          # Session scratch                (one session only)
│
└── landed/             # Archive umbrella
    ├── flight-plans/   # Shipped plans
    └── specs/          # Shipped designs
```

**Organized by when you read what** — not by topic. The folder / file name tells the AI when to consult it.

## Why it exists

Most "AI memory" systems fail by **saving everything**. The signal drowns. You get a junk drawer that even the AI gives up on reading.

`flightdeck` does the opposite:

- **Strict write gate**: only content that changes future behavior, influences decisions, or gets referenced repeatedly.
- **Lifecycle for every folder**: `kneeboard/` lives one session; incident reports upgrade to project rules on third recurrence; specs/flight-plans archive into `landed/` after ship.
- **Authority order**: when sources disagree, the protocol says who wins.
- **Landing ritual**: 90% of session-end classifications are obvious. Only true ambiguity triggers brainstorming.
- **Read-time decomposition**: cockpit / manifest / logbook separate what you read every session from what you open on demand from what you almost never re-read.

The full discipline lives in [`skills/flightdeck-workflow/SKILL.md`](skills/flightdeck-workflow/SKILL.md).

## Design philosophy

> **Semantic clarity outranks thematic consistency.**

The flightdeck aviation metaphor is used where it sharpens operational intent — *not* as a theme to apply uniformly. Two folders (`specs/`, `sketches/`) intentionally keep neutral names because no aviation equivalent improves them. New concepts face the same test: if a word fits the metaphor but reads confusingly, reject it.

## Install

### Claude Code ✅ tested

In any Claude Code session:

```text
/plugin marketplace add Yuelioi/flightdeck
/plugin install flightdeck@flightdeck-marketplace
```

To update: re-run `/plugin install`. To uninstall: `/plugin uninstall flightdeck`.

### Codex CLI ⚠️ manifest in place, behavior untested

```text
/plugins
```

Then search "flightdeck" → select → `Install Plugin`.

### Cursor ⚠️ manifest in place, behavior untested

In Cursor Agent chat:

```text
/add-plugin flightdeck
```

Or search "flightdeck" in the plugin marketplace.

### Gemini CLI ⚠️ manifest in place, behavior untested

```bash
gemini extensions install https://github.com/Yuelioi/flightdeck
```

Update later:

```bash
gemini extensions update flightdeck
```

### Alternative — direct skill install (Claude Code only, no marketplace)

For users who prefer a direct copy into `~/.claude/skills/`:

```powershell
# Windows
git clone https://github.com/Yuelioi/flightdeck.git
cd flightdeck
.\install.ps1
```

```bash
# macOS / Linux
git clone https://github.com/Yuelioi/flightdeck.git
cd flightdeck
./install.sh
```

### Scaffold a `flightdeck/` in your project

```powershell
.\install.ps1 -Scaffold minimal     # just cockpit.md
.\install.ps1 -Scaffold full        # all 11 subdirs + 3 entry files
```

```bash
./install.sh --scaffold=minimal
./install.sh --scaffold=full
```

## Usage

After install, the skill auto-loads whenever your project has a `flightdeck/` directory. You can also force-invoke it.

### Day 1 — bootstrap a new project

In Claude Code, `cd` into your project and type:

```text
/flightdeck:flightdeck-workflow
```

The skill detects the missing `flightdeck/` directory, asks you to confirm, then walks through a short interview (Active focus, first Next session item) and writes `flightdeck/cockpit.md`. From the next session onward, the SessionStart hook auto-loads the skill whenever `flightdeck/` is present — no slash needed.

**Other tools / scripted setup**: clone this repo and run `install.sh --scaffold=minimal`, or copy `scaffolds/minimal/flightdeck/` into your project manually.

### What the AI does on every session

```
1. Reads flightdeck/cockpit.md
2. Reconciles against `git status` (branch, uncommitted changes, stashes)
3. Executes the first "Next session" item, or asks if state is inconsistent
```

### Slash commands

| Command | Auto-loads? | What it does |
| --- | --- | --- |
| `/flightdeck:flightdeck-workflow` | Yes — auto-injected via SessionStart hook when `flightdeck/` exists in cwd | Force-load the main protocol. **Also bootstraps**: if no `flightdeck/` exists, asks to create one and walks you through `cockpit.md` setup. Then runs the entry checklist. |
| `/flightdeck:preflight` | No — explicit only | Re-anchor a drifted long session: re-read `cockpit.md`, reconcile with `git status` / branch / stash / commit timeline, surface stale `kneeboard/`. |
| `/flightdeck:landing` | No — explicit only | Clean session wrap: classify new knowledge via (a)–(h) heuristics, update cockpit, apply lifecycle transitions, prompt incident-report → checklist promotion when gate fires, optionally commit. |
| `/flightdeck:emit-agents-md` | No — explicit only | Regenerate `AGENTS.md` at repo root from `flightdeck/cockpit.md` (bridges to AGENTS.md-consuming tools: Codex CLI, Copilot, Cursor, Windsurf, Continue, Cody). Run after `cockpit.md` changes. |
| `/flightdeck:walkaround` | No — explicit only | Audit `flightdeck/` for protocol drift across 8 categories (missing frontmatter, stale kneeboard, dangling refs, manifest ↔ folder mismatch, stale Blockers, Recently finished length, AGENTS.md drift, orphan incident reports). Reports CRITICAL / WARNING / INFO — never auto-fixes. |

All commands except `flightdeck-workflow` carry `disable-model-invocation: true` — they fire only on explicit slash, never auto-triggered from conversation context.

### Routing table — what triggers what

The skill watches the conversation and consults the right folder automatically:

| What you say / what's happening | Skill routes AI to |
| --- | --- |
| "What were we doing?" / session start | `flightdeck/cockpit.md` |
| "Why did the migration break?" | `flightdeck/incident-reports/` (then debug) |
| "How do I run the tests?" | `flightdeck/checklists/` |
| "Let's design a new X" | `flightdeck/specs/` |
| "Break this into tasks" | `flightdeck/flight-plans/` |
| "Here's review feedback from another AI" | `flightdeck/safety-reviews/` (must add disposition) |
| "Save this for later" | `flightdeck/sketches/` (or refused if low-signal) |

### Session end

Say "let's wrap up" or similar. The AI runs the [landing ritual](skills/flightdeck-workflow/exit-ritual.md):

```
1. Apply classification heuristics to any new knowledge (bug → incident-reports/, 
   procedure → checklists/, one-off → no write)
2. Update cockpit.md (Last updated, Next session) + manifest/logbook as needed
3. Commit
```

The next session — even a different AI, even a different developer — picks up exactly where this one stopped.

## Compatibility

| Tool | Status | Manifest |
| --- | --- | --- |
| Claude Code | ✅ tested | [`.claude-plugin/`](.claude-plugin/) |
| Codex CLI / App | ⚠️ untested | [`.codex-plugin/`](.codex-plugin/) |
| Cursor | ⚠️ untested | [`.cursor-plugin/`](.cursor-plugin/) |
| Gemini CLI | ⚠️ untested | [`gemini-extension.json`](gemini-extension.json) + [`GEMINI.md`](GEMINI.md) |

The skill content under [`skills/`](skills/) is **tool-agnostic markdown**. Manifests are thin pointers that let each AI tool discover the skill. "Untested" means the manifest is in place and the install command above should work, but no one has verified the AI actually follows the protocol end-to-end. PRs with verification logs welcome.

## Documentation

- [SKILL.md](skills/flightdeck-workflow/SKILL.md) — the entry point your AI loads
- [folder-semantics.md](skills/flightdeck-workflow/folder-semantics.md) — what each folder holds and why
- [templates.md](skills/flightdeck-workflow/templates.md) — incident-report / sketch / safety-review / cockpit / INDEX templates
- [preflight SKILL.md](skills/preflight/SKILL.md) — explicit `/flightdeck:preflight` slash command
- [landing SKILL.md](skills/landing/SKILL.md) — explicit `/flightdeck:landing` slash command
- [exit-ritual.md](skills/flightdeck-workflow/exit-ritual.md) — session-end decision tree
- [TEST_PLAN.md](TEST_PLAN.md) — the RED-GREEN-REFACTOR cycle status
- [MIGRATION.md](MIGRATION.md) — workshop → flightdeck upgrade notes

## Contributing

### Verify a manifest actually works

The Codex / Cursor / Gemini manifests are in place but **behaviorally untested**. The most valuable PR right now: install on one of those tools, run a short session in a project with `flightdeck/`, and confirm the AI honors entry / triggers / landing. Open a PR with the transcript and flip the matrix from ⚠️ untested to ✅ tested.

### Skill improvements

Skill changes follow a RED-GREEN-REFACTOR discipline: **no edit without a failing test first**. See [TEST_PLAN.md](TEST_PLAN.md) for the test methodology.

If you find a rationalization the skill doesn't address (an agent that wriggled out of the protocol), open an issue with the transcript — that's the most valuable contribution.

## Why not just AGENTS.md?

[AGENTS.md](https://agents.md) is the cross-tool standard for project-level AI instructions, stewarded under the Linux Foundation and adopted by 60,000+ repos by mid-2026 — a controlled study showed 28.6% runtime reduction and 16.6% token reduction. If you only need "give the AI a static list of project rules", AGENTS.md alone is enough; flightdeck is overkill.

Flightdeck fits **on top of** AGENTS.md, not in place of it. Different concerns:

| Concern | AGENTS.md alone | Flightdeck |
| --- | --- | --- |
| Static project rules / style guide | ✓ | (use AGENTS.md) |
| Session-to-session continuity (cockpit, hand-off) | — | ✓ |
| Lifecycle state machine (spec → plan → landed) | — | ✓ |
| Write gate against junk-drawer accumulation | — | ✓ |
| Incident report log (what went wrong, root causes) | — | ✓ |
| External review disposition tracking | — | ✓ |
| Cross-tool reach | native | via `/flightdeck:emit-agents-md` |

Flightdeck **emits** into AGENTS.md — `/flightdeck:emit-agents-md` regenerates a fenced block in `AGENTS.md` from `flightdeck/cockpit.md`. It doesn't replace AGENTS.md or compete with it: flightdeck is the operating protocol, AGENTS.md is the wire format.

If you're using a tool that consumes AGENTS.md natively (Codex CLI, Copilot, Cursor, Windsurf, Continue, Cody): flightdeck's emitter is the bridge. Maintain `cockpit.md` once; AI tools that read AGENTS.md see fresh project state.

## Roadmap

See [TEST_PLAN.md](TEST_PLAN.md) for the v1.0 release gate. Beyond v1.0:

- **Continuance benchmark**: a "pick up the thread" test suite for any AI agent. Give it a mid-project flightdeck/, say "continue", measure recovery.
- **Synthesis / compression**: tools for compressing many archived specs into themed retrospectives.
- **Live INDEX automation**: optional hook to keep `INDEX.md` AUTO-sections in sync without manual intervention.
- **Verify Codex / Cursor / Gemini end-to-end** (PRs welcome — manifests already in place).
- **Optional new folders** (briefing/, blackbox/, crew-handover/, experiments/) — deferred from v1.0 rebrand; revisit when real usage justifies each.

## License

MIT. See [LICENSE](LICENSE).
