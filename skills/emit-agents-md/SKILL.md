---
name: emit-agents-md
description: Use when explicitly invoking the workshop AGENTS.md emitter — regenerates `AGENTS.md` at repo root from `workshop/board.md` between fenced markers, preserving any hand-authored content outside the markers. Triggered by `/workshop:emit-agents-md`.
disable-model-invocation: true
---

# Workshop AGENTS.md Emitter

User-triggered regeneration of `AGENTS.md` at repo root from the current state of `workshop/board.md`. Use after `board.md` changes (e.g., at session-exit) so non-Claude AI tools (Codex CLI, Copilot, Cursor, Windsurf, Continue, Cody, etc.) reading `AGENTS.md` see fresh project state.

## Why this exists

AGENTS.md is the cross-tool standard for project-level AI instructions, stewarded by the Agentic AI Foundation under the Linux Foundation; ~60k+ repos adopted by mid-2026 with measurable 28.6% runtime / 16.6% token wins. Workshop tracks the same information (current focus, next actions, in-flight artifacts) inside `workshop/board.md`. The emitter is the bridge: workshop authors maintain ONE file (`board.md`), and AI tools that don't speak workshop natively read the auto-regenerated `AGENTS.md`.

## Run this checklist

### Step 1: Read `workshop/board.md`

Use Read on `workshop/board.md`. Extract these fields verbatim:

- The `**Active focus**:` line value.
- All numbered items under `## Next session`.
- All rows under `## In flight` whose first column is NOT `_none_` (skip the placeholder row).
- All bullet items under `## Hanging tasks` whose content is not literally `(none)`.

### Step 2: Read current `AGENTS.md` at repo root (if present)

Use Read on `AGENTS.md` at the project root.

- **File exists with workshop markers** (`<!-- BEGIN: workshop -->` and `<!-- END: workshop -->`): note the content BEFORE the BEGIN marker and AFTER the END marker — both blocks of hand-authored prose MUST be preserved verbatim.
- **File exists without workshop markers**: the entire file is hand-authored. You will add the workshop block at the top (above all existing content) and leave the rest untouched.
- **File does not exist**: you will create it with the workshop block + a footer comment inviting hand-authored additions below.

### Step 3: Construct the new workshop block

The block to insert between markers (or as the whole new file body if AGENTS.md was missing):

```
<!-- BEGIN: workshop -->
<!-- Auto-regenerated from workshop/board.md by /workshop:emit-agents-md.
     Do NOT edit between these markers — your edits will be overwritten on
     next regeneration. Add hand-authored content OUTSIDE the markers. -->

## Current focus

<Active focus value, verbatim from board.md>

## Next session

<Numbered list copied from board.md Next session. Rewrite relative markdown links so they resolve from repo root: any link target that is NOT an HTTP/HTTPS URL, NOT an anchor (`#...`), and NOT already absolute (`/...`) must be prefixed with `workshop/` — since board.md lives in `workshop/` but AGENTS.md lives at repo root. Example: `[specs/foo.md](specs/foo.md)` → `[specs/foo.md](workshop/specs/foo.md)`. Link text stays unchanged; only the path inside the parentheses is rewritten.>

## In flight

<If there are non-placeholder rows: render them as a bullet list, one per row,
 in the form: "- **<Artifact>** (state: <State>) — <Owner / Reason>. Refs: <Refs>".
 Apply the same relative-link-rewriting rule as the Next session block (prefix with `workshop/` unless URL / anchor / absolute).
 If only the _none_ placeholder is present: write the single line:
 "None — all artifacts are at implicit lifecycle states based on folder location.">

## Hanging tasks

<If board has hanging tasks: bullet list copied verbatim.
 Else: single line "None.">

## More

For full project state — playbooks, scars, critiques, plans — read `workshop/board.md` and the linked artifacts.

<!-- END: workshop -->
```

### Step 4: Write the regenerated AGENTS.md

Use Write to save.

- File missing: write `<workshop block>\n\n<!-- Hand-authored content below this line is preserved across emitter runs. -->\n`
- File had markers: pre-marker content + new workshop block + post-marker content
- File had no markers: workshop block + blank line + entire existing content

### Step 5: Verify idempotency

Re-run Steps 1-4. The output of the second run MUST be byte-identical to the first. If diff is non-empty, the extractor has nondeterminism (e.g., trailing whitespace, ordering instability). Fix and re-verify.

### Step 6: Report

Report concisely:

```
AGENTS.md regenerated.
  Active focus: <one-line>
  Next session items: <N>
  In flight rows: <N>
  Hanging tasks: <N>
  Hand-authored content preserved: <yes / no / n/a (file missing before)>
```

## Idempotency rules

- Read board.md fields in a fixed order (Active focus → Next session → In flight → Hanging tasks). Don't reorder.
- Empty sections must produce the placeholder text — never be omitted (otherwise the second run might re-omit, but the first might have included, causing diffs).
- One blank line between sections inside the workshop block. No trailing whitespace inside markers.
- **Link prefixing is deterministic**: a relative link `(path)` always becomes `(workshop/path)` — no path normalization (e.g., do NOT collapse `workshop/../README.md` to `README.md`; emit the prefixed form). Determinism beats prettiness.

## Don't do

- Don't read or modify content OUTSIDE the fenced markers in AGENTS.md.
- Don't add markers around pre-existing hand-authored content — leave it alone, add the workshop block above it.
- Don't include scar / playbook / spec details in the regenerated block — those are linked, not embedded. AGENTS.md stays terse.
- Don't run this from a non-clean working tree without warning the user — mid-edit `board.md` produces a stale snapshot.
