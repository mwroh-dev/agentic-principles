# State Contract Over Prose Runbook

Status: confirmed
Last reviewed: 2026-06-08

## Principle

The condition→action→trace structure is the established pattern for agent phase instructions. ReAct (2210.03629, ICLR 2023) demonstrated that separating context reasoning (Thought), execution (Action), and outcome verification (Observation) into explicit labeled units significantly outperforms unstructured narrative reasoning. Memory-Augmented State Machine Prompting (2510.18395, 2025) applied this to tactical agent decision-making through explicit state definitions, transition conditions, and state-action mappings — achieving 60% win rate against StarCraft II's hardest AI versus 0% for unstructured prose-instruction baselines.

The pattern generalizes to all phase gate instructions: make the condition explicit, make the action explicit, make the outcome observable. Prose runbooks violate this pattern.

When instructions chain conditions narratively — "Before X, read A. If A, stop. Also read B. Then proceed" — the model must infer execution structure from connectives. "Before," "also," and "then" are ambiguous: they do not distinguish sequence from concurrency, required gates from soft preferences, or terminal conditions from advisory notes. The model resolves them by surface pattern-matching, not logical inference, producing silent misexecution that looks syntactically correct.

The state contract removes the ambiguity:

| Pre-condition | Action | If blocked |
|---|---|---|
| A is present | Proceed to X | Stop |
| B is present | Proceed to X | Stop |

State contracts also create explicit audit checkpoints: each row is independently verifiable. A reviewer can confirm whether condition A was correctly evaluated before action X was taken. A prose runbook has no equivalent checkpoint structure — there is no row to point to, no decision that is independently verifiable.

This principle applies specifically to instructions that govern agent decision-making, phase transitions, and checkpoint gates. Human-facing documentation and concept explanations are appropriately written in prose — the constraint is on machine-executed instructions, not on human-readable content.

## Evidence

### Academic Papers

- **ReAct: Synergizing Reasoning and Acting in Language Models** (arXiv:2210.03629, ICLR 2023, Yao et al.) — Demonstrates that interleaving explicit Thought (condition/context reasoning), Action (execution step), and Observation (outcome verification) significantly outperforms chain-of-thought prompting on knowledge-intensive and decision tasks. The tripartite structure is the earliest named embodiment of the condition→action→trace pattern for agent instructions. Directly establishes that structured, labeled execution traces produce more reliable and more auditable agent behavior than unstructured narrative chains.

- **Memory-Augmented State Machine Prompting** (arXiv:2510.18395, Oct 2025) — Proposes MASMP: agent instructions expressed as explicit finite state machine structures with named tactical states, natural-language transition conditions, and state-action mappings. Achieves 60% win rate against StarCraft II's hardest AI (Lv7) versus 0% for unstructured prose-instruction baselines. Direct empirical validation that state contract format produces substantially more reliable execution than narrative prose under the same task conditions.

- **Premise Order Matters in Reasoning with Large Language Models** (arXiv:2402.08939, Feb 2024, Google DeepMind) — Confirms that the surface arrangement of conditions in natural language determines which conditions the model treats as primary, independently of the author's intent. Reordering logically equivalent premises reduces GPT-4-turbo accuracy by 20–30%. Structured formats that explicitly label conditions as parallel gates or required preconditions override this surface-order bias — validating the table format as a layout that removes position-dependent priority assignment.

- **RuozhiBench: Evaluating LLMs with Logical Fallacies and Misleading Premises** (arXiv:2502.13125, Feb 2025) — Finds even the best-performing model achieves only 62% accuracy on questions with implicit conditional structures, compared to human performance above 90%. Confirms that models resolve conditionals in natural language using surface pattern matching rather than explicit structural parsing. This baseline failure rate applies directly to prose phase gate instructions containing implicit conditional logic.

- **Shift-Up: A Framework for Software Engineering Guardrails in AI-native Software Development** (arXiv:2604.20436, 2026, Lipsanen et al.) — Demonstrates that embedding machine-readable structured artifacts stabilizes agent behavior and reduces implementation drift. Validates that structured representation of conditions and requirements produces more reliable agent behavior than unstructured natural language equivalents.

### Official Best Practices

- **Anthropic — Effective Context Engineering for AI Agents** (https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents, 2025) — Prescribes organizing prompts into distinct labeled sections using XML tags or Markdown headers to delineate behavioral scope. Implies that unstructured prose without explicit section boundaries creates ambiguous scope for agent execution.

- **Anthropic — Building Effective Agents** (https://www.anthropic.com/research/building-effective-agents, 2024) — Prescribes "maintaining simplicity in agent design" and warns that incorrect assumptions about agent behavior are "a common source of error." Emphasizes explicit success criteria over implicit reasoning chains.

- **OpenAI — Prompt Engineering Guide** (https://platform.openai.com/docs/guides/prompt-engineering, OpenAI) — Prescribes breaking complex tasks into "a series of clear steps" and restructuring compound instructions so each step carries one requirement. Demonstrates that separating extraction and generation into ordered steps improves compliance over bundled prose.

### Named in Literature?

The ReAct pattern (Thought/Action/Observation) is named and widely cited. Memory-Augmented State Machine Prompting is named. Closest general concepts: "structured prompt decomposition" (instruction-following literature), "precondition-action pairs" (planning/STRIPS tradition). The specific formulation — replacing temporal prose runbooks with explicit state contracts for agent phase gate instructions — is not independently named but follows directly from the ReAct and MASMP patterns.

## When to Apply

| Signal | Decision | What breaks if ignored |
|---|---|---|
| Instruction contains "before X, do Y" or "also check Z" sequences | Convert to ordered table of precondition → action pairs | Model resolves connectives as soft preference, not hard gate |
| Phase transition requires multiple gates to all pass | Use explicit parallel-gate table, not prose listing | Model treats first satisfied gate as sufficient |
| Checkpoint names and their allowed verdicts are described in prose | Convert to table: checkpoint name, trigger, accepted verdicts, blocked action | Model misidentifies which verdicts unblock which checkpoints |
| Instructions say "do not proceed unless..." | Convert to explicit precondition column | Model treats prose "unless" as advisory, not terminal |
| The same instruction has both explanatory prose and executable conditions | Separate: state contract for execution, prose section for explanation | Explanatory context contaminates conditional parsing |

Do not apply when: the instruction is human-facing documentation, a concept explanation, or a rationale section. Prose is appropriate for explaining *why* a structure exists. State contracts are appropriate for specifying *what* the agent must do under *which* conditions.

## Known Limits

- **Schema overhead for simple instructions**: A two-condition gate can be expressed as a table or as "check A and B before proceeding." When conditions are few and clearly sequential, table format adds structure without reducing ambiguity meaningfully. Apply this pattern when prose contains three or more chained conditions, temporal connectives that could be read as sequence or concurrency, or conditions that modify each other.

- **Tables lose narrative context**: State contracts are harder to read for human reviewers than prose runbooks. Mitigate by keeping a prose explanation section above the state contract, separated by a heading.

- **Format alone does not guarantee interpretation**: A well-formatted table is still parsed by a language model. Format reduces, but does not eliminate, interpretation errors. Combine structured format with explicit headers that name what each column means.

- **Not applicable to open-ended reasoning tasks**: This pattern applies to procedural instructions with defined conditions and actions. Open-ended reasoning tasks (analysis, generation, summarization) do not have gate conditions and should not be forced into state contract format.

## Promotion History

Candidate — created 2026-06-07 from observed failure pattern: validate-skill.mjs prose runbooks and browser-flow prompt.md "Before/Also/Then" chains caused silent misexecution in model-directed agent pipelines.

Note: Original citations arXiv:2311.09669, arXiv:2408.00682, arXiv:2301.01569 were incorrect IDs (matched unrelated physics/math/ML papers). Corrected to verified papers on 2026-06-07 per Gate 3 (Fact) check.

Confirmed — 2026-06-08. 5-dimension score: 21/25 (코어성 4, 리스크감소 5, 확장성 4, 제어성 4, 기록성 4). Reframed from "don't use prose" prohibition to "condition→action→trace is the established pattern." Added arXiv:2210.03629 (ReAct, ICLR 2023) and arXiv:2510.18395 (MASMP) as direct empirical validation. 제어성 and 기록성 connections made explicit: state contracts create auditable checkpoints and traceable decision logs.
