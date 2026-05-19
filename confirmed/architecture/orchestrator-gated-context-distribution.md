# Orchestrator-Gated Context Distribution

Status: confirmed
Last reviewed: 2026-05-18
Score: 8.8/11

## Principle

In a multi-agent system, the orchestrator is the sole holder of the full artifact produced by any discovery or data-mining phase. It distributes only role-relevant projected views to each subagent. Subagents return decision artifacts, not internal reasoning.

**Pull, not push.** The orchestrator's pipeline definition specifies what fields each consumer receives. The orchestrator extracts and delivers only those fields; consumers do not receive the full artifact and filter it themselves.

**Taxonomy stays local.** A classifier or routing agent that determines intent, persona, or scenario owns its classification taxonomy privately. (This is pull-not-push applied to classification agents: downstream agents do not receive the taxonomy, only the result.) Downstream agents receive only the decision result — a `classification_id`, `risk_level`, or `intent_category` — not the taxonomy or the classification reasoning that produced it.

**Outputs are decision artifacts, not reasoning logs.** Subagent outputs contain decision values and their immediate rationale summary only. The orchestrator synthesizes decision artifacts, not transcripts.

The orchestrator's role is threefold: hold the full artifact, project role-scoped views to each subagent, collect and synthesize subagent decision artifacts. Full isolation and full sharing are both failure modes. The orchestrator is the only viable gateway.

## Artifact Flow

Artifact flow follows four invariant steps:

**Phase output → Orchestrator holds full artifact.** The data-mining or discovery phase produces one full artifact. The orchestrator is the only entity that reads this artifact in full. No subagent receives it directly.

**Orchestrator → Consumer-specific projected views.** For each downstream consumer, the orchestrator creates a projected view containing only the fields that consumer's role requires. The required field list is defined at design time in the pipeline definition as a declarative specification. A projection specification takes the form: consumer role, required field names from the source artifact, and result artifact schema. Example:
```
consumer: security_assessment
required_fields: [auth_type, required_scopes, endpoints]
result_schema: { risk_level: enum, constraints: list, rationale: string }
```
Two consumers can share a field — for example, `throttling_policy` — if both roles need it; projection is not exclusive. The orchestrator applies the projection; subagents do not select from a larger set.

**Consumers → Result artifacts.** Each consumer produces a result artifact: the conclusion, recommendation, or constraint determined by that consumer. Internal reasoning does not appear in the result artifact. The result artifact schema is defined at design time, versioned, and fixed for a given pipeline version.

**Orchestrator → Synthesized output.** The orchestrator collects all result artifacts, detects conflicts between consumer outputs, and produces a synthesized artifact — for example, a `merged_decision_record` — containing merged decisions and flagged conflicts. Conflict detection is a structural operation on the result artifact schemas, not a prose interpretation task.

## Evidence

### Performance Evidence

- **SWE Context Bench** (arXiv:2602.08316) — On real GitHub issue resolution, a filtered 217-token summary achieved 34.34% task resolution versus 27.27% for unfiltered 25,600-token full context. Autonomous full-context retrieval dropped to 22.22%, below the no-context baseline of 21.21%. Minimal, filtered context outperforms full context by 30% relatively. Full context does not improve agent performance; it degrades it. This single-agent result establishes the mechanism: if a single agent degrades with irrelevant context, multiple agents receiving the same undifferentiated source artifact compound the degradation independently.

- **Scaling Agent Systems** (arXiv:2512.08296) — Error amplification by architecture type — using the paper's terminology: Independent MAS (zero sharing): 17.2×; Decentralized MAS (peer-to-peer full sharing): 7.8×; Centralized MAS (single coordinator mediating all inter-agent communication): 4.4×. The orchestrator-mediated architecture in this pattern maps to the Centralized MAS category. Centralized architecture dominates both extremes by a wide margin. The measurement directly quantifies the cost of the two failure modes — full isolation and full sharing — that this pattern avoids.

- **"Lost in the Middle"** (arXiv:2307.03172, Liu et al.) — LLM accuracy follows a U-curve over context position: models perform well on information at the start or end of context and poorly on information in the middle. When relevant information is buried in a long context, performance drops below the closed-book baseline. Irrelevant context actively degrades reasoning. This provides the mechanistic explanation for why providing full shared context to all agents produces worse results than providing minimal role-scoped context.

- **Agyn multi-agent system** (arXiv:2602.01465) — Achieves 72.2% on SWE-bench Verified using isolated agent sandboxes with structured artifact communication, not a shared message pool. Large artifacts are persisted to the filesystem rather than inserted into model context. Agents communicate via structured result artifacts, not conversational context dumps. This is the highest published score on the benchmark and was achieved with strict context isolation, not liberal sharing.

### Architectural Precedents

- **Parnas Information Hiding** (CACM 1972, Vol. 15 No. 12) — "Every module is characterized by its knowledge of a design decision which it hides from all others. Its interface reveals as little as possible about its inner workings." Applied here: a classifier module's design secret is its taxonomy; its interface is the decision result. Exposing the taxonomy through the interface violates information hiding and creates coupling between the classifier's internal design and every downstream consumer.

- **HEARSAY-II Blackboard Architecture** (Erman et al., ACM Computing Surveys 1980, Vol. 12 No. 2) — Knowledge Sources are self-contained expert modules that each privately own their domain knowledge and post only hypotheses — partial solutions — to the shared blackboard. Knowledge Sources do not expose their internal knowledge bases to other Knowledge Sources. The blackboard (orchestrator role) mediates all inter-module communication. The 1980 architecture instantiates taxonomy-local ownership and decision-artifact-only outputs.

- **MetaGPT role-filtered subscriptions** (arXiv:2308.00352) — "Sharing all information with every agent can lead to information overload." MetaGPT implements role-specific subscriptions over a shared pool; agents receive only messages matching their role profile. The subscription filter is the declarative field list in this pattern's vocabulary. The paper reports that role-filtered subscriptions reduce hallucination and improve task coherence compared to full broadcast.

### Industry Documentation

- **Anthropic Context Engineering** (anthropic.com/engineering/effective-context-engineering-for-ai-agents) — "Each subagent gets exactly the context it needs for its specific task and nothing else." "Good context engineering means finding the smallest possible set of high-signal tokens that maximize the likelihood of desired outcomes." Subagents return "condensed, distilled summaries" rather than full context. This directly prescribes the projected view delivery and decision-artifact return pattern.

## When to Apply

| Signal | Decision | What breaks if ignored |
|--------|----------|----------------------|
| A new consumer is being added to a pipeline and someone proposes giving it the full source artifact. | Define the consumer's field requirements declaratively and create a projected view; do not pass the full artifact. | The consumer receives irrelevant fields that degrade its reasoning quality. As the full artifact grows over time, consumer performance degrades silently without any code changes — there is no signal that the artifact has become harmful to the consumer. |
| A classifier or routing agent's taxonomy or heuristics need to be updated. | Update only the classifier agent's knowledge; downstream agents do not need to change. | If the taxonomy is shared, all downstream agents must be re-evaluated for consistency on every taxonomy update. Update scope becomes the entire pipeline instead of one agent, and every taxonomy iteration is a regression risk for every consumer. |
| A subagent is returning its full reasoning chain or intermediate scratchpad to the orchestrator. | Define a result artifact schema for that agent; require it to return only the decision values and a one-line rationale summary. | Orchestrators that receive reasoning transcripts instead of decision artifacts cannot synthesize across lanes without reading and re-interpreting prose. Synthesis becomes an LLM reasoning task instead of a merge operation, introducing an additional error-amplification step. |
| Two parallel lanes produce conflicting outputs — for example, Lane A recommends approach X and Lane B requires an incompatible approach Y for the same decision dimension. | Conflict detection and resolution are the orchestrator's responsibility; lanes do not communicate with each other. | If lanes are allowed to communicate and negotiate directly, they enter coordination loops, accumulate shared context, and lose the performance advantage of minimal context per agent. The coordination overhead compounds as the number of lanes grows. |
| The full artifact is being shared directly with all agents for completeness or because it is simpler to implement. | Implement orchestrator-level projection even if it requires additional design work. | SWE Context Bench shows that full unfiltered context performs no better than no context at all — 27.27% versus 21.21% baseline, a gap of six percentage points that is smaller than the eight-point advantage of filtered minimal context at 34.34%. The 30% relative gap between filtered and unfiltered context compounds as artifact size grows. |

## Known Limits

**Projection definition cost.** Declaring field requirements at design time requires knowing in advance what each consumer needs. For exploratory or open-ended agents whose information needs are not predictable, pre-declared projection is not possible. In these cases, use a semantic retrieval layer — the agent queries for what it needs at runtime — rather than pre-declared field projection. Semantic retrieval is slower and introduces retrieval error; it is the correct tradeoff only when the field requirement cannot be specified in advance.

**Conflict resolution scope.** When two consumers produce genuinely irreconcilable outputs — for example, security constraints that make the proposed technical architecture infeasible — the orchestrator cannot resolve this through merging. The resolution requires a Phase (serial) follow-up, not a Lane (parallel) retry. Design pipelines to detect this class of conflict at the synthesis stage and route to a resolution phase. Attempting to resolve structural conflicts through orchestrator merging produces an output that satisfies neither consumer's constraint, which is worse than surfacing the conflict explicitly.

**Result artifact schema versioning.** Result artifact schemas must be versioned. If a consumer's decision vocabulary changes — new risk levels, new authentication types, new classification categories — the orchestrator's synthesis logic may misinterpret the output without an explicit schema version check. Define schema versions for all result artifacts and enforce compatibility checks at the synthesis stage. Unversioned schemas make silent misinterpretation indistinguishable from correct synthesis.

**Full isolation failure mode.** Removing orchestrator mediation entirely — fully isolated agents with no shared state — does not solve the context problem; it introduces error amplification at 17.2× in empirical measurement (arXiv:2512.08296). The orchestrator's synthesis step is load-bearing: without it, errors in one consumer do not get corrected by information from other consumers, and no conflict detection occurs. Full isolation is not a valid simplification of this pattern. The performance gain from minimal per-agent context requires the orchestrator synthesis layer to remain intact.

**Projection maintenance under schema evolution.** When the source artifact's schema changes — new fields added by an upstream discovery phase — every projection specification must be audited. Fields that were correctly excluded may become relevant; existing projected fields may become irrelevant or renamed. The maintenance cost of projection specifications scales with the number of downstream consumers. Treat projection definitions as first-class schema artifacts, subject to the same review process as the source artifact schema itself.

## Relationship to Related Patterns

**Subagent context isolation** (`subagent-per-task-isolation`) governs whether context accumulates between dispatches — apply it first, then apply this pattern to specify what the isolated context contains. This pattern prescribes what that context contains: field-level projection from a source artifact, taxonomy locality, and structured result artifact return contracts. The two patterns are complementary: isolation is the prerequisite; this pattern specifies the content of the isolated context.

**Phase vs. Lane execution** (`phase-vs-lane-execution`) determines dispatch topology. This pattern is topology-agnostic — apply artifact projection regardless of whether consumers run serially or in parallel. The artifact flow (source artifact → projected view → result artifact → synthesis) applies equally to serial phase specialists and parallel lanes.
