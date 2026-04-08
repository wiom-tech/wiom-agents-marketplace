# Output Comparison
## SKILL.md — Version 1.0

Conforms to AGENT_STACK_FORMAT.md §5.4 (9-section skill format). Amendment mode skill per §12.5.5.

---

## Purpose

Diff the BASELINE output against the newly re-derived output. Classify every difference as TRACED, FOUNDER_OVERRIDE, or UNTRACED. Surface FOUNDER_OVERRIDE and UNTRACED items to human for resolution. This is the regression gate.

---

## When

Amendment mode only (§12.5). After all domain skills have re-derived the complete output from new inputs. Before final validation.

---

## Required Inputs

| Input | Why |
|---|---|
| BASELINE output (locked) | What existed before amendment |
| Newly re-derived output (from domain skills) | What the agent produced from new inputs |
| CHANGE SUMMARY (from input-diff skill) | What changed in the inputs — used to determine if output changes are traced |
| IMPACT TABLE (from input-diff skill) | Which sections were expected to change — used to flag unexpected changes |
| FOUNDER DECISIONS (from input-diff skill, optional) | Decisions human made during original run — used to detect overrides |

---

## Framework

```
Step 1: Section-by-section diff
  → For each section in the output template (§0-§N):
    - Compare BASELINE section vs new output section
    - Status: UNCHANGED or CHANGED
    - If CHANGED: capture specific differences (what was added/removed/modified)

Step 2: Classify each CHANGED section
  → For each CHANGED section, apply classification in order:

    Test 1 — TRACED?
      Does the change map to an entry in the IMPACT TABLE?
      Is the specific modification explainable by a specific input change?
      → IF YES to both: classify as TRACED. Record which input change caused it.

    Test 2 — FOUNDER_OVERRIDE?
      Does the change overwrite a STRUCTURAL decision from FOUNDER DECISIONS?
      → IF YES: classify as FOUNDER_OVERRIDE. Record which decision is affected.
      → IF FOUNDER DECISIONS not available: skip this test.

    Test 3 — UNTRACED
      → IF neither TRACED nor FOUNDER_OVERRIDE: classify as UNTRACED.
      → This is the regression signal.

Step 3: Build classification table
  → Produce CHANGE CLASSIFICATION TABLE:
    | Section | Status | Classification | Input Change (if TRACED) | Detail |

  → Summary counts:
    - UNCHANGED: [count]
    - TRACED: [count]
    - FOUNDER_OVERRIDE: [count]
    - UNTRACED: [count]

Step 4: Present to human
  → Show full classification table
  → For TRACED items: "These changes are explained by input changes. Accept all?"
    - Human confirms or flags individual items for review
  → For FOUNDER_OVERRIDE items: surface each one:
    "Section [§X] change overwrites your original decision: [decision].
     (A) Keep original decision — revert this section
     (B) Accept new derivation — override your previous decision
     (C) Modify — provide new instruction"
    - Human picks. Log with AUTHORITY.
  → For UNTRACED items: surface each one:
    "Section [§X] changed but I cannot trace this to any input change.
     This may be regression.
     (A) Accept with justification — human provides reason
     (B) Revert to BASELINE for this section
     (C) Investigate — agent explains its derivation for this section"
    - Human picks. Log with AUTHORITY.

Step 5: Apply resolutions
  → For each FOUNDER_OVERRIDE resolved:
    - If KEEP: revert section to BASELINE version
    - If ACCEPT: keep new derivation. Log: "Founder decision overridden with approval"
    - If MODIFY: apply human's new instruction. Re-derive section. Re-classify.
  → For each UNTRACED resolved:
    - If ACCEPT: keep change. Log human's justification as AUTHORITY.
    - If REVERT: restore BASELINE section
    - If INVESTIGATE: agent shows derivation chain for that section.
      Human then picks ACCEPT or REVERT.

Step 6: Produce final output + AMENDMENT_LOG
  → Final output = resolved output (all classifications handled)
  → AMENDMENT_LOG.md per §12.5.3:
    - Baseline version, changed inputs, change summary
    - Full classification table with resolutions
    - Founder decisions preserved vs overridden
    - Version bump
```

---

## Rules

R1: IF a section is UNCHANGED → no action. Do not surface to human.
R2: IF a TRACED change does not map to a specific input change → reclassify as UNTRACED. "Affected sections" in impact table is not sufficient — the specific modification must be explainable.
R3: IF all changes are TRACED and human confirms → proceed directly to validation. No unnecessary friction.
R4: IF FOUNDER DECISIONS not available → skip FOUNDER_OVERRIDE classification. Log: "Original log not provided — founder decision tracking unavailable." All non-TRACED changes become UNTRACED.
R5: IF human picks INVESTIGATE for an UNTRACED item → agent must show the derivation chain: "I derived this from [input section] → [skill step] → [output section]. The change happened because [explanation]." Human then re-classifies.
R6: IF human provides justification for an UNTRACED item → log justification as AUTHORITY. This is now a STRUCTURAL decision.
R7: IF MODIFY resolution triggers a change in another section → re-run classification for that section. Do not assume cascade is safe.
R8: IF zero UNTRACED and zero FOUNDER_OVERRIDE → log: "Clean amendment — all changes traced to input changes."
R9: IF UNTRACED count > 50% of changed sections → PAUSE. "More than half of changed sections are unexplained. Consider whether this should be a fresh build instead of amendment."

---

## Output

1. **Resolved output** — complete output with all classifications handled, ready for validation
2. **CHANGE CLASSIFICATION TABLE** — full table with resolutions
3. **AMENDMENT_LOG.md** — per §12.5.3 format

---

## Fail Conditions

| Condition | Severity | Action |
|---|---|---|
| BASELINE not provided | S1 | ABORT — cannot compare without baseline |
| Re-derived output not provided (domain skills didn't complete) | S1 | ABORT |
| UNTRACED item not resolved by human | S1 | STOP — regression not cleared |
| FOUNDER_OVERRIDE item not resolved by human | S1 | STOP — decision conflict not cleared |
| UNTRACED count > 50% of changes (after human review) | S2 | FLAG — recommend fresh build |
| MODIFY resolution triggers cascade not re-classified | S1 | STOP — re-run classification |

---

## Does Not Do

- Does not re-derive output sections — domain skills already did that
- Does not modify inputs
- Does not decide whether to keep or override founder decisions — human decides
- Does not accept UNTRACED changes silently — every one must be resolved
- Does not run validation checks — validation skill does that after this skill
- Does not version bump — assembly/output handles that
- Does not skip classification for "small" changes — every diff is classified regardless of size

---

## Traceability

| This Skill Enforces | Authority |
|---|---|
| Change classification (TRACED/FOUNDER_OVERRIDE/UNTRACED) | §12.5.1 Step 6 |
| Regression protection via UNTRACED detection | §12.5.2 |
| Founder decision preservation | §12.5.1 Step 6, §12.5.2 |
| Full classification — no selective patching | §12.5.2 design principle |
| AMENDMENT_LOG.md production | §12.5.3 |
| Amendment constraints — update not expand | §12.5.4 |
| Human resolves all non-TRACED items | §12.5.1 Step 6 |

---

*Output Comparison — SKILL.md v1.0*
