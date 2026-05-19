# Evaluation as Behavioral Specification

Status: confirmed
Last reviewed: 2026-05-18

## Principle

Most teams write evals after behavior is implemented — this pattern inverts that ordering. An eval is not a post-hoc check; it is the behavioral specification. Writing the eval first forces the requirement to be observable and measurable before implementation begins.

An agent without evals has no specification. What it does is undefined behavior.

An eval case must encompass: input + expected observable behavior + scoring rule + execution trace + pass/fail verdict. Inspecting the final output alone is insufficient.

This reframes evaluation from output matching (does the result look right?) to behavioral verification (did the agent follow the correct procedure, in the correct order, to produce the correct output?). An output that appears correct but lacks policy evidence in the underlying action trace is a failure. This applies regardless of output type — text, tool calls, structured data, or generated files.

If a requirement cannot be expressed as a measurable eval criterion, the requirement is underspecified.

A second agent evaluating a first agent's output against a rubric (Agent-as-a-Judge) is the agentic extension of this principle: the rubric is the behavioral specification, and the evaluating agent enforces it.

## Evidence

### Academic Papers

- **AgentProcessBench** (arXiv 2603.14465, 2026) — Establishes that final-output evaluation is insufficient for tool-using agents. Step-level labeling (+1/0/-1) of each intermediate action in a trajectory is required. 1,000 trajectories, 8,509 human-labeled steps with 89.1% inter-annotator agreement. Tool-use failures cause irreversible side effects that outcome scores cannot detect.
- **Agent Behavioral Contracts / ABC** (arXiv 2602.22302, Feb 2026) — Brings Design-by-Contract to AI agents, providing the closest direct analog to "eval as behavioral specification." Contract C = (P, I, G, R): Preconditions, Invariants, Governance policies, Recovery mechanisms. Tested across 1,980 sessions: contracted agents detect 5.2–6.8 soft violations per session that uncontracted baselines miss entirely, with 88–100% hard constraint compliance.
- **Agent-as-a-Judge** (arXiv 2410.10934, NeurIPS workshop Oct 2024) — Demonstrates that a second agent evaluating a first agent's output against a rubric provides richer intermediate feedback across the full task-solving process, not just a terminal score. Generalizes to any output type: text, tool call sequences, structured artifacts.
- **Process Reward Models for LLM Agents** (arXiv 2502.10325, Feb 2025) — Shows that outcome-only rewards produce sparse signals. PRMs evaluate each intermediate action step, capturing the decision-making process where failures originate — regardless of what the final output contains.

### Official Best Practices

- **Anthropic "Demystifying Evals for AI Agents"** (Jan 2026, anthropic.com/engineering/demystifying-evals-for-ai-agents) — Defines a **transcript** as "the complete record of a trial, including outputs, tool calls, reasoning, intermediate results." Distinguishes transcript-based grading from outcome-based grading. The outcome is the environment state (e.g., database contents), which differs from what the agent reported — making trace checking the only way to detect certain failure classes.
- **OpenAI Trace Grading** (platform.openai.com/docs/guides/trace-grading) — Productized as a named feature: "A trace captures the end-to-end record of model calls, tool calls, guardrails, and handoffs for one run. Graders score those traces with structured criteria."

### Named in Literature?

Yes. Multiple overlapping names: "trajectory evaluation" (Anthropic/OpenAI), "process-level evaluation" (PRM literature), "step-level verification" (AgentProcessBench), "behavioral contracts" (ABC). The concept is now mainstream.

## When to Apply

| Signal | Decision | What breaks if ignored |
|--------|----------|------------------------|
| Agent makes decisions via tool calls or middleware | Use trace-level eval; score each intermediate action in the trajectory, not just the final output | Output-only scoring misses wrong tool selection, wrong argument, and wrong ordering — all invisible in the final result |
| Agent is responsible for policy compliance (security, approval gating, redaction) | Use contract / invariant eval; define preconditions and invariants as explicit scoring rules | Soft violations — averaging 5.2–6.8 per session (ABC, 2026) — pass output-based evals while actual compliance fails |
| Passing eval but failing production behavior | Run output-vs-trace gap analysis; compare what the eval scored against what the execution trace shows | The eval measures the wrong signal; the gap widens as production usage diverges from the eval input distribution |
| Authoring a new agent capability | Write the eval before writing implementation; define input + expected behavior + scoring rule first | Implementation begins without a specification; the agent's behavior is undefined by default — what it does is not what was required |

## Known Limits

1. **Eval gaming** — An agent trained or prompted against a specific eval learns to satisfy the eval's criteria without performing the intended behavior. The eval passes; the behavior is wrong. Mitigate by varying eval inputs and using held-out rubric items.
2. **Underspecified eval** — The eval passes because the scoring rule is too coarse to distinguish correct from incorrect behavior. The agent does the wrong thing but scores well. Mitigate by grounding rubric items in concrete, observable actions rather than subjective output quality.
3. Trace collection adds overhead; agents must be instrumented before eval can be trace-first.
4. Trace-based evals are more brittle to implementation changes than output-based evals — balance coverage depth with maintenance cost.
5. Does not apply to one-off exploratory runs where behavioral repeatability is not required.

## Promotion History

Candidate — promoted from initial draft, 2026-05-18.
Promotion-ready — strong academic backing (4 papers) + Anthropic + OpenAI official docs. Pattern is explicitly named in literature under multiple terms.
