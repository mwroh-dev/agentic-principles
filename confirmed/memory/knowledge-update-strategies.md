# Knowledge Update Strategies — Strategy A vs B

Status: confirmed
Last reviewed: 2026-05-18

## Principle

When an agent accumulates semantic knowledge across runs, two primary update strategies exist. The choice depends on the agent's task type, output verifiability, and tolerance for drift.

---

### Strategy A — Immediate Post-Run Synthesis

After each run completes, the agent synthesizes new knowledge immediately and writes it to the semantic store.

**Characteristics:**
- Zero latency between run completion and knowledge update
- Each episode directly influences the next run
- Simple to implement

**Best for:** Agents with verifiable, concrete outputs — coding, schema generation, test writing. When a run produces a definite artifact (a passing test, a valid schema), the learning is reliable enough to absorb immediately.

**Failure modes:**
- Error propagation: a failed or unusual run contaminates the knowledge store
- Self-reinforcing drift: if the agent makes a systematic mistake, it immediately encodes that mistake as "learned"
- Drift bound: O(T·ε) — unbounded over time

---

### Strategy B — Batch / Threshold Synthesis

The agent buffers N episodic records and synthesizes knowledge after N runs (or when a trigger condition is met).

**Characteristics:**
- Cross-episode patterns emerge before becoming encoded
- Single-episode noise is averaged out
- Drift bound: O(N·ε) — bounded per batch

**Best for:** Agents whose outputs are abstract or hard to verify run-by-run — reasoning, planning, security policy, orchestration routing. When no single run is authoritative, waiting for multiple data points produces more reliable patterns.

**Trigger variants:**
- **Fixed N:** synthesize every N runs (e.g., N=3 for abstract agents, N=5 for meta/routing)
- **Prediction-error trigger (Nemori 2026):** only synthesize when existing knowledge *fails* to anticipate the actual result. Ideal for stable-policy agents (security, routing) where knowledge should only update on genuine surprise, not routine confirmation.

**Failure modes:**
- Stale knowledge: if the environment changes rapidly, batch synthesis lags
- Buffer management: episodic records must be retained until synthesis; retention window needs explicit policy

---

### Agent Type → Strategy Mapping

| Agent type | Recommended strategy | Rationale |
|---|---|---|
| Coding / schema generation / test writing | A (immediate) | Outputs are verifiable; learning is reliable per run |
| Goal analysis / planning | B (N=3) | Abstract; multiple episodes needed to separate signal from noise |
| Service discovery / integration | B + prediction-error | Service method patterns are stable; update only on deviation |
| Security / risk policy | B + prediction-error | Policy should only shift on genuine surprise, not routine |
| Orchestrator / routing | B + prediction-error + meta layer | Routing knowledge changes only on fallback; meta layer tracks pipeline-level patterns across N=5+ runs |

## Evidence

- **Reflexion** (NeurIPS 2023) — canonical Strategy A implementation: immediate verbal reflection after each run.
- **A-Mem** — adaptive memory with immediate update; shows Strategy A works when output quality is verifiable.
- **ExpeL** (AAAI 2024) — Strategy B implementation: synthesizes patterns after accumulating N episodes; demonstrates cross-episode generalization.
- **SYNAPSE** (arXiv:2601.02744, Jan 2026) — batch episodic-semantic memory via spreading activation: +32% multi-hop F1 vs A-Mem baseline; 95% token reduction vs full-context methods on the LoCoMo benchmark.
- **GAM: Hierarchical Graph-based Agentic Memory** (arXiv:2604.12285, Apr 2026) — event-driven Semantic Consolidation Phase (Strategy B) vs immediate stream updates (Strategy A): +13% F1 vs Mem0, +18% temporal reasoning F1, +86% on LongDialQA. Explicitly identifies "Memory Loss" and "Semantic Drift" as documented failure modes of immediate stream updates. Strongest direct empirical comparison of Strategy A vs B.
- **Nemori** (2026) — prediction-error trigger: consolidate only when existing knowledge fails to anticipate actual result. Prevents unnecessary drift in stable-policy agents.
- **OpenAI Agents SDK Cookbook** (developers.openai.com/cookbook/examples/agents_sdk/context_personalization) — Official practitioner recommendation for Strategy B: "Two-phase memory processing (note taking → consolidation) is more reliable than one-shot." Session-end batch consolidation is the prescribed architectural pattern.
- **Anthropic "Effective Context Engineering for AI Agents"** (anthropic.com/engineering/effective-context-engineering-for-ai-agents) — Practitioner pattern for Strategy A: in-run incremental writes to persistent memory ("regularly writes notes persisted to memory outside of the context window"). Endorses immediate writes for within-run state tracking of verifiable, concrete outputs.

## When to Apply

| Signal | Decision | What breaks if ignored |
|--------|----------|----------------------|
| Agent output quality degrades immediately after a failed or unusual run | Switch from Strategy A to B — the agent is absorbing noise immediately | A single bad run contaminates the semantic store; the agent encodes the failure as "learned behavior" and repeats it |
| Agent's semantic knowledge is stable but updates after every routine run with no meaningful change | Switch to prediction-error trigger — updates should only fire on genuine surprise | Unnecessary updates add latency and introduce noise; stable-policy agents drift when their policy shouldn't change |
| Abstract reasoning agent shows self-reinforcing errors (same wrong pattern across multiple runs) | Switch from A to B (N=3) — immediate synthesis encoded a systematic mistake before cross-episode averaging could correct it | Drift bound grows unboundedly (O(T·ε) for Strategy A vs O(N·ε) for Strategy B) |
| New agent with empty semantic store is assigned prediction-error trigger | Fall back to fixed-N until the semantic store has at least one populated entry | Prediction-error requires existing knowledge to compare against; cold-start agents with this trigger never update |

## Known Limits / Failure Modes

- **Transition policy unsolved:** No formal rule for when an episodic record graduates to semantic status. Current practice uses fixed N or prediction-error as proxies.
- **Strategy A + abstract reasoning = high risk:** Abstract agents are susceptible to self-reinforcing drift when using Strategy A. The literature consistently points to Strategy B for these cases.
- **Prediction-error trigger requires a prediction:** The agent must have existing semantic knowledge to compare against. For cold-start agents, fall back to fixed-N until the semantic store is populated.
