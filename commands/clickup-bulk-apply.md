---
description: Apply a previewed ClickUp bulk change to up to 100 tasks after explicit confirmation.
argument-hint: (run clickup-bulk-plan first)
---

You are executing the /clickup-bulk-apply slash command. This applies a bulk
change that was already planned and previewed by the clickup-bulk-plan skill.

## Preconditions

- A bulk plan must already have been shown to the user in this conversation
  (task list resolved, operation defined, count known). If not, tell the user
  to run the bulk-plan skill first and stop.

- The affected set must be 100 tasks or fewer. If larger, tell the user to use
  the clickup-batch runner instead (give the exact command) and stop.

## Workflow

1. Restate the exact change - operation, affected task count, list.

2. Show the confirmation summary and WAIT for an explicit yes:

   > About to apply {operation} to {N} tasks in list {list}.
   > This is destructive and visible to the whole workspace.
   >
   > Proceed? (y/n)

3. If (and only if) the operation kind is delete, this is a mass destructive
   delete. Require a SECOND explicit confirmation, separate from step 2:

   > This permanently DELETES {N} tasks and cannot be undone.
   > Type DELETE to confirm, or anything else to cancel.

   Proceed only if the user replies with an explicit destructive confirmation.

4. On yes (and, for a delete, the second confirmation in step 3), call
   bulk_update_tasks with the task_ids, the operation, confirmed_by_user: true,
   and a confirmation_summary matching what you showed. For a delete, the
   operation object must also include delete_confirmed: true (the MCP tool
   rejects a bulk delete without it).

5. Report the updated and failed counts from the tool response. If any failed,
   list the failing task ids and their errors.

6. On no (or a declined delete confirmation) - "Cancelled. No tasks changed."
   and stop.

## Safety rules

- Never call bulk_update_tasks without an explicit yes to the summary.

- For a delete operation, never call bulk_update_tasks without BOTH the step 2 yes AND the separate step 3 destructive confirmation, and the operation must carry delete_confirmed: true.

- Never exceed 100 task ids - route larger jobs to clickup-batch.

- Never fabricate confirmation_summary - derive it from what you showed.
