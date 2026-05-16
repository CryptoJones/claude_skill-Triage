# Priority Context

Shared reference for both skills in this repository — `TaskPriorityReorder`
(manual override) and the forthcoming `triage` (signal-driven recommender).
Each skill's `SKILL.md` cites this doc so the underlying concepts don't
get duplicated and drift.

---

## The append-only ID problem

Claude Code's `TaskCreate` tool assigns task IDs in creation order
and **never reuses them**. So whatever you create first naturally
becomes the lowest-numbered (often interpreted as "highest priority")
task in the queue.

That's the wrong default the moment a more urgent thing comes in:

- You can't just "renumber" a task — there's no `TaskReorder` tool.
- Updating a task's metadata doesn't change its ID, so it stays in
  its original sort position.
- Deletion + re-creation gives the recreated task a *new* (later)
  ID, which moves it to the bottom of the natural sort.

This is the constraint both skills in this repository work around.

---

## What "priority" means here

In Claude Code's task model, the queue is a list of `pending` tasks.
There isn't a numeric priority field. The ordering convention this
repository uses is:

> **Lowest task ID = highest priority. The "next task to work on" is
> the lowest-ID pending task.**

This convention is what the agent's `TaskList` output naturally
surfaces first, which is what makes "lowest ID = top" a useful
heuristic in practice. It also means changing priority order *requires*
changing IDs — which means delete + recreate.

---

## Manual / automatic split

This repo hosts two skills with complementary jobs:

### `TaskPriorityReorder` — manual override (currently live)

The operator says **"bump X to top"** (or `swap`, `promote`, `demote`,
etc.) and the skill performs the delete + recreate cycle to make the
queue reflect the new order. The trigger is an explicit human instruction.

See [`SKILL.md`](SKILL.md) at the repo root for the full procedure.

### `triage` — signal-driven recommender (planned, Triage v0.9)

The operator says **"what should I do next?"** (or "what's hot?",
"reorder by signal") and the skill wraps the
[Triage CLI](https://github.com/CryptoJones/Triage) to surface the
top-N tasks ranked by signal-driven scoring (CI failure, deadline
pressure, blocker propagation, RunPod cost, etc.) with the rule
contributions that pushed each task there. The human still confirms
before any actual reorder via `TaskPriorityReorder`.

See [`triage/SKILL.md`](triage/SKILL.md) for the placeholder + the
planned procedure that will activate when Triage v0.9 ships.

---

## The delete-then-recreate invariant

Both skills, when they *do* change the queue, follow the same pattern:

```
1. TaskList                    → snapshot the current pending tasks
2. TaskGet for each affected   → capture subject + description
                                 + activeForm + metadata (CRITICAL:
                                 capture all four before deleting any)
3. resolve the new order       → based on the operator's instruction
                                 OR the Triage-computed ranking
4. TaskUpdate status=deleted   → on each pending task being reordered
5. TaskCreate, in new order    → lowest new ID = top priority
6. show the new queue          → in BBS-aesthetic to match Triage CLI
```

Critical invariants:

- **Capture all four fields** for every task before any delete. A
  `TaskGet` failure mid-batch loses content irrecoverably.
- **In-progress tasks are never silently recreated.** Recreate erases
  the in-progress state. Either skip them, or warn loudly + require
  explicit confirmation. See the safety section in
  [`SKILL.md`](SKILL.md).
- **Completed and deleted tasks are historical record.** They stay at
  their original IDs. The skills only touch *pending* tasks.

---

## Presentation: BBS aesthetic

When either skill renders the post-reorder queue back to the operator,
the visual language matches the Triage CLI's `bbs` theme so the
operator sees a consistent presentation whether they're looking at
manual Claude Code reorders or automatic Triage recommendations:

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

Markers (`← bumped`, `← demoted`, `← swapped`, `← signal-driven`)
flag which tasks moved and why. ASCII fallback (`+--+`, `|`, `==`)
is acceptable on non-unicode surfaces.

---

## When NOT to invoke either skill

- **"What are my pending tasks?"** → just `TaskList`. No reorder.
- **"Mark task X as in-progress"** → just `TaskUpdate`. No reorder.
- **"Add a new task"** → just `TaskCreate`. No reorder.
- **A task is `in_progress` and the operator wants to demote it**
  → STOP and ask. Recreating it loses the in-progress state.

---

## Why skills (not memory rules)

Memory rules can be forgotten under context pressure. Skill triggers
fire automatically when the frontmatter description-keywords match
the operator's phrasing. That's the right shape for **multi-step
procedures where missing or reordering any step loses information
irrecoverably** — which is exactly what reorder-by-delete-and-recreate
is.

If the procedure could be done in one tool call, it'd be a memory
rule. Because it's five tool calls in a precise order with mid-stream
verification, it's a skill.

---

Proudly Made in Nebraska. Go Big Red! 🌽 https://xkcd.com/2347/
