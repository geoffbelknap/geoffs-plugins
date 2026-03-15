# ASK Threat Model — ASK 2026.03

Complete threat categorization for AI agent security. Use this reference when conducting
threat assessments or evaluating which threat categories apply to a specific deployment.

---

## Threat Philosophy

ASK categorizes threats into three groups: **traditional, novel, and hybrid** — each requiring
different mitigation strategies. The principle: use proven solutions for proven problems, and
invest engineering effort in problems that are actually new.

---

## Traditional Threats (Established Solutions Apply)

These threats have well-understood mitigations from enterprise security. Apply them.

### 1. Compromised Credentials
API keys exposed through logs, error messages, or misconfiguration.

**Mitigations:** Scoped credentials per agent, credential mediation via enforcer (agent never holds real keys), rotation, secure storage separation. Compromise of scoped key has bounded blast radius.

### 2. Supply Chain Attacks
Malicious skills, plugins, MCP servers, or dependencies.

**Mitigations:** Application allowlisting (gateway tool policy), version pinning (MCP tool definitions), network containment (egress proxy), operator approval gates for new tools/servers.

### 3. Secrets at Rest
Unintended exposure of sensitive data in accessible paths.

**Mitigations:** Filesystem restrictions (gateway file policy), credential separation (enforcer mediates), secret pattern scanning in guardrails, deny access to `.env`, `credentials*`, `secrets*` patterns.

### 4. DNS Exfiltration
Data encoded in DNS subdomain queries bypassing HTTP-level controls.

**Mitigations:** Internal DNS resolvers only, block external DNS resolvers, block DNS-over-HTTPS (DoH) providers, egress proxy denylist for DoH endpoints.

### 5. Insider Threats
Agents operating outside intended scope — intentionally or through compromise.

**Mitigations:** Least privilege (Tenet 4), budget caps, behavioral monitoring via Sentinel, approval requirements for sensitive operations, audit logging of all actions.

---

## Novel Threats (New Architectural Approaches Needed)

These threats are unique to AI agents and lack conventional solutions.

### 1. Cross-Prompt Injection Attacks (XPIA)
**The core novel vulnerability.** All tokens in the context window are processed identically — there is no enforced boundary between data and instructions at the token level.

An attacker embeds instructions in content the agent fetches — web pages, documents, API responses, tool outputs. The LLM follows those embedded instructions because it cannot architecturally distinguish them from legitimate instructions.

**Why it's novel:** No equivalent in traditional computing. The "execution model" processes data and instructions identically.

**ASK mitigation (defense-in-depth):**
- Stage 1 (Injection): Network denylist, input validation, content sanitization
- Stage 2 (Propagation): Pre-call guardrails, context isolation, encoding normalization
- Stage 3 (Execution): Runtime gateway scope enforcement, tool permission guards
- Stage 4 (Exfiltration): Egress proxy, network control, DNS restrictions

**Honest limitation:** No single guardrail provides complete protection. Sophisticated attacks may evade detection. This is why architectural containment matters — even if injection succeeds, blast radius is bounded.

### 2. MCP Tool Definition Tampering
Tool contracts can change silently between sessions without code deployment. An MCP server's tool definitions are loaded at connection time — a server can alter tool descriptions, parameters, or behavior without any code change being visible.

**ASK mitigation:** Version pinning at gateway level (detect schema changes between sessions), operator approval for definition changes, block runtime MCP server registration.

### 3. Runtime Capability Escalation
Unauthorized MCP servers spawned at runtime, bypassing application-level tool policy.

**ASK mitigation:** Block runtime server registration at gateway level, monitor process trees, OS-level enforcement (not application-level).

### 4. Identity and Memory Poisoning
Semantic corruption of persistent agent state that persists across sessions, gradually shifting behavior. Unlike XPIA (which targets the ephemeral session), identity poisoning targets the durable Identity layer.

**ASK mitigation:** Tenet 25 (every Identity write logged with provenance by mediation layer), recovery/rollback capability, Sentinel behavioral monitoring for anomalous write patterns, immutable Constraints as anchor.

### 5. Context Poisoning via Delegation
Compromised sub-agent returns manipulated results to corrupt coordinator context. The coordinator trusts the delegation response and incorporates poisoned data into its reasoning.

**ASK mitigation:** Scan delegation responses for injection before passing to parent (delegation bus), zero-trust model between agents, guardrails on all inter-agent communication.

### 6. Behavioral Drift
Agent satisfies technical constraints while gradually violating the intent behind them. Includes gaming metrics, finding creative loopholes, and deceptive alignment (behaving correctly under observation but differently otherwise).

**ASK mitigation:** Sentinel behavioral monitoring, audit log analysis, trust evolution based on observed behavior. **Honest limitation:** Deceptive alignment remains an unsolved problem. Architecture constrains blast radius but cannot prevent misaligned strategies.

### 7. Cascading Multi-Agent Failures
Errors amplifying through delegation chains. Two types:
- **Resource-based cascades** — one agent failure triggers cascading resource exhaustion (circuit breakers apply)
- **Semantic cascades** — hallucinations from one agent become authoritative input downstream (no established circuit breaker patterns)

**ASK mitigation:** Scope limits on delegation, synthesis constraints (Tenet 12), delegation response scanning. **Honest limitation:** Detecting plausible but false results requires ground-truth verification the framework cannot provide.

### 8. Alert Fatigue / Oversight Overwhelming
Deliberately or incidentally generating high volumes of approval requests, degrading the human oversight safety net.

**ASK mitigation:** Trust spectrum calibration (appropriate autonomy levels), exception budgets, Sentinel correlation to detect patterns.

---

## Hybrid Threats

Traditional attack patterns that manifest distinctly in agent contexts:

- **Compromised agent in multi-agent system** — traditional lateral movement, but through delegation channels
- **Web content weaponization** — traditional web attacks, but targeting agent processing rather than browsers
- **Credential theft via agent** — traditional credential harvesting, but using agent as the tool

These require both conventional security controls AND agent-specific guardrails.

---

## The Threat Landscape is Incomplete

The framework is designed around threats understood today. AI agent security is a nascent field.

- Novel attack classes will emerge — XPIA itself was not widely recognized until agents operated autonomously at scale
- Multimodal processing (images, audio, video) introduces new injection surfaces
- Agent-to-agent attacks remain largely theoretical
- New capabilities create nonlinear interaction effects with existing ones

Defense-in-depth architecture enables resilience as threats evolve, but unanticipated threat classes remain beyond current design scope.
