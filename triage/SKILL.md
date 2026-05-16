# triage *(placeholder — not yet a live skill)*

> **Shared concepts:** see [`../priority-context.md`](../priority-context.md)
> for the append-only-ID problem, the manual/automatic skill split,
> the delete-then-recreate invariant, and the BBS presentation
> aesthetic — referenced by both this skill (when it goes live) and
> the existing `TaskPriorityReorder` companion.

This file is intentionally **without frontmatter** so Claude Code will
not auto-load it as a skill yet. The directory exists only to reserve
the slot for the `triage` skill that ships with [Triage](https://github.com/CryptoJones/Triage)
v0.9.

## When this becomes live

Once Triage v0.9 ships, this file will gain a frontmatter block like:

```yaml
---
name: triage
description: Surface the current Triage priority queue with rule contributions. Use when Aaron asks "what should I do next?", "what's hot right now?", "reorder by signal", or "show me the triage queue".
---
```

…and the body below will become the live procedure.

---

## Planned procedure (forward-declared)

The skill will wrap the `triage` CLI from the Triage meta-scheduler
and surface a signal-driven recommendation. The human still confirms;
the skill never auto-executes a reorder.

Sketch:

1. Run `triage tick --theme bbs` to refresh signal-driven priorities.
2. Run `triage list --json` to capture the ranked queue.
3. For the top N (default 3) tasks, run `triage why <id>` to capture
   each task's rule contributions.
4. Render the result back to the operator as a BBS-styled markdown
   table (matching the `TaskPriorityReorder` confirmation aesthetic),
   showing **task subject + priority + dominant rule**.
5. Offer an optional `TaskPriorityReorder` follow-up: *"Want me to
   reflect this ordering in Claude Code's task list?"* — only proceed
   on explicit yes.

## Companion: TaskPriorityReorder

The existing `TaskPriorityReorder` skill at the repo root handles the
**manual** override ("bump X to top", "swap X and Y", "demote X").
This `triage` skill will handle the **automatic / signal-driven**
recommendation ("what should I do next?").

The two skills coexist by design — see the top-level
[`README.md`](../README.md) for the manual / automatic split.

## Why a placeholder

Reserving the directory now means:

- The install path advice in the top-level README can already point
  here without 404-ing once v0.9 lands.
- The companion `TaskPriorityReorder/SKILL.md` cross-references the
  forthcoming `triage` skill without those links going stale.
- Anyone watching the repo can see the planned shape of the skill
  before any code lands.

When v0.9 ships, the only diff to this file will be: add frontmatter,
replace this "Why a placeholder" section with the live procedure
section, and the skill will activate on next `/hooks` reload.

Proudly Made in Nebraska. Go Big Red! 🌽 https://xkcd.com/2347/
