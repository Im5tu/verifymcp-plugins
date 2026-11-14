---
name: verifymcp
description: Check an MCP server on VerifyMCP before installing or connecting it. Use when you need an independent 0-100 trust score, a side-by-side comparison, the change history, or a known-malware finding. The public endpoint is https://mcp.verifymcp.io (Streamable HTTP, no sign-in).
license: MIT
compatibility: Works with the VerifyMCP remote MCP server, or with the public read URLs on verifymcp.io when that server is not connected.
---

# VerifyMCP

VerifyMCP publishes independent trust scores for Model Context Protocol servers. Every server in the official MCP registry is re-scanned daily and scored 0-100 on a published rubric: supply-chain provenance, endpoint security, schema quality and stability. A score is an observation VerifyMCP can evidence. It is not a vendor self-attestation and not an endorsement.

Use this skill before installing or connecting an MCP server you did not write. Also use it to compare servers that do the same job, to read what changed, and to check known malware.

VerifyMCP is not a package registry and not a download mirror. It hosts no artefacts and installs nothing. It does not judge whether a server's output is correct.

## How to reach the data

Prefer the MCP server when it is connected.

- Endpoint: `https://mcp.verifymcp.io`
- Transport: Streamable HTTP
- Protocol: `2026-07-28` (as stated in `https://verifymcp.io/llms.txt`)
- Registry name: `io.verifymcp/mcp`
- Manifest: `https://verifymcp.io/.well-known/mcp.json`
- Every tool works without signing in. OAuth is optional. Protected-resource metadata (RFC 9728) is at `https://mcp.verifymcp.io/.well-known/oauth-protected-resource` (scope `directory:read`, authorisation server `https://verifymcp.io`).

If the tools are not connected, read the public documents. They need no key. `{server}` and `{component}` are the two path segments of a directory URL such as `https://verifymcp.io/servers/{server}/{component}`.

- Trust report, markdown: `https://verifymcp.io/servers/{server}/{component}.md`
- Trust report, JSON: `https://verifymcp.io/servers/{server}/{component}.json` (schema `https://verifymcp.io/schemas/trust-report.json`)
- Change feed, JSON: `https://verifymcp.io/servers/{server}/{component}/changelog.json` (`since`, `materiality`, `limit`)
- Change feed, RSS: `https://verifymcp.io/servers/{server}/{component}.xml`
- Name lookup: `https://verifymcp.io/opensearch/suggest.json?q=`
- OpenAPI for the read API: `https://verifymcp.io/openapi.json`

The tool list below is the surface captured on the server's own directory page on 14 November 2026 (`https://verifymcp.io/servers/io-verifymcp-mcp/mcp`). If a connected `tools/list` differs, follow the live list.

## Tools

All nine tools are read-only. `identifier` accepts a package name, an endpoint URL, a registry name, or a verifymcp slug. Pass `component` when a server ships more than one channel (for example `npm` alongside a remote endpoint).

| Tool | Required input | What it returns |
| --- | --- | --- |
| `list_servers` | none | Directory filter. Name matching only: descriptions and capabilities are not searched, so a plain-English question matches nothing. Returns the first 100. |
| `get_server` | `identifier` | Full trust report: score, per-category verdicts and reasons, tools with context-token cost, recent change history. Call this before installing or trusting a server. |
| `get_server_comparison` | `identifiers` (array of 2 to 5) | Side-by-side scores, per-category breakdown, tool counts and context-token cost. |
| `get_server_alternatives` | `identifier` | Other servers doing a similar job, with scores. Neighbours share a product, publisher namespace or category. This is not a semantic search. |
| `get_server_diagnostics` | `identifier` | Evidence behind a score: TLS, DNSSEC, authorisation, redirects, transports, provenance, install scripts, known CVEs and dependency health. Use this to explain a low score. |
| `get_server_tools` | `identifier` | Every tool, with parameters, output schema and context-token cost. Use when `get_server` reported the tool list was truncated. |
| `get_server_install_config` | `identifier` | Ready-to-paste install snippets. Read `get_server` first: this is configuration, not a safety judgement. An MCPB bundle has no launch command; the tool returns the download URL, the registry SHA-256 and commands to verify the file. |
| `list_products` | none | Products servers integrate with (examples given by the tool: github, notion) and how many servers cover each. Use the slug with `list_servers`. |
| `list_known_malware` | none | Components carrying a supply-chain malware finding. |

`list_servers` optional arguments, in the words of the captured schema:

- `query` (string): name fragment matched against server and package names.
- `minScore` / `maxScore` (integer): inclusive score bounds.
- `product` (string): servers classified as working with this product, for example `github`.
- `types` (array): ecosystems; a server matches any one of them.
- `scored` (string): include unscored servers. Defaults to scored only.
- `sort` (string): default is best match, then trust, when `query` is set, otherwise trust. A given sort is applied exactly.

`list_known_malware` takes optional `status`: `active` (the default), `unverified`, `cleared` or `all`.

Shared output fields on these tools: `asOf`, `count`, `items`, `total` (matches before the 100-item cap), `truncated`, `notice` (how to narrow a truncated result, or why an empty one is empty), `source` (the canonical verifymcp.io page).

`get_server_install_config` also returns `requiresInput`. A non-empty array means the snippets are incomplete until the caller supplies values the registry declares and VerifyMCP cannot show (a secret, or a path only the caller knows). Those values are never placed in a snippet. `context` is present whenever a server resolved. `bundle` is set for MCPB bundles only.

Examples published on the server page (14 November 2026). They illustrate arguments. They are not a closed list of allowed values:

```json
{"query": "github"}
```

```json
{"minScore": 70, "product": "github"}
```

```json
{"minScore": 80, "sort": "score_desc", "types": ["remote"]}
```

```json
{"scored": "scored", "types": ["npm", "pypi"]}
```

```json
{"query": "postgres", "types": ["npm"]}
```

## Check a server before installing it

1. Resolve the name. Call `list_servers` with a `query` that is a name fragment, or open the suggest URL. If `truncated` is true, narrow with `minScore`, `types` or `product` and read `notice`.
2. Call `get_server` with the identifier. Read the total, the tier, each category verdict, the tool list and the recent change history. The headline score of a server that ships several channels is its best component. Pass `component` when you need a specific channel.
3. If the tool list was truncated, call `get_server_tools`.
4. If the score is low, or a category is unexplained, call `get_server_diagnostics` and open the fix guide linked from the finding. Docs index: `https://verifymcp.io/docs`. Rubric: `https://verifymcp.io/docs/scoring`.
5. Call `list_known_malware` (default `status` is `active`) or read the malware section of the trust report before you install a package.
6. When you are choosing among two to five candidates, call `get_server_comparison`. For neighbours of one server, call `get_server_alternatives`.
7. Call `get_server_install_config` only after the report is acceptable for the task. If `requiresInput` is non-empty, the snippets are not finished.

A high score does not replace reading the tool descriptions. Tool text is an instruction channel the model will see.

## How to read a score

The number is 0 to 100. 100 means the component passed every verifiable check that applies to its type. Bands, from the scoring docs:

- 80 and above: high trust
- 60-79: medium
- below 60: low

Half of every score is a shared core, scored the same way for a package and a remote endpoint. The other half is specific to the delivery channel. Each half is 50 percent.

| Shared core | Weight | Package only | Weight | Remote only | Weight |
| --- | --- | --- | --- | --- | --- |
| Schema quality and AI usability | 18% | Supply chain security | 30% | Endpoint security | 40% |
| Stability and change management | 14% | Provenance and transparency | 20% | Transport and reachability | 10% |
| Tool coverage | 8% | | | | |
| Tool safety | 6% | | | | |
| Capabilities | 4% | | | | |

Package rubric: `https://verifymcp.io/docs/scoring/package-rubric`. Remote rubric: `https://verifymcp.io/docs/scoring/remote-rubric`.

Signals inside a category sit on a geometric weight ladder (16, 8, 4, 2, 1), so the first checks matter most. Anything VerifyMCP cannot verify scores 0 and keeps its weight: an access-gated endpoint, an unreachable repository, a vendor with no data, or no 30-day history yet. A score means the check was positively verified.

Some findings cap the whole track at 0: a confirmed-malware verdict, an unpatched critical CVE that has a fix available, or a broken or plaintext-HTTP connection. A critical vulnerability with no published fix is surfaced and does not cap. A model suspicion of malware is surfaced and does not cap.

Liveness is separate from the rubric. Consecutive days unreachable:

| Days | State | Effect |
| --- | --- | --- |
| 0 | Live | Scored on each crawl |
| 1-2 | Grace | Score held at the last good value |
| 3-6 | Degraded | Last good score decays toward 0 |
| 7 or more | Dead | Score pinned to 0 until a manual reinstatement |

A server is dead only when every channel is dead. Scores are re-crawled daily. A move can also be a rubric change; those are listed at `https://verifymcp.io/docs/scoring/changelog`.

The publisher badge is not part of the score. Official publisher, owner verified, and no publisher proof are three states of evidence about the name. No proof is the normal state and is not an accusation.

The score leaves these questions open. The scoring page states them:

- What a tool does when it is called. The only tool call to a remote endpoint names a tool that cannot exist. Real tools are never invoked. Tool safety scores the text, not the behaviour behind it.
- Where data goes. No real tool is called with real input. A package run in the sandbox has no network.
- Behaviour in your environment. A package is run once as stdio in an isolated sandbox. A remote endpoint is judged on how it answered the probe.
- Latency, uptime percentage, or a service-level guarantee. The daily probe asks whether the component is still there.
- Whether the publisher is a trustworthy business.
- A source-code audit. Supply chain uses a vendor malware verdict, known advisories, the install-script command string and published provenance.
- Whether a tool asks for too much access, or whether its name matches what it does.

There is no paid tier and no way to buy a higher score.

## Change history

`get_server` includes the recent change history for the component.

For the full feed, fetch `changelog.json` or the RSS document at the URLs above. Each entry has a date. The captured feed also carries `kind`, `summary`, `materiality`, `direction` and, for a score move, `scoreDelta`. Do not treat that field list as closed: the OpenAPI description says response bodies grow additively.

Read the summary before you treat a score move as a change to the server. The feed says when a day's move is a rubric change rather than a change to the server. Materiality labels seen on the VerifyMCP component's own feed include `functional` and `security`; use the label on the entry you are reading.

## Known malware

`list_known_malware` lists components where a vendor scan returned a malware verdict. The same register is on `https://verifymcp.io/known-malware`.

Default `status` is `active`. Pass `unverified`, `cleared` or `all` when you need those rows. A confirmed verdict caps the score at 0. An unverified or model-only suspicion does not, and it is still worth reading before you install.

## Data licence

Scores, findings, trust reports, change feeds, badges and the aggregate statistics are [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/). Publisher-written names and descriptions stay under the publisher's own terms.

You may use the data to vet servers in your own CI and internal tooling, including inside a company. Redistributing the data, reselling it, or making it the reason someone pays for a product needs permission from VerifyMCP (`hello@verifymcp.io`).

When you repeat a figure, credit it:

```
Data from [VerifyMCP](https://verifymcp.io), [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)
```

Add "modified" if you changed the data.
