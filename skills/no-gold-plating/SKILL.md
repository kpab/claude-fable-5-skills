---
name: no-gold-plating
description: Keep Claude Fable 5 / 5.1 changes minimal — no unrequested refactors, abstractions, defensive code, feature creep, or test files out of proportion to the change. Use in code review, bug fixes, and any coding session where diffs come back bigger than the ask, with extra helpers, error handling, "while I was here" cleanup, or more committed tests than the change warrants. Particularly important at high and xhigh effort.
---

# No Gold-Plating

At higher effort, Fable 5's thoroughness can spill over into the diff: speculative abstractions, defensive checks for impossible states, cleanup nobody requested. Fable 5.1 adds a variant of its own on open-ended features: fixing nearby code, extending behavior the task never mentioned, and committing more test files than the change warrants. Each of these enlarges review surface and risk. The discipline:

## Rules

- The diff should map one-to-one to the request. A bug fix touches the bug; it does not reorganize the neighborhood.
- No helpers, layers, or abstractions for a single call site. Inline it.
- Do not design for requirements that don't exist yet. The simplest correct implementation wins; generality is a cost paid today for a benefit that may never arrive.
- Validate only at system boundaries — user input, external APIs, file contents. Inside the system, trust the framework and your own invariants; do not add error handling for states that cannot occur.
- Prefer changing code over compatibility shims, flags, or deprecation layers, unless the code is a published interface someone else depends on.
- A pre-existing bug, performance issue, or unmentioned behavior you run into stays untouched unless the requested behavior cannot work without fixing it. Report it as a follow-up in your summary.
- When the task is ambiguous, implement the reading its wording and the surrounding code most directly support, state that assumption in your summary, and do not build for the other readings as well.
- Commit tests only where the task asks for them or the repository already keeps tests for this kind of change. Size them like the neighboring test files — roughly one focused test per stated behavior — and never promote scratch checks into permanent test files. Verify however you like meanwhile; scratch scripts need not survive.
- If you notice genuinely valuable adjacent work, note it in one sentence at the end. Do not do it.

This is about extras only: every behavior the task does ask for gets implemented completely.

## Self-check before finalizing a diff

1. Could a reviewer trace every hunk back to the user's request in one step?
2. Did I add any function, parameter, or branch "just in case"?
3. Is any new error-handling path actually reachable?
4. Is the number of test files I'm committing proportionate to the change, and would this repo normally test a change like this?

If any answer is wrong, cut it.
