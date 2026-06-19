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

The plugin handles running the MCP server. Configure auth once before calling
LimaCharlie tools.

## Auth

You need two values from LimaCharlie: an organization ID and an organization
API key.

1. Open LimaCharlie and choose your organization.
2. Copy the org ID from the URL: `app.limacharlie.io/orgs/<org-id>/...`.
3. Go to `Organization Settings` -> `Access Management` -> `REST API`.
4. Click `Create API Key`, create a key for this MCP, and copy the secret when
   LimaCharlie shows it.
5. Run this and paste the API key into the hidden prompt:

```bash
uvx --from git+https://github.com/geoffbelknap/limacharlie-mcp \
  limacharlie-mcp-configure \
  --oid "paste-your-org-id-here"
```

Then start a new Codex or Claude chat with the plugin enabled and call
`lc_auth_status`.

## Recommended First Run

After auth setup, start a new Codex or Claude chat with the plugin enabled.
Use `limacharlie-review` first and call:

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
