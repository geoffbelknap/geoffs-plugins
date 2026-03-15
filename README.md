# Geoff's Plugins

A Claude Code plugin marketplace by Geoff Belknap.

## Plugins

| Plugin | Description |
|---|---|
| **[ask-framework](./plugins/ask-framework)** | ASK (Agent Security Framework) compliance reviewer — audit agent architectures against 24 security tenets, verify cognitive model separation, assess trust spectrum positioning, analyze XPIA kill chain posture, and generate compliant configurations. Updated for ASK 2026.03. |

## Installation

```bash
# Add the marketplace
/plugin marketplace add geoffbelknap/geoffs-plugins

# Install a plugin
/plugin install ask-framework@geoffs-plugins
```

## About ASK

The ASK framework treats AI agents as **principals to be governed**, not tools to be configured. It assumes the agent is always compromisable and requires all enforcement to exist outside the agent's reach.

ASK defines four non-negotiable elements (Workspace, Mediation Layer, Audit Log, Human Override), a cognitive model (Mind/Body/Workspace with Constraints/Session/Identity separation), 24 tenets organized across 8 categories, and a trust spectrum from Assisted to Delegated autonomy.

Full framework documentation: [github.com/geoffbelknap/ask](https://github.com/geoffbelknap/ask)

## License

[Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE) — free to share and adapt for any purpose, including commercial, with attribution.
