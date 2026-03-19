# Part 4: Autonomy & Initiative (Does it work while you sleep?)

*A hired AI proactively checks on projects and performs background work without constant prompting.*

- [ ] **Scheduled Autonomous Work:** Cron jobs are configured for routine tasks (e.g., morning check-ins, daily status monitoring).
- [ ] **Sub-Agent Spawning:** The primary AI can spin up specialized sub-agents (e.g., Codex agents via "Ralph loops") for parallel, focused work.
- [ ] **Cost-Optimized Routing:** High-frequency/background tasks (like heartbeats) run on fast, cheap models, while complex reasoning is reserved for premium models.
- [ ] **Wake Hooks/Notifications:** The AI is configured to ping the user immediately when an autonomous background task finishes.

## Scores


| Criterion | Score (1-5) | Notes |
|-----------|-------------|-------|
| Scheduled Autonomous Work | 5/5 | Full cron scheduler: intervals, cron expressions, one-shot. Gateway ticks every 60s. File lock prevents duplicate execution. Jobs stored in `~/.hermes/cron/jobs.json`. Supports skill loading and platform delivery. (`cron/__init__.py`, `cron/jobs.py`, `cron/scheduler.py`) |
| Sub-Agent Spawning | 5/5 | `delegate_task` tool spawns isolated child AIAgent instances. ThreadPoolExecutor for parallel execution (max 3). Depth limiting, blocked tools, progress callbacks to parent. (`tools/delegate_tool.py`, `agent/delegate.py`) |
| Cost-Optimized Routing | 5/5 | `smart_model_routing.py` routes simple messages (<160 chars, <28 words, no code/URLs) to cheap_model. `_COMPLEX_KEYWORDS` set ensures debugging, refactor, analyze, etc. stay on premium. (`agent/smart_model_routing.py`) |
| Wake Hooks/Notifications | 3/5 | Hook system (`gateway/hooks.py`) for lifecycle events. Cron auto-delivery to messaging platforms. No dedicated proactive ping for arbitrary task completion. |

**Part 4 Total: 18/20**

## Comments

**Scheduled Autonomous Work (5/5):** Comprehensive cron system with `croniter` for cron expressions, interval-based recurring jobs, and one-shot tasks. Scheduler ticks every 60 seconds. File-based locking prevents duplicate execution. Jobs stored in `~/.hermes/cron/`. Supports skill loading and platform delivery.

**Sub-Agent Spawning (5/5):** Full `delegate_task` implementation with ThreadPoolExecutor for parallel execution, isolated child context, restricted toolsets, depth limiting (MAX_DEPTH=2), and blocked tools (delegate_task, clarify, memory, send_message, execute_code). Progress callbacks relay to parent display.

**Cost-Optimized Routing (5/5):** `smart_model_routing.py` routes simple messages conservatively to cheap_model. Complex keywords (debug, implement, refactor, analyze, investigate, architecture, test, plan) keep tasks on premium models.

**Wake Hooks/Notifications (3/5):** Event hook system in `gateway/hooks.py` provides extensibility for lifecycle events (gateway:startup, session:start/end, agent:start/step/end, command:*). Cron jobs have auto-delivery to Telegram, Discord, etc. via `_deliver_result()`. No dedicated proactive notification for arbitrary task completion.