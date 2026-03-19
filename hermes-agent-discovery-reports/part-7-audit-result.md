# Part 7: Extensibility & Customization (How far can you customize the agent?)

*This category evaluates how infinitely extensible the agent is—allowing deep customization of prompts, behaviors, tools, and integrations.*

---

## System Prompts (Base Layer)

*The foundational instruction layer that defines the agent's core identity and behavior.*

- [ ] **System Prompt File Exists:** There is a dedicated system prompt file that can be edited/customized
- [ ] **System Prompt Loading:** The system prompt is loaded from a configurable file path (not hardcoded)
- [ ] **System Prompt Templates:** There are template/system prompt examples users can start from
- [ ] **System Prompt Variables:** The system prompt supports variable interpolation (e.g., {{user_name}}, {{date}})
- [ ] **Multi-System Prompt Support:** Multiple system prompts can be defined and switched between

## Agent Prompts (Session Layer)

*Prompts that define specific agent behaviors in different contexts.*

- [ ] **Agent Prompt Definitions:** Users can define custom agent prompts/profiles
- [ ] **Agent Presets:** There are pre-built agent presets (e.g., "Code Agent", "Research Agent")
- [ ] **Agent Prompt Inheritance:** Agent prompts can inherit/extend from base prompts
- [ ] **Dynamic Agent Switching:** The agent can switch between defined profiles during runtime

## Skills (Bundled Capabilities)

*Skills are bundled sets of tools, prompts, and behaviors for specific tasks.*

- [ ] **Skill System Exists:** There is a formal "skill" concept/system in the agent
- [ ] **Skill Definition Format:** Skills are defined in a structured format (e.g., YAML, JSON, Markdown)
- [ ] **Skill Structure:** Skills contain tools, prompts, and/or memory configurations
- [ ] **Skill Loading:** Skills can be loaded from external files/directories
- [ ] **Skill Marketplace:** There is a registry or marketplace for discovering/ installing skills
- [ ] **Custom Skill Creation:** Users can create their own skills from scratch
- [ ] **Skill Dependencies:** Skills can declare dependencies on other skills or tools

## Commands (User-Invokable)

*Commands are user-facing invocations that trigger specific behaviors.*

- [ ] **Command Registry:** There is a registry of available commands
- [ ] **Custom Command Definition:** Users can define custom commands
- [ ] **Command Aliases:** Commands can have aliases (shortcuts)
- [ ] **Command Arguments:** Commands support arguments/parameters
- [ ] **Command Categories:** Commands are organized into categories

## Tools (Executable Actions)

*Tools are executable functions the agent can call.*

- [ ] **Tool Registry:** There is a registry of available tools
- [ ] **Tool Definition API:** Tools can be defined programmatically (not just built-in)
- [ ] **Tool Schema:** Tools have typed input/output schemas
- [ ] **Tool Sandboxing:** Tools run in a sandboxed environment for security
- [ ] **Tool Approval:** Tools require human approval before execution (configurable)
- [ ] **Tool Categories:** Tools are organized into categories (e.g., file, web, system)
- [ ] **Tool Documentation:** Each tool has documentation/description

## Hooks (Lifecycle Events)

*Hooks allow custom code to run at specific lifecycle events.*

- [ ] **Hook System Exists:** There is a formal hook/event system
- [ ] **Pre-Request Hooks:** Code can run before agent requests
- [ ] **Post-Response Hooks:** Code can run after agent responses
- [ ] **Tool Execution Hooks:** Code can run before/after tool execution
- [ ] **Error Hooks:** Code can handle/catch errors
- [ ] **Custom Hooks:** Users can define custom hooks
- [ ] **Hook Scripting:** Hooks can be written in common languages (JS, Python, etc.)

## MCP (Model Context Protocol)

*MCP is a protocol for connecting the agent to external resources and tools.*

- [ ] **MCP Client:** The agent can act as an MCP client
- [ ] **MCP Server:** The agent can run as an MCP server
- [ ] **MCP Resource Access:** MCP resources can be accessed by the agent
- [ ] **MCP Tool Integration:** MCP tools can be invoked by the agent
- [ ] **MCP Prompts:** MCP prompt templates can be used
- [ ] **MCP Configuration:** MCP connections are configurable

## Plugins (Extensions)

*Plugins extend the core functionality of the agent.*

- [ ] **Plugin System Exists:** There is a formal plugin architecture
- [ ] **Plugin API:** There is a documented API for building plugins
- [ ] **Plugin Loading:** Plugins can be loaded from external files
- [ ] **Plugin Lifecycle:** Plugins have init/enable/disable/unload lifecycle
- [ ] **Plugin Configuration:** Plugins can be configured via files or runtime
- [ ] **Plugin Dependencies:** Plugins can depend on other plugins
- [ ] **Plugin Sandboxing:** Plugins run in isolation

## Extensions (Third-Party)

*Extensions are third-party integrations or add-ons.*

- [ ] **Extension Registry:** There is a registry of available extensions
- [ ] **Extension Installation:** Extensions can be installed (e.g., npm install, pip install)
- [ ] **Extension Updates:** Extensions can be updated
- [ ] **Extension Configuration:** Extensions are configurable
- [ ] **Extension Uninstallation:** Extensions can be removed

## Agent Configuration (Settings)

*General configuration options for agent behavior.*

- [ ] **Config File:** Agent configuration is stored in a config file (YAML, JSON, TOML)
- [ ] **Environment Variables:** Configuration can be overridden via environment variables
- [ ] **Runtime Config Updates:** Configuration can be changed at runtime without restart
- [ ] **Config Validation:** Configuration is validated on load
- [ ] **Config Documentation:** All config options are documented
- [ ] **Config Profiles:** Different configuration profiles can be saved and switched
- [ ] **Model Configuration:** Can configure which LLM models to use
- [ ] **Temperature/Tuning:** Can adjust model parameters (temperature, top_p, etc.)
- [ ] **Context Window:** Can configure context window size
- [ ] **Rate Limiting:** Can configure rate limits for API calls

---

## Scores

Fill in your evaluation scores here.

| Criterion | Score (1-5) | Notes |
|-----------|-------------|-------|
| System Prompt File Exists | 5/5 | SOUL.md in HERMES_HOME via load_soul_md() (agent/prompt_builder.py:438-458) |
| System Prompt Loading | 5/5 | load_soul_md() loads from configurable HERMES_HOME path (agent/prompt_builder.py:438-458) |
| System Prompt Templates | 5/5 | Layered templates: SOUL.md, AGENTS.md, .cursorrules, .cursor/rules/*.mdc (agent/prompt_builder.py:459-535) |
| System Prompt Variables | 5/5 | Dynamic variables: session_id, model, provider, timestamp injected (run_agent.py:2098-2108) |
| Multi-System Prompt Support | 5/5 | SOUL.md + personality overlays + context files via build_context_files_prompt() (agent/prompt_builder.py:459-535) |
| Agent Prompt Definitions | 5/5 | Personalities defined in cli-config.yaml.example:406-448, loaded via cli.py:1127-1131 |
| Agent Presets | 5/5 | Built-in presets: helpful, concise, technical, creative, kawaii, catgirl, pirate (cli-config.yaml.example:406-448) |
| Agent Prompt Inheritance | 5/5 | SOUL.md base + /personality overlay pattern; context files layered (agent/prompt_builder.py) |
| Dynamic Agent Switching | 5/5 | /personality command via _handle_personality_command() (cli.py:2991-3020) |
| Skill System Exists | 5/5 | Formal skill system in tools/skills_tool.py with SKILL.md format |
| Skill Definition Format | 5/5 | YAML frontmatter, agentskills.io compatible (tools/skills_tool.py:1-50) |
| Skill Structure | 5/5 | SKILL.md + subdirectories pattern with metadata.hermes.related_skills (skills/software-development/plan/SKILL.md) |
| Skill Loading | 5/5 | skills_list and skill_view functions load from skills/index-cache (tools/skills_tool.py) |
| Skill Marketplace | 5/5 | GitHub, ClawHub, well-known endpoints, skills.sh marketplace integration (tools/skills_hub.py, hermes_cli/skills_hub.py) |
| Custom Skill Creation | 5/5 | Agent-managed skill creation via skill_manage tool (tools/skill_manager_tool.py) |
| Skill Dependencies | 5/5 | metadata.hermes.related_skills for inter-skill dependencies (tools/skills_tool.py) |
| Command Registry | 5/5 | hermes_cli/commands.py CommandDef dataclass with COMMAND_REGISTRY (hermes_cli/commands.py:28-45) |
| Custom Command Definition | 5/5 | CommandDef entries with full metadata (aliases, args_hint, subcommands) |
| Command Aliases | 5/5 | aliases tuple in CommandDef supports shortcuts (hermes_cli/commands.py:28-45) |
| Command Arguments | 5/5 | args_hint and subcommands parameters in CommandDef |
| Command Categories | 5/5 | category field: Session, Configuration, Tools & Skills, Info, Exit |
| Tool Registry | 5/5 | ToolRegistry in tools/registry.py with register(), get_definitions(), dispatch() |
| Tool Definition API | 5/5 | register() API with name, toolset, schema, handler, check_fn, requires_env |
| Tool Schema | 5/5 | OpenAI-format tool schemas via get_definitions() (tools/registry.py) |
| Tool Sandboxing | 4/5 | ToolContext sandboxing for RL training (environments/tool_context.py); Docker sandboxes for benchmarks; general tools lack explicit sandboxing |
| Tool Approval | 5/5 | Dangerous command approval via security.md patterns, ACP permissions adapter (acp_adapter/permissions.py) |
| Tool Categories | 5/5 | TOOLSETS dict with web, terminal, browser, file, vision categories (toolsets.py:50-120) |
| Tool Documentation | 5/5 | Full documentation in website/docs/user-guide/features/tools.md |
| Hook System Exists | 5/5 | HookRegistry in gateway/hooks.py with HOOK.yaml + handler.py pattern (gateway/hooks.py:36-60) |
| Pre-Request Hooks | 4/5 | session:start, agent:start events; gateway-level pre-request hooks present (gateway/hooks.py:10-19) |
| Post-Response Hooks | 4/5 | session:end, agent:end events; agent:step per-turn hooks (gateway/hooks.py:14-16) |
| Tool Execution Hooks | 5/5 | pre_tool_call and post_tool_call hooks in plugin system (hermes_cli/plugins.py:40-70) |
| Error Hooks | 3/5 | Errors caught and logged silently in gateway hooks (gateway/hooks.py:19); no explicit error hook type |
| Custom Hooks | 5/5 | HOOK.yaml + handler.py pattern allows custom hooks (gateway/hooks.py:1-19) |
| Hook Scripting | 3/5 | Python-only handler.py; no multi-language scripting (gateway/hooks.py:7) |
| MCP Client | 5/5 | Full MCP client with stdio/HTTP transports (tools/mcp_tool.py:70-90) |
| MCP Server | 5/5 | MCPServerTask with stdio and HTTP transport modes (tools/mcp_tool.py:680-760) |
| MCP Resource Access | 5/5 | list_resources and read_resource utility tools (tools/mcp_tool.py:1180-1380) |
| MCP Tool Integration | 5/5 | MCP server tools registered as local tools via _discover_and_register_server (tools/mcp_tool.py:1180-1230) |
| MCP Prompts | 5/5 | list_prompts and get_prompt utility tools (tools/mcp_tool.py:1350-1380) |
| MCP Configuration | 5/5 | mcp_servers in config.yaml, _load_mcp_config() (tools/mcp_tool.py:1600-1620) |
| Plugin System Exists | 5/5 | PluginManager in hermes_cli/plugins.py (1-450) |
| Plugin API | 5/5 | PluginContext with register_tool(), register_hook() (hermes_cli/plugins.py:225-290) |
| Plugin Loading | 5/5 | Three sources: ~/.hermes/plugins/, ./.hermes/plugins/, pip entry-points (hermes_cli/plugins.py:160-220) |
| Plugin Lifecycle | 5/5 | VALID_HOOKS: pre/post_tool_call, pre/post_llm_call, on_session_start/end (hermes_cli/plugins.py:40-70) |
| Plugin Configuration | 5/5 | plugin.yaml manifest with requires_env dependency gating (hermes_cli/plugins.py:340-400) |
| Plugin Dependencies | 5/5 | requires_env dependency gating in manifest (hermes_cli/plugins.py:340-400) |
| Plugin Sandboxing | 3/5 | Python import isolation and try/except wrappers; no true process-level isolation |
| Extension Registry | 2/5 | No separate extension registry; plugins handle third-party; pip packages serve as extensions |
| Extension Installation | 2/5 | pip install used but no hermes-specific extension install command |
| Extension Updates | 2/5 | pip update mechanism available but no hermes-specific update command |
| Extension Configuration | 2/5 | Depends on individual pip package; no unified hermes extension config |
| Extension Uninstallation | 2/5 | pip uninstall available; no hermes-specific uninstall command |
| Config File | 5/5 | config.yaml in HERMES_HOME, load_config/save_config in hermes_cli/config.py |
| Environment Variables | 5/5 | OPTIONAL_ENV_VARS and REQUIRED_ENV_VARS with get_env_value/save_env_value (hermes_cli/config.py:399-499) |
| Runtime Config Updates | 5/5 | _set_nested() for dotted key paths + save_config() (hermes_cli/config.py:889-901, 1287) |
| Config Validation | 5/5 | validate_requested_model(), migration system, get_missing_config_fields() (hermes_cli/config.py:1055-1120, hermes_cli/models.py:1042-1105) |
| Config Documentation | 5/5 | website/docs/user-guide/configuration.md, website/docs/reference/environment-variables.md |
| Config Profiles | 5/5 | preset selection in cli.py; migration presets: basic/full/skills (hermes_cli/config.py:1055-1068) |
| Model Configuration | 5/5 | DEFAULT_CONFIG['model'], auxiliary model config, _get_model_config() (runtime_provider.py:48-60) |
| Temperature/Tuning | 5/5 | Explicit temperature kwarg in auxiliary_client.py (agent/auxiliary_client.py:163-170) |
| Context Window | 5/5 | DEFAULT_CONTEXT_LENGTHS dict, get_model_context_length() (agent/model_metadata.py:35-90) |
| Rate Limiting | 5/5 | _rate_limits.json in gateway/pairing.py, _check_rate_limit() in mcp_tool (gateway/pairing.py:225-245, tools/mcp_tool.py:347-365) |

---

**Part 7 Total: 291/305**

## Comments

Part 7 demonstrates exceptional extensibility across most dimensions. Key highlights:

**Strengths:**
- **System Prompts (5/5):** Layered multi-file system with SOUL.md, context files, personality overlays, and dynamic variable injection is industry-leading
- **Skills (5/7):** Comprehensive marketplace with GitHub/ClawHub/skills.sh integration, agent-managed skill creation, and dependencies
- **MCP (6/6):** Full Model Context Protocol implementation with both client and server modes, resources, tools, and prompts
- **Plugins (6/7):** Three-source discovery, lifecycle hooks, tool registration API, and dependency gating
- **Agent Configuration (10/10):** Complete configuration system with runtime updates, validation, profiles, and per-model tuning

**Weaknesses:**
- **Tool Sandboxing (4/5):** Sandboxing exists for benchmarks/RL environments but not enforced for general user tools
- **Hook Scripting (3/5):** Python-only; no multi-language scripting support
- **Error Hooks (3/5):** Errors caught but no explicit error hook type for custom handling
- **Plugin Sandboxing (3/5):** Python import isolation but no process-level isolation
- **Extensions (2/5):** No separate extension system; uses generic pip packages instead of a first-class hermes extension architecture

**Notable Equivalencies:**
- MCP client = hermes-agent as MCP client to external servers
- MCP server = MCPServerTask allows hermes to act as MCP server
- Extensions are handled via the plugin system rather than a separate mechanism

The agent excels at extensibility through its plugin system, MCP integration, and skill marketplace. The main gap is the absence of a formal first-class extension system separate from plugins, and Python-only hook scripting limits flexibility for non-Python developers.
