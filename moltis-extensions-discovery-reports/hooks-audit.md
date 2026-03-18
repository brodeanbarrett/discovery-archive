# Hooks/Plugins Extension Mechanism Audit

**Target:** `./moltis-extensions-discovery/moltis/crates/plugins/`
**Docs:** `./moltis-extensions-discovery/moltis/docs/src/hooks.md`
**Examples:** `./moltis-extensions-discovery/moltis/examples/hooks/`

---

## Part 1: Extension Mechanism Identification

### Terms Used

| Term | Definition |
|------|------------|
| **Hook** | An extension point that observes, modifies, or blocks actions at key lifecycle events |
| **Hook Handler** | A trait (`HookHandler`) implemented by both native Rust hooks and shell-based external hooks |
| **Hook Event** | A lifecycle event (`HookEvent` enum) that triggers hook execution |
| **Shell Hook** | External command-based hook executed via `sh -c` |
| **Bundled Hook** | Native Rust hooks compiled directly into the binary |
| **Hook Payload** | Typed JSON data (`HookPayload` enum) passed to handlers |

### What Hooks Extend

Hooks extend three categories of agent behavior:

1. **Tool Calls** (`BeforeToolCall`, `AfterToolCall`, `ToolResultPersist`)
2. **Messages** (`MessageReceived`, `MessageSending`, `MessageSent`)
3. **Sessions** (`SessionStart`, `SessionEnd`, `Command`)

---

## Part 2: Entry Points & Discovery

### Hook Discovery Mechanism

Hooks are discovered from `HOOK.md` files in configured directories.

**Discovery Paths (in priority order):**

1. **Project-local:** `<workspace>/.moltis/hooks/<name>/HOOK.md`
2. **User-global:** `~/.moltis/hooks/<name>/HOOK.md`

Source: `moltis/crates/plugins/src/hook_discovery.rs:38-47`

```rust
pub fn default_paths() -> Vec<(PathBuf, HookSource)> {
    let workspace_root = moltis_config::data_dir();
    vec![
        (workspace_root.join(".moltis/hooks"), HookSource::Project),
        (moltis_config::data_dir().join("hooks"), HookSource::User),
    ]
}
```

### Entry Point Files

| File | Purpose |
|------|---------|
| `HOOK.md` | Hook manifest with TOML frontmatter metadata |
| `handler.sh` | Shell script executed for shell-based hooks |

### Hook Registration

Registration occurs via the `HookRegistry` in `moltis-common`:
- Shell hooks: Converted to `ShellHookHandler` via `ShellHookConfig`
- Bundled hooks: Registered directly as `Arc<dyn HookHandler>`

---

## Part 3: Extension Structure & Skeleton

### Directory Structure

```
~/.moltis/hooks/<hook-name>/
├── HOOK.md          # Required: manifest + documentation
└── handler.sh       # Optional: shell handler script
```

### Required Files

| File | Required | Purpose |
|------|----------|---------|
| `HOOK.md` | **Yes** | Contains TOML frontmatter metadata |
| `handler.sh` | No* | *Required only for shell-based hooks |

### HOOK.md Structure

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

---

## Part 4: Manifest & Metadata

### manifest.toml Format (HOOK.md Frontmatter)

**Required Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Unique hook identifier |
| `events` | array | List of `HookEvent` names to subscribe |

Source: `moltis/crates/plugins/src/hook_metadata.rs:42-60`

**Optional Fields:**

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

---

## Part 5: API & Interfaces

### HookHandler Trait

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

### Hook Event Types (15 Lifecycle Events)

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

### Environment Variables Available to Hooks

| Variable | Source | Description |
|----------|--------|-------------|
| All from `env` table in HOOK.md | User config | Custom environment variables |
| PATH | System | System binary search path |

---

## Part 6: Configuration & Settings

### Hook Configuration in moltis.toml

Source: `moltis/crates/plugins/src/hooks.rs:14-35`

```toml
[hooks]
[[hooks.hooks]]
name = "audit-log"
command = "./hooks/audit.sh"
events = ["BeforeToolCall", "AfterToolCall"]
timeout = 5
env = { LOG_FILE = "/tmp/audit.log" }

[[hooks.hooks]]
name = "notify-slack"
command = "./hooks/slack.sh"
events = ["SessionEnd"]
env = { SLACK_WEBHOOK_URL = "https://..." }
```

### ShellHookConfig Structure

```rust
pub struct ShellHookConfig {
    pub name: String,
    pub command: String,
    pub events: Vec<HookEvent>,
    pub timeout: u64,           // default: 10
    pub env: HashMap<String, String>,
}
```

---

## Part 7: Lifecycle Management

### Load Order

1. Bundled hooks (compiled into binary)
2. Project-local hooks (`<workspace>/.moltis/hooks/`)
3. User-global hooks (`~/.moltis/hooks/`)

Project-local hooks take precedence over global hooks with the same name.

Source: `moltis/crates/plugins/src/hook_discovery.rs:14-19`

### Activation/Deactivation

Hooks are automatically:
- **Activated** when requirements are met (OS, bins, env vars)
- **Skipped** (not error) when requirements unmet

Source: `moltis/crates/plugins/src/hook_eligibility.rs:19-60`

### Error Handling & Circuit Breaker

Source: `moltis/docs/src/hooks.md:321-329`

- **Threshold**: 5 consecutive failures
- **Cooldown**: 60 seconds
- **Recovery**: Auto-re-enabled after cooldown

```rust
// HookStats tracks failures for circuit breaker
pub struct HookStats {
    pub call_count: AtomicU64,
    pub failure_count: AtomicU64,
    pub consecutive_failures: AtomicU64,
    pub total_latency_us: AtomicU64,
    pub disabled: AtomicBool,
    pub disabled_at: Mutex<Option<Instant>>,
}
```

---

## Part 8: Distribution & Packaging

### Hook Packaging

Hooks are distributed as:
- **Shell scripts** with `HOOK.md` manifest in a directory
- No special packaging format—just filesystem directory

### Installation Method

```bash
# Copy hook directory to hooks folder
cp -r /path/to/hook ~/.moltis/hooks/<hook-name>/

# Make handler executable
chmod +x ~/.moltis/hooks/<hook-name>/handler.sh
```

---

## Part 9: Security & Permissions

### What Hooks Can Do

1. **Read** all hook payloads (full access to session data)
2. **Modify** payloads for modifying events
3. **Block** actions for blocking events
4. **Execute** arbitrary shell commands
5. **Access** environment variables
6. **Write** to filesystem (if hook script has permissions)

### What Hooks Cannot Do (Built-in Protections)

1. **Timeout protection**: Default 10s, configurable per hook
2. **Circuit breaker**: Auto-disable after 5 consecutive failures
3. **Eligibility requirements**: Can specify required binaries/env vars

### Shell Hook Protocol

Source: `moltis/docs/src/hooks.md:212-256`

**Input (stdin):** JSON payload

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

**Output (stdout + exit code):**

| Exit Code | Stdout | Result |
|-----------|--------|--------|
| `0` | (empty) | Continue normally |
| `0` | `{"action":"modify","data":{...}}` | Replace payload data |
| `1` | — | Block (stderr = reason) |

---

## Part 10: Developer Experience

### CLI Commands

Source: `moltis/docs/src/hooks.md:331-345`

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

### Debugging

Hooks can write to stderr (visible in logs) or to log files:

```bash
echo "debug info" >&2
```

---

## Part 11: Code Examples

### Example 1: Minimal Hook

**File:** `~/.moltis/hooks/my-hook/HOOK.md`

```markdown
+++
name = "my-hook"
description = "Logs all tool calls"
events = ["BeforeToolCall"]
command = "./handler.sh"
+++
```

**File:** `~/.moltis/hooks/my-hook/handler.sh`

```bash
#!/bin/bash
payload=$(cat)
event=$(echo "$payload" | jq -r '.event')
echo "$(date) $event" >> /tmp/hook-log.txt
exit 0
```

### Example 2: Block Dangerous Commands

Source: `moltis/examples/hooks/dcg-guard/handler.sh`

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

### Example 3: Modify Payload

Source: `moltis/docs/src/hooks.md:242-256`

```bash
#!/bin/bash
payload=$(cat)
tool=$(echo "$payload" | jq -r '.data.tool')

if [ "$tool" = "bash" ]; then
    # Add safety flag
    modified=$(echo "$payload" | jq '.data.arguments.command = "set -e; " + .data.arguments.command')
    echo "{\"action\":\"modify\",\"data\":$(echo "$modified" | jq '.data')}"
fi

exit 0
```

### Example 4: Session Notification

Source: `moltis/examples/hooks/notify-discord.sh`

```bash
#!/usr/bin/env bash
set -euo pipefail

if [ -z "${DISCORD_WEBHOOK_URL:-}" ]; then
    echo "DISCORD_WEBHOOK_URL not set, skipping" >&2
    exit 0
fi

INPUT=$(cat)
SESSION=$(echo "$INPUT" | grep -o '"session_key":"[^"]*"' | head -1 | cut -d'"' -f4)

curl -s -X POST "$DISCORD_WEBHOOK_URL" \
    -H 'Content-Type: application/json' \
    -d "{\"content\": \"Moltis session ended: ${SESSION}\"}"

exit 0
```

### Example 5: Native Rust Hook (Bundled)

Source: `moltis/crates/plugins/src/bundled/boot_md.rs`

```rust
use async_trait::async_trait;
use moltis_common::hooks::{HookAction, HookEvent, HookHandler, HookPayload, Result};

pub struct BootMdHook {
    workspace_dir: PathBuf,
}

#[async_trait]
impl HookHandler for BootMdHook {
    fn name(&self) -> &str {
        "boot-md"
    }

    fn events(&self) -> &[HookEvent] {
        &[HookEvent::GatewayStart]
    }

    fn priority(&self) -> i32 {
        100 // Run early
    }

    async fn handle(&self, _event: HookEvent, _payload: &HookPayload) -> Result<HookAction> {
        let boot_path = self.workspace_dir.join("BOOT.md");
        // Read and inject BOOT.md content
        Ok(HookAction::ModifyPayload(serde_json::json!({
            "boot_message": startup_message,
        })))
    }
}
```

---

## Summary

The Moltis hooks system provides a flexible extension mechanism with:

- **15 lifecycle events** for observing/modifying agent behavior
- **Dual handler types**: Shell-based (external commands) and Native (Rust)
- **Filesystem-based discovery** with priority ordering
- **TOML frontmatter manifest** for hook metadata
- **Eligibility checking** for OS, binaries, and environment variables
- **Circuit breaker** for fault tolerance
- **Action model**: Continue, ModifyPayload, Block

Key files:
- Core types: `moltis/crates/common/src/hooks.rs`
- Plugin implementation: `moltis/crates/plugins/src/`
- Documentation: `moltis/docs/src/hooks.md`
- Examples: `moltis/examples/hooks/`
