# Stage-Local Completion and Repair

Status: confirmed
Last reviewed: 2026-06-19

## Principle

Each stage in a multi-stage agent pipeline must own a **stage-local DONE gate**: a completion criterion evaluated against the stage's own output, independently of downstream stages. When a new input variant breaks the pipeline, repair is bounded to the earliest stage whose local gate fails. These properties are inseparable: without a local gate, there is no principled basis for bounding repair scope. This is a structural property of the pipeline, independent of model capability; even a perfect single-pass model provides no stage-level attribution if no gate demarcates stage ownership.

A global-final gate — one that checks only the terminal output — hides which stage is broken. Every failure forces full pipeline re-execution regardless of which stages already produced valid outputs, re-exposing validated data to mutation and masking the structural location of the defect. The cost of a missing local gate is paid on every repair cycle, compounding with pipeline length.

## Evidence

### Academic Papers

- **Which Agent Causes Task Failures and When? On Automated Failure Attribution of LLM Multi-Agent Systems** (arXiv:2505.00212, May 2025, Penn State / Duke; ICML 2025 Spotlight) — Formalizes "failure-responsible agent" and "decisive error step" as the two coordinates of stage-level failure attribution in 127 LLM multi-agent systems. The paper's central claim — that repair guidance requires attributing failure to a specific agent *and* a specific step within that agent — presupposes exactly the stage-local gate model: repair scope is bounded by attribution. The Who&When dataset provides the first large-scale empirical grounding for this assumption.

- **DoVer: Intervention-Driven Auto Debugging for LLM Multi-Agent Systems** (arXiv:2512.06749, Dec 2025/Jan 2026) — Proposes intervention-driven debugging: rather than re-running the full pipeline, DoVer edits the message or plan at the attributed step and verifies recovery. Validates 30–60% of repair hypotheses through targeted stage-local interventions, recovering 18–28% of previously failed tasks on AssistantBench and GAIA without full pipeline restart. This is an empirical demonstration that bounded, stage-local repair is both feasible and effective.

- **MASPrism: Lightweight Failure Attribution for Multi-Agent Systems Using Prefill-Stage Signals** (arXiv:2605.07509, May 2026, Sun Yat-sen University) — Attributes pipeline failure to specific contributing steps using token-level log-likelihood and attention weights from a single prefill pass, without replay or retraining. Stage-level attribution is the prerequisite operation that makes local repair possible; MASPrism makes it lightweight enough to run continuously.

- **Why Do Multi-Agent LLM Systems Fail? (MAST)** (arXiv:2503.13657, Mar 2025; NeurIPS 2025) — Empirical study of 200+ tasks across 7 MAS frameworks. Classifies failures by the pipeline stage at which they originate (Pre-Execution, Execution, Post-Execution), demonstrating that stage-level attribution is necessary to select the right intervention. Finds that improved orchestration-level specification (not full pipeline re-design) yields +14% task success.

### Engineering Analogies

The pattern is not novel in software engineering. It is the foundational design principle of two established categories of build and deployment systems. These are analogies from adjacent domains — not AI agent literature — but they are named, production-validated implementations of stage-local completion and repair.

- **Bazel / Ninja incremental build systems** (bazel.build) — Each build action has an explicit contract (input files + command → output files). Bazel evaluates each action's local completion criterion (hash of inputs unchanged + output exists and valid) before deciding whether to re-execute it. When a downstream step fails, only actions whose inputs changed are invalidated and re-run. Actions with unchanged, verified inputs are never re-executed regardless of downstream failures. This is stage-local completion and bounded repair in production at scale.

- **CI pipeline stage reruns** (CloudBees CD, GitHub Actions, GitLab CI) — CI platforms implement explicit "rerun only the failed stage" semantics. CloudBees CD documentation states explicitly: "release management can rerun a failed stage to recover from stage errors without potentially rerunning expensive stages that do not require a rerun." This is the direct operational equivalent of the principle applied to LLM pipelines.

### Official Best Practices

- **Anthropic "Building Effective Agents"** (anthropic.com/research/building-effective-agents) — Recommends checkpointing and pausing at stage boundaries before irreversible actions, and notes that agents should "gain ground truth from the environment at each step to assess progress." The checkpoint recommendation implies that each stage boundary is an evaluation point — which is a stage-local gate. The guidance does not name the pattern explicitly.

- **Anthropic "Demystifying Evals for AI Agents"** (anthropic.com/engineering/demystifying-evals-for-ai-agents) — States that evals run at each step of the agent pipeline in CI/CD, "as the first line of defense." Step-level evals are stage-local gates by another name; the documentation treats them as standard practice for production pipelines.

### Named in Literature?

No unified name exists in AI agent literature. The concept appears under: "decisive error step" and "failure attribution" (arXiv:2505.00212), "intervention-driven debugging" (DoVer, arXiv:2512.06749), and "stage-level failure localization" (MAST, arXiv:2503.13657). The specific two-part formulation — stage-local *completion* gate enabling stage-local *repair* scope — is consistent across sources as an assumed prerequisite, but is not yet consolidated under a single pattern name.

In software engineering, the concept is named: "incremental build" (Bazel, Ninja) and "stage rerun" (CI pipeline tooling). The AI agent literature is converging on the same abstraction under the label "failure attribution."

## When to Apply

| Signal | Decision | What breaks if ignored |
|--------|----------|------------------------|
| Pipeline has 3 or more sequential stages, each with distinct input/output contracts | Define a local completion gate for each stage; run it before passing output to the next stage | Terminal gate failures require full pipeline re-execution to isolate the defect; repair cost scales with pipeline length |
| A new input variant fails the terminal gate | Identify the earliest stage whose local gate fails; replay only from that stage | Full re-execution re-processes all stages, including those that already produced validated outputs; already-validated data may be re-mutated |
| Stage outputs are cached or persisted as intermediate artifacts | Stage-local gates determine which cached outputs remain valid after a failure | Without per-stage validity assessment, all cached outputs must be invalidated on any failure — defeating the purpose of caching |
| A multi-stage pipeline is being extended with a new stage | Define the new stage's local DONE gate before integrating it; verify it independently before connecting to adjacent stages | A new stage without a local gate becomes a silent failure point; its failures surface only at the terminal gate, with all upstream stages implicated |
| Post-failure repair is expensive (compute, data integrity, downstream contamination) | Bound repair scope to the failing stage; do not re-execute stages that passed local gates | Re-executing passing stages introduces unnecessary mutation risk and compute cost proportional to the number of passing stages upstream of the defect |

Do not apply when:
- Stage contracts have changed (schema update, model upgrade, prompt revision) — changed contracts invalidate all prior local gate passes; full re-validation is required from the first affected stage.
- Stage outputs have hidden inter-stage semantic dependencies not captured in either stage's local contract — in this case, local gate passes may be individually valid while the joint output is semantically corrupt.
- The pipeline has only one stage, or all stages share the same input/output contract — there is no stage boundary at which a local gate adds value over the terminal gate.

## Known Limits

**False confidence from local gate pass**: A stage-local gate checks the stage's output against its own contract. It does not check whether that output is semantically coherent with the outputs of other stages. In pipelines where downstream stages interpret upstream outputs with implicit assumptions, a local gate pass masks contract-level corruption that only becomes visible when stages are composed. Local gates are necessary but not sufficient for end-to-end correctness.

**Idempotency requirement**: Selective re-execution from a specific stage requires that stage to be idempotent — running it twice on the same input must produce the same output. Stages with non-deterministic LLM calls or external state mutations do not satisfy this. The local gate model depends on idempotency; verify it before relying on bounded repair.

**Stage contract drift**: Local gates are only as strong as the contracts they check. As pipeline requirements evolve, stage contracts drift from what the local gate actually validates. Without explicit versioning and re-derivation of gate logic on contract change, the gate provides false assurance. Treat stage contracts and their gates as co-versioned artifacts.

**Full re-run triggers**: The following conditions override local gate passes and require full pipeline re-execution: (1) any stage contract definition has changed; (2) a shared resource or mutable state accessed by multiple stages has been externally modified; (3) a model or prompt used in a passing stage has been upgraded. Local repair is valid only when the stages not being re-run are provably unchanged.

**Attribution accuracy gap and inapplicability domain**: The Who&When study (arXiv:2505.00212) finds that the best current automated attribution method achieves only 53.5% accuracy for agent identification and 14.2% for step identification. Until attribution tooling improves, human confirmation of which stage failed is required before bounded repair is applied in production. This pattern does not apply to pipelines that cannot be decomposed into stages with independently verifiable contracts — including monolithic single-agent loops, pipelines with shared mutable state across all stages, and systems where stage boundaries are not defined in advance of execution.

## Promotion History

Candidate created 2026-06-19. The principle combines two observations into an inseparable unit: (1) stage-local completion gates make it possible to evaluate each stage independently, and (2) stage-local repair scope is the direct consequence — the earliest stage whose gate fails is the only stage that requires repair. Academic evidence is moderate (failure attribution literature presupposes the model but does not name it as a unit); engineering evidence (Bazel, CI stage reruns) is strong and production-validated.

2026-06-19: Promoted to confirmed/execution/ via 4-phase refinement pipeline. Spec gate: PASS (all 6 criteria). Prose gate: APPROVED (0 issues).
