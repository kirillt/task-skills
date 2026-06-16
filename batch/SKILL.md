---
name: batch
description: Run one batch-oriented task over a collection inside a single scratch task. Use when a workflow processes a collection or dynamically generated item set in repeated fixed-size batches, starts with a review table gate, and normally waits for user approval before continuing.
---

# Batch

## Overview

Use this skill for one task that iterates a collection in repeated review gates.

This is not the same as `$bundle`.
- `$bundle` runs several different tasks as phases.
- `$batch` runs one task over many items or dynamically generated work units.

This skill operates directly on:
- task templates under repo-root `tasks/`
- scratch tasks under repo-root `scratch/`

Before executing the first batch, use `$task-register` to create the batch scratch task record.

## Shared Rules

- Resolve task paths relative to the repo root's `tasks/` directory, not relative to the skill folder.
- Reserved non-task folders such as `logs` and `data` are never valid task targets.
- Every newly created scratch task must get a unique timestamp-based id.
- Use `YYYYMMDD-HHMMSS` in every scratch-task id.
- Do not create singleton scratch task paths that absorb later invocations.

## When To Use

Use this skill when:
- the task is one logical run over a collection of items
- the collection may be explicit up front or generated progressively from a stable rule
- the work should proceed in repeated fixed-size batches
- the user must see a first batch table before batch processing can continue,
  with optional fast-mode continuation after that when explicitly enabled

Do not use this skill just because a task touches multiple records once. Use it only when the control flow itself is batch-oriented.

## Inputs

The caller must determine these pieces before invoking the batch loop:
- the batch task's scratch path
- the source collection or stable selection rule
- the current batch size
- the item identity scheme
- the batch table schema
- the proposal-generation logic for the current domain
- the approval behavior for the run

The batch size may be changed by the user between batches.

## Scratch Record Contract

In addition to normal scratch-task fields, a batch task must record enough state to resume exactly:
- collection source or generation rule
- current batch size
- review mode: normal or fast
- whether fast mode has already been offered, accepted, or declined for this
  run
- the exact fast-mode offer text shown with the first batch table
- processed items
- unreviewed items or the remaining selection rule
- scheduled items when applicable
- pending items in the current batch, including any unresolved items carried
  forward from an earlier batch by explicit user decision
- the latest batch shown to the user
- the approval / correction / rejection outcome for that latest batch
- the next actionable step

## Terminology

- Overall collection: the full item set or stable generation rule for the batch run.
- Processed items: items from the overall collection that have received a
  final disposition in the batch run, such as applied, rejected, skipped, or
  otherwise resolved by the domain workflow.
- Unreviewed items: items from the overall collection, or items that can still
  be produced by the stable generation rule, that have not yet been presented
  to the user in a batch table.
- Scheduled items: optional term for a known fixed item sequence in tasks where
  the future item set is fully known. This term applies only to some batch
  tasks; do not use it for progressively generated item sets.
- Current batch: the up-to-batch-size set of unreviewed items currently shown
  to the user for approval.
- Normal mode: the default review mode for every new batch run. In normal
  mode, show the batch table and wait for user approval or corrections before
  applying each batch.
- Fast mode: optional review mode for the current batch run. In fast mode, a
  domain task may skip the main table for batches whose rows are all
  non-blocking and may apply those rows and proceed to later batches in the
  same run. Fast mode must not hide pending items, unresolved rows, or
  domain-specific decisions that require user input. Fast mode does not relax
  completion rules: a scratch task with any pending items must remain
  unfinished.
- Pending items: items in the current batch that are waiting for the user's
  approval, correction, rejection, or another domain-specific decision.
  Domain tasks whose table rows are derived from input items may define
  pending output rows, but must record which input item each pending output
  row came from.
- Current batch rows: the table rows or proposals for the current batch. These
  rows may be approved, corrected, rejected, or blocked by domain-specific
  rules.
- Pending output rows: unresolved current-batch rows produced from input
  items in tasks where one input item may produce zero, one, or many output
  rows. They are pending items for approval-loop purposes and must be carried
  forward with source input item linkage and the exact decision still needed.

## Behavior

1. Create or resume exactly one scratch task for the whole batch run via `$task-register` when needed.
2. When starting a new batch run, report in chat before the first batch table:
   - what the current batch size is
   - how many items are in scope for the whole task when that total is already known
   - if the total is not yet known, it is allowed to omit it at this point
3. Materialize the current batch from the unreviewed item set or from the stable generation rule.
   - if retrieving the first batch makes the total item count knowable, report that total to the user before or alongside the first batch table
   - if the run uses scheduled items for a known fixed item sequence, show
     progress as `K/N`, where `N` is the total number of scheduled items and
     `K` is the number of processed items
4. Prepare or evaluate only up to the current batch size.
5. Generate proposals for that batch according to the domain workflow.
6. In normal mode, pass the batch table to `$display-table` for that batch.
   - The first batch table is mandatory.
   - The table must be shown even in auto-approve mode.
   - When a batch table is shown, it must include every current batch row as a
     separate row. Do not group, collapse, summarize, or replace table rows
     with ranges such as `15-30`, even when rows are repetitive or obvious.
     Boring rows may be omitted only by omitting the entire main table through
     an explicitly enabled fast-mode rule.
   - Together with the first batch table, offer fast mode once for the current
     run and record the exact offer text in scratch.
7. Default behavior:
   - stop after showing the batch table
   - treat the current batch items as pending items
   - wait for the user's approval, corrections, or rejections before applying
     the batch
8. If the run is in explicit auto-approve mode:
   - still show the batch table first
   - then apply the batch without waiting only if the domain rules allow it
9. Treat fast mode as enabled only when the user's reply explicitly accepts
   it. A plain batch confirmation without an explicit fast-mode mention counts
   only as confirmation of the current batch, not as fast-mode acceptance. The
   user must write at least `fast` or explicitly quote the fast-mode offer to
   accept fast mode. If the user explicitly declines fast mode, record the
   decline and do not offer fast mode again in the same run.
10. After approval, corrections, or rejections:
   - record the decision in the same scratch task
   - apply approved current-batch rows according to the domain workflow
   - leave corrected/rejected current-batch rows with explicit disposition
   - mark current-batch items with a final disposition as processed
   - keep any still-unresolved current-batch items as pending items with the
     exact decision needed from the user
11. If the user explicitly approves the current batch while leaving some items
    unresolved, carry those items forward as pending items instead of blocking
    the whole batch. Record the exact decision still needed for each carried
    pending item.
12. When carried pending items exist and the run proceeds to another batch,
    show a separate pending table before the main next-batch table. The pending
    table must show the unresolved items and the exact decision still needed.
13. In fast mode:
   - materialize and evaluate each next batch normally
   - if a batch has no pending items, no blocking rows, and no domain-specific
     user decision needed, the domain task may omit the main chat table, apply
     the batch, record the omitted table or equivalent candidate state in
     scratch, summarize the applied result briefly, and immediately proceed to
     the next batch
   - if carried pending items exist, show the pending table before continuing
   - if any row needs approval, correction, rejection, duplicate resolution,
     cancellation resolution, missing-information resolution, or another
     domain-specific decision, show the batch table and stop for user input
   - do not mark the scratch task completed while any pending items remain
14. After the current batch's final decisions are applied or recorded, if
    unreviewed items remain in the overall batch collection, immediately
    proceed to the next batch. The next batch follows the same table-first
    approval gate unless fast mode is enabled and the next batch qualifies for
    fast-mode processing. It must include the pending table first when carried
    pending items exist.
15. Do not mark the scratch task completed while any pending items remain,
    including carried pending items.
16. If no pending items remain, no unreviewed items remain in the overall
    collection, and all final-disposition work has been applied or recorded,
    mark the scratch task completed.

## Guardrails

- One batch run always uses one scratch task.
- Do not create a second scratch task for later batches of the same run.
- Do not start a new batch run without first telling the user the current batch size.
- If the total item count is known at run start, tell the user then.
- If the total is not known at run start but becomes known while retrieving the first batch, tell the user before or alongside the first batch table display.
- Do not skip the first batch table.
- Do not skip any table in normal mode.
- Do not collapse rows inside a shown batch table. Shown batch tables must be
  complete row-for-row; use fast mode to omit a whole eligible table instead of
  shortening it.
- Do not use fast mode before the user explicitly enables it for the current
  run.
- If the user declines fast mode, do not offer it again in that same scratch
  run.
- Do not use fast mode to hide pending items or unresolved/blocking rows.
- Do not mark items as processed before the batch decision is recorded.
- Do not mark a batch scratch task completed while pending items remain.
- Do not silently drop pending items when moving to another batch; carry them
  forward only after explicit user approval, and show them before the next
  batch table.
- Do not ask whether to continue between resolved batches when unreviewed
  items remain in the overall collection; proceed to the next batch table in
  normal mode, or to the next fast-mode evaluation when fast mode is enabled.
- Auto-approve is not the standard mode; standard mode is show the batch table and wait.
- Domain-specific skills still own item evaluation, table columns, and write-side consequences; `$batch` owns only the generic iteration and approval-loop structure.

## Output

Execute the batch step directly instead of replying with a plan.
