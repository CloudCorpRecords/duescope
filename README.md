# duescope

What does the published website-change dataset actually say? duescope is a local forwarding client for the hosted Duesvo MCP service: it answers `tools/list` locally from the captured tool definitions and forwards every call to the service, returning the answers exactly as the service sends them back.

All seven tools are read-only queries over one published dataset of website changes. You can inspect its shape, search its cells, look rows up by exact key, compare rows side by side, compute summary statistics and rankings over its numeric columns, and read where the data comes from and how to cite it. There is no write, signup, or send tool in this surface.

## The tools

- **`dataset_columns`** — The columns of the dataset, which of them are numeric, the row count and the provenance banner. Call this first to learn the schema. No parameters.
- **`dataset_search`** — Rows whose cells contain the `query` text (case-insensitive), up to 50; `limit` caps the count between 1 and 50. `query` is required.
- **`dataset_row`** — The rows where a `column` equals a `value` exactly (case-insensitive). Both are required.
- **`dataset_compare`** — The rows whose `column` is any of the given `values` (2 to 10 of them), in the order given — for "X vs Y" questions. Both are required.
- **`dataset_stats`** — Count, min, max, mean, median and sum of a numeric `column`; grouping commas and currency are handled, and non-numeric rows are excluded and counted. `column` is required.
- **`dataset_top`** — The highest rows by a numeric `column`, or the lowest when `ascending` is true; `limit` caps the count between 1 and 50. `column` is required.
- **`dataset_provenance`** — The source of the data, the date it was computed, the licence and the citation. Read this to attribute a figure correctly. No parameters.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/duescope mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "duescope": {
      "command": "npx",
      "args": ["-y", "@reneturcios/duescope", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/duescope
docker run --rm ghcr.io/cloudcorprecords/duescope dataset_search '{"query":"example"}'
```

## CLI

One binary serves both the command line and the MCP server. Each tool is a subcommand taking a single JSON string with that tool's arguments:

```bash
duescope dataset_columns '{}'
duescope dataset_search '{"query":"pricing","limit":10}'
duescope dataset_row '{"column":"domain","value":"example.com"}'
duescope dataset_compare '{"column":"domain","values":["a.example","b.example"]}'
duescope dataset_stats '{"column":"score"}'
duescope dataset_top '{"column":"score","limit":5}'
duescope dataset_provenance '{}'
duescope --version
```

Exit codes: `0` when the forwarded call succeeds, `1` when the upstream result is an error, and `2` for usage problems such as an unknown subcommand or a malformed JSON argument.

## Configuration

- `DUESVO_MCP_URL` — Overrides the upstream endpoint. The default is the pinned hosted endpoint, `https://www.duesvo.site/mcp`. (The override variable is a reconstruction added by this rebuild.)

The upstream registry record declares no headers, so no authentication header is ever sent; the tool surface was captured keyless.

Upstream failures — a JSON-RPC error from the service, a non-2xx response, an unreadable body, or a request that fails outright — are reported to the caller as an error result whose text starts with `duescope error:`.

## How forwarding works

The first forwarded call opens a session: an `initialize` request (protocol version `2025-03-26`), followed by a `notifications/initialized` notification, then the `tools/call` itself. If the service answers the initialize request with an `Mcp-Session-Id` header, that value is carried on every later request, and the open session is reused for the rest of the process. Replies are accepted either as plain JSON or as server-sent events. Successful results pass through verbatim, including any `isError` flag and any extra fields the service includes.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The test suite checks the served tool list against the captured surface (all 7 tools, deep-equal, in order, with titles served verbatim and no `required` key added to the two tools that carry none), exercises the session handshake and its reuse, verifies that no `Authorization` or `X-API-Key` header is ever sent under any environment, and verifies that arguments go upstream untouched and results come back untouched.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
