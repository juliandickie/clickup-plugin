---
name: clickup-task-search
description: Find and summarise ClickUp tasks in a list, space, or across the whole workspace. Use when the user asks to "find tasks", "search ClickUp", "what tasks are in list X", "everything assigned to Y", or wants a filtered task overview.
---

# ClickUp Task Search

Find tasks and present a compact summary. Never dump raw API JSON at the user.

## Inputs (from the user)

- A scope. One list - resolve its id via list_workspaces, list_spaces, list_folders, list_lists if the user is vague. Cross-list ("everything assigned to X", "all urgent tasks") - the workspace id is enough.

- Optional filters - status, assignee, tag, date window, text in name.

## Steps

1. Pick the right read for the scope. One known list - list_tasks with the list id. Anything spanning lists, or filtered by status/assignee/tag/date - filter_workspace_tasks with those filters server-side. Do not set include_field_schema on either (the default strip keeps the response small).

2. Assignee filters need user ids - resolve a name or email through list_members once, not by guessing.

3. Filter client-side only for what the API cannot express (text in name).

4. Output a markdown table - Task, Status, Assignees, Updated. Add a one-line count summary above it.

## Failure modes

- No list resolved - ask the user which list, listing the candidates you found.

- More than 500 tasks - summarise counts by status and ask the user to narrow before listing individually.
