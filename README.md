<p align="center">
  <h1 align="center">ROAR Protocol</h1>
  <p align="center"><strong>Real-time Open Agent Runtime</strong></p>
  <p align="center">Unified agent communication protocol combining MCP + A2A + ACP + ANP into one standard.</p>
</p>

---

## Why ROAR?

The agent ecosystem is fragmented:

| Protocol | What it does | Limitation |
|----------|-------------|------------|
| **MCP** (Anthropic) | Tool use — agents call tools | No agent-to-agent messaging |
| **A2A** (Google) | Agent-to-agent tasks | No tool invocation standard |
| **ACP** (IBM) | Complex workflows | Heavy, enterprise-focused |
| **ANP** (Community) | Agent networking | Early stage, limited adoption |

ROAR unifies all four into a single, backward-compatible protocol stack. Write ROAR once, speak to any agent regardless of which protocol they natively support.

---

## 5-Layer Architecture

```
┌──────────────────────────────────────────────────┐
│   Layer 5: Stream                                 │
│   Real-time events, token streaming, pub/sub      │
├──────────────────────────────────────────────────┤
│   Layer 4: Exchange                               │
│   Unified message format, intents, signing        │
├──────────────────────────────────────────────────┤
│   Layer 3: Connect                                │
│   Transport negotiation, sessions, keep-alive     │
├──────────────────────────────────────────────────┤
│   Layer 2: Discovery                              │
│   Agent cards, capability advertising, search     │
├──────────────────────────────────────────────────┤
│   Layer 1: Identity                               │
│   W3C DID-based identity, Ed25519 keys, tokens   │
└──────────────────────────────────────────────────┘
```

| Layer | Name | Purpose | Key Types |
|-------|------|---------|-----------|
| 1 | **Identity** | Agent registration, DID-based identity, capability tokens | `AgentIdentity`, `AgentCapability` |
| 2 | **Discovery** | Service mesh, capability advertising, matchmaking | `AgentCard`, `AgentDirectory`, `DiscoveryEntry` |
| 3 | **Connect** | Transport negotiation, session management, keep-alive | `ConnectionConfig`, `TransportType` |
| 4 | **Exchange** | Message formats, tool invocation, context sharing | `ROARMessage`, `MessageIntent` |
| 5 | **Stream** | Real-time events, token streaming, pub/sub | `StreamEvent`, `StreamEventType` |

---

## Specs

Detailed specification for each layer:

- [Layer 1: Identity](spec/01-identity.md) — W3C DID-based agent identity, Ed25519 signing, capability tokens
- [Layer 2: Discovery](spec/02-discovery.md) — Decentralized agent directory, capability search, agent cards
- [Layer 3: Connect](spec/03-connect.md) — Transport negotiation (stdio, HTTP, WebSocket, gRPC), sessions
- [Layer 4: Exchange](spec/04-exchange.md) — Unified message format, 7 intents, HMAC-SHA256 signing
- [Layer 5: Stream](spec/05-stream.md) — Real-time event streaming, 8 event types, pub/sub

---

## Quick Start

### Install the reference implementation

```bash
git clone https://github.com/mcpcentral/prowlrbot.git
cd prowlrbot
pip install -e .
```

### Create an agent identity

```python
from prowlrbot.protocols.roar import AgentIdentity

agent = AgentIdentity(
    display_name="my-agent",
    agent_type="agent",
    capabilities=["code", "review", "testing"],
)
print(agent.did)  # did:roar:agent:my-agent-a1b2c3d4
```

### Send a ROAR message

```python
from prowlrbot.protocols.roar import (
    AgentIdentity, ROARMessage, MessageIntent
)

sender = AgentIdentity(display_name="architect")
receiver = AgentIdentity(display_name="frontend")

msg = ROARMessage(
    **{"from": sender, "to": receiver},
    intent=MessageIntent.DELEGATE,
    payload={
        "action": "implement-component",
        "params": {"component": "UserDashboard", "framework": "react"},
    },
)

# Sign with HMAC-SHA256
msg.sign("shared-secret-key")
print(msg.verify("shared-secret-key"))  # True
```

### Use the MCP adapter (backward compatibility)

```python
from prowlrbot.protocols.roar import AgentIdentity, MCPAdapter

agent = AgentIdentity(display_name="my-agent")

# Convert an MCP tool call to a ROAR message
roar_msg = MCPAdapter.mcp_to_roar(
    tool_name="read_file",
    params={"path": "src/main.py"},
    from_agent=agent,
)

# Convert back to MCP format
mcp_call = MCPAdapter.roar_to_mcp(roar_msg)
# → {"tool": "read_file", "params": {"path": "src/main.py"}}
```

### Use the A2A adapter (agent-to-agent tasks)

```python
from prowlrbot.protocols.roar import AgentIdentity, A2AAdapter

sender = AgentIdentity(display_name="orchestrator")
worker = AgentIdentity(display_name="worker")

# Convert an A2A task to a ROAR message
roar_msg = A2AAdapter.a2a_task_to_roar(
    task={"action": "analyze_code", "files": ["src/app.py"]},
    from_agent=sender,
    to_agent=worker,
)

# Convert back to A2A format
a2a_task = A2AAdapter.roar_to_a2a(roar_msg)
```

---

## Protocol Adapters

ROAR is backward-compatible with existing protocols via adapters:

| Adapter | Direction | Purpose |
|---------|-----------|---------|
| `MCPAdapter` | MCP ↔ ROAR | Convert MCP tool calls to/from ROAR messages |
| `A2AAdapter` | A2A ↔ ROAR | Convert A2A agent tasks to/from ROAR messages |

This means you can:
- Use ROAR to communicate with MCP-only tools
- Use ROAR to delegate tasks to A2A-compatible agents
- Mix and match protocols in the same system
- Gradually migrate from MCP/A2A to native ROAR

---

## SDK

The ROAR SDK provides production-ready implementations:

```
src/prowlrbot/protocols/sdk/
├── client.py          # ROAR client for sending messages
├── server.py          # ROAR server for receiving messages
├── router.py          # Message routing and dispatch
├── streaming.py       # Real-time event streaming
├── crypto.py          # Ed25519 signing and verification
├── identity.py        # Identity management
├── discovery.py       # Agent directory operations
├── adapters/
│   ├── mcp.py         # MCP ↔ ROAR adapter
│   └── a2a.py         # A2A ↔ ROAR adapter
└── transports/
    ├── stdio.py       # stdio transport (for CLI/MCP)
    ├── http.py        # HTTP transport (for REST APIs)
    ├── websocket.py   # WebSocket transport (real-time)
    └── grpc.py        # gRPC transport (high-performance)
```

### Transport Selection

ROAR auto-selects the best transport:

| Context | Transport | Why |
|---------|-----------|-----|
| Claude Code terminal | stdio | MCP compatibility |
| Web API | HTTP | REST endpoints |
| Real-time dashboard | WebSocket | Streaming events |
| High-throughput | gRPC | Binary serialization |

---

## Message Intents

Every ROAR message has an intent that declares what the sender wants:

| Intent | Direction | Example |
|--------|-----------|---------|
| `EXECUTE` | Agent → Tool | "Run this shell command" |
| `DELEGATE` | Agent → Agent | "Implement this feature" |
| `UPDATE` | Agent → IDE | "Here's my progress" |
| `ASK` | Agent → Human | "Which approach should I take?" |
| `RESPOND` | Any → Any | Response to any intent |
| `NOTIFY` | Any → Any | One-way notification |
| `DISCOVER` | Any → Directory | "Find agents that can do X" |

---

## Stream Event Types

Real-time events for monitoring and coordination:

| Event Type | Source | Purpose |
|------------|--------|---------|
| `tool_call` | Agent | Agent is calling a tool |
| `mcp_request` | MCP Client | Incoming MCP tool request |
| `reasoning` | Agent | Agent's thinking/planning |
| `task_update` | War Room | Task status changed |
| `monitor_alert` | Monitor | Web/API change detected |
| `agent_status` | Agent | Agent went idle/busy/offline |
| `checkpoint` | Agent | Checkpoint for recovery |
| `world_update` | AgentVerse | Virtual world state change |

---

## Security

ROAR uses HMAC-SHA256 message signing by default:

```python
# Sign a message
msg.sign("shared-secret-key")

# Verify a message
is_valid = msg.verify("shared-secret-key")
```

For production deployments, ROAR also supports:
- **Ed25519** public key signing (via `AgentIdentity.public_key`)
- **JWT** tokens for web-based authentication
- **mTLS** for transport-level security

---

## Integration with ProwlrBot

ROAR is the communication backbone for the ProwlrBot ecosystem:

```
┌─────────────────────────────────────────────────────┐
│                   ProwlrBot Platform                 │
│                                                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────┐  │
│  │ ProwlrHub│  │ AgentVerse│  │   Marketplace    │  │
│  │ War Room │  │ Virtual   │  │   Skills/Agents  │  │
│  │ (MCP)    │  │ World     │  │   MCP Servers    │  │
│  └────┬─────┘  └─────┬────┘  └────────┬─────────┘  │
│       │              │                │              │
│       └──────────────┼────────────────┘              │
│                      │                               │
│              ┌───────┴───────┐                       │
│              │ ROAR Protocol │                       │
│              │  (this repo)  │                       │
│              └───────────────┘                       │
└─────────────────────────────────────────────────────┘
```

---

## Contributing

1. Fork this repo
2. Create a feature branch
3. Submit a PR with spec changes or SDK improvements

All protocol changes must include:
- Updated spec document
- Wire format examples
- Backward compatibility notes

---

## Links

- **Reference Implementation**: [ProwlrBot](https://github.com/mcpcentral/prowlrbot) — Full platform with ROAR SDK
- **War Room**: [ProwlrHub](https://github.com/mcpcentral/prowlrbot/tree/main/src/prowlrbot/hub) — Multi-agent coordination over MCP
- **Marketplace**: [prowlr-marketplace](https://github.com/mcpcentral/prowlr-marketplace) — Skills, agents, MCP servers
- **Virtual World**: [AgentVerse](https://github.com/mcpcentral/agentverse) — Interactive agent environment
- **Docs**: [prowlr-docs](https://mcpcentral.github.io/prowlr-docs)

## License

[Apache 2.0](LICENSE)
