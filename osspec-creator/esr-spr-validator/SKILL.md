# ESR/SPR Validator
## SKILL.md — Version 1.0

Conforms to AGENT_STACK_FORMAT.md §5.4 (9-section skill format).

---

## Purpose

Verify all events and parameters in the generated YAML against the ESR and SPR to ensure completeness and accuracy; surface any discrepancies before the output is accepted.

---

## When

Triggered after PRD YAML Generator produces the YAML output. Ends when all events and parameters are verified (PASS) or discrepancies are surfaced (STOP/WARN) and returned to human.

---

## Required Inputs

| Input | Why |
|---|---|
| Generated YAML (from PRD YAML Generator) | The artifact to validate |
| ESR — Event Schema Reference | Authoritative source for valid event names and schemas |
| SPR — Standard Parameters Reference | Authoritative source for valid parameters |
| OS file (SOURCE_OF_TRUTH) | Used to check ESR completeness — ensure no OS-referenced events are absent from YAML |

---

## Framework

```
Step 1: Extract all event names from generated YAML events[] section.
  → event name list

Step 2: Cross-check each event name against ESR.
  → validation result per event (FOUND / NOT_FOUND)

Step 3: For each matched event, compare YAML schema_fields against ESR
  schema definition for that event.
  → schema match result per event (MATCH / MISMATCH with diff)

Step 4: Scan ESR for events referenced in OS but absent from YAML.
  → completeness gap list (events in ESR relevant to OS not in generated YAML)

Step 5: Extract all parameters from generated YAML.
  Scan: operational_config.service_specific_params, balance_model,
        guard_specifications, state_machines, flows, and any field
        referencing a parameter ID (P### format).
  → parameter list

Step 6: Cross-check each parameter against SPR defined list.
  → validation result per parameter (FOUND / NOT_FOUND)

Step 7: Compile all findings into validation report.
  → validation report (PASS / STOP / WARN status per item)

Step 8: Surface findings to human.
  IF any S1 finding → STOP. Return report to human. Do not emit final YAML.
  IF only S2 findings → WARN. Surface report. Human decides whether to accept.
```

---

## Rules

```
R1: IF event name in YAML not found in ESR → STOP (K2)
R2: IF parameter in YAML not found in SPR → STOP (K3)
R3: IF event name matches ESR but schema_fields differ → WARN
R4: IF ESR contains event referenced in OS but absent from YAML
    → FLAG → return finding to PRD YAML Generator for addition,
    then re-run both skills
```

---

## Output

Validation report: findings per event and per parameter.
PASS / STOP / WARN status per item. Returned to human before output is accepted.

---

## Fail Conditions

| Condition | Severity | Action |
|---|---|---|
| Event in YAML not found in ESR | S1 | STOP |
| Parameter in YAML not found in SPR | S1 | STOP |
| Event name matches ESR but schema_fields differ | S2 | WARN |
| ESR event referenced in OS absent from YAML | S2 | FLAG |

---

## Does Not Do

- Does not invent new events or parameters
- Does not modify the YAML directly
- Does not modify OS, ESR, or SPR files
- Does not validate YAML structure (PRD YAML Generator handles Phase C/A/B)
- Does not resolve ESR/SPR discrepancies unilaterally — surfaces to human

---

## Traceability

| This Skill Enforces | Authority |
|---|---|
| Every event name in YAML matches ESR | AGENT_SPEC — M2 |
| Every parameter in YAML comes from SPR | AGENT_SPEC — M3 |
| ESR event referenced in OS missing from YAML → STOP | AGENT_SPEC — K2 |
| SPR parameter not in SPR → STOP | AGENT_SPEC — K3 |
| Never invents events or parameters | AGENT_SPEC — Constraints |

---

*ESR/SPR Validator — SKILL.md v1.0*
