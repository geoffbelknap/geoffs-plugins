# ASK Configuration Templates

Canonical configuration templates for ASK framework elements.
Read this file when generating or reviewing Mind, Gateway, Egress, or Enforcer configs.

---

## Table of Contents

1. [Mind Configuration](#mind-configuration)
2. [Gateway Configuration](#gateway-configuration)
3. [Egress Proxy Configuration](#egress-proxy-configuration)
4. [Enforcer Configuration](#enforcer-configuration)
5. [Audit Log Configuration](#audit-log-configuration)
6. [Composition Example](#composition-example)

---

## Mind Configuration

The Mind is the agent's constraints definition — what tier it operates at, which models it can use,
and what behavioral boundaries apply. It is a **read-only mount**; the agent cannot modify it.

```yaml
# mind.yaml — Agent constraints configuration
apiVersion: ask/v1
kind: Mind
metadata:
  name: research-agent-mind
  agent_id: agent-research-001
  version: "1.0.0"

spec:
  tier: restricted          # restricted | standard | elevated | privileged
  description: "Research agent for document analysis and summarization"

  models:
    allowed:
      - claude-sonnet-4-20250514
    max_context_window: 100000
    max_output_tokens: 4096

  behaviors:
    allowed_actions:
      - read_file
      - search_documents
      - summarize
      - respond_to_user
    denied_actions:
      - write_file
      - execute_code
      - send_email
      - access_network
    max_tool_calls_per_turn: 10
    max_turns: 50
    require_human_approval:
      - any_action_outside_scope

  data_access:
    allowed_paths:
      - /data/documents/**
      - /data/research/**
    denied_paths:
      - /data/credentials/**
      - /data/private/**
      - /etc/**
    allowed_schemas: []
    denied_schemas: []

  constraints:
    no_pii_in_output: true
    no_external_urls: true
    citation_required: true
```

### Mind Tiers

| Tier | Description | Typical Scope |
|---|---|---|
| `restricted` | Read-only, no side effects | Analysis, summarization, Q&A |
| `standard` | Limited writes, scoped tools | Document editing, data entry |
| `elevated` | Broader tool access, some network | API calls, integrations |
| `privileged` | Wide access, requires strong enforcement | DevOps, admin tasks |

**Key invariant:** Higher tiers demand stricter Gateway, Egress, and Enforcer configs — not looser ones.

---

## Gateway Configuration

The Gateway mediates all tool calls, commands, file access, and MCP interactions.
Default posture is **deny-all**; only explicitly allowed operations pass.

```yaml
# gateway.yaml — Runtime tool/command control
apiVersion: ask/v1
kind: Gateway
metadata:
  name: research-agent-gateway
  agent_id: agent-research-001

spec:
  default_action: deny

  tools:
    allowed:
      - name: read_file
        constraints:
          paths: ["/data/documents/**", "/data/research/**"]
          max_size_bytes: 10485760  # 10MB
      - name: search_documents
        constraints:
          index: "research-corpus"
          max_results: 50
      - name: summarize
        constraints:
          max_input_tokens: 50000
    denied:
      - name: "*"  # Deny everything not explicitly allowed

  commands:
    allowed: []
    denied:
      - pattern: "*"

  mcp_servers:
    allowed: []
    denied:
      - pattern: "*"

  file_operations:
    read:
      allowed_paths: ["/data/documents/**", "/data/research/**"]
      denied_paths: ["**/*.env", "**/*.key", "**/*.pem"]
    write:
      allowed_paths: []
      denied_paths: ["**"]
    delete:
      allowed_paths: []
      denied_paths: ["**"]

  rate_limits:
    tool_calls_per_minute: 30
    file_reads_per_minute: 20
    total_actions_per_session: 500
```

### Gateway Enforcement Checklist

- [ ] Default action is `deny`
- [ ] Every allowed tool has explicit constraints
- [ ] File paths use allowlist, not blocklist
- [ ] MCP servers are explicitly listed (not wildcard allowed)
- [ ] Rate limits are set for all action types
- [ ] Write/delete operations are denied unless explicitly needed

---

## Egress Proxy Configuration

The Egress Proxy controls all outbound network traffic from the agent's environment.
The agent process cannot make direct outbound connections.

```yaml
# egress.yaml — Outbound network control
apiVersion: ask/v1
kind: EgressProxy
metadata:
  name: research-agent-egress
  agent_id: agent-research-001

spec:
  default_action: deny

  rules:
    - name: allow-internal-api
      action: allow
      destination:
        hosts: ["api.internal.company.com"]
        ports: [443]
        protocols: [https]
      constraints:
        methods: [GET]
        max_request_size_bytes: 1048576
        max_response_size_bytes: 10485760

    - name: deny-all-external
      action: deny
      destination:
        hosts: ["*"]

  dns:
    allowed_resolvers: ["10.0.0.53"]
    block_external_dns: true

  logging:
    log_all_requests: true
    log_blocked_requests: true
    include_headers: false
    include_body: false
```

### Egress Posture Levels

| Posture | Description |
|---|---|
| **No egress** | Agent has zero outbound network. Strongest posture. |
| **Allowlist only** | Specific hosts/ports/methods. Default for most agents. |
| **Denylist** | Block known-bad, allow rest. Weakest; avoid for sensitive agents. |

---

## Enforcer Configuration

The Enforcer is a per-agent policy sidecar that runs in a **separate isolation boundary**.
It validates actions against the Mind and Gateway before they execute.

```yaml
# enforcer.yaml — Per-agent policy sidecar
apiVersion: ask/v1
kind: Enforcer
metadata:
  name: research-agent-enforcer
  agent_id: agent-research-001

spec:
  isolation:
    type: sidecar_container    # sidecar_container | separate_process | separate_vm
    shared_namespaces: []       # No shared namespaces with agent
    communication: unix_socket  # unix_socket | localhost_port | grpc

  policy_source:
    type: mounted_config
    path: /etc/ask/policies/
    watch_for_changes: true

  enforcement:
    mode: enforce              # enforce | audit_only | dry_run
    on_policy_load_failure: deny_all
    on_evaluation_error: deny_all
    on_timeout: deny_all
    evaluation_timeout_ms: 5000

  hooks:
    pre_action:
      - validate_against_mind
      - validate_against_gateway
      - check_rate_limits
      - check_session_limits
    post_action:
      - log_to_audit
      - update_counters
      - check_output_constraints

  health:
    liveness_endpoint: /healthz
    readiness_endpoint: /readyz
    check_interval_seconds: 10
```

### Enforcer Invariants

1. The Enforcer process/container is **never** in the same isolation boundary as the agent.
2. The Enforcer **cannot** be restarted, reconfigured, or stopped by the agent.
3. On any failure (timeout, crash, policy error), the Enforcer **denies** the action.
4. The agent communicates with the Enforcer only through a defined interface (socket/port).

---

## Audit Log Configuration

```yaml
# audit.yaml — Immutable event logging
apiVersion: ask/v1
kind: AuditLog
metadata:
  name: research-agent-audit
  agent_id: agent-research-001

spec:
  sink:
    type: external_service     # external_service | append_only_volume | syslog
    endpoint: "https://audit.internal.company.com/v1/events"
    transport: https
    auth:
      type: service_account
      secret_ref: audit-sa-token

  events:
    capture:
      - agent_action
      - tool_call
      - file_access
      - network_request
      - policy_evaluation
      - enforcement_decision
      - session_lifecycle
    include_inputs: true
    include_outputs: true
    include_timing: true

  retention:
    min_days: 90
    immutable: true

  access_control:
    agent_can_read: false
    agent_can_write: false
    agent_can_delete: false
```

---

## Composition Example

A complete ASK-compliant deployment ties all elements together:

```
┌─────────────────────────────────────────────────┐
│                 Deployment Unit                   │
│                                                   │
│  ┌───────────┐   ┌────────────┐   ┌───────────┐ │
│  │   Agent    │──▶│  Enforcer  │──▶│  Gateway   │ │
│  │ (isolated) │   │ (sidecar)  │   │ (mediator) │ │
│  └───────────┘   └────────────┘   └───────────┘ │
│       │               │                │          │
│       ▼               ▼                ▼          │
│  ┌─────────┐   ┌──────────┐    ┌────────────┐   │
│  │  Mind    │   │ Audit Log│    │ Egress     │   │
│  │ (r/o)   │   │ (ext.)   │    │ Proxy      │   │
│  └─────────┘   └──────────┘    └────────────┘   │
└─────────────────────────────────────────────────┘
```

Every action flows: Agent → Enforcer → Gateway → (external resource)
Every event flows: (any component) → Audit Log (external sink)
