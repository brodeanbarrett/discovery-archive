# Skills Extension Mechanism Audit Report

## Overview

The skills system in Moltis provides an extensible framework for adding agent capabilities through pluggable skill packages. Skills follow the Agent Skills open standard and are implemented in the `moltis-skills` crate.

**Source Code**: `moltis/crates/skills/src/`
**Key Modules**: `lib.rs` (lines 1-17), `types.rs`, `discover.rs`, `parse.rs`, `install.rs`, `manifest.rs`, `registry.rs`, `prompt_gen.rs`, `formats.rs`, `requirements.rs`, `watcher.rs`, `migration.rs`

---

## Part 1: Extension Mechanism Identification

### Terms Used

| Term | Definition |
|------|------------|
| **Skill** | A directory containing a `SKILL.md` file with YAML frontmatter and markdown instructions |
| **Skill Package** | A collection of one or more skills, typically distributed as a GitHub repository |
| **SkillMetadata** | Lightweight metadata parsed from SKILL.md frontmatter, loaded at startup (`types.rs:126-157`) |
| **SkillContent** | Full skill content: metadata + markdown body, loaded on demand (`types.rs:221-224`) |
| **SkillSource** | Enum indicating where a skill was discovered: Project, Personal, Plugin, Registry (`types.rs:113-122`) |

### What Skills Extend

Skills extend **agent capabilities** by providing:
- **Prompt templates**: Instructions injected into the agent's system prompt
- **Tool definitions**: Via `allowed_tools` field restricting which tools the skill can use
- **Binary requirements**: External tool dependencies declared in the manifest

Reference: `lib.rs:3-4` - "Skills are directories containing a `SKILL.md` file with YAML frontmatter and markdown instructions, following the Agent Skills open standard."

---

## Part 2: Entry Points & Discovery

### How Moltis Discovers Skills

The discovery system uses the `SkillDiscoverer` trait (`discover.rs:13-17`) with a filesystem-based implementation `FsSkillDiscoverer`.

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

### Entry Point Files

- **SKILL.md**: The primary entry point - a markdown file with YAML frontmatter
- **Plugin formats**: `.claude-plugin/plugin.json` for Claude Code plugins (`formats.rs:26`)

### How Skills Are Registered

1. Skills are discovered at startup via `FsSkillDiscoverer.discover()`
2. For registry skills, entries are filtered by enabled/trusted state in `skills-manifest.json`
3. The registry maintains a HashMap of skill name → metadata for quick lookup (`registry.rs:28-29`)

---

## Part 3: Extension Structure & Skeleton

### Directory Structure

```
<skill-directory>/
└── SKILL.md          # Required: YAML frontmatter + markdown instructions
```

**Multi-skill repos**: Skills can be nested at any level (recursive scan) (`install.rs:282-300`)

### Required Files

| File | Required | Description |
|------|----------|-------------|
| `SKILL.md` | Yes | Contains YAML frontmatter + markdown body |

### Optional Files

| File | Description |
|------|-------------|
| `_meta.json` | OpenClaw metadata sibling file (`parse.rs:196-222`) |

---

## Part 4: Manifest & Metadata

### SKILL.md Format

**Location**: `moltis/crates/skills/src/parse.rs`

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

### Required Fields

- `name`: Skill identifier (lowercase alphanumeric, hyphens, 1-64 chars) (`parse.rs:11-23`)

### Optional Fields

- `description`, `homepage`, `license`, `compatibility`, `allowed_tools`, `dockerfile`, `requires`

### Name Validation

From `parse.rs:11-23`:
```rust
pub fn validate_name(name: &str) -> bool {
    !name.is_empty()
        && name.len() <= 64
        && name.chars().all(|c| c.is_ascii_lowercase() || c.is_ascii_digit() || c == '-' || c == ':')
        && !name.starts_with('-')
        && !name.ends_with('-')
        && !name.starts_with(':')
        && !name.ends_with(':')
        && !name.contains("--")
        && !name.contains("::")
}
```

Valid examples: `my-skill`, `a`, `skill123`, `plugin:skill`, `pr-review-toolkit:code-reviewer`

---

## Part 5: API & Interfaces

### Trait: SkillDiscoverer

**File**: `discover.rs:13-17`

```rust
#[async_trait]
pub trait SkillDiscoverer: Send + Sync {
    async fn discover(&self) -> anyhow::Result<Vec<SkillMetadata>>;
}
```

### Trait: SkillRegistry

**File**: `registry.rs:12-25`

```rust
#[async_trait]
pub trait SkillRegistry: Send + Sync {
    async fn list_skills(&self) -> anyhow::Result<Vec<SkillMetadata>>;
    async fn load_skill(&self, name: &str) -> anyhow::Result<SkillContent>;
    async fn install_skill(&self, source: &str) -> anyhow::Result<SkillMetadata>;
    async fn remove_skill(&self, name: &str) -> anyhow::Result<()>;
}
```

### Prompt Generation

**File**: `prompt_gen.rs`

```rust
pub fn generate_skills_prompt(skills: &[SkillMetadata]) -> String {
    // Generates <available_skills> XML block for system prompt injection
}
```

Output format (`prompt_gen.rs:11-31`):
```xml
## Available Skills

<available_skills>
<skill name="commit" source="skill" path="/path/to/SKILL.md">
Create git commits
</skill>
</available_skills>
```

### Requirements Checking

**File**: `requirements.rs:104-146`

```rust
pub fn check_requirements(meta: &SkillMetadata) -> SkillEligibility {
    // Checks bins, any_bins, filters install options by OS
}
```

---

## Part 6: Configuration & Settings

### Skills Manifest

**File**: `manifest.rs`, `types.rs:9-62`

Stored at `~/.moltis/skills-manifest.json`:

```json
{
  "version": 1,
  "repos": [
    {
      "source": "owner/repo",
      "repo_name": "owner-repo",
      "installed_at_ms": 1234567890,
      "commit_sha": "abc123...",
      "format": "skill",
      "skills": [
        {
          "name": "my-skill",
          "relative_path": "owner-repo/skills/my-skill",
          "trusted": true,
          "enabled": true
        }
      ]
    }
  ]
}
```

### Per-Skill Settings

| Field | Type | Description |
|-------|------|-------------|
| `trusted` | bool | User has reviewed and trusts this skill |
| `enabled` | bool | Skill is active for agent use |

**Trust gating rule** (`skills-security.md:14`): "You cannot enable untrusted skills."

---

## Part 7: Lifecycle Management

### Load Order

1. On startup, `FsSkillDiscoverer.discover()` scans all configured paths
2. For Project/Personal sources: scans one level deep for `SKILL.md` files
3. For Registry sources: filters by enabled/trusted state from manifest
4. For Plugin sources: uses plugins manifest for filtering
5. `generate_skills_prompt()` injects available skills into system prompt

### Installation

**File**: `install.rs:18-113`

```rust
pub async fn install_skill(source: &str, install_dir: &Path) -> anyhow::Result<Vec<SkillMetadata>> {
    // 1. Parse source (owner/repo format)
    // 2. Clone via GitHub tarball API
    // 3. Auto-detect format (SKILL.md, Claude Code, etc.)
    // 4. Scan for skills
    // 5. Record in manifest with trusted=false, enabled=false
}
```

### Update/Removal

- **Update**: Re-install replaces the repo, preserves manifest entry
- **Removal**: `install.rs:116-133` - deletes directory and manifest entry

### Migration

**File**: `migration.rs`

Automatically migrates legacy plugins to unified skills system:
- Moves `installed-plugins/` → `installed-skills/`
- Merges `plugins-manifest.json` → `skills-manifest.json`

---

## Part 8: Distribution & Packaging

### Distribution Method

Skills are distributed as **GitHub repositories** containing `SKILL.md` files.

### Installation Command

```bash
moltis skills install <owner/repo>
```

Downloads tarball from `https://api.github.com/repos/{owner}/{repo}/tarball`

### Supported Formats

| Format | Detection | Description |
|--------|-----------|-------------|
| `Skill` | Root or nested `SKILL.md` | Native format |
| `ClaudeCode` | `.claude-plugin/plugin.json` | Claude Code plugins |
| `Codex` | `codex-plugin.json` | Codex plugins |
| `Generic` | Fallback | Treats `.md` files as prompts |

### Claude Marketplace Support

The system supports importing skills from Claude Code marketplace via the `ClaudeCodeAdapter` (`formats.rs:71-264`).

---

## Part 9: Security & Permissions

### Trust Lifecycle

**File**: `skills-security.md:6-14`

```
installed → trusted → enabled
```

States:
- **installed**: Repo is on disk
- **trusted**: User explicitly marked skill as reviewed
- **enabled**: Skill is active for agent use

### Provenance Pinning

- Records `commit_sha` for installed repos (`install.rs:143`)
- UI shows short SHA for provenance review

### Re-Trust on Drift

If local HEAD differs from pinned `commit_sha`:
- Auto-mark `trusted=false`
- Auto-disable all skills in repo
- Re-enable blocked until explicit trust

### Security Guardrails

**File**: `skills-security.md:35-45`

- `confirm=true` required for dependency installs
- Host installs blocked when sandbox mode is off
- Suspicious command chains blocked by default (`curl ... | sh`, base64 decode chains)

### Emergency Kill Switch

`skills.emergency_disable` - immediately disables all third-party skills/plugins

### Security Audit Log

Events logged to `~/.moltis/logs/security-audit.jsonl`:
- Installs, removals, trust changes
- Enable/disable actions
- Dependency install attempts
- Source drift detection

---

## Part 10: Developer Experience

### CLI Tools

| Command | Description |
|---------|-------------|
| `moltis skills install <repo>` | Install a skill repo |
| `moltis skills remove <source>` | Remove an installed repo |
| `moltis skills list` | List available skills |

### Agent Tools

**File**: `skill-tools.md:8-14`

| Tool | Description |
|------|-------------|
| `create_skill` | Write new SKILL.md to `.moltis/skills/<name>/` |
| `update_skill` | Overwrite existing skill's SKILL.md |
| `delete_skill` | Remove a skill directory |

### Skill Watcher

**File**: `watcher.rs`

Monitors skill directories for filesystem changes using debounced notifications (500ms debounce). Emits `skills.changed` events via WebSocket for UI refresh.

---

## Part 11: Code Examples

### Minimal Skill Example

```yaml
---
name: hello-world
description: A simple skill that greets the user
---
# Hello World Skill

You are a friendly assistant. When the user says hello, respond with a warm greeting and ask how you can help.
```

### Working Skill with Full Manifest

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

### Project-Local Skill Creation

Skills created via the `create_skill` tool are stored in `.moltis/skills/<name>/SKILL.md` and automatically discovered on the next message.

---

## Summary

The Moltis skills system provides a comprehensive extension mechanism following the Agent Skills open standard. Key characteristics:

1. **Simple entry point**: `SKILL.md` with YAML frontmatter
2. **Multiple discovery paths**: Project, Personal, Registry, Plugin sources
3. **Trust-based security**: Skills require explicit trust before enabling
4. **Format adapters**: Supports native SKILL.md, Claude Code plugins, Codex plugins
5. **Lifecycle management**: Installation, removal, migration from legacy plugins
6. **Developer tools**: CLI commands, agent tools for runtime skill management, filesystem watcher
7. **Security hardening**: Provenance pinning, drift detection, audit logging, sandbox support
