# ASK Framework Plugin

Claude Code plugin for the [ASK (Agent Security Framework)](https://github.com/geoffbelknap/ask).

## What It Does

- **Compliance reviews** — Structured pass/fail audit against the 22 ASK tenets
- **XPIA analysis** — Kill chain posture assessment (Injection → Propagation → Execution → Exfiltration)
- **Design generation** — Produce Mind, Gateway, Egress Proxy, and Enforcer configurations
- **Architecture guidance** — Design ASK-compliant agent deployments from scratch

## Skills

### `ask-review`

The core skill. Loaded when Claude detects ASK-related work — compliance reviews, XPIA analysis, agent security architecture, enforcement gap identification, or configuration generation.

Includes reference files for deep dives:
- `references/configurations.md` — Mind, Gateway, Egress, Enforcer config templates
- `references/xpia-patterns.md` — XPIA attack patterns and defensive architectures
- `references/multi-agent.md` — Multi-agent trust and delegation patterns

## Install

```bash
/plugin install ask-framework@geoffs-plugins
```
