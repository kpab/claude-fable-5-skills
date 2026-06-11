---
name: grounded-progress
description: Make Claude Fable 5 progress reports verifiable against actual tool results during long autonomous runs. Use for any multi-hour or overnight agent session, scheduled pipelines, CI agents, or whenever the user has been burned by status updates claiming work that wasn't done. Apply before launch, not after a fabricated report appears.
---

# Grounded Progress

On long runs the failure mode isn't usually bad work — it's a report that drifts from the work: "tests passing" when they were never run, "deployed" when the command errored. The fix is an evidence rule applied at report time.

## The evidence rule

Before any progress claim leaves your output, bind it to a tool result from *this* session:

- Completed → name the command/test/check whose output proves it.
- Failed → say so, and include the relevant output verbatim (trimmed, not paraphrased into optimism).
- Skipped or deferred → state it as skipped, with the reason.
- Not yet verified → label it explicitly as unverified; never round up to done.

A claim with no pointable evidence does not ship. Either produce the evidence (run the check now) or downgrade the claim.

## Report shape for long runs

1. One line: overall state (on track / blocked / partially done).
2. Verified completions, each with its evidence pointer.
3. Failures and skips, stated plainly.
4. Unverified work-in-progress, labeled as such.
5. The single thing needed from the user, if anything.

## Anti-patterns

- Hedged completions ("should be working now") — run the check instead.
- Aggregate claims ("all endpoints migrated") when only a sample was verified — report the sample as the sample.
- Re-reporting old evidence for new claims — evidence must postdate the work it certifies.
