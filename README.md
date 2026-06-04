# Geoff's Plugins

A Claude Code plugin marketplace by Geoff Belknap.

## Plugins

| Plugin | Description |
|---|---|
| **[ask-framework](./plugins/ask-framework)** | ASK (Agent Security Framework) compliance reviewer, architecture designer, and threat analyst — audit agent architectures against 25 security tenets, design seven-layer enforcement architectures, verify cognitive model separation, assess XPIA kill chain posture, analyze traditional/novel/hybrid threats, and generate compliant configurations. Updated for ASK 2026.03. |
| **[ax-optimizer](./plugins/ax-optimizer)** | Agent Experience (AX) reviewer for CLIs, MCP servers, and agent-facing tool surfaces — score AX maturity, identify implementation gaps, recommend concrete refactors, and design trajectory tests. |

## Installation

```bash
# Add the marketplace
/plugin marketplace add geoffbelknap/geoffs-plugins

# Install a plugin
/plugin install ask-framework@geoffs-plugins
/plugin install ax-optimizer@geoffs-plugins
```

## About ASK

The ASK framework treats AI agents as **principals to be governed**, not tools to be configured. It assumes the agent is always compromisable and requires all enforcement to exist outside the agent's reach.

ASK defines four non-negotiable elements (Workspace, Mediation Layer, Audit Log, Human Override), a cognitive model (Mind/Body/Workspace with Constraints/Session/Identity separation), 25 tenets organized across 8 categories, seven enforcement layers, and a trust spectrum from Assisted to Delegated autonomy.

Full framework documentation: [github.com/geoffbelknap/ask](https://github.com/geoffbelknap/ask)

## License

[Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE) — free to share and adapt for any purpose, including commercial, with attribution.
