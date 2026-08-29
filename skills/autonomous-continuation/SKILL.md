---
name: autonomous-continuation
description: Keep Claude Fable 5 running to completion in unattended pipelines — no stopping on a statement of intent, no mid-run permission questions, no self-imposed session splits over context worries. Use for overnight runs, scheduled jobs, CI agents, and any harness where nobody is watching to type "continue".
---

# Autonomous Continuation

Deep into long sessions, two rare but expensive stalls can appear: ending a turn on a promise ("I'll now run the migration") without the tool call, and pausing to ask permission the original request already granted. Unattended, each stall is a dead pipeline until a human notices. Prevention is a turn-ending discipline plus an autonomy contract.

## Autonomy contract (for the system prompt of unattended runs)

You are operating without a human in the loop; questions cannot be answered mid-run. For reversible actions within the original request's scope, proceed rather than asking permission; once the work is complete, offering follow-ups that fall outside that scope is fine. Stop and end the turn only for: irreversible/destructive actions not clearly covered by the request, a genuine scope change, or missing input that only the user possesses. In those cases state precisely what you need.

## Turn-ending check

Before ending any turn, read your final paragraph. If it is a plan, a question you could answer yourself, a list of in-scope next steps, or a first-person promise about undone work — the turn is not over. Execute, then end. A turn legitimately ends in exactly two states: task complete, or blocked on user-only input.

## Context-budget composure

If the harness surfaces remaining-context numbers, Fable 5 can preemptively offer to summarize and hand off. Where possible, don't show the countdown. If it must be shown, add: context is managed by the harness; do not stop, trim your work, or propose a new session on account of it.

## Mid-run messages to the user (for the harness author, not the model)

When the user must see content verbatim before the run ends — a partial deliverable, an answer to a question they asked mid-loop — give the agent a client-side `send_to_user` tool and render its input directly in your UI. Tool inputs are never summarized, and calling the tool doesn't end the turn. Defining it isn't enough: Fable 5 rarely calls it unprompted, so pair it with an instruction to use it for user-facing content only, never narration or reasoning.

## Checkpoint placement (attended runs)

When a human *is* available, pause only at points that genuinely need them — destructive steps, scope changes, user-only input — and ask the question as the turn's final act, not buried mid-report.
