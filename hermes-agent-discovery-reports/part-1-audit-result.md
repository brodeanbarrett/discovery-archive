# Part 1: Identity Architecture (Who is the AI?)

*A hired AI has a defined role, personality, and specific perspective, rather than acting as a generic, sycophantic assistant.*

- [ ] **`SOUL.md` exists and is configured:** Defines the AI's voice, tone, and behavioral boundaries.
- [ ] **Anti-behaviors are defined:** Explicitly lists what the AI is NOT (e.g., "not sycophantic," "not robotic," "not preachy").
- [ ] **`IDENTITY.md` exists and is configured:** Gives the AI a specific name, explicit job title/role, and scope of responsibility.
- [ ] **Permission to push back:** The AI is explicitly instructed to point out flaws, disagree when appropriate, and ask clarifying questions instead of guessing.

## Scores

Fill in your evaluation scores here.

| Criterion | Score (1-5) | Notes |
|-----------|-------------|-------|
| SOUL.md exists and configured | 5/5 | Well implemented: `hermes_cli/default_soul.py` contains comprehensive `DEFAULT_SOUL_MD` with voice ("You're a peer"), tone guidance, and behavioral boundaries. Extensive documentation in `website/docs/user-guide/features/personality.md` defines SOUL.md's purpose, loading behavior, and security scanning. |
| Anti-behaviors defined | 4/5 | Anti-sycophancy explicitly stated in default_soul.py "Avoid" section. Style guidance ("Write like a person, not a spec sheet", "No filler", "No hype words") implicitly covers robotic/preachy. Minor deduction: "not robotic" and "not preachy" not labeled as explicit anti-behaviors. |
| IDENTITY.md exists and configured | 3/5 | No `IDENTITY.md` file exists. Name/role defined within SOUL.md ("You are Hermes, an AI assistant made by Nous Research") and as `DEFAULT_AGENT_IDENTITY` fallback in `agent/prompt_builder.py`. Identity scattered across files rather than centralized. [EQUIVALENT] SOUL.md serves IDENTITY.md purpose. |
| Permission to push back | 5/5 | Explicitly stated in default_soul.py: "Push back when you disagree" and "Sit in ambiguity when that's the honest answer." Practical examples demonstrate disagreement (e.g., "Rust won't help much" response). callbacks.py includes `clarify_callback` for prompting clarifying questions. |

**Part 1 Total: 17/20**

## Comments
**Strengths:**
- SOUL.md is well-implemented with comprehensive voice, tone, and behavioral guidance in `hermes_cli/default_soul.py`
- Explicit anti-sycophancy stance with specific examples of prohibited phrases ("Great question!", "Absolutely!", etc.)
- "Push back when you disagree" is explicitly stated with practical examples in the voice section
- Extensive documentation in `website/docs/user-guide/features/personality.md` covers SOUL.md loading, security scanning, and best practices

**Weaknesses:**
- No dedicated `IDENTITY.md` file; identity (name, job title) is embedded within SOUL.md and `DEFAULT_AGENT_IDENTITY` fallback
- "not robotic" and "not preachy" are implied through style guidance but not explicitly labeled as anti-behaviors
- Identity configuration is scattered: SOUL.md contains voice AND identity; DEFAULT_AGENT_IDENTITY in prompt_builder.py is a separate fallback

**Recommendation:**
- Consider extracting name/role/job-title into a dedicated IDENTITY.md section or documenting the [EQUIVALENT] relationship explicitly
- Add explicit "not robotic" and "not preachy" labels to the Avoid section for clarity

