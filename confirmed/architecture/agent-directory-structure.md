# Agent Repository Directory Structure

Status: confirmed
Last reviewed: 2026-05-18

## Principle

A repository hosting a multi-agent orchestration system should separate directories by **ownership and lifecycle**, not by file type. Five concerns must be kept structurally distinct:

```
runtime-code/        ← agents execute from here
reusable-skills/     ← shared playbooks, not private to one agent
episodic-evidence/   ← immutable audit records (gitignored)
semantic-knowledge/  ← accumulated learning (committed)
documentation/       ← source-of-truth docs only
```

### Runtime Code (`agents/`)
All orchestration logic, enforcement, contracts, connectors, and execution surfaces. Nothing here should be an example or placeholder — this is the live runtime.

### Reusable Skills (`skills/`)
Shared operating procedures usable across multiple workflows. Skills are not private implementation details of a single caller. If a procedure is only ever called by one agent, it belongs inside `agents/`, not `skills/`.

### Skill Discovery Boundary

Files under `skills/` are discoverable surfaces, not agent-private implementation details. Placement in `skills/` does not restrict which agents may invoke a skill — it only declares the skill is shared. If a procedure is private to one agent, it belongs in that agent's implementation layer or behind a runtime gateway.

Public skills must be reusable across multiple workflows or safe to expose as a shared playbook. Exclusivity must be enforced through tool permissions (`disallowedTools: Skill`, `Skill(name)` deny rules), scoped subagent configuration, or discovery control — not by folder naming or prompt descriptions. See [`skill-discovery-is-not-authorization.md`](skill-discovery-is-not-authorization.md).

### Episodic Evidence (`artifacts/`)
Immutable runtime evidence: audit logs, dry-run reports, approval records, run summaries. Regenerated each run. **Gitignored** — these are ephemeral outputs, not accumulated learning.

### Semantic Knowledge (`knowledge/`)
Agent-owned evolving patterns accumulated across runs. **Committed to git** — this learning must survive deployments and machine changes. Each agent has its own subdirectory (`knowledge/{agent-type}/episodic/`, `knowledge/{agent-type}/semantic/`).

The `knowledge/` directory may begin as a structural placeholder (directories only, no content) before the knowledge update loops are implemented. The placeholder commits the design intent and reserves the namespace.

### Documentation (`docs/`)
Source-of-truth product and architecture documentation only. Temporary plans, scratch notes, and migration docs must not live here long-term — they create false signal for agents reading the docs directory.

## Gitignore Policy

| Directory | Git policy | Reason |
|---|---|---|
| `artifacts/` | Gitignored | Ephemeral — regenerated each run |
| `knowledge/` | Committed | Accumulated — must survive deployments |
| `docs/` | Committed | Source of truth |
| `agents/`, `skills/` | Committed | Runtime source |

**Decision rule:** Before adding a directory to gitignore, ask: "Is this file regenerable, or does it accumulate value over time?" Regenerable → gitignore. Accumulating → commit.

## Evidence

- **CORAL** (Qu et al., arXiv:2604.01658, Apr 2026) — Three-way filesystem split: `attempts/` (pipeline-written, gitignored episodic records), `notes/` (agent-curated semantic understanding), `skills/<name>/` (reusable procedural knowledge). Explicit design rationale: `attempts/` is immutable per run; `notes/` is mutable across runs; `skills/` is reusable across agents. Direct structural precedent for the episodic-evidence/semantic-knowledge/reusable-skills separation.
- **SSGM** (Lam et al., 2026) — Immutable Episodic Log and Mutable Active Graph as structurally separate subsystems. Drift in the mutable graph can be corrected by replaying the immutable log — only possible if the two are physically separate.
- **Brown "Audit Trails"** (Ojewale et al., 2025) — Audit records are governance infrastructure, not agent-facing resources. They require tamper-evidence properties that are incompatible with mutable knowledge stores.
- **Anthropic "Equipping Agents for the Real World with Agent Skills"** (anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills) — Anthropic's own implementation uses a three-tier separation: `scripts/` (runtime executable code), `skills/<name>/SKILL.md` (reusable procedural knowledge packaged for reuse), `reference.md` (documentation loaded contextually). Progressive disclosure design: metadata loads at startup, skill content loads on relevance, references load on demand.
- **"Episodic Memory is the Missing Piece for Long-Term LLM Agents"** (Pink et al., arXiv:2502.06975, Feb 2025) — Provides theoretical grounding for why episodic-evidence and semantic-knowledge must be separate subsystems: they have different write policies, retention horizons, and consolidation triggers. Identifies the consolidation policy (when episodic records graduate to semantic status) as an open problem requiring explicit architectural separation.

## When to Apply

| Signal | Decision | What breaks if ignored |
|--------|----------|----------------------|
| Runtime-generated files (logs, reports, run summaries) are committed to git | Move to `artifacts/` and gitignore — these are ephemeral, not accumulated | Git history bloats; merge conflicts on every run; no meaningful diff signal in version control |
| Agent-accumulated knowledge (learned patterns, calibration data) is gitignored | Move to `knowledge/` and commit — this learning must survive deployments | Knowledge resets on every deployment; agents start cold on each run regardless of prior learning |
| A `skills/` entry is only ever called by one agent | Move the skill inside `agents/` — it is a private implementation detail, not a shared playbook | Skills directory fills with one-off procedures that appear reusable but are not; consumers break when the single caller's contract changes |
| Temporary planning docs remain in `docs/` after the migration they describe is complete | Delete them — `docs/` is source-of-truth only; stale plans create false signal for agents | Agents reading docs/ receive outdated architectural descriptions that contradict current behavior |

## Known Limits / Failure Modes

- **Premature migration:** Moving files from the artifact store to knowledge/ before knowledge update loops are implemented adds overhead without benefit. The migration should happen when agents are about to read knowledge across runs, not before.
- **Temporary docs accumulation:** Without active enforcement, temporary planning documents accumulate in `docs/`. Enforce a cleanup rule: temporary docs are deleted when the migration or refactor they describe is complete.
- **Skill boundary drift:** Over time, skills that started as shared playbooks get repurposed as private wiring for a single agent. Periodic audits should check whether each skill in `skills/` is genuinely reusable.
