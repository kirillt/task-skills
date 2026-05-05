---
name: start
description: Start a repository task template or a one-off scratch task. Use when the user invokes `$start` to begin one concrete task now; if the input contains multiple task entries, automatically convert into `$bundle`.
---

# Start

## Overview

Use this skill to begin one task run.

This skill operates directly on:
- task templates under repo-root `tasks/`
- scratch tasks under repo-root `scratch/`

Before executing a started task, use `$task-register` to create the scratch task record.

## Shared Rules

- Resolve task paths relative to the repo root's `tasks/` directory, not relative to the skill folder.
- Reserved non-task folders such as `logs` and `data` are never valid task targets.
- Every newly created scratch task must get a unique timestamp-based id.
- Use `YYYYMMDD-HHMMSS` in every scratch-task id.
- Do not create singleton scratch task paths that absorb later invocations.

## Input

The exact remainder after `$start` is required.

- Empty input is invalid.
- First try to resolve the first token as a task path under repo-root `tasks/`.
- Task paths may use `\` for nesting, such as `companies\cache`.
- The `.md` extension may be omitted and derived implicitly.
- If the first token resolves to a task folder or concrete template task, treat the remaining text as that task's invocation arguments.
- If the input does not resolve to a task path, treat the entire input as a one-off task idea and initialize it directly as a unique scratch task.
- If the input clearly contains more than one task entry, automatically convert into `$bundle` and say so explicitly in chat before proceeding.
- Multi-task separators follow the same conventions as `$bundle`:
  - prefer one task per line
  - if a single-line form is needed, separate task entries with ` ;; `

## Behavior

1. If the input is empty, stop and ask the user to provide a task path or one-off task idea.
2. Resolve the first token against repo-root `tasks/` using the same nesting convention as task commands.
3. If more than one task entry is present:
   - switch to `$bundle` automatically
   - tell the user explicitly that `$start` detected multiple tasks and is continuing as `$bundle`
4. If the target resolves to a folder:
   - list tasks in that folder briefly
   - do not create scratch
   - stop
5. If the target resolves to a concrete template task:
   - read that task template
   - create a new scratch task with a unique timestamp-based id via `$task-register` before substantive work begins
   - write the concrete execution plan, current status, source template path, and any task arguments into that scratch task
   - execute the target task template directly after scratch setup, following its own procedure, batching, and output rules
6. If the input does not resolve to a task path:
   - normalize the full input into a short human-readable slug prefix that summarizes the task idea
   - create a unique scratch task id as `<slug>-<YYYYMMDD-HHMMSS>`
   - initialize `scratch/<task_id>.md` immediately via `$task-register`
   - write the one-off task idea, a concrete initial plan, and current status into that scratch task
   - continue the one-off task from that scratch state in the same invocation when feasible, otherwise stop with the initialized scratch task recorded

## Guardrails

- Do not treat `$start` as a hardcoded cache-hydration command.
- Do not create `scratch/start.md`; scratch belongs to the target task, not to this skill.
- One-off tasks must never be mistaken for template-backed tasks once resolution fails.
- `$start` never auto-resumes an older scratch task by task name or task path.

## Output

Execute the task directly instead of replying with a plan.
