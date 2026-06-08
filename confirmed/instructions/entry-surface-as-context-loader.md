# Entry Surface as Context Loader

Status: confirmed
Last reviewed: 2026-06-08

## Principle

A skill or agent entry surface — prompt.md, CLAUDE.md, or equivalent entry point — has one permitted responsibility: declaring which context to load for the current task. It is not a policy document, a phase runbook, a feature specification, or a CLI reference. Every sentence in an entry surface that does not declare what to load is a sentence that belongs in a different file.

When an entry surface accumulates policy, it becomes a secondary policy owner. It will drift from the canonical policy files. The model reads the entry surface first; stale policy in the entry surface overrides fresh policy in the canonical file based on position, not authority. The entry surface thereby becomes the highest-authority document in the model's context — the opposite of its intended role.

The correct entry surface for a 4-phase skill is fewer than 30 lines:

```markdown
# Skill Entry

Self-identify as: <agent-name>

For Phase 1 — Prepare:
Load: agents/orchestrator/AGENT.md, references/prepare-gates.md

For Phase 2 — Analyze:
Load: agents/analyzer/AGENT.md, references/checkpoints.md

For Phase 3 — Generate:
Load: agents/generator/AGENT.md, references/generation-policy.md

For Phase 4 — Verify:
Load: agents/verifier/AGENT.md, references/verify-policy.md
```

Everything else — what each phase does, what gates must pass, what policies apply — belongs in the loaded files, not in the entry surface.

This principle is distinct from `entry-adapter-as-runtime-surface.md` (candidates). That principle addresses portability: separating the agent skeleton from runtime-specific adapter files. The present principle addresses content: what belongs inside the entry file regardless of runtime. The two principles compose: the entry adapter file should contain only loading declarations, not policy content.

## Evidence

### Academic Papers

- **Codified Context: Infrastructure for AI Agents in a Complex Codebase** (arXiv:2602.20478, Feb 2026, Vasilopoulos) — Proposes separating a hot-memory constitution (immediate access, entry-level) from cold-memory specification documents (on-demand specialist knowledge), validated across 283 sessions on a 108,000-line C# system. The architecture implies that the entry-level constitution should route to specialist documents rather than contain specialist content — conflating the two means the always-loaded entry tier carries knowledge that should be loaded on demand, wasting context and potentially overriding more specific on-demand documents.

- **Lost in the Middle: How Language Models Use Long Contexts** (arXiv:2307.03172, Jul 2023, Stanford) — Demonstrates that information in early context positions receives disproportionate weight. An entry surface that carries policy content makes that policy early-context; subsequent loaded files that carry authoritative versions of the same policy arrive mid-context. The model preferentially follows the entry-surface version regardless of which is current.

- **Evaluating AGENTS.md: Are Repository-Level Context Files Helpful for Coding Agents?** (arXiv:2602.11988, Feb 2026, Gloaguen et al.) — Finds that repository context files (AGENTS.md) *reduce* task success rates compared to providing no context, while increasing inference cost by over 20%. The finding is that "unnecessary requirements from context files make tasks harder" and the recommendation is that context files should "describe only minimal requirements." Directly validates the principle that entry surfaces should carry only what is necessary — not accumulate comprehensive policy content.

- **Arbiter: Detecting Interference in LLM Agent System Prompts** (arXiv:2603.08993, Mar 2026, Mason) — Finds 152 interference instances across Claude Code, Codex CLI, and Gemini CLI prompts. Establishes that "prompt architecture (monolithic, flat, modular) strongly correlates with observed failure class" — monolithic prompts accumulate policy from multiple sources and produce distinct interference patterns compared to modular architectures. Directly validates that concentrated policy in a single entry surface creates interference that modular separation prevents.

### Official Best Practices

- **Anthropic — Building Effective Agents** (https://www.anthropic.com/research/building-effective-agents, 2024) — Prescribes "maintain simplicity in your agent's design" as a core principle. The architectural patterns described (prompt chaining, routing, parallelization) presuppose that entry points route to specialized agents rather than carrying the full behavioral specification. The pattern library implies thin entry routing + specialized agent documents as the correct structure.

- **Anthropic — Claude Code Best Practices** (https://code.claude.com/docs/en/best-practices, 2025–2026) — Prescribes CLAUDE.md as a minimal, concise routing layer: "CLAUDE.md is loaded every session, so only include things that apply broadly. For domain knowledge or workflows that are only relevant sometimes, use skills instead." Identifies the "over-specified CLAUDE.md" as a named failure pattern: "If your CLAUDE.md is too long, Claude ignores half of it because important rules get lost in the noise." Directly validates the entry-surface-as-loader principle.

- **OpenAI Agents SDK — Projected Views** (https://openai.github.io/openai-agents-python/agents/, 2025) — Recommends that the agent object's `instructions` field carry "role identity and loading instructions"; behavioral policy and domain knowledge belong in tool outputs or dynamically loaded documents. The SDK architecture treats the instructions field as the entry surface and separates it from content.

### Named in Literature?

No unified name. Closest named concepts: "minimal entry requirements" (arXiv:2602.11988), "hot-memory constitution" (arXiv:2602.20478), "context loading declaration" (engineering practice). The specific formulation — that entry surfaces must contain only loading declarations and that policy content accumulated in entry surfaces overrides owner documents via position bias — is not named as a single pattern.

## When to Apply

| Signal | Decision | What breaks if ignored |
|---|---|---|
| Entry prompt contains a phase runbook section | Extract to phase-specific agent document; replace with load declaration | Entry prompt grows with each phase; early-position bias makes entry content override phase docs |
| Entry prompt contains a policy statement also in a reference doc | Remove from entry prompt; reference doc is the owner | Model follows entry-prompt version after reference doc is updated |
| Feature boundary ("v1", "beta") appears in entry prompt | Move to feature owner doc; entry prompt has one-line pointer | Feature version is locked in entry surface; requires coordinated updates across files |
| Entry prompt length grows with each new feature | Each growth event is a sign that policy was deposited in the wrong layer | Entry prompt becomes the de facto policy owner; canonical policy files are ignored |
| CLI command list appears in entry prompt | CLI commands belong in CLI reference or agent projection; entry prompt loads that reference | Internal implementation details leak to public entry surface |

Do not apply when: the entry surface is a single-agent, single-file system with no separate reference documents. In that case, the entry file is necessarily the owner of all content. This principle applies when a document ecosystem exists and an entry surface is being distinguished from owner documents within it.

## Known Limits

- **Minimal entry surfaces require reliable load-path delivery**: Replacing policy in the entry surface with a load declaration is only safe if the loading mechanism reliably delivers the referenced file. If projected view construction is inconsistent or the reference file may be absent, minimal entry surfaces break silently. Validate load paths before thinning the entry surface.

- **Aggressive thinning can remove essential context**: Some context must arrive early because later-loaded documents depend on it being already present. Removing all content from the entry surface can cause loaded documents to lack the framing they assume. Keep framing context (agent identity, task scope, what is being done and why) in the entry surface; move only policy rules and procedural runbooks to owner documents.

- **Load declaration ambiguity**: "Load references/policy.md" is only unambiguous if there is one policy file with that name. When multiple policy files might match, loading declarations become a resolution problem. Name loaded files unambiguously; version them if necessary.

- **Does not apply to single-file systems**: This principle requires a file ecosystem to be meaningful. For a single-agent, single-file Claude/Codex deployment, the entry file is the only file and must carry all content. Applying this principle in that context produces a loading declaration with no file to load.

## Promotion History

Candidate — created 2026-06-07 from observed failure: browser-flow prompt.md accumulated Compose feature boundaries, Phase 5 extract definitions, replay permission policy, and 15+ inline behavioral rules over successive feature additions. Result: entry surface was longer than most owner documents and served as de facto policy owner, causing model to follow stale entry-surface versions after owner files were updated. Related to but distinct from `candidates/entry-adapter-as-runtime-surface.md`.

Confirmed — 2026-06-08. 5-dimension score: 20/25 (코어성 4, 리스크감소 4, 확장성 4, 제어성 4, 기록성 4). Gate 3 (Fact): arXiv:2602.11988 directly measures the failure mode (context files reduce task success rate). Four independent academic papers, three official best practice sources. Known Limits and When Not to Apply present.
