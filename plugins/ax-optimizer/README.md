# AX Optimizer Plugin

Claude Code plugin for reviewing and refactoring CLIs, MCP servers, and other
agent-facing tool surfaces so they are optimized for Agent Experience (AX).

## What It Does

- **AX reviews** — Inspect existing CLIs, MCP servers, API wrappers, and tool surfaces
- **Maturity scoring** — Score discovery, schemas, outputs, errors, lifecycle, cleanup, policy, and tests
- **Implementation findings** — Produce severity-ranked findings with evidence, impact, fixes, and acceptance tests
- **Refactor roadmaps** — Order improvements by agent impact and implementation leverage
- **Trajectory tests** — Design end-to-end evals for happy path, recovery, cleanup, unsupported requests, and policy boundaries
- **Plugin/skill readiness** — Structure findings so they can feed repeatable agent workflows

## Skills

### `ax-optimization-review`

Implementation-oriented reviewer. Loaded when Claude detects work involving
Agent Experience optimization for CLIs, MCP servers, API wrappers, tool schemas,
structured output, error contracts, lifecycle semantics, cleanup behavior, or
trajectory evaluation.

The skill follows a repeatable workflow:

1. Identify the tool surface.
2. Inventory operations.
3. Map realistic agent workflows.
4. Review discovery, schemas, outputs, errors, state, identity, lifecycle, side effects, cleanup, policy, CLI behavior, and MCP tool quality.
5. Score AX maturity.
6. Produce findings, a refactor roadmap, and trajectory tests.

## Install

```bash
/plugin install ax-optimizer@geoffs-plugins
```
