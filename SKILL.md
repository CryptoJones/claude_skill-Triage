---
name: TaskPriorityReorder
description: Re-order the pending task list by priority. The TaskCreate tool issues IDs append-only, so changing priority means deleting the affected tasks and recreating them in the new order (lowest new ID = highest priority). Use when Aaron says "move task X to top", "bump task X up", "this is now top priority", "reorder tasks by priority", "task X is more important than Y", or any "shuffle the queue" framing.
---

# TaskPriorityReorder

When Aaron wants the task list reordered, this skill captures every pending task's full content, deletes them all, and recreates them in the new desired order. The IDs change (lowest = highest priority), but every task's subject + description + activeForm + metadata is preserved.

## When to fire

- "Move task #X to top"
- "Bump task X up the queue"
- "Reorder tasks by priority"
- "Task X is now top priority"
- "Task X is more important than Y — swap them"
- "Re-prioritize the queue: I want X first, then Y, then Z"

Don't fire on:

- "What are my pending tasks?" → just call TaskList
- "Mark task X as in-progress / completed / deleted" → just TaskUpdate
- "Add a new task" → just TaskCreate
- A task is already **in_progress** and Aaron wants it bumped down → STOP and ask first. Reorder-by-delete-and-recreate loses the in-progress state. Either complete it first, or accept the recreated task starts fresh as pending.

## The procedure

### 1. Snapshot

```
TaskList  # see what's currently pending + the order
```

For each pending task, call `TaskGet` with its ID to capture the full body:
- `subject` (exact string)
- `description` (full text, including any markdown / line breaks)
- `activeForm` (the present-continuous variant)
- `metadata` (if any — Aaron sometimes uses this for priority tags or links)

**Critical:** capture all four. The summary in TaskList alone isn't enough — descriptions can be many paragraphs.

### 2. Resolve the new order

Take Aaron's instruction + the current order and compute the new sequence. Patterns:

- **"Move #X to top"**: X first, everyone else in their current relative order.
- **"Swap X and Y"**: X and Y trade positions, everyone else unchanged.
- **"Top priority is now A, B, C"**: A, B, C first in that order, others follow in their current relative order.
- **"This new thing is most important"** (with a new task being added): create the new task FIRST so it gets the lowest ID, then reorder the rest if needed.

If Aaron's instruction is ambiguous, ask one focused clarifying question — don't guess at the order.

### 3. Decide what stays as-is

- **In-progress tasks**: ideally leave them alone. If Aaron explicitly says to reorder them too, warn that the in-progress state is lost on recreate, then proceed only if he confirms.
- **Completed / deleted tasks**: never recreated. They stay in the historical record at their original IDs.

### 4. Delete + recreate

For each pending task being reordered:

```
TaskUpdate  taskId: <id>  status: deleted
```

Then in the new priority order:

```
TaskCreate  subject: <captured>  description: <captured>  activeForm: <captured>  metadata: <captured>
```

The new tasks get new IDs in TaskCreate order — lowest = highest priority.

### 5. Confirm

After the cycle, show Aaron the new task list with IDs + subjects in priority order. He should see the change reflect what he asked for.

## Worked example

Aaron's pending tasks before:

```
#18 [in_progress] [TOP PRIORITY] Plan correcthorsebatterystaple CLI
#19 [pending]     Add $10 to donation row for dave
#20 [pending]     Plan for new AWS Helper Program
#21 [pending]     Audit https://github.com/ipython/xkcd-font
```

Aaron: "the donation row is what matters tonight — bump it to top"

Skill runs:
1. TaskList sees #18 in_progress (leave alone), #19 + #20 + #21 pending
2. TaskGet each pending one, capture content
3. Compute new order: [#19's content, #20's content, #21's content]  →  Aaron wants #19 first, so: [#19, #20, #21]
4. Delete #19, #20, #21 (in any order)
5. Re-create in priority order: $10 donation first (becomes new #22), AWS Helper (new #23), xkcd-font audit (new #24)
6. Show new list:
   ```
   #18 [in_progress] [TOP PRIORITY] Plan correcthorsebatterystaple CLI
   #22 [pending]     Add $10 to donation row for dave   ← bumped
   #23 [pending]     Plan for new AWS Helper Program
   #24 [pending]     Audit https://github.com/ipython/xkcd-font
   ```

## Things to NOT do

- **Don't recreate in-progress tasks.** Their state can't be recreated; doing so loses progress.
- **Don't drop content.** If a task has a 50-line description with code blocks, you must capture all 50 lines. Skim the captures before TaskCreate to confirm they're intact.
- **Don't reorder completed tasks** — they're historical record. Leave them at their original IDs.
- **Don't batch-delete then batch-recreate without verifying the captures first.** A `TaskGet` failure mid-delete would lose content irrecoverably. Capture all → verify → THEN delete → THEN recreate.

## Why this is a skill, not just a memory rule

Multi-step procedure with a precise sequence (capture → verify → delete → recreate → confirm) where missing or reordering any step loses content. Memory rules can be forgotten under context pressure; skill triggers fire automatically when the description-keywords match. This is exactly the failure-mode shape (irrecoverable state loss on a routine operation) that skills are built for.
