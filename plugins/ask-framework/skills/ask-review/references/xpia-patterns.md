# XPIA Attack Patterns & Defensive Architectures

Cross-Plugin/Indirect Injection Attack (XPIA) patterns and ASK-compliant defenses.
Read this file when analyzing XPIA kill chain posture or designing input safety controls.

---

## Table of Contents

1. [XPIA Kill Chain Deep Dive](#xpia-kill-chain-deep-dive)
2. [Attack Patterns by Stage](#attack-patterns-by-stage)
3. [Defensive Architecture Patterns](#defensive-architecture-patterns)
4. [Detection Strategies](#detection-strategies)
5. [Common Misconfigurations](#common-misconfigurations)

---

## XPIA Kill Chain Deep Dive

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  1. INJECTION │───▶│2. PROPAGATION│───▶│ 3. EXECUTION │───▶│4. EXFILTRATION│
│               │    │              │    │              │    │              │
│ Malicious     │    │ Payload      │    │ Agent acts   │    │ Data leaves  │
│ content       │    │ reaches      │    │ on injected  │    │ via agent's  │
│ enters system │    │ the agent    │    │ instructions │    │ action scope │
└──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
     ▲ DEFEND            ▲ DEFEND            ▲ DEFEND            ▲ DEFEND
     │ HERE               │ HERE              │ HERE              │ HERE
     Guardrails           Guardrails          Gateway             Egress Proxy
     Input validation     Context isolation   Scope enforcement   Network control
```

**ASK requires defense at stages 1–3, not just stage 4.**
Defending only at exfiltration is insufficient — the agent has already been compromised.

---

## Attack Patterns by Stage

### Stage 1: Injection

Malicious instructions enter the system through legitimate-looking input channels.

| Pattern | Vector | Example |
|---|---|---|
| **Document injection** | Uploaded files, PDFs, spreadsheets | Hidden text in white-on-white, PDF metadata, invisible Unicode |
| **Web content injection** | Scraped pages, API responses | Injected instructions in HTML comments, JSON fields, meta tags |
| **Tool result injection** | MCP server responses, API results | Compromised or malicious MCP server returns instructions |
| **Memory/context injection** | Conversation history, RAG results | Poisoned vector DB entries, manipulated conversation logs |
| **Email injection** | Email body, subject, attachments | Instructions embedded in email content agent processes |
| **Image injection** | OCR'd images, screenshots | Instructions rendered in images the agent reads |
| **Orchestrator relay** | Multi-agent message passing | Agent A sends poisoned output that Agent B treats as trusted |

### Stage 2: Propagation

The injected payload reaches the agent's context window without being filtered.

| Pattern | Failure Mode |
|---|---|
| **No guardrail layer** | Raw external content goes directly to agent prompt |
| **Post-agent guardrails** | Guardrails run after agent has already processed input |
| **Incomplete coverage** | Guardrails on chat input but not on tool results |
| **Guardrail bypass** | Encoding tricks, language switching, prompt fragmentation |
| **Trust inheritance** | Content from "trusted" source skips guardrails |

### Stage 3: Execution

The agent follows injected instructions, executing unintended actions.

| Pattern | Failure Mode |
|---|---|
| **Scope escalation** | Agent calls tools not in its declared scope |
| **Prompt override** | Injected content overrides system prompt constraints |
| **Tool chaining** | Injected instructions chain multiple tool calls |
| **File write abuse** | Agent writes malicious content to accessible paths |
| **Configuration tampering** | Agent modifies its own config through writable paths |

### Stage 4: Exfiltration

Data is extracted from the system through the agent's action surface.

| Pattern | Vector |
|---|---|
| **Direct network egress** | Agent makes HTTP call to attacker-controlled server |
| **Tool-mediated egress** | Agent uses email/Slack/webhook tool to send data |
| **File-mediated egress** | Agent writes sensitive data to shared/public path |
| **Encoding egress** | Data hidden in legitimate-looking outputs (steganography) |
| **Multi-hop egress** | Data passed through agent chain to one with network access |

---

## Defensive Architecture Patterns

### Pattern 1: Pre-Agent Guardrail Pipeline

```
External Input ──▶ [Guardrail Pipeline] ──▶ Agent Context
                        │
                   ┌────┴────┐
                   │ Checks: │
                   │ • Injection detection (classifier)
                   │ • Content sanitization
                   │ • Schema validation
                   │ • Encoding normalization
                   │ • Size limits
                   └─────────┘
```

**Applies to:** Tool results, document content, API responses, user input
**ASK tenets:** 13 (input guardrails), 14 (kill chain interruption)

### Pattern 2: Gateway Scope Lock

```
Agent ──action──▶ [Enforcer] ──validate──▶ [Gateway] ──execute──▶ Tool
                      │                        │
                 Check Mind:              Check allowlist:
                 • Is action in scope?    • Is tool allowed?
                 • Rate limits ok?        • Are params valid?
                 • Session limits ok?     • Path constraints met?
```

**Applies to:** Every tool call, file operation, command execution
**ASK tenets:** 1 (enforcement separation), 5 (least privilege), 7 (default-deny gateway)

### Pattern 3: Egress Containment

```
Agent Environment
┌──────────────────────────┐
│                          │
│  Agent ──network──▶ ✗    │  (no direct outbound)
│                          │
│  Agent ──▶ Gateway ──▶ Egress Proxy ──▶ Internet
│                          │    │
│                          │    ├─ Allowlist check
│                          │    ├─ Method/path check
│                          │    └─ Payload inspection
└──────────────────────────┘
```

**Applies to:** All outbound network traffic
**ASK tenets:** 6 (egress mediation), 14 (exfiltration defense)

### Pattern 4: Restricted Context Processing

When the agent must handle untrusted content, constrain its action surface:

```
Normal mode:  Agent has tools A, B, C, D, E
                     │
                     ▼
Untrusted input arrives
                     │
                     ▼
Restricted mode: Agent has tools A, B only (read-only)
                     │
                     ▼
Processing complete, output sanitized
                     │
                     ▼
Normal mode restored: Agent has tools A, B, C, D, E
```

**ASK tenet:** 15 (restricted context for untrusted content)

---

## Detection Strategies

### Classifier-Based Detection

Deploy a separate model or classifier to score inputs for injection risk:

```yaml
guardrail:
  type: injection_classifier
  model: dedicated-classifier-v2
  threshold: 0.85
  action_on_detect: block_and_log
  fallback_on_error: block  # Fail closed
```

**Important:** The classifier must run **outside** the agent's process.
Running it inside the agent violates tenet 1.

### Heuristic Detection

Pattern-based checks that catch common injection attempts:

- Instruction-like phrases in non-instruction contexts
- Role/persona switching language ("you are now...", "ignore previous...")
- Base64/hex encoded blocks in text content
- Unusual Unicode characters (zero-width joiners, RTL overrides)
- Dramatic tone shifts within a single input

### Canary Token Detection

Plant known tokens in sensitive data; detect if they appear in agent outputs:

```yaml
canary:
  tokens:
    - context: "customer_database"
      token: "CANARY-DB-7f3a2b"
      alert_on: output_contains
    - context: "internal_api_key"
      token: "CANARY-KEY-9d1c4e"
      alert_on: output_contains
```

---

## Common Misconfigurations

### ❌ Guardrails after the agent

```
Input ──▶ Agent ──▶ [Guardrail] ──▶ Output
              ^
              └── Agent already processed injection. Too late.
```

### ❌ Guardrails inside the agent process

```
Agent Process
┌─────────────────────┐
│ [Guardrail] ──▶ LLM │  ← Agent can bypass or disable guardrail
└─────────────────────┘
```

### ❌ Tool results bypass guardrails

```
User Input ──▶ [Guardrail] ──▶ Agent ──▶ Tool Call ──▶ Tool Result ──▶ Agent
                                                          ^
                                                          └── No guardrail here!
```

### ✅ Correct: All inputs through guardrails

```
User Input ──▶ [Guardrail] ──▶ Agent
Tool Result ──▶ [Guardrail] ──▶ Agent
Document ──▶ [Guardrail] ──▶ Agent
API Response ──▶ [Guardrail] ──▶ Agent
```
