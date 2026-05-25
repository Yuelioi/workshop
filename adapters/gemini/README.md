# Adapter: Gemini CLI

**Status**: ⚠️ manifest in place, behaviorally untested

## What's in place

- [`gemini-extension.json`](../../gemini-extension.json) — Gemini CLI extension manifest pointing at `GEMINI.md` as context file.
- [`GEMINI.md`](../../GEMINI.md) — `@`-includes all four skill files (SKILL.md, folder-semantics.md, templates.md, exit-ritual.md).

## Install

```bash
gemini extensions install https://github.com/Yuelioi/workshop
```

Update later:

```bash
gemini extensions update workshop
```

## What "untested" means

Gemini CLI's extension mechanism loads `GEMINI.md` as project / session context. Our `GEMINI.md` uses the `@` include syntax to pull in the four skill files. This means Gemini sees all the protocol content directly, not as a discoverable "skill" with a trigger condition. What has **not** been verified:

- That Gemini honors the `@` include syntax and resolves all four files.
- That auto-triggering on `workshop/` works (Gemini may not have skill-trigger semantics — it may simply load the protocol every session).

## Likely Gemini-specific concerns

- **No conditional loading**: unlike Claude's skill triggers, Gemini may load the protocol unconditionally. This is fine for projects that have `workshop/` but adds noise for projects that don't.
- **Token cost**: GEMINI.md @-includes pull in ~2000+ words every session.

## How to verify (and flip the matrix to ✅ tested)

1. Install on Gemini CLI per the command above.
2. Open a project with `workshop/board.md` populated.
3. Start a fresh session, ask "What were we doing?" — confirm the AI reads `board.md` first.
4. Try one routing scenario from the README routing table.
5. Open a PR that:
   - Updates the README compatibility matrix `⚠️ untested` → `✅ tested`.
   - Pastes the verification transcript here.
   - Notes whether a conditional-load workaround is needed.
