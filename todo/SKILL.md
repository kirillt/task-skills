---
name: task-todo
description: Create a scratch task for later execution without starting it yet. Use when the user invokes `$task-todo` to queue a task now and continue it later.
---

# Todo

## Overview

Use this skill to register a task for later work without executing it now.

This skill registers a task for later execution without starting it now.

This skill operates directly on:
- task templates under repo-root `tasks/`
- scratch tasks under repo-root `scratch/`

Use `$task-register` to create the scratch task record.

## Shared Rules

- Resolve task paths relative to the repo root's `tasks/` directory, not relative to the skill folder.
- Reserved non-task folders such as `logs` and `data` are never valid task targets.
- Every newly created scratch task must get a unique timestamp-based id.
- Use `YYYYMMDD-HHMMSS` in every scratch-task id.
- Do not create singleton scratch task paths that absorb later invocations.

## Input

The exact remainder after `$task-todo` is required.

- Empty input is invalid.
- First try to resolve the first token as a task path under repo-root `tasks/`.
- Task paths may use `\` for nesting, such as `companies\refresh`.
- The `.md` extension may be omitted and derived implicitly.
- If the first token resolves to a task folder or concrete template task, treat the remaining text as that task's queued invocation arguments.
- If the input does not resolve to a task path, treat the entire input as a one-off task idea and register it directly as a unique scratch task.

## Behavior

1. If the input is empty, stop and ask the user to provide a task path or one-off task idea.
2. Resolve the first token against repo-root `tasks/` using the same nesting convention as task commands.
3. If the target resolves to a folder:
   - list tasks in that folder briefly
   - do not create scratch
   - stop
4. If the target resolves to a concrete template task:
   - read that task template
   - create a new scratch task with a unique timestamp-based id via `$task-register`
   - write the target template path, intended invocation, queued arguments, and a short first-step plan into that scratch task
   - set `Status` to `Not started yet`
   - do not execute the target task
5. If the input does not resolve to a task path:
   - normalize the full input into a short human-readable slug prefix that summarizes the task idea
   - create a unique scratch task id as `<slug>-<YYYYMMDD-HHMMSS>`
   - initialize `scratch/<task_id>.md` via `$task-register`
   - write the one-off task idea and an initial plan into that scratch task
   - set `Status` to `Not started yet`
   - do not execute the task idea
6. Report how to pick the task up later:
   - `$continue` will list it among unfinished tasks
   - an explicit `$continue <id-or-prefix>` may be used to resume it directly

## Guardrails

- Do not execute the target task procedure in this skill.
- Do not mutate canonical business entities as part of merely creating the todo scratch task.
- Do not reuse or overwrite an existing unfinished scratch task as a shortcut.

## Output

Create the scratch task directly instead of replying with a plan.
