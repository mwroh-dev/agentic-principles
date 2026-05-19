# Prompt Ambiguity As Root Cause

Status: confirmed
Last reviewed: 2026-05-19

## Principle

When an orchestrator or subagent produces output that appears arbitrary, inconsistent, or outside the intended scope, audit the input prompt for judgment room before concluding the model has malfunctioned. Underspecified prompts — those that omit required constraints, leave evaluation criteria implicit, or rely on context the model cannot access — transfer decision authority to the model by default. The model does not fail; it exercises the latitude the prompt granted.

This distinction matters operationally: a model fix targets the wrong layer. Reformulating the prompt to make every required decision explicit eliminates the unwanted variance at its source.

Three mechanisms link input ambiguity to unpredictable output: (1) models fill unspecified arguments by generating plausible defaults, which may not match the caller's intent; (2) underspecified prompts regress significantly more often across model or prompt changes than fully specified ones; (3) models exhibit low uncertainty when processing ambiguous sentences, so they do not surface the ambiguity — they resolve it silently.

Treat prompt underspecification as a reliability defect, not a model capability limitation. The corrective action is prompt revision, not model substitution.

## Evidence

### Academic Papers

- **"What Prompts Don't Say: Understanding and Managing Underspecification in LLM Prompts"** (arXiv:2505.13360, May 2025, CMU / UW) — Shows that underspecified prompts are 2× more likely to regress across model or prompt changes than fully specified ones, with accuracy drops exceeding 20 percentage points. Models infer missing requirements in ~41% of cases but this behavior is fragile and non-transferable.

- **"Learning to Ask: When LLM Agents Meet Unclear Instruction"** (arXiv:2409.00557, August 2024, EMNLP 2025) — Demonstrates that LLM agents presented with ambiguous tool-use instructions do not flag the ambiguity; they silently hallucinate plausible argument values. The paper introduces the NoisyToolBench benchmark and the Ask-when-Needed (AwN) framework, which substantially improves safety and accuracy by triggering clarification requests instead of assumption generation.

- **"Do Pre-Trained Language Models Detect and Understand Semantic Underspecification? Ask the DUST!"** (arXiv:2402.12486, February 2024, ILLC Amsterdam) — Empirically shows that models exhibit little uncertainty when processing semantically underspecified sentences, contrary to what the ambiguity would theoretically warrant. Models identify underspecification when explicitly asked but fail to surface it unprompted — matching the mechanism that makes ambiguous prompts produce confident yet arbitrary outputs.

- **"A Taxonomy of Prompt Defects in LLM Systems"** (arXiv:2509.14404, September 2025) — Classifies recurring prompt failures across six dimensions; Specification and Intent ambiguity is identified as the leading defect category causing models to misinterpret intended objectives and produce undesired outputs. Frames unclear specification as a software engineering defect requiring systematic mitigation.

### Official Best Practices

- **Anthropic — "Effective Context Engineering for AI Agents"** (https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) — States directly: "If a human engineer can't definitively say which tool should be used in a given situation, an LLM can't be expected to do better." Prescribes that prompts must be specific enough to guide behavior effectively and that tool definitions must be "extremely clear with respect to their intended use." Identifies vague high-level guidance as a named failure mode.

- **OpenAI — "Prompt Engineering"** (https://platform.openai.com/docs/guides/prompt-engineering) — Prescribes explicit definition of ambiguity behavior: "when to ask, abstain, or proceed." Instructs that models respond best when prompts specify context, outcome, format, and style rather than relying on model inference. Frames clarity as a prerequisite for reliable outputs, not an optimization.

### Named in Literature?

No unified name. Closest named concepts: *prompt underspecification* (arXiv:2505.13360), *specification ambiguity* (arXiv:2509.14404), *unclear instruction* (arXiv:2409.00557). The specific formulation — that apparent model arbitrariness is traceable to prompt-level judgment room and the corrective action is prompt revision not model change — is implied across sources but not named as a single pattern.

## When to Apply

Apply when:
1. A model output is unexpectedly inconsistent, out-of-scope, or arbitrary across runs with the same prompt
2. An agent silently fills in a parameter, makes a scope decision, or resolves a conflict without flagging it
3. A prompt contains subjective quality terms (e.g., "appropriate," "natural," "reasonable") without operational definitions
4. A prompt delegates selection among multiple valid paths without specifying the selection criterion
5. Debug effort has focused on model version or temperature tuning without first auditing prompt specificity

Do not apply when: the prompt is fully specified and the model's output is demonstrably inconsistent with explicit instructions — in that case the issue is instruction-following capacity or context-length degradation, not underspecification.

## Known Limits

- **Specification overhead vs. flexibility tradeoff**: Fully eliminating judgment room can produce brittle prompts that break on inputs slightly outside the anticipated distribution. Prompts require a specificity level calibrated to the task's variance — not maximal, not minimal.

- **Latent underspecification is hard to detect statically**: A prompt may appear well-specified but contain implicit assumptions that only surface at runtime under specific inputs. Regression benchmarks covering edge-case inputs are required to identify latent underspecification before production.

- **Does not apply to capability gaps**: If the required output depends on knowledge, reasoning, or tool access the model does not have, prompt revision cannot compensate. This pattern addresses variance from ambiguity, not variance from missing capability.

## Promotion History

Candidate — created 2026-05-19 from user insight (source: https://www.minwoo.cloud/blog/korean-prompt-corrector).
Confirmed — promoted 2026-05-19, scored 19/25 by user decision.
