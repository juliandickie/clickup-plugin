---
description: Create or update a single ClickUp task after showing a confirmation summary.
argument-hint: <create|update> [details]
---

You are executing the /clickup-task slash command. Create or update one
ClickUp task on the user's behalf, with explicit confirmation before the write.

## Arguments

The user invoked: `/clickup-task $ARGUMENTS`

First token is the action - `create` or `update`. Remaining text is the
details (task name, target list or task id, fields). Ask for anything missing.

## Workflow

1. For create - resolve the target list id (use the navigation MCP tools if
   the user named a list rather than gave an id). For update - confirm the
   task id exists with get_task.

2. Show the confirmation summary and WAIT for an explicit yes:

   > About to {create|update}:
   > {field: value lines}
   >
   > Proceed? (y/n)

3. On yes, call create_task or update_task with confirmed_by_user: true and
   a confirmation_summary that matches exactly what you showed.

4. Report - "Done - task {id}." On no - "Cancelled. No change made." and stop.

## Safety rules

- Never call the write tool without showing the summary and getting yes.
- Never set confirmed_by_user false. Never fabricate confirmation_summary.
