# Phase vs Lane — Serial vs Parallel Execution

Status: confirmed
Last reviewed: 2026-05-18

## Principle

In any multi-agent execution system, dispatch topology is derived from the dependency structure of the task graph — not from implementation defaults.

A **phase** (serial dispatch) is a set of task nodes where at least one node consumes the output of a predecessor; no node enters the ready queue until its predecessor emits. A **lane** (parallel dispatch) is a set of task nodes sharing no dependency edges; independent nodes enter the ready queue simultaneously, execute concurrently, and merge at an explicit barrier node.

The dispatch decision follows three rules:
1. Task B depends on task A's output → phase; B is blocked until A emits.
2. Task B has no dependency on task A → separate lanes; both dispatch immediately.
3. Resource contention exists between A and B (shared write targets, rate-limited endpoints, exclusive locks) → treat as a hidden dependency edge; serialize or add a coordination primitive.

The dependency graph is constructed at plan time and encoded in the execution plan artifact. Leaving allocation to runtime inference is an architectural defect. The scheduler-theoretic literature classifies this as a structural property of the DAG, not an inference-time output (arXiv 2604.11378).

## Evidence

### Academic Papers

- **DynTaskMAS: Dynamic Task Graph-driven Framework for Asynchronous and Parallel LLM-based Multi-Agent Systems** (arXiv 2503.07675, ICAPS 2025) — Decomposes tasks into DAGs; independent nodes execute as parallel lanes, dependent nodes as serial phases. Results: 21–33% reduction in execution time, 35.4% improvement in resource utilization vs sequential baselines, near-linear throughput scaling to 16 concurrent agents.
- **AdaptOrch: Task-Adaptive Multi-Agent Orchestration** (arXiv 2602.16873, Feb 2026) — A Topology Routing Algorithm selects topology (parallel, sequential, hierarchical, or hybrid) from structural DAG properties: parallelism width, critical path depth, and inter-task coupling. Results: 12–23% improvement over static single-topology baselines.
- **Flash-Searcher: Fast and Effective Web Agents via DAG-Based Parallel Execution** (arXiv 2509.25301, 2025) — Replaces sequential chains with DAGs; independent reasoning paths execute concurrently while logical ordering constraints are preserved as DAG edges.
- **Verified Multi-Agent Orchestration: Plan-Execute-Verify-Replan** (arXiv 2603.11445, ICLR 2026 Workshop) — Independent sub-questions run as parallel lanes; dependent sub-questions form serial phases. Answer completeness improves from 3.1 → 4.2 on a 1–5 scale vs single-agent.
- **A Scheduler-Theoretic Framework for LLM Agent Execution** (arXiv 2604.11378, Apr 2026) — Classifies parallel/serial dispatch as a structural property of an explicit DAG: "if nodes A and B have no dependency edge, they are dispatched in parallel regardless of what the LLM would infer."

### Official Runtime Documentation

- **Anthropic "How we built our multi-agent research system"** (anthropic.com/engineering/multi-agent-research-system) — Dispatches parallel worker agents for independent subtasks; sequences only where result dependencies require. Reports ~90% performance improvement from parallelization.
- **Anthropic "When to use multi-agent systems"** — Use parallel subagents when tasks are structurally isolated; use sequential execution when downstream agent inputs depend on upstream agent outputs.
- **OpenAI "Orchestration and handoffs"** (developers.openai.com/api/docs/guides/agents/orchestration) — Defines two fundamental orchestration primitives: agent-as-tool (parallel dispatch) and handoff (sequential transfer of control) — directly corresponding to lanes and phases.

### Terminology Note

"Phase vs Lane" is not a named term in the literature. The concept appears as "topology-adaptive orchestration" (AdaptOrch), "DAG-based execution" (Flash-Searcher, DynTaskMAS, VMAO), and "scheduler-theoretic framework" (arXiv 2604.11378). The principle — dependency graph analysis determines serial vs parallel dispatch — is an explicitly stated mechanism in all major 2025–2026 orchestration papers.

## When to Apply

| Signal | Decision | What breaks if ignored |
|--------|----------|------------------------|
| Two or more tasks exist with no dependency edges between them | Assign to separate lanes; dispatch in parallel | Unnecessary serialization wastes throughput proportional to the number of parallelizable task nodes |
| Pipeline throughput is lower than the sum of individual task durations | Audit for incorrectly serialized independent tasks; reassign to lanes | The throughput gap widens proportional to the degree of incorrect serialization; execution time exceeds the critical path duration |
| Parallel execution produces incorrect or inconsistent results | Audit for unencoded dependency edges; convert affected lanes to a phase | Race conditions and inconsistent state — lanes write conflicting outputs because the dependency constraint existed but was not encoded |
| Tasks share a write target, rate-limited resource, or exclusive lock | Serialize the affected tasks or add an explicit coordination primitive before dispatching as a lane | Execution failure at runtime when two lanes contend for the same resource; the contention was invisible in the task graph |

## Known Limits

- **Implicit dependencies:** Dependency analysis does not automatically detect shared mutable state, external API rate limits, third-party ordering guarantees, or write-after-read conflicts on shared storage. These require manual identification and explicit encoding as dependency edges or coordination primitives.
- **Replanning invalidates coordination primitives**: When a pipeline is replanned mid-execution — for example, upgrading a phase to a lane to improve throughput — coordination primitives declared for the original topology (locks, barriers, serialization points) may no longer match the new execution graph. The contention requirement that justified the original serialization is not automatically visible at replanning time. Audit coordination primitives whenever topology is changed dynamically.
- **Implicit synchronization barriers:** Barriers between lanes must be declared explicitly in the execution plan. Implicit barrier assumptions produce timing bugs that surface only under concurrency.
- **Inapplicable scope:** This pattern does not apply to single-agent, single-task pipelines. Parallelism requires at least two structurally independent task nodes. Applying the pattern to a single-node graph produces no benefit and adds unnecessary plan complexity.

## Promotion History

Candidate — extracted from agent session analysis, 2026-05-18.
Promotion-ready — strongest academic backing across 7 candidates (5 papers, 2025–2026) plus Anthropic and OpenAI official documentation. Pattern is consistently named ("DAG-based execution," "topology-adaptive orchestration") and empirically validated.

## Properties to Restore

위반이 감지됐을 때, 수정된 결과가 만족해야 할 속성들이다.
구체적인 구현 방법은 컨텍스트에 따라 달라질 수 있다.

| Violation detected | Properties the fix must satisfy |
|--------------------|--------------------------------|
| Independent tasks serialized → throughput loss | Dispatch topology must reflect the dependency structure of the task graph; tasks with no shared state or dependency edges must be dispatchable concurrently |
| Dependent tasks parallelized → race conditions or non-deterministic results | Tasks where the output of one is consumed as the input of another must be ordered such that the producer completes before the consumer is dispatched |
| Resource contention between concurrent tasks causes runtime failure | Any shared write target, exclusive lock, or rate-limited resource must be encoded as a dependency edge or covered by an explicit coordination primitive before the affected tasks are dispatched |
