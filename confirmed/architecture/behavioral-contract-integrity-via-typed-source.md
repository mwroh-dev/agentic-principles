# Behavioral Contract Integrity via Typed Source

Status: confirmed
Last reviewed: 2026-06-08

## Principle

Role contract families follow the typed-source → generator → artifact pipeline. A typed source definition expresses the behavioral contract once; a generator serializes it into all member artifacts; tests verify the artifacts match the source. This is the agent-domain application of Infrastructure as Code: role configuration — what this agent is and is not authorized to do — is code, not hand-authored text.

For agent behavioral contracts specifically, the pipeline solves a failure mode that conventional IaC does not face. Configuration drift in IaC produces a deployment error or runtime exception — an observable signal. Authority boundary drift in behavioral contracts produces an agent that acts outside its intended scope with no immediate failure signal. The contract continues to be read, execution continues, and the violation is only detectable by observing behavior against intent. The pipeline breaks this drift mechanism at its source: there is no per-member editing surface. The typed definition is the only editing surface. Every member artifact is derived, not authored.

The failure mode is specific to role families. Two runner presets may both contain a `forbiddenResponsibilities` field. If the field content is hand-edited free text, there is no guarantee that the same prohibition is expressed consistently, completely, or at all across members. A model reading one runner preset gets a different behavioral contract than a model reading another. The agents produce inconsistent behavior not because the orchestration logic changed but because the role specification itself drifted without any version gate.

```
typed role contract definitions (definitions.js)
       ↓  generator
role preset artifacts (*.json, prompts/**/*.md)
       ↓  tests
verify artifacts match source — not that specific phrases are present
```

The typed definition is the authoritative behavioral contract. Generated artifacts are serialized contract representations. Tests verify that the serialization ran and matches the source — not that specific text phrases appear in the generated file.

Typed source also enables systematic oversight: a reviewer can inspect the type definition and verify all role family members' contracts are consistent in a single read. Hand-maintained presets require reading each file separately and comparing manually — practically infeasible for role families above three or four members. A change to the typed definition produces one diff that represents the full contractual change. Scattered edits to hand-maintained files produce N diffs with no unified record of the contractual intent.

## Evidence

### Academic Papers

- **Agent Behavioral Contracts: Formal Specification and Runtime Enforcement for Reliable Autonomous AI Agents** (arXiv:2602.22302, 2026) — Defines every agent as a formal contract C=(P, I, G, R): preconditions, invariants, goals, and recovery behaviors. Demonstrates that separating behavioral contracts from execution code enables specification-level verification. The corollary is direct: if the formal specification is maintained as hand-edited free text rather than a typed source, specification-level verification is defeated regardless of the formal contract language used.

- **Prompt Stability Matters: Evaluating and Optimizing Auto-Generated Prompt in General-Purpose Systems** (arXiv:2505.13546, May 2025, Ke Chen et al.) — Demonstrates that prompts generated from stable typed templates exhibit significantly lower behavioral variance than hand-authored counterparts. Behavioral variance is the direct measurement of what happens when role specifications drift: models receiving drifted specifications produce inconsistent behavior across runs.

- **Shift-Up: A Framework for Software Engineering Guardrails in AI-native Software Development** (arXiv:2604.20436, 2026, Lipsanen et al.) — Demonstrates that "embedding machine-readable requirements and architectural artifacts stabilizes agent behavior, reduces implementation drift." Typed, machine-readable artifacts produce more stable agent behavior than narrative equivalents — directly applying to role contract specifications.

- **Contract-based Design and Verification of Multi-Agent Systems with Quantitative Temporal Requirements** (arXiv:2412.13114, 2024) — Proves that individual agent role interfaces can be fully specified and compositionally verified. Compositional verification requires that each agent's contract be expressed in a checkable form — untyped hand-maintained free text is not checkable by the compositional framework the paper proposes.

### Official Best Practices

- **OpenAI Agents SDK — Agent Configuration Reference** (https://openai.github.io/openai-agents-python/agents/, 2025) — The SDK's canonical agent representation is a typed structured object: `name`, `instructions`, `output_type`, `tools`. The SDK architecture implies that agent specifications are typed, not hand-authored free text inline with application code. Per-deployment hand-authoring is the anti-pattern the SDK's structured interface is designed to prevent.

- **Anthropic — Building Effective Agents** (https://www.anthropic.com/research/building-effective-agents, 2024) — Prescribes simplicity and composability as the primary agent design criteria. Composability requires that agent role contracts be consistent and independently verifiable. Hand-maintained free text contracts cannot be composed — the only composable form is a typed definition with a known schema.

### Named in Literature?

No unified name. Closest named concepts: "formal behavioral contract" (arXiv:2602.22302), "typed agent specification" (OpenAI Agents SDK), "machine-readable requirements" (Shift-Up), "Infrastructure as Code" (general software engineering — the IaC pattern is the direct structural analogue). The specific formulation — that agent behavioral contracts must follow a typed-source → generator → artifact pipeline to preserve internal consistency across role family members — is not named as a single pattern.

## When to Apply

| Signal | Decision | What breaks if ignored |
|---|---|---|
| Multiple JSON presets share the same role family but have different phrasing in `forbiddenResponsibilities` | Extract to typed shared-trait definition; generate field from type | Agents in the same role family have inconsistent authority boundaries; misalignment is invisible to schema validator |
| "Why does runner A not mention this prohibition but runner B does?" has no answer | Hand-authoring without consulting existing entries | One agent acts within prohibited scope while another does not; inconsistency not detected until a failure |
| Adding a new role preset requires reading existing presets to "learn" the conventions | Missing typed source; conventions live in existing hand-authored text | New entries silently miss conventions; class-level inconsistency accumulates |
| Tests check specific phrases inside role preset JSON fields | Replace with "generated artifact matches typed source" tests | Tests catch known deviations; cannot catch deviations in untested fields |
| A contract change must be applied to all presets in a role family | Change typed source once; regenerate all artifacts | Manual update misses one or more presets; role family members have inconsistent contracts |

Do not apply when: the behavioral specification is genuinely narrative and non-repeating (a project-specific context explanation, an open-ended goal description unique to one agent). Generation applies to the structural, typed portion of the contract — authority, forbidden responsibilities, required evidence — not to open-ended narrative fields.

Do not apply when: the system has only one agent. This principle addresses consistency across multiple agents in the same role family. A single-agent system has no cross-entry consistency requirement.

## Known Limits

- **Typed source requires upfront schema design**: Before a generator can be written, the behavioral contract schema must be fully specified. For early-stage agent systems where the contract shape is still being discovered, hand-authoring is appropriate. Apply this principle when the contract schema has stabilized and role families have more than three members.

- **Generator staleness**: Committed generated artifacts diverge from typed source if the generator is not run after source changes. CI tests that compare generated output to current source prevent this; developer tooling that auto-runs the generator on source changes eliminates the gap. Without either, typed source and generated contracts drift — the same drift problem the principle is designed to prevent.

- **Type system cannot express all behavioral nuance**: Some behavioral constraints are context-dependent in ways that a typed field cannot fully capture. In these cases, the generator produces a typed template for the repeatable portions and leaves designated open sections for hand-authoring. The principle still applies to the typed portions; open sections should be named explicitly in the schema so their presence is required by the schema validator.

## Promotion History

Candidate — created 2026-06-07 from observed pattern: skills/loop-station/presets/roles/*.json had hand-maintained `forbiddenResponsibilities`, `authority`, and `sharedTraits` fields across 9+ entries. Phrasing diverged silently across runner/orchestrator/judgment presets. Tests checking specific text phrases could not detect class-level inconsistency. Resolved by introducing definitions.js → generate.js → artifacts pipeline.

Revised 2026-06-07 (candidate): Reframed from "general build pipeline" principle to "behavioral contract integrity" principle. Coreiness argument strengthened via arXiv:2602.22302 and arXiv:2412.13114.

Confirmed — 2026-06-08. 5-dimension score: 20/25 (코어성 4, 리스크감소 4, 확장성 4, 제어성 4, 기록성 4). Reframed from "don't hand-maintain JSON" prohibition to "typed-source → generator → artifact IS the pipeline pattern" (IaC analogy as named structural precedent). 제어성 argument made explicit: typed source is the only form auditable by a single-read review across the full role family. 기록성 argument made explicit: one-diff contract change vs. N-diff scattered edits.
