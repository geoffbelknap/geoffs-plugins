# Geoff's Plugins

A Claude Code compatible plugin marketplace for Geoff's hand crafted sustainably harvested plugins, skills and MCPs.

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
/plugin install limacharlie-mcp@geoffs-plugins
```

## Plugins

### [ask-framework](./plugins/ask-framework)



[ASK](https://askframework.org/) compliance reviewer, architecture designer, and threat analyst for agentic systems.

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

[Defined Networking](https://www.defined.net/) / Managed Nebula MCP integration for coding and DevOps agents.

Use it to inspect and manage Defined Networking infrastructure, including networks, hosts, enrollment codes, roles, firewall rules, tags, routes, audit logs, downloads, and host diagnostics.

```bash
/plugin install defined-mcp@geoffs-plugins
```

Set `DEFINED_API_KEY` in your Claude Code or MCP client environment before using this plugin. Do not paste API keys into shell commands or commit them to config files.

### [limacharlie-mcp](./plugins/limacharlie-mcp)

[LimaCharlie](https://limacharlie.io/) MCP suite for security operations agents.

Use it to run focused MCP profiles for auth/core reference, fleet onboarding,
administration, content maintenance, detection triage, containment, eviction,
recovery verification, and read-only posture review.

```bash
/plugin install limacharlie-mcp@geoffs-plugins
```

Configure LimaCharlie credentials through Vault for deployment. The plugin MCP
config uses `uvx` and the public
[geoffbelknap/limacharlie-mcp](https://github.com/geoffbelknap/limacharlie-mcp)
repository; do not commit API keys or Vault tokens to plugin config.

## Related Projects

- ASK framework: [github.com/geoffbelknap/ask](https://github.com/geoffbelknap/ask)
- Defined MCP server: [github.com/geoffbelknap/defined-mcp](https://github.com/geoffbelknap/defined-mcp)
- LimaCharlie MCP server: [github.com/geoffbelknap/limacharlie-mcp](https://github.com/geoffbelknap/limacharlie-mcp)

Each plugin directory has its own README with details, expected prompts, and setup notes.

## License

[Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE) — free to share and adapt for any purpose, including commercial, with attribution.
