# ASK Framework Plugin — ASK 2026.03

Claude Code plugin for the [ASK (Agent Security Framework)](https://github.com/geoffbelknap/ask).

## What It Does

- **Compliance reviews** — Structured pass/fail audit against the 24 ASK tenets
- **XPIA analysis** — Kill chain posture assessment (Injection → Propagation → Execution → Exfiltration)
- **Cognitive model review** — Verify Constraints/Session/Identity separation
- **Trust spectrum assessment** — Evaluate autonomy vs capability positioning
- **Design generation** — Produce Mind, Gateway, Egress Proxy, and Enforcer configurations
- **Architecture guidance** — Design ASK-compliant agent deployments from scratch
- **Implementation verification** — Pass/fail checklist for every element and tenet

## Skills

### `ask-review`

The core skill. Loaded when Claude detects ASK-related work — compliance reviews, XPIA analysis, agent security architecture, cognitive model verification, trust spectrum assessment, enforcement gap identification, or configuration generation.

Includes reference files for deep dives:
- `references/configurations.md` — Mind, Gateway, Egress, Enforcer, Delegation, Audit Log configs
- `references/xpia-patterns.md` — XPIA attack patterns and defensive architectures
- `references/multi-agent.md` — Multi-agent trust and delegation patterns
- `references/cognitive-model.md` — Constraints/Session/Identity deep dive
- `references/agent-lifecycle.md` — Agent states, halt types, startup sequence
- `references/checklist.md` — Implementation verification checklist

## Install

```bash
/plugin install ask-framework@geoffs-plugins
```
