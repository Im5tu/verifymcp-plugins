![VerifyMCP](assets/logo-512.png)

# VerifyMCP plugins

Independent trust scores, tool surfaces and change history for MCP servers. Every server in the official MCP registry is re-scanned daily and scored 0-100 on a published rubric: supply-chain provenance, endpoint security, schema quality and stability. Scores are observations VerifyMCP can evidence, never a vendor's own attestation.

This repository holds public configuration, documentation and a skill. It does not contain the VerifyMCP service.

- Site: [https://verifymcp.io](https://verifymcp.io)
- Docs: [https://verifymcp.io/docs](https://verifymcp.io/docs)
- Scoring: [https://verifymcp.io/docs/scoring](https://verifymcp.io/docs/scoring)
- Remote rubric: [https://verifymcp.io/docs/scoring/remote-rubric](https://verifymcp.io/docs/scoring/remote-rubric)
- Package rubric: [https://verifymcp.io/docs/scoring/package-rubric](https://verifymcp.io/docs/scoring/package-rubric)
- Data licence: [https://verifymcp.io/docs/data-licence](https://verifymcp.io/docs/data-licence) ([CC BY-NC 4.0 deed](https://creativecommons.org/licenses/by-nc/4.0/))
- Agent guide: [https://verifymcp.io/llms.txt](https://verifymcp.io/llms.txt)

## MCP endpoint

Remote server: [https://mcp.verifymcp.io](https://mcp.verifymcp.io)

Streamable HTTP. Registry name `io.verifymcp/mcp`. Manifest: [https://verifymcp.io/.well-known/mcp.json](https://verifymcp.io/.well-known/mcp.json) (copied here as `server.json`). `llms.txt` states the negotiated protocol as `2026-07-28`.

Every tool works without signing in. In ChatGPT, Authentication: None is enough. OAuth is optional. RFC 9728 metadata is at [https://mcp.verifymcp.io/.well-known/oauth-protected-resource](https://mcp.verifymcp.io/.well-known/oauth-protected-resource) (scope `directory:read`, authorisation server `https://verifymcp.io`).

The directory page prints the same URL with a trailing slash. Both address this host. The snippets below use the manifest URL, with no trailing slash.

## Tools

Captured from the server's own directory page on 14 November 2026: [https://verifymcp.io/servers/io-verifymcp-mcp/mcp](https://verifymcp.io/servers/io-verifymcp-mcp/mcp). Nine tools, all readable without a credential. `identifier` is a package name, an endpoint URL, a registry name, or a verifymcp slug.

| Tool | Required | Purpose |
| --- | --- | --- |
| `list_servers` | | Filter the directory by name fragment, ecosystem, product or score. Name matching only. Returns the first 100. |
| `get_server` | `identifier` | Full trust report: score, category verdicts, tools and context-token cost, recent change history. |
| `get_server_comparison` | `identifiers` (2 to 5) | Side-by-side scores, categories, tool counts and context-token cost. |
| `get_server_alternatives` | `identifier` | Other servers in the same product, namespace or category, with scores. |
| `get_server_diagnostics` | `identifier` | Evidence behind a score: TLS, DNSSEC, authorisation, redirects, transports, provenance, install scripts, CVEs, dependency health. |
| `get_server_tools` | `identifier` | Full tool schemas when `get_server` truncated the list. |
| `get_server_install_config` | `identifier` | Install snippets. Configuration, not a safety judgement. Read `get_server` first. |
| `list_products` | | Product slugs (the tool's examples are github and notion) and how many servers cover each. |
| `list_known_malware` | | Supply-chain malware findings. `status` is `active` (default), `unverified`, `cleared` or `all`. |

Optional `list_servers` arguments: `query`, `minScore`, `maxScore`, `product`, `types`, `scored`, `sort`. Optional `component` on the single-server tools selects a channel when a server ships several.

Bands, from the scoring docs: 80 and above is high trust, 60-79 is medium, below 60 is low. A confirmed-malware verdict, an unpatched critical CVE that has a fix, or a broken or plaintext-HTTP connection caps the score at 0. Detail is in the [scoring guide](https://verifymcp.io/docs/scoring).

Per-server documents, with no key: `https://verifymcp.io/servers/{server}/{component}.md`, the same path with `.json`, and `{component}/changelog.json`.

## Install

`mcp.json` at the repository root is a generic `mcpServers` block (`type` `streamableHttp`, `url`). Clients do not all read that shape. Use the block for your client.

### Claude (Desktop and claude.ai)

Custom connectors are available on Claude, Cowork and Claude Desktop. On an individual Pro or Max plan: Customize, Connectors, Add, Add custom connector. Paste `https://mcp.verifymcp.io`. Under Authentication choose **No sign in**. Team and Enterprise owners add it under Organization settings, Connectors, Add, Custom, Web, with the same authentication choice. Steps: [Get started with custom connectors using remote MCP](https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp).

Claude Code. The server page publishes this command with the config key `io-verifymcp-mcp` and a trailing slash on the URL:

```bash
claude mcp add --transport http verifymcp 'https://mcp.verifymcp.io'
```

### ChatGPT

Add a custom MCP server from ChatGPT Plugins (plus, then Add custom MCP server). Server URL: `https://mcp.verifymcp.io`. Set authentication to **No authentication**. VerifyMCP's `llms.txt` states that Authentication: None is fine. Official guide: [Add a custom MCP server](https://developers.openai.com/api/docs/guides/custom-mcp-server).

### Cursor

Project file `.cursor/mcp.json`, or `~/.cursor/mcp.json` for every project. [Cursor MCP docs](https://cursor.com/docs/mcp).

```json
{
  "mcpServers": {
    "verifymcp": {
      "url": "https://mcp.verifymcp.io"
    }
  }
}
```

The current [deeplink reference](https://cursor.com/docs/reference/deeplinks) documents prompts, commands and rules. It does not document an MCP install deeplink, so this repository does not publish one. Install from the JSON above, or from Customize.

### VS Code

`.vscode/mcp.json`, or the user file from the command MCP: Open User Configuration. [MCP configuration reference](https://code.visualstudio.com/docs/agents/reference/mcp-configuration). `type` `http` is Streamable HTTP, with SSE as a fallback.

```json
{
  "servers": {
    "verifymcp": {
      "type": "http",
      "url": "https://mcp.verifymcp.io"
    }
  }
}
```

### Windsurf (Devin Desktop)

The English Windsurf MCP page now documents Devin Desktop. User config is `~/.config/devin/mcp_config.json` (`%APPDATA%\devin\mcp_config.json` on Windows). A URL uses Streamable HTTP unless `transport` is `sse`. Reference: [MCP configuration](https://docs.devin.ai/cli/extensibility/mcp/configuration.md).

```bash
devin mcp add -s user verifymcp https://mcp.verifymcp.io
```

```json
{
  "mcpServers": {
    "verifymcp": {
      "url": "https://mcp.verifymcp.io",
      "transport": "http"
    }
  }
}
```

No login step. The tools answer without OAuth.

### Hermes Agent

In `~/.hermes/config.yaml`. Streamable HTTP is the default for a `url` entry. [MCP config reference](https://hermes-agent.nousresearch.com/docs/reference/mcp-config-reference).

```yaml
mcp_servers:
  verifymcp:
    url: "https://mcp.verifymcp.io"
```

```bash
hermes mcp add verifymcp --url https://mcp.verifymcp.io
```

The portable plugin in `hermes/verifymcp/` is the catalogue package (`plugin.json`, `mcp.json`, bundled skill). Copying that directory to `~/.hermes/plugins/verifymcp` is the local install. `hermes plugins install verifymcp` waits on the catalogue entry in `hermes/plugin-catalog/verifymcp.yaml`, which is a draft for a later pull request to [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent). See `hermes/verifymcp/README.md`.

### Generic `mcpServers`

`mcp.json` in this repository:

```json
{
  "mcpServers": {
    "verifymcp": {
      "type": "streamableHttp",
      "url": "https://mcp.verifymcp.io"
    }
  }
}
```

The server page's own cross-client block uses `type` `http` and the key `io-verifymcp-mcp`. Check the client docs before assuming either key is read.

The server page also publishes blocks for Codex, OpenCode and OpenClaw, under the key `io-verifymcp-mcp` and with a trailing slash on the URL. The same shapes with the key and URL used above:

```toml
[mcp_servers.verifymcp]
url = "https://mcp.verifymcp.io"
```

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "verifymcp": {
      "type": "remote",
      "url": "https://mcp.verifymcp.io",
      "enabled": true
    }
  }
}
```

```bash
openclaw mcp add verifymcp --url 'https://mcp.verifymcp.io' --transport streamable-http
```

## Skill

[skills.sh](https://www.skills.sh/) / the Vercel skills CLI discovers `skills/verifymcp/SKILL.md`.

```bash
npx skills add Im5tu/verifymcp-plugins
```

The skill tells an agent when to check a trust score before installing a server, how to compare servers, how to read change history and known malware, and how to interpret the 0-100 score.

## Hermes plugin

`hermes/verifymcp/` follows the portable Agent Plugins v1 layout Hermes loads today:

- `plugin.json` with schema `https://agent-plugins.org/schemas/1.0.0/plugin.schema.json`
- `mcp.json` with schema `https://agent-plugins.org/schemas/1.0.0/mcp.schema.json` and `type` `streamable-http`
- `skills/verifymcp/SKILL.md`, the same text as the skills.sh skill

Native `plugin.yaml` plus `register(ctx)` is a different plugin type. This package does not include it.

`hermes/plugin-catalog/verifymcp.yaml` is the draft entry. Its `sha` is a placeholder until a reviewer pins a 40-character commit. Catalogue fields match `plugin-catalog/README.md` in NousResearch/hermes-agent. `tier` is `community`.

## Files

| File | Role |
| --- | --- |
| `server.json` | Registry manifest, copied from `https://verifymcp.io/.well-known/mcp.json` |
| `mcp.json` | Generic client config |
| `skills/verifymcp/SKILL.md` | skills.sh skill |
| `hermes/verifymcp/` | Hermes portable plugin |
| `hermes/plugin-catalog/verifymcp.yaml` | Draft Hermes catalogue entry |
| `assets/` | `logo-512.png` and `icon-192.png` from verifymcp.io |

## Licence

Files in this repository are under the MIT licence. See [LICENSE](LICENSE).

VerifyMCP scores, trust reports, change feeds and badges are not in this repository. They are [CC BY-NC 4.0](https://verifymcp.io/docs/data-licence). Credit a figure you repeat:

```
Data from [VerifyMCP](https://verifymcp.io), [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)
```
