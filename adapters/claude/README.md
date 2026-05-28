# Adapter: Claude Code

**Status**: ✅ tested

## Manifests

- [`.claude-plugin/plugin.json`](../../.claude-plugin/plugin.json) — plugin manifest.
- [`.claude-plugin/marketplace.json`](../../.claude-plugin/marketplace.json) — self-hosted marketplace declaration.

## Install — primary path (plugin marketplace)

In any Claude Code session:

```text
/plugin marketplace add Yuelioi/flightdeck
/plugin install flightdeck@flightdeck-marketplace
```

To update: re-run `/plugin install`. To uninstall: `/plugin uninstall flightdeck`.

This is the recommended path — it gives proper version tracking and lifecycle.

## Install — alternative (direct copy)

For users who don't want to use the plugin marketplace, the installers at the repo root copy `skills/flightdeck-workflow/` directly into the user-level Claude Code skills directory.

| OS | Target path |
| --- | --- |
| macOS / Linux | `~/.claude/skills/flightdeck-workflow/` |
| Windows | `%USERPROFILE%\.claude\skills\flightdeck-workflow\` |

```powershell
.\install.ps1
```

```bash
./install.sh
```

After install:

```
~/.claude/skills/flightdeck-workflow/   # auto-loaded via SessionStart hook
├── SKILL.md
├── folder-semantics.md
├── templates.md
└── exit-ritual.md
~/.claude/skills/preflight/             # /flightdeck:preflight explicit trigger
└── SKILL.md
~/.claude/skills/landing/               # /flightdeck:landing explicit trigger
└── SKILL.md
~/.claude/skills/walkaround/            # /flightdeck:walkaround integrity audit
└── SKILL.md
~/.claude/skills/emit-agents-md/        # /flightdeck:emit-agents-md AGENTS.md emitter
└── SKILL.md
```

## Verification

After install (either path), in a Claude Code session:

1. Start a session in any project directory.
2. The `flightdeck-workflow` skill should appear in the available skills list with description starting "Use when a project has a flightdeck/ directory...".
3. Force-invoke with `/flightdeck:flightdeck-workflow` and confirm the entry checklist runs.
4. Force-invoke `/flightdeck:preflight` and `/flightdeck:landing` — these should run the corresponding rituals explicitly.

If the skill does not appear:
- Direct install: check `ls ~/.claude/skills/flightdeck-workflow/SKILL.md` exists.
- Marketplace install: check `~/.claude/plugins/` for the cached plugin.
- Either: verify SKILL.md frontmatter is intact (`name:` and `description:`).

## How invocation works

- The skill is loaded automatically when Claude detects its description matches the session context (a project with `flightdeck/`).
- Force-invoke via `/flightdeck-workflow`.
- Flightdeck is **self-contained**: it does not require any other plugin to function. If you also have `superpowers` installed, the SKILL.md mentions its `brainstorming` / `writing-plans` skills as optional companions — fine if present, fine if absent.

## Uninstall

Marketplace path:

```text
/plugin uninstall flightdeck
```

Direct path:

```bash
# macOS / Linux
rm -rf ~/.claude/skills/flightdeck-workflow
```

```powershell
# Windows
Remove-Item -Recurse -Force "$env:USERPROFILE\.claude\skills\flightdeck-workflow"
```
