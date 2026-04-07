# OSSPEC Creator — Shared Context
## Version 1.0

Implementation context per AGENT_STACK_FORMAT.md §5.3.

---

## Domain

- **Product:** Machine-readable YAML PRDs for the Wiom CSP build pipeline
- **Users:** Engineers and AI codegen agents consuming PRD YAMLs to generate service code
- **Output:** `{service-name}-prd.yaml` conforming to yaml-prd-template.yaml v1.0

---

## Current Versions

| Document | Version | Role |
|---|---|---|
| yaml-prd-template.yaml | v1.0 | SOURCE_OF_TRUTH (YAML schema) |
| AGENT_SPEC.md | v1.0 | SOURCE_OF_TRUTH |
| SKILLS_INDEX.md | v1.0 | SOURCE_OF_TRUTH |
| CONTEXT.md | v1.0 | SOURCE_OF_TRUTH |
| prd-yaml-generator/SKILL.md | v2.0 | SOURCE_OF_TRUTH |
| esr-spr-validator/SKILL.md | v1.0 | SOURCE_OF_TRUTH |

---

## Canonical Sources

| Data Type | Source File | Section |
|---|---|---|
| Format definitions (all layers) | AGENT_STACK_FORMAT.md | §1–§12 |
| Agent objective + measures of success | AGENT_SPEC.md | INSTRUCTIONS |
| Kill conditions | AGENT_SPEC.md | INSTRUCTIONS |
| Derived checks | AGENT_SPEC.md | CHECKS |
| Skill routing + execution order | SKILLS_INDEX.md | Execution Order |
| YAML schema + section taxonomy | yaml-prd-template.yaml | — |
| YAML generation rules (26 sections + conformance phases) | prd-yaml-generator/SKILL.md | FRAMEWORK + RULES |
| ESR/SPR validation rules | esr-spr-validator/SKILL.md | FRAMEWORK + RULES |

---

## Key Decisions Already Locked

| Decision | Source | Impact |
|---|---|---|
| ESR/SPR validation is a separate skill, not embedded in YAML generator | JUDGMENT #1 — Human selected option B | Two-skill pipeline: generate then validate |
| `# TBD` YAML comments permitted for mandatory sections with no OS data | Human decision during skill conversion | M7 check targets `<<...>>` strings only, not `# TBD` comments |
| `return_202_silently` is authoritative for idempotency on_duplicate field | Human decision — overrides yaml-prd-template.yaml value | Used in prd-yaml-generator SKILL.md §17 (Idempotency) |
| Cross-PRD Python script excluded from skill | Human decision during conversion | Cross-PRD validation is a manual/external check — not executed by this agent |
| OS must be LOCKED before generation — no draft PRDs | prd-yaml-generator SKILL.md — Pre-flight | K1 fires if OS is not locked |

---

*OSSPEC Creator — Shared Context v1.0*
