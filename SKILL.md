---
name: TaskPriorityReorder
description: Re-order the pending task list by priority. The TaskCreate tool issues IDs append-only, so changing priority means deleting the affected tasks and recreating them in the new order (lowest new ID = highest priority). Use when Aaron says "move task X to top", "bump task X up", "promote X", "demote X", "deprioritize X", "this is now top priority", "this is more urgent than Y", "this can wait", "swap X and Y", "reorder tasks by priority", "task X is more important than Y", or any "shuffle the queue" framing.
---

# TaskPriorityReorder

> **Shared concepts:** see [`priority-context.md`](priority-context.md)
> for the append-only-ID problem, the manual/automatic skill split,
> the delete-then-recreate invariant, and the BBS presentation
> aesthetic — all referenced by this skill and shared with the
> companion `triage` skill.

When Aaron wants the task list reordered, this skill captures every pending task's full content, deletes them all, and recreates them in the new desired order. The IDs change (lowest = highest priority), but every task's subject + description + activeForm + metadata is preserved.

## When to fire

Promotion / "bump up" framings:

- "Move task #X to top"
- "Bump task X up the queue"
- "**Promote** X"
- "X is now top priority"
- "X is more urgent than Y"
- "Make X the next thing"

Demotion / "can wait" framings:

- "**Demote** X"
- "**Deprioritize** X"
- "X can wait"
- "Push X to the bottom"
- "X is lower priority than I made it"

Swap framings:

- "Swap X and Y"
- "X and Y should trade places"
- "X is more important than Y — swap them"

Full reorder framings:

- "Reorder tasks by priority"
- "Re-prioritize the queue: I want X first, then Y, then Z"
- "Shuffle the queue"

## When NOT to fire

- "What are my pending tasks?" → just call `TaskList`.
- "Mark task X as in-progress / completed / deleted" → just `TaskUpdate`.
- "Add a new task" → just `TaskCreate`.
- "What should I do next?" (signal-driven question) → that's the future companion `triage` skill (Triage v0.7), not this one.

## ⚠️ In-progress safety

The reorder mechanism is **delete + recreate**. An `in_progress` task that is deleted and recreated comes back as `pending` — **the in-progress state is irrecoverable**. Treat this as the skill's primary failure mode.

Decision tree when Aaron's instruction touches an `in_progress` task:

```
                       ┌─────────────────────────────────┐
                       │  Does instruction implicate an  │
                       │  in_progress task?              │
                       └────────────────┬────────────────┘
                                        │
                          ┌─────────────┴─────────────┐
                          │                           │
                         no                          yes
                          │                           │
                          ▼                           ▼
                  proceed normally        Was it implicit (e.g. "reorder
                                          everything", "swap top three")
                                          or explicit ("bump the
                                          in-progress task down")?
                                                      │
                                       ┌──────────────┴──────────────┐
                                       │                             │
                                    implicit                      explicit
                                       │                             │
                                       ▼                             ▼
                              SILENTLY EXCLUDE the          STOP and ask:
                              in_progress task from         "Reordering #N means
                              the reorder. Operate          deleting and recreating
                              only on pending siblings.     it — that loses the
                              State this in the             in-progress state.
                              confirmation list.            Continue anyway?"
                                                            Proceed only on
                                                            explicit yes.
```

Never silently recreate an `in_progress` task. The user must consent to losing the state.

## The procedure

### 1. Snapshot

```
TaskList  # see what's currently pending + the current order
```

For each pending task that will be touched, call `TaskGet` with its ID to capture the full body:

- `subject` — exact string
- `description` — full text, including markdown / line breaks
- `activeForm` — the present-continuous variant
- `metadata` — if any (Aaron sometimes uses this for priority tags or links)

**Critical:** capture all four for every task you plan to delete. The summary line in `TaskList` alone isn't enough — descriptions can be many paragraphs of code, links, or context.

### 2. Resolve the new order

Take Aaron's instruction + the current order and compute the new sequence. Pattern catalog:

| Instruction shape                                     | New order                                                           |
|-------------------------------------------------------|---------------------------------------------------------------------|
| "Move X to top"                                       | X first; everyone else keeps current relative order.                |
| "Bump X up" (no destination)                          | X moves up exactly one slot.                                        |
| "Promote X" / "X is more urgent than Y"               | X moves to immediately above Y. (If no Y, X goes to top.)           |
| "Demote X" / "X can wait" / "Deprioritize X"          | X moves to bottom of pending. (Or "below Y" if Y named.)            |
| "Swap X and Y"                                        | X and Y trade positions; everyone else unchanged.                   |
| "Top priority is A, B, C"                             | A, B, C first in that order; others follow current relative order. |
| "Reorder everything: X, Y, Z, …"                      | Use Aaron's full list verbatim.                                     |
| "This new thing is most important" + new task added   | `TaskCreate` the new task FIRST (lowest new ID), then reorder rest. |

If Aaron's instruction is ambiguous, ask **one** focused clarifying question — don't guess at the order.

### 3. Decide what stays as-is

- **In-progress tasks**: see the safety section above.
- **Completed / deleted tasks**: never recreated. They stay at their original IDs in the historical record.
- **Tasks not implicated in the reorder**: leave alone (don't churn their IDs unnecessarily — every recreate burns a new ID).

### 4. Delete + recreate

For each pending task being reordered:

```
TaskUpdate  taskId: <id>  status: deleted
```

Then in the new priority order:

```
TaskCreate
  subject:     <captured>
  description: <captured>
  activeForm:  <captured>
  metadata:    <captured>
```

The new tasks get new IDs in `TaskCreate` order — **lowest new ID = highest priority**.

### 5. Confirm

After the cycle, show Aaron the new task list with IDs + subjects in priority order, using the BBS aesthetic to match the companion [Triage](https://github.com/CryptoJones/Triage) CLI (see "Presentation guidance" below).

## Presentation guidance

Render the post-reorder confirmation as a fenced code block, BBS-style, so the visual language matches Triage's `triage list` output:

```
╔═══════════════════════════════════════════════════════════╗
║              T A S K   Q U E U E   ( a f t e r )          ║
╚═══════════════════════════════════════════════════════════╝
  ID    STATUS          SUBJECT
  ════  ════════════    ══════════════════════════════════════
  #18   [in_progress]   Plan correcthorsebatterystaple CLI
  #22   [pending]       Add $10 to donation row for dave   ← bumped
  #23   [pending]       Plan for new AWS Helper Program
  #24   [pending]       Audit https://github.com/ipython/xkcd-font
```

Guidance:

- Use double-line box drawing (`╔═╗║╚╝`) for the banner.
- Use double-equals (`══`) for column rules.
- Mark the task(s) Aaron explicitly changed with `← bumped`, `← demoted`, `← swapped`, etc.
- Keep the table to one screen — truncate long subjects with `…` if needed.
- If you're emitting to a markdown surface (Claude's reply), use a fenced code block so the box-drawing aligns.

If the terminal/surface doesn't support unicode box drawing, fall back to ASCII (`+--+`, `|`, `==`).

## Worked example — "move to top"

Aaron's pending tasks before:

```
#18 [in_progress] [TOP PRIORITY] Plan correcthorsebatterystaple CLI
#19 [pending]     Add $10 to donation row for dave
#20 [pending]     Plan for new AWS Helper Program
#21 [pending]     Audit https://github.com/ipython/xkcd-font
```

Aaron: *"the donation row is what matters tonight — bump it to top"*

Skill runs:

1. `TaskList` sees #18 in_progress (leave alone), #19 + #20 + #21 pending.
2. `TaskGet` each pending one, capture content.
3. Compute new order: Aaron wants #19 first → `[#19, #20, #21]`.
4. Delete #19, #20, #21.
5. Re-create in priority order: donation (new #22), AWS Helper (new #23), xkcd-font (new #24).
6. Confirm:

```
╔═══════════════════════════════════════════════════════════╗
║              T A S K   Q U E U E   ( a f t e r )          ║
╚═══════════════════════════════════════════════════════════╝
  ID    STATUS          SUBJECT
  ════  ════════════    ══════════════════════════════════════
  #18   [in_progress]   Plan correcthorsebatterystaple CLI
  #22   [pending]       Add $10 to donation row for dave   ← bumped to top of pending
  #23   [pending]       Plan for new AWS Helper Program
  #24   [pending]       Audit https://github.com/ipython/xkcd-font
```

## Worked example — "swap X and Y"

Aaron's pending tasks before:

```
#30 [pending] Refactor the auth middleware
#31 [pending] Write the migration plan
#32 [pending] Fix the flaky e2e test
#33 [pending] Update the dependency lockfile
```

Aaron: *"swap the auth refactor and the flaky test — flaky test first"*

Skill runs:

1. `TaskList` shows the four pending; none `in_progress`.
2. `TaskGet` all four (every reorder needs the surrounding context captured so nothing is dropped).
3. Compute new order:
   - Original (top-to-bottom): #30, #31, #32, #33.
   - Aaron wants #32 in #30's position. Swap.
   - New order: `[#32, #31, #30, #33]`.
4. Delete #30, #31, #32, #33 in any order.
5. Re-create in the new priority order: flaky test (new #34), migration plan (new #35), auth refactor (new #36), lockfile (new #37).
6. Confirm:

```
╔═══════════════════════════════════════════════════════════╗
║              T A S K   Q U E U E   ( a f t e r )          ║
╚═══════════════════════════════════════════════════════════╝
  ID    STATUS      SUBJECT
  ════  ══════════  ══════════════════════════════════════════
  #34   [pending]   Fix the flaky e2e test          ← swapped up
  #35   [pending]   Write the migration plan
  #36   [pending]   Refactor the auth middleware    ← swapped down
  #37   [pending]   Update the dependency lockfile
```

Note that **everyone got new IDs**, not just the two swapped — because reorder-by-recreate always burns IDs for every task touched. Tasks that didn't move (#31 → #35, #33 → #37) still got new IDs because they were part of the recreate batch. That's expected.

## Worked example — "demote X"

Aaron's pending tasks before:

```
#40 [pending] Ship the Q3 marketing site
#41 [pending] Investigate the slow query
#42 [pending] Reply to legal about the contract
#43 [pending] Tidy up the README
```

Aaron: *"demote the marketing site — legal is the only thing that has a deadline"*

Skill runs:

1. `TaskList` shows all four pending.
2. `TaskGet` all four to be safe.
3. New order: legal first (Aaron's emphasis), then keep relative order of others, with marketing site at bottom: `[#42, #41, #43, #40]`.
4. Delete + recreate.
5. Confirm:

```
╔═══════════════════════════════════════════════════════════╗
║              T A S K   Q U E U E   ( a f t e r )          ║
╚═══════════════════════════════════════════════════════════╝
  ID    STATUS      SUBJECT
  ════  ══════════  ══════════════════════════════════════════
  #44   [pending]   Reply to legal about the contract  ← promoted (deadline)
  #45   [pending]   Investigate the slow query
  #46   [pending]   Tidy up the README
  #47   [pending]   Ship the Q3 marketing site         ← demoted
```

## Things to NOT do

- **Don't recreate `in_progress` tasks** — see the safety section.
- **Don't drop content.** A task may have a 50-line description with code blocks; capture all 50 lines. Skim every capture before `TaskCreate` to confirm intact.
- **Don't reorder completed tasks** — they're historical record. Leave them at original IDs.
- **Don't batch-delete then batch-recreate without verifying captures first.** A `TaskGet` failure mid-delete loses content irrecoverably. **Capture all → verify → THEN delete → THEN recreate.**
- **Don't skip the confirmation render.** Aaron needs to see the queue post-reorder; the skill is only "done" when he can verify the change.

## Why this is a skill, not just a memory rule

Multi-step procedure with a precise sequence (capture → verify → delete → recreate → confirm) where missing or reordering any step loses content. Memory rules can be forgotten under context pressure; skill triggers fire automatically when the description-keywords match. This is exactly the failure-mode shape (irrecoverable state loss on a routine operation) that skills are built for.

The forthcoming companion skill `triage` (planned for Triage v0.7) handles the **automatic** side: "what should I do next?" wraps the [Triage](https://github.com/CryptoJones/Triage) CLI and surfaces a signal-driven recommendation. `TaskPriorityReorder` remains the **manual** override.
