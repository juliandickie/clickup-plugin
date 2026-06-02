# Contributing

Read this before opening a pull request here. It will save you the work.

## This repository is generated

`juliandickie/clickup-plugin` is not where development happens. It is the lean, installable payload, built and published automatically from a separate development monorepo on every release tag. Every file here except this one and the README is overwritten on each publish.

That has one hard consequence - a pull request opened against this repository will be erased by the next release and cannot be merged. Please do not send code PRs here.

## What to do instead

- Bugs, behaviour questions, feature requests - open an issue on this repository. Issues are the right channel and are not overwritten.

- Include, where relevant - the plugin version (see `.claude-plugin/marketplace.json` `metadata.version`), the tool or command involved, what you expected, what happened, and whether `CLICKUP_DRY_RUN` was set. Never paste your `pk_` token or audit-log contents into an issue.

## How changes actually ship

The development monorepo holds the MCP server TypeScript source, the batch runner, the plugin payload, tests, and the design specs and implementation plans. The flow is - work on a feature branch, open a pull request against the monorepo's `main` (its pre-push hook enforces this), merge after review, then push a `vX.Y.Z` tag. A GitHub Action builds the bundle from source and publishes the payload here, version-bumps this repo's `marketplace.json`, and cuts a matching release. This repository is never hand-edited.

## Conventions the project holds itself to

- Test-driven where there is logic - the MCP client, config, tools, and the batch runner all have unit tests, and the suite must be green before a release.

- Every write is gated behind an explicit confirmation, bulk is capped at 100 with a second signal required for bulk delete, and a dry-run switch blocks all writes. Changes must not weaken that surface.

- Tool argument types are derived from a single Zod schema per tool, not duplicated.

- The version is single-sourced from plugin/.claude-plugin/plugin.json. The build injects it into the MCP server and the publish workflow patches marketplace.json from it, so it is never hand-edited anywhere else.

- House text style - no em or en dashes, no colons in headings, straight quotes, a blank line between list items, no emojis. This applies to docs, skill prose, and user-facing strings.

## Security

If you find a security issue (a way to write without confirmation, a token leak, an injection), do not open a public issue with exploit details. Contact the maintainer privately through the GitHub profile linked in `marketplace.json`.
