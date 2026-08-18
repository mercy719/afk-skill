---
name: afk
description: "Scan today's project context, reconcile unfinished work, and propose an executable plan for review before starting a long-running goal."
argument-hint: "[objective | today | resume | report]"
disable-model-invocation: true
---

# AFK

Turn either an explicit objective or today's unfinished project work into a reviewable proposal, then (only after approval) a persistent Goal.

## Operating rule

Plan first. Goal second. Keep discovery, approval, execution, and handoff as separate states.

Use available evidence before asking questions. Ask only when a safety boundary or a genuine blocker requires the user's judgement.

## Choose a mode

- No argument or `today`: discover unfinished work in the current project from today's Codex sessions.
- A concrete objective: use that objective directly. Do not scan sessions or RAG unless a missing historical fact is needed.
- `resume`: inspect the active Goal and continue its next unfinished task.
- `report`: inspect the active Goal and current project evidence; produce a status report without starting new work.

Before creating anything, check whether an unfinished Goal already exists. Resume or steer it instead of silently creating a second Goal.

## Discover

For the `today` path:

1. Identify the project from the current working directory, git remote, and branch.
2. Read today's sessions scoped to this project. Prefer FreeRag's `CodexAppServerAdapter` (`thread/list`, then `thread/read(includeTurns=True)`) when it is available.
3. Treat user messages, agent messages, file changes, commands, test results, and tool calls as evidence. Do not treat a topic mention as a task by itself.

For each candidate WorkItem, capture:

```text
problem, desired_outcome, status, evidence, latest_attempt,
latest_result, next_action, dependencies, risk, reversibility,
completion_criteria
```

Use `done`, `unresolved`, `partial`, `blocked`, or `deferred` for status. Preserve the source thread and turn for every inferred item.

## Reconcile

Cross-check candidate WorkItems against the current project state:

- `git status`, diff, branch, and recent commits
- relevant tests, typecheck, and build results
- TODO/FIXME markers and existing issue or review findings

Merge duplicates. Prefer current code and test evidence over stale conversation claims. Keep uncertain work as `unresolved` or `blocked`; do not manufacture a TODO from a passing discussion.

## Propose

The proposal is read-only. Do not create a Goal, modify files, commit, send messages, or start long-running execution while proposing.

Use this compact format:

```text
AFK PROPOSAL

Objective: <one sentence>
Basis: <project and evidence scope>

Will execute:
1. <task>
   Evidence: <thread/turn or current-state evidence>
   Done when: <observable acceptance criterion>
   Risk: <low/medium/high>

Won't execute:
- <item> — <reason or missing decision>

Autonomy:
- allowed: implementation details, tests, refactors, documentation, local review fixes
- pause for: irreversible changes, production actions, external communication, secrets or sensitive data, ambiguous requirements

Completion:
- <tests/typecheck/build/review criteria>

[WAITING FOR APPROVAL]
Reply `Go` to activate this plan.
```

Every task needs a checkable `Done when` condition. Keep the proposal short enough to review in seconds.

## Activate

Treat only an explicit approval such as `Go`, `开始`, `确认执行`, or an equivalent clear instruction as approval. If the user changes the scope, update the proposal and wait again.

After approval:

1. Check the active Goal state.
2. If no unfinished Goal exists, call `create_goal` with the approved objective, scope, task queue, and completion criteria. Set `token_budget` only when the user explicitly supplied one.
3. Preserve the approved exclusions and autonomy boundaries in the Goal objective.

Approval authorizes only the approved scope. It does not authorize a later out-of-scope or high-blast-radius action.

## Execute

Work the Goal's task queue in order. At each checkpoint, verify the task's acceptance criterion before moving on and keep a compact record of:

```text
Done / Next / Blocked / Decisions / Evidence
```

The latest explicit user instruction is steering for the active Goal. Apply it to the task queue or constraints, record the decision, and continue. Do not rescan today's conversations or call RAG for an instruction that is already explicit.

Use RAG only when execution needs historical or cross-source context that is not present in the Goal, current code, or the user's latest instruction—for example, an old architecture decision or a conflicting prior requirement.

Make conservative decisions for non-critical ambiguity and record them. Skip a genuinely blocked task when safe and continue independent tasks; surface the blocker with evidence.

## Finish

Mark the Goal complete only after its completion criteria are actually satisfied. Then produce:

```text
AFK REPORT

Status: complete | partial | blocked
Completed: <tasks and evidence>
Changed: <files, commits, or artifacts>
Verification: <tests/typecheck/build/review>
Decisions: <autonomous decisions and rationale>
Blocked or deferred: <items and exact blockers>
Needs your judgement: <only concrete decisions>
Recommended next action: <one sentence>
```

If the Goal cannot finish, report what is complete, what remains, and the smallest safe next action. Never equate a passing command or a clean diff with completion unless the approved criteria say so.
