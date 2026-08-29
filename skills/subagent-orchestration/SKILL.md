---
name: subagent-orchestration
description: Patterns for delegating work to parallel subagents with Claude Fable 5 — when to split tasks, async coordination, long-lived workers, and fresh-context verifiers. Use when designing multi-agent harnesses, when a large task has independent parts, when runs bottleneck on sequential steps, or when self-review keeps missing its own mistakes.
---

# Subagent Orchestration

Fable 5 dispatches and sustains parallel subagents far more dependably than prior models. Used well this cuts wall-clock time and improves verification quality; used badly it burns tokens on coordination overhead. The patterns:

## When to delegate

Split out a subtask when it is (a) independent of your current working context, (b) large enough to amortize the handoff, and (c) specifiable in a few sentences plus file pointers. Don't delegate tightly coupled edits — coordination costs exceed parallelism gains.

## Coordination

- Launch independent subagents in the same turn and keep working while they run; don't block on the slowest one.
- Intervene only on signal: a subagent off-track or missing context it can't discover itself.
- Prefer long-lived subagents that carry context across related subtasks over respawning per subtask — repeated context loading is the dominant hidden cost.

## Fresh-context verifiers

For checking finished work, a separate verifier subagent with a clean context outperforms self-critique: it can't share your blind spots because it doesn't share your assumptions. Give the verifier the *specification* and the *output* — not your reasoning — and have it report discrepancies against the spec at a defined cadence (every N components, every X hours) rather than once at the end.

## Handoff template

A good subagent brief contains exactly: goal and the reason behind it (one sentence each; intent lets the subagent connect the task to what it finds), inputs (paths/data), definition of done (checkable), constraints (what not to touch), and where to write results. If you can't fill these in, the task isn't ready to delegate.
