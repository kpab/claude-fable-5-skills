---
name: markdown-memory
description: A file-based lesson memory that Claude Fable 5 reads and maintains across sessions. Use for any recurring agent (daily jobs, long projects, team assistants) where the same corrections keep being re-made, and when bootstrapping a new agent from past session history. Fable 5 benefits from recorded lessons noticeably more than prior models did.
---

# Markdown Memory

Fable 5 is unusually good at exploiting written records of its own past mistakes and confirmed approaches. A directory of Markdown files is enough — no database required. What matters is the maintenance discipline, because a memory full of stale or duplicate notes is worse than none.

## Layout

```
memory/
  lessons/
    one-lesson-per-file.md
  INDEX.md   # one line per lesson, regenerated when lessons change
```

## Lesson file format

- Line 1: a one-sentence summary that makes sense without opening the file.
- Body: what happened, what the correct approach is, and *why it mattered* — the why is what generalizes.
- Record both corrections (things that went wrong) and confirmations (approaches validated under pressure).

## Maintenance rules

- One lesson per file. If a new event refines an existing lesson, update that file; never create a near-duplicate.
- Don't record what the repo, docs, or chat history already state — memory is for what's *not* written down elsewhere.
- Delete lessons proven wrong. A confidently wrong note does more damage than a missing one.
- Read INDEX.md at session start; open full lesson files only when relevant.

## Bootstrapping from history

To seed memory for an existing project, review past sessions (delegating chunks to subagents if history is large), extract recurring themes and corrections, and write them as lesson files in the format above. Then make reading INDEX.md part of the standing instructions.
