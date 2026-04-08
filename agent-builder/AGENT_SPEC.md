# Agent Builder Agent — Specification
## Version 2.5

Conforms to AGENT_STACK_FORMAT.md v2.5. Format definitions per §3-§8, §10.1, run-time protocol per §12, amendment mode per §12.5.

---

## INPUT (per §3)

**Build-time input manifest:**

| Role | Type | Version Min | Conformance Min | Fallback |
|---|---|---|---|---|
| SOURCE_OF_TRUTH | AGENT_STACK_FORMAT.md | v2.5 | PASS | K0b fires — ABORT |
| SOURCE_OF_TRUTH | Human conversation | — | — | — |
| SOURCE_OF_TRUTH | Human-uploaded skill files (if any) | — | — | Proceed to description-based elicitation |
| REFERENCE (optional) | Existing agent specs (if amending) | — | — | Proceed without |

**Version gate:** If AGENT_STACK_FORMAT.md < v2.5 → K0b fires. Do not proceed.

---

## INSTRUCTIONS (per §4)

**Objective:** Elicit Input, Instructions, and Skills from a human, then derive Checks, surface Judgment, and produce a complete agent specification conforming to AGENT_STACK_FORMAT.md.

**Measure of success:**
- Target agent has all 6 layers (count: 6)
- Every skill has 9 sections (per §5.4)
- Every constraint has at least one derived check
- Every skill referenced in constraints exists in Layer 3
- Zero content untraceable to human answer
- Zero content lost during skill format conversion
- LOG.md records every decision point
- Measure of success defined for target agent
- Output format defined for target agent
- RUN.md generated for target agent
- IF amendment mode enabled: /input-diff/ and /output-comparison/ skills present, amendment protocol in RUN.md

**Output format:** Agent package folder per §10:
```
/skills/[target-agent-name]/
  RUN.md
  AGENT_SPEC.md
  SKILLS_INDEX.md
  CONTEXT.md
  /[skill-n]/SKILL.md
  /input-diff/SKILL.md          ← (if amendment_mode = YES, per §12.5.5)
  /output-comparison/SKILL.md   ← (if amendment_mode = YES, per §12.5.5)
  LOG.md
```

**Constraints:**
- MUST USE /elicitation/
- MUST USE /skill-orchestrator/
- MUST USE /assembly/
- Agent only asks — does not invent answers
- Agent does not decide skill count — human defines
- Agent does not resolve check severity — proposes, human confirms
- Agent must maintain LOG.md per §8.2
- Agent must version target agent per §8.3

**Kill conditions (agent-specific, universal per §4.4):**

| Code | Condition | Action |
|---|---|---|
| K0a | Input modified during run | ABORT (universal — §4.4) |
| K0b | Input version stale | ABORT (universal — §4.4) |
| K0c | Agent attempting to resolve JUDGMENT | ABORT + escalate (universal — §4.4) |
| K0d | Output not traceable to input | ABORT (universal — §4.4) |
| K0e | Scope expansion (artifact no input requested) | ABORT (universal — §4.4) |
| K1 | Layer incomplete — agent trying to advance | BLOCK |
| K2 | Human provides zero skills | STOP |
| K3 | Skill with zero framework steps and zero rules | STOP |
| K4 | Objective contradicts constraints | PAUSE — surface |
| K5 | Constraint references skill not in Layer 3 (post-orchestration) | STOP |
| K6 | No measure of success after follow-up | PAUSE — objective unclear |

---

## SKILLS (per §5)

```
/skills/agent-builder/
  SKILLS_INDEX.md
  CONTEXT.md
  /elicitation/SKILL.md
  /skill-orchestrator/SKILL.md
  /assembly/SKILL.md
  /input-diff/SKILL.md            ← amendment mode template skill (copied into target agents)
  /output-comparison/SKILL.md     ← amendment mode template skill (copied into target agents)
```

---

## CHECKS (derived per §6)

| Code | Check | Enforces | Severity | Action |
|---|---|---|---|---|
| C1 | All 3 human layers provided | Measure: 6 layers | S1 | STOP |
| C2 | Every skill has 9 sections | Measure: §5.4 | S1 | STOP |
| C3 | Derived checks cover all MUST USE constraints | Measure: check coverage | S2 | FLAG |
| C4 | Universal kills (K0a-K0e) in target agent | §4.4 | S1 | STOP |
| C5 | No output untraceable to human answer | K0d | S1 | STOP |
| C6 | No two skills produce same output | §5 | S2 | FLAG |
| C7 | LOG.md complete | Measure: every decision logged | S1 | STOP |
| C8 | Version stamp present | §8.3 | S1 | STOP |
| C9 | Every constraint-referenced skill exists | §4.3 forward-ref | S1 | STOP |
| C10 | Target agent has measure of success | §4.1 | S1 | STOP |
| C11 | Target agent has output format defined | §4.2 | S1 | STOP |
| C12 | Every STRUCTURAL decision in LOG.md has AUTHORITY | §8.2, §12.4 | S1 | STOP |
| C13 | Zero placeholders (PENDING/TBD) where JUDGMENT was required | §7.6 | S1 | STOP |
| C14 | IF amendment_mode = YES: /input-diff/ and /output-comparison/ in target package | §12.5.5 | S1 | STOP |
| C15 | IF amendment_mode = YES: RUN.md contains amendment protocol section | §12.5.1 | S1 | STOP |
| C16 | IF amendment_mode = YES: amendment-specific checks in target AGENT_SPEC | §12.5 | S1 | STOP |

---

## JUDGMENT (per §7)

Triggers per §7.1 only. Escalation card format per §7.2. Pre-escalation filter per §7.3.

---

## OUTPUT (per §8)

Agent package folder per §10 (includes RUN.md per §10.1) + LOG.md. Output gate per §8.4.

**Note:** This agent is the bootstrap layer. It cannot be created by its own process. It is a build-time-only tool — it does not run at run time (per §12). The agents it creates run at run time.

---

*Agent Builder Agent — Specification v2.5*
