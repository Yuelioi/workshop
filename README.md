# workshop

🇨🇳 **中文用户**: see [README.zh.md](README.zh.md) for a Chinese version.

> A persistent workbench protocol for AI coding sessions.

Your AI assistant forgets everything between chats. `workshop` is a directory convention + skill that gives it a persistent workbench across sessions — so the next session knows what you were doing, why, and what to do next.

## What it is

A `workshop/` directory layout your AI reads and writes by convention:

```
workshop/
├── board.md            # Current state — read first, updated last
│
├── specs/              # Design docs                    (when designing)
├── plans/              # Implementation plans           (when breaking down work)
├── playbooks/          # Repeatable procedures          (commands + checklists)
├── scars/              # Lessons learned (no "forgot")  (recurring traps)
├── reference/          # External material              (RFCs, competitor code)
│
├── sketches/           # Long-term ideas                (unstarted)
├── critiques/          # External review feedback       (raw + disposition)
└── wip/                # Session scratch                (one session only)
```

**Organized by when you read what** — not by topic. The folder name tells the AI when to consult it.

## Why it exists

Most "AI memory" systems fail by **saving everything**. The signal drowns. You get a junk drawer that even the AI gives up on reading.

`workshop` does the opposite:

- **Strict write gate**: only content that changes future behavior, influences decisions, or gets referenced repeatedly.
- **Lifecycle for every folder**: `wip/` lives one session; scars upgrade to project rules on third recurrence; specs/plans archive into `finish/` after ship.
- **Authority order**: when sources disagree, the protocol says who wins.
- **Exit ritual**: 90% of session-end classifications are obvious. Only true ambiguity triggers brainstorming.

The full discipline lives in [`skills/workshop-workflow/SKILL.md`](skills/workshop-workflow/SKILL.md).

## Install

### Claude Code ✅ tested

In any Claude Code session:

```text
/plugin marketplace add Yuelioi/workshop
/plugin install workshop@workshop-marketplace
```

To update: re-run `/plugin install`. To uninstall: `/plugin uninstall workshop`.

### Codex CLI ⚠️ manifest in place, behavior untested

```text
/plugins
```

Then search "workshop" → select → `Install Plugin`.

### Cursor ⚠️ manifest in place, behavior untested

In Cursor Agent chat:

```text
/add-plugin workshop
```

Or search "workshop" in the plugin marketplace.

### Gemini CLI ⚠️ manifest in place, behavior untested

```bash
gemini extensions install https://github.com/Yuelioi/workshop
```

Update later:

```bash
gemini extensions update workshop
```

### Alternative — direct skill install (Claude Code only, no marketplace)

For users who prefer a direct copy into `~/.claude/skills/`:

```powershell
# Windows
git clone https://github.com/Yuelioi/workshop.git
cd workshop
.\install.ps1
```

```bash
# macOS / Linux
git clone https://github.com/Yuelioi/workshop.git
cd workshop
./install.sh
```

### Scaffold a `workshop/` in your project

```powershell
.\install.ps1 -Scaffold minimal     # just board.md
.\install.ps1 -Scaffold full        # all 10 subdirs
```

```bash
./install.sh --scaffold=minimal
./install.sh --scaffold=full
```

## Usage

After install, the skill auto-loads whenever your project has a `workshop/` directory. You can also force-invoke it.

### Day 1 — bootstrap a new project

```bash
cd your-project
git clone https://github.com/Yuelioi/workshop /tmp/workshop && \
  /tmp/workshop/install.sh --scaffold=minimal
# or copy scaffolds/minimal/workshop/ from this repo manually
```

Edit `workshop/board.md`:

```markdown
**Active focus**: implement X feature
## Next session
1. Set up data model in src/models/X.ts
```

Open your AI tool in the project. The skill loads automatically.

### What the AI does on every session

```
1. Reads workshop/board.md
2. Reconciles against `git status` (branch, uncommitted changes, stashes)
3. Executes the first "next session" item, or asks if state is inconsistent
```

### Force-invoke in Claude Code

If your project doesn't have `workshop/` yet, or the skill didn't auto-trigger:

```text
/workshop:workshop-workflow
```

This loads the skill explicitly and runs the entry checklist.

### Explicit slash commands (v0.5.0+)

Two focused one-command triggers for re-anchoring mid-session:

```text
/workshop:session-enter   # run entry checklist explicitly (re-read board, reconcile)
/workshop:session-exit    # run exit ritual explicitly (classify, update board, commit)
```

Both bypass model-auto-invocation — they fire only when you type the slash. Useful for:

- Drifted a long session, want to re-anchor on the board (`session-enter`)
- About to wrap, want the checklist enforced (no junk wip / hanging critiques) (`session-exit`)
- Post-ship pause-and-record before moving on (`session-exit`)

These are thin wrappers over `workshop-workflow`'s entry / exit sections — same rules, faster trigger.

### Routing table — what triggers what

The skill watches the conversation and consults the right folder automatically:

| What you say / what's happening | Skill routes AI to |
| --- | --- |
| "What were we doing?" / session start | `workshop/board.md` |
| "Why did the migration break?" | `workshop/scars/` (then debug) |
| "How do I run the tests?" | `workshop/playbooks/` |
| "Let's design a new X" | `workshop/specs/` |
| "Break this into tasks" | `workshop/plans/` |
| "Here's review feedback from another AI" | `workshop/critiques/` (must add disposition) |
| "Save this for later" | `workshop/sketches/` (or refused if low-signal) |

### Session end

Say "let's wrap up" or similar. The AI runs the [exit ritual](skills/workshop-workflow/exit-ritual.md):

```
1. Apply classification heuristics to any new knowledge (bug → scars/, 
   procedure → playbooks/, one-off → no write)
2. Update board.md (Last updated, Next session, In flight)
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

- [SKILL.md](skills/workshop-workflow/SKILL.md) — the entry point your AI loads
- [folder-semantics.md](skills/workshop-workflow/folder-semantics.md) — what each folder holds and why
- [templates.md](skills/workshop-workflow/templates.md) — scar / sketch / critique / board / INDEX templates
- [session-enter SKILL.md](skills/session-enter/SKILL.md) — explicit `/workshop:session-enter` slash command
- [session-exit SKILL.md](skills/session-exit/SKILL.md) — explicit `/workshop:session-exit` slash command
- [exit-ritual.md](skills/workshop-workflow/exit-ritual.md) — session-end decision tree
- [TEST_PLAN.md](TEST_PLAN.md) — the RED-GREEN-REFACTOR cycle status (skill is currently v0.x, pre-test)

## Contributing

### Verify a manifest actually works

The Codex / Cursor / Gemini manifests are in place but **behaviorally untested**. The most valuable PR right now: install on one of those tools, run a short session in a project with `workshop/`, and confirm the AI honors entry / triggers / exit. Open a PR with the transcript and flip the matrix from ⚠️ untested to ✅ tested.

### Skill improvements

Skill changes follow a RED-GREEN-REFACTOR discipline: **no edit without a failing test first**. See [TEST_PLAN.md](TEST_PLAN.md) for the test methodology.

If you find a rationalization the skill doesn't address (an agent that wriggled out of the protocol), open an issue with the transcript — that's the most valuable contribution.

## Why not just AGENTS.md?

[AGENTS.md](https://agents.md) is the cross-tool standard for project-level AI instructions, stewarded under the Linux Foundation and adopted by 60,000+ repos by mid-2026 — a controlled study showed 28.6% runtime reduction and 16.6% token reduction. If you only need "give the AI a static list of project rules", AGENTS.md alone is enough; workshop is overkill.

Workshop fits **on top of** AGENTS.md, not in place of it. Different concerns:

| Concern | AGENTS.md alone | Workshop |
| --- | --- | --- |
| Static project rules / style guide | ✓ | (use AGENTS.md) |
| Session-to-session continuity (board, hand-off) | — | ✓ |
| Lifecycle state machine (spec → plan → done) | — | ✓ |
| Write gate against junk-drawer accumulation | — | ✓ |
| Scar log (what went wrong, root causes) | — | ✓ |
| External critique disposition tracking | — | ✓ |
| Cross-tool reach | native | via `/workshop:emit-agents-md` |

Workshop **emits** into AGENTS.md — `/workshop:emit-agents-md` regenerates a fenced block in `AGENTS.md` from `workshop/board.md`. It doesn't replace AGENTS.md or compete with it: workshop is the operating protocol, AGENTS.md is the wire format.

If you're using a tool that consumes AGENTS.md natively (Codex CLI, Copilot, Cursor, Windsurf, Continue, Cody): workshop's emitter is the bridge. Maintain `board.md` once; AI tools that read AGENTS.md see fresh project state.

## Roadmap

See [TEST_PLAN.md](TEST_PLAN.md) for the v1.0 release gate. Beyond v1.0:

- **Continuance benchmark**: a "pick up the thread" test suite for any AI agent. Give it a mid-project workshop/, say "continue", measure recovery.
- **Synthesis / compression**: tools for compressing many archived specs into themed retrospectives.
- **Live INDEX automation**: optional hook to keep `INDEX.md` AUTO-sections in sync without manual intervention.
- **Verify Codex / Cursor / Gemini end-to-end** (PRs welcome — manifests already in place).

## License

MIT. See [LICENSE](LICENSE).
