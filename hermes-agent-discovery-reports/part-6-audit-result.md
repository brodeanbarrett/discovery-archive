# Part 6: Behavioral Evaluation (The "Colleague" Test)

*If the infrastructure is working correctly, the day-to-day interaction should feel fundamentally different.*

- [ ] **Continuity:** Can you reference a project from three days ago without re-explaining the context?
- [ ] **Initiative:** Does the AI surface pending items or suggest next steps without being asked?
- [ ] **Depth:** Do the AI's suggestions reflect a deep understanding of your business/projects rather than generic "best practices"?
- [ ] **Accountability:** Can you ask the AI, "Did you finish that task?" and get a deterministic, verifiable answer?

---

**Audit Scoring:**
If you leave checkboxes empty in **Parts 1, 2, or 5**, you are still *using an AI tool*, not hiring an AI. The foundational difference relies entirely on Identity, Memory, and Safety. Tools and Autonomy (Parts 3 & 4) can be scaled up over time as trust builds.

---

## Scores

Fill in your evaluation scores here.

| Criterion | Score (1-5) | Notes |
|-----------|-------------|-------|
| Continuity | 4/5 | SessionDB + FTS5 + memory_tool + session_search_tool provide full cross-session recall infrastructure |
| Initiative | 4/5 | TodoStore persists across compression; prompt guidance for surfacing pending items and suggesting skills |
| Depth | 4/5 | Memory tool stores project conventions/tool quirks; skill_manage captures discovered approaches |
| Accountability | 4/5 | TodoStore provides deterministic status; SessionDB tracks tool_call_count per session |

**Part 6 Total: 16/20**

## Comments

### Continuity (4/5)
The system has complete infrastructure for cross-session continuity:

1. **SessionDB** (`hermes_state.py` lines 36-104): Stores all sessions with full message history, timestamps, FTS5 full-text search via `messages_fts` virtual table, and `parent_session_id` chains for lineage tracking. Sessions persist at `~/.hermes/state.db`.

2. **session_search tool** (`model_tools.py` line 90, `agent/prompt_builder.py` lines 142-146): Explicit `SESSION_SEARCH_GUIDANCE` instructs the agent: *"When the user references something from a past conversation or you suspect relevant cross-session context exists, use session_search to recall it before asking them to repeat themselves."*

3. **memory tool** (`memory_tool.py` lines 89-548, `agent/prompt_builder.py` lines 128-140): Persistent curated memory in `~/.hermes/memories/MEMORY.md` and `USER.md` injected into every session's system prompt snapshot. Stores environment facts, project conventions, tool quirks, and user preferences — exactly the durable facts needed to avoid re-explaining.

The infrastructure is strong. Continuity ultimately depends on the agent leveraging these tools at runtime; the system provides clear behavioral guidance but cannot force the model to call session_search when the user makes an implicit reference.

### Initiative (4/5)
The system actively prompts initiative through multiple mechanisms:

1. **TodoStore** (`tools/todo_tool.py` lines 25-269): In-memory task list persists across context compression via `format_for_injection()` (lines 90-137), ensuring todos survive compression events. The tool schema encourages the agent to decompose complex tasks and track progress.

2. **SKILLS_GUIDANCE** (`agent/prompt_builder.py` lines 148-155): Explicitly instructs: *"After completing a complex task (5+ tool calls), fixing a tricky error, or discovering a non-trivial workflow, save the approach as a skill with skill_manage... When using a skill and finding it outdated, incomplete, or wrong, patch it immediately with skill_manage(action='patch') — don't wait to be asked."* This directly enables surfacing discovered approaches as next steps.

3. **Session lineage** (`hermes_state.py` lines 490-532): `get_next_title_in_lineage()` enables multi-session task continuation with numbered variants ("my task #2", "my task #3"), helping the agent recognize when work spans sessions.

The infrastructure actively encourages initiative. Actual initiative depends on model behavior; the guidance is present but not enforced.

### Depth (4/5)
The system supports domain-specific depth through curated memory and skill capture:

1. **memory_tool.py** design: Stores "environment facts, project conventions, tool quirks, and things learned" (`memory_tool.py` lines 6-7). The guidance explicitly prioritizes *"what reduces future user steering"* — facts that prevent repeating corrections — over procedural task details (`agent/prompt_builder.py` lines 133-135).

2. **skill_manage** (`agent/prompt_builder.py` lines 151-154): Non-trivial discoveries (workflows, tricky errors) are captured as reusable skills. The agent is instructed to maintain them proactively.

3. **Threat-scanned memory** (`memory_tool.py` lines 49-86): Content injection detection ensures memory quality — only legitimate domain knowledge enters the prompt.

The infrastructure distinguishes between generic advice and project-specific knowledge. Deep suggestions depend on what was previously captured; the system provides the right mechanisms but the value compounds over time as memory accumulates.

### Accountability (4/5)
The system provides verifiable task completion tracking:

1. **TodoStore** (`tools/todo_tool.py` lines 25-269): Returns full current list on every call with deterministic status (`pending`, `in_progress`, `completed`, `cancelled`). "Did you finish that task?" maps directly to checking todo status.

2. **SessionDB instrumentation** (`hermes_state.py` lines 270-329): Tracks `tool_call_count`, `message_count`, `input_tokens`, `output_tokens` per session. `end_session()` records `end_reason` for full audit trail.

3. **Compression survival** (`todo_tool.py` lines 90-137): Todo list survives context compression events via `format_for_injection()`, ensuring todos remain accurate even in long sessions.

The only gap is that TodoStore is in-memory per session — if the process restarts mid-session, active todos are lost. For full accountability across process restarts, the todo state would need disk persistence (similar to SessionDB). The infrastructure for verification is present; the persistence boundary is the limiting factor.
