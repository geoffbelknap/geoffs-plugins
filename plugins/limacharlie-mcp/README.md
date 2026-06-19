# LimaCharlie MCP

Standalone LimaCharlie MCP suite for direct API-backed investigation,
administration, review, and incident workflows.

## What It Includes

- Focused MCP profiles: core, fleet, admin, content, detect, contain, evict,
  recover, and review.
- Vault-first auth and reauth flows from
  [geoffbelknap/limacharlie-mcp](https://github.com/geoffbelknap/limacharlie-mcp).
- Read-only review/tuning aggregate tools for posture review, fleet health,
  detection noise, content coverage, case backlog, output health, and access
  hygiene.
- Skills for auth onboarding, posture review, detection tuning, detect triage,
  containment, eviction, and recovery verification.

## Install

```bash
/plugin install limacharlie-mcp@geoffs-plugins
```

The MCP servers use `uvx` to run the published GitHub package:

```bash
uvx --from git+https://github.com/geoffbelknap/limacharlie-mcp limacharlie-mcp-review
```

## Auth

Use Vault-backed credentials for deployment. Configure the runtime environment
with:

- `LC_SECRET_PROVIDER=vault`
- `LC_VAULT_ADDR`
- `LC_VAULT_TOKEN_FILE`
- `LC_API_KEY_REF`

For local development only, the underlying MCP also supports
`LC_SECRET_PROVIDER=env` with `LC_API_KEY`. Do not paste production LimaCharlie
API keys into plugin configs or commit them to this repository.

## Recommended First Run

Start with `limacharlie-review` and call:

1. `lc_auth_status`
2. `lc_tool_catalog`
3. `lc_review_org_posture`

Use narrower profiles for operational work:

- `limacharlie-detect` for triage.
- `limacharlie-contain` for guarded containment previews.
- `limacharlie-evict` for adversary eviction workflows.
- `limacharlie-recover` for post-incident restoration and verification.

The MCP intentionally does not expose live telemetry firehose/spout behavior.
Use bounded reads, explicit windows, cursors, and LimaCharlie outputs for
operational telemetry pipelines.
