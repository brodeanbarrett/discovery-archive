# Part 3: Tools & Capabilities (Can it act?)

*A hired AI can operate in your environment, not just talk about it.*

- [ ] **Messaging Integration:** The AI lives in a native communication channel (e.g., Telegram, Slack) rather than requiring a separate browser tab.
- [ ] **File System Access:** The AI can read and write files to maintain its own memory and manage projects.
- [ ] **Web Access:** The AI can search the web and read documentation to stay current.
- [ ] **Command Execution:** The AI can run shell commands/scripts on the machine.
- [ ] **Minimum Authority Principle Applied:** The AI is only granted access to the specific tools and data it needs for its current role, starting with read-only access.

## Scores

| Criterion | Score (1-5) | Notes |
|-----------|-------------|-------|
| Messaging Integration | 5/5 | 11 native platform adapters: Telegram, Discord, Slack, Signal, Email, SMS, WhatsApp, Matrix, Mattermost, HomeAssistant, DingTalk. Each adapter handles receive/send natively (no browser tab). |
| File System Access | 4/5 | Full read via `file_tools.py`, write via `file_operations.py`. Path restrictions, loop detection, hidden/skills cache blocking. Memory tool for persistent storage. Minor deduction: write operations less guarded than reads. |
| Web Access | 5/5 | `web_tools.py` with Firecrawl/Tavily/Parallel backends. `web_search_tool`, `web_extract_tool` with LLM summarization. `website_policy.py` enforces access control. `web_research_env.py` for RL training benchmarks. |
| Command Execution | 5/5 | `terminal_tool.py` with 5 execution environments (local, Docker, Modal, Singularity, Daytona). `approval.py` detects 24+ dangerous command patterns with interactive approval. `tirith_security.py` pre-exec scanning. |
| Minimum Authority Principle | 4/5 | `tools_config.py` enables/disables tools per-platform. Dangerous commands require approval. Secrets blocklisted from subprocess env. Minor deduction: no per-task scoping; default is full read/write rather than least-privilege. |

**Part 3 Total: 23/25**

## Comments

**Messaging Integration (5/5):** Hermes implements an extensive platform adapter system with 11 native communication channels (`gateway/platforms/`). Each adapter handles both inbound message reception and outbound responses natively — no browser tab required. Notable: Telegram supports 4096-char messages with MarkdownV2; Slack supports 39000-char messages with mrkdwn conversion; Signal supports 8000 chars with E2EE; WhatsApp supports 65536 chars via HTTP bridge.

**File System Access (4/5):** The `file_tools.py` module provides read with pagination (max 2000 lines), binary detection, and image handling. The `file_operations.py` module (via `ShellFileOperations`) provides write capabilities. Security measures include: path expansion blocking for hidden dirs/skills cache, loop detection after 3 identical reads, and binary file detection. Deduction: write operations rely on shell environment rather than direct Python I/O, making them less granularly controllable than reads.

**Web Access (5/5):** The `web_tools.py` module provides three-tier web access: (1) `web_search_tool` uses Firecrawl, Tavily, or Parallel backends; (2) `web_extract_tool` fetches URLs with LLM summarization via OpenRouter/Gemini; (3) `web_crawl_tool` uses Firecrawl for site crawling. The `website_policy.py` module enforces per-domain access control before every fetch. The `web_research_env.py` provides a full RL training environment with FRAMES benchmark.

**Command Execution (5/5):** The `terminal_tool.py` supports 5 execution environments: local (default), Docker containers, Modal cloud sandboxes, Singularity containers, and Daytona. The `approval.py` module maintains 24+ dangerous command patterns (recursive rm, chmod 777, mkfs, fork bombs, pipe-to-shell, SQL DROP, etc.) with interactive approval prompts and smart auto-approval for low-risk commands. The `tirith_security.py` module provides pre-execution content scanning for homograph URLs, pipe-to-interpreter, and terminal injection attacks.

**Minimum Authority Principle (4/5):** The `hermes_cli/tools_config.py` module provides granular tool enable/disable per platform with per-tool API key configuration. The `approval.py` dangerous command system requires explicit approval for high-risk operations. The `local.py` environment blocks Hermes-internal secrets (API keys, provider tokens) from subprocesses. Minor deduction: tools are not scoped to specific tasks/purposes; default state is full capability rather than least-privilege per role.
