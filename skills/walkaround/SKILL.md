---
name: walkaround
description: Use when explicitly invoking the flightdeck integrity audit — checks cockpit.md / manifest.md / logbook.md / specs / flight-plans / incident-reports / checklists / kneeboard for protocol drift, dangling references, stale entries, and lifecycle mismatches. Triggered by `/flightdeck:walkaround`.
disable-model-invocation: true
---

# Flightdeck Walkaround

User-triggered integrity audit of a flightdeck for protocol drift. The protocol is markdown + filesystem conventions; drift is the silent killer of advice systems. Walkaround surfaces drift loudly so the author can fix it. Implemented as a slash skill (markdown checklist the AI follows), NOT a CLI binary, to preserve flightdeck's plain-markdown + git bet.

## When to invoke

- Periodically (e.g., before a release commit, weekly during active work).
- After a long absence from the project — drift accumulates silently.
- When something feels off (cockpit doesn't match reality, incident-reports unread, AGENTS.md stale).
- Before promoting flightdeck conventions to a downstream project — walkaround should be ✅ clean on the source.

## Severity legend

- **CRITICAL** — protocol contract broken (e.g., scar missing required frontmatter, dangling internal reference). Fix before proceeding with new work.
- **WARNING** — drift that will accumulate (e.g., stale wip, missing AGENTS.md regeneration, stale Blockers). Fix soon, before the next release.
- **INFO** — heads-up that may or may not need action (e.g., orphan scar, approaching Recently finished cap). Judge per item.

## Audits

Run all 8 in order. For each, report findings with the severity tag.

### 1. Incident reports / checklists frontmatter (CRITICAL on miss)

For each `flightdeck/incident-reports/*.md` and `flightdeck/checklists/*.md` (NOT in `landed/`):
- Read the frontmatter (between `---` markers).
- Confirm `when_to_read`, `applies_to`, `last_updated` are ALL present.
- If any missing: **CRITICAL** — file is invisible to flightdeck routing per the v0.6 hard-fail rule (see [flightdeck-workflow/SKILL.md § Frontmatter requirements](../flightdeck-workflow/SKILL.md#frontmatter-requirements-hard-fail)).

### 2. Stale kneeboard files (WARNING on miss)

For each `flightdeck/kneeboard/*.md`:
- Read frontmatter for `last_touched:`.
- If missing: **WARNING** — `last_touched:` is required (see [templates.md § kneeboard](../flightdeck-workflow/templates.md#kneeboard)).
- If present but date predates the most recent commit by ≥ 7 days: **WARNING** — file has survived multiple landings. Either has a `defer_reason:` (acceptable, report as INFO instead) or should have been classified/deleted.

(Walkaround uses a stricter 7-day default than landing's "predates current session" — walkaround catches lingering kneeboard files that survived multiple landings.)

### 3. Dangling internal references (CRITICAL on miss)

For each markdown file under `flightdeck/` and at repo root (`README.md`, `CHANGELOG.md`, `AGENTS.md`, `TEST_PLAN.md`, etc.):
- Extract all markdown links of the form `[text](path)` where `path` is NOT an HTTP/HTTPS URL and NOT an anchor-only (`#...`).
- For each link, resolve the path relative to the file's directory.
- Check if the target file exists.
- If not: **CRITICAL** — broken cross-reference. Report the source file:line + the broken target path.

### 4. Orphan incident reports (INFO)

For each `flightdeck/incident-reports/*.md`:
- If `status: superseded` and no `→ project-rules` annotation (or similar promotion marker) in the file: **INFO** — was the upgrade actually declared?
- If `status: obsolete` and `last_updated` < 90 days old: **INFO** — is it really obsolete, or did someone forget to re-evaluate?

### 5. Manifest ↔ folder lifecycle mismatch (WARNING)

Compare `flightdeck/manifest.md` `## In flight` table against `specs/` and `flight-plans/` folder state:
- Files in `specs/` or `flight-plans/` (NOT in `landed/`) with frontmatter `state: blocked` or `state: awaiting-review` MUST have a row in manifest `In flight`. Missing row: **WARNING**.
- Files in `landed/specs/` or `landed/flight-plans/` must NOT have a `state:` field (or it must be `done`). Stray state: **WARNING**.
- Every row in manifest `In flight` must point to a real file. Broken row: **WARNING**.

### 6. Stale Blockers entries (WARNING)

For each bullet in manifest's `## Blockers` section:
- If the bullet references a shipped version (e.g., "v0.5 work" when v0.5+ shipped per `CHANGELOG.md` / git tags): **WARNING**.
- If the bullet's described condition is described in past tense or refers to a completed deliverable: **WARNING**.

Cleanup is one-line — usually rephrase to point at the current open item, or remove if no longer relevant.

### 7. Recently finished length (INFO / WARNING)

Count entries under logbook's `## Recently finished`:
- 5 entries: ✅ at cap, no action.
- 6+ entries: **WARNING** — exit-ritual auto-trim should have prevented this. Drop oldest until count = 5.
- 0 entries: **INFO** — new project; nothing shipped yet.

### 8. AGENTS.md regeneration drift (WARNING)

If `AGENTS.md` exists at repo root with flightdeck markers (`<!-- BEGIN: flightdeck -->` / `<!-- END: flightdeck -->`):
- Extract the flightdeck block content.
- Mentally re-run the recipe from `skills/emit-agents-md/SKILL.md` against current `flightdeck/cockpit.md` (Active focus, Next session, Hanging tasks) and `flightdeck/manifest.md` (In flight).
- Compare. If different in any meaningful field: **WARNING** — emit-agents-md hasn't been re-run since `cockpit.md` changed.

If `AGENTS.md` doesn't exist or has no flightdeck markers: skip (the project hasn't dogfooded the emitter yet; that's optional).

## Output format

```
=== /flightdeck:walkaround report ===
Audit run: <ISO date>
Flightdeck root: <path>

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
=== /flightdeck:walkaround report ===
Audit run: <ISO date>
Flightdeck root: <path>

✅ Clean.
```

Omit any severity line whose count is 0.

## Handling findings

- **CRITICAL**: fix before any other work. These are broken contracts.
- **WARNING**: schedule for the current session if quick; otherwise add a hanging task to cockpit.md.
- **INFO**: judge per item. Some are useful nudges; some are noise. Don't auto-fix.

Walkaround never auto-fixes. The author decides.

## Don't do

- Don't auto-fix any finding — walkaround surfaces, author resolves.
- Don't run walkaround against other repositories or foreign `flightdeck/` directories — false drift signals.
- Don't include `landed/` archived files in most audits — they're history, not subject to current-state rules.
- Don't fail loudly on optional missing folders (a project with no `incident-reports/` directory is fine).
- Don't bump cockpit `Last updated` from running walkaround — walkaround is read-only by design.
