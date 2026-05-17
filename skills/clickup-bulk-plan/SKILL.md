---
name: clickup-bulk-plan
description: Plan a ClickUp bulk change and preview it before any write. Use when the user asks to "bulk update", "set status on all tasks matching", "mass change ClickUp tasks", or any multi-task mutation.
---

# ClickUp Bulk Plan

This skill ONLY plans and previews. It never writes. Writing is the
/clickup-bulk-apply command after the user confirms.

## Steps

1. Resolve the target list and the filter that selects the tasks.

2. Call list_tasks, filter client-side to the target set, and show the user - the exact task count, the operation, and a sample of 10 affected task names.

3. State the routing decision explicitly -

   - 10 or fewer tasks - the /clickup-bulk-apply command will use the bulk_update_tasks MCP tool.

   - More than 10 tasks - instruct the user to run the bundled clickup-batch runner (give the exact command, default dry-run). clickup-batch ships with the plugin and is on PATH. The MCP bulk tool still hard-refuses more than 100 as a safety backstop, but the recommended path for more than 10 is the runner.

4. Stop. Hand off to the user to invoke /clickup-bulk-apply (small) or clickup-batch (large). Do not call any write tool yourself.

## Safety

- Always show the count and a sample before recommending apply.

- Never call bulk_update_tasks, create_task, update_task, delete_task, or set_custom_field from this skill.
