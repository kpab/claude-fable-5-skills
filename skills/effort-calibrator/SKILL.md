---
name: effort-calibrator
description: Choose and tune the effort setting for Claude Fable 5 workloads. Use whenever the user asks which effort level to use ("what effort should this batch job run at?", "is xhigh worth it here?"), complains that Fable 5 is slow or expensive, wants to cut latency or token cost, designs a pipeline/harness that mixes routine and hard tasks, or reviews API code that sets output_config.effort.
---

# Effort Calibrator

On Fable 5, effort is the primary dial trading intelligence against latency and cost. Defaults that were right for earlier models are usually wrong here: Fable 5 at lower effort frequently beats earlier models at `xhigh`, so over-provisioning effort wastes money and time without adding quality.

## Starting points by workload

| Workload | Start at |
|---|---|
| Routine transforms, classification, short edits, chat, subagents | `medium` (try `low` if latency matters) |
| Most analysis and writing | `high` (the general default) |
| Coding and agentic/tool-heavy work | `high` (the API and Claude Code default on Fable 5) — even for workloads that ran at `xhigh` on earlier Opus models; escalate to `xhigh` only for the most capability-sensitive tasks |
| Hardest capability-sensitive work: large migrations, multi-day autonomous runs, novel research | `xhigh` |
| Frontier problems only, where evals show headroom above `xhigh` and token spend is unconstrained | `max` |

The signal for `max` is evals showing headroom above `xhigh` on your actual task: on most workloads it adds significant cost for small gains and can tip into overthinking.

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

Route by task class, not by a single global setting: a triage step at `low`/`medium` that escalates hard cases to `high`/`xhigh` usually dominates any fixed choice on both cost and quality. For long agentic loops, effort composes with [task budgets](https://platform.claude.com/docs/en/build-with-claude/task-budgets), an advisory token budget for the whole loop.

## API shape

Set effort via `output_config`: `{"output_config": {"effort": "low" | "medium" | "high" | "xhigh" | "max"}}`. Setting `high` is equivalent to omitting the parameter. `max_tokens` is a hard limit on total output — thinking plus response text — so at higher effort leave enough headroom for extended thinking; short-response workloads may not need a change. Verify parameter names and values with the current API reference before shipping: https://platform.claude.com/docs/en/build-with-claude/effort
