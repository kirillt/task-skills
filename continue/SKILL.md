---
name: continue
description: Continue a scratch task by exact path or id prefix. Use when the user invokes `$continue` to resume an unfinished task.
---

# Continue

## Overview

Use this skill to resume an existing scratch task.

This skill operates only on repo-root `scratch/`.

## Shared Rules

- Resolve scratch paths relative to the repo root's `scratch/` directory, not relative to the skill folder.
- In `$continue` mode, only the scratch task id and the scratch task's recorded state are authoritative.
- Do not consult task templates at all in continuation mode.

## Input

An optional relative path or id prefix under `scratch/`.

- It may use `\` for nesting, such as `companies\cache\20260503-120000`
- It may resolve to a folder or to a concrete scratch task file
- It may also be an ambiguous prefix such as `hello-world`

If no input is provided, treat that as the broadest ambiguous case: discover unfinished scratch tasks from the whole repo-root `scratch/` tree, list them, and ask the user which one to continue.

## Behavior

1. Build the candidate set from unfinished scratch tasks only.
2. If input is provided:
   - if `scratch/<path>/` exists, list tasks in that folder briefly and stop
   - else if `scratch/<path>.md` exists or the path already points to a `.md` scratch file, continue only that exact scratch task
   - else treat the input as a prefix filter over scratch-task ids
3. If no input is provided:
   - enumerate `scratch/**/*.md` recursively
   - exclude `scratch/data/**`
   - filter to unfinished scratch tasks only:
     - terminal statuses are `completed`, `canceled`, and `superseded`
     - `Not started yet` counts as unfinished and should be listed as a valid continuation target
     - malformed files with no readable status are not auto-continued
4. If a prefix input was provided:
   - collect all unfinished scratch tasks whose id starts with that prefix
   - if there is no match, stop and ask what was intended
   - do not continue automatically from a prefix match; present the chooser table and require an explicit selection
5. If zero unfinished scratch tasks exist, report that there is nothing to continue and stop.
6. If the current input is empty or ambiguous, present a chooser table:
   - assemble a compact Markdown table with exactly two columns and pass it to `$display-table`:
     - `Scratch task`
     - `Description`
   - in `Scratch task`, use the basename when unambiguous, otherwise the path relative to `scratch/`
   - in `Description`, summarize each task in at most 100 characters from its scratch state
   - include enough context to distinguish `Not started yet` items from actively in-progress ones
   - ask the user which one to continue
   - stop
7. If an exact scratch-task path was provided and resolved, continue only that task.
8. When continuing one scratch task:
   - read the scratch task first
   - identify the next actionable step from the scratch task's recorded state
   - progress strictly from the scratch task id and its recorded state
   - store bulky shared artifacts under `scratch/data/` only when needed and reference them from the scratch task
9. If incomplete, update the same scratch task with current status, completed work, next steps, and exact blockers.
10. If complete, note that the scratch task is completed.

## Guardrails

- Never treat `$continue` as casual prose.
- Do not consult task templates at all in continuation mode.

## Output

Continue the task directly instead of replying with a plan.
