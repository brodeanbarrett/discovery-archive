# MCP Extension Mechanism Audit Report

**Target Crate:** `./moltis-extensions-discovery/moltis/crates/mcp/`  
**Documentation:** `./moltis-extensions-discovery/moltis/docs/src/mcp.md`  
**Date:** 2026-03-18

---

## Part 1: Extension Mechanism Identification

### Terms Used
- **MCP** - Model Context Protocol, an open protocol for connecting AI assistants to external tools
- **MCP servers** - External tool servers that implement the MCP specification
- **Tools** - Functions MCP servers expose that agents can call
- **Transport** - The communication mechanism (stdio or SSE/HTTP)

### What MCP Extends
MCP servers extend Moltis agent capabilities by providing:
- **Tools** - Functions the agent can call (e.g., search, file operations, API calls)
- **Resources** - Data the agent can read (defined in `ServerCapabilities`, see `types.rs:91-98`)
- **Prompts** - Pre-defined prompt templates (also in `ServerCapabilities`)

The protocol version implemented is `2024-11-05` (defined in `types.rs:163`).

---

## Part 2: Entry Points & Discovery

### How Moltis Discovers MCP Servers

MCP servers are discovered through two mechanisms:

1. **Configuration File (`moltis.toml`)**: Server definitions in `[mcp.servers]` section
   - Located at `~/.moltis/mcp-servers.json` (persisted registry)
   - Loaded via `McpRegistry::load()` in `registry.rs:105-120`

2. **Import from OpenClaw**: Legacy `mcp-servers.json` format support
   - Handled by `openclaw-import` crate
   - Merges with existing servers, skips duplicates by name

### Entry Point Files
- **Primary config**: `mcp-servers.json` in data directory (`~/.moltis/`)
- **User config**: `moltis.toml` with `[mcp.servers]` section
- **Registry path**: Set in `gateway/src/server.rs:1718`

### Server Registration
Servers are registered via:
- `McpRegistry::add()` in `registry.rs:135-139`
- `McpManager::add_server()` in `manager.rs:438-453`
- Web UI: Settings → MCP Servers → Add Server

---

## Part 3: Extension Structure & Skeleton

### Structure for MCP Server Definition

```rust
// From registry.rs:37-67
pub struct McpServerConfig {
    pub command: String,           // Required for stdio transport
    pub args: Vec<String>,         // Command arguments
    pub env: HashMap<String, String>,  // Environment variables
    pub enabled: bool,             // Whether server is active
    pub transport: TransportType, // Stdio or Sse
    pub url: Option<Secret<String>>,    // Required for SSE transport
    pub headers: HashMap<String, Secret<String>>,  // Custom HTTP headers
    pub oauth: Option<McpOAuthConfig>,  // Manual OAuth override
}
```

### Required Configuration
- **For stdio transport**: `command` (required), `args` (optional)
- **For SSE transport**: `transport = "sse"`, `url` (required)

---

## Part 4: Manifest & Metadata

### mcp-servers.json Format

The registry format (JSON) contains a map of server names to configurations:

```json
{
  "servers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/me/projects"],
      "env": {},
      "enabled": true,
      "transport": "stdio"
    },
    "github": {
      "command": "npx", 
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_TOKEN": "ghp_..." },
      "enabled": true,
      "transport": "stdio"
    },
    "remote_api": {
      "transport": "sse",
      "url": "https://mcp.example.com/mcp?api_key=$REMOTE_MCP_KEY",
      "headers": { "Authorization": "Bearer ${REMOTE_MCP_TOKEN}" },
      "enabled": true
    }
  }
}
```

### Server Configuration Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `command` | string | stdio only | Executable to run |
| `args` | array | no | Command-line arguments |
| `env` | object | no | Environment variables |
| `enabled` | boolean | no | Default: true |
| `transport` | string | no | `"stdio"` or `"sse"`, default: `"stdio"` |
| `url` | string | SSE only | Remote server URL |
| `headers` | object | SSE only | HTTP headers |
| `oauth` | object | no | Manual OAuth configuration |

### Transport Types

#### stdio (`transport.rs:1-249`)
- Local process via stdin/stdout
- Uses `tokio::process::Command` to spawn child process
- JSON-RPC 2.0 communication over piped stdin/stdout
- Server stderr captured and logged as warnings

#### Streamable HTTP/SSE (`sse_transport.rs:1-896`)
- Remote server via HTTP
- POST requests for JSON-RPC
- GET for SSE event streams
- Supports OAuth Bearer token injection
- Session ID management for stateful connections
- MCP protocol version header: `MCP-Protocol-Version`
- Session header: `Mcp-Session-Id`

---

## Part 5: API & Interfaces

### MCP Protocol Features Supported

#### JSON-RPC 2.0 (`types.rs:9-53`)
- `JsonRpcRequest` - Request with id, method, params
- `JsonRpcResponse` - Response with result or error
- `JsonRpcNotification` - Fire-and-forget messages
- `JsonRpcError` - Error with code, message, data

#### Protocol Handshake (`client.rs:143-179`)
```rust
// Initialize request (types.rs:67-73)
InitializeParams {
    protocol_version: String,
    capabilities: ClientCapabilities,
    client_info: ClientInfo,
}

// Response (types.rs:82-88)
InitializeResult {
    protocol_version: String,
    capabilities: ServerCapabilities,
    server_info: ServerInfo,
}
```

#### Tool Definitions (`types.rs:114-128`)
```rust
// Tool definition (types.rs:115-122)
pub struct McpToolDef {
    pub name: String,
    pub description: Option<String>,
    pub input_schema: serde_json::Value,
}

// Tools list result (types.rs:125-128)
pub struct ToolsListResult {
    pub tools: Vec<McpToolDef>,
}
```

#### Tool Execution (`types.rs:130-160`)
```rust
// Tool call parameters (types.rs:131-135)
pub struct ToolsCallParams {
    pub name: String,
    pub arguments: serde_json::Value,
}

// Tool result (types.rs:154-160)
pub struct ToolsCallResult {
    pub content: Vec<ToolContent>,  // Text, Image, or Resource
    pub is_error: bool,
}
```

### Trait Abstractions (`traits.rs`)

```rust
// Transport layer (traits.rs:18-31)
#[async_trait]
pub trait McpTransport: Send + Sync {
    async fn request(&self, method: &str, params: Option<Value>) -> Result<JsonRpcResponse>;
    async fn notify(&self, method: &str, params: Option<Value>) -> Result<()>;
    async fn is_alive(&self) -> bool;
    async fn kill(&self);
}

// Client layer (traits.rs:37-59)
#[async_trait]
pub trait McpClientTrait: Send + Sync {
    fn server_name(&self) -> &str;
    fn state(&self) -> McpClientState;
    fn tools(&self) -> &[McpToolDef];
    async fn list_tools(&mut self) -> Result<&[McpToolDef]>;
    async fn call_tool(&self, name: &str, arguments: Value) -> Result<ToolsCallResult>;
    async fn is_alive(&self) -> bool;
    async fn shutdown(&mut self);
}
```

### Resource Access
Server capabilities include resources (see `ServerCapabilities` in `types.rs:90-98`), though the current implementation focuses on tools.

---

## Part 6: Configuration & Settings

### User Configuration in moltis.toml

```toml
[mcp.servers.filesystem]
command = "npx"
args = ["-y", "@modelcontextprotocol/server-filesystem", "/Users/me/projects"]

[mcp.servers.github]
command = "npx"
args = ["-y", "@modelcontextprotocol/server-github"]
env = { GITHUB_TOKEN = "ghp_..." }

[mcp.servers.remote_api]
transport = "sse"
url = "https://mcp.example.com/mcp?api_key=$REMOTE_MCP_KEY"
headers = { Authorization = "Bearer ${REMOTE_MCP_TOKEN}" }
```

### Environment Placeholder Substitution (`remote.rs:113-176`)

Both URLs and headers support `$NAME` and `${NAME}` placeholders:
- Resolved from Moltis-managed environment overrides
- Can be set via `[env]` in config or Settings → Environment Variables
- Placeholder syntax validated in `remote.rs:226-242`

### Configuration Validation
- Command existence checked at startup via `doctor_commands.rs`
- URL required for SSE transport
- Command required for stdio transport

---

## Part 7: Lifecycle Management

### Connection Management

#### Client States (`client.rs:27-37`)
```rust
pub enum McpClientState {
    Connected,     // Transport spawned, not yet initialized
    Ready,         // Initialize completed, initialized notification sent
    Authenticating, // OAuth authentication in progress
    Closed,        // Server process exited or shut down
}
```

#### Server Lifecycle Flow
```
Start → Initialize → Ready → [Tool Calls] → Stop
         │                    │
         ▼                    ▼
   Health Check ◄──────── Heartbeat
         │                    │
         ▼                    ▼
  Crash Detected ────────► Restart
                               │
                         Backoff Wait
```

### Reconnection Handling (`manager.rs:113-252`)

1. **Automatic start of enabled servers**: `start_enabled()` at `manager.rs:114-133`
2. **Graceful restart**: `restart_server()` at `manager.rs:270-280`
3. **OAuth token refresh**: `try_refresh()` in `auth.rs:175-224`
   - Uses refresh token if available
   - Falls back to interactive flow if refresh fails

### Error Handling (`error.rs:1-58`)

```rust
pub enum Error {
    Io(std::io::Error),
    Reqwest(reqwest::Error),
    SerdeJson(serde_json::Error),
    UrlParse(url::ParseError),
    Transport(McpTransportError),
    Manager(McpManagerError),
    Message { message: String },
    External { context: String, source: Box<dyn StdError + Send + Sync> },
}

pub enum McpManagerError {
    OAuthRequired { server: String },
    OAuthStateNotFound,
    NotSseTransport { server: String },
    MissingSseUrl { server: String },
    ServerNotFound { server: String },
}

pub enum McpTransportError {
    Unauthorized { www_authenticate: Option<String> },
    HttpError { status: u16, body: String },
    Other { source: Box<dyn StdError + Send + Sync> },
}
```

### Health Monitoring (`manager.rs:356-410`)

- Process health checked via `is_alive()` on transport
- Status reported via `status_all()` and `status()` methods
- States: running, connecting, authenticating, stopped, dead

---

## Part 8: Distribution & Packaging

### How MCP Servers Are Distributed

1. **npm packages** (for Node.js servers):
   ```bash
   npx @modelcontextprotocol/server-filesystem
   npx @modelcontextprotocol/server-github
   npx @modelcontextprotocol/server-postgres
   ```

2. **Python packages**:
   ```bash
   pip install mcp-server-filesystem
   ```

3. **Custom executables**: Any stdio-based server

4. **Remote HTTP services**: Streamable HTTP endpoints

### Installation Method

Servers are configured in `moltis.toml` or via the web UI (Settings → MCP Servers). No formal package manager within Moltis - external package managers (npm, pip) handle installation.

### Popular MCP Servers

| Server | Description | Install |
|--------|-------------|---------|
| filesystem | Read/write local files | `npx @modelcontextprotocol/server-filesystem` |
| github | GitHub API access | `npx @modelcontextprotocol/server-github` |
| postgres | PostgreSQL queries | `npx @modelcontextprotocol/server-postgres` |
| sqlite | SQLite database | `npx @modelcontextprotocol/server-sqlite` |
| puppeteer | Browser automation | `npx @modelcontextprotocol/server-puppeteer` |
| brave-search | Web search | `npx @modelcontextprotocol/server-brave-search` |

---

## Part 9: Security & Permissions

### Network Restrictions

- **Reserved headers blocked** (`remote.rs:270-275`):
  - `accept`
  - `content-type`
  - `mcp-protocol-version`
  - `mcp-session-id`

- **SSRF protection**:
  - URLs validated via `url::Url::parse()` in `remote.rs:36-37`
  - Display URL sanitization in `remote.rs:67-107`

### Secrets Management

- **Secret types**: Uses `secrecy::Secret<String>` for sensitive values
- **URL sanitization** (`remote.rs:67-107`):
  - Username/password redacted
  - Query parameters redacted (except placeholders)
  - Display shows `[REDACTED]` for query values

- **Header sanitization**:
  - Only header names exposed in UI and status
  - Values stored in secret-aware types

### Security Considerations (from docs)

```admonition warning
MCP servers run with the same permissions as Moltis. Only use servers from trusted sources.
```

- Review server code before running
- Limit file access - use specific paths, not `/`
- Use environment variables for secrets
- Prefer placeholders in remote URLs/headers
- Network isolation - run untrusted servers in containers

---

## Part 10: Developer Experience

### CLI Tools for MCP

While there are no dedicated MCP CLI tools in the aioops suite, MCP server availability is checked via:
- `mcp-scan` command availability check in `gateway/src/services.rs:847`

### Debugging MCP Connections

#### Check Server Status
Web UI: Settings → MCP Servers
- Connection status (connected/disconnected/error)
- Available tools
- Sanitized remote URL and header names
- Recent errors

#### View Logs
```bash
# View gateway logs
tail -f ~/.moltis/logs.jsonl | grep -i mcp
```

#### Test Locally
```bash
# JSON-RPC tool list request
echo '{"jsonrpc":"2.0","method":"tools/list","id":1}' | node server.js
```

### MCP Tool Naming

Tools are prefixed with `mcp__<server>__` (double underscore) to avoid conflicts:
- Example: `mcp__filesystem__read_file`
- Format validated in `tool_bridge.rs:229-239`

---

## Part 11: Code Examples

### Minimal MCP Server Configuration

#### stdio transport (moltis.toml):
```toml
[mcp.servers.my_server]
command = "node"
args = ["server.js"]
```

#### SSE transport (moltis.toml):
```toml
[mcp.servers.remote_api]
transport = "sse"
url = "https://mcp.example.com/mcp"
headers = { "x-api-key" = "$REMOTE_MCP_KEY" }
```

### Working MCP Server Example

#### Node.js Server (from docs):
```javascript
// server.js
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new Server(
  { name: "my-server", version: "1.0.0" },
  { capabilities: { tools: {} } }
);

server.setRequestHandler("tools/list", async () => ({
  tools: [{
    name: "hello",
    description: "Says hello",
    inputSchema: {
      type: "object",
      properties: {
        name: { type: "string", description: "Name to greet" }
      },
      required: ["name"]
    }
  }]
}));

server.setRequestHandler("tools/call", async (request) => {
  if (request.params.name === "hello") {
    const name = request.params.arguments.name;
    return { content: [{ type: "text", text: `Hello, ${name}!` }] };
  }
});

const transport = new StdioServerTransport();
await server.connect(transport);
```

#### Configure in Moltis:
```toml
[mcp.servers.my_server]
command = "node"
args = ["server.js"]
```

### OAuth Configuration Example

```toml
[mcp.servers.private_api]
url = "https://mcp.example.com/mcp"
transport = "sse"

[mcp.servers.private_api.oauth]
client_id = "your-client-id"
auth_url = "https://auth.example.com/authorize"
token_url = "https://auth.example.com/token"
scopes = ["mcp:read", "mcp:write"]
```

---

## Architecture Summary

```
┌─────────────────────────────────────────────────────────────────┐
│                        Moltis Gateway                           │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    McpManager                           │   │
│  │  - Registry (McpRegistry)                               │   │
│  │  - Clients (HashMap<name, McpClient>)                  │   │
│  │  - Auth Providers (OAuth)                               │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
           │                                    │
           ▼                                    ▼
┌──────────────────────┐           ┌──────────────────────────┐
│   StdioTransport    │           │      SseTransport        │
│  - Spawn process    │           │  - HTTP POST requests    │
│  - stdin/stdout     │           │  - SSE event streams     │
│  - JSON-RPC 2.0     │           │  - OAuth injection        │
└──────────────────────┘           └──────────────────────────┘
           │                                    │
           ▼                                    ▼
┌──────────────────────┐           ┌──────────────────────────┐
│  Local MCP Server    │           │   Remote MCP Server     │
│  (npm, Python, etc) │           │   (HTTP endpoint)       │
└──────────────────────┘           └──────────────────────────┘
```

---

## File Reference Index

| File | Purpose |
|------|---------|
| `crates/mcp/src/lib.rs` | Module exports and documentation |
| `crates/mcp/src/types.rs` | Protocol types (JSON-RPC, MCP structures) |
| `crates/mcp/src/client.rs` | MCP client implementation |
| `crates/mcp/src/transport.rs` | stdio transport implementation |
| `crates/mcp/src/sse_transport.rs` | HTTP/SSE transport implementation |
| `crates/mcp/src/manager.rs` | Lifecycle management |
| `crates/mcp/src/registry.rs` | Server configuration persistence |
| `crates/mcp/src/auth.rs` | OAuth 2.1 authentication |
| `crates/mcp/src/remote.rs` | Remote config resolution, sanitization |
| `crates/mcp/src/tool_bridge.rs` | Tool adaptation for agent system |
| `crates/mcp/src/traits.rs` | Transport and client trait abstractions |
| `crates/mcp/src/error.rs` | Error types |
| `crates/gateway/src/mcp_service.rs` | Gateway integration |
| `docs/src/mcp.md` | User documentation |
