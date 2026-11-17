# BUILD REPORT — duescope

Clean-room rebuild of `duesvo/site` v1.0.1 (remote-only
MCP server at `https://www.duesvo.site/mcp`), built
2026-10-28 as a forwarding client. Package:
`@reneturcios/duescope` 0.1.0, registry name
`io.github.CloudCorpRecords/duescope`.

## Surface built

Full surface: **7 of 7 captured tools built**, none omitted.

| Tool | Required fields as served |
|------|---------------------------|
| `dataset_columns` | NO `required` key in the capture — served verbatim, none invented |
| `dataset_search` | `query` |
| `dataset_row` | `column`, `value` |
| `dataset_compare` | `column`, `values` |
| `dataset_stats` | `column` |
| `dataset_top` | `column` |
| `dataset_provenance` | NO `required` key in the capture — served verbatim, none invented |

Each served tool carries exactly the keys `name`, `title`,
`description`, `inputSchema`, exactly as captured: the capture
carries `title` keys on every tool and they are served verbatim.
The tool literals were generated programmatically from
`harness/fixtures/duesvo-tools.json` by the script at
`/tmp/b49-duescope-scratch/gen.py` (parse + re-serialise; the
generator's round-trip assertions confirmed the emitted literals
parse back to the fixture's tool objects) — never retyped — and
the registration-integrity test suite plus the parity harness
confirmed the served literals deep-equal the fixture's 7 tool
objects, in order. Tools are registered via the MCP SDK
low-level Server API (`ListToolsRequestSchema` /
`CallToolRequestSchema` handlers), not McpServer+zod.

## Fixture conflicts

**None.** The spec and the fixture agree on all 7 tools' names,
titles, descriptions, required fields (including the two tools
with no `required` key: `dataset_columns` and
`dataset_provenance`) and property shapes.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned: `https://www.duesvo.site/mcp`.
- Env override `DUESVO_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent. Tests assert that no `Authorization`
  header is sent under any environment — including with key-like
  env vars (`DUESVO_API_KEY`, `API_KEY`, `AUTHORIZATION`) set —
  and that no `X-API-Key` header is ever sent; upstream requests
  carry only `Content-Type`, `Accept` and, when issued,
  `Mcp-Session-Id`.
- Error rendering `duescope error:` text for upstream failures —
  labelled reconstruction.
- Upstream results pass through verbatim, including
  `structuredContent`, `isError` and extra fields. tools/call is
  forwarded as streamable-HTTP JSON-RPC: initialize
  (protocolVersion `2025-03-26`) → notifications/initialized →
  tools/call, honouring `Mcp-Session-Id` when issued; plain-JSON
  and SSE replies are both parsed. The forwarding core takes an
  injectable fetch; tests stub the upstream — no live network
  calls in tests.

## Deviations

None.

## Lockfile

- Route: generated clean from birth in the build dir with no
  `node_modules` and no pre-existing lockfile:
  `npm install --package-lock-only --ignore-scripts --offline`
  (Route 3).
- Parses as JSON; `name` == `@reneturcios/duescope`.
- Package entries: **128**.
- sha256: `c73ce6d8595b47c16879d222c3b2fad53af0ba2a35de6d1d8d08ebcadad59a57`
  (re-verified unchanged after the subsequent offline `npm ci`).

## server.json

- Description exactly: "Read-only queries over a published website-change dataset: search, rows, stats and provenance." —
  **94 characters** (verified programmatically).
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/duescope:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio.
  Dockerfile (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/duescope"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/duescope`
  and its smoke test asserts the server name `duescope`.

## Verification (all run 2026-10-28)

1. `npx tsc --noEmit` → exit 0.
2. `npm run build` → exit 0; real `dist/` produced (cli, forward,
   index, mcp, types — .js, .d.ts, .js.map each).
3. `node dist/cli.js --version` → `0.1.0`.
4. `npm test` → **58 tests, 58 pass, 0 fail** (registration-integrity
   suite deep-equals the served tools/list against the fixture's 7
   tools, including title verbatim and required-key exactness
   assertions — dataset_columns and dataset_provenance have NO
   required key — plus session handling/reuse, env override, the
   no-Authorization / no-X-API-Key assertions under any
   environment, verbatim per-tool argument forwarding for all 7
   tools and result pass-through including structuredContent and
   upstream isError, SSE parsing, error rendering for both the
   JSON-RPC-error and transport-failure paths, MCP-layer
   validation, and CLI behaviour including a working subcommand
   for every one of the 7 tools against a local stub server).
5. Parity: `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/duesvo-tools.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/duescope/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/duescope/parity-out` → **verdict PASS, parity 100.0**
   (7/7 identical, 0 missing, 0 extra, no schema diffs). Output in
   `parity-out/`.
