# Board — [project name]

**Last updated**: YYYY-MM-DD by [who] (one-line state)
**Active focus**: [current main thread]

## Next session

1. [first concrete action — executable just by reading this]
2. [optional second]
3. [optional third]

## In flight (only artifacts whose state diverges from folder location)

<!-- Most artifacts don't need a row. They're implicit: specs/ = pending, plans/ = in progress,
     */finish/ = done. A row appears HERE only when frontmatter state: diverges from location. -->

| Artifact | State | Owner / Reason | Refs |
| --- | --- | --- | --- |
| _none_ | | | |

**Status legend**: 🔵 awaiting review · 🔴 blocked · 🗑️ scrapped
(⚪ pending / 🟡 in progress / ✅ done are implicit from location.)

## Blockers

- [items waiting on external decision / answer]

## Deferred

- [items intentionally postponed; link to original source]

## Recently finished (cap 5, FIFO)

- [newest entry; ≤ 3 lines; link to commit / PR]

## Hanging tasks

- (none)

---

**Board hygiene** (skill: workshop-workflow):
- 300 lines hard ceiling. Aim for < 200.
- `In flight` only lists artifacts with explicit `state:` frontmatter (the divergent ones). Implicit ⚪/🟡/✅ state is inferred from folder location.
- `Recently finished` cap 5 entries FIFO. Per-entry summary ≤ 3 lines. Longer content → link to commit / archived plan.
- `Last updated` bumps ONLY when: Next session changes / Active focus shifts / Recently finished gains an entry / major task completes. Not on typo fixes, grep, or routine commits.
