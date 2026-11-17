# Changelog

## 0.1.0 (2026-10-28)

- First release: a forwarding client for the hosted Duesvo MCP server. All 7 captured tools are answered locally for tools/list, and every tools/call is forwarded upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when the upstream issues one).
- Tools: `dataset_columns`, `dataset_search`, `dataset_row`, `dataset_compare`, `dataset_stats`, `dataset_top`, `dataset_provenance`. The capture carries `title` keys on every tool and they are served verbatim; `dataset_columns` and `dataset_provenance` carry no `required` key in the capture and none is served.
- Upstream results pass through verbatim, including the isError flag; upstream failures surface as `duescope error:` text results.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). The upstream endpoint can be overridden via `DUESVO_MCP_URL`. The upstream record declares no headers, so no auth header is ever sent.
