# Plan as Control Structure

Status: confirmed
Last reviewed: 2026-05-19

## Principle

A **plan** is a high-level definition of direction and checkpoint conditions — not a sequential execution directive. A **control structure** is a mechanism that detects divergence between expected and actual state and triggers corrective action.

Do not pass a plan directly to a subagent as an implementation script. Instead, decompose the plan into a **priority task queue**: an ordered, dynamically updated list of executable units. The orchestrator advances the queue step by step, comparing actual state at each checkpoint against the plan's expected conditions. When actual state diverges, the orchestrator replans — it does not continue executing the original sequence.

The plan's function is deviation detection, not execution instruction. A plan that is not being checked against runtime state is inert. A plan that drives execution without checkpoint verification is a rigid script — it cannot recover from unexpected outcomes and accumulates undetected drift.

Treat each checkpoint as a gate: if the gate condition is not satisfied, halt and replan before proceeding. Priority task queues must be updated dynamically as execution produces new information.

## Evidence

### Academic Papers

- **ReAct: Synergizing Reasoning and Acting in Language Models** (arXiv:2210.03629, Oct 2022, Google Brain / Princeton) — Introduces interleaved reasoning traces and actions, where reasoning traces "help the model induce, track, and update action plans as well as handle" deviations from expected execution paths. This formalizes plan monitoring as an integral part of every execution step, not a post-hoc review.

- **DoReMi: Grounding Language Model by Detecting and Recovering from Plan-Execution Misalignment** (arXiv:2307.00329, Jul 2023, IROS 2024) — Proposes using LLM-generated constraints as plan specifications and a vision-language model as a continuous constraint monitor; violations immediately trigger replanning. Directly demonstrates that separating plan-as-specification from plan-as-execution-sequence yields higher task success rates and shorter completion times.

- **Verified Multi-Agent Orchestration: A Plan-Execute-Verify-Replan Framework** (arXiv:2603.11445, 2026) — Formalizes the Plan-Execute-Verify-Replan cycle as the core orchestration loop; verification drives adaptive replanning at each sub-task boundary. Improved answer completeness from 3.1 to 4.2 and source quality from 2.6 to 4.1 (1–5 scale) versus single-agent baselines on complex queries.

- **LLM-Based Generalizable Hierarchical Task Planning with Event-Driven Replanning** (arXiv:2511.22354, Nov 2025) — CoMuRoS architecture separates centralized task management (plan) from decentralized execution; a Task Manager LLM reclassifies task events and triggers replanning when "task failures or user intent changes" are detected. Achieved 9/10 success rate on collaborative recovery and 0.91 task correctness across multiple LLMs.

### Official Best Practices

- **Building Effective AI Agents** (https://www.anthropic.com/research/building-effective-agents, Anthropic, 2024) — Prescribes that orchestrators "dynamically break down tasks" rather than predetermining steps, and that agents "pause for human feedback at checkpoints or when encountering blockers." Directly instantiates plan-as-control-structure by treating checkpoints as execution gates and requiring environmental feedback at each step.

- **Effective Harnesses for Long-Running Agents** (https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents, Anthropic Engineering) — Demonstrates a concrete implementation: a continuously updated feature list with priority ordering, `claude-progress.txt` as runtime state log, and git commits as checkpoint recovery points. Agents read the plan at session start, select the highest-priority incomplete item, and self-verify before advancing — operationalizing the priority task queue pattern.

### Named in Literature?

No unified name. Closest named concepts: "Plan-Execute-Verify-Replan" (VMAO, arXiv:2603.11445), "event-driven replanning" (CoMuRoS, arXiv:2511.22354), and "plan-execution misalignment detection" (DoReMi, arXiv:2307.00329), but the specific formulation — using the plan as a deviation-detection control structure with dynamic priority task queue decomposition — is implied across sources and not named as a single pattern.

## When to Apply

| Condition | Apply? |
|---|---|
| Agent executes multi-step tasks spanning multiple turns or sessions | Yes |
| Execution state can diverge from plan expectations (external dependencies, failures, new information) | Yes |
| Plan has 3+ sequential steps with state dependencies between them | Yes |
| Subagent receives a plan and is expected to self-direct execution | Yes |
| Task is a single-turn, stateless call with no sequential dependencies | No |

Apply when:
1. An orchestrator dispatches a plan to one or more subagents, and the plan contains steps whose validity depends on prior steps completing successfully.
2. Execution involves external state (APIs, file system, user input, other agents) that can produce unexpected outcomes.
3. The system must recover gracefully from partial failures without restarting the full task.

Do not apply when: the task is a single deterministic function call with no branching and no external state. In that case, the plan and the execution are identical; introducing checkpoints adds overhead with no recovery benefit.

## Known Limits

- **Checkpoint design requires domain knowledge**: Checkpoints must be defined before execution begins. Vague checkpoints ("make progress on X") do not enable divergence detection — they must specify measurable expected state (e.g., "feature Y passes test Z"). Poorly designed checkpoints degrade the pattern to a script with pauses.

- **Dynamic queue updates require orchestrator availability**: Replanning requires the orchestrator to be reachable when a checkpoint fails. In fully autonomous, offline, or long-running subagent loops where the orchestrator is not polled, this pattern cannot trigger timely correction.

- **Priority ordering is a heuristic, not a guarantee**: Decomposing a plan into a priority task queue assumes tasks can be meaningfully ordered and that order is stable enough to re-evaluate at runtime. Tasks with strong mutual dependencies or emergent ordering requirements may require more sophisticated DAG-based scheduling than a linear priority queue provides.

- **Does not apply to purely reactive, event-driven systems**: In systems where each action is fully determined by the current event with no multi-step planning state (e.g., stateless request handlers, simple rule-based routers), this pattern is inapplicable. Those systems have no plan to maintain and no deviation to detect.

## Promotion History

Candidate — created 2026-05-19 from user insight (source: https://www.minwoo.cloud/blog/growth-team-mindset).
Confirmed — promoted 2026-05-19, scored 19/25 by user decision.

## Properties to Restore

위반이 감지됐을 때, 수정된 결과가 만족해야 할 속성들이다.
구체적인 구현 방법은 컨텍스트에 따라 달라질 수 있다.

| Violation detected | Properties the fix must satisfy |
|--------------------|--------------------------------|
| Plan treated as execution script without checkpoints → drift accumulates undetected | Actual runtime state must be compared against the plan's expected conditions at each defined checkpoint before the next step is permitted to execute |
| Plan not compared against runtime state → no divergence detection | When actual state diverges from plan expectations, execution must halt and replanning must occur before continuation; continuation without replanning is not a valid recovery path |
| Failure point cannot be isolated | Plan structure must allow any failing step to be identified without re-executing the entire sequence; checkpoints must partition the execution such that bisection of failure is possible |
