# ClickUp for Claude Code

Talk to ClickUp from Claude Code with token-disciplined task access and first-class bulk operations. This plugin exists because the stock ClickUp MCP connector has two practical problems - it inlines a full custom-field option schema on every task (tens of thousands of redundant tokens on a real list) and it has no real bulk surface. This plugin fixes both - it strips per-task field schemas by default and ships gated, audited bulk operations plus a separate runner for very large idempotent jobs.

Works with any ClickUp workspace. You supply your own personal token.

## Install

Standalone, just this plugin:

```
/plugin marketplace add juliandickie/clickup-plugin
/plugin install clickup@clickup
```

Or, if you use the aggregate outfit catalog, the plugin is also listed there and installs the same way with `clickup@outfit`.

## Authentication

You need a ClickUp personal API token. Create one in ClickUp at Settings - Apps - Generate API Token. It starts with `pk_`. The plugin uses it read-write against your own account, so it can do anything you can do in the ClickUp UI.

Supply the token in any one of these ways. The plugin resolves them in this order.

1. The plugin config prompt - on install Claude Code asks for your ClickUp API token and stores it as a sensitive plugin setting.

2. The `CLICKUP_API_TOKEN` environment variable - set it to the `pk_` token directly, or to a 1Password secret reference like `op://Vault/ClickUp/credential` (requires the `op` CLI installed and signed in).

3. A machine-global file at `~/.config/clickup-plugin/config.toml` containing:

```toml
api_token = "pk_your_token_here"
```

The token is never written to any repository, never logged, and never sent anywhere except `api.clickup.com`. Every person who uses the plugin supplies their own.

## What you get

Skills (these activate on their own when relevant).

- `clickup-task-search` - find and summarise tasks in a list or space.

- `clickup-list-audit` - a canonical snapshot of a list - counts by status, custom-field coverage, relationship density.

- `clickup-bulk-plan` - plan a bulk change and preview it. This skill only ever plans. It never writes.

Commands (explicit, confirmed writes).

- `/clickup-task` - create or update a single task, after showing you a confirmation summary and waiting for your yes.

- `/clickup-bulk-apply` - apply a previously previewed bulk change to up to 10 tasks, after explicit confirmation. Larger sets use the bundled clickup-batch runner. Bulk deletes require a second, separate confirmation.

MCP tools (Claude calls these as needed, with the safety rules below).

- Navigation - `list_workspaces`, `list_spaces`, `list_folders`, `list_lists`, `get_list`.

- Read - `list_tasks` (auto-paginated, schema-stripped by default), `get_task`, `list_custom_fields`.

- Single writes (gated) - `create_task`, `update_task`, `delete_task`, `set_custom_field`, `add_comment`, `set_task_relationship`.

- Bulk (gated, capped) - `bulk_update_tasks`.

## Token discipline - the core difference

ClickUp returns every custom field's full option list (`type_config`) on every task. On a list with dozens of options across hundreds of tasks that is tens of thousands of redundant tokens per pull. `list_tasks` and `get_task` strip `type_config` by default. When you genuinely need the option catalogue, call `list_custom_fields` once for the list, or pass `include_field_schema: true` on a specific call. This is the single biggest reason to use this plugin over the stock connector.

## Safety model

Every write is gated. The write tools refuse to run unless invoked through a `/clickup-*` command that has shown you a confirmation summary and received an explicit yes. There is no path to a silent write.

- Bulk operations via the MCP tool are recommended for up to 10 tasks. Sets larger than 10 use the bundled clickup-batch runner. The MCP tool still hard-refuses more than 100 as a safety backstop.

- A bulk delete needs a second explicit confirmation in addition to the normal one. A 100-task delete cannot happen by accident.

- Set `CLICKUP_DRY_RUN=1` to block all writes globally. In dry-run, write tools return a clearly labelled synthetic response and never touch ClickUp. Reads still work.

- Every mutating call is appended to a local audit log at `~/.local/share/clickup-plugin/audit.log` with a timestamp, the operator, the operation, your confirmation summary, and the affected ids. The audit write is best-effort and never aborts your operation.

## Large or idempotent jobs

`bulk_update_tasks` is recommended for sets of up to 10 tasks. For larger sets, or any job you want to run idempotently with a dry-run preview and a written report, use the `clickup-batch` runner. It ships with this plugin (installed as `clickup-batch` on PATH). It is dry-run by default, replaces a marker block in place so re-runs do not duplicate, and paces writes under ClickUp's rate limit. The MCP bulk tool hard-refuses more than 100 tasks as a safety backstop, but the recommended path for more than 10 tasks is the runner.

## Requirements

- Node 18 or newer (the bundled MCP server runs on it).

- A ClickUp personal token (`pk_`).

## Optional configuration

- `CLICKUP_BASE_URL` - override the API base (defaults to `https://api.clickup.com/api/v2`). An unresolved `${...}` value is ignored and the default is used.

- `CLICKUP_DRY_RUN` - set to `1` to block all writes.

## Notes

This repository is generated. It is the lean, installable payload built and published from a separate development monorepo. Do not send pull requests here - see CONTRIBUTING.md. Day-to-day usage, examples, and troubleshooting are in USAGE.md.

## License

MIT.
