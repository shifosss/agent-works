## Workflow Orchestration

### 0. Identity

Always call me AlexZ. Use this address at the start of any response.

### 1. Planning

#### 1.1 When to plan

- Enter plan mode for any non-trivial task (3+ steps or architectural decisions). Bug fixes may bypass plan mode under §6 — see §6.1.
- Use plan mode for verification steps, not just building.
- Write detailed specs upfront to reduce ambiguity.
- Ask whether to use the `superpowers:brainstorming` skill before planning a large feature update (architecture change, pipeline workflow change).
- If the work goes sideways, STOP and re-plan immediately. Do not keep pushing.

#### 1.2 Plan mode is the approval gate; `tasks/todo.md` is the record

- Plan mode (Claude Code's built-in) gates execution: no edits until the user approves.
- After approval, write the plan into `tasks/todo.md` as the first action. The file persists across sessions.
- Update item status (`[ ]` → `[x]`) as each step completes.

#### 1.3 Plan format

Each step is a row:

```
1. <step> → verify: <check>
2. <step> → verify: <check>
```

The verify clause is mechanical (test passes, command exits 0, output matches), not a vibe ("looks right").

#### 1.4 Assumptions section

Plan-mode drafts begin with a short **Assumptions** section listing load-bearing premises — assumptions where, if wrong, the plan direction changes. Minor assumptions (test runner, formatter, etc.) are not surfaced.

#### 1.5 Interpretations section

If the task itself has 2+ plausible readings, list them under **Interpretations** and ask the user to pick before drafting steps. This applies to task-level ambiguity only, not implementation forks.

#### 1.6 Progress reporting

- Provide a high-level summary at each step.
- After completing the plan, append a `## Review` section to `tasks/todo.md` documenting outcomes, deviations from plan, and any follow-up items.

### 2. Dispatching Work

#### 2.1 Parallel dispatch (all conditions must be met)

- 3+ unrelated tasks or independent domains.
- No shared state between tasks.
- Clear file boundaries with no overlap.

#### 2.2 Sequential dispatch (any condition triggers)

- Tasks have dependencies (B needs output from A).
- Shared files or state (merge-conflict risk).
- Unclear scope (need to understand before proceeding).

#### 2.3 Background dispatch

- Research or analysis tasks (not file modifications).
- Results are not blocking the current work.

#### 2.4 Model tier policy

Recommendation; choose by judgment.

|   Tier   | Model        | Use when                                                     |
| :------: | ------------ | ------------------------------------------------------------ |
|  Heavy   | `opus 4.6`   | Architecture design, multi-file refactors, novel algorithm implementation, security/correctness audits, debugging subtle concurrency or numerical issues |
| Standard | `sonnet 4.6` | Everyday coding, code review, test writing, config generation, standard debugging, API integration, codebase search, doc generation |
|  Light   | `Haiku`      | Dependency checks, lint/format, log parsing, boilerplate scaffolding |

### 3. Domain Ownership

When dispatching parallel agents (per §2.1), split work along non-overlapping domain boundaries. Each agent owns its domain end-to-end. No file overlap.

Common splits:

- **ML pipelines** : Model/training | Serving/infra | Evaluation
- **Web apps** : Frontend | Backend | Infra/deploy
- **Data systems** : Ingestion | Storage/schema | Query/API

If a project does not fit a common split, define your own up front and write it into `tasks/todo.md` before dispatching.

### 4. Execution Discipline

#### 4.1 Simplicity first

- Make every change as simple as possible. Touch the minimum code that solves the problem.
- No features beyond what was asked. No abstractions for single-use code. No flexibility that was not requested.

#### 4.2 Surgical changes

- Touch only what the task requires. Every changed line should trace directly to the user's request.
- Elegance refactors are bounded to files already being modified for the current task. Never reach into untouched files in the name of elegance.
- Match existing style within a file, even when you would write it differently from scratch.
- If you notice unrelated dead code or hackiness in an untouched file, mention it — do not delete or rewrite it.

#### 4.3 No laziness

- Find root causes. No temporary fixes. Senior-developer standards.
- A passing test built on a symptom-level fix is technical debt — see §5.2.

### 5. Verification

#### 5.1 Goal conversion

Convert every non-trivial task to a verifiable goal before starting. The converted goal is the completion contract — work is not complete until the goal evaluates true.

Examples:

- "Add validation" → "Write tests for invalid inputs, then make them pass."
- "Fix the bug" → "Write a test reproducing the bug, then make a root-cause fix."
- "Refactor X" → "Tests pass before and after; behavior unchanged."

For genuinely qualitative tasks (UX, design feel), state the closest heuristic check and flag it as qualitative.

#### 5.2 Red phase: investigate, then fix

When a red phase appears (failing test, broken build, surfaced error), do NOT default to the smallest patch that turns it green. Sequence:

1. Investigate the root cause.
2. Confirm the cause with evidence (logs, additional tests, narrowed reproduction).
3. Propose the simplest direct fix that addresses the cause.
4. Only then go green.

If root-cause investigation reveals the fix needs multi-file changes, promote to plan mode per §6.1.

#### 5.3 Done criteria

- Never mark a task complete without proving it works.
- Diff behavior between `main` and your changes when relevant.
- Ask: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness.

### 6. Bug Handling

#### 6.1 Severity gate (default autonomous, promote when needed)

- **Default — autonomous fix.** For single-file, clear-cause bugs (typo, off-by-one, obvious null check), just fix it. No clarification, no plan mode. Diagnose, fix, verify, report.
- **Promote to plan mode** when any of the following hold:
  - Multi-file changes required.
  - Cause is unclear after initial diagnosis.
  - Touches auth, data, schema/migrations, or public APIs.
- **In-flight promotion.** If a fix that started in autonomous mode discovers any of the above mid-stream, stop and promote.

#### 6.2 Irreversibility pause

During an autonomous fix, if a fork is encountered that is hard to reverse — data deletion, public API signature change, migration, permanent destructive operation — stop and surface the options to the user. Reversible diagnostic forks ("which file is the bug in?") are resolved autonomously.

#### 6.3 Postmortem record

After a non-trivial bug fix, append an entry to `docs/bugs-and-fixes.md` (relative to the working-directory root; create `docs/` if missing). Include:

- Bug summary.
- Symptoms / reproduction.
- Root cause.
- Solutions tried, in order.
- Final fix.

### 7. Learning

#### 7.1 Two stores

- `tasks/lessons.md` — process and behavioral lessons (corrections, preferences, workflow rules). Cross-project.
- `docs/bugs-and-fixes.md` — technical bug postmortems (per §6.3). Project-local.

#### 7.2 Lesson capture

- After any user correction, append a lesson to `tasks/lessons.md` describing the pattern and the rule that prevents recurrence.
- Iterate ruthlessly until the same mistake stops appearing.
- Read `tasks/lessons.md` at session start.

#### 7.3 Meta-audit

The orchestration is working if:

1. Diffs touch only what was requested.
2. Fewer rewrites due to overcomplication.
3. Clarifying questions arise before implementation, not after mistakes.
4. Repeat corrections decline over time.

At the end of each work session (or weekly, whichever comes first), self-audit against these signals. If a signal is consistently negative, write a meta-lesson into `tasks/lessons.md` proposing an orchestration change.

#### 7.4 Orchestration changes require approval

Self-audit produces a *proposal*. Edits to `Workflow Orchestration.md` itself require explicit user approval before any change is applied. The audit can flag and recommend; it cannot apply.

### 8. Version Control

- Manage the repo using git.
- Log progress after each stage, step, or task is finished.
