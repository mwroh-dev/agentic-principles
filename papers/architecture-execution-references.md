# Architecture & Execution — Annotated Paper References

Status: confirmed
Last reviewed: 2026-05-19

This file is the canonical citation index for papers referenced in `confirmed/architecture/`, `confirmed/execution/`, `confirmed/agents/`, `confirmed/observability/`, and `confirmed/safety/`.

---

## Agent Contract & Role Separation

### Agent Behavioral Contracts (ABC)
**Citation:** arXiv:2602.22302, Feb 2026.
**Key claim:** Defines every agent as a formal contract C=(P, I, G, R): preconditions, invariants, goals, and recovery behaviors. Across 1,980 sessions, contracted agents detected 5.2–6.8 soft violations per session that uncontracted baselines missed entirely, with 88–100% hard constraint compliance.
**Cited in:** `confirmed/architecture/agent-as-contract-not-code.md`, `confirmed/execution/evaluation-before-implementation.md`

### CodeDelegator: Role Separation in Code-as-Action Agents
**Citation:** arXiv:2601.14914, 2026.
**Key claim:** Empirically demonstrates that mixing a judgment layer (Delegator) with an execution layer (Coder) causes context pollution. Separating them eliminates interference and is the central empirical result.
**Cited in:** `confirmed/architecture/agent-as-contract-not-code.md`

### Contract-based Design and Verification of Multi-Agent Systems
**Citation:** arXiv:2412.13114, 2024.
**Key claim:** Applies assume-guarantee contract theory to multi-agent systems, proving that role interface can be fully specified and compositionally verified without access to implementation.
**Cited in:** `confirmed/architecture/agent-as-contract-not-code.md`

---

## Rule vs Skill Layer Separation

### Policy Compiler for Secure Agentic Systems
**Citation:** arXiv:2602.16708, Feb 2026.
**Key claim:** Formalizes agent policies as Datalog-based declarative rules compiled into a reference monitor that intercepts skill execution at the runtime boundary without LLM participation.
**Cited in:** `confirmed/architecture/rule-is-not-skill.md`

### SoK: Agentic Skills — Beyond Tool Use in LLM Agents
**Citation:** arXiv:2602.20867, Feb 2026.
**Key claim:** Defines a skill as "applicability condition + execution policy + termination criteria" and explicitly distinguishes the skill layer from the governance/audit layer. Mixing governance into skill definitions degrades both auditability and reusability.
**Cited in:** `confirmed/architecture/rule-is-not-skill.md`

### Externalization in LLM Agents
**Citation:** arXiv:2604.08224, Apr 2026.
**Key claim:** Governance is the coordination layer above externalized skills, protocols, and memory. Direct quote: "Capabilities that earlier systems expected the model to recover internally are now externalized into memory stores, reusable skills, interaction protocols, and the surrounding harness."
**Cited in:** `confirmed/architecture/rule-is-not-skill.md`

---

## Observability

### A Taxonomy of AgentOps
**Citation:** arXiv:2411.05285, Nov 2024.
**Key claim:** Directly argues for separating raw trace data, interpretability artifacts, and debuggability insights into distinct layers. Conflating layers prevents reproducible debugging.
**Cited in:** `confirmed/observability/3-tier-observability-model.md`

### Beyond Black-Box Benchmarking
**Citation:** arXiv:2503.06745, Mar 2025.
**Key claim:** Mixing telemetry with evaluation results produces benchmarks that cannot be reproduced across runs.
**Cited in:** `confirmed/observability/3-tier-observability-model.md`

### AgentTrace
**Citation:** arXiv:2602.10133, Feb 2026.
**Key claim:** Three-surface taxonomy (operational, cognitive, contextual) for structured log separation. Each surface must be independently queryable without requiring access to the others.
**Cited in:** `confirmed/observability/3-tier-observability-model.md`

### MAESTRO: Multi-Agent Evaluation Suite
**Citation:** arXiv:2601.00481, Jan 2026.
**Key claim:** Implements separate export pipelines for raw telemetry and system-level evaluation signals. Downstream evaluation quality degrades when these pipelines share a data sink.
**Cited in:** `confirmed/observability/3-tier-observability-model.md`

### AI Observability for Large Language Model Systems: Multi-Layer Analysis
**Citation:** arXiv:2604.26152, Apr 2026.
**Key claim:** Identifies absence of inter-layer signal separation as the primary cause of single-layer monitoring failures in LLM systems.
**Cited in:** `confirmed/observability/3-tier-observability-model.md`

---

## Evaluation

### EDDOps: Evaluation-Driven Development and Operations of LLM Agents
**Citation:** arXiv:2411.13768, Nov 2024/2025.
**Key claim:** Proposes evaluation as the explicit starting point of the agent development lifecycle. Without upfront criteria, agent pipeline design, artifact definitions, and LLM boundary placement converge on implementer defaults rather than system requirements.
**Cited in:** `confirmed/execution/evaluation-before-implementation.md`

### TDAD: Test-Driven AI Agent Definition
**Citation:** arXiv:2603.08806, 2026.
**Key claim:** Operationalizes "evaluation before implementation" as a compiler pipeline: behavioral specification → test suite → prompt synthesis. Evaluation ordering determines what the agent becomes.
**Cited in:** `confirmed/execution/evaluation-before-implementation.md`

### Toward Architecture-Aware Evaluation Metrics for LLM Agents
**Citation:** arXiv:2601.19583, 2026 (CAIN '26).
**Key claim:** Evaluation metrics must be anchored to specific architectural components (memory, planning, tool-use, inter-agent communication). Without this linkage upfront, architectural boundaries cannot be set.
**Cited in:** `confirmed/execution/evaluation-before-implementation.md`

### Formally Specifying the High-Level Behavior of LLM-Based Agents
**Citation:** arXiv:2310.08535, Oct 2023.
**Key claim:** High-level declarative behavioral specification written before implementation automatically constrains the agent's decoding monitor. Post-hoc specification cannot reconstruct the same constraints.
**Cited in:** `confirmed/execution/evaluation-before-implementation.md`
