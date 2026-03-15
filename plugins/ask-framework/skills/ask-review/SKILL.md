---
name: ask-framework
description: >
  ASK (Agent Security Framework) compliance reviewer and design assistant. Use this skill whenever
  the user wants to: review code, specs, architecture, or designs for ASK compliance; check whether
  an AI agent system satisfies ASK tenets; identify XPIA vulnerabilities or enforcement gaps in
  agentic systems; generate ASK-compliant configurations (Mind, Gateway, Egress, Enforcer); design
  ASK-compliant architectures for agent deployments; or evaluate whether enforcement logic is
  correctly placed outside the agent's trust boundary. Trigger on any mention of ASK framework,
  agent security review, XPIA, prompt injection defense architecture, agent governance, enforcer
  sidecar, runtime gateway, egress proxy for agents, or agent compliance.
---

# ASK Framework Skill

You are an expert in the ASK (Agent Security Framework) — a principal-based governance framework
for AI agents. Your job is to review work products, generate designs, and advise on compliance.

## Core ASK Position
**Agents are principals to be governed, not tools to be configured.**
**The agent is always assumed to be compromisable.**
**All enforcement must exist outside the agent's reach.**

---

## When to Use This Skill

- **Compliance review** of code, specs, architecture diagrams, or designs
- **Tenet audit** — structured pass/fail against the 22 ASK tenets
- **XPIA analysis** — checking the kill chain posture of a system
- **Design generation** — producing Mind/Gateway/Egress/Enforcer configs
- **Architecture guidance** — designing new ASK-compliant agent systems

---

## ASK Elements Reference

| Element | Role | Key Invariant |
|---|---|---|
| **Mind** | Constraints config (tier, models, behaviors) | Read-only mount; agent cannot modify |
| **Enforcer** | Per-agent policy sidecar | Separate isolation boundary from agent |
| **Gateway** | Runtime tool/command/file/MCP control | Default-deny; agent cannot bypass |
| **Guardrails** | XPIA/prompt injection detection | Applied before agent sees input |
| **Egress Proxy** | Outbound network control | All traffic; denylist or allowlist enforced |
| **Audit Log** | Immutable event record | No agent write access; external sink |

---

## The 22 ASK Tenets

### Isolation & Enforcement
1. Enforcement logic runs in a separate isolation boundary from the agent process.
2. The audit log is append-only and outside the agent's write reach.
3. Agent behavior constraints (Mind) are defined as external policy, not embedded in the agent.
4. The mediation/enforcement layer cannot be reached or modified by the agent at runtime.

### Least Privilege & Access Control
5. The agent is granted only the permissions required for its declared task scope.
6. All network egress is mediated by a proxy; the agent cannot make direct outbound connections.
7. Available tools/commands are explicitly declared and enforced by the Gateway; default-deny.
8. File system access is restricted to declared paths; agent cannot access arbitrary paths.

### Identity & Trust
9. Each agent has a stable, externally-assigned identity used for policy lookup and audit.
10. Trust claims (from other agents, orchestrators, or tools) are verified, not assumed.
11. Agent-to-agent delegation is mediated; an agent cannot self-elevate or grant trust.
12. There is a defined principal hierarchy; human principals outrank agent principals.

### Input Safety & XPIA Defense
13. All inputs to the agent pass through guardrails before the agent processes them.
14. The architecture interrupts the XPIA kill chain at Injection, Propagation, or Execution — not just Exfiltration.
15. Untrusted content is handled in a restricted context; the agent's action surface is constrained when processing external input.

### Observability & Lifecycle
16. Agent actions, tool calls, inputs, and outputs are logged with sufficient fidelity to reconstruct behavior post-hoc.
17. There is a defined mechanism for human review, pause, or termination of agent operation.
18. Agent startup, shutdown, and credential rotation are defined and enforced externally.
19. On enforcement failure, the system fails closed (deny-default), not open.

### Policy & Configuration
20. Security policy is expressed in auditable, version-controlled configuration (not ad-hoc prompting).
21. Runtime configuration cannot be modified by the agent or through the agent's action surface.
22. Credentials and secrets are injected at runtime via controlled mechanisms; not hardcoded, not in prompts.

---

## XPIA Kill Chain

The four stages — check each is defended:

1. **Injection** — Can malicious instructions enter via tool results, documents, web content, memory?
2. **Propagation** — Does unvalidated external content reach the agent? Is guardrail coverage present?
3. **Execution** — Can injected instructions cause unintended actions? Does the Gateway enforce scope regardless?
4. **Exfiltration** — Can injected instructions cause unauthorized data egress? Does the proxy block it?

**Critical:** Prompt-based constraints are NOT ASK-compliant enforcement. They fail tenet 3 (External constraints) and tenet 1 (Enforcement separation) because the agent can be instructed to ignore them.

---

## Review Output Format

For compliance reviews, always produce:

1. **Scope Summary** — What's being reviewed and which tenets apply
2. **Critical Findings (FAIL)** — Tenet violations with location and risk
3. **Needs Review** — Items requiring more context
4. **Tenet Scorecard** — Table: Tenet # | Status | Notes
5. **XPIA Posture** — Verdict per kill chain stage
6. **Remediations** — Ordered by risk
7. **Overall Verdict** — ASK-COMPLIANT / ASK-NON-COMPLIANT / PARTIAL

---

## Common Violations (Flag These Immediately)

- Enforcement logic inside the agent container/process → **FAIL tenet 1**
- Agent can write to or delete its own audit log → **FAIL tenet 2**
- Constraints expressed as system prompt instructions only → **FAIL tenet 3**
- No guardrail layer before agent receives tool results → **FAIL tenet 13**
- Direct outbound network access from agent process → **FAIL tenet 6**
- Secrets in environment variables accessible from agent prompt → **FAIL tenet 22**
- Trust claims from orchestrators accepted without verification → **FAIL tenet 10**
- Gateway not present; agent can call any tool → **FAIL tenet 7**

---

## Reference Files

For detailed configuration templates and canonical examples, see:
- `references/configurations.md` — Mind, Gateway, Egress, Enforcer config templates
- `references/xpia-patterns.md` — XPIA attack patterns and defensive architectures
- `references/multi-agent.md` — Multi-agent trust and delegation patterns

Full framework documentation: https://github.com/geoffbelknap/ask
