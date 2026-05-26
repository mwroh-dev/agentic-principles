# Schema-Valid Is Not Runtime-Executable

Status: confirmed
Last reviewed: 2026-05-22

## Principle

**A schema-valid LLM output is not an executable output. They are different properties that must be checked by different gates.**

An LLM output passes through four sequential gates before it may be executed:

| Gate | What it checks | Failure means |
|------|---------------|---------------|
| **Shape** | JSON schema compliance, required fields, type constraints | Output cannot even be parsed as the intended structure |
| **Semantic** | Vocabulary membership, constraint non-contradiction, legality in the domain's declared vocabulary | Output is structurally valid but semantically incoherent or uses undefined terms |
| **Fact** | External resource reconciliation — does the referenced sheet/column/node/entity actually exist in the current state? | Output references things that do not exist |
| **Supportability** | Can the current system's composition or primitive layer actually execute this operation? | Output is self-consistent but requires a combination or lowering path the system does not support |

Each gate is a hard stop. A gate failure does not trigger automatic correction. The output is classified as `blocked` or `needs_human_checkpoint`. The orchestrator does not self-repair.

**Gates run at the orchestrator dispatch boundary — before subagents are invoked.** A gate failure at any step prevents dispatch. The subagent never receives an unvalidated input. Placing gates inside subagents after dispatch is the wrong location: the subagent inherits the bad spec as fact, every downstream step pays token cost on bad input, and cascading failures compound. The pre-dispatch position is enforced architecturally, not by convention.

The failure mode this pattern prevents: treating a schema-valid output as an execution contract, then dispatching to subagents where runtime failure surfaces too late. Schema validation catches parser-level errors. It does not catch hallucinated column names, unsupported compositions, or contradictory constraint sets. Operating without gates 2–4 causes silent execution of invalid plans that produce correct-looking artifacts with incorrect semantics.

**Relation to `rule-is-not-skill.md`**: Each gate is a rule, not a skill. It fires deterministically regardless of orchestrator state. The orchestrator does not decide whether to apply a gate.

## Evidence

### Academic Papers

- **Neuro-Symbolic Verification on Instruction Following of LLMs (NSVIF)** (arXiv:2601.17789, Jan 2026) — Formulates output verification as a constraint-satisfaction problem that distinguishes *logical constraints* (hard rules: format, vocabulary, prohibited tokens — shape + semantic gates) from *semantic constraints* (meaning alignment — semantic gate). A unified solver orchestrates both independently. Provides interpretable feedback when either class fails. Directly validates the distinction between structural and semantic validity as separate checkable properties.

- **Talk Less, Verify More** (arXiv:2601.00224, 2026) — Demonstrates that reverse translation of generated structured output back to natural language catches semantic misalignment that schema validation misses entirely. The paper distinguishes: semantic validity (does the output mean what the user intended?) from executability (does it run?). Validates semantic gate as a distinct and necessary step.

- **ToolGate: Contract-Grounded and Verified Tool Execution for LLMs** (arXiv:2601.04688, Jan 2026) — Formalizes each tool call as a Hoare-style contract with preconditions and postconditions that check current world state. Precondition = fact gate: does referenced entity exist in verified state? Postcondition = state reconciliation after execution. Directly validates the fact/existence gate as the conceptual bridge between schema-valid and runtime-executable.

- **Verify Before You Fix** (arXiv:2604.10800, Apr 2026) — States the execution-grounding invariant explicitly: no action may be taken without execution-based confirmation. Validates Gates 3 and 4 jointly: *"predictions are probabilistic inferences, not verified conclusions, and acting on them without grounding in observable evidence leads to compounding failures across downstream stages."*

- **VeriGuard** (arXiv:2510.05156, Google Research, Oct 2025) — Dual-stage architecture: offline policy synthesis and verification (Gates 1–3) + online action monitoring against a pre-verified capability envelope (Gate 4). The online gate is a lightweight supportability check against what the current system has been verified to support. Validates the supportability gate as architecturally distinct from semantic and fact gates.

- **AgentFixer** (arXiv:2603.29848, IBM Research, ICSE 2026) — 15-tool diagnostic framework spanning schema checks (Gate 1), semantic anomaly detection and reasoning-action alignment (Gate 2), factual consistency and cross-stage coherence (Gate 3), and execution-grounded output validation (Gate 4). Production system, not theory.

- **Architecting Resilient LLM Agents: Plan-Validate-Execute** (arXiv:2509.08646, Sep 2025) — Formalizes the Plan-Validate-Execute (P-V-E) pattern: a dedicated Verifier agent sits between the Planner and the Executor. The Executor never receives unvalidated planner output. Directly establishes pre-dispatch validation as a named, independently deployable architectural layer applicable to LangChain, CrewAI, and AutoGen.

- **VeriMAP: Verification-Aware Planning for Multi-Agent Systems** (arXiv:2510.17109, Oct 2025) — Planner defines verification criteria (Python subtask verification functions) before subagents are dispatched. The gate is encoded at planning time, not discovered after execution fails. Validates that pre-dispatch gate specification is the orchestrator's responsibility.

- **Why Do Multi-Agent LLM Systems Fail? (MAST)** (arXiv:2503.13657, Mar 2025) — Empirical study of 200+ tasks across 7 MAS frameworks. Identifies specification failures (unclear task specs, role/constraint violations dispatched to subagents) as one of three root failure categories. Intervention: improved orchestration-level specification → +14% improvement. Provides empirical evidence that failures root-cause to the orchestrator dispatch boundary, not inside subagents.

- **Trajectory Cost Reduction in LLM Agents** (arXiv:2509.23586, 2025) — Documents the snowballing token cost problem: when a subagent receives a bad input (oversized context, wrong tool, incorrect spec), every subsequent step in the trajectory amplifies the cost. Pre-dispatch validation eliminates the snowball at its origin.

### Official Best Practices

- **OpenAI Structured Outputs** (developers.openai.com/api/docs/guides/structured-outputs) — Official statement: *"While Structured Outputs ensures the response conforms to your schema format, you still need to validate that email fields contain valid emails, dates fall within expected ranges, and numeric values are reasonable."* Explicit vendor acknowledgment that schema compliance (Gate 1) does not cover semantic validity (Gate 2) or domain correctness (Gates 3–4).

- **OpenAI Agents SDK — Guardrails** (openai.github.io/openai-agents-python/guardrails/) — Input guardrails run only on the first agent in the pipeline (orchestrator entry point) before any handoff to subagents. If the guardrail tripwire fires, the subagent never executes. This is SDK-level architectural enforcement of pre-dispatch gating: the framework physically prevents dispatch on gate failure.

- **Anthropic Strict Tool Use** (platform.claude.com/docs/en/agents-and-tools/tool-use/strict-tool-use) — Official statement: *"The models can and may still hallucinate occasionally, so you might get perfectly formatted incorrect answers."* Same position as OpenAI: structural guarantee only, not semantic or domain correctness.

- **Anthropic "Building Effective Agents"** (anthropic.com/research/building-effective-agents) — Orchestrator defines the four-part contract (objective, output format, tool guidance, task boundaries) before dispatch. Safety is "embedded at the infrastructure level"; minimal footprint and trust hierarchy are orchestrator-level pre-dispatch constraints, not subagent-level checks.

- **PydanticAI Output Validators** (ai.pydantic.dev/output/) — Provides Gate 1 (schema validation) and `RunContext`-aware validators that can reach external state for Gate 3 (existence checks). The framework explicitly provides two separate validator hooks for these two concerns, validating that they require different mechanisms.

### Engineering Analogies

- **Kubernetes Admission Controllers** — API requests pass through *mutating* admission (shape normalization, Gate 1) then *validating* admission (policy enforcement, Gate 2) as two sequential, ordered phases. A structurally valid object can be rejected by the validating phase. Direct structural precedent for ordered, sequential validation gates.

- **MLIR Verifier Pattern** — Parse (shape gate) → structural verifier (semantic + type consistency gate) → per-operation verifier (fact/constraint gate) → pass precondition check (supportability gate). Verifiers run at every stage boundary. A verified IR can be semantically invalid for a given lowering target — the supportability gate catches this. Described in MLIR official documentation and PLDI 2025 verification dialect paper.

### Named in Literature?

No unified name. Closest named concepts: "execution-grounded verification" (arXiv:2604.10800, arXiv:2604.13120), "multi-stage output validation" (practitioner literature, FutureAGI 2026), "contract-grounded tool execution" (ToolGate). The specific formulation — four ordered gates where Gate 1 pass does not imply Gate 2–4 pass, and failure at any gate → blocked rather than auto-corrected — is consistent across sources but not yet named as a single pattern.

## When to Apply

| Signal | Decision | What breaks if skipped |
|--------|----------|------------------------|
| LLM output is used to drive an action with external effects (write, delete, schedule, submit) | Implement all 4 gates in order; block on any failure | Schema-valid but semantically broken instructions execute silently; state corruption is not immediately visible |
| Output references external entities (columns, nodes, accounts, endpoints) that may not exist | Implement Gate 3 (fact reconciliation) before execution | Referenced resources don't exist; execution fails at runtime with no prior indication |
| Output uses a composition or operation type that may not be supported in the current build | Implement Gate 4 (supportability check) | Runtime crash or degraded execution on a path that looked valid at generation time |
| System generates structured outputs via LLM and the consumers are deterministic code paths | Add Gate 2 (semantic vocabulary check) after schema validation | Undefined terms pass schema, reach code, cause key-not-found or unhandled-case failures |
| Orchestrator dispatches structured plans to subagents in a multi-agent pipeline | All 4 gates must run at the orchestrator dispatch boundary before any subagent is invoked | Subagent inherits bad spec as fact; every downstream step compounds token cost and failure probability on invalid input |

Do not apply when: the LLM output is advisory only (shown to a human, not executed by code) — Gates 3 and 4 add overhead without benefit. Apply Gate 1 and Gate 2 even for advisory outputs when outputs feed downstream LLM calls.

## Known Limits

**Auto-correction temptation**: Blocked outputs create pressure to add auto-correction logic ("just substitute the closest valid column name"). Auto-correction introduces a new failure mode: silent substitution of incorrect semantics. The principle requires resisting this. Blocked = blocked. Correction paths must be explicit and auditable.

**Gate 4 is system-version-dependent**: Supportability is a function of what the current deployed system supports. A Gate 4 pass on version N may fail on version N+1. Supportability checks must be versioned with the system they describe.

**Gate 2 vocabulary size**: The semantic gate is only as strong as the declared vocabulary. Undeclared but valid vocabulary creates false Gate 2 failures. The vocabulary must be maintained as a first-class artifact.

**Performance**: Four gate passes add latency. For high-frequency paths, consider caching Gate 1 and Gate 2 results against a content hash. Gate 3 and Gate 4 must always be fresh (external state can change).

## Promotion History

Candidate — created 2026-05-22 from user insight (orchestrator skill 2개 구축 중 schema-valid spec이 subagent runtime에서 깨지는 경험). Evidence found: NSVIF (2601.17789), ToolGate (2601.04688), VeriGuard (2510.05156), AgentFixer (2603.29848), Verify Before You Fix (2604.10800), Plan-Validate-Execute (2509.08646), VeriMAP (2510.17109), MAST (2503.13657), Trajectory Cost Reduction (2509.23586); OpenAI Agents SDK guardrails (architecturally enforced pre-dispatch gating) + Anthropic official; Kubernetes + MLIR engineering analogies. Pattern not yet named as a unit — candidate status maintained.

**5-dimension score: 24/25** (코어성 5, 리스크감소 5, 확장성 5, 제어성 5, 기록성 4)
→ Meets confirmed threshold (19+). Recommend promotion. 확장성 5로 상향: multi-agent dispatch가 늘수록 pre-dispatch gate의 가치가 직접 비례해서 증가함 (MAST, Trajectory Cost 근거).

## Properties to Restore

위반이 감지됐을 때, 수정된 결과가 만족해야 할 속성들이다.
구체적인 구현 방법은 컨텍스트에 따라 달라질 수 있다.

| Violation detected | Properties the fix must satisfy |
|--------------------|--------------------------------|
| Schema-valid output dispatched to subagent without Semantic, Fact, or Supportability gate | All four gates must be traversed in order before any subagent receives the output; a pass on Gate N does not authorize dispatch — Gates N+1 through 4 must still complete |
| Gate failure auto-corrected rather than blocked | A gate failure must result in a blocked state that halts dispatch; auto-correction is not a valid resolution path — any correction must be explicit, auditable, and followed by a full gate re-traversal from the beginning |
| A gate skipped because an earlier gate passed | No gate may be omitted regardless of upstream gate outcomes; each gate checks a distinct property that earlier gates do not cover |
| Validation placed inside subagents after dispatch | All gates must run at the orchestrator dispatch boundary before any subagent is invoked; post-dispatch validation in subagents does not satisfy this property |
