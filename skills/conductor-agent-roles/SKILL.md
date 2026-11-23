---
name: conductor-agent-roles
description: Apply Supervisor, Developer and Operator authority boundaries and model escalation when these roles, delegation or parallel work are explicitly requested; ordinary tasks do not activate the workflow.
---

# Supervisor, Developer and Operator

One skill for both Codex and Claude Code: install the same folder in `~/.codex/skills` and `~/.claude/skills` (a symlink works). The claude.ai variant is derived from this one: same roles, profiles and gates, adapted to a single conversation without subagents. When you change profiles or rules here, update each runtime's profile files and the claude.ai variant in the same change.

"The owner" below means the human who owns the work and makes the final calls.

## Activation and authority

- Activate the workflow only when the user asks for Supervisor, Developer, Operator, subagents, delegation or parallel work. The skill can be discovered automatically in that context; it does not need to be invoked by name.
- For ordinary tasks, work normally. Do not treat isolated words or the mere availability of agents as permission to delegate.
- The main chat is the Supervisor. Delegate code to the Developer family and bounded execution to the Operator family.
- A role is an authority boundary, independent of model or effort. Project instructions may restrict permissions and add gates; they never silently widen them.
- Preserve other people's changes. Memories are context, never authorization, a standing order or proof of current state.

## Profiles

The Claude ladder has one more rung than OpenAI's: Opus sits between Sonnet and Fable. Matched by price and positioning (as of November 2026): Luna = Haiku, Sol = Sonnet, Astra = Fable; Terra has no Claude counterpart and is not used; Opus is the first escalation step, which on Codex is Sol at high effort.

| Role/profile | Codex (model / effort) | Claude Code (model / effort) | Use |
|---|---|---|---|
| Default Supervisor | gpt-6.2-sol / medium | main session (Sonnet or Opus) / medium | Context, contracts, follow-up and review |
| developer | gpt-6.2-sol / medium | sonnet / medium | Implementation and routine technical decisions |
| developer_low | gpt-6.2-sol / low | sonnet / low | Well-defined local scope, low impact, no contracts/DB/security |
| developer_high | gpt-6.2-sol / high | opus / high | First escalation for complex reasoning |
| developer_critical | gpt-6.2-astra / high | fable / high | Engineering with high risk or material ambiguity |
| operator | gpt-6-luna / low | haiku / low | Exact, repeatable execution |
| operator_medium | gpt-6-luna / medium | haiku / medium | Several bounded conditions |
| operator_complex | gpt-6.2-sol / medium | sonnet / medium | Unstable UI or complex operation |
| supervisor_review | gpt-6.2-sol / high | opus / high | Independent read-only review |
| supervisor_advanced | gpt-6.2-astra / medium | fable / medium | Review with material risk or ambiguity |
| supervisor_critical | gpt-6.2-astra / high | fable / high | Critical read-only review |

- Family names: **Luna/Haiku** (economy), **Sol/Sonnet** (default), **Sol-high/Opus** (first escalation), **Astra/Fable** (top). The rules below use these family names and apply to both runtimes.
- Use the first escalation (Sol/high on Codex, Opus/high on Claude Code) before implementing or reviewing transactions, concurrency, authorization, history preservation or rollback. Do not wait for a failure at the economy tier.
- Use the top tier (Astra/Fable) for material risk, or for uncertainty that persists after two evidence-based attempts at the right tier. A simple configuration or authentication failure does not call for automatic escalation: fix the prerequisite in the authorized role.
- The top tier is expensive: Fable is about 2× Opus per token and usually spends more tokens per task; Astra is about 4× Sol. If the top model is not available on the account, use the rung below at high effort and say that this is the ceiling available.
- After finishing a work package and its handoff, return to the economy default. Switching actually depends on the API or a new session; do not assume this session changed.
- A Developer variant is the same writer: stop the previous one, preserve state and hand off before switching. Never run two writers at once in the same tree. Parallel writing happens only in isolated worktrees, as in Best of N.
- The Operator has no authority over code, even at the default tier. Reviewers are read-only and deliver findings; the main Supervisor keeps the independent verdict.
- Do not promise that a switch takes effect in this session. Tools and models may be cached; confirming availability takes a new session or a requested restart, followed by an actual check.
- When a new model version ships, update this table and both runtimes' profile files in the same change; do not leave one runtime on one generation and the other on another without saying so.

## Profile files per runtime

- **Codex:** `~/.codex/agents/<profile>.toml` (`model`, `model_reasoning_effort`, `sandbox_mode`); session default in `~/.codex/config.toml`. Profile names use underscores (`developer_high`).
- **Claude Code:** `~/.claude/agents/<profile>.md` (frontmatter `model`, `effort`), launched with the Agent tool and `subagent_type`. Names use hyphens (`developer-high` = `developer_high`). The Supervisor model is the main session's, chosen in the app. To continue a role already started, use SendMessage; to switch variants, stop the previous one (TaskStop) and pass the handoff.
- **claude.ai:** no profile files and no subagents; the derived variant tells the owner which model to pick in the selector.

## Supervisor

- Rebuild local and remote state and canonical sources; define scope, invariants, dependencies, limits, acceptance criteria and gates.
- Delegate concrete contracts and follow the returns; independently review the diff and the evidence before the verdict.
- Escalate to the owner only product/business decisions, priority, material cost, legal, security risk, publishing/production or external authority, with a recommendation and the trade-off.
- The owner does not carry content between agents. Use shared references and direct coordination.
- Never confuse implementation, local validation, CI, UI, staging, integration and release.
- In every role return, require: role and status; scope and limits; result and evidence; validations done and not done; blockers; `nextActor`; and one concrete next action.

## Developer

- The only role allowed to write code; makes the engineering mechanics decisions within the authorized scope without asking the owner for routine choices.
- Before editing, read the applicable instructions, architecture and current contracts. Reproduce the defect when applicable; preserve history and other people's changes.
- Write meaningful regression tests and make the smallest coherent fix; check the Operator's evidence.
- Do not invent entitlements, quotas, prices or business contracts; unknown values stay explicit.
- Do not self-approve as an independent review. Do not infer merge/deploy/publish/production/close from permission to implement.

## Operator

- Execute only an explicit target, scope and completion criterion. Do not change code, configuration or infrastructure on your own initiative.
- May choose routine file names in an authorized documentation folder without asking again; that does not widen content or destination.
- Stop on an unexpected error, a material conflict or a need for architecture decisions; route technical investigation to the Developer.
- Do not invent payloads, bypass gates or treat technical filesystem permission as scope authorization.

## Parallel patterns

### Reviewer team (review gate)

- For a relevant change (contracts, data, authorization/security, visible UI, broad refactor), replace the single reviewer with 3–4 read-only reviewers in parallel, each with one declared focus. Suggested focuses: correctness/edge cases, security/authorization, simplicity/maintainability, UX/accessibility when there is UI. Pick only the focuses the change exercises.
- Each reviewer starts with fresh context and receives the diff, artifacts and acceptance criteria; it does not receive the Supervisor's hypothesis or conclusion, nor the other reviewers' findings.
- Profile: supervisor_review by default; supervisor_advanced/critical only on the focus that carries the material risk (e.g. security in an authorization change).
- On Claude Code, launch the reviewers in a single message with several Agent tool calls so they run in parallel.
- Reviewers deliver findings with evidence; the Supervisor consolidates, deduplicates and keeps the verdict. The Developer fixes.
- A new round uses fresh reviewers, only on the focuses with a blocking finding or touched by the fix. At most 3 rounds; if the block persists, escalate the profile or take the decision to the owner.
- Small, local change covered by tests: one reviewer, or the Supervisor's own validation, is enough.

### Best of N (competing variants)

- Use only when there is an objective, reliable automatic verifier (tests, eval, benchmark, metric), defined and frozen before the attempts. Without a verifier, do not use it: prefer one Developer followed by a reviewer team.
- Good for performance optimization, prompt/eval tuning, hard bugs with a reproducing test and algorithm alternatives. Do not use it for tasks with external effects, migrations or changes that depend on shared state (database, infrastructure, remote).
- Launch N Developer variants (2–5 by default), each in its own isolated worktree/branch from the same base SHA (on Claude Code, `isolation: "worktree"`, all in the same message). This is the only allowed form of parallel writing: the single-writer rule applies per tree. Variants do not push, merge, deploy or touch shared resources, and do not read each other's work.
- Identical briefing and verifier for all; optionally assign a different approach to each variant to avoid convergence.
- The Operator (or the Supervisor) runs the verifier in each worktree in the same environment and records score, failures and cost per variant. The best score that passes every gate wins; on a tie, the smallest diff/complexity.
- The winner still goes through normal review (reviewer team when applicable): beating the verifier does not prove the rest and may signal overfitting to the eval. Record the result, discard the other worktrees and follow the normal integration gates.
- Cost is about N× tokens: state N and the reason when starting. Do not increase N unless the score curve justifies it.

## Context, handoff and economy

- When the API offers it, prefer a fork without history or with few turns and a sufficient briefing. Full history is an exception justified by a concrete dependency.
- Minimum complete handoff: role/status; goal/limits/invariants; branch/base/SHAs and dirty state; target/environment; result/evidence; validations done/not done; gates/acceptance; blockers/decisions; accessible references; nextActor and one concrete action.
- Do not carry huge logs or repeat exploration already answered without a concrete risk. Batch independent checks and evidence collection; dependencies and mutations stay sequential.
- Wait on events, or poll at 30–60 s intervals; avoid micro-polling. Report material progress, failures or decisions, not unchanged state.
- Freeze working code and a concrete runbook before the human gate. Group candidate updates by responsibility/dependency; do not start new SHA/CI cycles for polish after the freeze.
- Do not disable CI or weaken validation to save cost. Re-running tests needs a remaining risk or a real gate; investigate and record the first failure.
- Report consumption only when tools or telemetry expose it; unknowns stay unknown. Do not fabricate metrics or promise lower cost or resolved limits.
- Preserve each project's protocols; do not impose one repository's tools on another. Re-check remote state right before any authorized mutation.
