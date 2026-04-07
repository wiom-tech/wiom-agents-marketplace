# OSSPEC Creator — Skills Index
## Version 1.0

Routing contract per AGENT_STACK_FORMAT.md §5.2.

---

## Available Skills

| # | Skill | Folder | Type | Mandatory | Purpose |
|---|---|---|---|---|---|
| 1 | PRD YAML Generator | /prd-yaml-generator/ | Domain | Yes | Generate structured YAML PRD from OS document with Phase C/A/B conformance checking |
| 2 | ESR/SPR Validator | /esr-spr-validator/ | Domain | Yes | Verify all events and parameters in generated YAML against ESR and SPR for completeness and accuracy |

---

## Execution Order

```
prd-yaml-generator → esr-spr-validator
```

Sequential. Both mandatory. prd-yaml-generator output feeds esr-spr-validator.

---

## Conflict Resolution

| Conflict | Resolution |
|---|---|
| ESR/SPR Validator finds S1 (event not in ESR, param not in SPR) | STOP. Return to human with specific finding. Do not emit YAML. |
| ESR/SPR Validator flags missing events from ESR cross-check | Return finding to human. Re-run prd-yaml-generator with addition, then re-run esr-spr-validator. |
| PRD YAML Generator conformance fails after 3 passes | Emit conformance-gaps.md. Do not emit prd.yaml. Surface to human for resolution. |
| Human contradicts earlier answer | PAUSE. Surface both. Human picks. Log. |

---

## Re-Run Triggers

| Trigger | What Re-Runs |
|---|---|
| ESR/SPR Validator flags missing events from ESR | prd-yaml-generator + esr-spr-validator |
| Human modifies OS | Both skills. Log. Version bump if structural. |
| Human modifies ESR or SPR | esr-spr-validator only |
| Human changes objective or constraints | Both skills. Major version bump. |

---

*OSSPEC Creator — Skills Index v1.0*
