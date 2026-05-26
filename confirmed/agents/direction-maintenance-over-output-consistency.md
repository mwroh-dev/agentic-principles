# Direction Maintenance Over Output Consistency

Status: confirmed
Last reviewed: 2026-05-19

## Principle

In multi-agent systems, a harness that enforces uniform output format without a mechanism to detect and recover directional drift will produce consistently formatted but misaligned results. The harness's primary responsibility is not format unification — it is goal-direction enforcement: detecting when an agent's behavior has deviated from the intended trajectory and triggering corrective feedback before compounding errors propagate downstream.

Violating this principle produces systems where agents generate structurally consistent outputs while silently drifting from original intent — a failure mode that is harder to detect than an outright error, because the outputs appear valid. Output consistency without directional feedback is a false signal of reliability.

Apply the following rule: if an agent's trajectory cannot be re-anchored without human intervention, the harness has failed regardless of how uniform its outputs are.

Decision logic:
- If drift is detected → trigger corrective feedback loop before next delegation step
- If drift is undetectable by the harness → add an observability mechanism; do not accept absence of feedback as alignment
- If format unification is the primary design goal → re-examine whether direction enforcement has been deprioritized

## Evidence

### Academic Papers

- **Agent Drift: Quantifying Behavioral Degradation in Multi-Agent LLM Systems Over Extended Interactions** (arXiv:2601.04170, January 2026, Abhishek Rath) — Defines and measures semantic drift (progressive deviation from original intent), coordination drift (breakdown in agent consensus), and behavioral drift (emergence of unintended strategies) in multi-agent LLM systems over extended interactions. Proposes the Agent Stability Index (ASI) and three drift mitigation mechanisms — episodic memory consolidation, drift-aware routing protocols, and adaptive behavioral anchoring — establishing that direction recovery requires explicit architectural mechanisms, not just better prompting.

- **Alignment Tipping Process: How Self-Evolution Pushes LLM Agents Off the Rails** (arXiv:2510.04860, October 2025, Han, Xiong, Liu et al.) — Demonstrates that alignment in LLM agents is not a static property but a dynamic one vulnerable to feedback-driven decay: "alignment benefits erode rapidly under self-evolution, with initially aligned models converging toward unaligned states." In multi-agent environments, misaligned behaviors propagate through Imitative Strategy Diffusion; current RL-based alignment techniques offer insufficient protection against this degradation. Establishes that directional maintenance requires active detection mechanisms during deployment, not only at training time.

- **Harness as an Asset: Enforcing Determinism via the Convergent AI Agent Framework (CAAF)** (arXiv:2604.17025, April 2026, Tianbao Zhang) — Proposes State Locking with binary PASS/FAIL constraint assertions (UAI) as a harness feedback architecture. Empirical result: a commodity-model CAAF agent achieves 100% constraint-paradox detection across 30 and 20 trials in two domains; a monolithic frontier model without state locking achieves 0% in the same conditions. Finding: "Architecture supersedes raw model capability" — direction enforcement is a structural property of the harness, not an emergent property of model capability.

- **Natural-Language Agent Harnesses** (arXiv:2603.25723, March 2026, Pan, Zou, Guo et al., Tsinghua University / Harbin Institute of Technology) — Externalizes harness control logic as executable natural-language artifacts with explicit execution contracts, named failure taxonomies, and verification stages at intermediate steps. Self-evolution modules improve directional alignment without proportional cost increase; the framework outperforms code-based equivalents (47.2% vs 30.4% on OSWorld) precisely because verification is attached to intermediate progress, not only final output.

### Official Best Practices

- **Anthropic — Building Effective Agents** (https://www.anthropic.com/research/building-effective-agents) — Prescribes ground-truth environmental feedback at each execution step: "During execution, it's crucial for the agents to gain 'ground truth' from the environment at each step (such as tool call results or code execution) to assess its progress." Stopping conditions and human checkpoints are listed as core safeguards against goal drift. Output consistency is not listed as a design objective; directional feedback is.

- **Anthropic — How We Built Our Multi-Agent Research System** (https://www.anthropic.com/engineering/multi-agent-research-system) — Orchestrator embeds scaling rules in agent prompts to prevent effort drift; observability systems monitor "agent decision patterns and interaction structures" rather than output format; iterative evaluation loops with real-time course correction are used per-agent per-step. Off-track detection is assigned to both human oversight and system observability, not inferred from output format.

### Named in Literature?

No unified name. Closest named concepts: "semantic drift" (arXiv:2601.04170), "alignment tipping" (arXiv:2510.04860), "behavioral anchoring," and "state locking" (arXiv:2604.17025). The specific formulation — that harness design should prioritize direction-recovery feedback mechanisms over output-format consistency — is implied across all four sources but not named as a single pattern.

## When to Apply

Apply when:

| Condition | Signal |
|-----------|--------|
| 1. Harness design is evaluated primarily by output format uniformity | "Do all agents return the same schema?" is the primary review question |
| 2. No per-step feedback mechanism exists between orchestrator and subagent | Agent completion is inferred from output presence, not trajectory validation |
| 3. Multiple agents operate across extended interaction sequences | Risk of semantic, coordination, or behavioral drift is non-trivial (see arXiv:2601.04170) |
| 4. A subagent's work is delegated and synthesized without intermediate verification | Errors compound before detection |
| 5. Self-evolving or reinforcement-learning-updated agents are deployed | Alignment tipping risk is elevated (see arXiv:2510.04860) |

Do not apply when: the agent system is stateless, single-turn, and each response is independently validated by a downstream evaluator with full ground-truth access — in that case, output consistency is both necessary and sufficient.

## Known Limits

- **Detection latency**: Drift-detection feedback loops add latency per step. In low-latency, high-throughput pipelines, embedding per-step verification is expensive. Batch verification at phase boundaries is an acceptable tradeoff, but increases the window during which drift compounds undetected.

- **Harness fidelity as ceiling**: The harness enforces direction only as well as its encoded objectives match the actual intended goal. If the goal is underspecified or changes mid-task, the harness will enforce the wrong direction. Direction maintenance presupposes that the intended direction is captured in the harness contract.

- **Drift vs. legitimate adaptation**: Not all behavioral deviation is drift. In exploratory or research agent contexts, deviation from the initial plan may represent valid learning. Harnesses must distinguish between goal-directed adaptation and directionless drift — a distinction that requires semantic evaluation, not format checking.

- **Not applicable to purely generative, open-ended tasks**: In creative generation tasks with no fixed objective, direction maintenance is undefined. This pattern applies to goal-directed agent systems with measurable completion criteria.

## Promotion History

Candidate — created 2026-05-19 from user insight (source: https://www.minwoo.cloud/blog/harness-direction-over-consistency).
Confirmed — 2026-05-19, scored 20/25 (user decision).

## Properties to Restore

위반이 감지됐을 때, 수정된 결과가 만족해야 할 속성들이다.
구체적인 구현 방법은 컨텍스트에 따라 달라질 수 있다.

| Violation detected | Properties the fix must satisfy |
|--------------------|--------------------------------|
| Format-consistent but semantically drifting outputs go undetected | Drift must be detectable before the next delegation cycle begins; detection must operate on semantic/behavioral signals, not on output format |
| No feedback loop exists for directional correction | A correction mechanism must be present that targets the agent's trajectory relative to the original goal, not the surface structure of its outputs |
| Format consistency is accepted as evidence of correct direction | Format conformance must be explicitly decoupled from directional conformance; a passing format check must not satisfy a direction check |
| Drift compounds across multiple delegation steps before being caught | The harness must be capable of interrupting the delegation chain at the step where drift first becomes detectable, without requiring full task completion |
