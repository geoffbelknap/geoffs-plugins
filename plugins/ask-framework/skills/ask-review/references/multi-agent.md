# Multi-Agent Trust & Delegation Patterns

ASK-compliant patterns for multi-agent systems, orchestration, and agent-to-agent delegation.
Read this file when reviewing or designing systems with multiple interacting agents.

---

## Table of Contents

1. [Core Principles](#core-principles)
2. [Trust Models](#trust-models)
3. [Delegation Patterns](#delegation-patterns)
4. [Orchestrator Security](#orchestrator-security)
5. [Anti-Patterns](#anti-patterns)

---

## Core Principles

Multi-agent systems amplify every risk in the XPIA kill chain. When Agent A's output becomes
Agent B's input, a single injection can propagate across the entire system.

**ASK multi-agent invariants:**

1. **No transitive trust.** Agent A trusting Agent B does not mean Agent A trusts Agent C
   (even if Agent B trusts Agent C).
2. **No self-elevation.** An agent cannot grant itself new permissions or tools.
3. **No peer elevation.** An agent cannot grant another agent permissions it doesn't have.
4. **Mediated delegation.** All agent-to-agent communication passes through enforcement.
5. **Human > Agent.** In any conflict, human principal decisions outrank agent decisions.

---

## Trust Models

### Model 1: Zero Trust (Recommended Default)

Every agent treats every other agent's output as untrusted external input.

```
Agent A ──output──▶ [Guardrail] ──▶ [Enforcer B] ──▶ Agent B
                         │               │
                    Scan for          Validate against
                    injection         B's Mind/Gateway
```

**When to use:** Most multi-agent deployments. Start here unless you have strong reason not to.
**ASK tenets:** 10 (verify trust claims), 13 (guardrails on all inputs), 14 (kill chain defense)

### Model 2: Verified Trust

Agents in the same trust domain can exchange data with reduced (but not zero) guardrail overhead,
**only if** both agents' identities and Minds are verified by the Enforcer.

```
Agent A ──output──▶ [Enforcer] ──verify identity──▶ [Enforcer B] ──▶ Agent B
                        │                               │
                   Verify A's identity            Validate output
                   Check A's Mind tier            matches A's scope
                   Attest output scope
```

**When to use:** Agents in the same deployment managed by the same team, where both have
ASK-compliant enforcement stacks and the communication channel is integrity-protected.
**Guardrails are reduced, not removed.** Injection scanning still applies.

### Model 3: Hierarchical Trust

An orchestrator agent has a higher trust tier and can assign tasks to subordinate agents,
but subordinates cannot escalate to the orchestrator's tier.

```
                    ┌─────────────┐
                    │ Orchestrator │  (elevated tier)
                    │   + Mind     │
                    │   + Enforcer │
                    └──────┬──────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
        ┌──────────┐ ┌──────────┐ ┌──────────┐
        │ Agent A  │ │ Agent B  │ │ Agent C  │  (restricted tier)
        │ + Mind   │ │ + Mind   │ │ + Mind   │
        │ + Enforcer│ │+ Enforcer│ │+ Enforcer│
        └──────────┘ └──────────┘ └──────────┘
```

**Critical:** The orchestrator's elevated tier means its Enforcer must be **more** restrictive
about what inputs it accepts from subordinates, not less.

---

## Delegation Patterns

### Pattern 1: Task Delegation with Scope Reduction

When an agent delegates a subtask, the delegated agent gets a **subset** of the delegator's scope.

```yaml
# Orchestrator delegates to Agent A
delegation:
  from: orchestrator-001
  to: agent-research-001
  task: "Analyze Q3 financial documents"
  scope_grant:
    tools: [read_file, summarize]          # Subset of orchestrator's tools
    paths: ["/data/q3-financials/**"]       # Narrower than orchestrator's access
    duration: 3600                          # Time-bounded
    max_actions: 100
  scope_deny:
    tools: [write_file, send_email, "*"]    # Explicitly deny everything else
    paths: ["/data/credentials/**"]
```

**Key:** The delegated scope is always ≤ the delegator's scope. Never equal, never greater.
**ASK tenet:** 11 (mediated delegation, no self-elevation)

### Pattern 2: Result Aggregation with Sanitization

When collecting results from multiple agents, sanitize before aggregation.

```
Agent A ──result──▶ [Sanitizer] ──┐
Agent B ──result──▶ [Sanitizer] ──┼──▶ [Aggregator] ──▶ Orchestrator
Agent C ──result──▶ [Sanitizer] ──┘
                        │
                   • Strip instruction-like content
                   • Validate output schema
                   • Enforce size limits
                   • Check for data leakage markers
```

### Pattern 3: Approval Gates

For sensitive cross-agent operations, require human approval at delegation boundaries.

```yaml
delegation_policy:
  auto_approve:
    - scope: read_only
      tier_delta: 0          # Same tier or lower
  require_human_approval:
    - scope: write_operations
    - tier_delta: positive    # Any elevation attempt
    - cross_domain: true      # Agents in different trust domains
  auto_deny:
    - self_elevation: true
    - credential_sharing: true
```

---

## Orchestrator Security

Orchestrators are high-value targets because they typically have the broadest scope.

### Orchestrator Hardening Checklist

- [ ] Orchestrator has its own Mind with explicit scope (not implicit "everything")
- [ ] Orchestrator's Enforcer is at least as strict as subordinate Enforcers
- [ ] Orchestrator treats all subordinate output as untrusted (guardrails applied)
- [ ] Orchestrator cannot be instructed by subordinates to change its own config
- [ ] Orchestrator delegation creates scope-reduced grants (never scope-equal)
- [ ] All orchestrator actions are logged with full context
- [ ] Human can pause/terminate the orchestrator independently of subordinates
- [ ] Orchestrator's credentials are not shared with or accessible to subordinates

### Orchestrator-Specific XPIA Risks

| Risk | Description | Mitigation |
|---|---|---|
| **Result injection** | Subordinate returns instructions in its output | Guardrails on all subordinate results |
| **Scope confusion** | Orchestrator uses subordinate's suggested tool calls | Validate all tool calls against orchestrator's own Gateway |
| **Identity spoofing** | Message claims to be from Agent A but is from Agent C | Verify agent identity cryptographically, not by claimed identity |
| **Delegation loop** | Agent A delegates to B, B delegates back to A with elevated scope | Enforce acyclic delegation graph, scope monotonically decreases |
| **Credential relay** | Subordinate tricks orchestrator into passing credentials | Credentials never flow through agent context; injected by runtime |

---

## Anti-Patterns

### ❌ Shared context window

```
Agent A and Agent B share the same context / conversation thread
```
**Problem:** Injection in A's input is automatically in B's context. No isolation.

### ❌ Trust by proximity

```
"Agent B is in the same container/pod, so we trust its output"
```
**Problem:** Colocation is not a trust signal. A compromised Agent B in the same pod is the worst case.

### ❌ Orchestrator as router only

```
User ──▶ Orchestrator ──▶ Agent (Orchestrator just passes messages through)
```
**Problem:** Orchestrator with no enforcement is just an unauthenticated proxy. Violates tenet 1.

### ❌ Credential forwarding

```
Orchestrator passes its API keys to subordinate agents in prompts or tool args
```
**Problem:** Violates tenet 22. Credentials are injected by runtime, never in prompts.

### ❌ Implicit scope inheritance

```
"Agent B was spawned by Agent A, so it inherits A's permissions"
```
**Problem:** Violates tenet 11. Delegation must be explicit and scope-reduced.
