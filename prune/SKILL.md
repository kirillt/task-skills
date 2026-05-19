---
name: prune
description: Prune completed scratch tasks, or preview what would be deleted. Use when the user invokes `$prune` to clean the scratch buffer safely.
---

# Prune

## Overview

Use this skill to remove completed scratch tasks safely.

This skill operates only on repo-root `scratch/`.

## Input

`--dry-run` is optional.

- Default behavior with empty input: delete completed scratch tasks immediately.
- `--dry-run`: show what would be deleted, but do not delete anything.
- No other input is valid unless this skill is explicitly extended later.

## Behavior

1. Inventory everything under `scratch/` recursively.
2. Classify scratch task files by status.
3. Safe-to-prune criterion:
   - only scratch tasks with `Completed` status are safe to prune automatically
4. Produce a Keep/Delete plan every time.
5. Present the Keep/Delete plan by passing one Markdown table to `$display-table` with exactly two columns:
   - `Keep`
   - `Delete`
6. If `--dry-run` is present:
   - do not delete anything
   - report what would be deleted
7. Otherwise:
   - delete only completed scratch tasks
   - report what was deleted
8. Also output the top 3-5 unfinished scratch follow-ups to run next, ranked by time-to-progress and unblock impact.

## Guardrails

- `$prune` does not execute pending scratch follow-ups.
- Do not delete unfinished, blocked, ambiguous, or malformed scratch tasks automatically.

## Output

Prune directly instead of replying with a plan.
