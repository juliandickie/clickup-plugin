---
name: clickup-task-search
description: Find and summarise ClickUp tasks in a list or space. Use when the user asks to "find tasks", "search ClickUp", "what tasks are in list X", or wants a filtered task overview.
---

# ClickUp Task Search

Find tasks and present a compact summary. Never dump raw API JSON at the user.

## Inputs (from the user)

- A list id, or enough navigation context (workspace, space, folder) to resolve one. Use list_workspaces, list_spaces, list_folders, list_lists to drill down if the user is vague.

- Optional filters - status, assignee, text in name.

## Steps

1. Resolve the target list id via the navigation tools.

2. Call list_tasks with the list id. Do not set include_field_schema (the default strip keeps the response small).

3. Filter client-side to what the user asked for.

4. Output a markdown table - Task, Status, Assignees, Updated. Add a one-line count summary above it.

## Failure modes

- No list resolved - ask the user which list, listing the candidates you found.

- More than 500 tasks - summarise counts by status and ask the user to narrow before listing individually.
