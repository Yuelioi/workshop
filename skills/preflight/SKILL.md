---
name: preflight
description: Use when explicitly invoking the flightdeck entry ritual — reconciles cockpit.md against repo state, runs the staleness check, surfaces stale kneeboard files, loads a routing catalog (checklists / incident-reports / bundle READMEs with their when_to_read + last_updated), and reports the first "next session" item. Triggered by `/flightdeck:preflight`.
disable-model-invocation: true
---

# Flightdeck Preflight

User-triggered explicit entry ritual. Same logic as the [workflow](../workflow/SKILL.md) skill's Entry Checklist section, exposed as a one-command slash for cases where:

- A long session has gone off the rails and you want to re-anchor on the cockpit.
- The auto-loaded `workflow` didn't fire (e.g. fresh `flightdeck/` mid-session).
- The user wants a clean, predictable starting point before delegating to other skills.

## Run this checklist exactly

1. **Read `flightdeck/cockpit.md`** — focus on `Last updated` and the "Next session" section.

2. **Reconcile against repo state.** Run these checks in parallel:
   - `git branch --show-current` — matches `Active focus`?
   - `git status --short` — does the first "Next session" item show up as in-progress files?
   - `git stash list` — any entries not mentioned in cockpit?
   - `git log -1 --format=%cs` — is `Last updated` more than ~14 days behind the most recent commit?

3. **Surface stale `kneeboard/` files.** For each file in `flightdeck/kneeboard/` whose `last_touched:` frontmatter predates the current session's start (or is missing): report it to the user with the file path. Do NOT auto-delete or auto-classify. The user resolves at landing; this step exists so they enter the session aware of pending hygiene debt.

4. **Mismatch handling** — **always ask the user before acting**:
   - If branch differs: "Cockpit says focus is X but branch is Y — which is current?"
   - If `git status` is clean but cockpit says in-progress: "Cockpit flags 'X in progress' but tree is clean — did it ship?"
   - If stash exists not in cockpit: "Stash entry from <date> not on cockpit — pick up, drop, or note?"
   - If cockpit > 14 days stale: "Cockpit last updated <date>, most recent commit <date>. Cockpit may be stale — refresh first?"

5. **Load the routing catalog** (know-what-exists, NOT read-all). Build a compact table of routed resources so their triggers are in context before work starts:
   - **Discover** recursively under `flightdeck/checklists/` and `flightdeck/incident-reports/` (exclude anything under `landed/`): every flat `*.md`, and every subdirectory `README.md`.
   - **Read and parse** each file's frontmatter (the YAML between `---`) as YAML — do not pattern-match a single line; multi-line / quoted / comma-bearing `when_to_read` values must survive.
   - **Extract** per file: path, `when_to_read`, `applies_to`, `last_updated`. Do NOT extract `skip_when` (it is a match-time negative-routing concern, not a catalog one).
   - **Classify by kind**: flat file in `checklists/` → checklist; flat file in `incident-reports/` → incident-report; subdirectory `README.md` with `bundle: true` → bundle; subdirectory `README.md` lacking `bundle: true` → malformed bundle.
   - **Do NOT list bundle leaves** (non-README files inside a bundle): they carry no routing frontmatter and are reached via the README's `reading_order`. Listing them breaks the single-entry guarantee.
   - **Never let a file vanish**: if frontmatter won't parse or `when_to_read` is missing, still list the file with a `⚠ parse error` / `⚠ missing when_to_read` marker. All markers are non-blocking — preflight is read-only; hard-fail enforcement stays in `workflow` / `walkaround`.
   - Print the catalog in the grouped format defined in [Output format](#output-format).
6. **All reconciled → execute the first "Next session" item.**
   Before executing, match the task's keywords / touched paths against the catalog's `applies_to` and load the matching bodies. Then state the item back to the user in one sentence ("Executing: [item description]") and proceed.

## Fallback when "Next session" is empty

Don't auto-start anything. Search in order, present candidates to user:

1. `flightdeck/flight-plans/` (excluding `landed/`) — already broken down, immediately executable.
2. `flightdeck/specs/` (excluding `landed/`) — designed but no plan; ask "write plan now or execute directly?"
3. `flightdeck/sketches/` — unstarted ideas; ask which (if any) to promote.

Actively-implementing artifacts SHOULD already be in manifest's `In flight` table. If fallback finds something not on the manifest, flag as a manifest sync bug.

## Output format

Report concisely:

```
Cockpit reconciled (Last updated: 2026-05-25; Active focus: <X>; tree clean)

Routing catalog (loaded this session — know-what-exists, not read-all):

[Checklists]
| File | when_to_read | applies_to | last_updated |
|---|---|---|---|
| checklists/comments.md | before writing or editing any source-code comment | comments, code-style, documentation | 2026-05-29 |

[Incident reports]
| File | when_to_read | applies_to | last_updated |
|---|---|---|---|
| incident-reports/parser-recursion.md | before designing a recursive parser | parser, recursion | 2026-04-02 |

[Bundles]  (read the README first; leaves load via its reading_order)
| README | when_to_read | applies_to | last_updated |
|---|---|---|---|
| checklists/plugin-spec/README.md | before authoring a plugin spec | plugin, spec | 2026-05-30 |

[Malformed bundles]  (omitted when none)
| README | issue |
|---|---|
| checklists/foo/README.md | ⚠ missing bundle: true |

Next session item #1: <item description>

→ Matching task keywords against applies_to before executing. Catalog is know-what-exists only — NOT a substitute for /flightdeck:walkaround, and does not mean these files were read. Bodies load on demand when a trigger matches.

Proceeding.
```

Grouping rules: one group per kind, explicit `[...]` headers; a bundle goes under `[Bundles]` regardless of whether it lives in `checklists/` or `incident-reports/`; a subdirectory `README.md` without `bundle: true` goes under `[Malformed bundles]`, never `[Bundles]`. Omit any group with no entries. If there are no routed resources at all, print `Routing catalog: (empty — no routed resources yet)`.

Or if blocked:

```
Reconcile flagged:
- <mismatch 1>
- <mismatch 2>

Resolve which?
```

## Don't do

- Don't auto-pick a fallback when `Next session` is empty — always ask.
- Don't bump `Last updated` from this skill — entry doesn't modify cockpit.
- Don't load every incident-report/checklist **body** upfront — the routing catalog (step 5) loads only their metadata; full bodies are routed by `applies_to` when relevant tasks come up. Printing the catalog must not cascade into reading every file.
- Don't grep the codebase looking for "things to do" — cockpit.md is authoritative.

For deeper background on the flightdeck convention (folder semantics, scenario triggers during the session, write gate), see [workflow/SKILL.md](../workflow/SKILL.md).
