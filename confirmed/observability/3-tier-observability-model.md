# 3-Tier Observability Model

Status: confirmed
Last reviewed: 2026-05-19

## Principle

Observability data for AI agent systems must be separated into three distinct, non-overlapping tiers:

1. **Raw telemetry** — LLM input/output, tool calls, and event streams, collected at execution-unit granularity (per agent invocation, per tool call). This tier is immutable and must be preserved in original form.
2. **Evidence artifact** — Structured bundles of telemetry, assembled per scenario or evaluation run, that serve as the input to evaluation pipelines. These bundles contain references to raw telemetry, not copies or summaries.
3. **Report** — Failure attribution, root-cause analysis, and performance aggregations derived from evidence artifacts. Reports never substitute for the underlying evidence they reference.

Merging tiers produces two failure modes: (a) semantic summaries that cannot be traced back to source events, making failures non-reproducible; (b) evaluation artifacts that cannot be isolated from operational noise, invalidating benchmark comparisons. Each tier must be independently queryable, versioned, and retained on its own schedule.

Do not conflate evidence artifact construction with report generation. An artifact is a stable, referenceable input; a report is a derived output that may expire as models or criteria change.

## Evidence

### Academic Papers

- **A Taxonomy of AgentOps for Enabling Observability of Foundation Model based Agents** (arXiv:2411.05285, Nov 2024, anonymous) — Directly argues for separating raw trace data, interpretability artifacts, and debuggability insights into distinct layers. Demonstrates that conflating these layers prevents reproducible debugging of agent failures.

- **Beyond Black-Box Benchmarking: Observability, Analytics, and Optimization of Agentic Systems** (arXiv:2503.06745, Mar 2025, anonymous) — Proposes a pipeline that generates raw telemetry and evaluation analysis as separate outputs. Empirically shows that mixing telemetry with evaluation results produces benchmarks that cannot be reproduced across runs.

- **AgentTrace: A Structured Logging Framework for Agent System Observability** (arXiv:2602.10133, Feb 2026, anonymous) — Introduces a three-surface taxonomy (operational, cognitive, contextual) for structured log separation. Formalizes the principle that each surface must be independently queryable without requiring access to the others.

- **MAESTRO: Multi-Agent Evaluation Suite for Testing, Reliability, and Observability** (arXiv:2601.00481, Jan 2026, anonymous) — Implements separate export pipelines for raw telemetry and system-level evaluation signals. Validates that downstream evaluation quality degrades when these pipelines share a data sink.

- **AI Observability for Large Language Model Systems: Multi-Layer Analysis** (arXiv:2604.26152, Apr 2026, anonymous) — Identifies absence of inter-layer signal separation as the primary cause of single-layer monitoring failures in LLM systems. Provides empirical failure taxonomy for systems that colocate raw and derived observability data.

### Official Best Practices

- **AI Agent Observability — Evolving Standards and Best Practices** (https://opentelemetry.io/blog/2025/ai-agent-observability/) — OpenTelemetry, 2025. States that in AI agent systems, telemetry serves as both a monitoring signal and an evaluation feedback loop, and that standardizing the shape of telemetry is necessary to avoid vendor lock-in from framework-specific formats. Explicitly recommends storing prompt and completion content in span events (not span attributes) so that content can be filtered or dropped at the collector level independently of the trace skeleton — a structural separation between raw content and indexed operational signal.

- **Inside the LLM Call: GenAI Observability with OpenTelemetry** (https://opentelemetry.io/blog/2026/genai-observability/) — OpenTelemetry, 2026. Specifies that LLM content (raw I/O) must be stored in span events rather than span attributes because attributes are always indexed and size-limited; this separation makes raw content independently filterable. Defines four primary observability areas — LLM client spans, agent spans, events, and metrics — as distinct collection targets.

- **Evaluate Agent Workflows** (https://developers.openai.com/api/docs/guides/agent-evals) — OpenAI, 2025. Prescribes a three-stage pipeline: (1) collect high-signal traces (raw telemetry), (2) build repeatable datasets from those traces (evidence artifacts), (3) run graded eval runs (reports). Explicitly states that teams should not move to datasets and eval runs until the raw trace layer is established, formalizing the dependency order between tiers.

- **Claude Code Monitoring — Observability Configuration** (https://docs.anthropic.com/en/docs/claude-code/monitoring-usage) — Anthropic, 2025. Separates raw API body logging (`OTEL_LOG_RAW_API_BODIES`) from tool content logging (`OTEL_LOG_TOOL_CONTENT`) as independent opt-in flags, requiring explicit configuration per tier. Tool input/output and raw API bodies are excluded from trace spans by default, enforcing tier separation at the instrumentation layer.

### Named in Literature?

No unified name. Closest named concepts: "AgentOps observability layers" (arXiv:2411.05285), "operational/cognitive/contextual surfaces" (arXiv:2602.10133), and OpenTelemetry's "LLM client spans / agent spans / events / metrics" four-area taxonomy — but the specific formulation of three tiers named raw telemetry, evidence artifact, and report is implied across sources and not named as a single pattern.

## When to Apply

Apply when:

| Condition | Why this tier model is required |
|-----------|--------------------------------|
| Agent failures must be reproducible from logs | Without immutable raw telemetry, the original execution context cannot be recovered |
| Evaluation criteria change independently of data collection | Evidence artifacts decouple what was collected from how it is scored |
| Multiple evaluators or graders run over the same run data | A single evidence artifact can be scored by multiple report-generation passes without re-collecting telemetry |
| Raw telemetry volume is high and must be selectively retained | Tier separation allows raw data retention schedules independent of report retention |
| Benchmark results must be compared across model versions | Evidence artifacts must remain stable; reports derived from them can be regenerated with new graders |
| Compliance or audit requires provenance from conclusion back to source event | The report → artifact → raw telemetry chain provides traceable provenance |

Do not apply when: the system executes a single deterministic function with no LLM calls and no branching tool use — flat logging suffices and tier separation adds overhead without benefit.

## Known Limits

- **Artifact assembly cost**: Constructing evidence artifacts from raw telemetry requires a secondary pipeline. In low-resource deployments, this pipeline may be omitted, collapsing tiers 1 and 2. This is an operational tradeoff, not a design recommendation; document the collapse explicitly if it occurs.

- **Reference integrity burden**: Evidence artifacts that reference raw telemetry by pointer (not copy) require raw telemetry to remain queryable for the artifact's lifetime. Mismatched retention policies between tier 1 and tier 2 will silently break artifact traceability.

- **Report invalidation on criteria change**: Reports are derived artifacts. When grading criteria, model versions, or scenario definitions change, existing reports are invalidated but evidence artifacts remain valid. Systems must track which grader version produced which report; failure to version reports causes silent staleness.

- **Does not apply to single-shot inference pipelines**: This pattern targets multi-step agent systems with tool use, branching, and evaluation feedback loops. Applying it to a request-response LLM endpoint with no tool calls introduces unnecessary architectural complexity.

## Promotion History

Candidate — created 2026-05-19 from user insight.
Confirmed — promoted 2026-05-19, score 22/25 (코어성 4, 리스크감소 5, 확장성 4, 제어성 4, 기록성 5).
