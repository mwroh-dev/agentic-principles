# Evaluation Before Implementation

Status: confirmed
Last reviewed: 2026-05-19

## Principle

Define what success and failure look like before writing a single line of agent implementation. This is not a quality-assurance step appended at the end — it is the first design decision.

When evaluation criteria are absent at design time, three things converge on implementer convenience: artifact boundaries (what each agent produces and consumes), agent scope (where one agent's responsibility ends and another begins), and skill surface (which tools and capabilities each agent exposes). These decisions become implicit and unrecoverable without rewriting the system.

Evaluation at this level is not output accuracy checking. It is correctness verification of the full orchestration procedure: did the right agent act, in the right order, on the right artifact, under the right authorization, producing the right observable trace? A result that appears correct but lacks policy evidence in the execution trace is a failure.

If a requirement cannot be expressed as a measurable evaluation criterion before implementation begins, the requirement is underspecified. Do not proceed until it can be.

This pattern is a prerequisite for `evaluation-as-behavioral-specification`: you must first establish that evaluation comes before implementation before specifying what a complete eval case must contain.

## Evidence

### Academic Papers

- **Evaluation-Driven Development and Operations of LLM Agents: A Process Model and Reference Architecture (EDDOps)** (arXiv:2411.13768, Nov 2024/2025, multiple institutions) — Proposes evaluation as the explicit starting point of the agent development lifecycle, not a trailing phase. Demonstrates empirically that without upfront evaluation criteria, agent pipeline design, artifact definitions, and LLM boundary placement converge on implementer defaults rather than system requirements.
- **Test-Driven AI Agent Definition (TDAD): Compiling Tool-Using Agents from Behavioral Specifications** (arXiv:2603.08806, 2026) — Operationalizes "evaluation before implementation" as a compiler pipeline: behavioral specification → test suite → prompt synthesis. The test cases (evaluation criteria) are written before the agent exists; the agent is generated to satisfy them. Directly demonstrates that evaluation ordering determines what the agent becomes.
- **Toward Architecture-Aware Evaluation Metrics for LLM Agents** (arXiv:2601.19583, 2026, CAIN '26) — Establishes that evaluation metrics must be anchored to specific architectural components (memory, planning, tool-use, inter-agent communication). Without this linkage defined upfront, architectural boundaries cannot be set — the architecture has no correctness criterion to satisfy.
- **Agent Behavioral Contracts** (arXiv:2602.22302, Feb 2026) — Applies Design-by-Contract to AI agents: Preconditions, Invariants, Governance policies, and Recovery mechanisms are declared before implementation. Across 1,980 sessions, contracted agents detected 5.2–6.8 soft violations per session that uncontracted baselines missed entirely, with 88–100% hard constraint compliance.
- **Formally Specifying the High-Level Behavior of LLM-Based Agents** (arXiv:2310.08535, Oct 2023) — Shows that high-level declarative behavioral specification written by users before agent implementation automatically constrains the agent's decoding monitor, preventing behavioral drift. Spec-first ordering is load-bearing: post-hoc specification cannot reconstruct the same constraints.

### Official Best Practices

- **Anthropic "Demystifying Evals for AI Agents"** (https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents, Jan 2026) — States directly: "Writing evals is useful at any stage in the agent lifecycle. Early on, evals force product teams to specify what success means." Notes that when two engineers interpret the same spec differently, an upfront eval suite resolves the ambiguity — while absence of evals leaves it unresolved until production failure. Also observes that agent evaluation targets "the harness *and* the model working together," meaning evaluation criteria determine harness architecture, not only model behavior.
- **OpenAI Evals Framework** (https://github.com/openai/evals) — Structured as a registry of evaluation criteria defined independently of any specific model or agent implementation, establishing the convention that evaluation criteria are reusable, version-controlled artifacts that precede and outlive individual implementations.

### Named in Literature?

No unified name. Closest named concepts: "evaluation-driven development" (EDDOps, arXiv:2411.13768), "test-driven agent definition" (TDAD, arXiv:2603.08806), "spec-first agent design" (arXiv:2310.08535), and "design-by-contract for agents" (ABC, arXiv:2602.22302). The specific formulation — that evaluation ordering is the primary determinant of agent architecture, artifact placement, and skill surface — is implied across all sources but not consolidated under a single pattern name.

## When to Apply

Apply when:
1. Starting the design of a new agent, sub-agent, or multi-agent pipeline — before defining which tools each agent gets, which artifacts it reads or writes, or how responsibilities are divided between agents.
2. Adding a new capability to an existing agent system where the current eval suite does not cover the new behavior — the eval must be written and reviewed before the capability is implemented.
3. A requirement has been stated in terms of desired output quality but cannot yet be expressed as a pass/fail criterion against observable agent behavior — the requirement must be made measurable before implementation proceeds.
4. An existing agent is being refactored or re-scoped — re-derive evaluation criteria from the new scope first; do not inherit the old eval suite by default.

Do not apply when: the agent is an exploratory prototype with no expected repeatability, or when the evaluation infrastructure itself is being built and a bootstrap implementation is necessary to define what traces look like.

## Known Limits

- **Bootstrap paradox**: To write evaluation criteria for trace-level correctness, the team needs to know what a trace looks like. In greenfield systems this requires a skeleton implementation before full eval specification is possible. Resolve by writing output-level criteria first, then upgrading to trace-level criteria once the first trace exists.
- **Evaluation drift**: Evaluation criteria written before implementation become stale as the system evolves. Without a process to update evals alongside implementation changes, the ordering benefit erodes. Treat eval criteria as first-class versioned artifacts with the same change-management discipline as code.
- **Premature closure**: Writing evaluation criteria too early can lock in a narrow solution space before the design has been adequately explored. Apply after enough scoping to know what correctness means, but before implementation commits begin.

## Promotion History

Candidate — created 2026-05-19 from user insight: evaluation criteria must precede implementation in agent system design; absence causes artifact models, agent boundaries, and skill surfaces to converge on implementer convenience.
Confirmed — promoted 2026-05-19, score 21/25 (코어성 5, 리스크감소 4, 확장성 5, 제어성 4, 기록성 3).

## Properties to Restore

위반이 감지됐을 때, 수정된 결과가 만족해야 할 속성들이다.
구체적인 구현 방법은 컨텍스트에 따라 달라질 수 있다.

| Violation detected | Properties the fix must satisfy |
|--------------------|--------------------------------|
| Implementation began before eval criteria were defined | Measurable eval criteria must exist and be reviewed before any implementation commits begin; criteria written after implementation must not be shaped by the implementation they evaluate |
| Requirements are present but not expressible as measurable criteria | Every requirement driving the implementation must be expressible as a pass/fail criterion against observable agent behavior; unmeasurable requirements must block implementation until made measurable |
| Artifact boundaries or agent scope determined by implementer convenience | Artifact boundaries, agent scope, and skill surface must derive from what the eval criteria require to be observable — not from what is easiest to implement |
