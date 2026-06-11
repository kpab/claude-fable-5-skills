---
name: skill-refactorer
description: Audit and rewrite existing agent skills, system prompts, and CLAUDE.md files for Claude Fable 5. Use whenever the user mentions migrating, upgrading, porting, or "fixing" a skill or prompt for Fable 5, complains that a previously good skill now produces worse output, or asks why Fable 5 ignores or over-follows old instructions. Also trigger when reviewing any skill written before June 2026.
---

# Skill Refactorer for Claude Fable 5

Skills and prompts written for earlier Claude models often encode *capability compensation*: step-by-step hand-holding, exhaustive enumeration of edge cases, and rigid micro-workflows that existed because older models needed them. Fable 5 follows instructions strictly enough that this old scaffolding becomes a straitjacket — the model obeys the obsolete recipe instead of using its own (now better) judgment. The result is output that is *worse* than no skill at all.

Your job: separate what the instruction *protects* from how it *micromanages*, keep the former, delete the latter.

## Workflow

1. **Inventory.** List every imperative instruction in the target skill/prompt as a separate line item.
2. **Classify each item** into exactly one bucket:
   - **Guardrail** — protects against a real, current risk (data loss, secrets, destructive commands, compliance). KEEP.
   - **Preference** — a genuine style/format choice the owner still wants (tone, language, naming conventions). KEEP, but compress to one sentence if it sprawls.
   - **Compensation** — exists only because an older model would otherwise fail (forced step sequences, "always re-read the file before editing", verbose output templates, repeated "do not forget" reminders, enumerations of failure modes the model no longer exhibits). DELETE or replace with a single statement of intent.
3. **Rewrite.** For every deleted compensation, ask: "what outcome was this trying to guarantee?" If the outcome still matters, state the outcome once, not the procedure. Example:
   - Before: "Step 1: open the file. Step 2: locate the function. Step 3: copy it to a scratch buffer. Step 4: edit the copy. Step 5: diff against the original. Step 6: ..."
   - After: "Edits must be verifiable: keep a way to diff your change against the original before finalizing."
4. **Tighten the description field.** Triggering logic stays in frontmatter `description`; the body should assume the skill already fired.
5. **Length check.** A refactored skill is typically 30–60% shorter. If it isn't, re-run step 2 — you kept compensations.
6. **A/B test.** Run one representative task with the old skill and one with the refactored skill. Compare outputs with the user before declaring victory.

## Red flags that mark a line as compensation

- Numbered procedures where the order doesn't actually matter
- The same warning repeated in different words
- Output templates that fix structure the user never asked for
- "Think step by step", "be careful", "double-check" without a concrete check
- Instructions to narrate or explain internal reasoning in the response (on Fable 5 this can trigger refusals — remove these entirely)

## What NOT to touch

- Domain facts, file paths, API contracts, schemas — these are knowledge, not scaffolding.
- Safety and permission boundaries. When in doubt whether something is a guardrail or a compensation, ask the user; never silently drop a guardrail.
