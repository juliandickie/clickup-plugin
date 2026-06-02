<p align="center">
  <img src="assets/hero.png" alt="ClickUp for Claude Code" width="100%">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Claude%20Code-plugin-8A2BE2" alt="Claude Code plugin">
  <img src="https://img.shields.io/badge/ClickUp%20API-v2-7B68EE" alt="ClickUp API v2">
  <img src="https://img.shields.io/badge/node-%3E%3D18-3C873A" alt="Node 18 or newer">
  <img src="https://img.shields.io/badge/license-MIT-success" alt="MIT license">
</p>

<p align="center">
  <b>Talk to ClickUp from Claude Code with token-disciplined task access and first-class, audited bulk operations.</b>
</p>

# ClickUp for Claude Code

You run real work in ClickUp, and you drive Claude Code from the terminal. The two should talk to each other. The stock ClickUp connector makes that expensive and nerve-wracking on a real workspace, in two specific ways.

First, it is wasteful. It inlines the full custom-field option schema (`type_config`) on every single task it returns. On a list with dozens of options across hundreds of tasks, that is tens of thousands of redundant tokens on a single pull, before you have asked a single question.

Second, it is not safe to scale. There is no real bulk surface, so a mass change is either a hundred careful single calls or a leap of faith. And the moment a tool can touch a hundred tasks at once, the fear is rational. One bad call could reassign a sprint or delete a quarter of a board, and you would have no record of what happened.

A tool you are afraid to run at scale is not really a tool. This plugin was built for exactly that tension. It strips the schema bloat by default, and it puts a hard, explicit confirmation in front of every write, with a capped bulk path, a dry-run kill switch, and a local audit log of every mutation. You get ClickUp control from Claude Code that is fast, cheap, and auditable enough to actually trust.

The plan is three steps - add the marketplace, install the plugin, paste your own ClickUp token. Then ask Claude to list a workspace and you are working.

Works with any ClickUp workspace. You supply your own personal token, and every person who uses the plugin supplies their own.

## Install

Standalone, just this plugin.

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

The token is never written to any repository, never logged, and never sent anywhere except `api.clickup.com`.

## Token discipline - the core difference

<p align="center">
  <img src="assets/token-discipline.png" alt="Token discipline - schemas stripped by default, fetched once on demand" width="100%">
</p>

ClickUp returns every custom field's full option list (`type_config`) on every task. On a list with dozens of options across hundreds of tasks that is tens of thousands of redundant tokens per pull. `list_tasks` and `get_task` strip `type_config` by default. When you genuinely need the option catalogue, call `list_custom_fields` once for the list, or pass `include_field_schema: true` on a specific call. This is the single biggest reason to use this plugin over the stock connector.

## Safety model

<p align="center">
  <img src="assets/safety-gates.png" alt="Every write passes through a confirmation gate before it reaches ClickUp" width="100%">
</p>

Every write is gated. The write tools refuse to run unless invoked through a `/clickup-*` command that has shown you a confirmation summary and received an explicit yes. There is no path to a silent write.

- Bulk operations via the MCP tool are recommended for up to 10 tasks. Sets larger than 10 use the bundled clickup-batch runner. The MCP tool still hard-refuses more than 100 as a safety backstop.

- A bulk delete needs a second explicit confirmation in addition to the normal one. A 100-task delete cannot happen by accident.

- Set `CLICKUP_DRY_RUN=1` to block all writes globally. In dry-run, write tools return a clearly labelled synthetic response and never touch ClickUp. Reads still work.

- Every mutating call is appended to a local audit log at `~/.local/share/clickup-plugin/audit.log` with a timestamp, the operator, the operation, your confirmation summary, and the affected ids. The audit write is best-effort and never aborts your operation.

## What you get

Skills (these activate on their own when relevant).

- `clickup-task-search` - find and summarise tasks in a list or space.

- `clickup-list-audit` - a canonical snapshot of a list - counts by status, custom-field coverage, relationship density.

- `clickup-bulk-plan` - plan a bulk change and preview it. This skill only ever plans. It never writes.

Commands (explicit, confirmed writes).

- `/clickup-task` - create or update a single task, after showing you a confirmation summary and waiting for your yes.

- `/clickup-bulk-apply` - apply a previously previewed bulk change to up to 10 tasks, after explicit confirmation. Larger sets use the bundled clickup-batch runner. Bulk deletes require a second, separate confirmation.

MCP tools (Claude calls these as needed, with the safety rules above).

- Navigation - `list_workspaces`, `list_spaces`, `list_folders`, `list_lists`, `get_list`.

- Read - `list_tasks` (auto-paginated, schema-stripped by default), `get_task`, `list_custom_fields`.

- Single writes (gated) - `create_task`, `update_task`, `delete_task`, `set_custom_field`, `add_comment`, `set_task_relationship`.

- Bulk (gated, capped) - `bulk_update_tasks`.

## Bulk and large jobs

<p align="center">
  <img src="assets/bulk-routing.png" alt="Bulk routing - 10 or fewer through the command, more through the runner, hard cap at 100" width="100%">
</p>

`bulk_update_tasks` is recommended for sets of up to 10 tasks. For larger sets, or any job you want to run idempotently with a dry-run preview and a written report, use the `clickup-batch` runner. It ships with this plugin (installed as `clickup-batch` on PATH). It is dry-run by default, replaces a marker block in place so re-runs do not duplicate, and paces writes under ClickUp's rate limit. The MCP bulk tool hard-refuses more than 100 tasks as a safety backstop, but the recommended path for more than 10 tasks is the runner.

## The audit log

<p align="center">
  <img src="assets/audit-log.png" alt="Every mutation appends one line to a local, never-uploaded audit log" width="100%">
</p>

Every mutating call appends one line to `~/.local/share/clickup-plugin/audit.log` - timestamp, operator, operation, your confirmation summary, and the affected task ids. It is local only and never uploaded. The audit write is best-effort. If it fails (a read-only disk, for example) it logs to stderr and your operation still completes. The log is the answer to "what did this actually change", and it is yours alone.

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
