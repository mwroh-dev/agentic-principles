# Structured Seed Directive

Status: confirmed
Last reviewed: 2026-05-19

## Principle

An initial directive (the "seed" — the first system-level instruction given to an agent) must include four elements: direction (what outcome to pursue), constraints (boundaries the agent must not cross), priorities (ordered criteria when trade-offs arise), and prohibitions (explicit exclusions). A directive that omits any of these elements causes the agent to fill the gap with statistically average behavior — the most common output for that task type in its training distribution.

The failure mode is not that the agent refuses to act. It acts — but produces generic output indistinguishable from a zero-context invocation. Strategic constraints ("do not do X," "prefer Y over Z when both are feasible") are the mechanism that pushes agent behavior outside the average. Without them, no amount of task description recovers differentiated output because the agent lacks the boundary conditions needed to select among competing valid completions.

Structured directives do not require length. They require coverage of all four elements. A 30-word directive covering direction, one constraint, one priority, and one prohibition outperforms a 200-word task description that covers only direction.

## Evidence

### Academic Papers

- **AGENTIF: Benchmarking Instruction Following of Large Language Models in Agentic Scenarios** (arXiv:2505.16944, May 2025, Tsinghua University) — Benchmarks 707 real-world agentic instructions averaging 11.9 constraints each; finds that current models "perform poorly, especially in handling complex constraint structures and tool specifications." Directly demonstrates that constraint structure — not task description alone — is the primary driver of agentic instruction-following failure.

- **5C Prompt Contracts: A Minimalist, Creative-Friendly, Token-Efficient Design Framework for Individual and SME LLM Usage** (arXiv:2507.07045, July 2025, Ugur Ari) — Formalizes prompt design into five components: Character, Cause, Constraint, Contingency, and Calibration. Empirically shows that including the Constraint and Contingency dimensions "consistently achieves superior token efficiency while maintaining rich and consistent outputs across diverse LLM architectures," validating that structured directive elements outperform unstructured task descriptions.

- **RECAST: Expanding the Boundaries of LLMs' Complex Instruction Following with Multi-Constraint Data** (arXiv:2505.19030, May 2025, Fudan University) — Demonstrates that models trained and prompted with constraint-rich instructions (up to 30+ constraints per example) "substantially improve in following complex instructions while maintaining general capabilities." Provides empirical evidence that constraint density in directives directly scales instruction-following fidelity.

- **Enhancing LLM Instruction Following: An Evaluation-Driven Multi-Agentic Workflow for Prompt Instructions Optimization** (arXiv:2601.03359, January 2026, Purpura et al.) — Shows that separating constraint optimization from task description optimization produces "significantly higher compliance scores" on Llama 3.1 8B and Mixtral-8x7B. Identifies that LLMs generate "substantively relevant content but fail to adhere to formal constraints" when constraints are not explicitly decomposed — i.e., task direction alone is insufficient.

### Official Best Practices

- **Prompting Best Practices — Claude 4 Models** (https://platform.claude.com/docs/en/docs/build-with-claude/prompt-engineering/claude-4-best-practices, Anthropic, 2025) — Explicitly states: "it's important to specify the task, intent, and relevant constraints upfront in the first human turn. Providing well-specified, clear, and accurate task descriptions upfront can help maximize autonomy and intelligence... ambiguous or underspecified prompts conveyed progressively over multiple user turns tend to relatively reduce token efficiency and sometimes performance." Also demonstrates that qualitative constraints like "be conservative" without explicit scope definitions cause model behavior to diverge unexpectedly from operator intent.

- **System Prompts Define the Agent as Much as the Model** (https://www.dbreunig.com/2026/02/10/system-prompts-define-the-agent-as-much-as-the-model.html, Drew Breunig, February 2026) — Reports an empirical prompt-swap experiment: two agents using identical underlying models (Claude Opus 4.5) produced immediately divergent workflows when their system prompts were exchanged. Concludes: "A given model sets the theoretical ceiling of an agent's performance, but the system prompt determines whether this peak is reached." Recurring directive patterns across production agent prompts include explicit restrictions, parallel-call mandates, and constraint-first ordering.

- **Building Effective Agents** (https://www.anthropic.com/research/building-effective-agents, Anthropic, December 2024) — Prescribes that each subagent needs "an objective, an output format, guidance on the tools and sources to use, and clear task boundaries." States that "without detailed task descriptions, agents duplicate work, leave gaps, or fail to find necessary information" — a direct operational consequence of under-specified initial directives.

### Named in Literature?

No unified name. Closest named concepts: "system prompt engineering," "instruction specification," "constraint-rich prompting," and the 5C framework's Constraint+Contingency components — but the specific formulation — that direction, constraints, priorities, and prohibitions are the four required structural elements of an initial agent directive, and that omitting any causes regression to average output — is implied across sources but not named as a single pattern.

## When to Apply

| Condition | Apply structured directive |
|-----------|---------------------------|
| Agent must produce output distinguishable from zero-context invocation | Yes |
| Agent has tool access and autonomy over execution path | Yes |
| Task has trade-off scenarios where the agent must choose between valid alternatives | Yes |
| Agent operates in a multi-step or long-horizon workflow | Yes |
| Agent output will be consumed by downstream agents or systems | Yes |
| Simple single-turn Q&A with no autonomy or tool use | No |

Apply when:
1. The agent will make autonomous decisions about approach, tool selection, or output format without per-step human approval.
2. Two or more valid implementation paths exist and the operator has a preference that is not deducible from the task description alone.
3. There are actions the agent must never take regardless of user input — these prohibitions must appear in the initial directive, not be inferred from context.

Do not apply when: the agent executes a single deterministic operation with no branching (e.g., a pure formatting or translation function with no decision points).

## Known Limits

- **Constraint conflicts are not automatically resolved**: If the initial directive contains contradictory constraints (e.g., "be comprehensive" and "be concise"), the agent selects one heuristically. Contradictory constraints must be resolved at directive-authoring time, not by the agent at runtime.

- **Prohibitions require specificity to be effective**: Generic prohibitions ("don't do anything bad") do not constrain behavior. Prohibitions must name concrete actions or output categories. Vague prohibitions may produce false compliance — the agent treats the prohibition as satisfied while taking the prohibited action in a different form.

- **Directive structure does not substitute for task clarity**: Covering all four elements with underspecified content does not prevent generic output. Each element must be precise. A structured but vague directive performs only marginally better than an unstructured one.

- **This pattern does not apply to pure retrieval or classification tasks**: Agents whose entire action space is lookup or label selection have no behavioral variance to constrain. Structured directives add overhead without benefit in these cases.

## Promotion History

Candidate — created 2026-05-19 from user insight (source: https://www.minwoo.cloud/blog/harness-direction-over-consistency).
Confirmed — 2026-05-19, user decision, score 19/25.

## Properties to Restore

위반이 감지됐을 때, 수정된 결과가 만족해야 할 속성들이다.
구체적인 구현 방법은 컨텍스트에 따라 달라질 수 있다.

| Violation detected | Properties the fix must satisfy |
|--------------------|--------------------------------|
| Initial directive missing one or more structural elements | The corrected directive must contain all four elements — direction, constraints, priorities, and prohibitions — each stated explicitly, not implied by context |
| Agent regressed to statistically average output due to absent constraints or prohibitions | The corrected directive must include at least one constraint or prohibition that distinguishes the intended behavior from the default output for the same task description |
| Omission of a structural element was unintentional | Any element intentionally omitted from the directive must be explicitly documented as a deliberate choice with stated rationale; absence alone is not sufficient |
| Contradictory constraints caused the agent to select heuristically | The corrected directive must resolve all conflicting elements so that the agent faces no situation where two stated constraints produce incompatible requirements |
