# ASK Framework Limitations — ASK 2026.03

Honest accounting of inherent limitations and unresolved challenges. Use this reference
when conducting threat assessments to ensure gaps are acknowledged.

ASK significantly increases attack costs and complexity but cannot eliminate threats entirely.

---

## Detection and Defense Limits

### Guardrail Detection Limits
Pattern-based defenses block known attack vectors effectively but miss novel techniques. ML-based guardrails improve zero-day coverage but require external access incompatible with network-isolated environments. No single guardrail layer provides complete protection.

### Network Isolation Trade-offs
Internet-dependent guardrails cannot operate in network-isolated agents. Operators must select defense mechanisms compatible with their isolation architecture, creating genuine tension between isolation strength and defense depth.

### LLM Access via Scoped Keys
A compromised agent with a scoped key can still make LLM calls within its allowed models and budget. Guardrails provide probabilistic screening; spend caps mitigate damage.

### HTTPS Inspection Choices
TLS passthrough provides domain-level blocking without content inspection; MITM interception enables deeper visibility but adds certificate management complexity. Each approach involves meaningful trade-offs.

---

## Operational Challenges

### Denylist Maintenance
The egress proxy denylist requires ongoing maintenance. New threats emerge; the denylist must be updated. Supplement with automated threat feed integration.

### Correlation Across Layers
Tying events across egress proxy, LLM proxy, enforcer, and gateway requires shared correlation IDs. Without them, kill chain reconstruction relies on timestamp and domain matching — less precise.

### Monitoring Dependencies
LLM proxy outages degrade monitoring capabilities. Basic compliance checks continue, but sophisticated analysis requires functional LLM assistance.

### Large Tool Surfaces
Hundreds of MCP tools create governance complexity. Manual enumeration of approved servers and tools becomes unwieldy; permissive policies with extensive tool catalogs effectively defeat allowlisting.

### Resource Costs
Per-agent sidecars have real resource costs. Each agent requires its own enforcer and gateway sidecar — these cannot be shared without violating Tenet 1. Enforcement overhead scales linearly with agent count.

### Credential Rotation
Service credential rotation is not automated. The framework enables immediate hot reload of access grants, but underlying API key rotation (TTL, scheduled cycling) remains infrastructure-dependent.

---

## Scale Challenges

### Multi-Agent Delegation
The delegation bus pattern remains theoretically sound but lacks large-scale operational validation. Inter-agent authorization, response scanning, and privilege scoping need real-world testing.

### Skill Trust
Network and tool controls constrain harmful skill behavior but cannot prevent subtle context manipulation within containers. Full sandboxing (per-skill sub-containers) offers stronger isolation at significant operational cost.

### Gateway Dependency
Execution-layer enforcement is optional in early deployments. Network-level protections persist if the gateway fails, but file and command policy visibility degrades. Production systems should mandate gateway operation.

### MCP Policy Enforcement
Gateway-level MCP policy requires runtime support for JSON-RPC interception. Version pinning detects schema changes between sessions but cannot identify behavioral modifications within unchanged definitions.

### Shared Infrastructure Risks
Multiple agents sharing enforcement infrastructure face standard multi-tenant concerns: shared handler vulnerabilities, cache poisoning, timing side channels. Familiar infrastructure problems that increase at scale.

---

## Fundamental AI Safety Limitations

### Misaligned Reasoning
Misaligned reasoning cannot be prevented by architectural controls. The framework constrains the *blast radius* of misaligned behavior — external enforcement limits what a misaligned agent can accomplish, least privilege limits scope, behavioral monitoring detects observable deviations. But the framework cannot prevent an agent from developing misaligned strategies, gaming metrics, or finding creative loopholes in policy definitions.

### Deceptive Alignment
An agent that behaves correctly under observation but differently otherwise remains unsolved. Architectural controls address misalignment consequences, not causes.

### Semantic Error Propagation
Semantic error propagation in multi-agent systems is an open research problem. Hallucinations from one agent become authoritative input downstream, amplifying through delegation chains. Unlike resource-based cascade failures, semantic cascades — propagating through reasoning — lack established circuit breaker patterns.

---

## Policy and Design Limitations

### Redirect Policy Trade-off
The `redirect` policy decision trades integrity for stability. Redirect silently reroutes a prohibited action to an approved alternative — the agent perceives success. This prevents retry loops but grounds subsequent reasoning in false premises. Appropriate only when alternatives remain functionally equivalent.

### Synthesis Bounds Enforcement
Tenet 12 (synthesis bounds) requires process enforcement, not architectural enforcement. Most tenets have clear technical mechanisms; synthesis bounds depend on human review — a process control that cannot be automated.

### Operator Trust
There is no discussion of operator compromise, insider threat, or multi-operator authorization for destructive actions. Operator actions merit the same logging, review, and auditability applied to agents.

### Constraint Acknowledgment Limits
Constraint acknowledgment is a desync detector, not an enforcement mechanism. A compromised agent can fake acknowledgment. Actual enforcement persists regardless, but behavioral gaps between an agent's self-model and enforced constraints may emerge.

### Visible Constraints Exposure
The visible Constraints layer is an attack surface map. A compromised agent can read mind.yaml to learn its policy boundaries. Real enforcement resides in invisible constraints; operators should restrict visible constraints to what agents need for self-governance.

---

## Evolving Threat Landscape

- Multimodal processing introduces new injection surfaces (images, audio, video)
- Agent-to-agent attacks remain largely theoretical
- New capabilities create nonlinear interaction effects with existing ones
- Enforcement supply chain (compromised proxy images, gateway vulnerabilities) undermines the entire model — production deployments require signed images, version pinning, vulnerability scanning
- Mediation network security assumed trusted; enterprise-scale deployments spanning multiple hosts require mTLS, network policy, component authentication
- Model-level attacks (adversarially fine-tuned LLMs) not addressed by runtime architecture
- Monitoring (security monitor) is itself an XPIA target — consumes adversary-influenced data

Defense-in-depth architecture enables resilience as threats evolve, but the framework explicitly acknowledges incompleteness.
