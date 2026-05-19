# Skill Execution Surface Types

Status: confirmed
Last reviewed: 2026-05-18

## Principle

Agent skills should be typed by their execution surface — the mechanism through which they run. The surface type determines the required file structure, what the runtime can assume, and what fallback paths are available.

Four surface types:

---

### `local_module`

A skill implemented as executable code that runs in-process without an LLM call.

**Use for:** Deterministic, code-driven operations — file parsing, data transformation, API calls with no planning required.

**Required files:**
- `SKILL.md` — frontmatter must declare `surface: local_module`
- `runner` (language-specific executable module)

**Not required:** `prompt.md` (no LLM involved), `manifest.json` (no routing metadata needed)

**Validator behavior:** When `surface: local_module` is declared, validators must skip prompt.md checks. Requiring prompt.md for local_module skills is a validator bug, not a skill gap.

---

### `repo_skill`

A skill backed by a prompt and reference materials, executed by an LLM using the skill's prompt as its instruction set.

**Use for:** Planning, analysis, and reasoning tasks where the skill's value is in the curated prompt + references, not in code.

**Required files:**
- `SKILL.md` — frontmatter must declare `surface: repo_skill`
- `prompt.md` — the LLM instruction set
- `manifest.json` — routing metadata
- `references/` — supporting materials the LLM reads
- `scripts/` (optional) — supporting automation

---

### `codex_subagent`

A skill dispatched to an external coding agent that operates in its own context with tool access.

**Use for:** Implementation tasks requiring file editing, test running, or multi-step code changes that exceed what a single LLM call can do reliably.

**Required files:**
- `SKILL.md` — frontmatter must declare `surface: codex_subagent`
- `prompt.md` — the agent's task instruction
- `manifest.json` — routing metadata
- `agent.md` — the agent definition the subagent loads at runtime

**Critical:** `agent.md` is loaded via `readFileSync` at runtime. If absent, the runtime throws. Only skills whose `manifest.json` declares `codex_subagent` surface are in this execution path — other agent definition directories without `agent.md` are not at runtime risk.

---

### `model_replay`

An offline or fallback execution path where a previous model output is replayed without a live LLM call.

**Use for:** Dry-run validation, offline testing, fallback when the primary surface is unavailable.

**Preference rule:** Prefer `repo_skill` and `codex_subagent` over `model_replay`. Use `model_replay` only for offline or fallback paths. Record when fallback occurs and why.

---

## Evidence

### local_module

- **Anthropic "Writing Tools for Agents"** (anthropic.com/engineering/writing-tools-for-agents) — Distinguishes deterministic tool execution (local code, no LLM) from LLM-driven tool planning. Prescribes matching execution mechanism to task type: deterministic operations need no LLM; planning operations need one.
- **Anthropic "Equipping Agents for the Real World with Agent Skills"** (anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills) — Anthropic's own implementation separates `scripts/` (deterministic executable code, no LLM call) from `SKILL.md`-based skills (LLM-driven instruction sets). This two-tier split directly instantiates the `local_module` vs `repo_skill` surface distinction in a production agent system.

### repo_skill

- **Anthropic "Building Effective Agents"** (anthropic.com/research/building-effective-agents) — Prescribes designing the Agent-Computer Interface (ACI) — tool definitions, prompt instructions, reference materials — as a first-class artifact before agent logic. Directly maps to the `repo_skill` surface requiring `prompt.md` + `references/`.
- **ReAct: Synergizing Reasoning and Acting in Language Models** (arXiv 2210.03629, ICLR 2023) — The foundational Thought→Action→Observation model: LLM-driven actions require a prompt that defines the action space. Supports the `repo_skill` `prompt.md` requirement.

### codex_subagent

- **LLM-Based Multi-Agent Systems for Software Engineering: Literature Review** (arXiv 2404.04834, 2024) — Identifies implementation tasks requiring file editing, test running, and multi-step code changes as a distinct execution class from single-LLM-call planning — supporting the external agent surface as a separate type.

### Surface-aware validation

- **AgentSpec: Customizable Runtime Enforcement for Safe and Reliable LLM Agents** (arXiv 2503.18666, 2025) — Demonstrates that safety and structural constraints must be enforced per execution path — validators that apply uniform rules across execution surfaces produce false positives and miss path-specific requirements.

---

## When to Apply

| Signal | Decision | What breaks if ignored |
|--------|----------|----------------------|
| A skill needs to be created and its execution mechanism is not yet defined | Choose the surface type first — before writing any files. Use the Surface Selection Guide below. | Files created without a declared surface type cannot be validated correctly; validators apply wrong rules and produce false positives or miss required files |
| A validator flags a skill for missing `prompt.md` but the skill does not use an LLM | The skill is `local_module` — the validator is surface-unaware. Fix the validator to read the `surface:` frontmatter before applying file checks | Surface-unaware validators block correct skills and create false confidence when they pass incorrect ones |
| A new execution backend is added to the system | Define it as a named surface type with explicit required files before any skills use it | Skills built on an unnamed surface type cannot be validated, audited, or routed consistently |
| A `codex_subagent` skill is deployed but `agent.md` is missing | The runtime will throw on `readFileSync` — add `agent.md` before deployment | Silent runtime failure; the subagent cannot load its definition and the skill is dead on first invocation |

## Known Limits / Failure Modes

- **Surface-unaware validators:** A validator that requires all skills to have `prompt.md` will falsely flag `local_module` skills. Validators must read the surface declaration in `SKILL.md` frontmatter before applying file checks.
- **`agent.md` false-positive severity:** A missing `agent.md` in an agent definition directory is only a runtime risk if that directory is reachable via a `codex_subagent` skill manifest. Check the execution path before rating this as critical.
- **`model_replay` overuse:** `model_replay` is prescribed for offline and fallback paths only. Using it as the primary surface to avoid LLM cost or latency is a misuse — model_replay outputs reflect a prior model state that may not match current prompts or context. Errors from stale outputs are harder to detect than live LLM errors. Audit any skill declared `model_replay` and verify it cannot be served by `repo_skill` or `local_module`.
- **Surface boundary drift:** Over time, a `repo_skill` may accumulate code execution in its `scripts/` directory, blurring the boundary with `codex_subagent`. A `repo_skill` that routinely executes files, patches code, or calls external services should be reclassified as `codex_subagent` and `agent.md` added before deployment.

## Surface Selection Guide

| Task type | Recommended surface |
|---|---|
| Deterministic code execution | `local_module` |
| LLM planning / analysis | `repo_skill` |
| Multi-step code implementation | `codex_subagent` |
| Offline / fallback path | `model_replay` |

## Provenance Requirements

For all surfaces: record requested surface, resolved surface, whether fallback was used, and fallback reason. This is the minimum audit trail for surface routing decisions.
