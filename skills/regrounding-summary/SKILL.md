---
name: regrounding-summary
description: Make Claude Fable 5's final report readable to someone who saw none of the work. Use for long agentic sessions, overnight runs, and multi-tool tasks — whenever summaries come back as arrow-chain shorthand, invented abbreviations, or references to reasoning the reader never saw. Pair with grounded-progress for status content; this skill governs the prose.
---

# Re-grounding Summary

During a long run the model builds private vocabulary — abbreviations, arrow chains, names for intermediate artifacts. Efficient while working; opaque in a final report. The reader is seeing everything for the first time, so the summary must be a re-grounding, not a continuation of the working thread.

## Rules for the final message

- Open with the outcome: one sentence answering "what happened?" or "what did you find?" — the TL;DR the reader would ask for. Detail follows, never leads.
- Complete sentences. No arrow chains (A → B → fails), no hyphen-stacked compound labels, no abbreviations you coined mid-run. If a term you built up is worth keeping, reintroduce it as if new.
- Never reference your own working notes or reasoning as if the reader saw them ("as established above" pointing at tool transcripts).
- Identifiers — files, commits, flags, endpoints — each get their own plain-language clause: what it is, why it matters here.
- Selectivity over compression: shorten by dropping details that wouldn't change the reader's next action, not by squeezing grammar out of the sentences. Readable beats short.
- Close with the one or two things needed from the reader, if any.

## Shape

1. Outcome (1 sentence)
2. What was done / found, in reader-facing language
3. Failures, skips, open questions — stated plainly
4. What's needed from you: ...

Working shorthand between tool calls is fine — that's thinking out loud. This skill applies the moment you address the human.
