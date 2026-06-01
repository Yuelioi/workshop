---
name: preflight
description: Use when explicitly invoking the flightdeck entry ritual — reconciles cockpit.md against repo state via root INDEX.md, loads a routing catalog from folder INDEX files (not per-file frontmatter), and reports the first "next session" item. Triggered by `/flightdeck:preflight`.
disable-model-invocation: true
---

# Flightdeck Preflight

User-triggered explicit entry ritual: reconcile `cockpit.md` against repo state (via root INDEX), load a routing catalog (via folder INDEX files), and **report** the next item — then stop. It does not execute the item; that's the next turn's job. Useful when:

- A long session has gone off the rails and you want to re-anchor on the cockpit.
- The auto-loaded `workflow` didn't fire (e.g. fresh `flightdeck/` mid-session).
- The user wants a clean, predictable starting point before delegating to other skills.

## Run this checklist exactly

0. **Read `flightdeck/rules.md`** if present. Apply its toggles for the whole ritual: when `git: false`, skip step 2's git reconcile entirely; honor `disabled_folders` (don't suggest them in fallback).

1. **Detect 1.x layout (non-silent).** If ANY of the following exists, stop and report before reconciling:
   - `flightdeck/manifest.md`
   - `flightdeck/logbook.md`
   - `flightdeck/kneeboard/`
   - `flightdeck/flight-plans/`
   - `flightdeck/incident-reports/`
   - `flightdeck/safety-reviews/`

   Tell the user: "1.x layout detected — migrate to 1.2?" and follow [MIGRATION.md](../../MIGRATION.md). Never migrate silently. Do not proceed with the rest of the checklist until the user decides.

2. **Read `flightdeck/INDEX.md`** (root INDEX) once, in full — it carries the global status summary (counts per folder). Then **read `flightdeck/cockpit.md`** once, in full — focus on `Last updated`, `Active focus`, and the `## Next session` section. These two reads together are the reconcile baseline; do not re-read either during the ritual.

3. **(skip entirely when `rules.md` sets `git: false`) Reconcile against repo state.** Run these checks independently (in parallel where supported):
   - `git branch --show-current` — matches `Active focus` in cockpit?
   - `git status --short` — does the first "Next session" item show up as in-progress files?
   - `git stash list` — any entries not mentioned in cockpit?
   - `git log -1 --format=%cs` — is `Last updated` more than ~14 days behind the most recent commit? (When `git: false`, compare against the newest `landed/HISTORY.md` entry instead.)

   Cross-check cockpit's `## Next session` against reality (branch, tree state). Flag any mismatch.

4. **Mismatch handling** — **always ask the user before acting**:
   - If branch differs: "Cockpit says focus is X but branch is Y — which is current?"
   - If `git status` is clean but cockpit says in-progress: "Cockpit flags 'X in progress' but tree is clean — did it ship?"
   - If stash exists not in cockpit: "Stash entry from <date> not on cockpit — pick up, drop, or note?"
   - If cockpit > 14 days stale: "Cockpit last updated <date>, most recent commit <date>. Cockpit may be stale — refresh first?"

5. **Load the routing catalog** (know-what-exists, NOT read-all). Read the folder INDEX files — do NOT glob individual files or read per-file frontmatter:
   - Read `flightdeck/checklists/INDEX.md` — it already lists each checklist's `when_to_read`, `applies_to`, and `status`.
   - Read `flightdeck/incidents/INDEX.md` — same structure for incident files.
   - If either INDEX is missing or obviously stale (file count in INDEX differs from root INDEX count), note it: "⚠ `<folder>/INDEX.md` missing or stale — walkaround owns the fix." This is non-blocking.
   - **Do NOT read individual checklist or incident files** at catalog time. The folder INDEX is the catalog. Only drill into an individual file when a trigger actually matches the current task (i.e. at execution time, not preflight time).

6. **Status sanity (from INDEX).** Scan each folder INDEX row for a missing `status` or an illegal status value for that folder's kind; report and offer to fix, non-silent. (Deeper file audits belong to walkaround.)

   Valid status values by folder:
   - Workflow folders (`sketches/`, `specs/`, `plans/`): `pending / active / awaiting-review / blocked / done / scrapped` (sketches: typically only `active / scrapped`)
   - Knowledge folders (`incidents/`, `checklists/`, `charts/`, `debriefs/`): `active / obsolete / superseded`

   List all findings before offering fixes. Offer once: "Fix all flagged files?" — do not fix any until the user confirms.

7. **All reconciled → report item #1, then STOP.** Read-only recon doesn't fly the mission. State the item in one sentence and hand off: "Preflight complete (read-only). Say 'go' to execute item #1." Do not load any file body or start the task — that's the next turn.

## Fallback when "Next session" is empty

Don't auto-start anything. Search in order (a missing directory counts as empty), present candidates to user:

1. `flightdeck/plans/` — surface `pending` / `blocked` / `active` plans (read `plans/INDEX.md`), most actionable first; a `done`-but-unlanded plan → offer to land it.
2. `flightdeck/sketches/` — unstarted ideas (read `sketches/INDEX.md`); ask which (if any) to promote to a spec.

## Output format

Report concisely:

```
Root INDEX: specs/ — 2 (1 active, 1 done) | plans/ — 1 active | incidents/ — 1 active | checklists/ — 2 active
Cockpit reconciled (Last updated: 2026-05-25; Active focus: <X>; tree clean)

Routing catalog (from folder INDEX files — know-what-exists, not read-all):

[Checklists]
| File | when_to_read | applies_to | status |
|---|---|---|---|
| checklists/comments.md | before writing or editing any source-code comment | comments, code-style | active |

[Incidents]
| File | when_to_read | applies_to | status |
|---|---|---|---|
| incidents/parser-recursion.md | before designing a recursive parser | parser, recursion | active |

[Catalog notes]  (omitted when clean)
- ⚠ incidents/INDEX.md missing — walkaround owns the fix

Next session item #1: <item description>

Preflight complete (read-only). Catalog is know-what-exists only — NOT a substitute for /flightdeck:walkaround, and does not mean these files were read. Bodies load on demand, when execution begins and a trigger matches.

→ Say "go" to execute item #1.
```

Omit any table group with no entries. If both folder INDEX files are absent or empty, print `Routing catalog: (empty — no routed resources yet)`.

Or if blocked:

```
Reconcile flagged:
- <mismatch 1>
- <mismatch 2>

Resolve which?
```

## Don't do

- Don't auto-execute item #1 — report and stop.
- Don't auto-pick a fallback when `Next session` is empty — always ask.
- Don't bump `Last updated` — entry doesn't modify cockpit.
- Don't glob individual checklist/incident files or read per-file frontmatter for the catalog — read folder INDEX files only.
- Don't drill into individual files until a trigger matches at execution time.
- Don't grep the codebase for "things to do" — cockpit.md is authoritative.
- Don't migrate silently — always ask the user before any structural change.
