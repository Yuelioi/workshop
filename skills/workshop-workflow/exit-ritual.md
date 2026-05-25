# Exit ritual

The protocol for closing an AI coding session cleanly so the next session can pick up without context loss.

## Core principle

**90% of session-end decisions are obvious.** Classify directly. Only **true ambiguity** triggers brainstorming. Default-brainstorm is high-friction → skipped → knowledge lost.

## Decision tree

```
Session is wrapping up
↓
Step 1: Are there pending hanging tasks?
        (incomplete critique disposition / wip files older than this session)
├─ yes → resolve them first, then continue
└─ no  → proceed to step 2

Step 2: Did this session produce new knowledge / discover a bug / agree on a decision?
├─ no  → only update board.md if Active focus shifted, then commit
└─ yes → for each piece of new knowledge:

         Apply classification heuristics in order, first match wins:

         (a) Bug + root cause → scars/
             (use scar template; check if existing scar topic — append [Case N])

         (b) "Every time we do X, follow these steps" → playbooks/
             (promote only on the second occurrence, not the first)

         (c) Design decision worth referencing later → specs/
             (if substantial, brainstorm with user first)

         (d) Multi-step task to execute later → plans/

         (e) External feedback received → critiques/
             (must include disposition section before exit)

         (f) Long-term idea worth remembering → sketches/

         (g) One-off log analysis, debug output, conversation byproduct
             → DO NOT WRITE (gating)

         (h) Spans multiple folders, no clear primary
             → brainstorm with user

Step 3: Update board.md
        - Refresh Last updated (date + author + one-line state)
        - Refresh Active focus if main thread shifted
        - Refresh Next session: 1-5 concrete items
        - **Update In flight row states** (see lifecycle table below)
        - Clear or carry over hanging tasks

Step 3a: Apply lifecycle transitions (semi-implicit — location is source of truth, frontmatter overrides for special states)

         | Event during session              | Action                                                                                          |
         | --------------------------------- | ----------------------------------------------------------------------------------------------- |
         | New spec written                  | Drop in `specs/`. No board row (implicit ⚪).                                                  |
         | New plan written                  | Drop in `plans/`. No board row (implicit 🟡).                                                  |
         | Impl done, review pending         | Add `state: awaiting-review` to plan frontmatter + add 🔵 row in `In flight` with review owner; if owner = user, add Hanging task. |
         | Review passed                     | Remove `state:` frontmatter; move spec + plan to `finish/`; delete the 🔵 row; add 1 entry to `Recently finished` (≤ 3 lines). |
         | Blocked                           | Add `state: blocked` frontmatter + 🔴 row in `In flight` with reason.                          |
         | Scrapped                          | Add `state: scrapped` frontmatter (or delete file); delete row; optional 1-line note in `Recently finished`. |

         Board's `In flight` shows ONLY rows where state diverges from location. A 20-task project with 18 plans in `plans/` does not need 18 rows — they're all implicit 🟡.

         Full state machine + review owner discussion: see SKILL.md `Lifecycle of specs and plans`.

Step 4: Commit
        - Use playbooks/commits.md if it exists
        - Otherwise: terse imperative subject + reasoning in body
```

### Step 5 — Check scar→playbook promotion gate (wrap-up)

For each scar touched (newly written or updated with `[Case N]` append) this session, evaluate the 3-criterion gate from [SKILL.md § Scar promotion gates](SKILL.md#scar-promotion-gates):

1. `[Case N] count ≥ 3`?
2. Recurred across ≥ 2 distinct sessions?
3. Remediation pattern stable across cases?

If ALL three hold: prompt the user "Promote `scars/<topic>.md` to `playbooks/<topic>.md`?". On user confirmation, move the file. On user defer or reject, leave alone — the gate will fire again next time. **No automatic promotion** — the gate guards against false positives; the user always decides.

If any criterion fails: skip silently. Don't promote-prompt for marginal cases; the user will see the scar again next time anyway via [proactive scar resurfacing](SKILL.md#during--scenario-triggers).

## Classification heuristics

These are first-match-wins rules — apply in order. The first one that matches the new piece of knowledge wins; do not over-think.

### (a) Bug + root cause → `scars/`

**Trigger phrase**: "I assumed X but actually Y" / "this kept failing because"

Always goes to `scars/`. Check existing scar topics first — if related, append `## [Case N]` to existing file. Do not start a new file for the same recurrence.

### (b) Repeated procedure → `playbooks/`

**Trigger phrase**: "every time we do X, the steps are" / "the way to do X correctly is"

Goes to `playbooks/` **only on the second occurrence**. First time is ad-hoc work; second time is a pattern worth recording.

### (c) Design decision → `specs/`

**Trigger phrase**: "we should architecturally do X" / "the design here is"

Substantial decisions go to `specs/`. If the reasoning is complex enough to matter later, brainstorm with the user first — don't free-write.

### (d) Multi-step task → `plans/`

**Trigger phrase**: "let me break this into steps" / "the plan is"

Produce a structured plan (hand-written or via a planning skill) — don't free-write into `plans/`.

### (e) External feedback → `critiques/`

**Trigger phrase**: user pastes review text from another AI / colleague

Goes to `critiques/`. **Must include disposition section before session exit** (see [templates.md#critique](templates.md#critique)).

### (f) Long-term idea → `sketches/`

**Trigger phrase**: "we could maybe one day" / "wouldn't it be cool if"

Goes to `sketches/`. Mark `Revisit when` condition if you can.

### (g) One-off → DO NOT WRITE

**Trigger phrase**: "the log output today was" / "I tried these 5 things and got"

Do not write. Gate strictly. Workshop is not a session log.

### (h) Ambiguous → brainstorm

**Trigger phrase**: "this is sort of a scar but also a playbook"

If genuinely ambiguous, brainstorm with the user. Use the AI-asks-user template below.

## AI-asks-user template (ambiguous classification)

When triggering (h), open the conversation with a structured ask:

```
I learned something from this session that I want to record, but I'm
not sure where it belongs. Here's the content:

> <one-paragraph summary of the knowledge>

Candidates:
- scars/  if the takeaway is "next time, avoid X because Y"
- playbooks/  if the takeaway is "the steps to do X are"
- specs/  if the takeaway is "the design decision is X"

My weak preference: <one candidate>, because <one reason>.

Which do you want? Or skip the write?
```

This forces a structured decision in seconds. The "skip the write" option is critical — defaulting to write is the failure mode.

## Hanging tasks — block session exit

A session **cannot be closed cleanly** while either of these is unresolved:

### Hanging critique disposition

If `critiques/<file>` exists without a complete `Disposition` section, either:
- Complete the disposition now, or
- Add to `board.md` "Hanging tasks": `- [ ] Finish disposition of [critiques/<file>](critiques/<file>)`

The hanging task must be reflected in `board.md`. The next session will see it on entry and resolve.

### Stale `wip/` files

Detection: any `wip/*.md` whose `last_touched:` frontmatter predates the current session's start. (Files without `last_touched:` count as stale by definition — see [templates.md § wip](templates.md#wip).)

**Session-exit BLOCKS until every stale wip is resolved.** No "I'll get to it next session" path.

For each stale wip, the user must choose one of:

1. **Classify** into another folder via the (a)–(h) heuristics above (most common: scars/ or playbooks/ if it crystallized into a lesson; sketches/ if it's a deferrable idea).
2. **Delete**. The default. Cost of deleting a useful note is much smaller than the cost of `wip/` turning into a junk drawer.
3. **Defer explicitly** by adding `defer_reason:` frontmatter with a 1-line justification + `last_touched:` set to today. The defer surfaces in `session-enter` again next time, with the reason visible.

There is no fourth option. wip survives one session by definition; carrying it requires explicit justification.

## Board update — what changes

```
Last updated:     ONLY in these 4 cases (otherwise leave alone):
                  (a) Next session content changes
                  (b) Active focus shifts (main thread moved)
                  (c) Recently finished gets a new entry
                  (d) A major task / phase completes (user-perceivable progress)
Active focus:     update if main thread shifted (otherwise leave)
Next session:     always update — at minimum confirm the first item is still right
In flight:        update if items started or completed (state column or row content)
Blockers:         update if blockers resolved or new ones appeared
Recently finished: add new entry if a major thing shipped this session.
                   AUTO-TRIM (not optional, not author-discretion): after
                   adding, count entries; if > 5, drop oldest until count = 5.
                   exit-ritual MUST enforce.
Hanging tasks:    update — add new hangings, clear resolved ones
```

**`Last updated` is not a session-activity log.** Common false triggers that should NOT bump it:
- Pure exploration / grep / reading code
- Typo fixes in source or board itself
- Internal helper refactor with no user-perceivable surface change
- Adding a single commit that doesn't complete a board task
- Running tests / build that already pass

If you touched `Last updated` after only one of those, undo it. Stale-by-date is a weaker problem than signal pollution.

**Length check before exit**: if board > 300 lines, trim now. Most common cause: `Recently finished` accumulating long per-entry summaries. Cap to 5 entries with ≤ 3-line summaries each. Anything older or longer → git log / archived plan file.

Avoid:
- Logging session activity into "Recently finished" (gating: only user-perceivable progress)
- Leaving stale items in `Next session` after they've been done
- Writing >3-line summaries in `Recently finished` (the commit message / archived plan is the right place)

## See also

[`SKILL.md` § Common mistakes](SKILL.md#common-mistakes--stop-and-reclassify) consolidates the per-symptom red flags and rationalizations to avoid.
