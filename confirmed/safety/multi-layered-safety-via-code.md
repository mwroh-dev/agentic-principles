# Multi-Layered Safety via Code, Not Prompting

Status: confirmed
Last reviewed: 2026-05-18

## Principle

**Prompt instructions cannot enforce safety policy. Code hooks can.**

The 4-layer safety model (applied in order, each layer catching what the layer above misses):

| Layer | Definition | What it catches | What slips through |
|-------|-----------|-----------------|-------------------|
| **Role** | Permission boundary — what the agent is allowed to do | Out-of-scope capability invocations | In-scope actions that violate policy |
| **Gate** | Scope check — is this action within the permitted scope? | Actions outside declared scope | In-scope but policy-violating actions |
| **Rule** | Policy check — does this action comply with declared rules? | Policy violations for in-scope actions | Runtime conditions not covered by declared rules |
| **Hook** | Runtime enforcement — code-level block or allow, deterministic | Everything the upper layers miss; must not rely on LLM judgment | Nothing, given an adequately specified Role layer. An underspecified Role layer propagates failures that Hook cannot catch — see Known Limits. |

Safety enforcement lives in middleware and hooks, not in the orchestrator's prompt. The orchestrator layer is kept thin by design. Each layer is independently deployable and testable. The Hook layer is the only layer that is deterministic; all layers above it involve judgment that an adversarial input can influence.

The pattern is not named in literature as a unit; closest named concepts are "runtime enforcement" (AgentSpec, 2025) and "layered guardrails" (LlamaFirewall, 2025).

## Evidence

### Academic Papers

- **AgentSpec: Customizable Runtime Enforcement for Safe and Reliable LLM Agents** (arXiv 2503.18666, 2025) — Achieves >90% prevention of unsafe code executions and 100% AV compliance with millisecond overhead. Safety constraints expressed as structured executable rules (triggers + predicates + enforcement mechanisms) hooking into the decision pipeline at key stages: before action (AgentAction), after observation (AgentStep), at completion (AgentFinish). Closest direct match to the Role→Gate→Rule→Hook architecture.
- **LlamaFirewall** (arXiv 2505.03574, Meta AI, May 2025) — Three independent code-level layers deployed in production: PromptGuard 2 (jailbreak detector), Agent Alignment Checks (chain-of-thought auditor), CodeShield (static analysis of generated code before execution). Directly instantiates "multiple independent gates" at scale.
- **AGrail: A Lifelong Agent Guardrail** (arXiv 2502.11448, Feb 2025) — Static prompt-based safety rules degrade over time; guardrails must be code-level, lifelong, and adaptive, with safety checks generated and optimized as executable tools — not prompts.
- **AIR: Improving Agent Safety through Incident Response** (arXiv 2602.11749, Feb 2026) — Attaches code hooks at two precise execution points in the agent loop — after each tool invocation and before each step — explicitly not via system prompts.
- **Design Patterns for Securing LLM Agents against Prompt Injections** (arXiv 2506.08837) — Constraining agent actions to prevent solving arbitrary tasks via code-level scope restriction significantly reduces prompt injection attack surface.

### Official Best Practices

- **Anthropic "Building Effective Agents"** (anthropic.com/research/building-effective-agents) — Safety frameworks require "securing agents' interactions" as a structural principle. Trust-level enforcement is implemented in code: subagents receive only minimum permissions; human-in-the-loop gates are triggered by code conditions.
- **Anthropic "Framework for Developing Safe and Trustworthy Agents"** (anthropic.com/news/our-framework-for-developing-safe-and-trustworthy-agents) — Official position: safety requires structural enforcement, not prompt-level instructions alone.
- **LangChain Guardrails** (docs.langchain.com/oss/python/langchain/guardrails) — Constitutional AI surfaced as a LangChain chain (code), not a system prompt. Pipeline model: input validation → execution sandboxing → output filtering, all as code.

## When to Apply

| Signal | Decision | What breaks if ignored |
|--------|----------|------------------------|
| Agent has compliance requirements (HIPAA, GDPR, SOC2, internal policy) | Implement Rule layer with code-level policy checks; do not rely on prompt instructions | Prompt-based guardrails fail under adversarial inputs; compliance violations go unblocked |
| Agent can take irreversible actions (send messages, delete data, call external APIs) | Implement Gate + Hook layers before every irreversible action | Without gate enforcement, unrecoverable actions execute when scope checks are bypassed; no rollback path exists |
| Audit finding shows agent bypassed a safety check via prompt manipulation | Migrate enforcement to Hook layer (deterministic code); remove prompt-based safety rules for that check | Jailbreak-class inputs bypass prompt-only defenses indefinitely; the bypass vector remains open |
| Role layer is vague or incompletely specified | Define Role layer permissions explicitly before adding Hook rules | Hook layer cannot compensate for undefined permissions; scope ambiguity propagates uncorrected through all downstream layers |

## Known Limits

**Latency**: The Hook layer adds latency to every action. High-frequency agents require performance profiling after hook implementation. AgentSpec reports millisecond overhead — acceptable for most workloads, but profiling is required before deployment in latency-sensitive pipelines.

**Deployment rigidity**: Code-level hooks require a deployment cycle to change. Prompt-based rules change without deployment. This rigidity is intentional for safety-critical rules, and the tradeoff — slower iteration in exchange for tamper-resistance — is by design, not a defect.

**Role layer dependency**: Hooks compensate for ambiguous scope or missing rules. Hooks do not compensate for a fundamentally underspecified permission model at the Role layer. An incomplete Role definition propagates uncorrectable failures through all downstream layers.

**Advisory agents**: Does not apply to purely advisory agents with no tool execution capability — enforcement hooks add overhead without benefit.

## Related: Skill Discovery as a Permission Surface

Prompt instructions cannot enforce skill authorization, just as they cannot enforce any other safety policy. A skill that is discoverable by the model is a capability surface; agent-specific exclusivity must be enforced at the Role layer (tool grants), the Gate layer (discovery scope), or the Hook layer (caller-aware runtime gateway).

See [`../architecture/skill-discovery-is-not-authorization.md`](../architecture/skill-discovery-is-not-authorization.md).

## Promotion History

Candidate — extracted from literature review, 2026-05-18.
Confirmed — strong independent backing (5 papers including production systems at Meta) + Anthropic + LangChain official. Pattern named as "runtime enforcement" and "layered guardrails" across multiple independent implementations. Quality gate passed 2026-05-18.

## Properties to Restore

위반이 감지됐을 때, 수정된 결과가 만족해야 할 속성들이다.
구체적인 구현 방법은 컨텍스트에 따라 달라질 수 있다.

| Violation detected | Properties the fix must satisfy |
|--------------------|--------------------------------|
| Safety policy enforced only via prompt instructions | The corrected enforcement mechanism must operate as deterministic code that activates regardless of what the LLM produces; prompt instructions alone must not constitute the sole enforcement of any safety-critical rule |
| Safety rule bypassable under adversarial or degraded LLM state | The corrected enforcement mechanism must activate on the action itself, not on the LLM's description or intent; a jailbroken or degraded model must not be able to suppress the enforcement by producing different output |
| No safety-critical rule enforced by code hooks | The corrected design must have at least one deterministic code-level enforcement point for every safety-critical rule; the enforcement must not depend on LLM judgment at the point of blocking |
| Role layer underspecified, causing scope ambiguity to propagate through downstream layers | The corrected Role layer must define permitted actions with enough precision that any in-scope action is distinguishable from any out-of-scope action without requiring LLM interpretation at enforcement time |
