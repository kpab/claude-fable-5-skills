# Changelog

All notable changes to this collection are recorded here. Dates are release dates of the version bump.

## [1.3.0] - 2026-09-02

Catch-up for Claude Fable 5.1 (released 2026-09-01). Fable 5 prompts run unchanged on 5.1 per Anthropic; this release folds in the documented behavior differences. Sources are recorded in `docs/official-verification.ja.md` (追補 6〜17).

### Changed
- All skill `description` fields now name "Claude Fable 5 / 5.1" so triggering works on either generation.
- `effort-calibrator`: re-sweep effort on 5.1 (level names don't map to the same thinking across generations); 5.1 reference points (`medium` ≈ Fable 5 at lower cost, `low` competitive with Opus/Sonnet); pipeline pattern split by generation — per-message effort changes on 5.1 (beta header `mid-conversation-output-config-2026-07-01`, cache preserved) versus separate-request escalation on Fable 5; new adjustment signals for search suppression at `low` and long deliverables at `xhigh`/`max`; cache-read pricing (0.025× base input) and its effect on early compaction; claude.ai / Cowork default of Medium.
- `no-gold-plating`: rules for pre-existing bugs (report, don't fix), ambiguous tasks (most direct reading, state the assumption), and test files (only where the task or repo convention calls for them, sized like neighbors, no promoted scratch checks); fourth self-check question on test-file proportion.
- `skill-refactorer`: two new red flags — anti-formatting rules written for older models, and "hold findings for the final response" narration suppressors; description also covers Fable 5 skills now running on 5.1.
- `autonomous-continuation`: new "Delivering the whole task" section (finish independent parts before asking, complete everything not blocked, run decided steps instead of announcing them); "don't stop because the session is long" folded into the contract; harness note on append-only history and turn-scoped system messages (`clear_at: "next_user_message"`, beta header `mid-conversation-system-clear-at-2026-08-21`).
- `grounded-progress`: harness note on receiving 5.1 progress updates (`thinking.display: "updates"`, beta header `thinking-display-updates-2026-08-18`) and on telling the model when tool output is hidden.
- `regrounding-summary`: opening one-line statement and short updates before the recap; short sentences and literal phrasing over mannered prose; mark reused source wording as a quotation.
- `subagent-orchestration`: harness shape for a non-blocking lead (launch returns immediately, results arrive in later user messages, separate wait tool); per-turn batching nudge for one-call-per-turn loops.
- `act-when-ready`: clarified that the planning limit does not limit progress updates.
- README (EN/JA), root `SKILL.md`, plugin manifests: 5.1 release date, legacy status of Fable 5, link to the 5.1 prompting guide, API-level breaking changes noted as out of scope.

### Docs
- `docs/official-verification.ja.md`: new 2026-09-02 addendum with quoted sources; summary table gains a "5.1 guide section" column noting that over-planning, memory, and fabricated-progress guidance exist only in the Fable 5 guide.

## [1.2.0] - 2026-08-29

### Changed
- `effort-calibrator`: separate-request escalation in the pipeline pattern, with the prompt-cache rationale; API shape section reconfirmed.
- `subagent-orchestration`: handoff template asks for the reason behind the goal.
- `autonomous-continuation`: follow-up offers after completion distinguished from mid-run permission questions; `send_to_user` tool pattern for harness authors.
- `skill-refactorer`: on-the-fly skill updates allowed under the existing guardrails.

### Added
- CI workflow running `claude plugin validate --strict` on manifests and skills.
- `docs/official-verification.ja.md`: 2026-08-29 addendum.

## [1.1.1] - 2026-07-10

### Changed
- `effort-calibrator`: API shape section added; workload table reorganized; imperative wording softened to conditional guidance.

## [1.1.0] - 2026-06-11

### Fixed
- `effort-calibrator`: Claude Code default effort corrected to `high` (was stated as `xhigh`); "beats older models at max" changed to `xhigh` per official wording.

### Added
- `docs/official-verification.ja.md`: skill-by-skill verification against official Fable 5 docs.
- Root `SKILL.md` for directory-listing compatibility.

## [1.0.1] - 2026-06-11

### Changed
- README explains activation paths for the six behavioral skills.
- `act-when-ready`: thinking blocks exempted from the rules.
- `effort-calibrator`: `max` effort level documented.

## [1.0.0] - 2026-06-11

Initial release with 10 Fable 5-native agent skills.
