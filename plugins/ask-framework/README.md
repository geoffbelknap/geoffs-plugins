# ASK Framework Plugin — ASK 2026.03

Claude Code plugin for the [ASK (Agent Security Framework)](https://github.com/geoffbelknap/ask).

## What It Does

- **Compliance reviews** — Structured pass/fail audit against the 25 ASK tenets
- **Cognitive model review** — Verify Constraints/Session/Identity separation
- **Trust spectrum assessment** — Evaluate autonomy vs capability positioning
- **Architecture design** — Design ASK-compliant agent deployments with seven enforcement layers
- **Configuration generation** — Produce Mind, Gateway, Egress Proxy, Enforcer, Delegation, and Audit Log configs
- **Threat analysis** — Categorize threats as traditional/novel/hybrid, assess attack surfaces
- **XPIA analysis** — Kill chain posture assessment (Injection → Propagation → Execution → Exfiltration)
- **Multi-agent design** — Isolation, delegation bus, coordinator hardening
- **Implementation verification** — Pass/fail checklist for every element and tenet
- **Limitations awareness** — Honest accounting of known gaps

## Skills

### `ask-review`

Compliance reviewer. Loaded when Claude detects ASK compliance review work — tenet audits, cognitive model verification, trust spectrum assessment, enforcement gap identification, principal model review, or agent lifecycle audit.

References:
- `references/checklist.md` — Implementation verification checklist with testing guide
- `references/cognitive-model.md` — Constraints/Session/Identity deep dive
- `references/agent-lifecycle.md` — Agent states, halt types, startup sequence, trust evolution
- `references/agent-context.md` — AI-ready system prompt material for ASK-aware agents

### `ask-design`

Architecture designer and configuration generator. Loaded when Claude detects ASK architecture or configuration work — enforcement layer design, mind.yaml generation, gateway/egress/enforcer config, multi-agent system design, or deployment topology planning.

References:
- `references/configurations.md` — Mind, Gateway, Egress, Enforcer, Delegation, Audit Log configs
- `references/multi-agent.md` — Multi-agent trust models, delegation patterns, anti-patterns
- `references/standards.md` — Standards landscape (NIST, OWASP, CoSAI, A2A) and alignment

### `ask-threats`

Threat analyst. Loaded when Claude detects threat model or XPIA analysis work — threat categorization, kill chain assessment, attack surface analysis, MCP security, identity poisoning, behavioral drift, or framework limitations review.

References:
- `references/xpia-patterns.md` — XPIA attack patterns, defensive architectures, detection strategies
- `references/threats.md` — Full threat model: traditional, novel, hybrid categories
- `references/limitations.md` — Known gaps, open questions, honest limitations accounting

## Install

```bash
/plugin install ask-framework@geoffs-plugins
```
