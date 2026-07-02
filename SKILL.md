---
name: claude-fable-5-skills
description: 10 Agent Skills built specifically for Claude Fable 5 — intent, boundaries, and verification hooks instead of the step-by-step micromanagement older-model skills relied on. See skills/ for the individual skill files.
---

# claude-fable-5-skills

10 battle-ready Agent Skills for Claude Fable 5 — the first skill collection designed around how Claude Fable 5 actually behaves, not how older models needed to be babysat.

See [README.md](README.md) for the full list and design principles. Each individual skill lives in its own folder: `skills/<name>/SKILL.md`.

## The 10 skills

- `skill-refactorer` — audits pre-Fable-5 skills/prompts, deletes capability-compensation scaffolding, keeps real guardrails
- `act-when-ready` — stops over-planning and re-deriving settled facts at high effort
- `effort-calibrator` — picks the right effort level per workload
- `no-gold-plating` — prevents diffs bigger than the ask
- `grounded-progress` — ties status reports to tool-result evidence
- `scope-guard` — separates diagnosis from unrequested state changes
- `subagent-orchestration` — patterns for parallel delegation and fresh-context verifiers
- `markdown-memory` — file-based lesson memory with maintenance discipline
- `autonomous-continuation` — keeps unattended runs from stalling
- `regrounding-summary` — final reports readable by someone who saw none of the work

## Install

```
/plugin marketplace add kpab/claude-fable-5-skills
/plugin install fable5-skills@claude-fable-5-skills
```
