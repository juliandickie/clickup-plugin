# Usage

Practical walkthroughs, the safety behaviour you will actually hit, and troubleshooting. For install and the high-level overview see README.md.

## First-time setup

1. Get a personal token in ClickUp - Settings - Apps - Generate API Token. It begins with `pk_`.

2. Give it to the plugin one of three ways (resolved in this order). The simplest for a single machine is the global config file.

   - On install, answer the plugin's token prompt.

   - Or export `CLICKUP_API_TOKEN=pk_...` (or `CLICKUP_API_TOKEN=op://Vault/ClickUp/credential` if you use the 1Password CLI).

   - Or create `~/.config/clickup-plugin/config.toml` with `api_token = "pk_..."` (or an `op://` reference; add `op_account = "your-team.1password.com"` on the next line if you are signed in to more than one 1Password account).

3. Sanity check - ask Claude to "list my ClickUp workspaces". You should see your workspaces. A 401 means the token is wrong or revoked.

## Finding and auditing work

- "Search ClickUp for open tasks in the Marketing list" - the `clickup-task-search` skill resolves the list (drilling through workspace, space, folder if you are vague), pulls the tasks, and gives you a compact table. It does not dump raw API JSON.

- "Audit the Sprint 14 list" - `clickup-list-audit` produces a fixed-format snapshot - total tasks, counts by status, what percent of tasks have each custom field populated, how many have dependencies or links. Run it again later and the format is identical so the two are comparable.

These read paths strip custom-field option schemas by default. If you specifically need a field's option list, ask for the field catalogue (it calls `list_custom_fields` once) rather than reading it off every task.

- "Find every urgent task assigned to Olga across the workspace" - `filter_workspace_tasks` filters across all lists, spaces, and folders in one call (status, assignee, tag, and date filters combine). No more walking lists one by one. Assignee filters take user ids - "who is on this workspace" calls `list_members` for the id, name, email, and role of every member.

- "What's the discussion on task X" - `get_task_comments` reads the 25 most recent comments (newest first), with `fetch_all` walking the full history when you need it. A comment with replies shows a reply_count - `get_comment_replies` pulls the thread.

## Reading Docs

The Docs tools run on ClickUp API v3 (the plugin's first v3 surface - see the README's API v3 posture note).

- "What docs are in the Marketing space" - `search_docs` filters by parent container, creator, or id, one page per call with a cursor to continue.

- "Read the onboarding doc" - `list_doc_pages` first for the table of contents (ids and names, no content), then `get_doc_pages` for the content in markdown. For a large doc, check the TOC before pulling every page.

## Changing a single task

Use `/clickup-task`. For example `/clickup-task update <task> set status to done`. The command resolves the target, shows you a confirmation summary, and waits. Nothing is written until you reply yes. On no, nothing happens.

## Bulk changes

1. Ask for a bulk plan - "set status Blocked on every task in list X assigned to nobody". The `clickup-bulk-plan` skill resolves the set, shows you the exact count and a sample, and states the routing - 10 or fewer goes through `/clickup-bulk-apply`; more than 10 goes to the bundled clickup-batch runner (which ships with the plugin and is on PATH). The MCP bulk tool still hard-refuses more than 100 as a safety backstop. This skill never writes.

2. Apply with `/clickup-bulk-apply`. It restates the change, shows a confirmation summary, and waits for an explicit yes. It reports updated and failed counts, and lists any failures with their errors.

3. Bulk delete is special. `/clickup-bulk-apply` will require a second, separate destructive confirmation (you type DELETE) on top of the normal yes, and the underlying tool refuses a delete without that second signal. This is deliberate - a 100-task delete cannot happen from a single careless yes.

## Testing safely with dry-run

Set `CLICKUP_DRY_RUN=1` in the plugin's environment. Every write tool then returns a clearly labelled synthetic response and makes no API call. Reads still work normally. Use this to rehearse a bulk plan end to end without touching ClickUp, then unset it to apply for real.

## Large or idempotent jobs

`bulk_update_tasks` is recommended for up to 10 tasks. For larger sets, or a job you want to re-run safely (for example appending a context block to many task descriptions), use the `clickup-batch` runner. It ships with this plugin and is available as `clickup-batch` on PATH. It is dry-run by default, writes a markdown report of every change, and on a second run replaces its marker block in place rather than appending a duplicate, so running it twice equals running it once. It also paces writes to stay under ClickUp's per-minute limit. The MCP bulk tool hard-refuses more than 100 as a safety backstop, but any set larger than 10 should use the runner.

## Rate limits and retries

ClickUp's API is rate limited (roughly 100 requests per minute on the free plan, higher on paid). The client retries on HTTP 429 and 5xx with backoff, honours the `Retry-After` and `X-RateLimit-Reset` headers, and gives a 429 a larger retry budget so a request can wait out a reset window. A large bulk run that briefly trips the limit will slow down and recover rather than mass-fail. If a few tasks still fail, `/clickup-bulk-apply` reports exactly which ones and why, and re-running it only retouches what is needed.

## The audit log

Every mutating call appends one line to `~/.local/share/clickup-plugin/audit.log` - timestamp, operator, operation, your confirmation summary, and the affected task ids. It is local only, never uploaded. The audit write is best-effort - if it fails (read-only disk, for example) it logs to stderr and your operation still completes.

## Troubleshooting

- 401 Unauthorized at startup - the token is missing, mistyped, or revoked. Regenerate it in ClickUp at Settings - Apps and re-supply it.

- "Failed to resolve 1Password reference ... multiple accounts found" - your machine is signed in to more than one 1Password account and the op CLI needs to be told which one holds the item. Set `CLICKUP_OP_ACCOUNT` (or `op_account` in config.toml) to that account's sign-in address, e.g. `my-team.1password.com`. Also make sure the 1Password app is unlocked - `op read` authorises through it.

- A bulk run reports many failures - usually transient rate limiting. Re-run `/clickup-bulk-apply`; it only changes what still needs changing.

- "exceeds the 100-task per-call cap" - the set is larger than 100. The MCP tool hard-refuses at this point as a safety backstop. Use the bundled clickup-batch runner, which is on PATH.

- A bulk delete is rejected with a `delete_confirmed` message - that is the second-signal guard working. Go through `/clickup-bulk-apply`, which collects the extra destructive confirmation for you.

- Every request fails to connect - check `CLICKUP_BASE_URL`. If it is unset it defaults correctly. If it was set to an unresolved `${...}` placeholder the plugin ignores it and uses the default, so a connect failure usually means a real custom base URL is wrong.

- Relationship direction - `set_task_relationship` supports `link`, `depends_on` (this task is blocked by the target), and `blocks` (the target is blocked by this task). Both dependency directions were verified against live ClickUp storage on 2026-08-28. `remove_task_relationship` undoes any of the three - the kind and direction must match how the relationship was created.
