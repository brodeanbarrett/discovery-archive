# Part 2: The Three-Layer Memory Architecture (Continuity & Depth)

*A hired AI doesn't start from zero every day. It relies on a specific, three-layered memory system to accumulate context, learn preferences, and retrieve facts automatically.*

## Layer 1: Tacit Knowledge (`MEMORY.md`)

*This layer captures how you operate—your patterns, preferences, and lessons learned (facts about you, not the world).*

- [x] **File Exists:** `MEMORY.md` is present in the AI's workspace and loaded on boot.
- [x] **Working Style Defined:** Captures your schedule, availability, and decision-making preferences (e.g., what "handle it" means to you).
- [x] **Communication Preferences Set:** Specifies how you prefer to receive information (e.g., short texts vs. long emails, when to interrupt vs. batch updates).
- [x] **Project Patterns Documented:** Lists naming conventions, ongoing priorities, and regular collaborators.
- [x] **Annoyances & Trust Levels Explicit:** Clearly states output styles you dislike, what the AI can do autonomously, and what requires approval.
- [ ] **Pattern Recognition:** The AI updates this file automatically when it notices new patterns in how you work.

## Layer 2: Daily Notes (`memory/YYYY-MM-DD.md`)

*This layer serves as a chronological log of what happened each day (the "when did we discuss X?" layer).*

- [ ] **Automated Generation:** A new markdown file is generated for each working day.
- [ ] **Key Events Tracked:** Logs discussions, system launches (e.g., starting a coding agent), and important incoming communications.
- [ ] **Decisions Logged:** Records what was decided, what technology/approach was chosen, and the reasoning behind it.
- [ ] **Facts Extracted:** Notes new roles, company associations, or deadline shifts (and links them to the Knowledge Graph).
- [ ] **Active Processes Monitored:** Tracks active long-running processes (like Ralph loops in tmux sessions), including their start times and current status.

## Layer 3: Knowledge Graph (`~/life/`)

*This is the deep storage layer, organized by entity rather than chronology.*

- [ ] **PARA Structure Implemented:** Folders are organized by Projects, Areas (people/companies), Resources, and Archives.
- [ ] **Entity Summaries (`summary.md`):** Every entity folder has a summary file for quick, high-level context that the AI loads first for fast recall.
- [ ] **Atomic Facts (`items.json`):** Every entity folder contains a JSON file storing self-contained facts.
- [ ] **Metadata Tracking:** Facts in the JSON include explicit metadata: `id`, `category`, `timestamp`, `source`, `status`, `relatedEntities`, and `accessCount`.
- [ ] **No Deletion Rule:** The system never deletes facts. Instead, outdated facts are marked with `status: "superseded"` and linked to the new fact via a `supersededBy` tag.

## Memory Operations & Maintenance (The "Heartbeat")

*Data is useless if it becomes a write-only graveyard. The system must actively maintain itself.*

- [ ] **Nightly Extraction Cron Job:** An automated job runs daily (e.g., at 11:00 PM) to review conversations, extract durable facts, and skip small talk.
- [ ] **Memory Decay System:** The system uses `accessCount` and `lastAccessed` metadata to prioritize "Hot facts" (last 7 days) in summaries, while letting "Cold facts" (30+ days) drop out of summaries (but remain in JSON storage).
- [ ] **Summary Rewrites:** `summary.md` files are actively rewritten/updated weekly based on fact temperatures.
- [ ] **Semantic Auto-Retrieval:** A vector-search backend (like QMD) runs continuously (e.g., re-indexing every 5 minutes) across all three layers so the AI automatically recalls relevant context without manual prompting.

## Scores

| Criterion | Score (1-5) | Notes |
|-----------|-------------|-------|
| Layer 1: MEMORY.md exists | 5/5 | `tools/memory_tool.py` lines 39, 112 - MEMORY.md loaded from `~/.hermes/memories/` |
| Layer 1: Working style defined | 4/5 | USER.md captures preferences, communication style, workflow habits (`memory_tool.py` lines 8-9, 84-93) |
| Layer 1: Communication preferences | 4/5 | USER.md includes communication preferences section (`memory.md` line 89) |
| Layer 1: Project patterns | 4/5 | MEMORY.md captures environment facts, conventions, tool quirks (`memory_tool.py` lines 76-82) |
| Layer 1: Annoyances & trust levels | 3/5 | USER.md has "pet peeves" section but no explicit trust/autonomy levels (`memory.md` line 90) |
| Layer 1: Pattern recognition | 1/5 | No automatic pattern recognition - agent must manually update memory via `memory` tool |
| Layer 2: Automated generation | 1/5 | No daily note generation system implemented |
| Layer 2: Key events tracked | 1/5 | Sessions stored in SQLite but no chronological daily log format |
| Layer 2: Decisions logged | 1/5 | No decision logging infrastructure |
| Layer 2: Facts extracted | 1/5 | No fact extraction from sessions to knowledge graph |
| Layer 2: Active processes monitored | 1/5 | No process tracking infrastructure |
| Layer 3: PARA structure | 1/5 | `~/life/` folder structure not implemented - uses flat `~/.hermes/memories/` |
| Layer 3: Entity summaries | 1/5 | No `summary.md` per entity folder pattern |
| Layer 3: Atomic facts | 1/5 | No `items.json` per entity structure |
| Layer 3: Metadata tracking | 1/5 | No metadata fields (id, category, timestamp, source, status, relatedEntities, accessCount) |
| Layer 3: No deletion rule | 2/5 | `memory_tool.py` supports `remove` action - deletion is allowed (line 288-300) |
| Nightly extraction cron | 1/5 | Cron system exists (`cron/jobs.py`) but no nightly memory extraction job defined |
| Memory decay system | 1/5 | No accessCount or lastAccessed metadata in memory entries |
| Summary rewrites | 1/5 | No automatic summary rewrite system |
| Semantic auto-retrieval | 2/5 | QMD skill exists (`optional-skills/research/qmd/SKILL.md`) but is optional/manual, not auto-retrieval |

**Part 2 Total: 34/100**

## Comments

### Layer 1 Assessment (18/30)
The bounded curated memory system (`tools/memory_tool.py`) is well-implemented with:
- Two stores: MEMORY.md (agent notes) and USER.md (user profile)
- Character limits (2200/1375) for bounded prompts
- Security scanning for injection patterns
- Frozen snapshot pattern for system prompt stability
- Session search via FTS5 (`tools/session_search_tool.py`)

**Gap**: No automatic pattern recognition when agent observes user behavior patterns.

### Layer 2 Assessment (5/25)
**COMPLETELY ABSENT** - No daily notes infrastructure:
- No `memory/YYYY-MM-DD.md` format
- No automatic daily file generation
- No chronological event logging
- Sessions stored but not parsed into daily summaries

### Layer 3 Assessment (7/25)
**COMPLETELY ABSENT** - No Knowledge Graph structure:
- No `~/life/` PARA folder organization
- No entity-based folder structure
- No `summary.md` per entity
- No `items.json` atomic facts storage
- No metadata tracking system
- No no-deletion rule (removal is allowed)

### Memory Operations Assessment (5/20)
**MOSTLY ABSENT**:
- Cron scheduler exists (`cron/jobs.py`, `cron/scheduler.py`) but no nightly extraction job
- No memory decay or temperature system
- No automatic summary rewrites
- QMD vector search exists as optional skill but not integrated as core auto-retrieval

### Notable Equivalencies
- Session storage with FTS5 (`hermes_state.py`) provides some daily session recall but not daily note generation
- QMD optional skill provides vector search capability but requires manual setup and invocation
- Character-limited bounded memory provides some curation but lacks the 3-layer architecture described in the spec
