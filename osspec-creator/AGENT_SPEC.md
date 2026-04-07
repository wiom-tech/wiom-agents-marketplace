# OSSPEC Creator — Agent Specification
## Version 1.0

Conforms to AGENT_STACK_FORMAT.md v2.4. Format definitions per §3–§8, §10.1, run-time protocol per §12.

---

## INPUT (per §3)

**Build-time input manifest:**

| Role | Type | Version Gate | Fallback |
|---|---|---|---|
| SOURCE_OF_TRUTH | OS file (Markdown) | Filename must contain version number + "locked" | K0b fires — ABORT |
| REFERENCE | ESR — Event Schema Reference (Markdown) | Filename must contain version number + "locked" | K0b fires — ABORT |
| REFERENCE | SPR — Standard Parameters Reference (Markdown) | Filename must contain version number + "locked" | K0b fires — ABORT |

All three inputs required. No optional inputs. No fallback — if any is missing or fails the version gate, agent does not proceed.

**Version gate:** If any file's filename is missing a version number OR "locked" → K0b fires → ABORT.

---

## INSTRUCTIONS (per §4)

**Objective:** Produce a precise, accurate YAML PRD from an OS document using ESR and SPR as references to ensure completeness and accuracy.

**Measure of success:**
- M1: Every state, rule, and logic block from OS appears in YAML (count match)
- M2: Zero event names in YAML not found in ESR
- M3: Zero parameters in YAML not found in SPR
- M4: Zero YAML content (states/transitions/events/params) without OS source
- M5: All REQUIRED sections present in emitted YAML
- M6: Each CONDITIONAL section present with OS justification or explicitly excluded in header
- M7: Zero `<<...>>` placeholder strings in emitted YAML

**Output format:** `{service-name}-prd.yaml` conforming to yaml-prd-template.yaml v1.0.
Output path: `docs/os/{service-name}-prd.yaml`.
If conformance fails after 3 passes: `docs/os/{service-name}-prd-conformance-gaps.md`.
Note: `# TBD` YAML comments are permitted for mandatory sections with no OS data. M7 targets `<<...>>` placeholder strings only.

**Constraints:**
- MUST USE /prd-yaml-generator/
- MUST USE /esr-spr-validator/
- Never modify or reinterpret OS language — extract verbatim
- Never resolve ambiguity in the OS unilaterally — surface to human
- Never include CONDITIONAL sections the OS does not warrant

**Kill conditions:**

| Code | Condition | Action |
|---|---|---|
| K0a | Input modified during run | ABORT (universal — §4.4) |
| K0b | Input version stale (filename missing version number or "locked") | ABORT (universal — §4.4) |
| K0c | Agent attempting to resolve JUDGMENT | ABORT + escalate (universal — §4.4) |
| K0d | Output not traceable to input | ABORT (universal — §4.4) |
| K0e | Scope expansion (artifact no input requested) | ABORT (universal — §4.4) |
| K1 | OS file structurally incomplete (missing major sections) | ABORT |
| K2 | ESR event referenced in OS not found in ESR file | ABORT |
| K3 | SPR parameter referenced in OS not found in SPR file | ABORT |

---

## SKILLS (per §5)

```
/skills/osspec-creator/
  SKILLS_INDEX.md
  CONTEXT.md
  /prd-yaml-generator/SKILL.md
  /esr-spr-validator/SKILL.md
```

---

## CHECKS (derived per §6)

| Code | Check | Enforces | Severity | Action |
|---|---|---|---|---|
| C1 | Every state from OS appears in YAML state_machines.states | M1 | S1 | STOP |
| C2 | Every rule, logic block, and transition from OS has corresponding YAML entry | M1 | S1 | STOP |
| C3 | Zero event names in YAML not found in ESR | M2 + ESR-SPR R1 + K2 | S1 | STOP |
| C4 | Zero parameters in YAML not found in SPR | M3 + ESR-SPR R2 + K3 | S1 | STOP |
| C5 | Zero YAML content (states/transitions/events/params) without OS source | M4 | S1 | STOP |
| C6 | All REQUIRED sections present in emitted YAML | M5 | S1 | STOP |
| C7 | Each CONDITIONAL section present with OS justification or explicitly excluded | M6 | S2 | FLAG |
| C8 | Zero `<<...>>` placeholder strings in emitted YAML | M7 | S1 | STOP |
| C9 | REQUIRED sections with no OS data have header + empty list + `# TBD` (not omitted) | PRD R1 | S1 | STOP |
| C10 | service_id follows `csp-{name}-service` pattern (no -svc/-srv/-app) | PRD R2 | S2 | FLAG |
| C11 | aws-eventbridge in dependencies iff OS defines timer-driven transitions | PRD R3 | S2 | FLAG |
| C12 | Every event mentioned in OS prose appears in events[] | PRD R4 | S1 | STOP |
| C13 | States with no outgoing transitions have `terminal: true` | PRD R5 | S2 | FLAG |
| C14 | operational_config enabled flags match conditions (jwt, outbox, opt-lock, upi) | PRD R6–R9 | S2 | FLAG |
| C15 | INTERNAL events absent from registry_event_mapping.producers | PRD R10 | S2 | FLAG |
| C16 | Conformance fails after 3 passes → conformance-gaps.md emitted, not prd.yaml | PRD R11 | S1 | STOP |
| C17 | LOCKED OS file not modified during generation | PRD R12 | S1 | STOP |
| C18 | Every event-handling flow includes processed_events idempotency step | PRD R13 | S2 | FLAG |
| C19 | Timer-driven flows include EventBridge scheduling as explicit step | PRD R14 | S2 | FLAG |
| C20 | All `next:` references in flows resolve to existing `step.id` in same flow | PRD R15 | S1 | STOP |
| C21 | Phase C→A→B conformance check executed on every generation | PRD R16 | S1 | STOP |
| C22 | Event schema_fields match ESR schema definitions | ESR-SPR R3 | S2 | FLAG |
| C23 | No OS-referenced event absent from YAML per ESR cross-check | ESR-SPR R4 | S2 | FLAG |

---

## JUDGMENT (per §7)

Triggers per §7.1 only. Escalation card format per §7.2. Pre-escalation filter per §7.3.

---

## OUTPUT (per §8)

`docs/os/{service-name}-prd.yaml` OR `docs/os/{service-name}-prd-conformance-gaps.md` per §10. Output gate per §8.4.

---

*OSSPEC Creator — Agent Specification v1.0*
