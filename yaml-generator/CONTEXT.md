# YAML Generator — Shared Context
## Version 1.0

Implementation context per AGENT_STACK_FORMAT.md §5.3. Format definitions per AGENT_STACK_FORMAT.md — not restated here.

---

## Domain

The YAML Generator produces machine-readable PRD YAML files from locked WIOM OS (Operating Specification) documents. Users are engineers and product leads in the WIOM build pipeline. Output is a `{service-name}-prd.yaml` companion file used by the WIOM code generation pipeline.

---

## Current Versions

| Document | Version | Role |
|---|---|---|
| AGENT_STACK_FORMAT.md | v2.4 | SOURCE_OF_TRUTH |
| AGENT_SPEC.md | v1.0 | SOURCE_OF_TRUTH |
| SKILLS_INDEX.md | v1.0 | SOURCE_OF_TRUTH |
| CONTEXT.md | v1.0 | SOURCE_OF_TRUTH |
| yaml-generation/SKILL.md | v1.0 | SOURCE_OF_TRUTH |

---

## Canonical Sources

| Data Type | Source File | Section |
|---|---|---|
| Format definitions (all layers) | AGENT_STACK_FORMAT.md | §1–§12 |
| Agent objective + measures | AGENT_SPEC.md | INSTRUCTIONS |
| Skill routing + execution order | SKILLS_INDEX.md | Execution Order |
| YAML generation rules + conformance | yaml-generation/SKILL.md | FRAMEWORK, RULES |
| Run-time protocol | AGENT_STACK_FORMAT.md | §12 |

---

## Key Decisions Already Locked

| Decision | Source | Impact |
|---|---|---|
| OS file is the sole source of truth — agent never invents content | AGENT_SPEC.md — Constraints | K0d fires on any invented content |
| Ambiguous OS content surfaces to human — never interpreted | AGENT_SPEC.md — Constraints | K1 fires on ambiguity |
| Conflicting OS values surface to human — never resolved by agent | AGENT_SPEC.md — Constraints | K2 fires on conflict |
| LOCKED OS file must not be modified | yaml-generation/SKILL.md R48 | LOCKED OS is read-only during generation |
| Conformance check C→A→B mandatory on every generation | yaml-generation/SKILL.md R47 | No PRD is complete without conformance |
| Codegen blocked on unresolved CRITICAL/OS_CRITICAL findings | yaml-generation/SKILL.md R49, R50 | User acknowledgment required to proceed |

---

*YAML Generator — Shared Context v1.0*
