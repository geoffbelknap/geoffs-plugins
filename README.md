# Geoff's Plugins

A Claude Code plugin marketplace for agent security, Agent Experience reviews, and infrastructure MCP integrations.

Use this marketplace when you want Claude Code to:

- review agent systems against the ASK security framework
- evaluate or refactor CLIs and MCP servers for better Agent Experience
- connect to Defined Networking / Managed Nebula through MCP

## Install

Add the marketplace once:

```bash
/plugin marketplace add geoffbelknap/geoffs-plugins
```

Then install the plugin you want:

```bash
/plugin install ask-framework@geoffs-plugins
/plugin install ax-optimizer@geoffs-plugins
/plugin install defined-mcp@geoffs-plugins
```

## Plugins

### [ask-framework](./plugins/ask-framework)

ASK compliance reviewer, architecture designer, and threat analyst for agentic systems.

Use it to audit agent architectures against ASK, design enforcement layers, verify cognitive model boundaries, assess XPIA posture, and generate compliant configurations.

```bash
/plugin install ask-framework@geoffs-plugins
```

### [ax-optimizer](./plugins/ax-optimizer)

Agent Experience reviewer for CLIs, MCP servers, API wrappers, and other agent-facing tool surfaces.

Use it to score AX maturity, identify implementation gaps, recommend concrete refactors, and design trajectory tests that prove the interface works well for coding and DevOps agents.

```bash
/plugin install ax-optimizer@geoffs-plugins
```

### [defined-mcp](./plugins/defined-mcp)

Defined Networking / Managed Nebula MCP integration for coding and DevOps agents.

Use it to inspect and manage Defined Networking infrastructure, including networks, hosts, enrollment codes, roles, firewall rules, tags, routes, audit logs, downloads, and host diagnostics.

```bash
/plugin install defined-mcp@geoffs-plugins
```

Set `DEFINED_API_KEY` in your Claude Code or MCP client environment before using this plugin. Do not paste API keys into shell commands or commit them to config files.

## Related Projects

- ASK framework: [github.com/geoffbelknap/ask](https://github.com/geoffbelknap/ask)
- Defined MCP server: [github.com/geoffbelknap/defined-mcp](https://github.com/geoffbelknap/defined-mcp)

Each plugin directory has its own README with details, expected prompts, and setup notes.

## License

[Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE) — free to share and adapt for any purpose, including commercial, with attribution.
