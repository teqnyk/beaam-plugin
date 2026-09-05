# Beaam plugin

Keep building while Beaam keeps watch. This plugin connects Claude Code and
Cowork to Beaam's production MCP server and adds a safe workflow for deriving a
monitoring plan from a local project.

## What it includes

- **Beaam MCP server** — current health, monitoring coverage, incidents,
  integrations, services, stacks, alert rules, maintenance and notifications.
- **Monitor project skill** — audits a repository without reading secrets,
  creates a consent-based monitoring plan, guides provider connections and
  verifies the resulting coverage.

The package is shared with Beaam's Codex plugin. Both use the same production
endpoint and product documentation so their behavior and listing content do not
drift.

## Setup

Install the plugin and connect Beaam when Claude prompts you. Authentication is
handled by Beaam's OAuth flow; no environment variables, API tokens or local
server processes are required.

The plugin connects only to:

`https://app.beaam.app/api/mcp`

## Usage

Ask Claude naturally, for example:

- “Monitor this project with Beaam.”
- “Audit this repository's monitoring coverage.”
- “Find the monitoring gaps in this app and propose a plan.”
- “Set up Beaam monitoring for this repository.”

Claude first performs a read-only audit and shows the proposed plan. It applies
changes only after explicit approval and then verifies what Beaam is actually
watching.

## Safety

The project audit reads dependency names, repository-relative file paths,
example environment-variable names, deployment configuration and public API
hostnames. It must not read `.env` files, credential stores or secret values.
OAuth and token-based connections remain in Beaam's secure connection flow.

## Support and policies

- [Documentation](https://beaam.app/docs/mcp/)
- [Support](https://beaam.app/docs)
- [Privacy policy](https://app.beaam.app/privacy)
- [Terms of service](https://app.beaam.app/tos)
