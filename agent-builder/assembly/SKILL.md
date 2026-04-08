# Assembly
## SKILL.md — Version 2.5

Conforms to AGENT_STACK_FORMAT.md §5.4 (9-section skill format).

---

## Purpose

Take validated human layers and produce the complete agent folder. Derive Checks seeded from measures of success. Generate RUN.md as execution entry point. Produce LOG.md as audit trail.

---

## When

After Skill Orchestrator passes validated skills, routing, and context.

---

## Required Inputs

| Input | Why |
|---|---|
| AGENT_STACK_FORMAT.md §6-§8, §10.1, §12 | Check derivation, RUN.md, output format, log, run-time protocol |
| Elicitation output (Layers 1-2) | Input structure + Instructions (includes measures + output format) |
| Sample output file (if provided in Layer 2) | REFERENCE — validate structure match, not copy content |
| Skill Orchestrator output (Layer 3) | Validated skills + routing + context |
| Full conversation history | For LOG.md |

---

## Framework

```
Step 1: Derive checks (per §6)
  → Seed from measures of success — each measure becomes checks
  → Expand from MUST USE constraints — checks per skill rule
  → Expand from MUST USE §X — checks for named rules
  → Add universal checks
  → Present to human: "Derived [N] checks. Confirm severity."

Step 2: Generate AGENT_SPEC.md
  → All sections reference AGENT_STACK_FORMAT.md — never restate (per §2)
  → Include output format from Layer 2 (§4.2)
  → Include run-time protocol reference (§12)

Step 3: Generate SKILLS_INDEX.md per §5.2

Step 4: Generate CONTEXT.md per §5.3

Step 5: Copy validated SKILL.md files from Skill Orchestrator

Step 6: Generate RUN.md per §10.1
  → Derive mechanically from AGENT_SPEC:
    - Boot sequence: always spec → index → context → skills
    - Run-time protocol: always §12.1 steps
    - Default output format: from AGENT_SPEC §4.2
    - Rules: extract universal (§7.5, §7.6, §12.4) + agent-specific constraints + kills
  → IF amendment_mode = YES:
    - Add "## Amendment Mode" section to RUN.md per §12.5
    - Include amendment run-time protocol (§12.5.1 steps)
    - List changeable inputs from Elicitation
    - Note whether founder decisions tracking is critical or optional
    - Reference /input-diff/ and /output-comparison/ skills
  → No human input needed. Pure derivation.

Step 7: Generate LOG.md per §8.2

Step 8: Add amendment skills (if amendment_mode = YES)
  → Copy /input-diff/SKILL.md into target agent package
  → Copy /output-comparison/SKILL.md into target agent package
  → Customize REQUIRED INPUTS in both skills with agent-specific:
    - Input document names and types (from Elicitation changeable_inputs)
    - Output section structure (from template/output format)
  → Add both skills to SKILLS_INDEX.md:
    - Type: domain
    - Mandatory: conditional (amendment mode only)
    - Execution order: input-diff → [existing domain skills] → output-comparison → validation
  → Add amendment-specific checks to AGENT_SPEC checks table:
    - All UNTRACED items resolved
    - All FOUNDER_OVERRIDE items resolved
    - AMENDMENT_LOG.md produced
  → IF amendment_mode = NO: skip this step entirely

Step 9: Post-assembly cross-check
  → Every constraint → at least one skill Traceability
  → Every skill in SKILLS_INDEX → matching SKILL.md exists
  → Every check → traces to measure, constraint, or rule
  → Versions match across files
  → No content untraceable
  → No placeholders (PENDING/TBD) where JUDGMENT was required
  → Every STRUCTURAL decision has AUTHORITY in LOG.md
  → LOG.md complete
  → RUN.md exists and references correct files
  → Output format defined
  → IF amendment_mode = YES: /input-diff/ and /output-comparison/ exist,
    amendment protocol in RUN.md, amendment checks in AGENT_SPEC
  → If any fails → S1 STOP. Surface specific gap to human.

Step 10: Version + emit per §8.3, §8.4
```

---

## Rules

R1: IF measure of success is countable → derive count-match check
R2: IF measure of success is binary → derive presence check
R3: IF MUST USE constraint → one check per rule in bound skill
R4: IF MUST USE §X → checks for named rules only
R5: IF MUST REFER → no automatic checks
R6: IF severity unclear → present to human
R7: IF cross-check fails → S1 STOP. Surface specific gap to human. Do not silently skip.
R8: IF content untraceable → S1 STOP. Surface to human: "This content has no source. Remove, or provide AUTHORITY?" Do not silently remove.
R9: No restatement of AGENT_STACK_FORMAT.md (per §2)
R10: IF output format missing → S1 STOP
R11: IF any STRUCTURAL decision in LOG.md has no AUTHORITY citation → S1 STOP (per §8.2, §12.4)
R12: IF amendment_mode = YES → /input-diff/ and /output-comparison/ MUST be present in target agent package. Missing either → S1 STOP.
R13: IF amendment_mode = YES → RUN.md MUST contain amendment protocol section. Missing → S1 STOP.
R14: IF amendment_mode = YES and founder_decisions_critical = YES → ORIGINAL LOG must be listed as required (not optional) input in amendment protocol.

---

## Output

```
/skills/[target-agent-name]/
  RUN.md
  AGENT_SPEC.md
  SKILLS_INDEX.md
  CONTEXT.md
  /[skill-1]/SKILL.md
  /[skill-n]/SKILL.md
  /input-diff/SKILL.md          ← (if amendment_mode = YES)
  /output-comparison/SKILL.md   ← (if amendment_mode = YES)
  LOG.md
```

---

## Fail Conditions

| Condition | Severity | Action |
|---|---|---|
| Human layers incomplete | S1 | STOP |
| Cross-check fails | S1 | STOP — surface gap |
| Content untraceable | S1 | STOP — surface to human |
| LOG.md incomplete | S1 | STOP |
| Output format missing | S1 | STOP |
| STRUCTURAL decision without AUTHORITY in LOG.md | S1 | STOP |
| Placeholder (PENDING/TBD) where JUDGMENT required | S1 | STOP |
| RUN.md missing or references wrong files | S1 | STOP |
| Amendment mode enabled but /input-diff/ or /output-comparison/ missing | S1 | STOP |
| Amendment mode enabled but RUN.md lacks amendment protocol | S1 | STOP |
| Measure of success has no derived check | S2 | FLAG |
| Contradiction between files | S2 | FLAG — surface |

---

## Does Not Do

- Does not add skills
- Does not modify validated skills
- Does not invent constraints or kills
- Does not decide severity (proposes, human confirms)
- Does not restate AGENT_STACK_FORMAT.md
- Does not write placeholders (PENDING/TBD) where JUDGMENT is required — per §7.6
- Does not defer JUDGMENT to post-completion review — per §7.6
- Does not silently remove untraceable content — surfaces to human per R8

---

## Traceability

| This Skill Enforces | Authority |
|---|---|
| Measures of success seed checks | §4.1 |
| Output format mandatory | §4.2 |
| Check derivation rules | §6 |
| "You decide" is not permission | §7.5 |
| JUDGMENT is synchronous — no placeholders | §7.6 |
| Decision log with AUTHORITY (STRUCTURAL vs STYLISTIC) | §8.2 |
| RUN.md generation | §10.1 |
| Amendment mode skills and protocol | §12.5, §12.5.5 |
| Run-time protocol inherited by target agent | §12 |
| Decision Authority Rule inherited by target agent | §12.4 |
| No restatement | §2 |

---

*Assembly — SKILL.md v2.5*
