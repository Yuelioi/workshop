---
name: emit-agents-md
description: Use when explicitly invoking the flightdeck AGENTS.md emitter — regenerates `AGENTS.md` at repo root from `flightdeck/cockpit.md` between fenced markers, preserving any hand-authored content outside the markers. Triggered by `/flightdeck:emit-agents-md`.
disable-model-invocation: true
---

# Flightdeck AGENTS.md Emitter

User-triggered regeneration of `AGENTS.md` at repo root from the current state of `flightdeck/cockpit.md`. Use after `cockpit.md` changes (e.g., at landing) so non-Claude AI tools (Codex CLI, Copilot, Cursor, Windsurf, Continue, Cody, etc.) reading `AGENTS.md` see fresh project state.

## Why this exists

*(Background rationale — do not copy any of this section into AGENTS.md.)*

AGENTS.md is the cross-tool standard for project-level AI instructions, stewarded by the Agentic AI Foundation under the Linux Foundation; ~60k+ repos adopted by mid-2026 with measurable 28.6% runtime / 16.6% token wins. Flightdeck tracks the same information (current focus, next actions, in-flight artifacts) across `flightdeck/cockpit.md` + `manifest.md`. The emitter is the bridge: flightdeck authors maintain these files, and AI tools that don't speak flightdeck natively read the auto-regenerated `AGENTS.md`.

## Run this checklist

### Step 1: Read `flightdeck/cockpit.md` and `flightdeck/manifest.md`

Use Read on `flightdeck/cockpit.md`. Extract these fields (content copied as-is — the only transformation is the relative-link prefixing applied per-block in Step 3):

- The `**Active focus**:` line value.
- All numbered items under `## Next session`.
- All bullet items under `## Hanging tasks` whose content is not literally `(none)`.

Then use Read on `flightdeck/manifest.md`. Extract:

- All rows under `## In flight` whose first column is NOT `_none_` (skip the placeholder row).

### Step 2: Read current `AGENTS.md` at repo root (if present)

Use Read on `AGENTS.md` at the project root.

- **File exists with flightdeck markers** (`<!-- BEGIN: flightdeck -->` and `<!-- END: flightdeck -->`): note the content BEFORE the BEGIN marker and AFTER the END marker — both blocks of hand-authored prose MUST be preserved verbatim.
- **File exists without flightdeck markers**: the entire file is hand-authored. You will add the flightdeck block at the top (above all existing content) and leave the rest untouched.
- **File does not exist**: you will create it with the flightdeck block + a footer comment inviting hand-authored additions below.

### Step 3: Construct the new flightdeck block

The block to insert between markers (or as the whole new file body if AGENTS.md was missing):

```
<!-- BEGIN: flightdeck -->
<!-- Auto-regenerated from flightdeck/cockpit.md by /flightdeck:emit-agents-md.
     Do NOT edit between these markers — your edits will be overwritten on
     next regeneration. Add hand-authored content OUTSIDE the markers. -->

## Current focus

<Active focus value, verbatim from cockpit.md>

## Next session

<Numbered list copied from cockpit.md Next session. Rewrite relative markdown links so they resolve from repo root: any link target that is NOT an HTTP/HTTPS URL, NOT an anchor (`#...`), and NOT already absolute (`/...`) must be prefixed with `flightdeck/` — since cockpit.md lives in `flightdeck/` but AGENTS.md lives at repo root. This includes `./` and `../` targets (prefix as-is, no normalization). Example: `[specs/foo.md](specs/foo.md)` → `[specs/foo.md](flightdeck/specs/foo.md)`. Link text stays unchanged; only the path inside the parentheses is rewritten.>

## In flight

<If there are non-placeholder rows in manifest.md: render them as a bullet list, one per row,
 in the form: "- **<Artifact>** (state: <State>) — <Owner / Reason>. Refs: <Refs>".
 Apply the same relative-link-rewriting rule as the Next session block (prefix with `flightdeck/` unless URL / anchor / absolute).
 If only the _none_ placeholder is present: write the single line:
 "None — all artifacts are at implicit lifecycle states based on folder location.">

## Hanging tasks

<If cockpit has hanging tasks: bullet list copied verbatim.
 Else: single line "None.">

## More

For full project state — checklists, incident-reports, safety-reviews, flight-plans — read `flightdeck/cockpit.md` and the linked artifacts.

<!-- END: flightdeck -->
```

### Step 4: Write the regenerated AGENTS.md

Use Write to save.

- File missing: write `<flightdeck block>\n\n<!-- Hand-authored content below this line is preserved across emitter runs. -->\n`
- File had markers: pre-marker content + new flightdeck block + post-marker content
- File had no markers: flightdeck block + blank line + entire existing content (no footer comment — the existing content already occupies the hand-authored space)

### Step 5: Verify determinism (do NOT write again)

The transform must be deterministic — a hypothetical re-run would produce a byte-identical block. Don't actually re-run Steps 1–4 or re-write the file; instead self-check the block you just wrote against this checklist:

- `## Current focus` appears once, with the `Active focus` value present and not duplicated.
- `## Next session` numbering is contiguous (no gaps or repeats).
- Every relative link `](...)` inside the block carries the `flightdeck/` prefix — scan each one.
- No trailing whitespace on any line inside the markers; exactly one blank line between sections.

Any miss is a construction bug — fix the block directly, don't mask it by claiming a clean re-run.

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

- Read cockpit.md / manifest.md fields in a fixed order (Active focus → Next session → In flight → Hanging tasks). Don't reorder.
- Empty sections must produce the placeholder text — never be omitted (otherwise the second run might re-omit, but the first might have included, causing diffs).
- One blank line between sections inside the flightdeck block. No trailing whitespace inside markers.
- **Link prefixing is deterministic**: a relative link `(path)` always becomes `(flightdeck/path)` — no path normalization (e.g., do NOT collapse `flightdeck/../README.md` to `README.md`; emit the prefixed form). Determinism beats prettiness.

## Don't do

- Don't read or modify content OUTSIDE the fenced markers in AGENTS.md.
- Don't add markers around pre-existing hand-authored content — leave it alone, add the flightdeck block above it.
- Don't include incident-report / checklist / spec details in the regenerated block — those are linked, not embedded. AGENTS.md stays terse.
- Don't run this from a non-clean working tree without warning the user — mid-edit `cockpit.md` produces a stale snapshot.
