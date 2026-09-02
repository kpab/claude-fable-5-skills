---
name: subagent-orchestration
description: Patterns for delegating work to parallel subagents with Claude Fable 5 / 5.1 — when to split tasks, async coordination, long-lived workers, fresh-context verifiers, and keeping tool calls batched in agent loops. Use when designing multi-agent harnesses, when a large task has independent parts, when runs bottleneck on sequential steps or on one-call-per-turn loops, or when self-review keeps missing its own mistakes.
---

# Subagent Orchestration

Fable 5 and 5.1 dispatch and sustain parallel subagents far more dependably than prior models. Used well this cuts wall-clock time and improves verification quality; used badly it burns tokens on coordination overhead. The patterns:

## When to delegate

Split out a subtask when it is (a) independent of your current working context, (b) large enough to amortize the handoff, and (c) specifiable in a few sentences plus file pointers. Don't delegate tightly coupled edits — coordination costs exceed parallelism gains.

## Coordination

- Launch independent subagents in the same turn and keep working while they run; don't block on the slowest one.
- Intervene only on signal: a subagent off-track or missing context it can't discover itself.
- Prefer long-lived subagents that carry context across related subtasks over respawning per subtask — repeated context loading is the dominant hidden cost.
- Harness shape that makes the first point possible: the launch tool returns immediately, each result comes back to the lead in a later user message when ready, and a separate tool lets the lead wait when it chooses to. The lead will often wait anyway; the savings come from the runs where it carries on.
- Batch independent tool calls. In coding and computer-use loops where the next reads are implied rather than named, 5.1 may issue one call per turn. Send a per-turn nudge — list what you need next, then request everything that doesn't depend on another result in this one response — as a turn-scoped system message appended after each batch of tool results, never by editing earlier turns.

## Fresh-context verifiers

For checking finished work, a separate verifier subagent with a clean context outperforms self-critique: it can't share your blind spots because it doesn't share your assumptions. Give the verifier the *specification* and the *output* — not your reasoning — and have it report discrepancies against the spec at a defined cadence (every N components, every X hours) rather than once at the end.

## Handoff template

A good subagent brief contains exactly: goal and the reason behind it (one sentence each; intent lets the subagent connect the task to what it finds), inputs (paths/data), definition of done (checkable), constraints (what not to touch), and where to write results. If you can't fill these in, the task isn't ready to delegate.
