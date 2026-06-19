# Vertical Slice Before Horizontal Scale

Status: confirmed
Last reviewed: 2026-06-19

## Principle

Breadth at any pipeline stage is prohibited until depth across all stages is confirmed. Run one sample through **all** N stages end-to-end before expanding any single stage horizontally. If stage k produces output that stages k+1 through N have never consumed, horizontal scale of stage k is blocked until those downstream stages are exercised.

The structural reason is failure attribution, not throughput. When stage k is scaled before downstream stages are exercised, failures propagate silently and surface only as corrupted terminal output. Attribution collapses: the defect's origin cannot be determined from the terminal output alone. This holds regardless of model capability — no model compensates for the absence of a per-stage execution trace before scale.

| Condition | Decision |
|-----------|----------|
| N stages; none exercised together | Run one sample through all N before expanding any |
| Stage k output; k+1 … N unvalidated | Block scale of k until k+1 … N are exercised on k's output |
| Stage k scaled to N samples; downstream stages unvalidated | Stop; run single slice through remaining stages first |
| All stages exercised on one sample | Horizontal scale permitted, with per-stage spot-checks |

## Evidence

### Academic Papers

- **Which Agent Causes Task Failures and When? On Automated Failure Attribution of LLM Multi-Agent Systems** (arXiv:2505.00212, May 2025, ICML 2025 Spotlight) — Constructs the Who&When dataset from 127 LLM multi-agent systems and reports that the best automated attribution method achieves 53.5% accuracy identifying the responsible agent and only 14.2% accuracy identifying the responsible step. Failure attribution from execution logs is empirically intractable without prior knowledge of which stage was independently validated. This paper establishes the severity of the attribution collapse this principle prevents: if stage-by-stage validation was not done before scale, post-hoc attribution is no better than guessing on step identity.

- **Why Do Multi-Agent LLM Systems Fail?** (arXiv:2503.13657, Mar 2025, Berkeley/CMU/Stanford) — Introduces MAST, the first Multi-Agent System Failure Taxonomy (14 failure modes across 1,600+ annotated traces, κ=0.88). Three of the top categories — system design issues, inter-agent misalignment, and task verification failures — map directly to what happens when stages are not individually exercised before scaling: design defects in stage k propagate as misalignment into stage k+1, and task verification failures surface only at the terminal stage. The taxonomy corroborates that end-to-end validation must precede scale to make individual-stage failures diagnosable.

- **CausalFlow: Causal Attribution and Counterfactual Repair for LLM Agent Failures** (arXiv:2605.25338, May 2026) — Models execution traces as sequential dependent steps and computes Causal Responsibility Scores (CRS) via counterfactual intervention to localize the failure-inducing step. CausalFlow is the post-hoc remediation tool; this principle is the preventive complement: run the trace once before scale so that if counterfactual analysis is needed, the trace is short and the intervention has a ground-truth baseline against which to measure a repair. The paper's framing — "causal attribution is necessary for reliable improvement" — implies that without a prior validated end-to-end trace, counterfactual intervention has no baseline.

- **Rethinking Failure Attribution in Multi-Agent Systems: A Multi-Perspective Benchmark and Evaluation** (arXiv:2603.25001, Mar 2026) — Argues that MAS failures "often admit multiple plausible attributions due to complex inter-agent dependencies and ambiguous execution trajectories." The ambiguity is structurally produced when inter-stage dependencies are not isolated before scale: if no single clean trace exists, attribution cannot distinguish stage-local defects from propagated upstream errors. This paper's multi-perspective attribution paradigm is a direct response to the attribution collapse this principle prevents.

### Official Best Practices

- **Anthropic Engineering Blog: End-to-End Validation in Long-Running Agent Systems** (anthropic.com/engineering, 2025) — Describes a two-agent system where the pipeline "run[s] a basic test on the development server to catch any undocumented bugs" before the coding agent is invoked on new features. The stated finding is that "Claude tended to make code changes and do unit testing, but would fail to recognize that features didn't work end-to-end" — confirming that per-stage unit passes do not substitute for an end-to-end slice. End-to-end testing via browser automation was required to expose integration failures invisible from code alone.

- **Anthropic "Building Effective Agents"** (https://www.anthropic.com/research/building-effective-agents) — Recommends "start[ing] with the simplest possible implementation and only add complexity when demonstrated through testing." This is the breadth-after-depth prescription: expand the pipeline scope only after the simpler baseline is proven end-to-end.

- **OpenAI "A Practical Guide to Building Agents"** (https://openai.com/business/guides-and-resources/a-practical-guide-to-building-ai-agents/) — Prescribes canary releases (roll out to small controlled groups before full-scale deployment) and shadow mode testing (run new agent behaviors alongside production before replacing them). Both are instances of this principle: confirm the full pipeline works on a narrow slice before committing to full scale.

### Engineering Precedent

- **"Walking Skeleton"** (Alistair Cockburn; popularized in *97 Things Every Software Architect Should Know*, O'Reilly) — A thin, end-to-end implementation that exercises all communication paths with minimal functionality. The prescription: "implement incrementally, adding end-to-end functionality" while "keeping the system running" — not building one layer fully before wiring in the next. The walking skeleton is the software architecture antecedent of this principle: breadth across layers precedes depth within any layer. Reference: https://www.oreilly.com/library/view/97-things-every/9780596800611/ch60.html

### Named in Literature?

No unified name exists in AI agent literature. The closest named concepts are:

- **"Walking Skeleton"** (Cockburn, 1996; software architecture) — thin end-to-end implementation before layer expansion
- **"Vertical Slice"** (Agile development) — delivering a thin but complete path through all system layers before widening any layer
- **"Canary Release" / "Shadow Mode"** (DevOps/MLOps) — expose a small sample to the full pipeline before full deployment
- **"Pilot Run"** (this KB, `pilot-run-before-batch.md`) — single-unit execution of the same operation before batch repetition

This principle names the **cross-stage** constraint that none of the above sources addresses as a single pattern: do not scale stage k until stages k+1 through N have been traversed by at least one output of stage k.

## When to Apply

| Signal | Decision | What breaks if ignored |
|--------|----------|------------------------|
| Building a new multi-stage pipeline | Run one sample through all stages before writing any batch loop | First time a real incompatibility is discovered is after N samples have been produced by stage k — all must be reprocessed or discarded |
| Adding a new stage to an existing pipeline | Run one sample through the new stage before integrating it into batch execution | The new stage's input contract does not match the prior stage's output format; discovered at scale, not at insertion |
| Debugging a pipeline failure with no prior end-to-end trace | Produce a single end-to-end trace first, then diagnose | Attribution ambiguity (arXiv:2505.00212: 14.2% step-level accuracy from logs alone) makes diagnosis intractable |
| Any stage in the pipeline has been recently modified | Re-run the vertical slice before resuming batch scale | The modified stage's output format has potentially shifted; downstream stages fail silently until terminal output is inspected |
| Single-stage pipeline (no downstream consumers) | Do not apply; pilot-run-before-batch suffices | N/A |

Do not apply when:
- The pipeline has only one stage (use `pilot-run-before-batch.md` instead).
- A prior validated end-to-end trace exists from the current execution context on the same pipeline version and input schema.
- The "pipeline" is a pure parallel fan-out with no sequential stage dependencies (use `phase-vs-lane-execution.md` for topology decisions).

**Boundary with related principles:**
- `pilot-run-before-batch.md`: Governs repeating the **same** operation over N inputs. This principle governs traversing **all different stages** once before widening any single one.
- `phase-vs-lane-execution.md`: Governs parallel vs serial dispatch topology. This principle governs **validation ordering before scale**, regardless of topology.
- `evaluation-before-implementation.md`: Governs defining success criteria before building. This principle governs **execution ordering** after criteria exist and a pipeline is already defined.

## Known Limits

- **Vertical slice does not guarantee batch stability.** One passing sample does not prove that stage k produces well-formed output for all inputs. After the slice is proven, a spot-check batch (5–10 samples) must follow before full-scale expansion, consistent with the guidance in `pilot-run-before-batch.md`.

- **Stage output contracts must be defined before the slice is run.** If there is no explicit schema or acceptance criterion for each stage's output, the slice cannot be declared "proven" — it merely ran without crashing. Combine with `evaluation-before-implementation.md` to ensure contracts exist before the slice is attempted.

- **Normalization and format issues do not surface on a single sample.** Edge cases in input encoding, character sets, or schema variation pass a single slice and fail at scale. The principle establishes a minimum floor, not a ceiling.

- **Not a substitute for observability.** Even with a passing slice, stage-level logging must be preserved for batch runs. The slice proves the pipeline can complete end-to-end; observability is required to detect regressions during scale.

- **Scope of application: sequential-stage pipelines only.** In pipelines where the stage graph is a DAG with conditional branching, a single vertical slice does not exercise all branches; one slice per branch is the correct extension. This principle does not apply to pure parallel fan-out topologies where stages have no sequential dependencies on each other's output — in those topologies, `pilot-run-before-batch.md` governs each branch independently.

## Promotion History

Candidate — created 2026-06-19. Derived from a pipeline retrospective in which expansion before end-to-end slice caused failure attribution collapse: hundreds of normalized samples accumulated before it was discovered that a verify stage had been silently rejecting inputs due to a schema mismatch introduced in the normalize stage. No single end-to-end trace had been run before batch execution began.

2026-06-19: Promoted to confirmed/execution/ via 4-phase refinement pipeline. Spec gate: PASS (all 6 criteria, Principle exactly 200 words). Prose gate: APPROVED (0 issues).
