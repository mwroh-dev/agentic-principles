# Artifact Placement Decision Framework

Status: confirmed
Last reviewed: 2026-05-18
Score: 8.2/11

## Principle

Any proposed artifact must pass through a 5-question placement test before it is committed to any location in the repository.

Apply in order; the first YES determines primary placement. If a second question also matches, see Known Limits for cross-cutting resolution. Question 1 (runtime enforcement) always takes priority.

1. Does this control runtime branching or require automated enforcement? → **runtime code layer**
2. Is this a human-facing rationale, policy explanation, or design decision? → **documentation layer** (ADR for decisions with rationale; general docs for explanations)
3. Is this a reusable procedure an agent executes repeatedly? → **procedure layer**
4. Is this an evolving heuristic, observed pattern, or not-yet-codified learning? → **semantic knowledge layer**
5. Does this span multiple stakeholder concern types (e.g., both a runtime invariant and a human-facing policy)? → **mirror across layers, designate one as primary**

Note on Q3: a procedure, as used here, is a reusable, named procedure that an agent invokes to complete a specific task — distinct from code libraries, which are imported, and from documentation, which is read but not executed.

Placement decisions are determined by stakeholder concern, enforcement need, and change rate — not by file type or convenience. Default to the procedure layer; promote to runtime code only when the invariant is empirically confirmed across multiple independent contexts.

Runtime semantics cannot be replaced by documentation. A document describing a constraint does not enforce it; only code enforces it. Placing procedural knowledge in the documentation layer produces drift; placing rationale in code produces fragility. A repository separates the **runtime code layer** (enforces invariants, cannot be bypassed), the **procedure layer** (reusable agent procedures invoked by name), the **episodic record layer** (immutable run records, audit trail — often excluded from version control), the **semantic knowledge layer** (evolving heuristics, committed), and the **documentation layer** (human-facing rationale, ADRs) because each layer answers a different stakeholder question. Conflating layers does not simplify the system — it removes the ability to reason about authority, change rate, and enforcement boundary independently.

## Evidence

### Stakeholder-Concern-Based Artifact Separation
- **ISO 42010:2011** (iso.org/standard/50508.html) — defines architecture description through stakeholder-concern-based viewpoints; different stakeholder concerns require distinct artifact types, and a single artifact cannot satisfy multiple incompatible concerns without losing precision
- **Kruchten 4+1 Architectural View Model** (arXiv:2006.04975) — four concurrent views each address distinct stakeholder concerns; multiple views are the correct structural form, not a workaround or redundancy to be eliminated
- **Architecture Description Standard** (archstandard.org/v1/standard/3-views-overview/) — asserts that "No single view provides a complete picture" and requires explicit correspondences between views to maintain coherence across artifact types

### Multi-Artifact Coexistence as Normal Condition
- **Multi-view consistency SLR** (link.springer.com/article/10.1007/s10270-018-00713-w) — heterogeneous artifact coexistence is the normal condition in real systems; the research challenge is consistency management between artifact types, not elimination of heterogeneity
- **Easterbrook et al. on managed inconsistency** (journals.sagepub.com/doi/10.1177/1063293X9400200307) — forced complete consistency between all views is neither achievable nor desirable; tolerance and managed inconsistency between artifact types are required properties of large systems

### Decision Rationale Belongs in Dedicated Artifacts
- **Google Architecture Decision Records** (docs.cloud.google.com/architecture/architecture-decision-records) — decisions with rationale belong in lightweight docs co-located with code, not in runtime artifacts; the record must be independently reviewable without executing code
- **Azure ADR guidance** (learn.microsoft.com/en-ie/azure/well-architected/architect-role/architecture-decision-record) — architecture decisions must be traceable and separately maintained from code; embedding rationale in code comments makes it invisible to non-code reviewers and eliminates traceability

### Artifact Role Alignment and Link Management
- **Traceability SLR** (arxiv.org/abs/2603.16208) — artifact ecosystems are inherently multi-artifact; the focus of sound artifact management is link management and role alignment between artifact types, not consolidation into fewer files

## When to Apply

| Signal | Decision | What breaks if ignored |
|--------|----------|------------------------|
| New domain-specific data discovered (service constraints, auth patterns, rate limits, vendor-specific friction) | Place in the semantic knowledge layer, not in the procedure layer or runtime code | Knowledge embedded in procedure-layer content produces stale procedures when patterns change; knowledge embedded in code requires a deployment for every observed pattern update |
| A new repeatable agent procedure is needed (deployment rollback sequence, data migration workflow, incident response checklist, code review gate) | Place in the procedure layer, not in the documentation layer or runtime code | Procedures in the documentation layer are read but not executed; an agent cannot invoke documentation as a workflow |
| A new schema, output shape, or enforcement boundary must be consistent across runs | Place in runtime code (validators, type definitions, contract tests), not in the documentation layer or procedure layer | Documentation-encoded schemas drift from runtime behavior silently; the discrepancy is detected only when production fails |
| A design decision needs to be recorded with rationale for human review | Write an ADR in the documentation layer, not as a code comment | Code comments decay and lose context; rationale is invisible to reviewers who read documentation but not inline code comments |
| An artifact genuinely serves two stakeholder concern types simultaneously (e.g., both a code-enforced constraint AND a policy reviewable by humans) | Mirror across both layers; designate one as primary with a pointer from the derived view back to the primary | Dual-primary ownership produces silent divergence — two copies both edited independently, no detection mechanism |

> **Placement-by-convenience check:** Before committing any artifact: verify placement against the 5-question test. Convenience is not a valid placement criterion. An artifact placed in the wrong layer creates false authority — an agent reading rationale stored in a procedure layer will treat it as executable instruction.

## Known Limits

- **Code/procedure boundary:** The boundary between procedure-layer content and runtime code enforcement follows one rule: a procedure that must never vary is code; a procedure that adapts to context belongs in the procedure layer. When in doubt, start in the procedure layer and promote to runtime code when the invariant is confirmed empirically across multiple independent contexts.

- **Semantic knowledge → procedure → code promotion path:** Without an explicit promotion trigger, knowledge accumulates indefinitely without becoming executable. Define a promotion condition for each step: a knowledge entry that is referenced or found relevant in three or more independent decision contexts (runs, PRs, incidents, debugging sessions) becomes a procedure candidate; a procedure that must hold as an invariant across all contexts becomes a runtime code-enforced contract. Without this trigger, the semantic knowledge layer becomes a graveyard of unactioned observations.

- **ADR decay:** ADRs require active maintenance to remain current. In distributed teams or separate repositories, ADRs decay and lose traceability without periodic review. Schedule ADR audits whenever the system they describe undergoes significant change; an ADR that describes a superseded decision without marking itself superseded is actively harmful — it misleads future reviewers about the current authoritative design.

- **Cross-cutting placement:** Some artifacts genuinely span multiple concern types (e.g., a security policy that is both a documentation-layer record for humans and a runtime code-enforced invariant). These require explicit mirroring with declared authority: one layer is designated primary, the other is a derived view with a pointer back to the primary. When two layers both apply, runtime enforcement (code) takes precedence as primary. Documentation-layer and procedure-layer representations are derived views. A derived view must include a pointer to the primary: `Source of truth: [primary layer reference]`. Dual-primary ownership — where both representations can be independently edited — produces silent divergence with no mechanism for detection or resolution.

## Local Mapping (Project-Specific)

Before applying this framework, identify which directories in your project correspond to each abstract layer. Inject the completed mapping as a local note alongside this document.

| Abstract Layer | Example Mapping | Authority |
|---------------|----------------|-----------|
| Runtime code | `src/`, `agents/`, `lib/` | Enforces invariants |
| Procedure | `skills/`, `playbooks/`, `workflows/` | Agent-invokable |
| Episodic record | `artifacts/`, `runs/`, `.runs/` | Immutable, often gitignored |
| Semantic knowledge | `knowledge/`, `context/`, `memory/` | Evolving, committed |
| Documentation | `docs/`, `adr/`, `decisions/` | Human-facing |

The abstract layer names used throughout this document are intentionally generic. The concrete directory names in any given project are implementation choices, not the framework itself. The framework governs which concern type belongs in which layer — the mapping of layers to paths is a local decision.
