# Skill Discovery Is Not Authorization

Status: confirmed
Last reviewed: 2026-05-26

## Principle

A discoverable skill is a model-visible capability surface, not an agent-private implementation detail. The execution surface — how a skill runs — is structurally separate from visibility and authority — who may invoke it. Discoverability is itself a capability grant: once a skill enters the model's discovery surface, any agent with the `Skill` tool can observe and invoke it unless an explicit mechanism prevents this.

What must enforce exclusivity: tool allowlists and denylists (`disallowedTools: Skill`, `Skill(name)` permission rules), scoped subagent configuration, and discovery surface control. The strongest restriction is `gateway_only` placement, which removes the skill from model-driven discovery entirely and routes invocation through a harness-level typed runtime command where caller identity can be checked before execution.

What must not be used as enforcement: folder naming (`skills/` placement), natural-language descriptions ("private — internal use only"), `skills:` frontmatter preload (which is content injection into a specific subagent context, not an access restriction), and `allow_implicit_invocation: false` or `disable-model-invocation: true` alone (these block model-driven invocation but leave explicit invocation paths open). None of these mechanisms produce an authorization boundary; they influence model behavior while the underlying invocation path remains accessible.

**Skill discovery is not authorization. An agent may use a skill, but a skill is not automatically owned by that agent. Prompt instructions are not enforcement.**

| Term | Definition |
|---|---|
| Agent | A model instance with a defined tool set, context window, and invocation scope. Identity is determined by configuration, not by prompt role claims. |
| Skill | A structured procedure file that the model can discover and invoke via the `Skill` tool, or that a user can invoke explicitly. Skills are not inherently scoped to any agent. |
| Tool / Command / MCP | Primitive callable actions. Tools are lower-level than skills; skills may invoke tools. Authorization at the tool layer (`disallowedTools`) is distinct from authorization at the skill layer. |
| Runtime gateway | A harness-level mechanism that intercepts skill invocation requests and enforces caller identity checks before execution. Not a current platform primitive — requires custom implementation. |
| Orchestrator | The top-level agent responsible for task decomposition and delegation. Distinguished from specialist agents by role, not by model capability. |

**Visibility × Authority matrix:**

| | Authorized | Not authorized |
|---|---|---|
| **Discoverable** | Model sees and can invoke — intended for public shared utilities | Model sees but must be blocked — **this is the failure mode** |
| **Not discoverable** | Agent can invoke via typed command (gateway_only) | Skill effectively disabled for this agent |

The four visibility/authority axis values:

- **`public_discoverable`**: visible to model/root session; any agent with `Skill` tool can invoke. Discoverability implies no authorization restriction.
- **`agent_preloaded`**: content injected into a specific subagent context via `skills:` frontmatter. NOT an allowlist. Quote: "This field controls which skills are preloaded, not which skills the subagent can access." (Claude Code docs)
- **`manual_only`**: not auto-invoked by model. Claude Code: `disable-model-invocation: true` blocks model-driven invocation but user `/skill-name` still works. Codex: `allow_implicit_invocation: false` blocks implicit invocation but explicit `$skill` still works. Both mechanisms lack authorization boundary guarantees; neither replaces allowlist enforcement.
- **`gateway_only`**: not discoverable by model; invoked only through a typed runtime command at the harness layer. Strongest restriction; requires custom harness implementation.

## Boundary with Existing Principles

| Principle | Relationship | What this principle adds |
|---|---|---|
| `agent-directory-structure.md` | Shared basis ("shared playbooks, not private") | Placement alone does not control who may invoke |
| `skill-surface-types.md` | Orthogonal | Execution surface ≠ authorization scope |
| `rule-is-not-skill.md` | Same family, different axis | rule-is-not-skill separates the enforcement LAYER from the invocation LAYER; this principle separates discovery (visibility) from authorization within the skill access path |
| `stage-scoped-tool-authority.md` (candidate) | Complementary | Tool scoping at dispatch-time ≠ skill visibility control |
| `multi-layered-safety-via-code.md` | Parent | This principle is an application of that principle to skill discovery specifically |

## Failure Modes

**Delegation bypass**: The orchestrator sees a specialist skill in the discovery surface and invokes it directly instead of delegating to the specialist agent. The pipeline stage collapses, the specialist contract is violated, and failure attribution is lost because the execution trace shows the orchestrator performing specialist work.

**Privilege escalation**: An agent invokes a skill beyond its authorized scope. Per arXiv:2601.11893 (SEAgent): "agent actions exceeding the least privilege required for a user's intended task." Discoverability without authorization enforcement makes every visible skill a potential escalation vector for every agent that holds the `Skill` tool.

**Confused deputy**: A skill executes on behalf of a caller with more authority than the skill should accept. Per arXiv:2512.06914 (Trust-Authorization Mismatch): "desynchronization between dynamic trust states and static authorization boundaries." When authorization is not checked at execution time, a skill inherits the authority of its caller regardless of whether that authority is appropriate.

**Boundary erosion**: When tool and skill grants are not stage-scoped, pipeline stages collapse as any agent invokes any visible skill across all stages simultaneously. The intended separation between compilation, validation, and execution stages dissolves, and the system behaves as a single flat execution context.

```
Bad flow:
user request
→ orchestrator sees `sheet-ops` skill in discovery surface
→ orchestrator reads skill and repo files
→ orchestrator constructs execution JSON and runs shell
→ specialist agent bypassed; delegation tree collapses

Correct flow:
user request
→ orchestrator classifies and delegates to approved specialist
→ specialist receives role-scoped context and tools
→ harness executes typed runtime command
→ orchestrator receives result and summarizes
```

## Anti-Patterns

1. "It is under `skills/`, therefore any agent may use it."
2. "It is only mentioned in one agent's prompt, therefore other agents cannot use it."
3. "The orchestrator prompt says 'do not use this skill', therefore it is safe."
4. "The skill description says 'private — for internal use only.'"
5. "`skills:` preload is an allowlist — I listed only the skills this agent needs."
6. "Disabling implicit invocation means disabling all invocation."
7. "A mode is a boundary."
8. "Folder naming enforces access control."

## Claude Code Notes

Restrict coordinator access: the coordinator agent should have `disallowedTools: Skill` unless invocation is intentionally justified and documented with explicit rationale. Use `tools` and `disallowedTools` fields to scope tool access per-subagent. `Agent(type)` in the `tools:` field restricts which subagent types a coordinator can spawn when running with `claude --agent`.

`skills:` is preload only — NOT an allowlist. Listing skills under `skills:` injects their content into the subagent context window; it does not restrict the subagent from discovering and invoking other skills. To restrict, use `disallowedTools: Skill` (blocks all skill invocation) or `Skill(name)` permission rules (blocks named skills).

`disable-model-invocation: true` blocks model-driven invocation and preloading; user `/skill-name` still works. Scope this mechanism to agent-to-agent authorization contexts, not to user-facing skill access.

`allowed-tools` in skill frontmatter pre-approves tools for that skill (reduces permission prompts during execution) but does NOT restrict available tools — "every tool remains callable" (Claude Code docs). This field is a UX convenience, not an authorization boundary.

`skillOverrides: { "name": "off" }` hides a skill from model discovery but does NOT prevent tool-level invocation. It must be combined with `disallowedTools` rules to achieve actual restriction.

```yaml
# Coordinator: routes only — does not execute skills
---
name: sheet-orchestrator
description: Routes sheet tasks to approved specialists. Does not execute skills.
tools: Agent(sheet-open-compiler, sheet-structured-runner, sheet-verifier)
disallowedTools: Skill, Bash, Read, Grep, Glob, Edit, Write
---
You are a router, not an executor.
```

```yaml
# Specialist: receives preloaded playbook — skills: is context injection only
---
name: sheet-open-compiler
description: Compiles open natural-language sheet requests into typed execution requests.
skills:
  - sheet-open-compiler-playbook
tools: Read, Grep, Bash(prepare-use *)
---
Use the preloaded playbook. Do not execute runtime commands directly.
```

## Codex Notes

Root `.agents/skills/` files are model-visible and discoverable. Treat this directory as a public capability surface: any skill placed here is accessible to any agent that can invoke the `$skill` command unless additional measures are taken.

Use `allow_implicit_invocation: false` to reduce model-driven auto-activation. This does NOT block explicit `$skill` invocation — it is not equivalent to an authorization boundary. It is a defense-in-depth measure, not a primary control.

**Codex does not have a documented per-agent skill allowlist equivalent to Claude Code's `Skill(name)` permission rules.** For strong per-agent boundaries, implement a harness-level gateway: caller identity check, allowed skill/capability map, typed request envelope, deterministic allow/deny before loading the implementation body.

Do not assume Claude Code permission semantics apply to Codex deployments. The two platforms have different authorization primitives; a boundary that exists in Claude Code via `Skill(name)` deny rules must be recreated in Codex via custom harness enforcement.

```
.agents/skills/
  sheet-ops-entry/
    SKILL.md      # public, thin, capture-only — no execution logic

.codex/agents/
  sheet-orchestrator.toml     # routes only
  sheet-open-compiler.toml    # receives internal playbook
  ...

internal/
  sheet-ops/
    compiler-playbook.md      # not in public discovery surface
    runtime-playbook.md

cmd/
  prepare-use
  use-open
  run-validated
```

## Implementation Checklist

1. Orchestrator agent has no direct `Skill` tool access unless intentionally justified and documented.
2. Orchestrator cannot run broad shell or file-read tools by default (limits filesystem-level bypass).
3. Orchestrator can only spawn approved specialist agent types.
4. Specialist agents receive only their required playbooks, skills, and tools — not the full shared `skills/` surface.
5. Skills in the shared `skills/` namespace are thin capture/delegation adapters — no side-effecting execution at the discovery layer.
6. Side-effecting operations live behind typed runtime commands at the harness layer.
7. Skill discovery placement (`public_discoverable` vs. `agent_preloaded` vs. `gateway_only`) is audited at refactor time.
8. `disable-model-invocation: true` / `allow_implicit_invocation: false` is not treated as sole authorization — used only as defense-in-depth alongside `disallowedTools` rules.
9. `skills:` preload is not treated as an allowlist. Explicit `disallowedTools: Skill` or `Skill(name)` rules added where agent-level restriction is required.
10. `skillOverrides: { "name": "off" }` is not used as sole authorization mechanism.
11. **Tests or evals include at least one negative case: the orchestrator must delegate instead of invoking a visible skill directly.**
12. Codex deployments implement a harness-level gateway for strong skill authorization boundaries.

## Evidence

### Academic Papers

- **SoK: Trust-Authorization Mismatch in LLM Agent Interactions** (arXiv:2512.06914, 2024) — B-I-P framework decomposes agent execution into Belief Formation (discovery), Intent Generation, and Permission Grant (authorization) as structurally distinct stages. Quote: "diverse threats share a common root cause: the desynchronization between dynamic trust states and static authorization boundaries." Directly names the failure this principle addresses.

- **SEAgent: Taming Various Privilege Escalation in LLM-Based Agent Systems** (arXiv:2601.11893, 2026, HKUST/ETH Zurich) — Implements per-agent attribute-based mandatory access control (ABAC/MAC). Achieves 0% attack success rate on InjecAgent and AgentDojo benchmarks. Quote: "agent actions exceeding the least privilege required for a user's intended task" = privilege escalation definition. Demonstrates that enforcement must be structural, not prompt-based.

- **Agent Skills for Large Language Models: Architecture, Acquisition, Security, and the Path Forward** (arXiv:2602.12430, 2026) — Proposes a four-tier gate-based permission model separating skill provenance from deployment capabilities. Documents that 26.1% of community-contributed skills contain vulnerabilities — the attack vector is the shared discovery namespace. The four-tier model maps directly to the visibility/authority axis.

- **AgentSpec: Customizable Runtime Enforcement for Safe and Reliable LLM Agents** (arXiv:2503.18666, 2025) — Achieves >90% prevention of unsafe code executions via runtime hooks: "triggers, predicates, enforcement mechanisms." Used in conjunction with `multi-layered-safety-via-code.md` precedent. (Note: specific hook stage names not independently verified from abstract alone.)

- **Design Patterns for Securing LLM Agents against Prompt Injections** (arXiv:2506.08837, 2025) — "Constraining the actions of agents to explicitly prevent them from solving arbitrary tasks" significantly reduces prompt injection attack surface. Supports the principle that structural capability restriction is the primary defense.

- **LlamaFirewall** (arXiv:2505.03574, Meta AI, 2025) — "Three powerful guardrails" deployed in production at Meta (PromptGuard 2, Agent Alignment Checks, CodeShield). Demonstrates production-scale enforcement via code-level mechanisms, not prompting.

### Official Best Practices

- **Claude Code Official Docs — Sub-agents** (code.claude.com/docs/en/sub-agents, 2026) — Direct quotes: "This field controls which skills are preloaded, not which skills the subagent can access." (on `skills:` preload). "`disallowedTools: Skill` prevents a subagent from invoking skills entirely." Confirms `Agent(type)` spawn restriction via `tools:` field.

- **Claude Code Official Docs — Skills** (code.claude.com/docs/en/skills, 2026) — Direct quotes: "every tool remains callable" (on `allowed-tools` — it is a pre-approval, not a deny-all). "`disable-model-invocation: true`: Only you can invoke the skill." `Skill(name)` and `Skill(name *)` permission rule syntax confirmed.

- **Codex Official Docs — Skills** (developers.openai.com/codex/skills, 2026) — Confirms: `allow_implicit_invocation: false` blocks implicit model invocation; "explicit `$skill` invocation still works." Root `.agents/skills/` files are model-visible. No per-agent skill allowlist documented.

- **Anthropic "Equipping Agents for the Real World with Agent Skills"** (anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills) — Defines skills as "organized folders of instructions, scripts, and resources that agents can discover and load dynamically." The framing is entirely capability-additive: skills extend what an agent can do, not what an agent is prevented from doing.

### Named in Literature?

No unified term. Closest named concepts:

- **Trust-Authorization Mismatch** — arXiv:2512.06914, which names the failure mode at scale
- **Privilege escalation in agent systems** — arXiv:2601.11893 (SEAgent), with MAC as the countermeasure
- **Excessive agency** — AWS Security Best Practices GENSEC05-BP01 (Principle of Least Privilege for agents)
- **Confused deputy problem** — classical capability security literature; applies when a skill invokes on behalf of a caller with excess authority

## When to Apply

| Signal | Decision | What breaks if ignored |
|---|---|---|
| A root/project skill is visible to an orchestrator | Audit orchestrator's `Skill` tool access; add `disallowedTools: Skill` or `Skill(name)` deny rules if not needed | Orchestrator stops delegating; delegation bypass failure mode |
| A specialist-only procedure is placed under public `skills/` | Move to agent-local or behind a `gateway_only` runtime command | Other agents discover and invoke it; specialist contract bypassed |
| A coordinator is observed executing rather than delegating | Audit `Skill`/`Bash`/file tool grants on coordinator | Delegation tree collapses; failure attribution lost |
| A skill has side effects or can invoke file/shell/runtime tools | Hide behind typed runtime command + caller identity check at harness layer | Side effects fire from any caller; audit trail lost |
| A system has both Claude Code and Codex variants | Enforce per-platform; do not assume `Skill(name)` semantics apply to Codex | Boundary present in Claude, absent in Codex |
| `skills:` is being used as an allowlist | Replace with explicit `disallowedTools: Skill` and `Skill(name)` deny rules | Agents can still discover and invoke non-preloaded skills |

Do not apply when: agent has no `Skill` tool access; single-agent runtime with no orchestrator; advisory-only system with no execution path; all agents have identical roles and grants.

## Known Limits

- **Platform semantics drift**: Claude Code and Codex permission model docs must be re-verified periodically. This analysis is pinned to 2026-05-26. Mechanisms documented here (`Skill(name)` syntax, `allow_implicit_invocation` behavior) may change in future releases.

- **Claude/Codex parity gap**: Claude Code has `Skill(name)` in `disallowedTools` for per-skill agent-level restriction. Codex has no documented equivalent per-agent skill allowlist. Do not claim parity where docs do not confirm it. Codex deployments require custom harness-level enforcement to achieve equivalent guarantees.

- **Implicit vs. explicit invocation boundary**: `allow_implicit_invocation: false` (Codex) and `disable-model-invocation: true` (Claude) block model-driven invocation only. Explicit invocation (`$skill` in Codex, `/skill-name` in Claude by user) remains available. These mechanisms lack allowlist-based authorization guarantees. Do not use them as sole access restriction for sensitive skills.

- **User-initiated invocation is out of scope**: `disable-model-invocation: true` does not prevent a user from typing `/skill-name`. This is correct behavior for human-in-the-loop systems. The principle governs agent-to-agent skill authorization — user invocation is a separate concern that should be addressed at UX/role boundary design, not in this principle.

- **Filesystem-level access bypass**: if an agent retains broad shell or file-read tool access, it can read skill files directly from the filesystem even when the `Skill` tool access is removed. Skill authorization requires both tool-layer restriction (`disallowedTools: Skill`) AND filesystem-level sandboxing. Gateway enforcement alone is insufficient if file access is broad.

- **Aspirational patterns without platform support**: "Caller-aware runtime gateways" (enforcement of caller identity at the skill execution boundary) and "typed runtime contracts" (structured pre/post-conditions on skill invocation) are currently aspirational. No Claude Code or Codex mechanism is documented that verifies caller identity at skill execution time. These require custom harness implementation and are not achievable by configuration alone.

- **Transitive invocation chains**: Skill A may invoke Skill B, which invokes Skill C. Authorization is typically checked at the first boundary (agent → Skill A) only. Transitive privilege escalation via skill chains is not addressed by any mechanism documented in this principle. Harness designers must audit skill-to-skill invocation graphs for transitive escalation paths.

- **Dynamic skill registration**: skills registered or created mid-session (e.g., via file-write tools) are not covered by static dispatch-time authorization. Runtime enforcement and filesystem write restrictions are required to close this gap. Static configuration of `disallowedTools` is insufficient for dynamically registered skills.

## Properties to Restore

위반이 감지됐을 때, 수정된 결과가 만족해야 할 속성들이다.

| Violation detected | Properties the fix must satisfy |
|---|---|
| A specialist-only skill is publicly discoverable and accessible to the orchestrator | The corrected design must either remove the orchestrator's `Skill` tool access for that skill (via `disallowedTools` or `Skill(name)` deny rule), or replace the visible skill with a thin entry adapter that delegates to a `gateway_only` runtime command; prompt warnings alone do not satisfy this |
| Orchestrator executes a visible skill directly instead of delegating | The corrected design must remove the orchestrator's `Skill` tool grant or replace the public skill with a typed runtime command behind a harness-level gateway; the orchestrator must be demonstrably unable to invoke the skill directly even when it can see it |
| A Codex deployment relies on `allow_implicit_invocation: false` as a sole security boundary | The corrected design must add a harness-level gateway that performs caller-identity + capability-map check before the skill body executes; the implicit flag is treated as defense-in-depth, not as an authorization boundary |
| `skills:` preload is documented or treated as an allowlist | The corrected design must add explicit `disallowedTools: Skill` or `Skill(name)` deny rules AND update team documentation to clarify that `skills:` is content injection only |
| Skill authorization enforced only via prompt instructions | The corrected enforcement mechanism must operate as deterministic configuration (tool permission rules or harness gateway) that activates regardless of what the LLM produces; prompt-based warnings are removed from authorization responsibility |

## Promotion History

Candidate — created 2026-05-26 from user-supplied architecture prompt.
Confirmed — 5-dim score 22/25 (코어성 5, 리스크감소 5, 확장성 4, 제어성 4, 기록성 4). Evidence gate: arXiv:2512.06914 (B-I-P framework, central claim); arXiv:2601.11893 (MAC enforcement, 0% ASR); Claude Code official docs (`Skill(name)` syntax, `skills:` NOT allowlist); Codex official docs (`allow_implicit_invocation` behavior). Application of `multi-layered-safety-via-code.md` to the skill discovery surface. Confirmed 2026-05-26.
