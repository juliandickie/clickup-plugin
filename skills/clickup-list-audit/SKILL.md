---
name: clickup-list-audit
description: Snapshot a ClickUp list - task counts by status, custom-field coverage, and relationship density. Use when the user says "audit this list", "list health", or "what's the state of list X".
---

# ClickUp List Audit

Produce a canonical audit so repeated runs are comparable.

## Steps

1. Resolve the list id (navigation tools if needed).

2. Call list_custom_fields once for the field catalogue (this is where option schemas belong).

3. Call list_tasks (default strip).

4. Compute - total tasks, count by status, percent of tasks with each custom field populated, count with dependencies or linked tasks.

## Output format (verbatim structure)

```markdown
# ClickUp List Audit - {list name}

Generated {YYYY-MM-DD HH:MM UTC}

## Summary

- Total tasks - {N}
- By status - {status: count, ...}
- Custom-field coverage - {field: percent, ...}
- Tasks with relationships - {N}

## Notes
- {caveats}
```
