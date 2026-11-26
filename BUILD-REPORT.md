# BUILD REPORT — dcfvalue

Clean-room rebuild of `com.edgethirteen.www/dcf-calculator`
v1.0.0 (remote-only MCP server at
`https://www.edgethirteen.com/api/mcp/dcf-calculator`),
built 2026-10-28 as a forwarding client. Package:
`@reneturcios/dcfvalue` 0.1.0, registry name
`io.github.CloudCorpRecords/dcfvalue`.

## Surface built

Full surface: **1 of 1 captured tools built**, none omitted.

| Tool | Required fields as served |
|------|---------------------------|
| `compute_dcf` | `fcf`, `growth`, `discount`, `terminal`, `years` |

Optional fields served as captured: `ticker`, `price`. The
served tool carries exactly the keys `name`, `description`,
`inputSchema`, exactly as captured: the capture carries NO
`title` key on the tool and none is served. The tool literal was
generated programmatically from
`harness/fixtures/dcf-tools.json` by the script at
`/tmp/b51-dcfvalue-scratch/gen.py` (parse + re-serialise; the
generator's round-trip assertions confirmed the emitted literal
parses back to the fixture's tool object, including property
key order `fcf, growth, discount, terminal, years, ticker,
price`) — never retyped — and the registration-integrity test
suite plus the parity harness confirmed the served literal
deep-equals the fixture's tool object. Tools are registered via
the MCP SDK low-level Server API (`ListToolsRequestSchema` /
`CallToolRequestSchema` handlers), not McpServer+zod.

## Fixture conflicts

**None.** The spec and the fixture agree on the tool's name,
description, required fields (`fcf`, `growth`, `discount`,
`terminal`, `years`), optional fields (`ticker`, `price`) and
property shapes, including the absence of a `title` key.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned:
  `https://www.edgethirteen.com/api/mcp/dcf-calculator`.
- Env override `DCFVALUE_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent. Tests assert that no `Authorization`
  header is sent under any environment — including with key-like
  env vars (`DCFVALUE_API_KEY`, `API_KEY`, `AUTHORIZATION`)
  set — and that no `X-API-Key` header is ever sent; upstream
  requests carry only `Content-Type`, `Accept` and, when issued,
  `Mcp-Session-Id`.
- Error rendering `dcfvalue error:` text for upstream
  failures — labelled reconstruction.
- Upstream results pass through verbatim, including
  `structuredContent`, `isError` and extra fields. tools/call is
  forwarded as streamable-HTTP JSON-RPC: initialize
  (protocolVersion `2025-03-26`) → notifications/initialized →
  tools/call, honouring `Mcp-Session-Id` when issued; plain-JSON
  and SSE replies are both parsed. The forwarding core takes an
  injectable fetch; tests stub the upstream — no live network
  calls in tests.

## Deviations

**None.** The full 20-file factory set was built as specified;
the offline lockfile route (Route 3) succeeded on the first
attempt, so no fallback route or detached-run workaround was
needed.

## Lockfile

- Route: **Route 3 (offline)**, first attempt:
  `npm install --package-lock-only --ignore-scripts --offline`,
  generated clean in the build dir with no `node_modules` and no
  pre-existing lockfile present.
- Parses as JSON; `name` == `@reneturcios/dcfvalue`.
- Package entries: **128**.
- sha256: `18e1e4cc89864923d77ffc605803484251e0a6f47e650838c095eb55df99129b`
  (re-verified unchanged after the subsequent full `npm install`).

## server.json

- Description exactly: "Discounted-cash-flow value per share from your own FCF, growth, discount and terminal inputs." —
  **93 characters** (measured from the written file).
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/dcfvalue:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio.
  Dockerfile (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/dcfvalue"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/dcfvalue`
  and its smoke test asserts the server name `dcfvalue`.

## Verification (all run 2026-10-28)

1. `tsc --noEmit` → exit 0.
2. `npm run build` (and the direct `tsc` binary) → exit 0; real
   `dist/` produced (cli, forward, index, mcp, types — .js,
   .d.ts, .js.map each).
3. `node dist/cli.js --version` → `0.1.0`.
4. Tests (`npm test` / `tsx --test test/*.test.ts`, fake-fetch
   only) → **53 tests, 53 pass, 0 fail** (registration-integrity
   suite deep-equals the served tools/list against the fixture's
   tool, including required-key exactness, property order, and
   the NO-`title`-key assertion, plus session handling/reuse,
   env override, the no-Authorization / no-X-API-Key assertions
   under any environment, verbatim argument forwarding and
   result pass-through including structuredContent and upstream
   isError, SSE parsing, error rendering for both the
   JSON-RPC-error and transport-failure paths, MCP-layer
   validation, and CLI behaviour including a working subcommand
   for the tool against a local stub server).
5. Parity: `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/dcf-tools.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/dcfvalue/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/dcfvalue/parity-out` → **verdict PASS, parity 100.0**
   (1/1 identical, 0 missing, 0 extra, no schema diffs). Output in
   `parity-out/`.
