# LimaCharlie MCP

Standalone LimaCharlie MCP suite for direct API-backed investigation,
administration, review, and incident workflows.

## What It Includes

- Focused MCP profiles: core, fleet, admin, content, detect, contain, evict,
  recover, and review.
- Managed local Vault auth and reauth flows from
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

By default, setup uses managed local Vault so the long-lived LimaCharlie API
key stays out of chat history, `.env` files, MCP client configuration, and
audit logs. The MCP uses that protected key to mint short-lived LimaCharlie
JWTs when tools need API access.

## Auth

You need two values from LimaCharlie: an organization ID and an organization
API key.

1. Open LimaCharlie and choose your organization.
2. Copy the org ID from the URL: `app.limacharlie.io/orgs/<org-id>/...`.
3. Go to `Organization Settings` -> `Access Management` -> `REST API`.
4. Click `Create API Key` and select permissions for the workflows you want.

   For first run plus read-only posture review, start with:

   ```text
   org.get
   sensor.list
   sensor.get
   insight.list
   insight.det.get
   insight.evt.get
   insight.stat
   audit.get
   output.list
   dr.list
   dr.list.managed
   fp.ctrl
   yara.get
   lookup.get
   ikey.list
   ingestkey.ctrl
   user.ctrl
   apikey.ctrl
   job.get
   replicant.get
   replicant.task
   ```

   `replicant.task` is needed for complete service-backed content review, such
   as listing rules managed through LimaCharlie services.

   Add mutation permissions only when you intend to use response, admin, or
   content-editing workflows. Do not add `live_stream.ctrl`; this MCP does not
   expose live firehose or streaming telemetry tools.
5. Create the key for this MCP, and copy the secret when
   LimaCharlie shows it.
6. Run this and paste the API key into the hidden prompt:

```bash
uvx --from git+https://github.com/geoffbelknap/limacharlie-mcp \
  limacharlie-mcp-configure \
  --oid "paste-your-org-id-here"
```

Then start a new Codex or Claude chat with the plugin enabled and ask:
"Check my LimaCharlie MCP auth status." The agent should confirm credentials
are configured without showing secrets.

## Recommended First Run

After auth setup, start a new Codex or Claude chat with the plugin enabled.
Use the review profile first and ask:

1. "Check my LimaCharlie MCP auth status."
2. "Show me which LimaCharlie MCP profile and tools are available."
3. "Review my LimaCharlie org posture."

Use narrower profiles for operational work:

- `limacharlie-detect` for triage.
- `limacharlie-contain` for guarded containment previews.
- `limacharlie-evict` for adversary eviction workflows.
- `limacharlie-recover` for post-incident restoration and verification.

The MCP intentionally does not expose live telemetry firehose/spout behavior.
Use bounded reads, explicit windows, cursors, and LimaCharlie outputs for
operational telemetry pipelines.
