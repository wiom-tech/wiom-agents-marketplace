# Agent Stack Format — Canonical Standard
## Version 2.4

This is the sole normative document for the agent stack format. No other file may redefine what is defined here. Other files may only reference sections of this document using "per AGENT_STACK_FORMAT.md §X."

---

## §1 — The 3+3 Model

Every agent has 6 layers split into two groups:

```
HUMAN PROVIDES:                    AGENT DERIVES:
  Layer 1: Input                     Layer 4: Checks
  Layer 2: Instructions              Layer 5: Judgment
  Layer 3: Skills                    Layer 6: Output
```

### Build Time vs Run Time

| Phase | What Happens | Frequency |
|---|---|---|
| **Build time** | Agent Builder creates the agent package. Human provides Instructions + Skills. Package is versioned and locked. | Once (then versioned) |
| **Run time** | The built agent executes against specific inputs. Human provides files + run-time parameters. Agent produces output. | Many times |

Instructions and Skills are build-time artifacts (static, versioned). Inputs and Outputs are run-time artifacts (vary per execution). See §12 for run-time protocol.

---

## §2 — No Restatement Rule

No file in an agent package may redefine format, rules, or definitions already stated in this document. Files may only:
- Reference: "per AGENT_STACK_FORMAT.md §X"
- Implement: apply the rules defined here to their specific context
- Extend: add agent-specific content not covered by this standard

If this document and another file contradict, this document wins.

---

## §3 — Layer 1: Input (human provides)

The files and artifacts this agent reads.

At build time, AGENT_SPEC defines input structure (roles, version gates, types). At run time, the human provides specific files matching that structure.

**Format:**

```
INPUT MANIFEST (build-time — defines structure):
─────────────────────
Required:
  - role: SOURCE_OF_TRUTH
    type: [what kind of file]
    version_min: [minimum version]
    conformance_min: [PASS | CONDITIONAL_PASS]

  - role: REFERENCE
    type: [what kind of file]
    version_min: [minimum version]

Optional:
  - role: REFERENCE
    type: [what kind of file]
    fallback: [what happens if missing]

VERSION GATE: If any REQUIRED input fails version_min or conformance_min → K0b fires.
─────────────────────

RUN-TIME INPUT (per execution — fills the structure):
  - file: [specific filename] (SOURCE_OF_TRUTH)
  - file: [specific filename] (REFERENCE)
```

**Rules:**
- Every input has a role: SOURCE_OF_TRUTH (derive from) or REFERENCE (validate against)
- At least one SOURCE_OF_TRUTH must exist
- Inputs are read-only — agent never modifies its own inputs
- Optional inputs must define fallback behavior

**Complete when:**
- [ ] Input structure defines roles, types, version gates
- [ ] Optional vs required distinguished
- [ ] Version gate stated

---

## §4 — Layer 2: Instructions (human provides)

Why this agent exists, what form its output takes, what it can and cannot do, and when it must stop.

**Format:**

```
OBJECTIVE:
[One sentence. Testable.]

MEASURE OF SUCCESS:
[Concrete, binary measures.]

OUTPUT FORMAT:
[Explicit structure — sections, file type, folder layout.]

CONSTRAINTS:
- Skill binding: [MUST USE / MUST REFER / MUST USE §X for each skill]
- [What agent cannot do]
- [What decisions agent cannot make]

KILL CONDITIONS:
[Universal + agent-specific]
```

### §4.1 — Objective and Measure of Success

**Objective:** One sentence stating what the agent produces and why. Must be testable — you can read the output and objectively determine if it's met. The objective serves both the agent (when am I done?) and the human (what am I building?).

**Measure of success:** Concrete, binary measures that verify the objective is met. These seed the derived checks in Layer 4.

A good measure of success is countable or comparable:
- "Every OS enum has a label key (count match)"
- "Zero hardcoded strings in display rules"
- "24 invariants → 24 code enforcements"

If the human cannot state a measure of success, the objective is not clear enough.

### §4.2 — Output Format

The explicit structure of the agent's output. Defined at build time as the default. May be adjusted at run time per §12.

Agent may suggest format based on objective when the human is uncertain. Format suggestion is structural, not content — it is permitted.

### §4.3 — Constraints

Constraints define the agent's operating edges and its relationship to skills.

**Skill binding levels:**

| Binding | Meaning | Check Implication |
|---|---|---|
| MUST USE /skill/ | Execute entire skill. Every rule enforced. | Checks must verify every rule. |
| MUST REFER /skill/ | Load as knowledge. Apply what's relevant. | No automatic checks. |
| MUST USE /skill/ §RULES R1-R5 | Execute only named rules. | Checks verify named rules only. |

Constraints may forward-reference skills not yet provided. Skill Orchestrator validates that every referenced skill exists in Layer 3.

### §4.4 — Kill Conditions

**Universal (every agent inherits — mandatory):**

| Code | Condition | Action |
|---|---|---|
| K0a | Input modified during run | ABORT |
| K0b | Input version stale | ABORT |
| K0c | Agent attempting to resolve JUDGMENT | ABORT + escalate |
| K0d | Output not traceable to input | ABORT |
| K0e | Scope expansion (creating artifact no input requested) | ABORT |

**Agent-specific:** At least 1 required. Each has a code, condition, and action (ABORT / STOP / BLOCK / PAUSE).

**Complete when:**
- [ ] Objective is one sentence and testable
- [ ] Measure of success is concrete and binary
- [ ] Output format is explicit
- [ ] Constraints list skill bindings
- [ ] Constraints define prohibitions
- [ ] 5 universal kills included
- [ ] At least 1 agent-specific kill
- [ ] No ambiguous language

---

## §5 — Layer 3: Skills (human provides)

The execution logic this agent uses. Human uploads or describes skills. Agent validates and converts to standard format.

### §5.1 — Folder Structure

```
/skills/[agent-name]/
  SKILLS_INDEX.md           ← skill routing
  CONTEXT.md                ← shared domain knowledge
  /[skill-1]/SKILL.md
  /[skill-n]/SKILL.md
```

### §5.2 — SKILLS_INDEX.md Format

SKILLS_INDEX.md must contain exactly these 4 sections, in this order:

**Section 1 — Available Skills**

Mandatory table with these columns (all required):

| Column | Type | Valid Values |
|---|---|---|
| # | Integer | Sequential from 1 |
| Skill | String | Canonical skill name |
| Folder | Path | `/skill-name/` (relative to agent root) |
| Type | Enum | `foundation` or `domain` |
| Mandatory | Enum | `Yes` or `Conditional` |
| Purpose | One sentence | What this skill produces |

**Section 2 — Execution Order**

Code block. Lists skill folder names in execution sequence. States whether sequential or parallel. States explicit dependencies.

**Section 3 — Conflict Resolution**

Table with: Conflict scenario → Resolution rule. At minimum one row required.

**Section 4 — Re-Run Triggers**

Table with: Trigger → Skills that re-run. At minimum one row required.

### §5.3 — CONTEXT.md Format

CONTEXT.md must contain exactly these 4 sections, in this order:

**Section 1 — Domain**

Prose. States: product, users, output. One short paragraph.

**Section 2 — Current Versions**

Mandatory table:

| Column | Content |
|---|---|
| Document | Filename |
| Version | vX.Y format |
| Role | `SOURCE_OF_TRUTH` or `REFERENCE` |

**Section 3 — Canonical Sources**

Mandatory table:

| Column | Content |
|---|---|
| Data Type | What kind of information |
| Source File | Filename |
| Section | §X or section name |

**Section 4 — Key Decisions Already Locked**

Mandatory table (leave empty with headers if none exist):

| Column | Content |
|---|---|
| Decision | What was decided |
| Source | Who or which document decided it |
| Impact | What it gates or constrains |

### §5.4 — SKILL.md Format (9 mandatory sections)

Each SKILL.md must contain exactly these 9 sections, in this order:

| # | Section | Format Required | Mandatory | Content Rules |
|---|---|---|---|---|
| 1 | PURPOSE | One sentence | Yes | States why this skill exists. Must be testable. |
| 2 | WHEN | Prose or condition list | Yes | States what triggers this skill and what ends it. |
| 3 | REQUIRED INPUTS | Table: Input → Why | Yes | Every input the skill reads and the reason it needs it. |
| 4 | FRAMEWORK | Numbered list (Step N: ...) | Yes | Each step names what it produces. No prose paragraphs between steps. |
| 5 | RULES | Numbered IF-THEN (RN: IF ... → ...) | Yes* | One trigger per rule. One action per rule. |
| 6 | OUTPUT | Prose or table | Yes | Names exactly what this skill produces. |
| 7 | FAIL CONDITIONS | Table: Condition → Severity → Action | Yes | Severity: S1/S2/S3. Action: STOP/FLAG/WARN. |
| 8 | DOES NOT DO | Bulleted list | Yes | Explicit exclusions. Prevents scope bleed. |
| 9 | TRACEABILITY | Table: This Skill Enforces → Authority | Yes | Authority citations only (§X format). No input file references. |

*RULES — if this skill has no conditional logic: write exactly `"No conditional rules. Framework steps are sufficient."`

**Valid RULES format:**
- Valid: `RN: IF [condition] → [action]`
- Valid actions: `STOP`, `BLOCK`, `ABORT`, `PAUSE`, `FLAG`, `WARN`, `ESCALATE`, `surface to human`, `apply rule`, `return to human with [specific gap]`
- Invalid: `RN: IF X → consider Y` — "consider," "may," "should" are not actions
- Invalid: Prose paragraph instead of IF-THEN
- Invalid: Multiple triggers in one rule — split into separate rules

**Valid FRAMEWORK format:**
- Valid: `Step N: [action]. → [named output or named state change]`
- Invalid: Steps with no named output or state change

### §5.5 — Skill Types

| Type | Behavior | Example |
|---|---|---|
| Foundation | Loaded first. Referenced by domain skills. Does not produce final output directly. | CEF, codegen-discipline |
| Domain | Executes in order. Produces the actual output. May depend on foundation skills. | entity-codegen, event-handler-codegen |

### §5.6 — Skill Dependency

A skill may declare dependency on another skill in its REQUIRED INPUTS. Skill Orchestrator validates that every declared dependency exists in the package.

**Complete when:**
- [ ] All skills provided or described
- [ ] Each skill has all 9 sections
- [ ] Execution order defined
- [ ] Conflict resolution rule exists
- [ ] No two skills produce same output field
- [ ] All skill dependencies exist in the package
- [ ] All skills referenced in Instructions §4.3 exist

---

## §6 — Layer 4: Checks (agent derives)

Binary verification of the agent's output. Derived from measures of success (§4.1), constraints (§4.3), and skill rules (Layer 3).

**Format:**

```
| Code | Check | Enforces | Severity | FAIL Action |
|---|---|---|---|---|
| C1 | [what is verified] | [measure/constraint/rule] | S1/S2/S3 | STOP/FLAG/WARN |
```

**Derivation rules:**
- Start with measures of success — each measure becomes one or more checks
- Every constraint with MUST USE binding must have at least one check per rule
- Every constraint with MUST USE §X binding must have checks for named rules only
- Agent proposes checks + severity. Human confirms severity.

**Severity (universal definitions):**
- S1 — STOP. Output not emitted.
- S2 — FLAG. Output emitted with conditions.
- S3 — WARN. Output clean. Finding logged.

**Conformance (universal thresholds):**
- Zero S1 = PASS
- Zero S1, any S2 = CONDITIONAL PASS
- Any S1 = FAIL (output not emitted)

---

## §7 — Layer 5: Judgment (agent surfaces)

Findings the agent cannot resolve mechanically, surfaced to human.

### §7.1 — Judgment Triggers (exhaustive)

Judgment fires ONLY on:
1. Contradiction between human inputs (within a layer or across layers)
2. Unresolved skill ownership conflict (two skills, same output)
3. Unresolved traceability gap (constraint with no skill or check)
4. Coverage gap (objective requires capability no skill provides)

No other trigger is valid.

### §7.2 — Escalation Card Format

```
JUDGMENT ESCALATION:
─────────────────────
Finding: [what]
Trigger: [which of the 4 triggers — by number]
Severity: [S1/S2]

Attempted resolution:
- [what was checked, what was the result]

Options:
  (A) [option + consequence]
  (B) [option + consequence]

Agent recommendation: [A or B, one-line rationale]
─────────────────────
```

### §7.3 — Pre-Escalation Filter

Before creating a JUDGMENT card, agent must:
1. Check if any existing rule, decision, or constraint resolves the finding
2. If resolvable → apply mechanically. Do not escalate.
3. Never surface as JUDGMENT anything resolvable by applying an existing rule.

### §7.4 — Judgment Resolution

When human provides a JUDGMENT decision:
1. Agent applies the decision
2. Agent logs it in LOG.md with timestamp + rationale
3. Agent checks if decision triggers a version bump

### §7.5 — "You Decide" Rule

IF human says "you decide" / "up to you" / "whatever works" in response to any question or JUDGMENT card:
1. This is NOT permission to proceed. It is an incomplete answer.
2. Agent must create a JUDGMENT card with:
   - Finding: Human deferred decision on [X]
   - Trigger: #1 (incomplete human input)
   - Options: (A) [agent's specific recommendation] (B) [alternative] (C) Human provides own answer
3. Human must select A, B, or C with explicit confirmation.
4. "You decide" is never valid resolution of a JUDGMENT card.
5. Selection logged with AUTHORITY citation.

This rule applies at both build time and run time, to the Agent Builder and to all agents it creates.

### §7.6 — JUDGMENT Is Synchronous

When JUDGMENT fires, the agent STOPS on that item. It does not:
- Write "PENDING" or "TBD" or "TO BE CONFIRMED" and continue
- Fill the field with a provisional answer and mark it for later review
- Defer the question to a post-completion review
- Batch JUDGMENT items for end-of-run confirmation

The only valid sequence is: agent asks → human answers → agent applies → agent continues.

A placeholder is not a pause. It is a silent skip disguised as governance. Any output containing "PENDING," "TBD," or equivalent placeholders where a JUDGMENT was required is invalid — conformance = FAIL.

This rule applies at both build time and run time, to the Agent Builder and to all agents it creates.

---

## §8 — Layer 6: Output (agent produces)

The artifact this agent produces, with conformance stamp, dependency, log, and gate.

### §8.1 — Conformance Stamp Format

```
OUTPUT:
─────────────────────
Artifact: [filename/folder]
Agent: [name] v[X]
Date: [date]

Input versions consumed:
  - [file] v[X] (role)

Checks: C1 ✅ C2 ✅ C3 ⚠ ...
Conformance: [PASS / CONDITIONAL PASS / FAIL]
Conditions: [S2/S3 findings if any]
JUDGMENT items: [count — must be 0]

Downstream dependency:
  - [who consumes this] requires version ≥ [X]

OUTPUT GATE:
  ✅ No S1 check failures
  ✅ No kill conditions triggered
  ✅ No unresolved JUDGMENT items
  → Output emitted.
─────────────────────
```

### §8.2 — LOG.md Format

```
# Agent Run Log
Agent: [name] v[X]
Date: [date]
Run-time instructions: [any per-run overrides, or "none"]
─────────────────────
[timestamp] [LAYER/SKILL]: [event]. [detail].
[timestamp] JUDGMENT #[N]: [finding]. Human decided: [decision].
[timestamp] DECISION: [what agent decided]. TYPE: [STRUCTURAL/STYLISTIC]. AUTHORITY: [source].
─────────────────────
```

**Decision logging — mandatory before acting:**

Every decision the agent makes during execution must be logged BEFORE the decision is applied. If it's not in the log, it didn't happen. If it happened but isn't logged, the output is invalid.

**Decision log entry format:**

```
[timestamp] DECISION: [what the agent is about to decide]
TYPE: STRUCTURAL | STYLISTIC
AUTHORITY: [which human answer authorizes this]
```

**Decision types:**

| Type | Definition | Requires AUTHORITY? | Example |
|---|---|---|---|
| STRUCTURAL | Changes what the output contains (sections, fields, inclusion/exclusion, severity, scope) | Yes — must cite human answer | Section structure, extraction targets, severity assignment, trimming rules |
| STYLISTIC | Changes how it looks, not what it says (formatting, tone, phrasing) | No — agent logs rationale | Bullet points vs prose, tone, jargon handling |

**Rules:**
- STRUCTURAL decision without human AUTHORITY → S1 STOP (output invalid)
- STYLISTIC decision → agent proceeds, logs rationale. Human can review in LOG.md.
- "You decide" from human is NOT valid AUTHORITY — see §7.5

### §8.3 — Versioning Rules

| Change | Version Bump |
|---|---|
| First build | v1.0 |
| JUDGMENT changes structure | major (v1.x → v2.0) |
| Skill added/removed | minor (v1.0 → v1.1) |
| Instruction/objective changed | major (v1.0 → v2.0) |
| Skill content updated | patch (v1.0 → v1.0.1) |

### §8.4 — Output Gate

Output is emitted ONLY when:
- All human layers provided (Input, Instructions, Skills)
- Zero S1 check failures
- Zero unresolved JUDGMENT items
- Zero placeholder values (PENDING/TBD) where JUDGMENT was required
- LOG.md complete

If any condition fails → no output.

---

## §9 — Cross-Layer Relationships

```
INPUT feeds → INSTRUCTIONS (objective defines what input is processed for)
INSTRUCTIONS feeds → SKILLS (constraints bind to skills via MUST USE/REFER)
SKILLS feeds → CHECKS (agent derives checks from measures + constraints + rules)
CHECKS feeds → JUDGMENT (unresolvable findings escalate)
JUDGMENT feeds → OUTPUT (all resolved before output emits)
OUTPUT feeds → next agent's INPUT (conformance stamp is the passport)
```

---

## §10 — Agent Package File Structure

```
/skills/[agent-name]/
  RUN.md              ← execution entry point (agent-derived — any LLM reads this to start)
  AGENT_SPEC.md       ← agent-specific: objective, measures, format, constraints, kills, checks, output
  SKILLS_INDEX.md     ← skill routing: types, order, dependencies, conflicts
  CONTEXT.md          ← shared domain knowledge
  /[skill-1]/SKILL.md ← individual skill (9 sections)
  /[skill-n]/SKILL.md
  LOG.md              ← audit trail (produced on each run)
```

All format details for these files are defined here in §3-§8. The files themselves contain only agent-specific implementation content.

### §10.1 — RUN.md (agent-derived)

RUN.md is the execution entry point. Any LLM reads this file to start the agent. It is derived by Assembly — not human-written.

**Format:**

```
# RUN THIS AGENT

You are [agent name]. Read your specification files, then follow the run-time protocol.

## Boot sequence
1. Read AGENT_SPEC.md (your objective, constraints, kills)
2. Read SKILLS_INDEX.md (your skill routing)
3. Read CONTEXT.md (your domain knowledge)
4. Read each /[skill]/SKILL.md per SKILLS_INDEX order

## Run-time protocol (per AGENT_STACK_FORMAT.md §12)
1. Ask human for input files
2. Validate inputs (versions, roles, gates)
3. Ask: "What format should the output be?" (default: [from AGENT_SPEC output format])
4. Ask: "Any specific instructions for this run?"
5. Execute skills per SKILLS_INDEX
6. Run checks per AGENT_SPEC
7. Surface JUDGMENT per §7 if any
8. Produce output + LOG.md

## Rules
- §7.5: "You decide" is not permission. Surface options.
- §7.6: JUDGMENT is synchronous. No placeholders. No deferral.
- §12.4: Log STRUCTURAL decisions with AUTHORITY. STYLISTIC with rationale.
- [Agent-specific rules extracted from constraints]
- [Agent-specific kill conditions]

## Start
Begin by reading files 1-4 above, then ask human for inputs.
```

**Derivation rules:**
- Boot sequence is always the same (read spec → index → context → skills)
- Run-time protocol is always §12.1 steps
- Rules section = universal rules (§7.5, §7.6, §12.4) + agent-specific constraints and kill conditions extracted from AGENT_SPEC
- No human input needed for RUN.md — pure derivation from AGENT_SPEC

---

## §11 — Governing Principle

**Generate with agents. Police with agents. Decide with humans.**

Human provides: Input, Instructions, Skills.
Agent derives: Checks, Judgment, Output.

---

## §12 — Run Time

At build time, the Agent Builder creates the agent package (static, versioned). At run time, the built agent executes against specific inputs.

### §12.1 — Run-Time Protocol

Every agent run follows this sequence:

```
1. Human provides specific input files for this run
2. Agent validates inputs per §3 (versions, roles, gates)
   → If validation fails → agent states what's wrong. Does not proceed.
3. Agent asks: "What format should the output be?"
   → Default format per §4.2 is proposed
   → Human confirms, adjusts, or asks for suggestion
   → Agent may suggest format based on objective (structural, not content)
4. Agent asks: "Any specific instructions for this run?"
   → Human may narrow scope, adjust focus, or say "none"
   → Run-time instructions are logged in LOG.md
5. Agent executes skills per SKILLS_INDEX.md
6. Agent runs checks per §6
7. Agent surfaces judgment per §7 (if any)
8. Agent produces output per §8 (includes LOG.md for this run)
```

### §12.2 — Run-Time Instructions

Run-time instructions are per-execution parameters from the human. They may narrow or focus the agent's work but cannot override build-time definitions.

| Allowed | Example |
|---|---|
| Narrow scope | "Focus on deposit entry types only" |
| Adjust format | "Use YAML instead of markdown for this run" |
| Skip optional | "Skip Hindi copy — English only" |
| Add context | "This module has a new state PENDING_REVIEW not yet in OS — flag it" |

| Not Allowed | Why |
|---|---|
| Override constraint | "Don't check for hardcoded strings" — violates build-time constraint |
| Override kill condition | "Proceed even if OS is unlocked" — violates K0b |
| Add skills | Skills are build-time. Run-time cannot add execution logic. |
| Change objective | Objective is build-time. Different objective = different agent. |

**Rule:** Run-time instructions may restrict. They may not expand.

### §12.3 — Run-Time LOG.md

Every run produces its own LOG.md. The log records:
- Specific input files provided
- Run-time instructions (or "none")
- Output format confirmed
- All decisions (STRUCTURAL with AUTHORITY, STYLISTIC with rationale)
- All check results
- All JUDGMENT decisions
- Final conformance result

### §12.4 — Decision Authority Rule (applies to all agents)

During skill execution, if the agent must make any choice not explicitly covered by a skill rule:

1. Classify: STRUCTURAL or STYLISTIC (per §8.2)
2. IF STRUCTURAL → STOP. Surface options to human. Log choice with AUTHORITY.
3. IF STYLISTIC → proceed. Log with rationale.
4. "You decide" / "up to you" from human is not resolution (per §7.5). Agent must propose specific options.

This rule is inherited by every agent the Agent Builder creates. It applies at both build time and run time.

---

*Agent Stack Format — Canonical Standard v2.4*
*The sole normative document. No file may redefine what is defined here.*
*Build time creates the agent. Run time uses it.*
