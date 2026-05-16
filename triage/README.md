# `triage` skill — placeholder

This directory reserves the slot for the `triage` Claude Code skill,
the automatic / signal-driven counterpart to `TaskPriorityReorder`.
It's not yet a live skill — see [`SKILL.md`](SKILL.md) for the
forward-declared procedure.

## What this skill will do

Wrap the [Triage](https://github.com/CryptoJones/Triage) CLI and
surface a signal-driven priority recommendation:

```
                 ┌────────────────────────────────────────┐
   user asks ──► │ "what should I do next?"               │
                 │ "what's hot right now?"                │
                 │ "reorder by signal"                    │
                 └────────────────┬───────────────────────┘
                                  ▼
                 ┌────────────────────────────────────────┐
                 │  triage tick --theme bbs               │
                 │  triage list --json                    │
                 │  triage why <id>  (top N)              │
                 └────────────────┬───────────────────────┘
                                  ▼
                 ┌────────────────────────────────────────┐
                 │  rendered BBS-styled table back to     │
                 │  the operator with rule contributions  │
                 │  (operator confirms before any         │
                 │   reorder of Claude Code's task list)  │
                 └────────────────────────────────────────┘
```

## Why it doesn't fire yet

The `SKILL.md` in this directory deliberately has **no frontmatter**.
Claude Code only loads skills that declare a `name` and `description`
in frontmatter, so this directory is invisible to the skill loader
until the implementation is ready. That avoids accidental triggers
while we're still designing the contract.

When [Triage v0.7](https://github.com/CryptoJones/Triage/blob/main/DESIGN.md)
ships, this README will be replaced with the live install / usage
docs and `SKILL.md` will gain its frontmatter.

## Companion: `TaskPriorityReorder`

The existing [`TaskPriorityReorder`](../SKILL.md) skill at the
repo root is the **manual** override: "bump X to top", "swap X
and Y", "demote X". This `triage` skill will be the **automatic
recommender**: "what should I do?" answered from the Triage CLI's
signal-driven priority queue.

The two coexist by design — see the top-level
[`README.md`](../README.md) for the manual / automatic split.

Proudly Made in Nebraska. Go Big Red! 🌽 https://xkcd.com/2347/
