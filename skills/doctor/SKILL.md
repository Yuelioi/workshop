---
name: doctor
description: Use when explicitly invoking the workshop integrity audit — checks board.md / specs / plans / scars / playbooks / wip for protocol drift, dangling references, stale entries, and lifecycle mismatches. Triggered by `/workshop:doctor`.
disable-model-invocation: true
---

# Workshop Doctor

User-triggered integrity audit of a workshop for protocol drift. The protocol is markdown + filesystem conventions; drift is the silent killer of advice systems. Doctor surfaces drift loudly so the author can fix it. Implemented as a slash skill (markdown checklist the AI follows), NOT a CLI binary, to preserve workshop's plain-markdown + git bet.

## When to invoke

- Periodically (e.g., before a release commit, weekly during active work).
- After a long absence from the project — drift accumulates silently.
- When something feels off (board doesn't match reality, scars unread, AGENTS.md stale).
- Before promoting workshop conventions to a downstream project — doctor should be ✅ clean on the source.

## Severity legend

- **CRITICAL** — protocol contract broken (e.g., scar missing required frontmatter, dangling internal reference). Fix before proceeding with new work.
- **WARNING** — drift that will accumulate (e.g., stale wip, missing AGENTS.md regeneration, stale Blockers). Fix soon, before the next release.
- **INFO** — heads-up that may or may not need action (e.g., orphan scar, approaching Recently finished cap). Judge per item.

## Audits

Run all 8 in order. For each, report findings with the severity tag.

### 1. Scars / playbooks frontmatter (CRITICAL on miss)

For each `workshop/scars/*.md` and `workshop/playbooks/*.md` (NOT in `finish/`):
- Read the frontmatter (between `---` markers).
- Confirm `when_to_read`, `applies_to`, `last_updated` are ALL present.
- If any missing: **CRITICAL** — file is invisible to workshop routing per the v0.6 hard-fail rule (see [workshop-workflow/SKILL.md § Frontmatter requirements](../workshop-workflow/SKILL.md#frontmatter-requirements-hard-fail)).

### 2. Stale wip files (WARNING on miss)

For each `workshop/wip/*.md`:
- Read frontmatter for `last_touched:`.
- If missing: **WARNING** — `last_touched:` is required (see [templates.md § wip](../workshop-workflow/templates.md#wip)).
- If present but date predates the most recent commit by ≥ 7 days: **WARNING** — file has survived multiple session-exits. Either has a `defer_reason:` (acceptable, report as INFO instead) or should have been classified/deleted.

(Doctor uses a stricter 7-day default than session-exit's "predates current session" — doctor catches lingering wip that survived multiple session-exits.)

### 3. Dangling internal references (CRITICAL on miss)

For each markdown file under `workshop/` and at repo root (`README.md`, `CHANGELOG.md`, `AGENTS.md`, `TEST_PLAN.md`, etc.):
- Extract all markdown links of the form `[text](path)` where `path` is NOT an HTTP/HTTPS URL and NOT an anchor-only (`#...`).
- For each link, resolve the path relative to the file's directory.
- Check if the target file exists.
- If not: **CRITICAL** — broken cross-reference. Report the source file:line + the broken target path.

### 4. Orphan scars (INFO)

For each `workshop/scars/*.md`:
- If `status: superseded` and no `→ project-rules` annotation (or similar promotion marker) in the file: **INFO** — was the upgrade actually declared?
- If `status: obsolete` and `last_updated` < 90 days old: **INFO** — is it really obsolete, or did someone forget to re-evaluate?

### 5. Board ↔ folder lifecycle mismatch (WARNING)

Compare `workshop/board.md` `## In flight` table against `specs/` and `plans/` folder state:
- Files in `specs/` or `plans/` (NOT in `finish/`) with frontmatter `state: blocked` or `state: awaiting-review` MUST have a row in board `In flight`. Missing row: **WARNING**.
- Files in `specs/finish/` or `plans/finish/` must NOT have a `state:` field (or it must be `done`). Stray state: **WARNING**.
- Every row in board `In flight` must point to a real file. Broken row: **WARNING**.

### 6. Stale Blockers entries (WARNING)

For each bullet in board's `## Blockers` section:
- If the bullet references a shipped version (e.g., "v0.5 work" when v0.5+ shipped per `CHANGELOG.md` / git tags): **WARNING**.
- If the bullet's described condition is described in past tense or refers to a completed deliverable: **WARNING**.

Cleanup is one-line — usually rephrase to point at the current open item, or remove if no longer relevant.

### 7. Recently finished length (INFO / WARNING)

Count entries under board's `## Recently finished`:
- 5 entries: ✅ at cap, no action.
- 6+ entries: **WARNING** — exit-ritual auto-trim should have prevented this. Drop oldest until count = 5.
- 0 entries: **INFO** — new project; nothing shipped yet.

### 8. AGENTS.md regeneration drift (WARNING)

If `AGENTS.md` exists at repo root with workshop markers (`<!-- BEGIN: workshop -->` / `<!-- END: workshop -->`):
- Extract the workshop block content.
- Mentally re-run the recipe from `skills/emit-agents-md/SKILL.md` against current `workshop/board.md` (Active focus, Next session, In flight, Hanging tasks).
- Compare. If different in any meaningful field: **WARNING** — emit-agents-md hasn't been re-run since `board.md` changed.

If `AGENTS.md` doesn't exist or has no workshop markers: skip (the project hasn't dogfooded the emitter yet; that's optional).

## Output format

```
=== /workshop:doctor report ===
Audit run: <ISO date>
Workshop root: <path>

CRITICAL findings (N):
  - <file:line> — <issue>
  - ...

WARNING findings (N):
  - ...

INFO findings (N):
  - ...

Total: N findings (X CRITICAL, Y WARNING, Z INFO)
```

If no findings overall:

```
=== /workshop:doctor report ===
Audit run: <ISO date>
Workshop root: <path>

✅ Clean.
```

Omit any severity line whose count is 0.

## Handling findings

- **CRITICAL**: fix before any other work. These are broken contracts.
- **WARNING**: schedule for the current session if quick; otherwise add a hanging task to board.
- **INFO**: judge per item. Some are useful nudges; some are noise. Don't auto-fix.

Doctor never auto-fixes. The author decides.

## Don't do

- Don't auto-fix any finding — doctor surfaces, author resolves.
- Don't run doctor against other repositories or foreign `workshop/` directories — false drift signals.
- Don't include `*/finish/` archived files in most audits — they're history, not subject to current-state rules.
- Don't fail loudly on optional missing folders (a project with no `scars/` directory is fine).
- Don't bump board `Last updated` from running doctor — doctor is read-only by design.
