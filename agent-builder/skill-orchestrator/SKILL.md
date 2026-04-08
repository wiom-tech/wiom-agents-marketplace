# Skill Orchestrator
## SKILL.md — Version 1.5

Conforms to AGENT_STACK_FORMAT.md §5.4 (9-section skill format).

---

## Purpose

Validate human-provided skills against the 9-section standard, convert to standard format if needed, identify coverage gaps, and surface gaps as JUDGMENT. Zero content loss during conversion.

---

## When

After Elicitation completes all 3 human layers.

---

## Required Inputs

| Input | Why |
|---|---|
| AGENT_STACK_FORMAT.md §5.4-§5.6 | Skill format, types, dependencies |
| Elicitation output — Layer 2 | Objective + constraints (skill bindings) |
| Elicitation output — Layer 3 | Uploaded files or descriptions |

---

## Framework

```
Step 1: Receive skills
  → IF uploaded: read each file
  → IF described: create skeleton SKILL.md. Mark incomplete sections.

Step 2: Validate format (9 sections per §5.4)
  → Missing section → return to human with specific gap

Step 3: Validate content
  → Framework uses numbered steps?
  → Rules use IF-THEN? No prose, no "may"?
  → Fail conditions have severity?
  → "Does Not Do" present?
  → Traceability contains authority only (not input references)?
  → No two skills produce same output?

Step 4: Validate forward references
  → Every skill in Layer 2 constraints exists in Layer 3?
  → Missing → S1 STOP

Step 5: Validate skill dependencies
  → Skill references another skill → does it exist?
  → Missing → S1 STOP

Step 6: Gap analysis
  → Every MUST USE constraint has a skill enforcing it?
  → Gap → JUDGMENT escalation card per §7.2: "(A) Add skill, (B) Accept gap, (C) Remove constraint"

Step 7: Build routing
  → Classify: foundation or domain (per §5.5)
  → Check order respects dependencies
  → Propose order + conflict rule. Human confirms.

Step 8: Pass to Assembly
```

---

## Rules

R1: IF section missing → "Skill [X] missing [section]."
R2: IF prose instead of IF-THEN → propose conversion. Human confirms.
R3: IF "may" / "should consider" → propose rewrite. Human confirms.
R4: IF two skills produce same output → JUDGMENT per §7.1 #2.
R5: IF traceability links to non-existent constraint → S2 FLAG. Surface: "Skill [X] claims to enforce [Y] but that constraint doesn't exist in Instructions."
R6: IF skill reads file not in inputs → "Add to inputs?"
R7: IF constraint references non-existent skill → S1 STOP.
R8: IF human picks "Add skill" → collect description, skeleton, re-validate.
R9: IF human picks "Accept gap" → log. Assembly derives check anyway.
R10: IF human picks "Remove constraint" → update Instructions. Log. Version bump.
R11: IF dependency order violated → surface.
R12: ZERO CONTENT LOSS during format conversion. Every word, every rule, every condition from the original skill must appear in the converted output. Content may be reorganized across sections. Content must NEVER be summarized, paraphrased, compressed, trimmed, or dropped. "Formatting" is not permission to rewrite. If the human wrote 15 rules, the output has 15 rules — not 10 "equivalent" ones.
R13: IF content from original skill does not map to any of the 9 standard sections → place in a NOTES section appended after TRACEABILITY. Surface to human: "These items didn't map to standard sections. Keep as NOTES, reassign to a section, or remove?" Human decides.
R14: After conversion, present side-by-side: original content vs converted output. Human confirms nothing was lost. Do not proceed until human confirms completeness.

---

## Output

Validated SKILL.md files, SKILLS_INDEX.md content, CONTEXT.md content, gap results. Passed to Assembly.

---

## Fail Conditions

| Condition | Severity | Action |
|---|---|---|
| Skill has zero framework steps AND zero rules | S1 | STOP |
| Two skills produce same output (unresolved) | S1 | STOP |
| Constraint references non-existent skill | S1 | STOP |
| Skill dependency doesn't exist | S1 | STOP |
| Content lost during format conversion | S1 | STOP — restore lost content |
| Converted output not confirmed by human | S1 | STOP — present side-by-side first |
| Skill references input not in manifest | S2 | FLAG |
| Constraint has no skill (gap accepted) | S3 | WARN |
| Skill uses "may" after conversion | S2 | FLAG |
| Traceability contains input references instead of authority | S2 | FLAG — move to REQUIRED INPUTS |

---

## Does Not Do

- Does not create skills without human description — skeletons only
- Does not decide if gap requires new skill — JUDGMENT, human decides
- Does not derive checks (Assembly does that)
- Does not modify objective or constraints on its own initiative (human-requested removal per R10 is allowed)
- Does not decide execution order without human confirmation
- Does not summarize, compress, paraphrase, or "optimize" skill content during conversion — reorganizes only
- Does not drop rules that seem redundant — if the human wrote it, it stays
- Does not write placeholders (PENDING/TBD) and continue — per §7.6, JUDGMENT is synchronous

---

## Traceability

| This Skill Enforces | Authority |
|---|---|
| 9-section skill format | §5.4 |
| Foundation vs domain skill types | §5.5 |
| Skill dependency validation | §5.6 |
| Forward reference validation (constraints → skills exist) | §4.3 |
| JUDGMENT triggers #1, #2, #3, #4 | §7.1 |
| "You decide" is not permission | §7.5 |
| JUDGMENT is synchronous — no placeholders | §7.6 |

---

*Skill Orchestrator — SKILL.md v1.5*
