# VerifyMCP for Hermes Agent

Portable Agent Plugins v1 package. It connects Hermes to the public VerifyMCP MCP server and bundles a skill that says when to read a trust score before installing another server.

Layout, which is what Hermes loads for a portable package:

- `plugin.json`: manifest (`https://agent-plugins.org/schemas/1.0.0/plugin.schema.json`)
- `mcp.json`: one Streamable HTTP server, `verifymcp`, at `https://mcp.verifymcp.io`
- `skills/verifymcp/SKILL.md`: the same skill as `skills/verifymcp/SKILL.md` at the repository root

There is no `plugin.yaml`. That file belongs to a native Hermes plugin with a Python `register(ctx)` entrypoint. This package has no Python.

## What a call does

Every tool call is an HTTPS request to `https://mcp.verifymcp.io` with the arguments the agent passes. The server answers without a sign-in. OAuth is optional (protected-resource metadata at `https://mcp.verifymcp.io/.well-known/oauth-protected-resource`, scope `directory:read`). This package stores no credentials and declares no environment variables.

Scores, trust reports and change feeds are [CC BY-NC 4.0](https://verifymcp.io/docs/data-licence). The tool list is whatever the endpoint advertises at runtime.

## Install before the catalogue lists it

The draft entry for `NousResearch/hermes-agent` is `hermes/plugin-catalog/verifymcp.yaml` in this repository. `hermes plugins install verifymcp` works only after that entry is merged and its `sha` is a real commit.

Until then, either copy this directory to `~/.hermes/plugins/verifymcp` and enable the plugin named `verifymcp`, or add the server in `~/.hermes/config.yaml` (Streamable HTTP is the default when `url` is set and `transport` is omitted):

```yaml
mcp_servers:
  verifymcp:
    url: "https://mcp.verifymcp.io"
```

Same endpoint from the shell: `hermes mcp add verifymcp --url https://mcp.verifymcp.io`

Portable packages stay disabled until they are enabled. The Hermes developer guide is the reference for that switch.
