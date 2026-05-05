---
name: task-register
description: Create a new scratch task record for a task run without executing any task logic by itself. Used by `$start`, `$bundle`, and `$task-todo` to keep scratch registration consistent.
---

# Task Register

## Overview

Use this skill to create a scratch task record in a consistent format.

This is primarily a helper skill for other task-container skills:
- `$start`
- `$bundle`
- `$task-todo`

It may also be invoked directly when the user explicitly wants a scratch task registered without broader task execution.

## Scope

This skill operates only on:
- repo-root `tasks/`
- repo-root `scratch/`

## Shared Rules

- Every newly created scratch task must get a unique timestamp-based id.
- Use `YYYYMMDD-HHMMSS` in every scratch-task id.
- Do not create singleton scratch task paths that absorb later invocations.
- Registering a scratch task does not execute the target task procedure by itself.
- Registering a scratch task does not mutate canonical business entities by itself.

## Inputs

The caller must determine these pieces before registration:
- target kind:
  - template-backed task
  - one-off task idea
  - bundle parent task
- scratch path to create
- intended invocation text when applicable
- initial status
- initial plan / first actionable step
- source template path when applicable

## Scratch Record Contract

When creating a new scratch task, write a concise record that includes:
- `Status`
- `Created`
- source template path when the run is template-backed
- intended invocation text when available
- one-off task idea when the run is not template-backed
- a short initial plan or next step

Preferred status values:
- `in progress` for `$start`
- `Not started yet` for `$task-todo`
- a bundle-in-progress status for `$bundle` parent runs

## Behavior

1. Receive or derive the target scratch path and registration metadata from the caller.
2. Validate that the target scratch path is new and unique for this run.
3. Create the scratch task record immediately before any execution that depends on it.
4. Use the caller-provided initial status:
   - `$start` creates a scratch record and begins execution right away
   - `$task-todo` creates a scratch record but does not execute the task
   - `$bundle` creates one parent scratch record for the whole phased run
5. Keep the record concise but sufficient for `$continue` to list and resume it later.

## Guardrails

- Do not overwrite an existing unfinished scratch task as a shortcut.
- Do not silently reuse an older scratch task just because the task path or task idea looks similar.
- Do not execute the target task merely because the scratch record was created.

## Output

Create the scratch task record directly instead of replying with a plan.
