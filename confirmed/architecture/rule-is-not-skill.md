# Rule Is Not Skill

Status: confirmed
Last reviewed: 2026-05-19

## Principle

A **rule** is a declarative constraint enforced uniformly by the runtime or a reference monitor — it fires identically regardless of the orchestrator's decision. A **skill** is a callable procedural module invoked by an orchestrator when a context condition is met — it fires only when chosen.

These two belong in separate layers. Placing them in the same layer causes three failure modes: (1) update cycle collision — rules require governance approval while skills iterate with product velocity; (2) audit scope bleed — auditors must inspect all skills to verify policy coverage when they should only inspect the constraint layer; (3) execution subject confusion — a rule enforced by the LLM is not enforced at all when the LLM is compromised or bypassed.

Separate the layers: the constraint layer (policy engine, schema validator, reference monitor, linter rule) must be able to block agent action without the agent's participation. The skill layer (callable procedures, domain routines, tool wrappers) must not carry enforcement responsibility. If a behavior must hold regardless of LLM state, it is a rule. If a behavior is invoked based on context-matching and can legitimately not fire, it is a skill.

## Evidence

### Academic Papers

- **Policy Compiler for Secure Agentic Systems** (arXiv:2602.16708, Feb 2026, anonymous) — Formalizes agent policies as Datalog-based declarative rules compiled into a reference monitor that intercepts skill execution at the runtime boundary. The reference monitor enforces constraints without LLM participation, making the separation of rule and skill layers an explicit compile-time architectural decision rather than a prompt convention.

- **SoK: Agentic Skills — Beyond Tool Use in LLM Agents** (arXiv:2602.20867, Feb 2026, anonymous) — Defines a skill as a module packaging "applicability condition + execution policy + termination criteria" and explicitly distinguishes the skill layer from the governance and audit layer. The paper states that mixing governance constraints into skill definitions degrades both auditability and skill reusability.

- **Cognitive Architectures for Language Agents (CoALA)** (arXiv:2309.02427, Sep 2023, Stanford / UMass) — Derives from cognitive science that procedural memory (skill routines) and declarative memory (facts and rules) require separate storage with different update mechanisms and access patterns. Procedural memory is updated through practice; declarative memory is updated through authoritative assertion. Conflating the two breaks both update paths.

- **Externalization in LLM Agents** (arXiv:2604.08224, Apr 2026, anonymous) — Frames harness engineering as the governance coordination layer that sits above externalized skills, protocols, and memory. Governance operates at the meta-level, orchestrating components without being embedded in any single skill. Direct quote: "Capabilities that earlier systems expected the model to recover internally are now externalized into memory stores, reusable skills, interaction protocols, and the surrounding harness."

### Official Best Practices

- **Guardrails and Human Review — OpenAI Agents SDK** (https://developers.openai.com/api/docs/guides/agents/guardrails-approvals) — Defines guardrails as a validation layer that operates independently of agent tools: "Guardrails validate input, output, or tool behavior automatically." The documentation prescribes that guardrails attach to execution boundaries rather than being embedded in tool definitions, and explicitly states that tool-level validation must be placed "next to the tool that creates the side effect" — not inside it.

- **Equipping Agents for the Real World with Agent Skills — Anthropic Engineering** (https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills) — Defines skills as "organized folders of instructions, scripts, and resources that agents can discover and load dynamically." The framing is entirely capability-additive: skills extend what an agent can do, not what an agent is prevented from doing. Constraint enforcement is addressed separately through installation trust and human curation, not through skill definitions.

### Named in Literature?

No unified name. Closest named concepts: "reference monitor separation" (arXiv:2602.16708), "declarative vs. procedural memory" (CoALA, arXiv:2309.02427), and "governance layer vs. skill layer" (arXiv:2602.20867, arXiv:2604.08224). The specific formulation — that rules and skills fail when placed in the same architectural layer — is implied consistently across these sources but not named as a single pattern.

## When to Apply

| Condition | Signal | Action |
|-----------|--------|--------|
| A behavior must hold even if the LLM is bypassed or compromised | Enforcement depends on model cooperation | Move to constraint layer (reference monitor, schema validator, linter) |
| A behavior's update cycle is driven by governance/compliance approval | Separate from product iteration velocity | Isolate in policy store with its own review gate |
| An auditor must verify policy coverage without reading all skills | Audit scope must be bounded | Constraint layer must be the single source of enforceable rules |
| A behavior fires only when the orchestrator decides it is applicable | Context-conditional invocation | Keep in skill layer; do not embed enforcement logic |
| A new runtime or agent framework is being designed | Architectural decision point | Define constraint and skill layers before populating either |

Do not apply when: the system has no external harness or enforcement runtime — e.g., a pure prompt-based agent with no code-level interceptor. In that system, the distinction exists conceptually but cannot be enforced architecturally; address this by introducing a harness boundary before applying this pattern.

## Known Limits

- **Bootstrapping cost**: Separating the layers requires a harness or reference monitor that can intercept agent actions. Systems without an existing harness boundary must build one before this pattern is enforceable. This is a one-time infrastructure cost, not a recurring one.

- **Apparent duplication**: A rule in the constraint layer and a matching instruction in a skill may coexist (e.g., "do not write to production DB" as a linter rule AND as a skill instruction). This is intentional defense-in-depth, not redundancy to eliminate. Remove the skill instruction only if the constraint layer provides complete coverage.

- **Rule granularity mismatch**: Declarative constraint languages (Datalog, OPA Rego, JSON Schema) express universal predicates well but struggle with context-sensitive policies (e.g., "allowed only when the user has role X and the request came from IP range Y"). When a policy is inherently context-sensitive, it may require a hybrid: a skill-callable policy evaluation function invoked by the orchestrator, with the actual enforcement still handled by the constraint layer via the function's output. Do not embed the enforcement decision in the skill itself.

- **Non-applicable domain**: This pattern does not apply to systems where the agent runtime has no distinction between enforcement and invocation — for example, single-turn prompt completions with no tool calls or external harness. Apply this pattern only when the agent system has at least one programmable execution boundary where constraints can be checked independently of model output.

## Promotion History

Candidate — created 2026-05-19 from user insight.
Confirmed — promoted 2026-05-19, score 21/25 (코어성 4, 리스크감소 5, 확장성 4, 제어성 5, 기록성 3).
