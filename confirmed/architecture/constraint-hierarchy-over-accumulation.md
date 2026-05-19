# Constraint Hierarchy over Accumulation

Status: confirmed
Last reviewed: 2026-05-18

## Principle

Accumulating prohibitions in an agent's instruction set is not equivalent to adding safety. Beyond a threshold, constraints compete with each other, produce noise, and degrade the reliability of core guidance — including the prohibitions that actually matter.

The correct model: **few hard invariants at the constitutional level, distributed enforcement everywhere else.**

Every proposed ban must answer one of four questions before it reaches the top level:

1. **Constitutional?** — Must always be blocked regardless of context; irreversible if violated; cannot be deferred → add to top-level hard invariants (2–4 maximum)
2. **Judgment?** — Context-dependent; varies by situation → express as positive posture, not prohibition
3. **Structural?** — Checkable by tool, test, or code → enforce via validator, hook, or constraint check
4. **Pattern?** — A failure mode observed in practice → accumulate in knowledge, not rules

Top-level instructions should frame what the system **is**, not what it is **not**. Reserve "never" and "don't" for the 2–4 genuinely irreversible hard invariants.

**The operating principle: raise the bar for bans, not the count.**

## Evidence

### Official Documentation

- **OpenAI "Reasoning Best Practices"** (platform.openai.com/docs/guides/reasoning-best-practices) — "Keep prompts simple and direct." Explicitly warns that excessive instructions and complex prompt engineering can harm performance on reasoning models. Constraint accumulation is identified as a reliability risk.
- **Anthropic "Reduce Prompt Leak"** (docs.anthropic.com/en/docs/test-and-evaluate/strengthen-guardrails/reduce-prompt-leak) — Defense rules added to counter prompt extraction increase prompt complexity and can degrade other behaviors. Adding more rules ≠ more safety.
- **OpenAI "Agent Builder Safety"** (platform.openai.com/docs/guides/agent-builder-safety) — Recommends structured outputs, separate judgment nodes, and distinct policy documents over long prohibition lists inside a single prompt. Structural enforcement outperforms accumulation.

### Academic Papers

- **"What Prompts Don't Say"** (arXiv:2505.13360) — Competing constraints produce less stable instruction-following. More requirements added to a prompt do not reliably improve performance; constraint competition causes selective non-compliance.
- **DMI: Policy-Mechanism Separation** (arXiv:2510.04607) — High-level policy and planning at the top layer; low-level enforcement mechanism at the bottom layer. Mixing policy statements with enforcement rules in the same layer degrades both.

### UX and Cognitive Load Research

- **Google Conversational Design** (developers.google.com/assistant/downloads/conversational-repair.pdf) — Too many constraints or choices increases confusion for both models and users. Applies symmetrically to instruction design.
- **Baymard UX Research** (baymard.com/blog/drop-down-usability) — Option overload degrades decision quality. The same pattern applies when instruction sets present too many competing rules.

## When to Apply

| Signal | Decision | What breaks if ignored |
|--------|----------|------------------------|
| Top-level instruction set has accumulated more than ~5 prohibition-type rules | Audit: apply the four-question test to each prohibition; push non-constitutional constraints to judgment, code, or knowledge | Constraint competition degrades compliance with the prohibitions that actually matter — the genuinely irreversible ones get drowned in noise |
| Agent inconsistency is traced to competing constraints | Identify which constraints conflict; demote the lower-priority one to judgment or structural enforcement | Without resolution, the agent selectively ignores constraints in unpredictable ways — not consistently, not debuggably |
| A new "don't" is proposed for the top level | Apply the four-question test first; only promote to constitutional if it is always-true, irreversible, and cannot be deferred | Reflexive promotion of new observations as top-level bans is the primary mechanism of constraint accumulation |
| A constraint is context-dependent (valid in some situations, not others) | Express as positive posture or judgment rule, not a prohibition | Context-dependent prohibitions produce false blocks in the valid cases and are silently ignored in the invalid cases |
| A constraint is checkable by tool or test | Move to validator, code hook, or automated check | Structural constraints in natural language prompts are less reliable than structural enforcement and consume instruction budget |

## Known Limits / Failure Modes

- **Genuine constitutional invariants still need to be prohibitions.** This principle does not argue against all constraints — it argues against accumulation below the constitutional threshold. Irreversible, always-true violations require hard bans. The principle is about where the bar is set, not that the bar disappears.
- **Distributed enforcement layers must actually exist.** Pushing constraints to judgment, code, or knowledge without building those layers is deferred accumulation, not elimination. The four-layer target (constitutional / judgment / structural / knowledge) requires investment in the non-constitutional layers.
- **"What counts as constitutional" requires judgment.** There is no mechanical test for constitutional status. Teams will disagree, and the threshold will drift toward accumulation without periodic audits. Schedule explicit reviews of the top-level constraint set.
- **Positive posture framing can obscure necessary limits.** Framing everything as "what the system is" risks undercommunicating genuine hard limits. Each constitutional prohibition should be explicitly flagged as such, not buried in posture language.
