# Best Practice Knowledge Base — Index

Last updated: 2026-06-08

Two tiers: **confirmed** (paper-backed, quality-gated, production-ready for agent injection) and **candidates** (observed patterns, pending independent confirmation).

---

## confirmed/

### architecture/

- [`agent-directory-structure.md`](confirmed/architecture/agent-directory-structure.md) — Runtime code, skills, episodic artifacts, semantic knowledge, and docs: what goes where and why
- [`artifact-placement-decision-framework.md`](confirmed/architecture/artifact-placement-decision-framework.md) — 5-question placement test: route any new artifact to runtime code, procedure, episodic record, semantic knowledge, or documentation layer by stakeholder concern
- [`orchestrator-gated-context-distribution.md`](confirmed/architecture/orchestrator-gated-context-distribution.md) — Orchestrator holds full artifact; consumers receive only role-scoped projected views; subagents return decision artifacts, not reasoning transcripts
- [`constraint-hierarchy-over-accumulation.md`](confirmed/architecture/constraint-hierarchy-over-accumulation.md) — Few constitutional hard invariants at top level; push everything else to judgment, code, or knowledge. Raise the bar for bans, not the count.
- [`phase-vs-lane-execution.md`](confirmed/architecture/phase-vs-lane-execution.md) — Serial vs parallel agent dispatch determined by explicit dependency graph analysis, not defaults
- [`purpose-scoped-authority.md`](confirmed/architecture/purpose-scoped-authority.md) — Federated sources with scoped authority over managed correspondences, not a single source of truth
- [`skill-surface-types.md`](confirmed/architecture/skill-surface-types.md) — local_module / repo_skill / codex_subagent / model_replay: surface types and their required file structures
- [`subagent-per-task-isolation.md`](confirmed/architecture/subagent-per-task-isolation.md) — Pattern A [confirmed]: fresh isolated context per subagent; Pattern B [candidate]: two-stage review ordering
- [`schema-valid-not-runtime-executable.md`](confirmed/architecture/schema-valid-not-runtime-executable.md) — Four ordered gates (shape → semantic → fact → supportability) at the orchestrator dispatch boundary; gate failure → blocked, not auto-corrected
- [`skill-discovery-is-not-authorization.md`](confirmed/architecture/skill-discovery-is-not-authorization.md) — Discoverability is a capability grant; `disallowedTools`/`Skill(name)` rules enforce access; folder naming, `skills:` preload, and implicit-invocation flags do not
- [`policy-single-owner-not-repeated.md`](confirmed/architecture/policy-single-owner-not-repeated.md) — Each policy has exactly one canonical owner file; copies must be replaced with loading declarations. Extends `purpose-scoped-authority.md`. (22/25)
- [`behavioral-contract-integrity-via-typed-source.md`](confirmed/architecture/behavioral-contract-integrity-via-typed-source.md) — Role contract families follow typed-source → generator → artifact pipeline; hand-maintained free text causes silent authority boundary drift across role family members. (20/25)

### instructions/

- [`entry-surface-as-context-loader.md`](confirmed/instructions/entry-surface-as-context-loader.md) — Entry surface (prompt.md, CLAUDE.md) contains only loading declarations; policy accumulation causes position-bias authority inversion. (20/25)
- [`state-contract-over-prose-runbook.md`](confirmed/instructions/state-contract-over-prose-runbook.md) — Condition→action→trace is the established pattern (ReAct, MASMP) for agent phase instructions; prose runbooks cause silent misexecution via connective ambiguity. (21/25)
- [`validator-structural-contract-not-content-police.md`](confirmed/instructions/validator-structural-contract-not-content-police.md) — Agent instruction verification requires two tiers: structural facts (automated) and quality judgment (model/human review); applying Tier 1 tools to Tier 2 questions locks monolithic architecture. (21/25)

### agents/

- [`evaluation-as-behavioral-specification.md`](confirmed/agents/evaluation-as-behavioral-specification.md) — Evals written before implementation; the eval IS the behavioral specification
- [`harness-hill-climbing.md`](confirmed/agents/harness-hill-climbing.md) — Atomic one-element-at-a-time harness iteration loop guided by eval failures
- [`per-agent-knowledge-patterns.md`](confirmed/agents/per-agent-knowledge-patterns.md) — Agent-type to knowledge update strategy mapping (A / B / B+prediction-error)

### memory/

- [`artifact-vs-knowledge.md`](confirmed/memory/artifact-vs-knowledge.md) — Episodic artifacts (immutable, gitignored) vs semantic knowledge (mutable, committed)
- [`knowledge-update-strategies.md`](confirmed/memory/knowledge-update-strategies.md) — Strategy A (immediate post-run synthesis) vs B (batch) vs B+prediction-error, by agent type

### safety/

- [`multi-layered-safety-via-code.md`](confirmed/safety/multi-layered-safety-via-code.md) — Role → Gate → Rule → Hook: four-layer code-level safety enforcement; prompt instructions cannot enforce policy

---

## candidates/

Patterns observed across sessions, partially supported by evidence, but not yet independently named or empirically validated as a unit.

- [`action-decomposition-first.md`](candidates/action-decomposition-first.md) — Define the action taxonomy (what agents do, what signals they produce) before building any agent. Promotion blocker: not yet named as a single pattern in literature.
- [`observable-signal-to-evaluator-mapping.md`](candidates/observable-signal-to-evaluator-mapping.md) — Map observable signal type to evaluator class (deterministic / semantic / judgmental). Promotion blocker: full signal→evaluator→improvement pipeline not independently named.
- [`subagent-per-task-isolation.md`](candidates/subagent-per-task-isolation.md) — Pattern B only: spec-compliance-first → code-quality-second two-stage review ordering. Promotion blocker: not independently named or empirically validated.

---

## papers/

- [`agent-memory-references.md`](papers/agent-memory-references.md) — Annotated bibliography: CoALA, CORAL, SSGM, MIRIX, Reflexion, A-Mem, ExpeL, SYNAPSE, Nemori, Brown Audit Trails
