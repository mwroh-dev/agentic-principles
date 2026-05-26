# Failure Logging Anti-Survivorship Bias

Status: confirmed
Last reviewed: 2026-05-19

## Principle

Record every execution attempt — including failures, partial completions, and abandoned paths — not only successful runs. A **failure trajectory** is the complete step-by-step execution log of a run that did not meet its acceptance criteria. **Survivorship bias** in agent observability occurs when only successful trajectories are retained, making the system's actual failure mode distribution invisible.

When only successes are logged, the inter-step decision state that preceded failure is lost. This prevents root-cause attribution: teams cannot determine which agent caused the failure, at which step, or why a specific assumption collapsed mid-run. In multi-agent systems, failures cascade and mask their origin — a downstream subagent's error presents as a final-stage failure even when the root cause was an upstream specification defect. Without full failure trajectories, this propagation path is unrecoverable.

Success records show which path worked under which conditions. Failure records expose the boundary conditions under which those paths break. Reliable agent systems require both.

## Evidence

### Academic Papers

- **Why Do Multi-Agent LLM Systems Fail?** (arXiv:2503.13657, March 2025, UC Berkeley / Cornell) — Systematically annotated 1,600+ execution traces from seven frameworks, identifying 14 failure modes across system design, inter-agent misalignment, and task verification. Inter-annotator agreement κ = 0.88. Directly demonstrates that failure mode taxonomy requires retaining failed traces; success-only logs yield no failure taxonomy.

- **Which Agent Causes Task Failures and When?** (arXiv:2505.00212, April 2025) — Introduces automated failure attribution over 127 multi-agent systems with fine-grained failure log annotations. Best method achieves only 53.5% accuracy in identifying the failure-responsible agent and 14.2% in pinpointing the failure step — confirming that even with failure logs, attribution is hard; without them, it is impossible.

- **Where LLM Agents Fail and How They Can Learn From Failures** (arXiv:2509.25370, September 2025) — Introduces AgentErrorBench, the first dataset of systematically annotated failure trajectories from ALFWorld, GAIA, and WebShop. Learning from failure trajectories yields 24% higher all-correct accuracy and 17% higher step accuracy versus strongest baselines that did not use failure data. Directly validates that retained failure trajectories enable measurable improvement.

### Official Best Practices

- **AgentRx: Systematic Debugging for AI Agents** (Microsoft Research, 2026, https://www.microsoft.com/en-us/research/blog/systematic-debugging-for-ai-agents-introducing-the-agentrx-framework/) — AgentRx diagnoses failures from retained execution trajectories. Retaining failed trajectories produced +23.6% absolute improvement in failure localization accuracy and +22.9% improvement in root-cause attribution over prompting baselines on a 115-trajectory benchmark. "Traditional success metrics (like 'Did the task finish?') don't tell us enough. To build safe agents, we need to identify the exact moment a trajectory becomes unrecoverable."

- **AI Agent Logging and Observability: Production** (OpenHelm, https://www.openhelm.ai/blog/ai-agent-logging-observability-production) — Prescribes: "Log everything in development. In production, sample non-error traces if volume is high. Always log full traces for errors." Explicitly inverts success-first logging: failures receive full retention while successful runs can be sampled.

- **Agent Observability: The Complete Guide** (Braintrust, 2026, https://www.braintrust.dev/articles/agent-observability-complete-guide-2026) — Documents that "agent failures often stay invisible until a customer reports the issue" without full-trace observability. Prescribes converting production failure traces directly into evaluation cases to prevent regression — a practice that requires failure trace retention.

- **How We Built Our Multi-Agent Research System** (Anthropic Engineering, https://www.anthropic.com/engineering/multi-agent-research-system) — Notes that human testers "find edge cases that evals miss" including "hallucinations on unusual queries, system failures, or subtle source selection biases" — failure patterns invisible without dedicated failure recording. Confirms agent non-determinism makes post-hoc debugging dependent on retained execution context.

### Named in Literature?

No unified name. Closest named concepts: "publication bias" (academic literature), "negative result suppression" (ML reproducibility research), "failure trajectory annotation" (arXiv:2509.25370), and "failure attribution" (arXiv:2505.00212) — but the specific formulation — that success-only logging in agent systems creates a structural blind spot analogous to survivorship bias, eliminating the inter-step decision record needed for root-cause analysis — is implied across sources but not named as a single pattern.

## When to Apply

| Condition | Action |
|-----------|--------|
| Agent system runs in production with any real-world consequences | Retain full traces for all failed and erroring runs |
| Multi-agent pipeline where failures can cascade across agents | Log complete inter-agent message chains, including failed handoffs |
| Evaluation or benchmarking of agent capability | Include failed trajectories in dataset; exclude failures only if explicitly documenting a success-rate metric |
| Post-incident root-cause analysis | Require failure traces as prerequisite; treat absence of failure log as a blocker, not a gap to work around |
| Iteration on agent prompts, tools, or architecture | Compare failure mode distribution before and after change, not only success rate |

Do not apply when: the system has no persistence layer and storing traces introduces unacceptable latency or cost; in that case, implement asynchronous logging to a sidecar store and sample successful traces while retaining all failure traces.

## Known Limits

- **Log volume cost**: Full retention of failure traces in high-throughput production systems increases storage cost. Mitigation: sample successful traces at a configurable rate (e.g., 10–20%); never sample failure traces. This asymmetric retention preserves failure mode visibility while controlling cost.

- **Failure label ambiguity**: In multi-agent pipelines, labeling a run as a "failure" requires a defined acceptance criterion. Without one, a run that produced a partial output may be logged as success or failure depending on observer interpretation, polluting the failure dataset. Define acceptance criteria before deployment.

- **Non-determinism limits reproduction**: Agent non-determinism means that retaining a failure trajectory does not guarantee the failure is reproducible. Logged traces provide the closest available approximation to the original execution context, but exact reproduction is not guaranteed. Treat failure trace analysis as evidence of failure class, not as a deterministic replay.

- **Does not apply to pure stateless batch pipelines**: In single-step, stateless transformation tasks (e.g., a single LLM call to classify a document with no tool use, planning, or multi-turn memory), the standard success/failure log at the call level is sufficient. The cascading-failure and inter-step-decision-loss problems that motivate this pattern do not arise in single-step contexts.

## Promotion History

Candidate — created 2026-05-19 from user insight.
Confirmed — 2026-05-19, scored 19/25 (user decision).

## Properties to Restore

위반이 감지됐을 때, 수정된 결과가 만족해야 할 속성들이다.
구체적인 구현 방법은 컨텍스트에 따라 달라질 수 있다.

| Violation detected | Properties the fix must satisfy |
|--------------------|--------------------------------|
| Only successful execution traces retained | The corrected logging design must retain traces for all execution attempts — including failures, partial runs, and abandoned runs — without filtering by outcome |
| Failure distribution invisible due to success-only retention | The corrected retention policy must make the failure distribution queryable independently of the success distribution; the two must not be conflated in a single aggregated record |
| Root cause unattributable because inter-step failure state was discarded | Each retained failure trace must capture sufficient inter-step decision state that the originating agent, step, and condition can be identified through log analysis alone |
| Failure traces not convertible to evaluation cases | The corrected failure records must be structured such that any retained failure trace can be used as an input to an evaluation or regression test without requiring access to the live system |
