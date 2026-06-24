# PB&J Machine MCP Server

Exposes your PB&J Machine Co. machine as a set of tools for LLM workflows. Connect this server to any MCP-compatible client — Claude, a custom agent, an internal Slack bot — and let it place orders, check inventory, and monitor assembly without human intervention.

---

## Setup

### Prerequisites

- Node.js 18+
- A running instance of the [PB&J Machine API](https://github.com/pbj-machine-co/pbj-api)
- An MCP-compatible client (Claude Desktop, Continue, or similar)

### Install

```bash
git clone https://github.com/pbj-machine-co/pbj-mcp-server
cd pbj-mcp-server
npm install
npm run build
```

### Configure

Create a `.env` file at the project root:

```bash
PBJ_API_URL=https://api.pbj.co
PBJ_API_KEY=your-64-character-hex-api-key
OPERATOR_ID=your-operator-id
```

### Connect to Claude Desktop

Add this to your `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "pbj": {
      "command": "node",
      "args": ["/path/to/pbj-mcp-server/dist/index.js"],
      "env": {
        "PBJ_API_URL": "https://api.pbj.co",
        "PBJ_API_KEY": "your-api-key",
        "OPERATOR_ID": "your-operator-id"
      }
    }
  }
}
```

---

## Tools

### `place_order`

Place a single sandwich order.

**Inputs:** `bread`, `spread`, `jelly`, `quantity` (optional, default 1), `dietary_flags` (optional)

**Returns:** Order ID and estimated completion time.

---

### `submit_batch`

Submit a batch order for 10 or more sandwiches.

**Inputs:** Array of order objects, each with `bread`, `spread`, `jelly`, `quantity`

**Returns:** Batch ID and queue position.

---

### `get_order_status`

Check the current assembly phase of an active order.

**Inputs:** `order_id`

**Returns:** Current phase, estimated time remaining, and any quality gate failures.

---

### `check_inventory`

Get current ingredient levels and flag anything low or expired.

**Inputs:** None

**Returns:** Status for all bread types, spreads, and jellies currently loaded in the machine.

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for tool development patterns, how to add new tools, and how to run the server against the machine simulator.
