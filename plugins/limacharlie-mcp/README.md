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

You need two values from LimaCharlie: an organization ID and a temporary
bootstrap API key.

1. Open LimaCharlie, log in, and choose your organization.
2. Copy the org ID from the URL: `app.limacharlie.io/orgs/<org-id>/...`.
3. Open a terminal on the host running your MCP, swap in your org ID where it
   says `paste-your-org-id-here`, and run this:

```bash
uvx --from git+https://github.com/geoffbelknap/limacharlie-mcp \
  limacharlie-mcp-configure \
  --oid "paste-your-org-id-here" \
  --provision-runtime-key
```

4. The command will print a temporary bootstrap key name and stop at a hidden
   `LimaCharlie API key secret` prompt. Leave it waiting there.
5. Go back to your browser and head to `Organization Settings` -> `Access
   Management` -> `REST API`.
6. Click `Create API Key`, name it exactly what the command printed, and give it only:

   ```text
   org.get
   apikey.ctrl
   ```

   The setup command uses this temporary key to create one dedicated runtime key
   named `limacharlie-mcp-runtime`, stores that runtime key in local Vault,
   and verifies it. It does not print either secret. Do not add
   `live_stream.ctrl`; this MCP does not expose live firehose or streaming
   telemetry tools.
7. Create your bootstrap key and copy the secret from the LimaCharlie dashboard.
8. Switch back to the terminal and paste the secret into the hidden prompt.
9. After setup verifies the runtime key, delete the printed bootstrap key from
   LimaCharlie. The runtime key is already stored in Vault.

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
