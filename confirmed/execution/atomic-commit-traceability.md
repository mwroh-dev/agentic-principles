# Atomic Commit Traceability

Status: confirmed
Last reviewed: 2026-05-19

## Principle

An **atomic task unit** is the smallest independently verifiable and reversible step in an execution sequence. Decompose every multi-step workflow into atomic task units — each producing one observable state change that can be verified before the next unit begins.

Atomic decomposition makes failure localization tractable: when the workflow breaks, the system performs a binary search over the execution trace (analogous to `git bisect`) to identify the first unit that violated its contract, then re-enters execution at the last verified checkpoint.

Workflows that skip atomization accumulate silent errors across steps. A failure observed at step N may have originated at step K (K < N), but without per-step verification checkpoints, the origin is unrecoverable. The only available action becomes full restart. Atomic decomposition converts full restarts into targeted rollbacks.

Rule: if a step cannot be independently verified and reversed, it is not atomic — split it further before execution.

## Evidence

### Academic Papers

- **Automating Complex Document Workflows via Stepwise and Rollback-Enabled Operation Orchestration** (arXiv 2512.04445, December 2025, accepted AAAI-2026) — Introduces AutoDW, which decomposes long-horizon workflows into atomic API-level operations, verifying and potentially rolling back each before proceeding. Achieves 90% instruction-level completion and outperforms non-atomic baselines by 40% (instruction) and 76% (session level).

- **Sherlock: Reliable and Efficient Agentic Workflow Execution** (arXiv 2511.00330, November 2025) — Identifies error-prone nodes in multi-step agent workflows and attaches verifiers selectively at those nodes; speculatively executes downstream steps and rolls back to the last verified output on failure. Delivers 18.3% accuracy improvement and 48.7% latency reduction over non-speculative execution.

- **A Trace-Based Assurance Framework for Agentic AI Orchestration: Contracts, Testing, and Governance** (arXiv 2603.18096, March 2026) — Instruments agent executions as Message-Action Traces (MAT) with step-level and trace-level contracts that produce machine-checkable verdicts, localizing the first violating step in long-horizon interactions without requiring full re-execution.

- **Advancing Agentic Systems: Dynamic Task Decomposition, Tool Integration and Evaluation using Novel Metrics and Dataset** (arXiv 2410.22457, October 2024) — Demonstrates that asynchronous, granular task graph decomposition significantly enhances responsiveness and scalability; introduces Node F1 Score and Structural Similarity Index as metrics for evaluating decomposition quality.

### Official Best Practices

- **Building Effective Agents** (https://www.anthropic.com/research/building-effective-agents, Anthropic, 2024) — Prescribes that agents require "ground truth from the environment at each step (such as tool call results or code execution) to assess progress," and that complex tasks perform better when "each consideration is handled by a separate LLM call, allowing focused attention." Directly instantiates per-step verification as a core design requirement.

### Named in Literature?

No unified name. Closest named concepts: *stepwise execution* (AutoDW), *selective rollback* (Sherlock), *step contracts* (Trace-Based Assurance Framework), and *commit-level granularity* (software engineering). The specific formulation — decompose to the smallest reversible unit specifically to enable binary-search failure localization across an execution trace — is implied across sources but not named as a single pattern.

## When to Apply

| Condition | Apply atomic decomposition |
|-----------|--------------------------|
| Workflow has ≥3 sequential steps with cumulative state | Yes — each step is a rollback boundary |
| Any step has side effects (writes, API calls, external state) | Yes — side effects require per-step verification before proceeding |
| Failure at step N cannot be diagnosed without knowing state at step N-1 | Yes — insert a checkpoint between steps |
| Task duration exceeds one LLM context window | Yes — atomic units are the only recoverable re-entry points |
| Steps are trivially reversible and stateless | No — overhead of checkpointing exceeds benefit |

Do not apply when: the entire operation is a single irreversible action with no intermediate state (e.g., a single database transaction managed by the DB engine's own ACID guarantees) — atomicity is already enforced at a lower layer.

## Known Limits

- **Granularity calibration cost**: Determining the correct atomic boundary requires upfront analysis of state dependencies. Over-decomposition (units too small) increases checkpoint overhead and context-switching cost between steps. Under-decomposition (units too large) defeats the purpose. No automated heuristic currently generalizes across domains.

- **Rollback is not always semantically reversible**: Atomic decomposition enables structural rollback (re-entering at a checkpoint), but side effects that have already propagated to external systems (sent emails, published events, debited accounts) are not reversed by rolling back the agent's internal state. Atomic decomposition must be paired with idempotent or compensating operations for external side effects.

- **Binary-search bisection requires monotonic failure**: The git-bisect analogy holds only when the workflow has a monotonic failure boundary — i.e., the system was correct up to step K and incorrect from step K+1. Non-monotonic failures (intermittent errors, race conditions, environment flakiness) require different diagnostic strategies; atomic decomposition alone does not solve them.

- **Does not apply to fully stateless, embarrassingly parallel workloads**: Workflows where every unit is independent and has no cumulative state (e.g., batch inference over independent inputs) gain no traceability benefit from atomic decomposition; each unit already fails in isolation without contaminating others.

## Promotion History

Candidate — created 2026-05-19 from user insight.
Confirmed — promoted 2026-05-19. Score: 19/25 (user decision).

## Properties to Restore

위반이 감지됐을 때, 수정된 결과가 만족해야 할 속성들이다.
구체적인 구현 방법은 컨텍스트에 따라 달라질 수 있다.

| Violation detected | Properties the fix must satisfy |
|--------------------|--------------------------------|
| Multi-step work committed as a single unit with no intermediate verification | Each atomic unit must be independently verifiable before the next unit begins; a failure at any step must not require the full sequence to be restarted |
| Failure localization requires full restart | It must be possible to isolate the first failing unit via binary search over the execution trace, without replaying steps that have already been verified |
| No per-step verification exists | Every unit in the sequence must produce an observable state change that can be verified before the next unit is entered; unverified steps must block forward progress |
| Rollback to a prior verified state requires replaying the full sequence | The system must be able to re-enter execution at any prior verified checkpoint without re-executing steps that preceded it |
