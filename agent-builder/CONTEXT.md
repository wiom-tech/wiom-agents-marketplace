# Agent Builder — Shared Context
## Version 2.5

Implementation context per AGENT_STACK_FORMAT.md §5.3. Format definitions live in AGENT_STACK_FORMAT.md — not restated here.

---

## Domain

- **Product:** Agent specifications for the Wiom build pipeline
- **Users:** Founder, product leads, engineering leads defining agent-driven workflows
- **Output:** Complete agent folders conforming to AGENT_STACK_FORMAT.md v2.5

---

## Current Versions

| Document | Version | Role |
|---|---|---|
| AGENT_STACK_FORMAT.md | v2.5 | SOURCE_OF_TRUTH |
| AGENT_SPEC.md | v2.5 | SOURCE_OF_TRUTH |
| SKILLS_INDEX.md | v2.5 | SOURCE_OF_TRUTH |
| CONTEXT.md | v2.5 | SOURCE_OF_TRUTH |
| elicitation/SKILL.md | v2.5 | SOURCE_OF_TRUTH |
| skill-orchestrator/SKILL.md | v1.5 | SOURCE_OF_TRUTH |
| assembly/SKILL.md | v2.5 | SOURCE_OF_TRUTH |
| input-diff/SKILL.md | v1.0 | SOURCE_OF_TRUTH (template — copied into target agents) |
| output-comparison/SKILL.md | v1.0 | SOURCE_OF_TRUTH (template — copied into target agents) |

---

## Canonical Sources

| Data Type | Source File | Section |
|---|---|---|
| Format definitions (all layers) | AGENT_STACK_FORMAT.md | §1–§12 |
| Layer structure (3+3 model) | AGENT_STACK_FORMAT.md | §1 |
| Input format rules | AGENT_STACK_FORMAT.md | §3 |
| Instruction format rules | AGENT_STACK_FORMAT.md | §4 |
| Skill format rules | AGENT_STACK_FORMAT.md | §5 |
| Check derivation rules | AGENT_STACK_FORMAT.md | §6 |
| JUDGMENT triggers + escalation card | AGENT_STACK_FORMAT.md | §7 |
| Output format + versioning | AGENT_STACK_FORMAT.md | §8 |
| Agent package file structure | AGENT_STACK_FORMAT.md | §10 |
| Run-time protocol | AGENT_STACK_FORMAT.md | §12 |
| Amendment mode protocol | AGENT_STACK_FORMAT.md | §12.5 |
| Agent-specific objective + measures | AGENT_SPEC.md | INSTRUCTIONS |
| Skill routing + execution order | SKILLS_INDEX.md | Execution Order |

---

## Key Decisions Already Locked

| Decision | Source | Impact |
|---|---|---|
| Agent Builder is build-time only — does not run at run time | AGENT_SPEC.md — Note | Cannot be bootstrapped by its own process |
| Agent never creates skills without human description — skeletons only | AGENT_SPEC.md constraints | Prevents content fabrication; K3 fires if skill has zero content |
| JUDGMENT is synchronous — no placeholders | AGENT_STACK_FORMAT.md §7.6 | Any output with PENDING/TBD where JUDGMENT was required = conformance FAIL |
| "You decide" from human is not resolution | AGENT_STACK_FORMAT.md §7.5 | Agent must surface structured options; human must select explicitly |

---

## Completeness Checklists

Quick-reference for elicitation. Canonical definitions per AGENT_STACK_FORMAT.md §3-§5.

### Layer 1 — Input: ask until...
- [ ] Input structure defines roles, types, version gates
- [ ] Optional vs required distinguished

### Layer 2 — Instructions: ask until...
- [ ] Objective is one sentence and testable
- [ ] Measure of success is concrete and binary (per §4.1)
- [ ] Output format is explicit (per §4.2)
- [ ] Skill bindings stated (MUST USE / MUST REFER / partial)
- [ ] Prohibitions defined
- [ ] Agent-specific kill conditions defined
- [ ] Amendment mode decision made (YES/NO per §12.5)
- [ ] IF amendment YES: changeable inputs identified, founder decisions criticality decided

### Layer 3 — Skills: ask until...
- [ ] Skills uploaded or described
- [ ] Each validated (9 sections per §5.4)
- [ ] Order, conflicts, dependencies defined
- [ ] All skills referenced in constraints exist

---

## Elicitation Principles

1. One layer at a time. Complete before advancing.
2. Ask, don't suggest — except for output format (structural, permitted per §4.2).
3. Follow up on vague answers. "The OS" → "Which OS? What version?"
4. Qualify each layer — pressure test answers before confirming.
5. Surface contradictions immediately.
6. Confirm before moving on: "Layer X complete. Here's what I have. Correct?"
7. For skills: ask "do you have skills to upload?" before asking for descriptions.

---

*Agent Builder — Shared Context v2.5*
