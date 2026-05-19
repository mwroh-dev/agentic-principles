# Artifact vs Knowledge — Separation Principle

Status: confirmed
Last reviewed: 2026-05-18

## Principle

In AI agent systems, two distinct categories of persistent output must be kept structurally separate:

**Artifacts** — immutable episodic records of what happened during a run.
- Tense: past ("this happened")
- Mutability: append-only or write-once per run
- Primary consumer: human auditors, compliance, approval gates
- Examples: audit logs (JSONL), dry-run reports, run summaries, action execution records
- Storage policy: gitignored — ephemeral, regenerated each run

**Knowledge** — mutable semantic patterns accumulated across runs.
- Tense: present ("this is known")
- Mutability: agents update over runs as patterns emerge
- Primary consumer: next-phase agents, future runs
- Examples: service capability maps, risk calibration patterns, routing failure patterns, ADR summaries
- Storage policy: committed to git — must survive deployments and machine changes

The distinction is not just about content — it is about **who reads it** and **whether it should change**. Conflating the two into a single store means neither works optimally: audit records lose tamper-evidence, and knowledge stores become append-only when they need to be updated.

## Evidence

- **CoALA: Cognitive Architectures for Language Agents** (Sumers et al., arXiv:2309.02427, TMLR 2023) — canonical four-way memory taxonomy: episodic (what happened) vs semantic (what is known) are structurally distinct, with different mutability and retrieval semantics.
- **MemGPT: Towards LLMs as Operating Systems** (Packer et al., arXiv:2310.08560, Oct 2023) — persistent episodic storage outside the context window achieves 93.4% accuracy vs 35.3% baseline (2.6× improvement) on Deep Memory Retrieval tasks; performance is invariant to context length where baseline LLMs degrade significantly. Demonstrates the concrete cost of conflating in-context working memory with persistent episodic storage.
- **CORAL** (Qu et al., arXiv:2604.01658, Apr 2026) — operationalizes the split as a filesystem: `attempts/` (pipeline-written, immutable) vs `notes/skills/` (agent-curated, deliberately editable).
- **Brown "Audit Trails"** (Ojewale et al., 2025) — audit records are governance infrastructure for compliance teams, not for agents. Must be tamper-evident and append-only.
- **SSGM** (Lam et al., 2026) — "SSGM pairs a rapidly updatable Mutable Active Graph with an append-only Immutable Episodic Log." Drift in the mutable graph can be corrected by replaying the immutable log. Only possible if the two are kept separate.
- **MIRIX** (Wang & Chen, 2025) — timestamped immutable episodic entries vs evolving semantic memory as separate subsystems.

## When to Apply

| Signal | Decision | What breaks if ignored |
|--------|----------|----------------------|
| An agent reads output from a previous run to inform its current run | Separate stores are required — create `knowledge/` (committed) for cross-run reading | Without separation, agents either re-derive everything (slow) or read stale episodic records as if they were live knowledge |
| A compliance audit, approval gate, or human review process needs to verify what happened | Audit records must be append-only and gitignored as `artifacts/` | Mutable stores lose tamper-evidence; auditors cannot distinguish original records from post-hoc edits |
| An agent is updating patterns over time but its store is accumulating like a log | The store is mistyped as episodic — restructure as semantic `knowledge/` with in-place updates | Append-only semantic stores grow without bound and become stale (old entries never expire) |
| A "cleanup" removes files that seem redundant but agents depend on them across runs | The removed files were knowledge, not artifacts — restore and commit | Deleting committed knowledge breaks future runs that expected accumulated patterns |

## Known Limits / Failure Modes

- **Migration timing ambiguity:** The split becomes necessary when agents need to read knowledge across runs. For single-run pipelines, a unified store is an acceptable approximation. Migrating too early adds overhead; migrating too late requires backfilling.
- **Transition policy unsolved:** There is no established rule for when an episodic record "graduates" to semantic status. As of 2026, this remains an open problem in the literature.
- **Overwrite vs append confusion:** If the storage layer truncates and rewrites files per run, it is producing per-run episodic snapshots (acceptable) not a growing audit trail (not acceptable for compliance). The distinction must be explicit in the storage implementation.
