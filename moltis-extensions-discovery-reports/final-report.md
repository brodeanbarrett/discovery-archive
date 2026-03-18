# Extension Discovery Report: Moltis

**Repository:** git@github.com:moltis-org/moltis.git  
**Audit Date:** 2026-03-18  
**Target:** Discover extension/plugin mechanisms in the moltis codebase

---

## 1. Executive Summary

Moltis provides **three distinct extension mechanisms** for extending agent capabilities:

| Mechanism | Purpose | Type |
|-----------|---------|------|
| **Hooks** | Observe, modify, or block lifecycle events | Event-based |
| **Skills** | Add prompt templates and tool restrictions | Package-based |
| **MCP Servers** | Connect to external tool providers | Protocol-based |

**Key Findings:**
- **Hooks**: 15 lifecycle events, shell-based or native Rust handlers, filesystem-based discovery
- **Skills**: Follows Agent Skills open standard, trust-based security model, supports Claude Code/Codex plugins
- **MCP**: Full JSON-RPC 2.0 protocol (2024-11-05), stdio and SSE transport support, OAuth authentication

**Ease of Extension Creation:**
- **Hooks**: Easy - simple shell scripts with HOOK.md manifest
- **Skills**: Easy - SKILL.md with YAML frontmatter
- **MCP**: Moderate - requires implementing MCP protocol server

---

## 2. Extension Mechanisms Overview

### 2.1 Hooks

- **Official Terminology:** "hooks", "hook handlers", "shell hooks"
- **Purpose:** Observe and modify agent behavior at 15 lifecycle events
- **Prevalence:** Primary extension mechanism for runtime behavior modification
- **Documentation Quality:** Excellent - `docs/src/hooks.md` with examples
- **File Locations:**
  - Core: `crates/plugins/src/`, `crates/common/src/hooks.rs`
  - Docs: `docs/src/hooks.md`
  - Examples: `examples/hooks/`

### 2.2 Skills

- **Official Terminology:** "skills", "skill packages"
- **Purpose:** Extend agent capabilities via prompt templates and tool restrictions
- **Prevalence:** Secondary mechanism for adding persistent agent behaviors
- **Documentation Quality:** Good - `docs/src/skills-security.md`, `skill-tools.md`
- **File Locations:**
  - Core: `crates/skills/src/`
  - Docs: `docs/src/skills-security.md`, `docs/src/skill-tools.md`

### 2.3 MCP Servers

- **Official Terminology:** "MCP servers", "MCP"
- **Purpose:** Connect external tool/resource providers via Model Context Protocol
- **Prevalence:** Tertiary mechanism for external integrations
- **Documentation Quality:** Good - `docs/src/mcp.md`
- **File Locations:**
  - Core: `crates/mcp/src/`
  - Docs: `docs/src/mcp.md`

---

## 3. Detailed Mechanism Documentation

---

### 3.1 Hooks - Deep Dive

#### 3.1.1 Overview

Hooks provide event-based extensibility with 15 lifecycle events spanning tool calls, messages, and sessions. They support both shell-based external handlers and native Rust bundled hooks.

#### 3.1.2 Terminology

| Term | Definition |
|------|------------|
| **Hook** | An extension point that observes, modifies, or blocks actions at key lifecycle events |
| **Hook Handler** | A trait (`HookHandler`) implemented by both native Rust hooks and shell-based external hooks |
| **Hook Event** | A lifecycle event (`HookEvent` enum) that triggers hook execution |
| **Shell Hook** | External command-based hook executed via `sh -c` |
| **Bundled Hook** | Native Rust hooks compiled directly into the binary |
| **Hook Payload** | Typed JSON data (`HookPayload` enum) passed to handlers |

#### 3.1.3 Discovery & Loading

**How extensions are discovered:**

Hooks are discovered from `HOOK.md` files in configured directories.

**Discovery Paths (in priority order):**
1. **Project-local:** `<workspace>/.moltis/hooks/<name>/HOOK.md`
2. **User-global:** `~/.moltis/hooks/<name>/HOOK.md`

Source: `moltis/crates/plugins/src/hook_discovery.rs:38-47`

**Entry points:**

| File | Purpose |
|------|---------|
| `HOOK.md` | Hook manifest with TOML frontmatter metadata |
| `handler.sh` | Shell script executed for shell-based hooks |

```rust
// From hook_discovery.rs:38-47
pub fn default_paths() -> Vec<(PathBuf, HookSource)> {
    let workspace_root = moltis_config::data_dir();
    vec![
        (workspace_root.join(".moltis/hooks"), HookSource::Project),
        (moltis_config::data_dir().join("hooks"), HookSource::User),
    ]
}
```

#### 3.1.4 Directory & File Structure

**Required structure:**
```
~/.moltis/hooks/<hook-name>/
├── HOOK.md          # Required: manifest + documentation
└── handler.sh       # Optional: shell handler script
```

**Full working skeleton:**
```markdown
+++
name = "my-hook"
description = "What it does"
events = ["BeforeToolCall"]
command = "./handler.sh"
timeout = 5
priority = 10

[requires]
os = ["darwin", "linux"]
bins = ["jq"]
env = ["WEBHOOK_URL"]
+++

# My Hook

Extended documentation goes here.
```

Source: `moltis/crates/plugins/src/hook_metadata.rs:1-20`

#### 3.1.5 Manifest / Configuration Schema

**Manifest file format:** Markdown with TOML frontmatter

**Required fields:**
| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Unique hook identifier |
| `events` | array | List of `HookEvent` names to subscribe |

Source: `moltis/crates/plugins/src/hook_metadata.rs:42-60`

**Optional fields:**
| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `description` | string | `""` | Human-readable description |
| `emoji` | string | none | Icon for display |
| `command` | string | none | Shell command to execute |
| `timeout` | u64 | `10` | Execution timeout in seconds |
| `priority` | i32 | `0` | Execution order (higher first) |
| `env` | table | `{}` | Environment variables for hook |
| `requires.os` | array | `[]` | Required operating systems |
| `requires.bins` | array | `[]` | Required binaries in PATH |
| `requires.env` | array | `[]` | Required environment variables |
| `requires.config` | array | `[]` | Required config values |

#### 3.1.6 API Reference

**HookHandler Trait:**

Defined in `moltis/crates/common/src/hooks.rs:225-257`:

```rust
#[async_trait]
pub trait HookHandler: Send + Sync {
    fn name(&self) -> &str;
    fn events(&self) -> &[HookEvent];
    fn priority(&self) -> i32 { 0 }
    async fn handle(&self, event: HookEvent, payload: &HookPayload) -> Result<HookAction>;
    fn handle_sync(&self, event: HookEvent, payload: &HookPayload) -> Result<HookAction>;
}
```

**Hook Event Types (15 Lifecycle Events):**

Source: `moltis/crates/common/src/hooks.rs:28-47`

**Modifying Events (Sequential):**
| Event | Description | Can Modify | Can Block |
|-------|-------------|------------|-----------|
| `BeforeAgentStart` | Before agent loop starts | yes | yes |
| `BeforeLLMCall` | Before prompt sent to LLM | yes | yes |
| `AfterLLMCall` | After LLM response | yes | yes |
| `BeforeToolCall` | Before tool execution | yes | yes |
| `BeforeCompaction` | Before context compaction | yes | yes |
| `MessageSending` | Before sending response | yes | yes |
| `ToolResultPersist` | When tool result is persisted | yes | yes |

**Read-Only Events (Parallel):**
| Event | Description |
|-------|-------------|
| `AfterToolCall` | After tool completes |
| `AfterCompaction` | After context compaction |
| `AgentEnd` | When agent loop completes |
| `MessageReceived` | When user message arrives |
| `MessageSent` | After response delivered |
| `SessionStart` | When new session begins |
| `SessionEnd` | When session ends |
| `GatewayStart` | When Moltis starts |
| `GatewayStop` | When Moltis shuts down |
| `Command` | When slash command used |

**Shell Hook Protocol:**

Input (stdin): JSON payload
```json
{
  "event": "BeforeToolCall",
  "data": {
    "tool": "bash",
    "arguments": { "command": "ls -la" }
  },
  "session_id": "abc123",
  "timestamp": "2024-01-15T10:30:00Z"
}
```

Output (stdout + exit code):
| Exit Code | Stdout | Result |
|-----------|--------|--------|
| `0` | (empty) | Continue normally |
| `0` | `{"action":"modify","data":{...}}` | Replace payload data |
| `1` | — | Block (stderr = reason) |

#### 3.1.7 Lifecycle

**Load sequence:**
1. Bundled hooks (compiled into binary)
2. Project-local hooks (`<workspace>/.moltis/hooks/`)
3. User-global hooks (`~/.moltis/hooks/`)

Project-local hooks take precedence over global hooks with the same name.

**Lifecycle events:**
- **Activation**: Automatic when requirements are met (OS, bins, env vars)
- **Deactivation**: Automatic when requirements unmet (not error)
- **Error handling**: Circuit breaker - auto-disable after 5 consecutive failures, 60s cooldown

#### 3.1.8 Distribution

**Package format:** Shell scripts with HOOK.md manifest in a directory

**Installation:**
```bash
# Copy hook directory to hooks folder
cp -r /path/to/hook ~/.moltis/hooks/<hook-name>/

# Make handler executable
chmod +x ~/.moltis/hooks/<hook-name>/handler.sh
```

#### 3.1.9 Security

**What hooks can do:**
1. Read all hook payloads (full access to session data)
2. Modify payloads for modifying events
3. Block actions for blocking events
4. Execute arbitrary shell commands
5. Access environment variables
6. Write to filesystem (if hook script has permissions)

**What hooks cannot do (built-in protections):**
1. **Timeout protection**: Default 10s, configurable per hook
2. **Circuit breaker**: Auto-disable after 5 consecutive failures
3. **Eligibility requirements**: Can specify required binaries/env vars

#### 3.1.10 Developer Experience

**CLI Commands:**
```bash
# List all discovered hooks
moltis hooks list

# List only eligible hooks (requirements met)
moltis hooks list --eligible

# Output as JSON
moltis hooks list --json

# Show details for a specific hook
moltis hooks info my-hook
```

#### 3.1.11 Code Examples

**Minimal Hook:**
```markdown
+++
name = "my-hook"
description = "Logs all tool calls"
events = ["BeforeToolCall"]
command = "./handler.sh"
+++
```

```bash
#!/bin/bash
payload=$(cat)
event=$(echo "$payload" | jq -r '.event')
echo "$(date) $event" >> /tmp/hook-log.txt
exit 0
```

**Block Dangerous Commands:**
```bash
#!/usr/bin/env bash
set -euo pipefail

INPUT=$(cat)

# Only inspect exec tool calls
TOOL_NAME=$(printf '%s' "$INPUT" | grep -o '"tool_name":"[^"]*"' | head -1 | cut -d'"' -f4)
if [ "$TOOL_NAME" != "exec" ]; then
    exit 0
fi

# Extract command
COMMAND=$(printf '%s' "$INPUT" | grep -o '"command":"[^"]*"' | head -1 | cut -d'"' -f4)
if [ -z "$COMMAND" ]; then
    exit 0
fi

# Check with dcg
DCG_INPUT=$(printf '{"tool_name":"Bash","tool_input":{"command":"%s"}}' "$COMMAND")
DCG_RESULT=$(printf '%s' "$DCG_INPUT" | dcg 2>&1) || {
    echo "$DCG_RESULT" >&2
    exit 1
}

exit 0
```

Source: `moltis/examples/hooks/dcg-guard/handler.sh`

---

### 3.2 Skills - Deep Dive

#### 3.2.1 Overview

Skills provide a package-based extension mechanism following the Agent Skills open standard. They allow developers to add prompt templates and restrict which tools skills can use.

#### 3.2.2 Terminology

| Term | Definition |
|------|------------|
| **Skill** | A directory containing a `SKILL.md` file with YAML frontmatter and markdown instructions |
| **Skill Package** | A collection of one or more skills, typically distributed as a GitHub repository |
| **SkillMetadata** | Lightweight metadata parsed from SKILL.md frontmatter, loaded at startup |
| **SkillContent** | Full skill content: metadata + markdown body, loaded on demand |
| **SkillSource** | Enum indicating where a skill was discovered: Project, Personal, Plugin, Registry |

#### 3.2.3 Discovery & Loading

**How extensions are discovered:**

The discovery system uses the `SkillDiscoverer` trait with a filesystem-based implementation.

**Default Search Paths** (`discover.rs:33-42`):
```rust
pub fn default_paths() -> Vec<(PathBuf, SkillSource)> {
    vec![
        (workspace_root.join(".moltis/skills"), SkillSource::Project),
        (data.join("skills"), SkillSource::Personal),
        (data.join("installed-skills"), SkillSource::Registry),
        (data.join("installed-plugins"), SkillSource::Plugin),
    ]
}
```

| Path | Source Type | Description |
|------|-------------|-------------|
| `~/.moltis/skills/` | Project | Project-local skills (one level deep) |
| `~/.moltis/data/skills/` | Personal | User's personal skills |
| `~/.moltis/data/installed-skills/` | Registry | Installed from marketplace |
| `~/.moltis/data/installed-plugins/` | Plugin | Bundled Claude Code plugins |

**Entry points:**
- **SKILL.md**: Markdown file with YAML frontmatter
- **Plugin formats**: `.claude-plugin/plugin.json` for Claude Code plugins

#### 3.2.4 Directory & File Structure

**Required structure:**
```
<skill-directory>/
└── SKILL.md          # Required: YAML frontmatter + markdown instructions
```

Multi-skill repos: Skills can be nested at any level (recursive scan)

#### 3.2.5 Manifest / Configuration Schema

**SKILL.md Format:**

```yaml
---
name: <skill-name>           # Required: lowercase, 1-64 chars, hyphens allowed
description: <description>   # Optional: short human-readable description
homepage: <url>              # Optional: homepage URL
license: <SPDX-id>           # Optional: SPDX license identifier
compatibility: <string>      # Optional: environment requirements
allowed_tools:               # Optional: tools this skill can use
  - tool1
  - tool2
dockerfile: <filepath>      # Optional: sandbox Dockerfile path
requires:                   # Optional: binary requirements
  bins:                     # All of these must be in PATH
    - binary1
  any_bins:                 # At least one must be in PATH
    - binary2
  install:                  # Installation instructions
    - kind: brew
      formula: <formula>
---
# Markdown body: skill instructions for the agent
```

**Name Validation:**
- lowercase alphanumeric, hyphens, 1-64 chars
- Valid examples: `my-skill`, `a`, `skill123`, `plugin:skill`

#### 3.2.6 API Reference

**SkillDiscoverer Trait** (`discover.rs:13-17`):
```rust
#[async_trait]
pub trait SkillDiscoverer: Send + Sync {
    async fn discover(&self) -> anyhow::Result<Vec<SkillMetadata>>;
}
```

**SkillRegistry Trait** (`registry.rs:12-25`):
```rust
#[async_trait]
pub trait SkillRegistry: Send + Sync {
    async fn list_skills(&self) -> anyhow::Result<Vec<SkillMetadata>>;
    async fn load_skill(&self, name: &str) -> anyhow::Result<SkillContent>;
    async fn install_skill(&self, source: &str) -> anyhow::Result<SkillMetadata>;
    async fn remove_skill(&self, name: &str) -> anyhow::Result<()>;
}
```

**Prompt Generation:**
```xml
## Available Skills

<available_skills>
<skill name="commit" source="skill" path="/path/to/SKILL.md">
Create git commits
</skill>
</available_skills>
```

#### 3.2.7 Lifecycle

**Load order:**
1. On startup, `FsSkillDiscoverer.discover()` scans all configured paths
2. For Project/Personal sources: scans one level deep for `SKILL.md` files
3. For Registry sources: filters by enabled/trusted state from manifest
4. For Plugin sources: uses plugins manifest for filtering
5. `generate_skills_prompt()` injects available skills into system prompt

**Trust lifecycle:**
```
installed → trusted → enabled
```

- **installed**: Repo is on disk
- **trusted**: User explicitly marked skill as reviewed
- **enabled**: Skill is active for agent use

**Security rule**: You cannot enable untrusted skills.

#### 3.2.8 Distribution

**Package format:** GitHub repositories containing SKILL.md files

**Installation:**
```bash
moltis skills install <owner/repo>
```

Downloads tarball from `https://api.github.com/repos/{owner}/{repo}/tarball`

**Supported Formats:**
| Format | Detection | Description |
|--------|-----------|-------------|
| `Skill` | Root or nested `SKILL.md` | Native format |
| `ClaudeCode` | `.claude-plugin/plugin.json` | Claude Code plugins |
| `Codex` | `codex-plugin.json` | Codex plugins |
| `Generic` | Fallback | Treats `.md` files as prompts |

#### 3.2.9 Security

**Provenance Pinning:**
- Records `commit_sha` for installed repos
- UI shows short SHA for provenance review

**Re-Trust on Drift:**
If local HEAD differs from pinned `commit_sha`:
- Auto-mark `trusted=false`
- Auto-disable all skills in repo
- Re-enable blocked until explicit trust

**Security Guardrails:**
- `confirm=true` required for dependency installs
- Host installs blocked when sandbox mode is off
- Suspicious command chains blocked by default

**Emergency Kill Switch:**
`skills.emergency_disable` - immediately disables all third-party skills/plugins

#### 3.2.10 Developer Experience

**CLI Tools:**
| Command | Description |
|---------|-------------|
| `moltis skills install <repo>` | Install a skill repo |
| `moltis skills remove <source>` | Remove an installed repo |
| `moltis skills list` | List available skills |

**Agent Tools:**
| Tool | Description |
|------|-------------|
| `create_skill` | Write new SKILL.md to `.moltis/skills/<name>/` |
| `update_skill` | Overwrite existing skill's SKILL.md |
| `delete_skill` | Remove a skill directory |

#### 3.2.11 Code Examples

**Minimal Skill:**
```yaml
---
name: hello-world
description: A simple skill that greets the user
---
# Hello World Skill

You are a friendly assistant. When the user says hello, respond with a warm greeting and ask how you can help.
```

**Working Skill with Full Manifest:**
```yaml
---
name: commit-helper
description: Create semantic git commits with conventional commits format
homepage: https://github.com/example/commit-helper
license: MIT
compatibility: Requires git installed
allowed_tools:
  - exec
  - read
requires:
  bins:
    - git
  install:
    - kind: brew
      formula: git
      os: [darwin, linux]
---
# Commit Helper

You help create well-structured git commits using the conventional commits specification.

## Rules
1. Use format: `<type>(<scope>): <description>`
2. Types: feat, fix, docs, style, refactor, test, chore
3. Include body with motivation and contrast
4. Use imperative mood

## When to use
- When user wants to commit changes
- When user asks about commit messages
```

---

### 3.3 MCP Servers - Deep Dive

#### 3.3.1 Overview

MCP (Model Context Protocol) servers provide a standardized way to connect external tool providers. Moltis implements JSON-RPC 2.0 protocol version 2024-11-05 with support for both stdio and SSE transports.

#### 3.3.2 Terminology

| Term | Definition |
|------|------------|
| **MCP** | Model Context Protocol, an open protocol for connecting AI assistants to external tools |
| **MCP servers** | External tool servers that implement the MCP specification |
| **Tools** | Functions MCP servers expose that agents can call |
| **Transport** | The communication mechanism (stdio or SSE/HTTP) |

#### 3.3.3 Discovery & Loading

**How extensions are discovered:**

MCP servers are discovered through two mechanisms:

1. **Configuration File (`moltis.toml`)**: Server definitions in `[mcp.servers]` section
2. **Import from OpenClaw**: Legacy `mcp-servers.json` format support

**Entry point files:**
- **Primary config**: `mcp-servers.json` in data directory (`~/.moltis/`)
- **User config**: `moltis.toml` with `[mcp.servers]` section

#### 3.3.4 Directory & File Structure

**Structure for MCP Server Definition:**
```rust
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

Required configuration:
- **For stdio transport**: `command` (required), `args` (optional)
- **For SSE transport**: `transport = "sse"`, `url` (required)

#### 3.3.5 Manifest / Configuration Schema

**mcp-servers.json Format:**
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

**Configuration Fields:**
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

#### 3.3.6 API Reference

**JSON-RPC 2.0 Types:**
```rust
// Request with id, method, params
InitializeParams {
    protocol_version: String,
    capabilities: ClientCapabilities,
    client_info: ClientInfo,
}

// Response
InitializeResult {
    protocol_version: String,
    capabilities: ServerCapabilities,
    server_info: ServerInfo,
}
```

**Tool Definitions:**
```rust
pub struct McpToolDef {
    pub name: String,
    pub description: Option<String>,
    pub input_schema: serde_json::Value,
}
```

**Transport Layer:**
```rust
#[async_trait]
pub trait McpTransport: Send + Sync {
    async fn request(&self, method: &str, params: Option<Value>) -> Result<JsonRpcResponse>;
    async fn notify(&self, method: &str, params: Option<Value>) -> Result<()>;
    async fn is_alive(&self) -> bool;
    async fn kill(&self);
}
```

#### 3.3.7 Lifecycle

**Client States:**
```rust
pub enum McpClientState {
    Connected,     // Transport spawned, not yet initialized
    Ready,         // Initialize completed, initialized notification sent
    Authenticating, // OAuth authentication in progress
    Closed,        // Server process exited or shut down
}
```

**Server Lifecycle Flow:**
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

**Reconnection Handling:**
1. Automatic start of enabled servers
2. Graceful restart with backoff
3. OAuth token refresh

#### 3.3.8 Distribution

**How MCP servers are distributed:**

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

**Popular MCP Servers:**
| Server | Description |
|--------|-------------|
| filesystem | Read/write local files |
| github | GitHub API access |
| postgres | PostgreSQL queries |
| sqlite | SQLite database |
| puppeteer | Browser automation |
| brave-search | Web search |

#### 3.3.9 Security

**Network Restrictions:**
- **Reserved headers blocked**:
  - `accept`, `content-type`, `mcp-protocol-version`, `mcp-session-id`

- **SSRF protection**:
  - URLs validated via `url::Url::parse()`
  - Display URL sanitization

**Secrets Management:**
- Uses `secrecy::Secret<String>` for sensitive values
- URL sanitization - username/password redacted, query parameters redacted
- Header sanitization - only header names exposed in UI and status

**Warning:**
```
MCP servers run with the same permissions as Moltis. Only use servers from trusted sources.
```

#### 3.3.10 Developer Experience

**Debugging MCP Connections:**

**Check Server Status:**
Web UI: Settings → MCP Servers
- Connection status (connected/disconnected/error)
- Available tools
- Sanitized remote URL and header names
- Recent errors

**View Logs:**
```bash
# View gateway logs
tail -f ~/.moltis/logs.jsonl | grep -i mcp
```

**MCP Tool Naming:**
Tools are prefixed with `mcp__<server>__` (double underscore) to avoid conflicts:
- Example: `mcp__filesystem__read_file`

#### 3.3.11 Code Examples

**Minimal MCP Server Configuration (stdio):**
```toml
[mcp.servers.my_server]
command = "node"
args = ["server.js"]
```

**Minimal MCP Server Configuration (SSE):**
```toml
[mcp.servers.remote_api]
transport = "sse"
url = "https://mcp.example.com/mcp"
headers = { "x-api-key" = "$REMOTE_MCP_KEY" }
```

**Node.js Server:**
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

**OAuth Configuration:**
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

## 4. Summary Comparison

| Aspect | Hooks | Skills | MCP |
|--------|-------|--------|-----|
| **Type** | Event-based | Package-based | Protocol-based |
| **Discovery** | Filesystem (`HOOK.md`) | Filesystem (`SKILL.md`) | Config file |
| **Entry Point** | `HOOK.md` + handler | `SKILL.md` | `mcp-servers.json` |
| **Complexity** | Easy | Easy | Moderate |
| **Language** | Shell/Rust | Markdown+YAML | Any (JSON-RPC) |
| **Security Model** | Requirements + circuit breaker | Trust lifecycle | SSRF + secrets |
| **Distribution** | Directory copy | GitHub repos | npm/pip/custom |

---

## 5. Key File Reference

| Component | Files |
|-----------|-------|
| Hooks Core | `crates/plugins/src/`, `crates/common/src/hooks.rs` |
| Hooks Docs | `docs/src/hooks.md` |
| Hooks Examples | `examples/hooks/` |
| Skills Core | `crates/skills/src/` |
| Skills Docs | `docs/src/skills-security.md`, `skill-tools.md` |
| MCP Core | `crates/mcp/src/` |
| MCP Docs | `docs/src/mcp.md` |

---

*Report generated: 2026-03-18*
