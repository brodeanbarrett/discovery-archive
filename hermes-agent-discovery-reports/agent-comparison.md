# AI Agent Framework Comparison: Hermes vs Moltis vs ZeroClaw

**Audit Framework:** "Hired AI" Readiness Checklist  
**Audit Date:** 2026-03-19  
**Generated from:** discovery-archive audits

---

## Executive Summary

| Agent | Overall Score | Verdict |
|-------|--------------|---------|
| **Hermes Agent** | 417/525 (79.4%) | Senior engineer with strong tools, limited institutional memory |
| **ZeroClaw** | 89/146 (61.0%) | Well-engineered, strong security, missing memory layer 3 & extensibility |
| **Moltis** | 102/220 (46.4%) | True autonomous framework, significant gaps in memory & safety |

---

## Score Comparison by Category

| Category | Max | Hermes | ZeroClaw | Moltis |
|----------|-----|--------|----------|--------|
| **Part 1: Identity** | 20 | 17 (85%) | 4/4 EQ (100%) | 16 (80%) |
| **Part 2: Memory** | 100 | 34 (34%) | 10/21 (48%) | 18 (18%) |
| **Part 3: Tools** | 25 | 23 (92%) | 5/5 (100%) | 25 (100%) |
| **Part 4: Autonomy** | 20 | 18 (90%) | 4/4 (100%) | 17 (85%) |
| **Part 5: Safety** | 35 | 18 (51%) | 15/25 (60%) | 13 (37%) |
RZ:| **Part 6: Behavioral** | 20 | 16 (80%) | 19/20 (95%) | 13 (65%) |
| **Part 7: Extensibility** | 305 | 291 (95%) | 31/63 (49%) | 153/230 (67%) |
| **TOTAL** | 525 | **417 (79%)** | **89 (61%)** | **102 (46%)** |

---

## Radar Chart Data

```
Category        | Hermes | ZeroClaw | Moltis
----------------|--------|----------|-------
Identity        |  85%   |  100%    |  80%
Memory          |  34%   |   48%    |  18%
Tools           |  92%   |  100%    | 100%
Autonomy        |  90%   |  100%    |  85%
Safety          |  51%   |   60%    |  37%
QS:Behavioral      |  80%   |   95%    |  65%
Extensibility   |  95%   |   49%    |  67%
```

---

## Detailed Analysis

### Part 1: Identity Architecture

| Agent | Score | Key Finding |
|-------|-------|-------------|
| **Hermes** | 17/20 | Strong SOUL.md with anti-sycophancy stance. IDENTITY.md is [EQUIVALENT] via SOUL.md |
| **ZeroClaw** | 4/4 EQ | Runtime generation during onboarding. User-friendly but less auditable |
| **Moltis** | 16/20 | Good anti-behaviors defined. IDENTITY.md scattered across files |

**Winner:** ZeroClaw (EQUIVALENT via runtime wizard) > Hermes (85%) > Moltis (80%)

---

### Part 2: Three-Layer Memory Architecture

| Agent | Layer 1 | Layer 2 | Layer 3 | Total |
|-------|---------|---------|--------|-------|
| **Hermes** | 18/30 | 5/25 | 7/25 | **34/100** |
| **ZeroClaw** | 6/6 | 4/5 | 0/5 | **10/21** |
| **Moltis** | 10/30 | 8/25 | 0/25 | **18/100** |

**Key Findings:**
- **Hermes:** Layer 1 excellent (bounded curation, FTS5), Layers 2 & 3 completely absent
- **ZeroClaw:** Daily notes well-implemented, Layer 3 (Knowledge Graph) completely missing
- **Moltis:** Session-based notes (event-driven vs time-driven), Layer 3 missing

**Winner:** Hermes (34%) > ZeroClaw (48%) > Moltis (18%)

> Note: All three agents lack a proper Knowledge Graph (Layer 3) - this is the most significant gap across all frameworks.

---

### Part 3: Tools & Capabilities

| Agent | Messaging | File | Web | Command | Min Auth | Total |
|-------|-----------|------|-----|---------|----------|-------|
| **Hermes** | 5/5 (11 platforms) | 4/5 | 5/5 | 5/5 | 4/5 | **23/25** |
| **ZeroClaw** | 5/5 (15+ platforms) | 5/5 | 5/5 | 5/5 | 5/5 | **5/5** |
| **Moltis** | 5/5 (5 platforms) | 5/5 | 5/5 | 5/5 | 5/5 | **25/25** |

**Winner:** Moltis (100%) = ZeroClaw (100%) > Hermes (92%)

All three agents have excellent tool infrastructure.

---

### Part 4: Autonomy & Initiative

| Agent | Scheduled | Sub-Agent | Cost Routing | Wake Hooks | Total |
|-------|-----------|-----------|--------------|------------|-------|
| **Hermes** | 5/5 | 5/5 | 5/5 | 3/5 | **18/20** |
| **ZeroClaw** | 4/4 | 4/4 | 4/4 | 4/4 | **4/4** |
| **Moltis** | 5/5 | 5/5 | 3/5 | 4/5 | **17/20** |

**Winner:** ZeroClaw (100%) > Hermes (90%) > Moltis (85%)

All three implement proper autonomous work systems.

---

### Part 5: Safety Rails & Accountability

| Agent | Trust Ladder | Approval Queue | No Social | No Money | No Sensitive | Email Security | Injection | Total |
|-------|--------------|---------------|-----------|----------|-------------|---------------|-----------|-------|
| **Hermes** | 2/5 | 3/5 | 1/5 | 1/5 | 3/5 | 3/5 | 4/5 | **18/35** |
| **ZeroClaw** | 5/5 | 5/5 | 0/5 | 0/5 | 0/5 | 0/5 | 5/5 | **15/25** |
| **Moltis** | 2/5 | 5/5 | 0/5 | 0/5 | 2/5 | 0/5 | 4/5 | **13/35** |

**Critical Gap:** None of the three agents implement non-negotiable safety rules.

**Winner:** Hermes (51%) > ZeroClaw (60%) > Moltis (37%)

---

### Part 6: Behavioral Evaluation

| Agent | Continuity | Initiative | Depth | Accountability | Total |
|-------|------------|------------|-------|---------------|-------|
| **Hermes** | 4/5 | 4/5 | 4/5 | 4/5 | **16/20** |
| **ZeroClaw** | 5/5 | 4/5 | 5/5 | 5/5 | **19/20** |
| **Moltis** | 4/5 | 1/5 | 4/5 | 4/5 | **13/20** |

**Key Finding:** Moltis has NO proactive initiative - a fundamental gap.

**Winner:** ZeroClaw (95%) > Hermes (80%) > Moltis (65%)

**Scoring Note:**

The ZeroClaw Part 6 audit scored 20/20 (5/5 each), but the evidence only demonstrates *infrastructure presence* (TaskPlanTool, GoalsEngine, AGENTS.md) rather than *verified behavioral outcomes*. Hermes received 4/5 for similar infrastructure (TodoStore, session_search). The adjusted score is 19/20.

---

### Part 7: Extensibility & Customization

| Category | Max | Hermes | ZeroClaw | Moltis |
|----------|-----|--------|----------|--------|
| System Prompts | 25 | 25 | 0 | 15 |
| Agent Prompts | 20 | 20 | 1 | 20 |
| Skills | 35 | 35 | 30 | 26 |
| Commands | 25 | 25 | 20 | 13 |
| Tools | 35 | 33 | 35 | 33 |
| Hooks | 35 | 29 | 25 | 31 |
| MCP | 30 | 30 | 0 | 18 |
| Plugins | 35 | 36 | 0 | 12 |
| Extensions | 25 | 10 | 0 | 0 |
| Configuration | 50 | 50 | 40 | 39 |
| **TOTAL** | **305** | **291 (95%)** | **31 (49%)** | **153 (67%)** |

**Winner:** Hermes (95%) > Moltis (67%) > ZeroClaw (49%)

**Key Finding:** Hermes has industry-leading extensibility; ZeroClaw has no formal plugin/extension system.

---

## Critical Gaps Comparison

| Gap | Hermes | ZeroClaw | Moltis |
|-----|--------|----------|--------|
| Knowledge Graph (Layer 3) | ❌ Missing | ❌ Missing | ❌ Missing |
| Non-negotiable safety rules | ❌ Missing | ❌ Missing | ❌ Missing |
| Email as untrusted channel | ⚠️ Partial | ❌ Missing | ❌ Missing |
| Pattern recognition | ❌ Missing | ❌ Missing | ❌ Missing |
| Memory decay system | ❌ Missing | ❌ Missing | ❌ Missing |
| Daily notes automation | ❌ Missing | ✅ Implemented | ⚠️ Event-driven |
| Formal extension system | ⚠️ Via pip | ❌ Missing | ❌ Missing |

---

## Strengths Comparison

| Agent | Primary Strengths |
|-------|-------------------|
| **Hermes** | Extensibility (95%), Skills marketplace, MCP integration, SOUL.md system |
| **ZeroClaw** | Behavioral evaluation (100%), 15+ messaging platforms, security-first design |
| **Moltis** | Tools (100%), Rust-based performance, approval queue system |

---

## Recommendations by Agent

### Hermes Agent
1. **Implement Layers 2 & 3 memory** - Daily notes + Knowledge Graph
2. **Add safety non-negotiables** - Explicit rules for social media, money, contracts
3. **Enable QMD as auto-retrieval** - Make vector search core to memory

### ZeroClaw
1. **Implement Knowledge Graph** - PARA structure with entity summaries
2. **Add system prompt extensibility** - Editable files with variable interpolation
3. **Build plugin architecture** - Formal system for third-party extensions

### Moltis
1. **Add proactive initiative** - Surface pending items without prompting
2. **Implement time-driven daily notes** - Not just event-driven
3. **Add non-negotiable safety rules** - Explicit prohibitions

---

## Conclusion

| Rank | Agent | Score | Best For |
|------|-------|-------|----------|
| 🥇 1 | **Hermes Agent** | 79% | Teams needing extensibility, MCP integration, and a flexible skill marketplace |
| 🥈 2 | **ZeroClaw** | 61% | Production deployments prioritizing security and behavioral consistency |
| 🥉 3 | **Moltis** | 46% | Rust-based systems needing native performance with basic agent capabilities |

**Universal Gap:** All three agents lack a proper Knowledge Graph (Layer 3) and non-negotiable safety rules. These should be prioritized for any framework aiming to be a true "Hired AI Employee."
