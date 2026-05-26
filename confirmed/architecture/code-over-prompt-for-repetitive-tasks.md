# Code Over Prompt for Repetitive Tasks

Status: confirmed
Last reviewed: 2026-05-19

## Principle

When a workflow step executes the same deterministic transformation on every invocation — fixed routing, field extraction, schema validation, format conversion — encode that step as code, not as a prompt. Reserve LLM invocations for steps where the output varies with context and cannot be predetermined.

Violating this principle causes three compounding problems: (1) token cost scales linearly with invocation count instead of amortizing to near-zero; (2) per-invocation output variance introduces non-determinism into steps that have a single correct answer; (3) debugging becomes probabilistic — the same input can silently produce different outputs across runs.

The decision rule is: if a human engineer could write a unit test with a fixed expected output for this step, implement it as code. If the correct output depends on semantic judgment that varies with context, use an LLM. Hybrid workflows — deterministic outer scaffolding with LLM-powered judgment nodes — capture both benefits.

## Evidence

### Academic Papers

- **Compiled AI: Deterministic Code Generation for LLM-Based Workflow Automation** (arXiv:2604.05150, April 2026, XY.AI Labs / Stanford / Cornell / Harvard) — Proposes a compile-once, execute-deterministically architecture in which LLMs generate code artifacts during a one-time compilation phase; subsequent workflow runs execute that code with zero model invocations. Achieves 96% task completion on function-calling benchmarks (BFCL, n=400) and a 57× reduction in token consumption at 1,000 transactions; break-even against runtime inference occurs at approximately 17 transactions.

- **Optimizing Agentic Workflows using Meta-tools** (arXiv:2601.22037, January 2026) — Introduces Agent Workflow Optimization (AWO), which mines workflow execution traces to identify recurring tool-call sequences and bundles them into deterministic composite meta-tools that bypass intermediate LLM reasoning steps. Reduces LLM calls by up to 11.9% and increases task success rate by up to 4.2 percentage points across two agentic benchmarks.

- **Agentic Compilation: Mitigating the LLM Rerun Crisis for Minimized-Inference-Cost Web Automation** (arXiv:2604.09718, April 2026) — Formalizes the "Rerun Crisis": a 5-step web workflow repeated 500 times costs ~$150 under continuous LLM inference. The paper introduces a compile-and-execute architecture that generates a one-time JSON blueprint from a single LLM call, then executes subsequent runs deterministically. Per-workflow cost drops to under $0.10; zero-shot compilation success rates reach 80–94% across task types.

- **Get Experience from Practice: LLM Agents with Record & Replay** (arXiv:2505.17716, 2025, Shanghai Jiao Tong University) — Proposes AgentRR, which records successful agent executions and replays structured experience plans for repeated tasks, decoupling intelligence from execution. Replay eliminates per-step LLM calls for confirmed task paths, reducing runtime cost and making execution bounded and auditable.

### Official Best Practices

- **Building Effective Agents** (https://www.anthropic.com/research/building-effective-agents, Anthropic) — Establishes the workflow/agent distinction: workflows use predefined code paths; agents use dynamic LLM-driven decision-making. Prescribes hardcoded logic as the default for tasks with predictable steps: "many patterns can be implemented in a few lines of code," and complexity should be added "only when it demonstrably improves outcomes."

- **A Practical Guide to Building Agents** (https://cdn.openai.com/business-guides-and-resources/a-practical-guide-to-building-agents.pdf, OpenAI) — Recommends deterministic routing and fixed workflows for steps with stable, pre-definable logic ("same steps, same logic, every time"), reserving agentic loops for tasks where "the path depends on what you find along the way." States that fixed workflows are "easier to test and trace" and that agentic architecture is frequently selected when a workflow "would give 90% of the value at a fraction of the operational cost."

### Named in Literature?

No unified name. Closest named concepts: "agentic compilation" (arXiv:2604.09718), "workflow vs. agent distinction" (Anthropic), "meta-tool optimization" (arXiv:2601.22037), and "record & replay" (arXiv:2505.17716). The specific formulation — replacing LLM prompt calls with deterministic code for steps whose output is fixed — is implied across these sources but not consolidated under a single term.

## When to Apply

Apply when:

| Condition | Signal |
|-----------|--------|
| Output is fully determined by input | A unit test with a fixed expected output can be written for this step |
| The step repeats across many invocations | The same transformation runs ≥17 times (empirical break-even from arXiv:2604.05150) |
| Variance between runs is a defect, not a feature | Non-deterministic output causes downstream errors or unpredictable pipeline behavior |
| The step has no semantic ambiguity | Routing by keyword, schema validation, field extraction from a fixed format, type conversion |
| Debugging requires reproducibility | An audit trail or unit-testable specification is required |

Do not apply when: the correct output depends on context that cannot be reduced to explicit rules — semantic similarity judgments, open-ended synthesis, reasoning over novel or ambiguous inputs. In these cases, LLM invocation is irreducible.

## Known Limits

- **Compilation overhead at low volume**: The compile-once approach requires an upfront LLM call plus engineering effort to validate generated code. Below the break-even threshold (~17 transactions per arXiv:2604.05150), runtime prompt invocation may be cheaper in total cost. Apply only when the step will be invoked frequently enough to amortize compilation cost.

- **Brittleness on input variation**: Deterministic code that handles a fixed schema breaks silently when the input format changes. LLM-based steps degrade gracefully on novel inputs; code-based steps fail hard. Pair deterministic steps with explicit input validation and version-controlled schemas to catch format drift early.

- **Boundary identification is non-trivial**: Distinguishing "judgment required" from "judgment not required" is itself a design decision requiring domain knowledge. Premature classification of a step as deterministic — when edge cases require semantic reasoning — produces silent errors that are harder to detect than LLM variance.

- **Does not apply to discovery or exploration phases**: When the task structure itself is unknown — exploratory research, open-ended problem decomposition, novel domain classification — deterministic code cannot be written before the task runs. This pattern applies only to confirmed, stable workflow steps.

## Promotion History

Candidate — created 2026-05-19 from user insight.
Confirmed — 2026-05-19. Scored 20/25 by user review. Promoted to confirmed/architecture/.

## Properties to Restore

위반이 감지됐을 때, 수정된 결과가 만족해야 할 속성들이다.
구체적인 구현 방법은 컨텍스트에 따라 달라질 수 있다.

| Violation detected | Properties the fix must satisfy |
|--------------------|--------------------------------|
| An LLM is invoked for a step whose correct output is fully determined by its input (fixed routing, schema validation, field extraction from a known format) | The step produces identical output for identical input on every invocation. A unit test with a fixed expected output can be written and passes consistently. |
| Invocation cost for a fixed transformation scales linearly with the number of times the workflow runs | The per-invocation cost of the step approaches zero after an initial setup cost. Total cost does not grow proportionally with invocation count for the same transformation. |
| The same deterministic transformation is implemented as a prompt in one part of the pipeline and as code in another | A single canonical implementation exists for each transformation. All pipeline paths that apply the same transformation call the same implementation. |
