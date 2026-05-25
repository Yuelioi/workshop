# Adapter: Claude Code

**Status**: ✅ tested

## Manifests

- [`.claude-plugin/plugin.json`](../../.claude-plugin/plugin.json) — plugin manifest.
- [`.claude-plugin/marketplace.json`](../../.claude-plugin/marketplace.json) — self-hosted marketplace declaration.

## Install — primary path (plugin marketplace)

In any Claude Code session:

```text
/plugin marketplace add Yuelioi/workshop
/plugin install workshop@workshop-marketplace
```

To update: re-run `/plugin install`. To uninstall: `/plugin uninstall workshop`.

This is the recommended path — it gives proper version tracking and lifecycle.

## Install — alternative (direct copy)

For users who don't want to use the plugin marketplace, the installers at the repo root copy `skills/workshop-workflow/` directly into the user-level Claude Code skills directory.

| OS | Target path |
| --- | --- |
| macOS / Linux | `~/.claude/skills/workshop-workflow/` |
| Windows | `%USERPROFILE%\.claude\skills\workshop-workflow\` |

```powershell
.\install.ps1
```

```bash
./install.sh
```

After install (v0.5.0+):

```
~/.claude/skills/workshop-workflow/   # auto-loaded via SessionStart hook
├── SKILL.md
├── folder-semantics.md
├── templates.md
└── exit-ritual.md
~/.claude/skills/session-enter/       # /workshop:session-enter explicit trigger
└── SKILL.md
~/.claude/skills/session-exit/        # /workshop:session-exit explicit trigger
└── SKILL.md
```

## Verification

After install (either path), in a Claude Code session:

1. Start a session in any project directory.
2. The `workshop-workflow` skill should appear in the available skills list with description starting "Use when a project has a workshop/ directory...".
3. Force-invoke with `/workshop:workshop-workflow` and confirm the entry checklist runs.
4. (v0.5.0+) Force-invoke `/workshop:session-enter` and `/workshop:session-exit` — these should run the corresponding rituals explicitly.

If the skill does not appear:
- Direct install: check `ls ~/.claude/skills/workshop-workflow/SKILL.md` exists.
- Marketplace install: check `~/.claude/plugins/` for the cached plugin.
- Either: verify SKILL.md frontmatter is intact (`name:` and `description:`).

## How invocation works

- The skill is loaded automatically when Claude detects its description matches the session context (a project with `workshop/`).
- Force-invoke via `/workshop-workflow`.
- Workshop is **self-contained**: it does not require any other plugin to function. If you also have `superpowers` installed, the SKILL.md mentions its `brainstorming` / `writing-plans` skills as optional companions — fine if present, fine if absent.

## Uninstall

Marketplace path:

```text
/plugin uninstall workshop
```

Direct path:

```bash
# macOS / Linux
rm -rf ~/.claude/skills/workshop-workflow
```

```powershell
# Windows
Remove-Item -Recurse -Force "$env:USERPROFILE\.claude\skills\workshop-workflow"
```
