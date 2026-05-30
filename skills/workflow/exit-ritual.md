# Exit ritual

The protocol for closing an AI coding session cleanly so the next preflight can pick up without context loss.

## Core principle

**90% of session-end decisions are obvious.** Classify directly. Only **true ambiguity** triggers brainstorming. Default-brainstorm is high-friction → skipped → knowledge lost.

## Decision tree

```
Session is wrapping up
↓
Step 1: Are there pending hanging tasks?
        (incomplete safety-review disposition / kneeboard files older than this session)
├─ yes → resolve them first, then continue
└─ no  → proceed to step 2

Step 2: Did this session produce new knowledge / discover a bug / agree on a decision?
├─ no  → only update cockpit.md if Active focus shifted, then commit
└─ yes → for each piece of new knowledge:

         Apply classification heuristics in order, first match wins:

         (a) Bug + root cause → incident-reports/
             (use incident-report template; check if existing topic — append [Case N])

         (b) "Every time we do X, follow these steps" → checklists/
             (promote only on the second occurrence, not the first)

         (c) Design decision worth referencing later → specs/
             (if substantial, brainstorm with user first)

         (d) Multi-step task to execute later → flight-plans/

         (e) External feedback received → safety-reviews/
             (must include disposition section before landing)

         (f) Long-term idea worth remembering → sketches/

         (g) One-off log analysis, debug output, conversation byproduct
             → DO NOT WRITE (gating)

         (h) Spans multiple folders, no clear primary
             → brainstorm with user

Step 3: Update cockpit.md
        - Refresh Last updated (date + author + one-line state)
        - Refresh Active focus if main thread shifted
        - Refresh Next session: 1-5 concrete items
        - **Update manifest.md In flight row states** (see lifecycle table below)
        - Clear or carry over hanging tasks

Step 3a: Apply lifecycle transitions (semi-implicit — location is source of truth, frontmatter overrides for special states)

         | Event during session              | Action                                                                                          |
         | --------------------------------- | ----------------------------------------------------------------------------------------------- |
         | New spec written                  | Drop in `specs/`. No manifest row (implicit ⚪).                                               |
         | New flight-plan written           | Drop in `flight-plans/`. No manifest row (implicit 🟡).                                        |
         | Impl done, review pending         | Add `state: awaiting-review` to flight-plan frontmatter + add 🔵 row in manifest `In flight` with review owner; if owner = user, add Hanging task. |
         | Review passed                     | Remove `state:` frontmatter; move spec + flight-plan to `landed/`; delete the 🔵 row; add 1 entry to logbook `Recently finished` (≤ 3 lines). |
         | Blocked                           | Add `state: blocked` frontmatter + 🔴 row in manifest `In flight` with reason.                 |
         | Scrapped                          | Add `state: scrapped` frontmatter (or delete file); delete row; optional 1-line note in logbook `Recently finished`. |

         Manifest's `In flight` shows ONLY rows where state diverges from location. A 20-task project with 18 flight-plans in `flight-plans/` does not need 18 rows — they're all implicit 🟡.

         Full state machine + review owner discussion: see SKILL.md `Lifecycle of specs and flight-plans`.

Step 4: Commit
        - Use checklists/commits.md if it exists
        - Otherwise: terse imperative subject + reasoning in body
```

### Step 5 — Check incident-report→checklist promotion gate (wrap-up)

For each incident report touched (newly written or updated with `[Case N]` append) this session, evaluate the 3-criterion gate from [SKILL.md § Incident report promotion gates](SKILL.md#incident-report-promotion-gates):

1. `[Case N] count ≥ 3`?
2. Recurred across ≥ 2 distinct sessions?
3. Remediation pattern stable across cases?

If ALL three hold: prompt the user "Promote `incident-reports/<topic>.md` to `checklists/<topic>.md`?". On user confirmation, move the file. On user defer or reject, leave alone — the gate will fire again next time. **No automatic promotion** — the gate guards against false positives; the user always decides.

If any criterion fails: skip silently. Don't promote-prompt for marginal cases; the user will see the incident report again next time anyway via [proactive incident report resurfacing](SKILL.md#during--scenario-triggers).

## Classification heuristics

These are first-match-wins rules — apply in order. The first one that matches the new piece of knowledge wins; do not over-think.

### (a) Bug + root cause → `incident-reports/`

**Trigger phrase**: "I assumed X but actually Y" / "this kept failing because"

Always goes to `incident-reports/`. Check existing topics first — if related, append `## [Case N]` to existing file. Do not start a new file for the same recurrence.

### (b) Repeated procedure → `checklists/`

**Trigger phrase**: "every time we do X, the steps are" / "the way to do X correctly is"

Goes to `checklists/` **only on the second occurrence**. First time is ad-hoc work; second time is a pattern worth recording.

### (c) Design decision → `specs/`

**Trigger phrase**: "we should architecturally do X" / "the design here is"

Substantial decisions go to `specs/`. If the reasoning is complex enough to matter later, brainstorm with the user first — don't free-write.

### (d) Multi-step task → `flight-plans/`

**Trigger phrase**: "let me break this into steps" / "the plan is"

Produce a structured flight-plan (hand-written or via a planning skill) — don't free-write into `flight-plans/`.

### (e) External feedback → `safety-reviews/`

**Trigger phrase**: user pastes review text from another AI / colleague

Goes to `safety-reviews/`. **Must include disposition section before landing** (see [templates.md#safety-review](templates.md#safety-review)).

### (f) Long-term idea → `sketches/`

**Trigger phrase**: "we could maybe one day" / "wouldn't it be cool if"

Goes to `sketches/`. Mark `Revisit when` condition if you can.

### (g) One-off → DO NOT WRITE

**Trigger phrase**: "the log output today was" / "I tried these 5 things and got"

Do not write. Gate strictly. Flightdeck is not a session log.

### (h) Ambiguous → brainstorm

**Trigger phrase**: "this is sort of an incident report but also a checklist"

If genuinely ambiguous, brainstorm with the user. Use the AI-asks-user template below.

## AI-asks-user template (ambiguous classification)

When triggering (h), open the conversation with a structured ask:

```
I learned something from this session that I want to record, but I'm
not sure where it belongs. Here's the content:

> <one-paragraph summary of the knowledge>

Candidates:
- incident-reports/  if the takeaway is "next time, avoid X because Y"
- checklists/  if the takeaway is "the steps to do X are"
- specs/  if the takeaway is "the design decision is X"

My weak preference: <one candidate>, because <one reason>.

Which do you want? Or skip the write?
```

This forces a structured decision in seconds. The "skip the write" option is critical — defaulting to write is the failure mode.

## Hanging tasks — block session exit

A session **cannot be closed cleanly** while either of these is unresolved:

### Hanging safety-review disposition

If `safety-reviews/<file>` exists without a complete `Disposition` section, either:
- Complete the disposition now, or
- Add to `cockpit.md` "Hanging tasks": `- [ ] Finish disposition of [safety-reviews/<file>](safety-reviews/<file>)`

The hanging task must be reflected in `cockpit.md`. The next preflight will see it on entry and resolve.

### Stale `kneeboard/` files

Detection: any `kneeboard/*.md` whose `last_touched:` frontmatter predates the current session's start. (Files without `last_touched:` count as stale by definition — see [templates.md § kneeboard](templates.md#kneeboard).)

**Landing BLOCKS until every stale kneeboard file is resolved.** No "I'll get to it next session" path.

For each stale kneeboard file, the user must choose one of:

1. **Classify** into another folder via the (a)–(h) heuristics above (most common: incident-reports/ or checklists/ if it crystallized into a lesson; sketches/ if it's a deferrable idea).
2. **Delete**. The default. Cost of deleting a useful note is much smaller than the cost of `kneeboard/` turning into a junk drawer.
3. **Defer explicitly** by adding `defer_reason:` frontmatter with a 1-line justification + `last_touched:` set to today. The defer surfaces in `preflight` again next time, with the reason visible.

There is no fourth option. kneeboard files survive one session by definition; carrying them requires explicit justification.

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
- Typo fixes in source or cockpit itself
- Internal helper refactor with no user-perceivable surface change
- Adding a single commit that doesn't complete a cockpit task
- Running tests / build that already pass

If you touched `Last updated` after only one of those, undo it. Stale-by-date is a weaker problem than signal pollution.

**Length check before exit**: if cockpit.md > 80 lines, trim immediately. If logbook.md `Recently finished` is accumulating long per-entry summaries, cap to 5 entries with ≤ 3-line summaries each. Anything older or longer → git log / archived flight-plan file.

Avoid:
- Logging session activity into "Recently finished" (gating: only user-perceivable progress)
- Leaving stale items in `Next session` after they've been done
- Writing >3-line summaries in `Recently finished` (the commit message / archived plan is the right place)

## See also

[`SKILL.md` § Common mistakes](SKILL.md#common-mistakes--stop-and-reclassify) consolidates the per-symptom red flags and rationalizations to avoid.
