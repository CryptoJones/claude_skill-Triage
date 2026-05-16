# claude-skill-TaskPriorityReorder

A [Claude Code](https://claude.com/claude-code) skill that re-orders the
pending task list by priority. The `TaskCreate` tool issues IDs
append-only — so "bump task X to the top" isn't a single API call. The
skill captures every pending task's full content, deletes them, and
recreates them in the new desired order, preserving subject +
description + activeForm + metadata.

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg?logo=apache)](LICENSE)
[![Codeberg](https://img.shields.io/badge/Codeberg-CryptoJones%2Fclaude--skill--TaskPriorityReorder-2185D0?logo=codeberg&logoColor=white)](https://codeberg.org/CryptoJones/claude-skill-TaskPriorityReorder)
[![GitHub](https://img.shields.io/badge/GitHub-CryptoJones%2Fclaude--skill--TaskPriorityReorder-181717?logo=github&logoColor=white)](https://github.com/CryptoJones/claude-skill-TaskPriorityReorder)

> Mirrored on both [GitHub](https://github.com/CryptoJones/claude-skill-TaskPriorityReorder) and
> [Codeberg](https://codeberg.org/CryptoJones/claude-skill-TaskPriorityReorder). Issues filed on
> either are welcome; commits are pushed to both.

---

## What it does

Claude Code's `TaskCreate` tool assigns IDs in creation order
(append-only). The agent's documented work-on-tasks-in-ID-order
heuristic means **whatever you created first naturally becomes
highest priority** — which is the wrong default the moment a more
urgent thing comes in.

The skill turns "bump task X to the top" / "reprioritize the queue"
into a deterministic procedure:

1. `TaskList` → see what's pending
2. `TaskGet` each pending one → capture subject + description + activeForm + metadata
3. Resolve the new priority order from the operator's instruction
4. `TaskUpdate status: deleted` on each pending task
5. `TaskCreate` them back in the new order — lowest new ID = highest priority

In-progress tasks are left alone by default (their state can't be
recreated through delete + recreate), with an explicit warning if the
operator asks to reorder one anyway.

See [`SKILL.md`](SKILL.md) for the full skill definition: trigger
phrase list, ordering pattern catalog ("move X to top", "swap X and Y",
"new top-N is A, B, C"), in-progress handling, and a worked example.

## Install

```bash
git clone https://github.com/CryptoJones/claude-skill-TaskPriorityReorder \
  ~/.claude/skills/TaskPriorityReorder
```

Or via Codeberg:

```bash
git clone https://codeberg.org/CryptoJones/claude-skill-TaskPriorityReorder \
  ~/.claude/skills/TaskPriorityReorder
```

Restart Claude Code (or open `/hooks` once) for the skill to be picked up.

## How Claude invokes it

Frontmatter `description` triggers it on any of:

- "Move task #X to top"
- "Bump task X up the queue"
- "Reorder tasks by priority"
- "Task X is more important than Y"
- "This is now top priority"
- Generic "shuffle the queue" framing

Explicit invocation: type `/TaskPriorityReorder` (Claude Code's
slash-command path).

## Why a skill, not a memory rule

Memory rules can be forgotten under context pressure. Skill triggers
fire automatically when the description-keywords match — which is
exactly the failure-mode shape (irrecoverable state loss on a routine
operation if a step is skipped) that skills are built for.

## License

Apache 2.0. See [LICENSE](LICENSE).

Proudly Made in Nebraska. Go Big Red! 🌽 https://xkcd.com/2347/
