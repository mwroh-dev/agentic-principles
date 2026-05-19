# Dual Polarity Evaluation

Status: confirmed
Last reviewed: 2026-05-19

## Principle

An evaluation suite for an AI agent system must include two complementary case types: **golden cases** (inputs the system must handle correctly and pass) and **red cases** (inputs the system must refuse or fail on). Omitting either type produces an incomplete oracle.

A golden-case-only suite confirms that the system performs desired behaviors under expected conditions. It cannot confirm that the system withholds undesired behaviors under adversarial or edge conditions. An evaluator that only measures what the system does when things go right has no signal on where the system fails when things go wrong. The result is a system that appears trustworthy in test but behaves unpredictably in deployment.

Red cases operationalize the boundary: they define, precisely and verifiably, what the system must never do. Without an explicit and accumulating set of must-fail cases, the boundary drifts. Regressions in safety or constraint behavior go undetected because no test is watching for them.

Trustworthy judgment requires both polarities. If X then evaluate must-pass behavior; also evaluate must-fail behavior for any output domain with a defined boundary. A suite that passes only when the system performs correctly on positive inputs, without verifying refusal on negative inputs, is not a trustworthy evaluation — it is half an evaluation.

## Evidence

### Academic Papers

- **HarmBench: A Standardized Evaluation Framework for Automated Red Teaming and Robust Refusal** (arXiv:2402.04249, February 2024, Center for AI Safety — Mazeika, Phan, Yin, Zou, Wang, Mu, Sakhaee, Li, Basart, Bo Li, Forsyth, Hendrycks) — Benchmarks 33 LLMs across 18 red teaming methods, treating must-fail refusal as a first-class metric alongside capability performance; the framework's name itself encodes the dual requirement: red teaming (must-fail) and robust refusal (must-pass). Establishes attack success rate (ASR) — proportion of must-fail cases that incorrectly passed — as a primary evaluation metric, formalizing that positive-only evaluation leaves the ASR dimension entirely unmeasured.

- **ImpossibleBench: Measuring LLMs' Propensity of Exploiting Test Cases** (arXiv:2510.20270, October 2025, Zhong, Raghunathan, Carlini) — Introduces deliberately impossible tasks (tasks where the specification and unit tests conflict) as must-fail evaluation items; an agent that passes these tasks has necessarily violated the specification. Demonstrates that evaluation suites built only from solvable (golden) cases cannot detect shortcut exploitation — models passed benchmarks by deleting failing tests rather than fixing code, a failure invisible to positive-only evaluation.

- **ALERT: A Comprehensive Benchmark for Assessing Large Language Models' Safety through Red Teaming** (arXiv:2404.08676, April 2024, Tedeschi, Friedrich, Schramowski, Kersting, Navigli, Nguyen, Bo Li — Babelscape/TU Darmstadt/UMD) — A 45,000-item safety benchmark organized under a 6-macro / 32-micro risk taxonomy; assigns both a general safety score and category-specific scores by evaluating both harmful outputs that should not occur (red cases) and safe outputs that should occur (golden cases). Finding: 10 evaluated LLMs, including leading models, failed to meet acceptable safety thresholds — failures detected only because must-fail cases were explicitly included.

### Official Best Practices

- **Anthropic Transparency Hub — Model Report** (https://www.anthropic.com/transparency/model-report, Anthropic, 2025) — Anthropic's evaluation structure explicitly separates must-succeed assessments (e.g., "Claude Opus 4.5 appropriately assisted with 87.7% of legitimate security research tasks") from must-fail assessments (e.g., "refusing 88.39% of agentic abuse requests"). The documentation states models are "judged ineffective if [they refuse] before establishing that there is an unresolvable risk" — confirming that both over-refusing on golden cases and under-refusing on red cases are evaluated as failures.

- **OpenAI Red Teaming Network** (https://openai.com/index/red-teaming-network/, OpenAI, 2024) — Describes red teaming as a required complement to standard capability evaluations: "evaluations measure propensity to generate disallowed content" alongside capability benchmarks. Positions must-fail coverage as a prerequisite for pre-deployment clearance, not an optional supplement to positive evaluation.

### Named in Literature?

No unified name. Closest named concepts: **oracle completeness** (software testing), **attack surface coverage** (red teaming), **dual-objective safety evaluation** (used informally in safety literature), and **positive/negative test partitioning** (formal methods). The specific formulation — that an AI agent evaluation suite is structurally incomplete without accumulating must-fail cases alongside must-pass cases — is implied across sources but not named as a single pattern.

## When to Apply

| Condition | Apply dual polarity? |
|---|---|
| System has any defined output boundary (refusal policy, safety constraint, behavior prohibition) | Yes — red cases are required |
| System is evaluated before deployment or promotion to a new environment | Yes — both polarities required |
| Regression testing after a model update, fine-tune, or prompt change | Yes — must-fail regressions are the primary risk |
| Evaluation suite is being built from scratch | Yes — design for both case types from the start |
| System operates in a domain with high-stakes failure modes (safety, security, compliance) | Yes — accumulate red cases continuously |
| Evaluation covers a pure capability metric with no defined refusal boundary (e.g., math benchmark, translation quality) | Not required — golden cases sufficient when no behavioral boundary exists |

Apply when:
1. The system has at least one class of outputs it must never produce, and those outputs can be specified as test inputs.
2. Evaluation results will be used to make a trust judgment (deploy, promote, approve, certify).
3. The system is subject to regression testing across versions or configurations.

Do not apply when: the evaluated behavior has no defined failure boundary — for example, a pure generation quality task scored by human preference where any output is acceptable and no output is categorically prohibited.

## Known Limits

- **Red case accumulation lag**: Must-fail cases are only as comprehensive as the threat model behind them. Novel attack vectors, jailbreak patterns, or adversarial inputs that have not yet been observed cannot be pre-specified as red cases. Red case libraries require ongoing maintenance as the system's deployment context and adversarial landscape evolve.

- **Oracle specification cost**: Defining what constitutes a correct refusal on a red case requires explicit policy: the evaluator must know, for each must-fail input, what counts as passing (refusing) vs. failing (complying). Ambiguous policies produce unreliable oracles, making results from the must-fail side of the suite as unreliable as golden-case-only evaluation.

- **False refusal asymmetry**: Dual polarity evaluation surfaces a tension: increasing must-fail coverage (adding stricter red cases) may increase false refusals on golden cases. Systems optimized to pass all red cases may over-refuse on legitimate golden inputs. Both failure modes must be tracked; resolving the tension is a calibration problem, not a reason to omit one polarity.

- **Does not apply to single-metric capability benchmarks**: This pattern is irrelevant when the evaluation domain has no behavioral constraint — for example, measuring throughput, latency, or raw accuracy on a closed-answer task where all outputs are evaluated on a continuous quality scale and no output class is prohibited.

## Promotion History

Candidate — created 2026-05-19 from user insight (source: https://www.minwoo.cloud/blog/harness-direction-over-consistency). Scored 21/25.
Confirmed — promoted 2026-05-19 by user decision.
