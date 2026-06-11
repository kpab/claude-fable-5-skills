# claude-fable-5-skills

**10 battle-ready Agent Skills for Claude Fable 5** — the first skill collection designed around how the new Mythos-class model actually behaves, not how older models needed to be babysat.

[日本語版 README はこちら →](README.ja.md)

> Anthropic's own guidance for Fable 5 notes that skills written for prior models are often too prescriptive for it and can *degrade* output quality. Old skills micromanage; Fable 5 needs intent, boundaries, and verification hooks instead. Every skill here is written Fable 5-first.

Works with **Claude Code, Claude Cowork, Cursor, Copilot, Gemini CLI**, and any harness that reads `SKILL.md` files (agentskills.io-style: YAML frontmatter + Markdown body).

## The 10 skills

| # | Skill | What it fixes |
|---|-------|---------------|
| 1 | [`skill-refactorer`](skills/skill-refactorer/SKILL.md) | **The meta-skill.** Audits your pre-Fable-5 skills/prompts, deletes capability-compensation scaffolding, keeps real guardrails. |
| 2 | [`act-when-ready`](skills/act-when-ready/SKILL.md) | Over-planning at high effort: re-deriving settled facts, surveying options it won't pursue. |
| 3 | [`effort-calibrator`](skills/effort-calibrator/SKILL.md) | Picking the right `effort` level per workload; Fable 5 at `medium` often beats older models at max. |
| 4 | [`no-gold-plating`](skills/no-gold-plating/SKILL.md) | Diffs bigger than the ask: unrequested refactors, speculative abstractions, impossible-state error handling. |
| 5 | [`grounded-progress`](skills/grounded-progress/SKILL.md) | Status reports on long runs must point at tool-result evidence — no more "tests passing" that never ran. |
| 6 | [`scope-guard`](skills/scope-guard/SKILL.md) | "Diagnose" ≠ "fix". No unrequested actions, no state changes on pattern-matched evidence. |
| 7 | [`subagent-orchestration`](skills/subagent-orchestration/SKILL.md) | Parallel delegation, long-lived workers, and fresh-context verifier subagents that out-perform self-critique. |
| 8 | [`markdown-memory`](skills/markdown-memory/SKILL.md) | A file-based lesson memory Fable 5 exploits unusually well — with the maintenance discipline that keeps it useful. |
| 9 | [`autonomous-continuation`](skills/autonomous-continuation/SKILL.md) | Unattended runs that stall on "I'll now run X" or mid-run permission questions. Includes the context-budget composure pattern. |
| 10 | [`regrounding-summary`](skills/regrounding-summary/SKILL.md) | Final reports readable by someone who saw none of the work — no arrow chains, no invented abbreviations. |

## Install

**Claude Code (recommended)** — install as a plugin, get updates automatically:

```
/plugin marketplace add kpab/claude-fable-5-skills
/plugin install fable5-skills@claude-fable-5-skills
```

**Manual** — copy any skill folder into your skills directory:

```bash
git clone https://github.com/kpab/claude-fable-5-skills
cp -r claude-fable-5-skills/skills/act-when-ready ~/.claude/skills/
```

Or cherry-pick per project by placing folders under `.claude/skills/` in the repo. Other harnesses: point your skill loader at the `skills/` directory; each skill is a self-contained folder with one `SKILL.md`.

## How the skills activate

Four skills are task-shaped and trigger automatically when a matching request appears: `skill-refactorer`, `effort-calibrator`, `subagent-orchestration`, `markdown-memory`.

The other six are standing behavioral rules. A model that is mid-over-planning won't reliably decide to invoke `act-when-ready` on its own, so don't count on auto-triggering for these. Two reliable paths:

- **Interactive sessions** — invoke explicitly (e.g. `/act-when-ready`) at the start of the session, or the moment the symptom shows up.
- **Unattended pipelines and always-on use** — copy the skill body into your system prompt or `CLAUDE.md`; this is how Anthropic's own Fable 5 guide ships these patterns. `autonomous-continuation` is written for exactly this.

## Design principles

1. **Intent over procedure.** Fable 5 follows instructions strictly; we state outcomes and boundaries, not step-by-step recipes.
2. **Short by construction.** Every skill fits on one screen. If a line doesn't change behavior, it's deleted.
3. **Verification hooks, not vibes.** Where correctness matters, the skill defines a *check* (evidence rule, turn-ending check, diff self-check), not an exhortation to "be careful".
4. **Original text.** Patterns are informed by Anthropic's public [Prompting Claude Fable 5](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5) guide and our own testing; all instruction text here is written from scratch.

## Status & disclaimer

Fable 5 shipped on **2026-06-09**. These skills track a moving target — expect refinements as the community learns the model. Issues and PRs with reproducible before/after examples are very welcome.

This is an unofficial community project, not affiliated with or endorsed by Anthropic. Verify current API parameters against the [official docs](https://platform.claude.com/docs) before production use.

## License

MIT — see [LICENSE](LICENSE).
