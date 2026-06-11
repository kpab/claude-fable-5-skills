---
name: effort-calibrator
description: Choose and tune the effort setting for Claude Fable 5 workloads. Use whenever the user asks which effort level to use ("what effort should this batch job run at?", "is xhigh worth it here?"), complains that Fable 5 is slow or expensive, wants to cut latency or token cost, designs a pipeline/harness that mixes routine and hard tasks, or reviews API code that sets output_config.effort.
---

# Effort Calibrator

On Fable 5, effort is the primary dial trading intelligence against latency and cost. Defaults that were right for earlier models are usually wrong here: Fable 5 at lower effort frequently beats earlier models at their maximum, so over-provisioning effort wastes money and time without adding quality.

## Starting points by workload

| Workload | Start at |
|---|---|
| Routine transforms, classification, short edits, chat | `medium` (try `low` if latency matters) |
| Most analysis and writing | `high` (the general default) |
| Coding and agentic/tool-heavy work | `high`, escalating to `xhigh` for harder tasks — `xhigh` is the best setting for most coding/agentic work and is Claude Code's default |
| Hardest capability-sensitive work: large migrations, multi-day autonomous runs, novel research | `xhigh` |
| Frontier problems only, where evals show headroom above `xhigh` and token spend is unconstrained | `max` |

`max` is rarely the right answer: on most workloads it adds significant cost for small gains and can tip into overthinking. Reach for it only when `xhigh` has measurably fallen short.

## Adjustment signals

Lower effort when:
- Tasks complete correctly but take longer than the work warrants
- The session is interactive and waiting hurts more than marginal quality helps
- Output shows over-deliberation: long context-gathering before trivial actions

Raise effort when:
- First-shot correctness matters more than turnaround (one-way-door changes, unattended runs)
- The task benefits from rigorous self-verification, which higher effort does noticeably better
- A task failed at the current level in a way that looks like shallow reasoning, not missing information

## Pipeline pattern

Route by task class, not by a single global setting: a triage step at `low`/`medium` that escalates hard cases to `high`/`xhigh` usually dominates any fixed choice on both cost and quality. Verify with the current API reference before shipping parameter names or values: https://platform.claude.com/docs/en/build-with-claude/effort
