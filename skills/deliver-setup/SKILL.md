---
name: deliver-setup
description: Set up or update personal integration defaults for deliver skills when choosing tools, accounts, or destinations.
---

# Deliver setup

Save integration choices once for reuse across agents and repositories.
An adapter is an existing connector, MCP tool, or CLI; no adapter code is required.

## Configuration

Use `$XDG_CONFIG_HOME/deliver/integrations.md`, falling back to
`~/.config/deliver/integrations.md` when `XDG_CONFIG_HOME` is unset or empty.
Each workflow skill reads this file directly. Explicit task choices and repository
instructions take precedence over these defaults.

## Choose

Read existing preferences, inspect available connections and CLIs, and check the repository
remote. Reuse existing choices; ask only about unresolved accounts, tools, or destinations.
Configure only what the user needs now:

| Integration | Record |
|---|---|
| Tracker | Provider, access method, host/workspace, default team/project for new items |
| Git host | Host, CLI (`gh`, `glab`, or equivalent), account if needed |
| Observability | Provider, access method, organization/project, query scope |
| Environment | References for base URLs and authentication: file paths, secret-manager references, or environment variable names |

Resolve ambiguous destinations with the user and record actual identifiers where available.
Do not assume the current repository is their global default. Match Git-host bindings to
the repository remote or PR/MR URL; resolve account conflicts before use.

Connections may differ between agents. If a saved access method is unavailable, resolve
that gap while preserving provider, account, and destination choices unless the user changes them.

## Verify

Use small read-only checks of identity and chosen destinations; tool availability alone
does not prove authentication, and read access does not prove write permission.
Check environment references without exposing secrets or exercising the deployed system.
Mark unavailable integrations unverified and explain the connection/login steps needed.
Do not install tools or change authentication unless requested.

## Save

Preview choices and verification status, then save the user's selections as concise Markdown
under `## Deliver integrations`. Create the parent directory if needed; update the same file,
preserving unrelated content and choices the user hasn't changed. Mark skipped integrations
unconfigured and note verification gaps.

Store credential references, never secrets. Leave project files and agent settings untouched
unless explicitly requested. Saved preferences do not replace each workflow's approval rules.

Read back the file, then report its path, configured integrations, and remaining connection steps.
