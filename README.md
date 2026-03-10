<p align="center">
  <h1 align="center">ROAR Protocol</h1>
  <p align="center"><strong>Real-time Open Agent Runtime</strong></p>
  <p align="center">Unified agent communication protocol combining MCP + A2A + ACP + ANP into one standard.</p>
</p>

---

## 5-Layer Architecture

| Layer | Name | Purpose |
|-------|------|---------|
| 1 | **Identity** | Agent registration, DID-based identity, capability tokens |
| 2 | **Discovery** | Service mesh, capability advertising, matchmaking |
| 3 | **Connect** | Transport negotiation, session management, keep-alive |
| 4 | **Exchange** | Message formats, tool invocation, context sharing |
| 5 | **Stream** | Real-time events, token streaming, pub/sub |

## Why ROAR?

The agent ecosystem is fragmented. MCP handles tool use. A2A handles agent-to-agent messaging. ACP handles complex workflows. ANP handles networking. ROAR unifies all of these into a single, backward-compatible protocol stack.

## Specs

- [Layer 1: Identity](spec/01-identity.md)
- [Layer 2: Discovery](spec/02-discovery.md)
- [Layer 3: Connect](spec/03-connect.md)
- [Layer 4: Exchange](spec/04-exchange.md)
- [Layer 5: Stream](spec/05-stream.md)

## Status

Early development. Star this repo to follow progress.

## Links

- [ProwlrBot](https://github.com/mcpcentral/prowlrbot) — Reference implementation
- [Docs](https://mcpcentral.github.io/prowlr-docs)

## License

[Apache 2.0](LICENSE)
