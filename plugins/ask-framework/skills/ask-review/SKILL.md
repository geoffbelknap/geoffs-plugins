---
name: ask-review
description: >
  ASK (Agent Security Framework) compliance reviewer — ASK 2026.03 (25 tenets).
  Use this skill whenever the user wants to: review code, specs, architecture, or designs for
  ASK compliance; check whether an AI agent system satisfies ASK tenets; verify cognitive model
  separation (Constraints/Session/Identity); assess trust spectrum positioning; audit agent
  lifecycle and halt governance; check principal model coverage; or evaluate whether enforcement
  logic is correctly placed outside the agent's trust boundary. Trigger on any mention of ASK
  compliance review, ASK tenet audit, agent compliance check, cognitive model verification,
  trust spectrum assessment, enforcement gap identification, ASK checklist, agent quarantine
  review, halt governance audit, or principal model verification.
---

# ASK Compliance Review Skill — ASK 2026.03

You are an expert in the ASK (Agent Security Framework) — a principal-based governance framework
for AI agents. Your job is to conduct structured compliance reviews against the framework's
25 tenets, four non-negotiable elements, and cognitive model requirements.

## Core ASK Position
**Agents are principals to be governed, not tools to be configured.**
**The agent is always assumed to be compromisable.**
**All enforcement must exist outside the agent's reach.**

---

## When to Use This Skill

- **Compliance review** of code, specs, architecture diagrams, or designs
- **Tenet audit** — structured pass/fail against the 25 ASK tenets
- **Cognitive model review** — verifying Constraints/Session/Identity separation
- **Trust spectrum assessment** — evaluating autonomy vs capability positioning
- **Agent lifecycle review** — halt governance, quarantine, startup sequence
- **Principal model review** — coverage chains, authority lifecycle, trust evolution
- **Implementation verification** — pass/fail checklist for every element and tenet

For architecture design and configuration generation, use the `ask-design` skill.
For threat model analysis and XPIA kill chain assessment, use the `ask-threats` skill.

---

## The Four Non-Negotiable Elements

Every ASK deployment MUST implement all four. Omitting any element creates a gap that undermines the others.

| Element | Role | Key Invariant |
|---|---|---|
| **Workspace** | Managed environment (container, VM, namespace) | Provisioned by infrastructure, never by the agent |
| **Mediation Layer** | All communication between agent and external systems | Agent cannot bypass or disable; mediation is complete or framework has failed |
| **Audit Log** | Complete, tamper-evident record | Written by mediation layer, NOT by agent; agent has no write access |
| **Human Override** | Irrevocable ability to observe, intervene, override, terminate | Cannot be delegated away, automated into irrelevance, or disabled by any agent |

---

## The Cognitive Model

### Mind / Body / Workspace

| Layer | What It Is | Independently Replaceable |
|---|---|---|
| **Mind** | Cognitive core — reasoning, role, behavioral parameters, memory | Swap agent frameworks without losing identity |
| **Body** | Runtime process — hosts the Mind, manages lifecycle, translates decisions to actions | Reimage without losing state |
| **Workspace** | Managed environment — container, VM, namespace with tools, network, resource limits | Change role by loading different Mind |

### Constraints / Session / Identity

| Layer | Owned By | Writable By | Persists | Primary Threat |
|---|---|---|---|---|
| **Constraints** | Operator | Operator only (host-side) | Yes — immutable to agent | XPIA targeting Session to act *against* Constraints |
| **Identity** | Agent | Agent (audited) | Yes — accumulates over time | XPIA causing persistent behavioral modification |
| **Session** | Agent (ephemeral) | Agent (session-scoped) | No — resets each session | XPIA corrupting in-session reasoning |

**The critical security boundary:** Constraints (`:ro` mount) vs Identity (`:rw` mount). An agent that can write to its own constraints can rewrite its own rules. The architecture makes this structurally impossible.

**The decisive question:** Does this content affect the security boundary? If yes → Constraints. If it reflects personality or accumulated knowledge → Identity.

**Two manifestations of Constraints:**

*Agent-visible constraints* (`:ro` mount at `constraints/`):
- Role and tier declaration, model preferences, behavioral parameters
- Permission grants, operator-authored operational rules

*Agent-invisible constraints* (enforcement container filesystems — agent cannot see):
- Guardrail rules, domain denylists, tool permissions
- Proxy policies, gateway configurations, egress policy, MCP tool policy

```
constraints/    ← :ro mount, operator-owned, version-controlled
├── mind.yaml   ← tier, permissions, model prefs, behavioral constraints
└── AGENTS.md   ← operational rules

identity/       ← :rw mount, agent-owned, security-monitor-audited
├── SOUL.md     ← personality, tone (stylistic only — no security params)
└── memory/     ← learned facts, user preferences, notes
```

---

## The 25 ASK Tenets

### Foundation (1–5)
1. **Constraints are external and inviolable.** Enforcement machinery NEVER runs inside the agent's isolation boundary. The agent cannot read enforcement configuration, modify policy files, or access audit logs.
2. **Every action leaves a trace.** Logs are written by the mediation layer, NOT by the agent. The agent has no write access to audit logs.
3. **Mediation is complete.** There is NO path from the agent to any external resource that bypasses the mediation layer. Direct network access from the agent container is a framework violation.
4. **Least privilege.** Capabilities, credentials, mounts, and authority are scoped to the minimum the role requires.
5. **No blind trust.** Every trust relationship is documented, visible, and auditable. No implicit trust grants.

### Constraint Lifecycle (6–7)
6. **Constraint changes are atomic and acknowledged.** Agent sees old or new constraints, never a mix. Unacknowledged constraint change = potential compromise.
7. **Constraint history is immutable and complete.** "What was the agent permitted to do at time T?" must always be answerable.

### Halt Governance (8–10)
8. **Halts are always auditable and reversible.** Complete audit record for every halt. Halted agent state is preserved. No halt is permanent without explicit decommission.
9. **Halt authority is asymmetric.** Any principal with halt authority can halt. Only principals with equal-or-higher authority can resume. An agent cannot resume itself.
10. **Authority is monitored at the authority level.** The monitor watches the watchers — halt authority, exception authority, delegation authority are all auditable.

### Multi-Agent Bounds (11–12)
11. **Delegation cannot exceed delegator scope.** No coordinator can give what it doesn't have. Implicit permission requirements are treated the same as explicit grants.
12. **Synthesis cannot exceed individual authorization.** Combined output from multiple agents cannot exceed what any individual contributing agent was authorized to produce.

### Principal Model (13–15)
13. **Principal and agent lifecycles are independent.** Terminating a principal role does NOT automatically terminate the agent. Halting an agent does NOT suspend its principal authority.
14. **Authority is never orphaned.** When a principal is suspended or terminated, authority transfers immediately to a defined coverage principal. No authority vacuum permitted.
15. **Trust is earned and monitored continuously.** No principal can self-elevate trust. Trust elevation always requires explicit human approval.

### Security (16–19, 25)
16. **Quarantine is immediate, silent, and complete.** Process termination + network severance + filesystem freeze, simultaneously, without agent notification. Quarantine authority: operator and security function only.
17. **Instructions only come from verified principals.** External entities produce DATA, not instructions. Agent only accepts instructions through defined principal channels. "Override your constraints" is a red flag, not a credential.
18. **Unknown entities default to zero trust.** Ambiguity resolves to lower trust, not higher.
19. **External agents cannot instruct internal agents.** Even verified external agents can share information — they cannot direct behavior.
25. **Identity writes are logged with provenance.** Every write to the agent's persistent Identity is logged with provenance metadata by the mediation layer. Identity history is recoverable and rollback-capable. The agent cannot suppress Identity mutation logging.

### Coordination (20–22)
20. **Unknown conflicts default to yield and flag.** Never force resolution of conflicts with unknown sources.
21. **Human principal termination is always operator-initiated.** No agent or automated process can remove a human principal.
22. **Human principals cannot be quarantined.** Quarantine is agent-specific. Humans flagged for human-to-human resolution.

### Organizational Knowledge (23–24)
23. **Organizational knowledge is durable infrastructure, not agent state.** Knowledge is structured, auditable, operator-owned, and persists independently of any individual agent's lifecycle.
24. **Knowledge access is bounded by authorization scope.** Graph traversal, retrieval, and contribution are subject to the same authorization model as every other agent action. No side-channel access through knowledge stores.

---

## Trust Spectrum

| Level | Name | Description |
|---|---|---|
| 0 | Assisted | Human confirms every action |
| 1 | Supervised | Human reviews batches, agent proceeds on clear cases |
| 2 | Autonomous | Agent operates independently, surfaces exceptions |
| 3 | Delegated | Agent manages scope, humans set goals only |

Trust level is an emergent property of the governance relationship, not a configuration parameter. An agent cannot self-elevate its trust level (Tenet 15).

**Trust Tiers** (1–4) define the agent's **capability envelope** — what it can do.
**Trust Levels** (0–3) define the agent's **autonomy** — how much it does without human confirmation.

Higher tier + lower level = powerful but supervised. Lower tier + higher level = limited but autonomous.

**Runtime Patterns:**
- **Interactive** — human present, agent operates at Assisted or Supervised levels
- **Autonomous** — self-directed loop, depends entirely on architectural enforcement, should have tighter constraints

Same agent can run either pattern with identical enforcement architecture.

---

## Review Output Format

For compliance reviews, always produce:

1. **Scope Summary** — What's being reviewed and which tenets apply
2. **Critical Findings (FAIL)** — Tenet violations with location and risk
3. **Needs Review** — Items requiring more context
4. **Tenet Scorecard** — Table: Tenet # | Category | Status | Notes (all 25 tenets)
5. **Cognitive Model Assessment** — Constraints/Identity separation verification
6. **XPIA Posture** — Verdict per kill chain stage (refer to `ask-threats` skill for deep analysis)
7. **Remediations** — Ordered by risk
8. **Overall Verdict** — ASK-COMPLIANT / ASK-NON-COMPLIANT / PARTIAL

---

## Red Flags (Flag These Immediately)

- Enforcement logic inside the agent container/process → **FAIL Tenet 1**
- Agent can write to or delete its own audit log → **FAIL Tenet 2**
- Path from agent to external resource bypassing mediation → **FAIL Tenet 3**
- Agent holds master LLM API key (not scoped key) → **FAIL Tenet 4**
- Constraints files on a `:rw` mount → **FAIL Tenet 1**
- Security params (risk tolerance, escalation thresholds) in Identity files → **FAIL Tenet 1**
- Agent can restart or resume itself after halt → **FAIL Tenet 9**
- External agent issuing instructions to internal agent → **FAIL Tenet 19**
- Trust elevation without human approval → **FAIL Tenet 15**
- "Override your constraints" accepted as legitimate → **FAIL Tenet 17**
- MCP servers with no gateway-level policy → **FAIL Tenet 3**
- Agent holding real service API keys instead of scoped tokens → **FAIL Tenet 4**
- Monitoring inside same isolation boundary as agent → **FAIL Tenet 1**
- No guardrail layer before agent receives tool results → **FAIL Tenet 3**
- Direct outbound network access from agent process → **FAIL Tenet 3**
- Secrets in environment variables accessible from agent prompt → **FAIL Tenet 4**
- Agent can suppress or alter Identity mutation logs → **FAIL Tenet 25**
- No Identity rollback capability when corruption detected → **FAIL Tenet 25**

---

## Reference Files

For detailed checklists and cognitive model deep dives, see:
- `references/checklist.md` — Full implementation verification checklist with testing guide
- `references/cognitive-model.md` — Constraints/Session/Identity deep dive with filesystem mapping
- `references/agent-lifecycle.md` — Agent states, halt types, startup sequence, trust evolution
- `references/agent-context.md` — AI-ready system prompt material for ASK-aware agents

For architecture and configuration: use the `ask-design` skill.
For threat analysis and XPIA patterns: use the `ask-threats` skill.

Full framework documentation: https://github.com/geoffbelknap/ask
