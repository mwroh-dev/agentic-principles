# Lessons As Anti-Pattern Registry

Status: confirmed
Last reviewed: 2026-05-19

## Principle

Maintain a dedicated, append-only failure log — separate from session history — that stores concise, extracted lessons from each task failure. Do not rely on raw session logs as the primary failure-learning mechanism.

Session logs grow too long to read in full and contain redundant noise (tool outputs, intermediate reasoning, retried steps) that dilutes the signal. When an agent or system re-ingests raw logs, it suffers from "summarization drift": repeated compression passes silently discard low-frequency details that are critical for catching edge cases.

A failure log functions as an anti-pattern registry: each entry records a failure condition, its root cause, and the corrective rule. Success records show direction; failure records expose the specific conditions, vulnerabilities, and recurring patterns that success records conceal. Extract discriminative rules by contrasting failed and succeeded trajectories, then store only those rules — not the trajectories themselves.

Feed the failure log as a prefix to subsequent agent runs. The agent reads compact lessons, not thousands of tokens of history. If the log entry does not fit in a single declarative sentence, it is too long.

Violating this principle causes: repeated mistakes across sessions, context rot from over-long histories, and failure modes that remain invisible because they are buried inside verbose logs no agent (or human) reads end-to-end.

## Evidence

### Academic Papers

- **Reflexion: Language Agents with Verbal Reinforcement Learning** (arXiv:2303.11366, March 2023, MIT / Northeastern University) — Agents that store natural-language post-mortems of each failure in an episodic buffer achieve 91% pass@1 on HumanEval versus 80% for the GPT-4 baseline, with no weight updates. Validates that a dedicated failure-note store, fed as a prefix on the next attempt, outperforms generic session replay.

- **Where LLM Agents Fail and How They Can Learn From Failures** (arXiv:2509.25370, September 2025, UIUC) — Introduces AgentErrorTaxonomy (memory / reflection / planning / action / system) and AgentErrorBench, the first systematically annotated failure dataset across ALFWorld, GAIA, and WebShop. Structured failure registries with root-cause isolation yield 24% higher all-correct accuracy and up to 26% relative improvement in task success over unstructured baselines.

- **Memory for Autonomous LLM Agents: Mechanisms, Evaluation, and Emerging Frontiers** (arXiv:2603.07670, 2026, Hong Kong Research Institute of Technology) — Documents ExpeL, which extracts discriminative success/failure "rules of thumb" from trajectory comparisons and stores them as reusable heuristics rather than raw logs. Identifies "summarization drift" as the failure mode of raw-log compression: each pass silently discards low-frequency details critical for edge cases. History-based deletion of frequently-retrieved but low-utility memories further improves performance.

- **ACON: Optimizing Context Compression for Long-Horizon LLM Agents** (arXiv:2510.00615, 2025, Microsoft Research) — Demonstrates that transforming verbose session histories into compact lesson-units preserves over 95% of task accuracy on AppWorld and OfficeBench while remaining within practical token budgets. Distilled compact lessons outperform naive session truncation and repeated full-history summarization.

### Official Best Practices

- **Effective Context Engineering for AI Agents** (https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents, Anthropic, September 2025) — Prescribes compaction that prioritizes architectural decisions and unresolved issues over raw tool outputs, and structured external notes (e.g. NOTES.md) persisted across context resets. States: "Start by maximizing recall to ensure your compaction prompt captures every relevant piece of information from the trace, then iterate to improve precision by eliminating superfluous content." Warns against overly aggressive compression that loses subtle but critical context.

- **Evaluating AI Agents: Real-World Lessons from Building Agentic Systems at Amazon** (https://aws.amazon.com/blogs/machine-learning/evaluating-ai-agents-real-world-lessons-from-building-agentic-systems-at-amazon/, AWS / Amazon, February 2026) — Mandates systematic failure classification across memory, planning, tool invocation, and authentication failure dimensions. Prescribes production dashboards, automated anomaly detection, and feedback loops as the mechanism for converting runtime failures into continuous improvement inputs.

### Named in Literature?

No unified name. Closest named concepts: **Reflexion** (arXiv:2303.11366), **ExpeL** (experience-learning from failure/success trajectory contrast), **AgentErrorTaxonomy** (arXiv:2509.25370), and **episodic memory with importance scoring** (Generative Agents). The specific formulation — maintain a separate, append-only, declarative failure registry distinct from session history, and feed it as a compact prefix — is implied across these sources but not named as a single pattern.

## When to Apply

| Condition | Apply? |
|-----------|--------|
| Agent runs across multiple sessions and must not repeat errors | Yes |
| Session log exceeds ~10k tokens of raw interaction history | Yes |
| Same failure class recurs across two or more sessions | Yes — extract as registry entry immediately |
| Context window compaction is triggered mid-session | Yes — distill active failure observations before compacting |
| Single-session, single-task agent with no memory persistence | No — overhead outweighs benefit |
| Task space is so narrow that all failure modes can be enumerated in the system prompt | No — inline the rules instead |

Apply when:
1. The agent operates across session boundaries and needs cross-session failure avoidance.
2. Raw session logs are too long to fit in context without degrading recall (context rot threshold).
3. A failure class has occurred more than once and carries a generalizable corrective rule.

Do not apply when: the agent is stateless by design and each run is independent with no shared memory infrastructure.

## Known Limits

- **Registry staleness**: Failure rules derived from past conditions may become incorrect after system changes (tool updates, environment shifts, model upgrades). Entries must carry a timestamp and be reviewed after major changes, or the registry becomes a source of outdated anti-patterns rather than valid ones.

- **Extraction quality dependency**: The quality of the registry depends entirely on the quality of failure extraction. If the extractor (human or LLM) produces vague entries ("something went wrong with tool X"), the registry gives the agent no actionable signal. Each entry must be a declarative, condition-specific rule; vague entries must be rejected or rewritten.

- **Garbage-in amplification**: Because failure-log entries are fed as high-priority context, an incorrect or misdiagnosed entry actively misleads subsequent runs more than an equivalent error buried in raw history. Incorrect entries cause more harm here than in generic session logs.

- **Not applicable to single-run, stateless pipelines**: Systems that reset state entirely between invocations gain nothing from a persistent failure registry. This pattern applies only where cross-run memory is architecturally supported.

## Promotion History

Confirmed — promoted 2026-05-19 from user insight (scored 21/25). Source: https://www.minwoo.cloud/blog/ai-failure-hooks-session-log-lessons

## Properties to Restore

위반이 감지됐을 때, 수정된 결과가 만족해야 할 속성들이다.
구체적인 구현 방법은 컨텍스트에 따라 달라질 수 있다.

| Violation detected | Properties the fix must satisfy |
|--------------------|--------------------------------|
| Lessons buried in raw session logs with no separate registry | The corrected design must place extracted lessons in a dedicated, append-only store that is readable without replaying or parsing any session log |
| Repeated mistakes not surfaced across sessions | Each failure entry in the corrected registry must be a single declarative, condition-specific rule — not a narrative — that can be fed as a prefix to subsequent runs without further processing |
| Registry entries too vague to produce actionable signal | Each corrected entry must state the failure condition and corrective rule in a form that a conforming agent can apply without ambiguity; entries that do not meet this standard must be rewritten or rejected |
| Session history too long to surface relevant failures | The corrected registry must be independently accessible at the start of a run without requiring the agent to ingest raw session history; compact lessons must be separable from the session log that produced them |
