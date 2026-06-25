---
name: subagent-router
description: Use when a task may benefit from subagents, isolated context, delegation, parallel search, worker agents, explorer agents, test agents, review agents, multi-file changes, unrelated failures, or when the user says go on and expects end-to-end execution.
---

# Subagent Router

## Core Rule

Treat this skill as the user's standing preference to keep the main session concise by delegating suitable work to subagents. Before doing context-heavy search, multi-file edits, test triage, or review inline, decide whether a subagent can own a bounded slice.

## Dispatch Gate

Spawn a subagent when at least one condition is true:

- The work splits into two or more independent questions or file scopes.
- A codebase search will read many files but only a concise answer is needed.
- A worker can make a bounded patch in a disjoint file set.
- Tests, lint, build, or review can run while the main session continues non-overlapping work.
- Failures are grouped by unrelated files, subsystems, or root causes.
- The user says `go on` and the next step includes implementation plus verification.

Do not spawn when the task is a single small local edit, a design question needing user judgment, a shared-state change where agents would edit the same files, or a high-risk config/secrets/global-harness task unless the user explicitly authorizes that scope.

## Roles

Use `explorer` for read-only codebase questions: where something is defined, which call sites matter, what tests cover a path, or what config is actually loaded.

Use `worker` for execution: implementing a bounded change, fixing a specific test file, running focused verification, or producing a patch in an assigned file set.

Use one subagent per independent domain. Prefer fewer, sharper agents over broad delegation.

## Prompt Contract

Every subagent prompt must include:

- Goal: the exact result needed.
- Scope: files, directories, commands, or question boundaries.
- Ownership: which files it may edit, or that it is read-only.
- Constraints: do not revert unrelated changes; work with concurrent edits; avoid global config unless assigned.
- Return shape: conclusion, changed paths, tests/commands run, failures, and risks.

For coding work, tell workers to edit directly in their assigned workspace/session and list changed paths in the final answer.

## Main Session Behavior

After dispatching, continue with non-overlapping work. Wait only when the result blocks the next critical decision. When a subagent returns, integrate the result by reading changed files or summaries, then run final verification in the main session before claiming completion.

Keep only decision-grade context in the main session:

- What was delegated
- What came back
- What changed
- What was verified
- What remains risky

## Red Flags

Stop and route before continuing inline when these thoughts appear:

| Thought | Correct action |
| --- | --- |
| "I'll just grep/read a lot quickly." | Use an explorer if only the answer matters. |
| "These failures look unrelated." | Split by test file or subsystem. |
| "A worker could do this while I inspect something else." | Dispatch the worker and continue. |
| "The prompt would be vague." | Narrow scope first; do not dispatch vague work. |
| "Two agents may touch the same files." | Do not parallelize that slice. |

## Completion Check

Before final response, confirm:

- All spawned agents are closed or no longer needed.
- Their outputs were reviewed, not blindly trusted.
- Final verification was run or the reason it could not run is stated.
- The final summary stays concise and does not paste subagent process logs.
