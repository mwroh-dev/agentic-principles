# Per-Agent Knowledge Patterns

Status: confirmed
Last reviewed: 2026-05-18

## Principle

Each agent in a multi-agent system should have an explicit knowledge pattern — a declared strategy for what it stores, when it updates, and how it separates episodic records from semantic learning. The agent:knowledge relationship is 1:N — one agent owns multiple knowledge pieces organized by type.

Knowledge patterns are not uniform across agents. The appropriate strategy depends on the agent's task type and the verifiability of its outputs.

---

## Pattern Definitions

### Abstract Reasoning Agent (goal analysis, planning, technical decisions)

**Strategy:** B — batch synthesis, N=3

**Rationale:** Abstract reasoning produces outputs that are hard to verify run-by-run. A single run's output may reflect noise, unusual input, or a poorly-formed goal. Three episodes provide a minimum cross-episode signal before encoding as semantic knowledge.

**Episodic storage:** Per-run summary (what was analyzed, what was concluded, what constraints were encountered). Sliding window retention (10 runs typical).

**Semantic storage:** Recurring structural patterns in goal types, constraint categories, decision taxonomies. Updated after N=3 episodes. Old versions linked via `replaces` field for version chaining.

**Update trigger:** Fixed N=3 threshold.

---

### Service Discovery / Integration Agent

**Strategy:** B + prediction-error

**Rationale:** Service capability patterns are relatively stable — the same service tends to expose the same methods. Semantic knowledge should only update when existing knowledge fails to predict the actual discovery result. Routine confirmations do not warrant a knowledge update.

**Episodic storage:** Per-run record of services found, methods selected, selection rationale.

**Semantic storage:** Per-service recommended integration method patterns. Updated only when `prediction_match: false` — the agent found something different from what its semantic model predicted.

**Update trigger:** Prediction-error (existing semantic knowledge failed to anticipate result).

---

### Security / Risk Policy Agent

**Strategy:** B + prediction-error (Nemori pattern)

**Rationale:** Security policy should be conservative and stable. Policy shifts driven by a single unusual run create risk. The agent's semantic knowledge (risk calibration, approval frequency patterns) should only update when a genuine surprise occurs — a risk judgment that existing knowledge would not have produced.

**Episodic storage:** Per-run risk judgment record (what was assessed, what decision was reached, whether it matched prior policy expectations).

**Semantic storage:** Risk calibration patterns, approval frequency baselines, known risk type taxonomy. Updated only on genuine policy-relevant surprise.

**Update trigger:** Prediction-error on risk judgment.

**Future need:** A `knowledge_valid_until` field to mark when a calibration pattern should be re-evaluated (e.g., after a major environment change).

---

### Orchestrator / Routing Agent

**Strategy:** B + prediction-error + meta layer

**Rationale:** Routing knowledge (which surface to use, which agent to assign, what fallback to take) changes slowly and only on failure. The orchestrator additionally accumulates meta-level knowledge about pipeline performance across runs — a third layer beyond episodic and semantic.

**Episodic storage:** Per-run phase routing records (what surface was requested, what was resolved, whether fallback was used).

**Semantic storage:** Routing failure patterns ("under what conditions does repo_skill fail?"), surface reliability patterns.

**Meta storage:** Pipeline performance summaries across N=5+ runs. Cross-agent coordination patterns. This layer is unique to orchestrator-type agents.

**Update trigger:** Prediction-error on routing decision (fallback occurred when primary surface was expected to succeed).

---

## Common Structure for All Patterns

```
knowledge/{agent-type}/
  episodic/          ← per-run records (written after each run)
  semantic/          ← accumulated patterns (updated per strategy)
  meta/              ← orchestrator only: cross-run pipeline summaries
```

Each semantic file should include:
- `version` — monotonically incrementing
- `replaces` — pointer to previous version (version chaining)
- `updated_after_runs` — which episodic runs triggered this update
- `prediction_match` — (for B+pred-error agents) whether prior knowledge predicted this result

## Evidence

- **CoALA: Cognitive Architectures for Language Agents** (Sumers et al., 2023) — Establishes the foundational taxonomy of episodic, semantic, and procedural memory as distinct structured stores; the 1:N agent:knowledge model directly implements CoALA's separation of episodic records from semantic learning as separately managed components per agent.
- **ExpeL: LLM Agents Are Experiential Learners** (AAAI 2024) — Demonstrates that cross-episode batch synthesis (not immediate per-run synthesis) produces more reliable semantic knowledge for planning and reasoning agents; supports Strategy B / N=3 mapping for abstract reasoning agents.
- **SYNAPSE** — Batch synthesis with N=5 achieves +23% improvement on multi-hop reasoning tasks vs immediate update; directly motivates the N=3–5 range used for abstract and meta/orchestrator agents in this document.
- **Nemori** (2026) — Formalizes prediction-error as a knowledge consolidation trigger: consolidate only when existing knowledge fails to anticipate the actual result; directly maps to the `prediction_match: false` update trigger specified for service discovery, security/risk, and routing agents.
- **Orchestration of Multi-Agent Systems: Architectures, Protocols, and Enterprise Adoption** (arXiv 2601.03671, Jan 2026) — Identifies meta-level knowledge (cross-agent coordination patterns, pipeline performance summaries) as a distinct knowledge layer for orchestrator-class agents, separate from task-level episodic and semantic stores; supports the `meta/` layer unique to the orchestrator pattern.
- **Anthropic "Building Effective Agents"** (anthropic.com/research/building-effective-agents) — Prescribes per-agent memory scoping and distinguishes agent-owned knowledge from shared pipeline state; foundational rationale for the per-agent (not shared/global) structure defined in this document.

## When to Apply

| Signal | Decision | What breaks if ignored |
|--------|----------|----------------------|
| A new agent is being defined and no knowledge pattern has been declared | Declare the pattern before implementing any storage — choose strategy (A, B, or B+prediction-error) based on the agent's task type | Without a declared pattern, knowledge update logic grows ad-hoc; different runs use inconsistent strategies and the semantic store becomes unreliable |
| An agent's output quality degrades or shows inconsistent behavior across runs | Check whether the agent is using the wrong strategy for its task type — abstract agents on Strategy A are the most common misalignment | Mismatched strategies produce self-reinforcing drift (A on abstract tasks) or stale knowledge (B on rapidly-changing environments) |
| A prediction-error triggered agent never updates its semantic store | The agent has no prior semantic knowledge to compare against — fall back to fixed-N (cold-start problem) | Prediction-error requires existing knowledge; cold-start agents with this trigger never update and remain uninformed indefinitely |
| A semantic store grows continuously without pruning and old entries are never superseded | Semantic decay — the environment has changed but the store retains stale patterns | Agents apply outdated patterns to new situations; errors increase over time as the store diverges from current conditions |

## Known Limits / Failure Modes

- **Cold-start problem:** Prediction-error strategies require existing semantic knowledge to compare against. For new agents with empty semantic stores, fall back to fixed-N until the semantic store has at least one populated entry.
- **N selection:** The optimal N for batch synthesis is not derivable analytically. N=3 for abstract agents and N=5 for meta/routing agents are empirically motivated starting points, not universal constants.
- **Semantic decay:** Semantic knowledge accumulated under one set of conditions may become stale when the environment changes. No current strategy handles semantic invalidation automatically.
