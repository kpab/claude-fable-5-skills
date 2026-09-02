---
name: effort-calibrator
description: Choose and tune the effort setting for Claude Fable 5 / 5.1 workloads. Use whenever the user asks which effort level to use ("what effort should this batch job run at?", "is xhigh worth it here?"), complains that Fable 5 or 5.1 is slow or expensive, wants to cut latency or token cost, designs a pipeline/harness that mixes routine and hard tasks, migrates a tuned effort setting from Fable 5 to 5.1, or reviews API code that sets output_config.effort.
---

# Effort Calibrator

On Fable 5 and 5.1, effort is the primary dial trading intelligence against latency and cost. Settings inherited from earlier models are usually wrong here, and so are settings inherited from Fable 5 by 5.1: an effort name does not buy the same amount of thinking across generations, so re-run your sweep on 5.1 even if you already tuned Fable 5. Reference points: Fable 5 at lower effort frequently beats earlier models at `xhigh`; Fable 5.1 at `medium` roughly matches Fable 5 at lower cost, and at `low` it often scores higher than Opus and Sonnet models at a similar cost per task, so include it wherever you would otherwise run a smaller model at higher effort.

## Starting points by workload

| Workload | Start at |
|---|---|
| Routine transforms, classification, short edits, chat, subagents | `medium` (try `low` if latency matters) |
| Most analysis and writing | `high` (the general default) |
| Coding and agentic/tool-heavy work | `high` (the API and Claude Code default on Fable 5 and 5.1) — even for workloads that ran at `xhigh` on earlier Opus models; escalate to `xhigh` only for the most capability-sensitive tasks |
| Hardest capability-sensitive work: large migrations, multi-day autonomous runs, novel research | `xhigh` — on 5.1 this is where the gains over Fable 5 are largest, at the price of longer thinking before the first response |
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
- At `low`, 5.1 answers from memory where it should have called a search or retrieval tool — raise effort for the turns that need fresh information, or add a prompt line saying that recognizing a name is not the same as knowing its current state

Watch `xhigh` and `max` on long deliverables (full rewrites, large tables, whole files): 5.1 can think for a long time first, sometimes drafting the deliverable in thinking and then writing it again as the reply. Run these at `high` unless evals show a gain; if you go higher, size `max_tokens` for thinking plus reply, and tell the model to use reasoning for decisions and the output for the text rather than composing the full deliverable twice.

## Pipeline pattern

Route by task class, not by a single global setting: a triage step at `low`/`medium` that escalates hard cases to `high`/`xhigh` usually dominates any fixed choice on both cost and quality. How you escalate depends on the generation:

- **Fable 5.1** (and Opus 5): change effort inside the conversation with a per-message setting — a `role: "system"` message carrying only `output_config.effort`, beta header `mid-conversation-output-config-2026-07-01`. The cached prefix survives, and it steers the model more reliably than changing the top-level value. Treat effort as per-step rather than per-session: raise it for the hard step, drop it back afterward.
- **Fable 5**: effort is request-level and part of the rendered prompt, so changing it mid-conversation invalidates the cached prefix. Escalate by re-submitting the hard case as a separate request, which leaves the triage conversation's cache intact for its own later turns.

Cost inputs also moved on 5.1: cache reads cost a quarter of the Fable 5 rate (0.025× base input, against 0.1× on other models), so long sessions that re-read a cached prefix are cheaper and compacting early to save money may no longer pay off. For long agentic loops, effort composes with [task budgets](https://platform.claude.com/docs/en/build-with-claude/task-budgets), an advisory token budget for the whole loop.

## API shape

Set effort via `output_config`: `{"output_config": {"effort": "low" | "medium" | "high" | "xhigh" | "max"}}`. Setting `high` is equivalent to omitting the parameter; the API and Claude Code default to `high` on both generations, while claude.ai and Cowork default to Medium. `max_tokens` is a hard limit on total output — thinking plus response text — so at higher effort leave enough headroom; short-response workloads may not need a change. Verify parameter names, values, and beta headers with the current API reference before shipping: https://platform.claude.com/docs/en/build-with-claude/effort
