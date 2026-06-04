---
name: ax-optimization-review
description: Review existing CLIs, MCP servers, API wrappers, or agent tool surfaces for Agent Experience (AX); produce implementation findings, maturity scores, refactor roadmaps, and trajectory tests. Use when the user asks to optimize a CLI/MCP for agents, audit tool schemas/results/errors, create AX review reports, or turn human-oriented tooling into agent-friendly interfaces.
---

# AX Optimization Review

Use this skill to review and improve existing CLIs, MCP servers, and related
tool surfaces so agents can use them reliably. The goal is implementation
guidance, not abstract principles.

## Core Standard

After every tool call, an agent should be able to answer:

1. What happened?
2. What changed?
3. What evidence proves it?
4. What should I do next?

If the interface does not provide enough information to answer those questions,
produce a concrete finding and refactor recommendation.

## Safety Rules

- Prefer static review first: source, schemas, docs, help text, tests, and sample outputs.
- Run non-destructive commands before any mutating command.
- Do not run mutating commands unless the user explicitly allows it or the repo has a safe isolated test fixture.
- Treat approval, policy, credentials, destructive actions, and external side effects as first-class review material.
- If live execution is unsafe or unavailable, produce a static review and state what runtime evidence is missing.

## Review Inputs

Gather what exists:

- CLI source code and command definitions.
- MCP server source code and tool definitions.
- Command help output.
- MCP tool list and JSON schemas.
- Example success and error outputs.
- Existing docs, tutorials, and tests.
- Target workflows the agent must complete.
- Known agent failure transcripts.
- Security, approval, or governance requirements.

If the user gives a target path, inspect local repo instructions first, then
use `rg --files`, `rg`, and focused file reads to find CLI/MCP implementation
points.

## Review Procedure

1. Identify the tool surface.
2. Inventory operations.
3. Map 3-7 realistic agent workflows.
4. Review discovery and documentation.
5. Review input schemas.
6. Review output contracts.
7. Review error contracts.
8. Review state, lifecycle, identity, and correlation.
9. Review side effects, cleanup, policy, and approval handling.
10. Review CLI agent mode and/or MCP tool quality.
11. Score AX maturity.
12. Produce findings, refactor roadmap, and trajectory tests.

Do not start with cosmetic documentation edits. First identify where the
interface contract prevents reliable agent execution.

## Operation Inventory

For each important command/tool, capture:

- Operation or command name.
- Resource type.
- Action type: read, create, update, delete, execute, approve, export, import, search, validate.
- Required inputs.
- Optional inputs.
- Output format.
- Error format.
- Side effects.
- Idempotency.
- Preconditions.
- Postconditions.
- Cleanup or rollback path.
- Whether common agent workflows need it.

Flag broad operations such as `execute`, `admin`, `run`, `call_api`, `do_task`,
or `manage`. Broad operations are not automatically wrong, but they are high
risk because the model must infer semantics.

## Workflow Mapping

Review against realistic trajectories, not isolated commands. Include:

- Discovery: find the right operation and required inputs.
- Happy path: complete a simple task end to end.
- Multi-resource tracking: operate on similar resources without mixing identities.
- State query: inspect state before choosing an action.
- Error recovery: encounter an expected failure and recover.
- Async readiness: wait, poll, or inspect until a job finishes.
- Cleanup: remove or revert created resources.
- Unsupported request: stop cleanly without faking success.
- Policy boundary: request approval or stop when authority is insufficient.

For each workflow, define initial state, user task, expected operation classes,
required evidence, final state, cleanup requirements, and failure conditions.

## Review Checks

### Discovery

Check for compact operation catalog, machine-readable schema export, parseable
help, operational MCP descriptions, permissions per operation, capability
flags, version info, backend differences, and known unsupported capabilities.

Common fixes: add `tool.capabilities`, `tool.version`, `tool.catalog`,
`--help --output json`, schema export, exact examples, and explicit unsupported
capability docs.

### Input Schemas

Check for required fields, optional fields, defaults, enums, accepted formats,
bounds, mutually exclusive fields, sensitive fields, selector versus payload
fields, dry-run flags, and idempotency keys.

Common fixes: replace free-form strings with structured fields, add enum
validation, add explicit selector types, add dry-run for mutations, add
idempotency keys.

### Output Contracts

A strong result envelope includes:

```json
{
  "ok": true,
  "operation": "resource.action",
  "request_id": "req_123",
  "resource": {
    "type": "resource_type",
    "id": "res_123",
    "name": "optional human name"
  },
  "state": {
    "previous": "old_state",
    "current": "new_state"
  },
  "data": {},
  "side_effects": [],
  "warnings": [],
  "observed_at": "2026-06-04T20:00:00Z"
}
```

Check for operation name, request id, resource id, resource name, current
state, previous state, structured data, warnings, side effects, timestamp,
pagination/truncation metadata, and artifact links.

Common fixes: add a consistent envelope, preserve human output separately from
agent output, add `--output json`, normalize MCP results, and use artifact
references for large outputs.

### Error Contracts

A strong error envelope includes:

```json
{
  "ok": false,
  "operation": "resource.create",
  "request_id": "req_123",
  "error": {
    "code": "invalid_resource_selector",
    "class": "input",
    "message": "No resource matched selector `prod-db-primaryy`.",
    "retryable": false,
    "same_input_retryable": false,
    "suggested_next_actions": [
      "List available resources.",
      "Retry with a valid resource id."
    ]
  },
  "side_effects": [],
  "observed_at": "2026-06-04T20:00:00Z"
}
```

Recommended error classes: `input`, `auth`, `policy`, `not_found`, `conflict`,
`transient`, `capacity`, `unsupported`, `internal`.

Check for code, class, retryability, same-input retryability, partial success,
side effects before failure, safe next actions, and documentation references.

Common fixes: map low-level errors into stable codes, add retryability
metadata, preserve raw logs as diagnostics, and return `unsupported` for
capability boundaries.

### State, Lifecycle, And Identity

Check for resource states, legal transitions, preconditions, postconditions,
readiness boundaries, terminal states, async job ids, polling/wait operations,
cleanup/rollback operations, stable ids, parent scope, provider/backend,
actor/principal, request id, correlation id, and created-by/run id.

Common fixes: add `resource.inspect`, `resource.wait`, terminal state metadata,
transition errors such as `invalid_state_transition`, stable ids everywhere,
and cleanup verification.

### Side Effects, Cleanup, And Policy

Check mutating operations for side-effect lists, created/updated/deleted
resources, notifications, files written, jobs started, credentials created,
approval requests, cleanup obligations, dry-run support, rollback support, and
policy states.

Common fixes: add `side_effects`, `cleanup_handles`, `--dry-run`/`plan`,
idempotent cleanup operations, cleanup verification checks, and explicit
approval/policy result states.

### CLI Agent Mode

For CLIs, check for non-interactive mode, no spinner/progress/ANSI output,
structured stdout, stable exit codes, bounded stderr, machine-readable errors,
quiet mode, and schema export.

Common fixes: add `--output json`, `--quiet`, `--non-interactive`, safe
confirmation handling, and structured diagnostics.

### MCP Tool Quality

For MCP servers, check that tool names are specific, descriptions are
operational, input schemas are complete, required context is explicit, results
and errors are structured, large outputs are summarized/linked, side effects
are visible, unsafe actions require explicit scope, and tool names are stable.

Common fixes: normalize CLI output before returning it through MCP, split broad
tools, add structured schemas/envelopes, add pagination/artifact references,
and add explicit policy states.

## Scoring Rubric

Score each applicable area from 0 to 3:

- 0: absent or hostile to agents.
- 1: partially present but brittle.
- 2: usable with gaps.
- 3: strong and reliable.

Areas:

- Discovery and documentation.
- Operation design.
- Input schemas.
- Output schemas.
- Error semantics.
- State and lifecycle.
- Identity and correlation.
- Side effects and cleanup.
- CLI agent mode, if applicable.
- MCP tool quality, if applicable.
- Policy and approval handling.
- Trajectory test coverage.

Interpretation:

- 0-10: human-only or agent-hostile.
- 11-20: basic structured output, still brittle.
- 21-28: usable AX with important gaps.
- 29-34: strong AX for most workflows.
- 35+: excellent AX, ready for delegated agent workflows.

Do not penalize a CLI-only tool for lacking MCP, or an MCP-only tool for
lacking CLI, unless the product goal requires that missing surface.

## Finding Format

Write findings as implementation tasks:

- Severity: critical, high, medium, low.
- Area: discovery, schema, output, error, lifecycle, identity, cleanup, CLI, MCP, policy, eval.
- Evidence: command, tool, output snippet, schema field, or workflow failure.
- Agent impact: what the agent cannot know or do reliably.
- Recommended fix: concrete implementation change.
- Acceptance test: how to verify the fix.

Example:

Severity: high.
Area: error.
Evidence: `deploy start --output json` returns `{ "success": false, "message": "failed" }` when the target environment is locked.
Agent impact: the agent cannot distinguish policy lock from transient failure and may retry repeatedly.
Recommended fix: return `error.class: policy`, `error.code: environment_locked`, lock owner, approval path, and `same_input_retryable: false`.
Acceptance test: invoking deployment against a locked environment returns the policy error envelope and the agent stops or requests approval.

## Report Shape

Produce:

1. Executive summary.
2. AX maturity score.
3. Tool surface inventory.
4. Workflow coverage summary.
5. Top findings by severity.
6. Detailed findings.
7. Refactor roadmap.
8. Suggested schemas or envelopes.
9. Suggested trajectory tests.
10. Residual risks.

Keep the report implementation-oriented. The reader should know what to change
first.

## Refactor Roadmap

Prefer this order:

1. Make outputs readable by agents: structured mode, separate logs, operation/request/resource ids.
2. Make failures actionable: error classes/codes, retryability, safe next actions, unsupported responses.
3. Make workflows reliable: inspect/list, readiness/wait, lifecycle states, transition errors, side effects.
4. Make cleanup and safety first-class: cleanup handles, dry-run/plan, idempotency keys, approval states.
5. Prove AX with trajectories: baseline, recovery, multi-resource, unsupported request, cleanup, policy tests.

## Trajectory Test Template

```json
{
  "id": "workflow_error_recovery_example",
  "category": "error_recovery",
  "initial_state": "A target resource does not exist.",
  "prompt": "Create or update the target safely, recover if the first selector is invalid, and report the final resource id.",
  "allowed_tools": ["resource.list", "resource.create", "resource.update", "resource.inspect"],
  "success_criteria": [
    "Agent identifies the invalid selector.",
    "Agent lists or inspects resources before retrying.",
    "Agent completes the workflow with a valid resource id.",
    "Agent reports evidence from tool results."
  ],
  "failure_criteria": [
    "Agent retries same invalid selector repeatedly.",
    "Agent claims success without evidence.",
    "Agent creates duplicate resources unnecessarily."
  ],
  "cleanup": ["Delete resources created during the run."],
  "metrics": ["tool_calls", "tokens", "wall_time", "retries", "residue"]
}
```
