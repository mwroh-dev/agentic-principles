# Agent Instruction Verification Requires Two Tiers

Status: confirmed
Last reviewed: 2026-06-08

## Principle

Agent instruction verification operates on two distinct tiers, each requiring a different class of tool.

**Tier 1 — Structural (automatable):** Binary facts about document existence and form. File is present, schema parses, required sections exist, reference paths resolve. These are machine-decidable. Automated checks are the correct and complete tool for this tier.

**Tier 2 — Quality (judgment-required):** Behavioral claims about instruction correctness. Policy is accurately stated, responsibility boundary is well-defined, instruction is unambiguous to a model reader. These are not binary. Model review or human review checklist is the correct tool for this tier.

Applying Tier 1 tools to Tier 2 questions is not a wrong implementation — it is the wrong class of tool. A text-pattern assertion (`assert.match(doc, /phrase/)`) does not check behavioral quality; it checks text content. These are different questions. The assertion can pass when the policy is wrong (phrase present, wrong context) and fail when the policy is correct (phrase absent, correct alternative wording). Both error directions are invisible to the pattern matcher.

The agent-system-specific cost of this category error is architectural. Content-matching checks treat document internals as a CI contract. Any refactoring that changes wording — moving a policy to its correct owner, splitting a monolithic prompt, renaming a section — breaks CI. The check thereby prevents the structural evolution that reduces prompt interference:

```
content-matching validator
       ↓ enforces
monolithic prompt architecture (all policy in one document)
       ↓ causes
prompt interference (Arbiter: 152 instances across 3 production systems)
       ↓ produces
agent behavioral failure — silent, without runtime error
```

The correct tier assignment:

| Concern | Tier | Verification method |
|---|---|---|
| Required file exists | Structural | `fs.existsSync(path)` — automated |
| Schema parses | Structural | JSON.parse succeeds — automated |
| Reference paths resolve | Structural | Path check — automated |
| Policy is in correct owner file | Structural | Owner file exists — automated |
| Policy is correctly stated | Quality | Model or human review checklist |
| Instruction clarity | Quality | Model review |
| Responsibility boundary separation | Quality | Model review |
| Section headings are meaningful | Quality | Model review — semantics, not text match |

Note on heading checks: asserting that `## Purpose` section exists is Tier 1 (structural — required section present). Asserting that `## Purpose` contains specific wording is Tier 2 applied with a Tier 1 tool — the same category error.

## Evidence

### Academic Papers

- **Arbiter: Detecting Interference in LLM Agent System Prompts** (arXiv:2603.08993, Mar 2026, Mason) — Finds 152 interference patterns across Claude Code, Codex CLI, and Gemini CLI system prompts. Establishes that prompt architecture (monolithic, flat, modular) "strongly correlates with observed failure class." Content-checking validators enforce monolithic architecture by making CI fail on any structural refactoring. This is the direct empirical cost: monolithic architecture produces the highest-interference configuration Arbiter measures.

- **Prompting in the Wild: An Empirical Study of Prompt Evolution in Software Repositories** (arXiv:2412.17298, Dec 2024, MSR 2025, Tafreshipour et al.) — Analyzed 1,262 prompt changes across 243 repositories. Finds that test-coupled prompts receive fewer structural improvements and accumulate literal drift. Establishes test coupling as a maintenance anti-pattern that slows the document evolution needed to maintain instruction quality.

- **When "Better" Prompts Hurt: Evaluation-Driven Iteration for LLM Applications** (arXiv:2601.22025, Jan 2026, Commey) — Establishes that "evaluating LLM applications differs from traditional software testing because outputs are stochastic, high-dimensional, and sensitive to prompt and model changes," and explicitly distinguishes automated checks from human rubrics and LLM-as-judge as distinct evaluation components serving different purposes. Automated checks (Tier 1 in this framework) are appropriate for binary structural facts; behavioral quality evaluation requires non-automated methods. Applying automated checks to quality evaluation is the specific category error this paper's framework is designed to prevent.

- **Shift-Up: A Framework for Software Engineering Guardrails in AI-native Software Development** (arXiv:2604.20436, 2026, Lipsanen et al.) — Distinguishes machine-readable structural artifacts (checkable) from narrative content (not checkable) in AI-native development. Validates that structural verification is automatable; content quality is not.

### Official Best Practices

- **Anthropic — Claude Code Best Practices** (https://code.claude.com/docs/en/best-practices, 2025–2026) — Identifies "over-specified CLAUDE.md" as a named failure pattern: "If your CLAUDE.md is too long, Claude ignores half of it because important rules get lost in the noise." Content-police validators enforce the exact condition this warns against — they prevent the pruning the guidance prescribes by failing CI on any wording change.

- **Anthropic — Building Effective Agents** (https://www.anthropic.com/research/building-effective-agents, 2024) — Prescribes evaluating agents by testing observable behavior, not prompt text internals. Content-matching is structurally opposite: it couples CI to document internals rather than to the agent's externally observable correctness.

### Named in Literature?

No unified name. Closest named concepts: "test coupling" (Tafreshipour et al. 2024), "prompt architecture" (Arbiter 2026), "evaluation tier separation" (arXiv:2601.22025 — automated vs. model/human). The specific formulation — two-tier verification model where structural facts are automatable and quality judgment is not — is not named as a single pattern but is derivable from the combination of Arbiter's architectural finding and 2601.22025's evaluation-type distinction.

## When to Apply

| Signal | Decision | What breaks if ignored |
|---|---|---|
| Test checks `assert.match(doc, /specific phrase/)` for a behavioral rule | Move to Tier 2 review checklist; replace with structural check (owner file exists) | Correct refactoring breaks CI; document architecture freezes; interference accumulates |
| CI fails when a policy is moved from entry prompt to its correct owner file | Validator scope error; fix the validator | Architectural improvement is blocked; monolithic structure is enforced by CI |
| Adding a feature requires updating both the document and the test that checks for its phrases | Test is content-checking; decouple | Features that update documents correctly still require test maintenance |
| Test asserts section headings present (e.g., `## Purpose` exists) | Acceptable — structural check | Section presence is Tier 1; heading content/wording is Tier 2 |
| Validator requires prompt.md to contain a policy that has a correct owner elsewhere | Remove content check; assert owner file exists | Prompt permanently co-owns policy with its canonical owner; both drift independently |

Do not apply when: the check is a genuine negative-space structural boundary — "this public package must not contain any file path matching this internal pattern" is structural. Negative-space structural checks do not create the architecture-lock problem.

## Known Limits

- **Structural validators cannot detect absent policy**: If a policy should exist but its owner file is missing entirely, a Tier 1 check catches this only if it explicitly checks for the owner file. Content validators at least catch "phrase absent." The correct replacement: assign ownership first, then check that the owner file exists. Without ownership assignment, removing content checks creates a monitoring gap.

- **Tier 2 checklists require running a reviewer**: Moving content checks to model/human review shifts the gate from automated to reviewed. For high-frequency CI, this adds latency. Mitigate by running checklist review at PR creation, not at every commit, and by caching results for unchanged documents.

- **Heading presence is a borderline case**: Asserting that `## Purpose` exists is Tier 1 (section present as structure). Asserting that `## Purpose` contains specific text is Tier 2 applied with a Tier 1 tool. The line: section presence = structural, section content = judgment.

- **Removing content checks before fixing ownership creates gaps**: Content-police validators represent implicit policy ownership claims. Remove them only after (1) assigning canonical owner, (2) replacing content check with structural check on owner file, (3) verifying owner contains policy via Tier 2 review.

## Promotion History

Candidate — created 2026-06-07 from observed pattern: validate-skill.mjs and browser-flow-capture.test.mjs each maintained 15+ `text.includes()` checks against prompt.md, making structural document refactoring impossible without modifying unrelated test fixtures.

Revised 2026-06-08 (candidate): Reframed from "validator implementation guideline" to causal chain (content-police → monolithic architecture → interference). Coreiness argument strengthened via Arbiter's empirical finding.

Confirmed — 2026-06-08. 5-dimension score: 21/25 (코어성 5, 리스크감소 4, 확장성 4, 제어성 4, 기록성 4). Reframed from "automated text matching cannot verify quality" prohibition to "two-tier verification model" as the established pattern. Added arXiv:2601.22025 as direct evidence for evaluation type distinction. 제어성 argument made explicit: tier separation is an oversight architecture decision (CI owns Tier 1, review process owns Tier 2). 기록성 argument made explicit: Tier 1-only validation produces meaningful binary audit logs; content checks produce false audit trails (phrase presence ≠ behavioral correctness).
