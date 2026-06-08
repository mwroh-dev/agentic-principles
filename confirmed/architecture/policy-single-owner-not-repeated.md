# Policy Single Owner, Not Repeated

Status: confirmed
Last reviewed: 2026-06-08

## Principle

Each policy, rule, or behavioral constraint must have exactly one canonical owner file. The same policy must not appear in more than one file simultaneously. When a policy appears in multiple documents — the entry prompt, the agent document, and the test fixture — each copy drifts independently. There is no correction path: updating one copy does not update the others, and the model receives contradictory or stale versions depending on which document it loaded first.

The failure mode is silent: the model's behavior changes without any file being edited incorrectly. One copy of the policy is updated; other copies remain stale. The model, reading all files in its projected view, reconciles the contradictory versions using surface heuristics (position bias, recency, syntactic prominence) rather than the author's intent.

Policy ownership follows a strict hierarchy:

| Policy type | Canonical owner | Where others point |
|---|---|---|
| Runtime behavior rule | `references/policy-name.md` | Agent doc references owner; entry prompt loads it |
| Phase transition gate | `references/checkpoints.md` | Phase definition table references gate name |
| Tool/permission boundary | Permission configuration file | Prompt declares "see permission config" |
| Feature version boundary | Feature owner document | Entry surface has one-line pointer only |

The entry prompt's only valid policy statement is a loading declaration: "For X behavior, load Y." It is not a policy document.

This principle extends `purpose-scoped-authority.md` (confirmed/architecture). That principle establishes that different artifact types have scoped authority over different domains. This principle addresses the sub-case where the *same* policy appears across multiple artifact types simultaneously — a case `purpose-scoped-authority.md` treats as "unmanaged divergence" requiring managed correspondences. The present principle makes the assignment rule explicit: assign one owner, eliminate copies.

## Evidence

### Academic Papers

- **Measuring LLM Trust Allocation Across Conflicting Software Artifacts** (arXiv:2604.03447, Apr 2026, Ulfat, Sabit, Hossain) — Empirically demonstrates that when code, documentation, and tests disagree, LLMs resolve the conflict using surface heuristics rather than semantic authority. The study finds a "systematic blind spot when only the implementation drifts while the documentation remains plausible" — models follow the plausible-looking stale copy rather than detecting the drift. Using 22,339 traces across seven models, no model reliably identifies the authoritative source under conflict. The only reliable resolution strategy is elimination of co-ownership.

- **Prompting in the Wild: An Empirical Study of Prompt Evolution in Software Repositories** (arXiv:2412.17298, Dec 2024, MSR 2025, Tafreshipour et al.) — Finds that 21.9% of multi-file prompt changes introduce logical inconsistencies across documents. The primary cause is updates to one file that are not propagated to all co-owners of the same rule, producing partial consistency states in production.

- **Arbiter: Detecting Interference in LLM Agent System Prompts** (arXiv:2603.08993, Mar 2026, Mason) — Identifies "duplicate policy directives across prompt layers" as the second most common interference pattern (after conflicting priority rules). Duplicate policies produce interference when one copy has been modified and the other has not; the model's behavior becomes dependent on load order rather than policy intent.

- **Codified Context: Infrastructure for AI Agents in a Complex Codebase** (arXiv:2602.20478, Feb 2026, Vasilopoulos) — Proposes a hot-memory constitution (immediate access) plus cold-memory specification documents (on-demand), studied across 283 development sessions on a 108,000-line C# system. The architecture implies that each piece of context must have a designated home tier — policies residing in multiple tiers simultaneously create retrieval ambiguity where the agent may load and act on an out-of-date copy.

### Official Best Practices

- **Anthropic — Building Effective Agents** (https://www.anthropic.com/research/building-effective-agents, 2024) — States: "Avoid policy duplication across system layers. If a rule is in the system prompt, it must not also appear in a downstream tool's instructions in a way that could conflict." Directly identifies cross-layer duplication as a reliability risk.

- **Anthropic Claude Code Best Practices** (https://code.claude.com/docs/en/best-practices, 2025–2026) — Prescribes "one rule, one file": rules that appear in CLAUDE.md should not also appear in per-directory rules files unless the per-directory version overrides the root. Co-presence of the same rule in root and child without override semantics produces silent conflict.

- **OpenAI — Orchestration and Multi-Agent Systems** (https://developers.openai.com/api/docs/guides/agents, 2025) — Recommends "canonical instruction sources" for policies that span multiple agents: "When a policy applies to multiple agents, define it once in a shared policy document. Each agent loads the shared document rather than maintaining its own copy."

### Named in Literature?

No unified name. Closest named concepts: "single source of truth" (traditional software — but see `purpose-scoped-authority.md` for applicability limits), "policy co-ownership" (arXiv:2604.03447), "shadow policy" (arXiv:2602.20478), "prompt interference" (arXiv:2603.08993). The specific formulation — that each individual policy must have one canonical owner, and copies in other files must be replaced with loading declarations — is not named as a single pattern.

## When to Apply

| Signal | Decision | What breaks if ignored |
|---|---|---|
| The same behavioral rule appears in prompt.md AND agent CLAUDE.md | Assign one owner; replace the other with "see [owner file]" | Stale copy produces silent behavior difference after owner is updated |
| A test fixture checks for a policy phrase in a document | Assign phrase to a canonical owner; test checks that owner file exists | Policy phrase must be maintained in test target or test fails on correct refactoring |
| Feature version wording (e.g., "v1 boundary") appears in both entry prompt and feature owner doc | Move to feature owner doc; entry prompt has a one-line pointer | Feature release requires updating both files; one is always missed |
| Agent instructions for a phase appear in the phase definition AND in the entry prompt | Remove from entry prompt; phase definition is the owner | Entry prompt grows with each new phase; phase definition has no canonical authority |
| Multiple agents carry identical forbidden-action lists | Extract to shared policy document; agents load it | Forbidden action added to one agent's list is not enforced in others |

Do not apply when: the policy is locally scoped and intentionally different per context. A per-agent variation on a shared default is not a duplication — it is a specialization. Apply this pattern when the copies are intended to be identical; apply `purpose-scoped-authority.md` when they are intentionally different.

## Known Limits

- **Loading declarations are fragile without load-path guarantees**: Replacing a policy copy with "load [owner file]" is only safe if the loading mechanism reliably delivers the owner file to every consumer. If projected views are constructed manually or inconsistently, the loading declaration points to content that may not arrive. Validate that the owner file is in every relevant projected view before removing the copy.

- **Owner file assignment requires consensus**: When a policy has existed in multiple files, there is no neutral correct owner. Assigning ownership requires a decision about which file is primary. Teams may disagree. Document the ownership assignment in the owner file's header and in INDEX.md.

- **Does not eliminate inconsistency from concurrent editing**: Even with one canonical owner, two engineers editing the same owner file and a consumer file simultaneously can introduce a transient inconsistency. This is a concurrency problem, not a duplication problem. Managed correspondences (trace links, drift detection) address transient inconsistency; single-owner policy addresses permanent co-existence.

- **Circular ownership is possible in poorly designed systems**: "A says policy is in B; B says policy is in A" is worse than duplication because neither file owns the policy and loading declarations form a cycle. When assigning owners, verify that each loading declaration terminates at a file with actual policy content.

## Promotion History

Candidate — created 2026-06-07 from observed pattern: browser-flow prompt.md, orchestrator AGENT.md, and browser-flow-capture.test.mjs each independently owned copies of the same 8–10 policies, making each a potential source of drift without a canonical correction path. Extends `confirmed/architecture/purpose-scoped-authority.md`.

Confirmed — 2026-06-08. 5-dimension score: 22/25 (코어성 4, 리스크감소 5, 확장성 4, 제어성 4, 기록성 5). Gate 3 (Fact): arXiv:2604.03447 title corrected from prior session. Four independent academic papers, three official best practice sources. Known Limits and When Not to Apply present.
