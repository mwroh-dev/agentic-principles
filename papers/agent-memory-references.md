# Agent Memory — Annotated Paper References

Status: confirmed
Last reviewed: 2026-05-18

This file is the canonical citation index for all papers referenced in `confirmed/`. Each entry includes the key claim relevant to agent design and which confirmed documents cite it.

---

## Memory Taxonomy

### CoALA — Cognitive Architectures for Language Agents
**Citation:** Sumers et al., 2023. https://arxiv.org/abs/2309.02427  
**Key claim:** Four-way memory taxonomy — episodic (what happened in earlier decision cycles), semantic (knowledge about the world and the agent itself), procedural (how to do things), working (current context). Episodic and semantic differ not just in content but in mutability, consumer, and retrieval semantics.  
**Cited in:** `confirmed/memory/artifact-vs-knowledge.md`

---

## Episodic vs Semantic Separation

### CORAL — Towards Autonomous Multi-Agent Evolution
**Citation:** Qu et al., 2026. https://arxiv.org/html/2604.01658v1  
**Key claim:** Filesystem-level split between `attempts/` (timestamped, pipeline-written, immutable) and `notes/skills/` (agent-curated, deliberately editable). Agents evolve skills without human intervention.  
**Cited in:** `confirmed/memory/artifact-vs-knowledge.md`, `confirmed/architecture/agent-directory-structure.md`

### SSGM — Stability and Safety Governed Memory
**Citation:** Lam et al., 2026. https://arxiv.org/html/2603.11768v1  
**Key claim:** Pairs a Mutable Active Graph (fast semantic reasoning) with an Immutable Episodic Log (operational source of truth). Drift in the mutable graph corrected by replaying the immutable log — requires physical separation. Drift bound O(N·ε) with batch vs O(T·ε) unbounded with immediate.  
**Cited in:** `confirmed/memory/artifact-vs-knowledge.md`, `confirmed/memory/knowledge-update-strategies.md`, `confirmed/architecture/agent-directory-structure.md`

### MIRIX — Multi-Agent Memory System for LLM-Based Agents
**Citation:** Wang & Chen, 2025. https://arxiv.org/abs/2507.07957  
**Key claim:** Timestamped immutable episodic entries vs evolving semantic memory as separate subsystems. Demonstrates the separation is necessary at system scale, not just conceptually.  
**Cited in:** `confirmed/memory/artifact-vs-knowledge.md`

---

## Audit Records

### Audit Trails for Accountability in Large Language Models
**Citation:** Ojewale et al., Brown University, 2025. https://arxiv.org/html/2601.20727v1  
**Key claim:** Audit records are governance infrastructure for "internal compliance teams, external regulators, and risk management" — not for agents. Must be tamper-evident and append-only. Human-facing, not agent-facing.  
**Cited in:** `confirmed/memory/artifact-vs-knowledge.md`, `confirmed/architecture/agent-directory-structure.md`

---

## Knowledge Update Strategies

### Reflexion
**Citation:** Shinn et al., NeurIPS 2023.  
**Key claim:** Canonical Strategy A implementation — verbal reflection after each run, immediately updating the agent's memory. Effective when output quality is verifiable.  
**Cited in:** `confirmed/memory/knowledge-update-strategies.md`

### A-Mem — Adaptive Memory
**Key claim:** Adaptive memory with immediate update (Strategy A). Demonstrates reliability when outputs are concrete and verifiable.  
**Cited in:** `confirmed/memory/knowledge-update-strategies.md`

### ExpeL — Experience-based Learning
**Citation:** AAAI 2024.  
**Key claim:** Strategy B — accumulates N episodic records before synthesizing cross-episode patterns. Demonstrates that batch synthesis produces more reliable generalizations than immediate update for abstract tasks.  
**Cited in:** `confirmed/memory/knowledge-update-strategies.md`

### SYNAPSE
**Key claim:** Batch synthesis with N=5 achieves +23% improvement on multi-hop reasoning tasks compared to immediate update. Quantifies the benefit of cross-episode aggregation.  
**Cited in:** `confirmed/memory/knowledge-update-strategies.md`

### Nemori
**Citation:** 2026.  
**Key claim:** Prediction-error trigger — consolidate knowledge only when existing semantic knowledge fails to anticipate the actual result. Prevents unnecessary updates in stable-policy domains. Ideal for security and routing agents where knowledge should shift only on genuine surprise.  
**Cited in:** `confirmed/memory/knowledge-update-strategies.md`, `confirmed/agents/per-agent-knowledge-patterns.md`
