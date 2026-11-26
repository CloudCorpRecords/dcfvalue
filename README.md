# dcfvalue

What is a share actually worth on your own assumptions? dcfvalue is a local forwarding client for the hosted discounted-cash-flow calculator MCP service: it answers `tools/list` locally from the captured tool definition and forwards every call to the service, returning the answers exactly as the service sends them back.

The one tool runs a simplified DCF valuation: free cash flow per share is projected forward at a constant growth rate for the years you choose, each year's cash flow is discounted back at your discount rate, and a Gordon-growth terminal value is added on top. The five core inputs — current free cash flow per share, growth, discount rate, terminal growth, and projection years — are judgment calls you supply directly; nothing about them is looked up. If you also pass a current `price`, or a `ticker` so the service can fetch the current price, the result can be set against the market price as a margin-of-safety comparison. There is no write, signup, or send tool in this surface.

## The tool

- **`compute_dcf`** — An intrinsic value per share from your own DCF inputs. Required: `fcf` (free cash flow per share), `growth` (projection growth rate, in percent), `discount` (discount rate, in percent), `terminal` (terminal growth rate, in percent), and `years` (number of projection years). Optional: `ticker` (used only to fetch the current price) and `price` (current price, used instead of a ticker lookup).

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/dcfvalue mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "dcfvalue": {
      "command": "npx",
      "args": ["-y", "@reneturcios/dcfvalue", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/dcfvalue
docker run --rm ghcr.io/cloudcorprecords/dcfvalue compute_dcf '{"fcf":5.2,"growth":8,"discount":10,"terminal":2.5,"years":10,"price":12.5}'
```

## CLI

One binary serves both the command line and the MCP server. The tool is a subcommand taking a single JSON string with that tool's arguments:

```bash
dcfvalue compute_dcf '{"fcf":5.2,"growth":8,"discount":10,"terminal":2.5,"years":10}'
dcfvalue --version
```

Exit codes: `0` when the forwarded call succeeds, `1` when the upstream result is an error, and `2` for usage problems such as an unknown subcommand or a malformed JSON argument.

## Configuration

- `DCFVALUE_MCP_URL` — Overrides the upstream endpoint. The default is the pinned hosted endpoint, `https://www.edgethirteen.com/api/mcp/dcf-calculator`. (The override variable is a reconstruction added by this rebuild.)

The upstream registry record declares no headers, so no authentication header is ever sent; the tool surface was captured keyless.

Upstream failures — a JSON-RPC error from the service, a non-2xx response, an unreadable body, or a request that fails outright — are reported to the caller as an error result whose text starts with `dcfvalue error:`.

## How forwarding works

The first forwarded call opens a session: an `initialize` request (protocol version `2025-03-26`), followed by a `notifications/initialized` notification, then the `tools/call` itself. If the service answers the initialize request with an `Mcp-Session-Id` header, that value is carried on every later request, and the open session is reused for the rest of the process. Replies are accepted either as plain JSON or as server-sent events. Successful results pass through verbatim, including any `isError` flag and any extra fields the service includes.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The test suite checks the served tool list against the captured surface (the 1 tool, deep-equal, with no `title` key served — the capture carries none — the `required` list exactly as captured, and every property in captured order), exercises the session handshake and its reuse, verifies that no `Authorization` or `X-API-Key` header is ever sent under any environment, and verifies that arguments go upstream untouched and results come back untouched.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
