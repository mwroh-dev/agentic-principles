# Agent As Contract Not Code

Status: confirmed
Last reviewed: 2026-05-19

## Principle

A **role contract** is a declarative specification that defines an agent's preconditions, invariants, goals, recovery behavior, allowed tools, and failure modes — without containing procedural logic that determines how those goals are achieved. An agent definition unit must contain only role contracts; procedural execution logic belongs in a separate execution layer.

When an agent definition unit contains source code or conditional execution logic alongside its role specification, the judgment layer (what to do, given what inputs) and the execution layer (how to do it) become structurally coupled. This coupling produces three concrete failures: (1) context windows accumulate both policy state and execution state, creating interference between the two; (2) neither layer can be tested in isolation; (3) the same execution logic cannot be reused across agents with different roles, and the same role cannot be enforced across different execution implementations.

Define an agent as: a named specification of inputs it accepts, outputs it produces, skills it may invoke, and failure modes it must handle. Keep all procedural logic — tool invocation sequences, retry loops, state transitions — in execution units that the agent specification controls by reference, not by inclusion.

## Evidence

### Academic Papers

- **Agent Behavioral Contracts: Formal Specification and Runtime Enforcement for Reliable Autonomous AI Agents** (arXiv:2602.22302, 2026) — Defines every agent as a formal contract C=(P, I, G, R): preconditions, invariants, goals, and recovery behaviors. The paper demonstrates that separating this behavioral contract from execution code enables runtime enforcement and independent validation; agents defined only in code cannot be checked at the specification level without running the implementation.

- **CodeDelegator: Mitigating Context Pollution via Role Separation in Code-as-Action Agents** (arXiv:2601.14914, 2026) — Empirically demonstrates that mixing a judgment layer (Delegator) with an execution layer (Coder) in a single agent unit causes context pollution: execution history contaminates policy decisions, degrading task completion rates. Separating the two layers into distinct agents eliminates this interference and is the central empirical result of the paper.

- **Contract-based Design and Verification of Multi-Agent Systems with Quantitative Temporal Requirements** (arXiv:2412.13114, 2024) — Applies assume-guarantee contract theory to multi-agent systems, proving that an individual agent's role interface can be fully specified and compositionally verified without access to its implementation. This establishes that role-level specifications are sufficient for system-wide reasoning — implementation details inside the agent definition add no verification value and reduce composability.

### Official Best Practices

- **OpenAI Agents SDK — Agent Definition Reference** (https://openai.github.io/openai-agents-python/agents/) — The SDK's canonical agent representation is a declarative specification: `name`, `instructions` (role contract), `output_type` (output boundary), and `tools` (allowed skill set). Procedural execution logic resides outside the agent object in the runner/orchestration layer. The documentation states that a `Prompt` object allows configuration of "instructions, tools and other config for an agent outside of your code," making the separation of specification from execution a first-class design decision.

- **OpenAI — Orchestration and Handoffs** (https://developers.openai.com/api/docs/guides/agents/orchestration) — Prescribes that agent handoff descriptions ("what this agent does and when to invoke it") are the primary mechanism for inter-agent routing, and that each agent's scope is defined by its instructions, tools, and output type — not by shared code. Routing logic resides in the orchestrator; each specialist agent exposes only a declared interface.

### Named in Literature?

No unified name. Closest named concepts: "behavioral contract" (arXiv:2602.22302), "role separation" (arXiv:2601.14914), "assume-guarantee interface" (arXiv:2412.13114), and "agent definition" (OpenAI Agents SDK). The specific formulation — that an agent's definition unit must contain only declarative role contracts and must exclude procedural execution logic — is implied consistently across these sources but is not named as a single pattern.

## When to Apply

| Condition | Apply contract-only agent definition? |
|---|---|
| Building a multi-agent system where ≥2 agents share a domain | Yes — shared execution logic must live in shared execution units, not duplicated inside each agent's definition |
| Designing an agent that will be reused across different orchestrators or runtimes | Yes — a role contract is portable; embedded procedural code is not |
| Debugging a failure where an agent's decision changed without its role specification changing | Yes — the cause is execution state leaking into the specification layer; separation eliminates this |
| Writing unit tests for agent behavior without running the full execution pipeline | Yes — a declarative contract is testable in isolation; embedded code requires the full runtime |
| Building a single-step, single-use script that invokes one model call | No — the overhead of separation is not justified for throwaway automation |
| Implementing internal retry or error-handling logic inside an execution tool | No — execution internals belong inside the tool, not in the agent's role contract |

Apply when:
1. An agent definition contains `if` branches that select between execution strategies — extract the strategies to execution units; the agent contract specifies only which strategies are allowed.
2. An agent definition is duplicated to serve two roles with different input/output types — the role contracts must differ; shared execution logic must be extracted to a shared tool or subagent.
3. An agent's context window grows across turns due to accumulating execution history — execution state is leaking into the specification layer; separate them.

Do not apply when: the system is a single agent with a single tool and a fixed input schema, and the system will not be extended. In this case, a formal contract layer adds indirection without benefit.

## Known Limits

- **Contract completeness requires upfront specification effort**: Defining preconditions, invariants, goals, and failure modes explicitly before implementation requires discipline. Teams under delivery pressure tend to embed logic directly into the agent definition and defer formalization. The pattern is most stable when contracts are written before execution code, not extracted afterward.

- **Dynamic role adjustment breaks static contracts**: When an agent's role must change at runtime (e.g., an orchestrator that also executes based on runtime conditions), a static declarative contract cannot fully capture the behavior. Hybrid designs that mix static contracts with dynamic role injection require explicit versioning of the contract.

- **Tooling ecosystem maturity**: Formal contract enforcement at runtime (as described in arXiv:2602.22302) requires tooling not yet available in most production agent frameworks as of May 2026. In the absence of enforcement tooling, the pattern operates as an architectural discipline rather than a mechanically verified constraint.

- **Not applicable to single-layer execution agents**: This pattern does not apply to agents that are purely execution units with no judgment responsibility — for example, a tool-wrapper agent that receives a fully specified action and executes it. In this domain, a role contract adds no separation benefit because judgment is entirely upstream.

## Promotion History

Candidate — created 2026-05-19 from user insight.
Confirmed — promoted 2026-05-19, score 21/25 (코어성 5, 리스크감소 4, 확장성 5, 제어성 4, 기록성 3).

## Properties to Restore

위반이 감지됐을 때, 수정된 결과가 만족해야 할 속성들이다.
구체적인 구현 방법은 컨텍스트에 따라 달라질 수 있다.

| Violation detected | Properties the fix must satisfy |
|--------------------|--------------------------------|
| Procedural logic (conditional branches, retry sequences, tool invocation order) is embedded inside the agent's role definition | The role definition contains only declarative elements: accepted inputs, produced outputs, allowed tools, and failure modes. No step-by-step execution logic appears in the role specification. |
| The same execution code is duplicated across agent definitions that serve different roles | Execution logic exists in a single shared execution unit referenced by multiple role contracts. Each role contract references the shared unit; it does not contain a copy. |
| An agent's context window grows across turns because execution history accumulates alongside policy state | Execution state and policy state reside in structurally distinct scopes. Clearing or resetting one does not affect the other. |
| A role contract cannot be verified without running the full execution pipeline | The role contract is independently checkable — its preconditions, invariants, and output types can be evaluated against a specification without invoking the execution layer. |
