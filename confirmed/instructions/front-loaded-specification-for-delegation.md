# Front-Loaded Specification For Delegation

Status: confirmed
Last reviewed: 2026-05-19

## Principle

When a task is delegated to an AI executor (subagent, LLM, or automated pipeline), the specification burden on the orchestrator increases relative to direct human execution. Specification completeness required for delegation is higher than for direct execution because an AI executor fills unspecified gaps with plausible-but-unverified content rather than surfacing ambiguity.

The causal chain: underspecification → gap-filling → output that appears valid → correction cost after delivery. Correction cost scales with delegation depth: the further downstream a gap is discovered, the more rework it generates.

Implementation quality is therefore determined by the number and precision of decisions made upfront, not by the number of corrections applied afterward. Repeated correction loops signal specification failure, not implementation failure. When an orchestrator delegates without exhausting specification gaps, the executor's output optimizes for plausibility, not correctness.

Define before delegating: (1) the exact acceptance condition, (2) what is out of scope, (3) which ambiguities should block rather than be resolved autonomously. Each unresolved ambiguity at delegation time becomes an autonomous decision made by the executor under uncertainty.

## Evidence

### Academic Papers

- **Specifications: The Missing Link to Making LLM Systems an Engineering Discipline** (arXiv:2412.05299, December 2024, multi-institution) — Argues that natural language prompts encourage underspecification, and that underspecified components produce outputs that are impossible to verify: "If a task is not well-specified, how can one even determine whether the task is being performed correctly?" Documents real-world failures (Air Canada chatbot fabricating refund policies; Chevrolet chatbot offering a car at $1) as direct downstream consequences of specification gaps at delegation time.

- **Compass vs. Railway Tracks: Unpacking User Mental Models for Communicating Long-Horizon Work to Humans vs. AI** (arXiv:2601.11848, January 2026, multi-institution) — Empirically finds that users create exhaustive "railway tracks" specifications for AI — rigid, direct, and highly detailed — in contrast to high-level "compass" guidance given to humans. Root cause: users perceive that AI cannot infer intent, prioritize autonomously, or make judgment calls, so they front-load detail to prevent deviation. Validates that delegation to AI increases the specification burden relative to human delegation.

- **Intelligent AI Delegation** (arXiv:2602.11865, February 2026) — Proposes an adaptive framework for AI delegation in which "clarity of intent" and "clear specifications regarding roles and boundaries" are listed as explicit prerequisites for successful delegation. Without these, delegation frameworks fail to adapt to environmental changes and cannot handle unexpected failures robustly.

### Official Best Practices

- **Best Practices for Claude Code** (https://code.claude.com/docs/en/best-practices, Anthropic, 2025–2026) — Prescribes: "The more precise your instructions, the fewer corrections you'll need." Documents a concrete failure pattern: correcting Claude more than twice on the same issue in one session signals that the initial prompt was underspecified. Recommends having the model interview the user before implementation on complex features to surface hidden specification gaps: "Once the spec is complete, start a fresh session to execute it." Also states: "Without clear success criteria, [the model] might produce something that looks right but actually doesn't work."

- **Specifications: The Missing Link (arXiv:2412.05299 — Position)** (https://arxiv.org/html/2412.05299v2, 2024) — States directly: "the key building block enabling [modular, reliable systems] is specification: the ability to describe what each component should do and how to verify its outputs." Frames specification completeness as the prerequisite for debugging, modularity, and trustworthy composition of LLM components — the infrastructure missing from current practice.

### Named in Literature?

No unified name. Closest named concepts: "specification completeness," "upfront requirement elicitation," "prompt specificity," and "railway tracks" (arXiv:2601.11848). The specific formulation — that delegating to an AI executor *increases* specification burden compared to direct execution, and that quality is set at delegation time rather than correction time — is implied across sources but not named as a single pattern.

## When to Apply

| Condition | Action |
|-----------|--------|
| Task is delegated to a subagent or automated pipeline | Enumerate acceptance criteria before issuing the task |
| Delegation depth > 1 hop (orchestrator → agent → tool) | Specify scope boundaries explicitly at each handoff |
| The executor cannot interrupt for clarification (non-interactive mode, batch job, CI pipeline) | Resolve all known ambiguities in the initial instruction; treat unresolved ambiguity as a blocking issue |
| Output will not be manually reviewed before use | Define a machine-verifiable acceptance condition in the task specification |
| Task involves judgment calls (prioritization, style, edge case handling) | Enumerate the judgment criteria explicitly; do not leave them to executor discretion |

Do not apply when: the executor has full bidirectional clarification capability (e.g., an interactive human conversation partner with no marginal cost to ask follow-up questions). In that context, the executor can surface gaps before acting, and front-loading every detail adds overhead without reducing error rate.

## Known Limits

- **Specification completeness is bounded by the orchestrator's ability to anticipate gaps**: An orchestrator cannot specify what it does not know is ambiguous. For novel or exploratory tasks, a structured clarification phase (the executor interviews the orchestrator) is required before the specification is considered complete. Front-loading alone is insufficient when the problem space is unknown.

- **Overly rigid specifications suppress valid executor judgment**: Exhaustive "railway tracks" specifications (arXiv:2601.11848) prevent the executor from applying domain knowledge to edge cases not anticipated by the orchestrator. Specification must define acceptance criteria and out-of-scope boundaries, but not necessarily implementation path. Specifying the *what* is required; specifying the *how* is often counterproductive.

- **Correction loops are a lagging signal**: By the time repeated corrections surface a specification gap, the executor's context is already polluted with failed attempts. Correction loops do not recover specification quality — they compound it. The signal that a specification was incomplete arrives after the cost is already incurred.

- **This pattern does not apply in real-time collaborative execution**: When the orchestrator and executor operate in tight interactive loops (human pair-programming, live chat), gaps are resolved dynamically at near-zero cost. Front-loading full specification in this context is unnecessary overhead. The pattern is load-bearing only when the executor operates autonomously between handoffs.

## Promotion History

Candidate — created 2026-05-19 from user insight (source: https://www.minwoo.cloud/blog/growth-team-mindset, scored 21/25).
Confirmed — promoted 2026-05-19 by user decision (score 21/25).
