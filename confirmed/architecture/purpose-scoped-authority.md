# Purpose-Scoped Authoritative Artifacts

Status: confirmed
Last reviewed: 2026-05-18

## Principle

A multi-artifact system does not converge on a single source of truth. Different artifact types serve different audiences with different semantics — forcing them into one canonical form degrades each artifact's fitness for its purpose.

The correct model: each artifact type holds **scoped authority** over its own domain. A code artifact is authoritative over enforcement semantics. A specification artifact is authoritative over intent. A knowledge artifact is authoritative over learned patterns. None overrides the others; each is first-class.

The distinction that matters is not "single vs multiple" but **managed vs unmanaged**. Multiple authoritative artifacts with explicit consistency management is a sound architecture. Multiple artifacts that diverge silently is a defect.

SSOT is appropriate only when artifacts are formal, derivable, and machine-checkable — design tokens that generate CSS, hardware DSLs that derive RTL. When artifacts represent policy, rationale, model instructions, or operational knowledge, forced unification obscures rather than clarifies.

The replacement: not one canonical source, but **managed correspondences** — trace links, consistency checks, drift reviews, and promotion paths between independently authoritative artifacts.

## Evidence

### Standards and Architecture Research

- **ISO 42010:2011** (iso.org/standard/50508.html) — Defines architecture description in terms of viewpoints and views: distinct representations for distinct stakeholder concerns. Does not prescribe a single view covering all concerns; requires correspondences between views to maintain consistency.
- **Kruchten 4+1 Architectural View Model** (Kruchten, arXiv:2006.04975) — Four concurrent views (logical, process, development, physical) plus scenarios. Multiple views are the prescribed form, not a workaround; each addresses different concerns for different stakeholders.
- **Architecture Description Standard** (archstandard.org/v1/standard/3-views-overview/) — Explicitly states "No single view provides a complete picture." Requires view-to-view correspondences and traceability rather than consolidation.

### Consistency and Traceability Research

- **UML Multi-View Consistency** (arXiv:1610.03960) — Frames the problem as "how to check consistency across multiple views," not "how to eliminate views." Treats heterogeneous representation as the normal starting condition.
- **Heterogeneous Artifacts Consistency** (arXiv:2103.14860) — Requirements, spec, and code co-exist as separate artifacts; the research challenge is consistency checking, not elimination of heterogeneity.
- **Traceability SLR** (arXiv:2603.16208) — Artifact ecosystems are inherently multi-artifact. The focus is link management and role alignment, not consolidation.
- **Link Degradation Research** (arXiv:2104.05891) — Manual reference links break over time; independent artifacts diverge silently when only linked by URL. Managed correspondences with consistency checks outperform both copy-and-maintain and link-only approaches.

### LLM-Specific Evidence

- **LLM Artifact Conflict** (arXiv:2604.03447) — When code, documentation, and tests conflict, LLMs preferentially trust the wrong artifact. Artifacts with explicit scope declarations reduce the likelihood of LLM misattribution under conflict.

### SSOT Applicability Boundary

- **Design Tokens** (Fowler, martinfowler.com/articles/design-token-based-ui-architecture.html) — SSOT applies when artifacts are mechanically derivable from a formal source: token file → generated CSS/code is a valid SSOT application.
- **PEak Hardware DSL** (arXiv:2308.13106) — Functional model, spec, and RTL derived from a single formal source. Valid SSOT in formal, machine-checkable domains.

## When to Apply

| Signal | Decision | What breaks if ignored |
|--------|----------|------------------------|
| Multiple artifact types (code, spec, docs, knowledge, skills) serve different audiences and purposes | Assign scoped authority per artifact type; do not consolidate | Forced consolidation degrades each artifact's fitness; code becomes annotated policy; docs become runtime-injected instructions |
| Artifacts are diverging silently with no consistency mechanism | Add trace links, consistency checks, and drift reviews — do not eliminate artifacts | Silent divergence produces LLM misattribution: the agent trusts the wrong artifact when they conflict |
| A team proposes SSOT consolidation to reduce duplication | Check whether artifacts are formally derivable from the proposed source; if not, reject consolidation in favor of managed correspondences | Non-derivable SSOT conflates multiple concerns; updates for one concern silently break another's semantics |
| An LLM agent reads artifacts that can contradict each other | Declare explicit scope for each artifact (what it is authoritative over) in its header | Agents reading conflicting artifacts without scope declarations apply wrong semantics — enforcing policy from a rationale doc, or learning from a non-authoritative copy |
| Artifacts must evolve independently at different rates | Preserve independence; establish a promotion path for deliberate synchronization | Forced synchronization slows evolution of stable artifacts to match volatile ones; treating them as one artifact risks all concerns on every update |

## Known Limits / Failure Modes

- **Correspondence overhead:** Explicit trace links, consistency checks, and drift reviews add coordination cost. For small teams or early-stage systems where artifact purposes have not yet diverged, SSOT is simpler and appropriate. Apply federated authority only when artifact purposes genuinely differ.
- **Scope boundary ambiguity:** Scoped authority requires each artifact's authority domain to be explicitly declared. Without declaration, the default assumption is "most recent artifact wins," which reintroduces unmanaged divergence silently.
- **Consistency check decay:** A consistency check suite that is not maintained and run after each artifact update produces the appearance of managed correspondences without the substance. Consistency checks must be treated as a first-class artifact, not optional tooling.
- **SSOT applicability misread:** SSOT is appropriate for formally derivable artifacts. Applying federated authority to design tokens or machine-checkable schemas adds unnecessary complexity where a derivation pipeline is the right mechanism.
