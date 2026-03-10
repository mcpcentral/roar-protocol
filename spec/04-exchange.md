# Layer 4: Exchange

> Unified message format, 7 intents, HMAC-SHA256 signing, protocol adapters

---

## Overview

The Exchange layer defines the universal message format for all ROAR communication. One message type, one signing scheme, seven intents — covers everything from tool calls to agent delegation to human questions.

---

## ROARMessage

### Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `roar` | string | Yes | Protocol version (default: "1.0") |
| `id` | string | Auto-generated | Unique message ID (`msg_<random>`) |
| `from` | AgentIdentity | Yes | Sender identity |
| `to` | AgentIdentity | Yes | Receiver identity |
| `intent` | MessageIntent | Yes | What the sender wants |
| `payload` | dict | Yes | Message content |
| `context` | dict | Optional | Session context, metadata |
| `auth` | dict | Optional | Authentication (signature, timestamp) |
| `timestamp` | float | Auto-generated | Unix timestamp |

---

## Wire Format

### EXECUTE (Agent → Tool)

```json
{
  "roar": "1.0",
  "id": "msg_a1b2c3d4e5",
  "from": {"did": "did:roar:agent:architect-f5e6d7c8", "display_name": "architect", "agent_type": "agent"},
  "to": {"did": "did:roar:tool:shell-9a8b7c6d", "display_name": "shell", "agent_type": "tool"},
  "intent": "execute",
  "payload": {"action": "run_command", "params": {"command": "pytest tests/ -v"}},
  "auth": {"signature": "hmac-sha256:a1b2c3...", "timestamp": 1710000000.0},
  "timestamp": 1710000000.0
}
```

### DELEGATE (Agent → Agent)

```json
{
  "roar": "1.0",
  "id": "msg_f6g7h8i9j0",
  "from": {"did": "did:roar:agent:architect-f5e6d7c8", "display_name": "architect"},
  "to": {"did": "did:roar:agent:frontend-k1l2m3n4", "display_name": "frontend"},
  "intent": "delegate",
  "payload": {"action": "implement-component", "params": {"component": "UserDashboard"}},
  "timestamp": 1710000000.0
}
```

---

## Message Intents

| Intent | Direction | Use case |
|--------|-----------|----------|
| `execute` | Agent → Tool | Run a tool/command |
| `delegate` | Agent → Agent | Delegate a task |
| `update` | Agent → IDE | Report progress |
| `ask` | Agent → Human | Request input |
| `respond` | Any → Any | Reply to a message |
| `notify` | Any → Any | One-way notification |
| `discover` | Any → Directory | Find agents |

---

## Signing

### HMAC-SHA256

```python
canonical = json.dumps(
    {"id": msg.id, "intent": msg.intent, "payload": msg.payload},
    sort_keys=True,
)
signature = hmac.new(secret.encode(), canonical.encode(), hashlib.sha256).hexdigest()
```

Stored as: `"hmac-sha256:<hex_digest>"`

---

## Protocol Adapters

### MCP ↔ ROAR

```python
# MCP → ROAR
roar_msg = MCPAdapter.mcp_to_roar("read_file", {"path": "src/main.py"}, agent)
# ROAR → MCP
mcp_call = MCPAdapter.roar_to_mcp(roar_msg)
# → {"tool": "read_file", "params": {"path": "src/main.py"}}
```

### A2A ↔ ROAR

```python
# A2A → ROAR
roar_msg = A2AAdapter.a2a_task_to_roar(task, sender, receiver)
# ROAR → A2A
a2a_task = A2AAdapter.roar_to_a2a(roar_msg)
```

---

## Design Decisions

1. **Single message type** — one `ROARMessage` for everything, differentiated by `intent`
2. **Canonical JSON signing** — `sort_keys=True` ensures deterministic serialization
3. **Adapter pattern** — backward compatibility via adapters, keeping the core clean
4. **Full identities in from/to** — enables capability-aware routing without directory lookups
