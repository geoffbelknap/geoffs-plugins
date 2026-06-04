# Defined MCP

Claude Code plugin for the [Defined Networking MCP server](https://github.com/geoffbelknap/defined-mcp).

Use it to let Claude Code inspect and manage Defined Networking / Managed Nebula infrastructure through the Defined API. The MCP supports networks, hosts, enrollment codes, roles, firewall rules, tags, routes, audit logs, downloads, and host diagnostics.

## Install

```bash
/plugin install defined-mcp@geoffs-plugins
```

Set `DEFINED_API_KEY` in your MCP client environment or secret store. Do not paste API keys into shell commands or commit them to local config files.

## Example Prompts

```text
List my Defined Networking networks and summarize hosts, roles, routes, and tags.
```

```text
Find blocked or offline hosts in my production network and explain what should be checked next.
```

```text
Create a new role for CI runners with least-privilege firewall rules.
```

## Requirements

- Node.js 24.16.0 or newer within the Node 24 line
- A Defined Networking API key with the scopes needed for the operations you want the agent to perform

The server runs from [github.com/geoffbelknap/defined-mcp](https://github.com/geoffbelknap/defined-mcp). See that repository for the full tool list, API coverage, and changelog.
