# Subagent Context Isolation / Two-Stage Review

Status: confirmed (Pattern A: Subagent Isolation) / candidate (Pattern B: Two-Stage Review)
Last reviewed: 2026-05-18

---

## Pattern A: Subagent Context Isolation [confirmed]

### Principle

A subagent's prompt must be fully self-contained. The orchestrator passes only task-relevant context, not accumulated dialogue history. Dispatch a fresh subagent per task — not a context-inheriting continuation of the orchestrator. The subagent derives all decisions from the task specification alone and returns only its final output to the parent; all intermediate work remains internal to the subagent's context.

Passing orchestrator history to subagents is an error, not a shortcut.

**Named failure modes (OpenAI official terminology):**
- **Context pollution** — irrelevant decisions from prior phases bleed into the subagent's reasoning, biasing its output toward earlier, superseded assumptions.
- **Context rot** — stale assumptions from earlier phases corrupt current reasoning, causing the subagent to behave as though earlier constraints still apply when the specification has since changed.

### Evidence

**Official documentation:**

- **Anthropic Claude Agent SDK** (anthropic.com/engineering/building-agents-with-the-claude-agent-sdk) — "Each subagent has an isolated 200K-token context. A parent agent spawns a subagent with a specific prompt to do its work, and returns only its final output to the parent, keeping all intermediate work internal." Explicit directive: do not pass orchestrator history to subagents. Additional: "Don't assume subagent outputs are correct — build orchestration logic that verifies or reviews outputs before passing them downstream."
- **OpenAI Codex Subagents** (developers.openai.com/codex/subagents) — Names "context pollution" and "context rot" as explicit failure modes of non-isolated subagents. Subagent isolation is prescribed as the solution.

**Academic papers:**

- **Context Engineering for Multi-Agent LLM Code Assistants** (arXiv 2508.08322, Aug 2025) — Context isolation is a first-class engineering concern. Subagents receiving only task-relevant context outperform agents with full accumulated history. Cross-contamination between phases is a documented failure mode.
- **Orchestration of Multi-Agent Systems: Architectures, Protocols, and Enterprise Adoption** (arXiv 2601.13671, Jan 2026) — Context isolation per subagent (each receives only task-relevant information, not full dialogue history) identified as a core architectural principle.
- **LLM-Based Multi-Agent Systems for Software Engineering: Literature Review** (arXiv 2404.04834, 2024) — The dominant multi-agent SE pattern: specialized agents with isolated contexts (analyst → coder → tester), each in a fresh context.
- **Verified Multi-Agent Orchestration: Plan-Execute-Verify-Replan** (arXiv 2603.11445, ICLR 2026 Workshop) — LLM-based verification pass is a structural orchestration component, separate from execution agents.

**Named in literature:** De facto standard, documented under "isolated context window" (Anthropic), "context pollution/rot" (OpenAI). Consistent across academic literature and official documentation.

### When to Apply

| Signal | Decision | What breaks if ignored |
|--------|----------|------------------------|
| Multi-phase pipeline with distinct specifications per phase | Apply subagent isolation to every dispatch; construct a fully self-contained prompt per task | Prior-phase decisions bleed into current reasoning (context pollution); subagent output reflects superseded assumptions |
| Subagent output quality degrades as orchestrator history grows | Switch to isolated dispatch; do not pass accumulated dialogue history | Context rot: stale assumptions from earlier phases corrupt current reasoning; prompt tuning does not fix a context boundary problem |
| Subagent produces outputs referencing superseded decisions from earlier phases | Diagnose context pollution; rebuild subagent prompts as self-contained specifications | The subagent reasons from stale context; output validity depends on which prior decisions it inherited arbitrarily |
| Task specification is fully definable without reference to prior conversational turns | Use isolated dispatch unconditionally | Full context injection guarantees decision provenance; inheriting history reduces token cost but introduces rot risk |

### Known Limits

- Fresh subagents require full context injection per task. Token cost is higher than context inheritance. Budget token usage per subagent accordingly.
- If the task prompt is ambiguous or incomplete, isolation does not help. Task prompts must be fully self-contained — isolation enforces correct scope, not correct specification.
- The orchestrator bears the responsibility of selecting and packaging only the task-relevant context. Poorly constructed injection reintroduces contamination from a different direction.

### Promotion History

- 2026-05-18: Confirmed. Subagent isolation is a de facto standard with strong backing from Anthropic and OpenAI official documentation plus four independent academic papers. Pattern A promoted to confirmed/architecture.

---

## Pattern B: Two-Stage Review [candidate — not yet named in literature]

### Principle

Review spec compliance first (does the output match the specification?), then code quality (is the implementation correct and maintainable?). Running both in a single pass systematically under-weights spec compliance because reviewers shift attention to code-level detail.

### Evidence

This ordering appears as an architectural element in multi-agent SE literature (analyst → coder → tester pipelines) and in Plan-Execute-Verify structures, but the specific spec-compliance-first → code-quality-second sequencing is not independently named or empirically validated as a distinct pattern. The observation is that single-pass review consistently surfaces implementation defects while missing scope drift and missing requirements.

Closest supporting structure: **Verified Multi-Agent Orchestration: Plan-Execute-Verify-Replan** (arXiv 2603.11445) separates the verification pass from the execution pass architecturally, which parallels the two-stage framing.

### When to Apply

- When spec compliance failures have been observed in single-pass review output.
- When the specification is sufficiently detailed that compliance is independently checkable before quality assessment begins.

### Known Limits

- Two-stage review doubles review latency. Apply only when spec compliance failures have been observed in single-pass review.
- This pattern is not yet independently named. Treat it as an operational heuristic, not an established architectural principle.

### Promotion History

- 2026-05-18: Candidate. The two-stage framing is not independently named or confirmed in literature. Remains candidate until external confirmation.
