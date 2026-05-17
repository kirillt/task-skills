---
name: bundle
description: Start a bundled multi-task run where several tasks share one parent scratch task. Use when the user invokes `$bundle` with more than one task entry.
---

# Bundle

## Overview

Use this skill to execute several tasks as one phased run.

This skill operates directly on:
- task templates under repo-root `tasks/`
- scratch tasks under repo-root `scratch/`

Before executing the first phase, use `$task-register` to create the shared parent scratch task record.

## Shared Rules

- Resolve task paths relative to the repo root's `tasks/` directory, not relative to the skill folder.
- Reserved non-task folders such as `logs` and `data` are never valid task targets.
- Every newly created scratch task must get a unique timestamp-based id.
- Use `YYYYMMDD-HHMMSS` in every scratch-task id.
- Do not create singleton scratch task paths that absorb later invocations.

## Input

`$bundle` expects more than one task entry.

Each input task uses the same resolution rules as `$start`:
- it may resolve to a template task under repo-root `tasks/`
- or it may become a one-off task idea when it does not resolve to a template path

Task-entry separators:
- prefer one task per line after `$bundle`
- if a single-line form is needed, separate task entries with ` ;; `

If only one task entry is provided, stop with an error and tell the user to use `$start` instead.

## Behavior

1. Parse the bundle input into distinct task entries.
2. If fewer than two task entries are present, stop and ask the user to use `$start` for a single task.
3. Create exactly one new parent scratch task with a unique timestamp-based id via `$task-register`.
4. Record each input task as its own phase inside that one scratch task.
5. For each phase:
   - apply the same template-resolution or one-off initialization rules as `$start`
   - the phase may itself be a batch-oriented task that uses `$batch` internally
   - do not create a second scratch task for the individual phase
   - store the phase plan and status inside the parent scratch task
6. Execute the phases in order.
7. If a phase blocks or needs user review, stop with the parent scratch task updated to the current phase and blocker state.

## Guardrails

- `$bundle` always uses one parent scratch task for the whole run.
- Each phase is recorded inside that parent scratch task, not as a standalone scratch task.
- A bundled phase may itself use `$batch`; in that case the parent bundle scratch tracks phase-level progress while the phase follows its own batch-table rules.
- `$bundle` must not silently accept a single-task input.

## Output

Execute the phased run directly instead of replying with a plan.
