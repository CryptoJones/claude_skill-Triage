<div align="center">

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║         c l a u d e _ s k i l l - T r i a g e                ║
║                                                              ║
║     Claude Code skills for the Triage meta-scheduler         ║
║                                                              ║
╚══════════════════════════════════════════════════════════════╝
```

**Skills that put Claude Code in charge of your priority queue.**
Manual overrides for snap reorders, plus the automatic
signal-driven counterpart that wraps the [Triage](https://github.com/CryptoJones/Triage) CLI.

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg?logo=apache)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude_Code-skills-D97757?logo=anthropic&logoColor=white)](https://claude.com/claude-code)
[![Codeberg](https://img.shields.io/badge/Codeberg-CryptoJones%2Fclaude__skill--Triage-2185D0?logo=codeberg&logoColor=white)](https://codeberg.org/CryptoJones/claude_skill-Triage)
[![GitHub](https://img.shields.io/badge/GitHub-CryptoJones%2Fclaude__skill--Triage-181717?logo=github&logoColor=white)](https://github.com/CryptoJones/claude_skill-Triage)

</div>

> Mirrored on both [GitHub](https://github.com/CryptoJones/claude_skill-Triage)
> and [Codeberg](https://codeberg.org/CryptoJones/claude_skill-Triage).
> Issues filed on either forge are welcome; commits land on both.

---

## What's inside

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│   TaskPriorityReorder   ✓ shipped (manual override)          │
│   ─────────────────────────────────────────────────          │
│   "Bump task X to the top." Captures every pending task's    │
│   full content, deletes them, recreates them in the new      │
│   order — because Claude Code's TaskCreate issues IDs        │
│   append-only, so reordering is delete + recreate.           │
│                                                              │
│                                                              │
│   triage                ◌ planned (Triage v0.9)              │
│   ─────────────────────────────────────────────────          │
│   "What should I do next?" Invokes `triage tick` from the    │
│   Triage CLI, parses the output, surfaces the recommended    │
│   priority order to the operator — with rule contributions.  │
│   The human still confirms; never auto-executes a reorder.   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

By design, the two are complementary:

| Skill                  | Trigger style                                      | Mechanism                                          |
|------------------------|----------------------------------------------------|----------------------------------------------------|
| `TaskPriorityReorder`  | "bump X", "swap X and Y", "this is more urgent"    | Delete + recreate against Claude Code's task tools |
| `triage`               | "what should I do?", "what's hot?", "reorder by signal" | Wraps the [Triage](https://github.com/CryptoJones/Triage) CLI |

---

## Install

The repo is laid out so a single clone yields both the live
`TaskPriorityReorder` skill (at the root) and the
[reserved `triage/` slot](triage/) for the forthcoming
`triage` skill:

```
claude_skill-Triage/
├── SKILL.md         ← TaskPriorityReorder (live)
└── triage/
    ├── SKILL.md     ← placeholder (Triage v0.9)
    └── README.md
```

```bash
# GitHub
git clone https://github.com/CryptoJones/claude_skill-Triage \
  ~/.claude/skills/triage-suite

# or Codeberg
git clone https://codeberg.org/CryptoJones/claude_skill-Triage \
  ~/.claude/skills/triage-suite
```

Claude Code recursively scans the `~/.claude/skills/` tree for any
`SKILL.md` with frontmatter, so cloning into a parent directory like
`triage-suite/` exposes the root `TaskPriorityReorder` skill now and
will pick up the `triage/` subdirectory automatically once its
`SKILL.md` gains frontmatter at v0.9. No re-install needed.

Restart Claude Code (or open `/hooks` once) for the skill to be picked up.

---

## Triggering

### `TaskPriorityReorder`

Fires on phrasing like:

- "Move task #X to top"
- "Bump task X up the queue"
- "Reorder tasks by priority"
- "Task X is more important than Y" / "...is now top priority"
- Generic "shuffle the queue" framing

Explicit invocation: `/TaskPriorityReorder`.

Full trigger catalog, in-progress task handling, and a worked example
live in [`SKILL.md`](SKILL.md).

### `triage` *(planned — Triage v0.9)*

Will fire on phrasing like:

- "What should I do next?"
- "What's hot right now?"
- "Reorder by signal"
- "Show me the triage queue"

Invokes `triage tick`, surfaces the top-N tasks with the rule
contributions that pushed them there. Operator confirms before
anything is reordered in Claude Code's own task list.

---

## Why a skill, not a memory rule

Memory rules can be forgotten under context pressure. Skill triggers
fire automatically when the frontmatter description matches the
operator's phrasing. That's the right tool for **multi-step
procedures with a precise sequence**, where missing or reordering any
step loses information irrecoverably.

`TaskPriorityReorder` is exactly that shape: capture → verify →
delete → recreate. Drop one step, lose content.

---

## Companion project: Triage

[**Triage**](https://github.com/CryptoJones/Triage) is the meta-scheduler
this skill repo wraps. Triage watches signals (cron windows, CI status,
deadlines, blockers, future runpod cost) and reorders its own priority
queue, with every reorder explainable via `triage why <id>`.

```
human says "bump X"     ──►   TaskPriorityReorder  ──►  Claude Code tasks
                                                              ▲
signals from world      ──►   Triage CLI           ──►  triage skill ──┘
```

---

## License

Apache 2.0. See [LICENSE](LICENSE).

Proudly Made in Nebraska. Go Big Red! 🌽 https://xkcd.com/2347/
