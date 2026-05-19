# Harness Hill-Climbing via Regression Suite

Status: confirmed
Last reviewed: 2026-05-18

## Principle

**Loop:**
```
eval fail → read trace → classify failure mode → patch ONE harness element → re-run eval → regression check
```

A harness is any component that wraps agent execution: prompt, tool surface, middleware, context management, or retrieval policy. Each iteration moves exactly one variable; the eval score is the objective function. This is a discrete hill-climbing algorithm applied to agent behavior.

Patchable elements (one per iteration): prompt, tool definition, middleware, retrieval policy, human-review threshold.

**Atomicity rule:** Changing exactly one element per iteration is the method, not a guideline. One change enables causal attribution — you know what caused the improvement. Changing more than one element per iteration is an error, not a shortcut. Multi-element patches produce confounded results that cannot be trusted as signal for future iterations.

**Prerequisite:** An established eval suite with known baseline scores. Without a regression baseline, no iteration produces meaningful signal.

## Evidence

### Academic Papers

- **Meta-Harness: End-to-End Optimization of Model Harnesses** (arXiv 2603.28052, Mar 2026, Stanford) — Implements this exact pattern. Exposes full execution traces to a proposer that makes targeted harness edits atomically: system prompts, tool definitions, completion-checking logic, or context management — one at a time. Achieves 76.4% pass rate on SWE-bench-style tasks, +7.7 points over baseline with 4x fewer tokens. The atomicity constraint is a core justified design choice empirically validated in this work.
- **Evaluation-Driven Development and Operations of LLM Agents (EDDOps)** (arXiv 2411.13768, Nov 2024) — Names the broader loop "EDDOps." Embeds evaluation as a continuous governing function — not a terminal checkpoint — unifying offline (dev-time) and online (runtime) evaluation in a closed feedback loop with explicit "evaluate → identify gap → patch → regression" cycles. Applies to any agent runtime.
- **Agent Harness for LLM Agents: A Survey** (Preprints.org, Apr 2026) — Formal definition using labeled-transition-system semantics, distinguishing safety and liveness properties. Analyzes 23 systems, 110+ papers. Frames harness engineering as the dominant leverage point for agent performance.

### Official Best Practices

- **Anthropic "Effective Harnesses for Long-Running Agents"** (anthropic.com/engineering/effective-harnesses-for-long-running-agents) — Prescribes incremental-progress pattern; each session leaves clear artifacts for the next. Structurally identical to "patch ONE element, check regression."
- **Anthropic "Equipping Agents for the Real World with Agent Skills"** (anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills) — Prescribes: identify gaps by running agents on representative tasks → observe failure → build skills to address specific shortcomings → scale. Anthropic's sanctioned hill-climbing loop.
- **Anthropic "Building Effective Agents"** — Recommends building incrementally and running evals programmatically. Explicitly warns against over-engineering before establishing eval baselines.

## When to Apply

**Prerequisite:** An established eval suite with baseline scores exists. Without a baseline, no iteration produces meaningful signal.

| Signal | Decision | What breaks if ignored |
|--------|----------|------------------------|
| A specific class of inputs consistently fails and the failure trace is readable | Enter the loop; patch exactly one harness element and re-run eval | Multi-element patches produce confounded results; causal attribution becomes impossible |
| Agent performance degrades toward a measurable target (policy compliance rate, hallucination rate) | Select the single harness element most likely responsible; patch atomically | Without the single-change constraint, regression checks cannot isolate which change caused improvement or regression |
| Failure mode is attributable to a single harness element (prompt drift, tool schema mismatch, retrieval noise, threshold miscalibration) | Classify the element, patch it, verify with regression suite before continuing | Misidentifying a multi-root failure as single-element produces partial fixes and misleads future iterations |
| Failure mode spans multiple harness elements simultaneously | Do not enter the loop; fix the fundamental spec gap first, then re-enter | Single-element patching on a multi-root failure masks the real problem and produces false improvement signals |

## Known Limits

- One-at-a-time patching is slow for severe failures across many dimensions. Prioritize by failure frequency: fix the highest-frequency failure mode first.
- Regression suites become stale as agent behavior evolves. Eval cases that were edge cases at baseline become common paths over time. Audit the eval suite after every 10 iterations or major capability change.
- Atomicity becomes impractical when a single failure mode spans multiple harness elements simultaneously — in that case, fix the fundamental spec gap first, then re-enter the loop.
- This pattern does not apply when the agent system lacks readable execution traces. Without trace observability, failure classification is impossible and the loop degenerates into random search.

## Promotion History

Candidate — 2026-05-18.
Promotion-ready — strong paper backing (Meta-Harness + EDDOps + Survey) + Anthropic official docs. Pattern is partially named in literature ("EDDOps"). Atomic-change rule is empirically validated in Meta-Harness (arXiv 2603.28052).
