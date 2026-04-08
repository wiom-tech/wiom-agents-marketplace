# Input Diff
## SKILL.md — Version 1.0

Conforms to AGENT_STACK_FORMAT.md §5.4 (9-section skill format). Amendment mode skill per §12.5.5.

---

## Purpose

Diff old vs new versions of changed input documents. Produce a structured change summary and impact table mapping each change to affected output sections.

---

## When

Amendment mode only (§12.5). After input validation confirms BASELINE is valid and CHANGED INPUT(S) are newer versions of the same document type.

---

## Required Inputs

| Input | Why |
|---|---|
| BASELINE output (locked, with conformance stamp) | Establishes what was produced from old inputs |
| CHANGED INPUT(S) — old version | What the agent consumed in the original run |
| CHANGED INPUT(S) — new version | What the agent will consume in this amendment run |
| ORIGINAL LOG (optional) | Founder decisions from the original run — loaded as CONTEXT |
| Target agent's AGENT_SPEC.md | To map input changes to output section structure |

---

## Framework

```
Step 1: Extract old input versions
  → From BASELINE conformance stamp: "Input versions consumed"
  → For each CHANGED INPUT, identify the old version consumed
  → IF old version not identifiable from stamp → ask human to provide

Step 2: Structural diff per changed input
  → For each CHANGED INPUT (old vs new):
    - Sections added
    - Sections removed
    - Sections modified (content changed)
    - Enums/states added or removed
    - Transitions added or removed
    - Rules added, removed, or modified
  → Produce CHANGE SUMMARY per input document

Step 3: Present change summary to human
  → "I detected these changes in [document] v[old] → v[new]:"
  → List each change with category (addition/removal/modification)
  → Ask: "Is this complete? Any changes I missed?"
  → Human confirms or adds missed changes

Step 4: Map changes to output sections
  → For each change in CHANGE SUMMARY:
    - Which output sections (§0-§N) are affected?
    - What is the rationale? (e.g., "new state added → §3.1 states, §3.2 actions, §6 failures")
  → Produce IMPACT TABLE:
    | Input Change | Type | Affected Sections | Rationale |

Step 5: Present impact table to human
  → "Based on the input changes, these output sections are affected:"
  → Human confirms or adds sections agent missed
  → IF human adds sections → log with AUTHORITY: "Human identified additional impact"

Step 6: Load founder decisions from original run
  → IF ORIGINAL LOG provided:
    - Extract all DECISION entries with TYPE: STRUCTURAL
    - Extract all JUDGMENT resolution entries
    - Pass as CONTEXT to downstream skills
  → IF ORIGINAL LOG not provided:
    - WARN: "No original log — founder decisions from original run cannot be tracked.
      FOUNDER_OVERRIDE classification in output-comparison will be unavailable."
    - Human acknowledges. Proceed.
```

---

## Rules

R1: IF BASELINE has no conformance stamp → ABORT. Not a valid baseline for amendment.
R2: IF CHANGED INPUT is not a newer version of same document type → ABORT. "This is not an update — it is a different document."
R3: IF old version of changed input is not identifiable → ask human. Do not guess.
R4: IF change summary is empty (no differences detected) → STOP. "No input changes detected. Amendment not needed."
R5: IF human adds a missed change → add to CHANGE SUMMARY. Log: "Human identified change not detected by diff."
R6: IF human adds a missed impact → add to IMPACT TABLE. Log with AUTHORITY.
R7: IF a change affects a section that does not exist in the output template → FLAG. "Input change affects [X] but output has no section for it. This may require fresh build, not amendment."
R8: IF multiple inputs changed and changes interact (e.g., OS state change + ESR event change for same lifecycle) → note interaction in IMPACT TABLE rationale.

---

## Output

Two artifacts passed to downstream skills:

1. **CHANGE SUMMARY** — per changed input document: list of additions, removals, modifications (human-confirmed)
2. **IMPACT TABLE** — mapping of each change to affected output sections with rationale (human-confirmed)
3. **FOUNDER DECISIONS** — extracted from ORIGINAL LOG (if available), passed as CONTEXT

---

## Fail Conditions

| Condition | Severity | Action |
|---|---|---|
| BASELINE has no conformance stamp | S1 | ABORT |
| CHANGED INPUT is not newer version of same document | S1 | ABORT |
| No differences detected between old and new input | S1 | STOP — amendment not needed |
| Human rejects change summary as incomplete after two rounds | S1 | STOP — input diff unreliable |
| Change affects section not in output template | S2 | FLAG — may need fresh build |
| ORIGINAL LOG not provided | S3 | WARN — FOUNDER_OVERRIDE tracking unavailable |

---

## Does Not Do

- Does not modify any input documents
- Does not re-derive output sections (domain skills do that)
- Does not classify output changes (output-comparison does that)
- Does not decide which sections to update vs skip — all sections are re-derived
- Does not invent input changes not present in the diff
- Does not assume impact — maps change to section, human confirms

---

## Traceability

| This Skill Enforces | Authority |
|---|---|
| Amendment run-time protocol Steps 3-4 | §12.5.1 |
| Input validation for amendment mode | §12.5.1 Step 2 |
| Human confirms change summary and impact | §12.5.1 Steps 3-4 |
| Founder decisions loaded as CONTEXT | §12.5.1 Step 5 |
| Amendment mode constraints — update not expand | §12.5.4 |

---

*Input Diff — SKILL.md v1.0*
