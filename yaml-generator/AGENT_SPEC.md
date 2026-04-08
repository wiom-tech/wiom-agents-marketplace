# YAML Generator Agent — Specification
## Version 1.0

Conforms to AGENT_STACK_FORMAT.md v2.4. Format definitions per §3–§8, §10.1, run-time protocol per §12.

---

## INPUT (per §3)

**Build-time input manifest:**

| Role | Type | Version Min | Conformance Min | Fallback |
|---|---|---|---|---|
| SOURCE_OF_TRUTH | OS file (locked markdown) | Any | Filename contains "locked" | K0b fires — ABORT |

**Version gate:** If OS filename does not contain "locked" → K0b fires. Do not proceed.

---

## INSTRUCTIONS (per §4)

**Objective:** Read a locked OS file and produce a complete, accurate, machine-readable YAML output that captures all information from the OS exactly as written, for use in code generation.

**Measure of success:**
- Every field in the OS appears in the YAML
- Zero fields in the YAML that don't trace to the OS
- YAML is valid and parseable
- All logic and governance rules captured accurately and comprehensively per the OS

**Output format:** Single `.yaml` file — `[service-name]-prd.yaml`

**Constraints:**
- MUST USE /yaml-generation/
- Agent must never invent information
- Agent must never interpret or complete ambiguous information
- Agent cannot resolve ambiguous OS content — must surface to human
- Agent cannot choose between conflicting values — must surface to human

**Kill conditions (universal per §4.4 + agent-specific):**

| Code | Condition | Action |
|---|---|---|
| K0a | Input modified during run | ABORT |
| K0b | OS file not locked (no "locked" in filename) | ABORT |
| K0c | Agent attempting to resolve JUDGMENT | ABORT + escalate |
| K0d | Output not traceable to OS input | ABORT |
| K0e | Scope expansion (creating artifact no input requested) | ABORT |
| K1 | Ambiguous OS content detected | PAUSE — surface to human |
| K2 | Conflicting values in OS detected | PAUSE — surface to human |

---

## SKILLS (per §5)

```
/yaml-generator/
  SKILLS_INDEX.md
  CONTEXT.md
  /yaml-generation/SKILL.md
```

---

## CHECKS (derived per §6)

### Input Checks

| Code | Check | Enforces | Severity | Action |
|---|---|---|---|---|
| C1 | OS filename contains "locked" before processing | K0b, R1 | S1 | STOP |

### Phase C — OS Internal Coherence

| Code | Check | Enforces | Severity | Action |
|---|---|---|---|---|
| C2 | Closed vocabulary — no undefined state/enum tokens in OS | C1 | S2 | FLAG |
| C3 | No cross-section contradictions in OS | C2 | S2 | FLAG |
| C4 | Transition completeness — no orphan/unreachable states in OS | C3 | S2 | FLAG |
| C5 | No OS_CRITICAL dependency readiness gaps blocking core flows | C4 | S1 | STOP |
| C6 | Invariants operationally defined in OS or YAML | C5 | S2 | FLAG |
| C7 | No unresolved dual authority in OS | C6 | S2 | FLAG |
| C8 | Parameter reference integrity — all OS param IDs defined | C7 | S2 | FLAG |

### Phase A — OS-to-YAML

| Code | Check | Enforces | Severity | Action |
|---|---|---|---|---|
| C9 | Every OS state in state_machines[].states | A1, M1 | S1 | STOP |
| C10 | Every OS transition in state_machines[].transitions | A2, M1 | S1 | STOP |
| C11 | Every invariant ID from OS cited in YAML | A3, M4 | S2 | FLAG |
| C12 | Every upstream dependency from OS in dependencies[] | A4, M1 | S1 | STOP |
| C13 | Every downstream signal from OS in events[] (emitted) | A5, M1 | S1 | STOP |
| C14 | Every consumed signal (incl. prose-discovered) in events[] | A6/A6a, M1 | S1 | STOP |
| C15 | Every failure mode from OS in failure_modes[] | A7, M1 | S1 | STOP |
| C16 | Every OS parameter in capacity/config or referenced | A8, M4 | S2 | FLAG |
| C17 | No invented state, transition, or event name | A9, M2, K0d | S1 | STOP |
| C18 | Upstream signal mapping present if OS defines it | A10, M1 | S1 | STOP |
| C19 | Derived-field model present if OS defines it | A11, M1 | S1 | STOP |
| C20 | Structural assumptions present if OS defines them | A12, M1 | S2 | FLAG |
| C21 | Write-back prohibition present if OS defines it | A13, M1 | S1 | STOP |
| C22 | Trigger-to-mutation matrix present if OS has it | A14, M1 | S1 | STOP |
| C23 | Every guard name has guard_specifications entry with error_code + http_status | A15, M4 | S1 | STOP |
| C24 | Every REST endpoint has endpoint_permissions entry | A16, M4 | S1 | STOP |

### Phase B — YAML Internal Consistency

| Code | Check | Enforces | Severity | Action |
|---|---|---|---|---|
| C25 | State enum alignment — state_machines[].states matches data_model entities[].fields[state].enum | B1, M4 | S1 | STOP |
| C26 | Every consumed event has idempotency entry | B2, M4 | S1 | STOP |
| C27 | Consumed event emitted_by in dependencies[] | B3, M4 | S1 | STOP |
| C28 | Dependency events[] entries exist in events[] | B4, M4 | S1 | STOP |
| C29 | Flow step connectivity — all next references valid | B5, M4 | S1 | STOP |
| C30 | state_machines entities in data_model.entities[] | B6, M4 | S1 | STOP |
| C31 | concurrency[] entities in data_model.entities[] | B7, M4 | S1 | STOP |
| C32 | terminal_states[] equals set of states where terminal: true | B8, M4 | S1 | STOP |
| C33 | processed_events entity in data_model.entities[] | B9, R39, M4 | S1 | STOP |
| C34 | Non-append-only domain entities have correlation_id + causation_id | B10, R38 | S2 | FLAG |
| C35 | Optimistic-locking entities have version field in data_model | B11, R36 | S1 | STOP |
| C36 | Upstream signal mapping local_states in state_machines[].states | B12, M4 | S1 | STOP |
| C37 | Trigger-mutation matrix sources in dependencies[]; triggers have corresponding consumed events | B13, M4 | S1 | STOP |
| C38 | Guard fm_ref references existing failure_modes[].id | B14, R21 | S1 | STOP |
| C39 | Endpoint roles in security.authorization.roles[] | B15, M4 | S1 | STOP |
| C40 | All events[].guards in guard_specifications[] | B16, M4 | S1 | STOP |
| C41 | Registry producers alignment — emitted non-INTERNAL events only | B17, R45 | S1 | STOP |
| C42 | Registry subscriptions alignment — consumed events only | B18, M4 | S1 | STOP |
| C43 | Outbox config enabled when namastack-outbox dependency present | B19, R41 | S1 | STOP |
| C44 | Optimistic lock config enabled when optimistic-locking strategy present | B20, R42 | S1 | STOP |
| C45 | JWT identity config enabled when JWT mechanism present | B21, R40 | S1 | STOP |

### Format + Content + Process Checks

| Code | Check | Enforces | Severity | Action |
|---|---|---|---|---|
| C46 | All mandatory YAML sections 1–4, 10–17, 19–26 present | R2, M1 | S1 | STOP |
| C47 | Conditional sections present only when OS provides relevant data | R3, M2 | S2 | FLAG |
| C48 | service_id follows csp-{service-name}-service pattern | R4 | S2 | FLAG |
| C49 | YAML output is syntactically valid and parseable | M3 | S1 | STOP |
| C50 | YAML filename is [service-name]-prd.yaml | Output format | S1 | STOP |
| C51 | Conformance check ran immediately after generation in C→A→B order | R47 | S1 | STOP |
| C52 | LOCKED OS file was not modified during generation | R48, K0a | S1 | STOP |

---

## JUDGMENT (per §7)

Triggers per §7.1 only. Escalation card format per §7.2. Pre-escalation filter per §7.3.

---

## OUTPUT (per §8)

Single `.yaml` file — `[service-name]-prd.yaml` (primary). Conformance gap report — `[service-name]-prd-conformance-gaps.md` (on conformance failure after 3 attempts). Output gate per §8.4.

---

*YAML Generator Agent — Specification v1.0*
