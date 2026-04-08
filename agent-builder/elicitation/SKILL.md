# Elicitation
## SKILL.md — Version 2.5

Conforms to AGENT_STACK_FORMAT.md §5.4 (9-section skill format).

---

## Purpose

Collect and qualify the 3 human-provided layers (Input, Instructions, Skills) through structured questioning. One layer at a time. Each layer is asked, qualified, then confirmed.

---

## When

From "build me an agent for X" until all 3 human layers are qualified and confirmed. Then passes to Skill Orchestrator.

---

## Required Inputs

| Input | Why |
|---|---|
| AGENT_STACK_FORMAT.md §3-§5, §12 | Layer definitions + run-time protocol |
| CONTEXT.md §Checklists | Quick-reference per layer |
| CONTEXT.md §Elicitation Principles | How to ask |
| Human conversation | Content |

---

## Framework

```
Step 1: Opening
  → "I'll help build this agent. I need 3 things: what it reads (Input), 
     what it does (Instructions), and how it does it (Skills). One at a time."
  → If human provides a brief, use it. Ask for what's missing.

Step 2: Layer 1 — Input
  Ask:
    → What kind of files does this agent read?
    → For each type: source of truth or reference?
    → Version requirements? Conformance requirements?
    → Any optional inputs? Fallback?
  Qualify:
    → "Is this the real source of truth, or a downstream artifact?"
  Confirm: "Layer 1 complete. [summary]. Correct?"

Step 3: Layer 2 — Instructions
  Ask objective:
    → "In one sentence — what does this agent produce and why?"
  Qualify objective:
    → "How will you measure success? Give me concrete measures."
    → IF vague: "What would you count or compare to know it worked?"
  Ask output format:
    → "What format should the output be?"
    → IF human uncertain: suggest based on objective.
       "Based on your objective, I'd recommend [X]. Here's why."
    → Human confirms or adjusts.
    → Note: this is the default format. At run time per §12, 
      the human can adjust per run.
  Ask sample output:
    → "Do you have a sample output — an example of what good output 
       looks like for this agent?"
    → IF YES: receive file. Pass to Assembly as REFERENCE input.
      Assembly uses it to validate structure match, not to copy content.
    → IF NO: proceed. Sample is optional.
  Ask constraints:
    → "What skills should it use?" (MUST USE / MUST REFER / partial)
    → "What must it NEVER do?"
    → "What decisions should it NOT make?"
  Qualify constraints:
    → "If the agent violates this, what breaks downstream?"
  Ask kill conditions:
    → "Under what conditions should it abort entirely?"
  Confirm: "Layer 2 complete. Objective: [X]. Measures: [Y]. 
            Format: [Z]. Sample: [yes/no]. Constraints: [W]. Kills: [V]. Correct?"

Step 4: Layer 3 — Skills
  Ask:
    → "Do you have skill files to upload?"
    → IF YES: receive files, pass to Skill Orchestrator
    → IF NO: for each skill named in constraints, ask:
        purpose, when, required inputs, framework steps, IF-THEN rules,
        output, fail conditions, exclusions, traceability
  Ask routing:
    → Execution order? Conflicts? Shared context?
  Confirm: "Layer 3 complete. [N] skills. Order: [X→Y→Z]. Correct?"

Step 5: Amendment Mode (per §12.5)
  Ask:
    → "Should this agent support amendment mode?"
    → "Amendment mode lets the agent update an existing locked output
       when an input document changes, instead of building from scratch."
  IF YES:
    → Ask: "Which input documents are likely to change over time?"
       (These become the CHANGED INPUT candidates)
    → Ask: "Does the agent produce founder decisions during a run
       that should be preserved across amendments?"
       (Determines whether ORIGINAL LOG loading is critical or optional)
    → Record: amendment_mode = YES, changeable_inputs = [...],
       founder_decisions_critical = YES/NO
    → Note: Assembly will add /input-diff/ and /output-comparison/ skills
  IF NO:
    → Record: amendment_mode = NO
    → Proceed

Step 6: Handoff
  → Summarize all 3 layers + amendment mode decision
  → "Passing to Skill Orchestrator for validation. Ready?"
  → If changes → re-open relevant layer
```

---

## Rules

R1: IF human says "the OS" → "Which OS? What version?"
R2: IF human says "latest" → "What's the latest locked version?"
R3: IF no SOURCE_OF_TRUTH in Layer 1 → "Which is the primary source?"
R4: IF objective not testable → ask to sharpen
R5: IF no measure of success after two follow-ups → K6 fires
R6: IF human uncertain about output format → suggest based on objective (permitted per §4.2)
R7: IF constraint has no downstream consequence → "This may be unnecessary. Keep or remove?"
R8: IF human contradicts earlier answer (within same layer or across layers) → PAUSE. Surface both answers. Human picks one. Log change.
R9: IF human names skill in constraints not provided in Layer 3 → note for Skill Orchestrator
R10: IF human says "you decide" / "up to you" / "whatever works" → per §7.5: this is NOT permission. Propose specific options. Human must pick one. Log choice with AUTHORITY.
R11: IF amendment_mode = YES and zero changeable_inputs identified → "Which inputs change? Without this, amendment mode has nothing to diff." Human must provide at least one.
R12: IF amendment_mode = YES → note in Layer 2 output that Assembly must add /input-diff/ and /output-comparison/ skills and amendment protocol to RUN.md.

---

## Output

Structured answer set for Layers 1-3 (including measures of success, output format, sample output if provided, and amendment mode decision). Passed to Skill Orchestrator.

---

## Fail Conditions

| Condition | Severity | Action |
|---|---|---|
| Human refuses to answer a layer | S1 | STOP |
| Answers contradict (within or across layers) | S1 | PAUSE — surface, human resolves |
| No measure of success after follow-up | S1 | K6 — objective unclear |
| No output format after follow-up | S2 | Suggest default, proceed |
| Zero skills provided | S1 | STOP |

---

## Does Not Do

- Does not invent answers
- Does not suggest skills unless asked
- Does not validate skill format (Skill Orchestrator does that)
- Does not derive checks (Assembly does that)
- Does not skip layers
- Does not accept vague measures without follow-up
- Does not define run-time behavior (§12 defines that for the built agent)
- Does not write placeholders and continue — per §7.6

---

## Traceability

| This Skill Enforces | Authority |
|---|---|
| 3+3 model — human provides 3 layers | §1 |
| Measure of success is mandatory | §4.1 |
| Output format is mandatory | §4.2 |
| Agent never resolves JUDGMENT | §4.4 K0c |
| "You decide" is not permission | §7.5 |
| JUDGMENT is synchronous — no placeholders | §7.6 |

---

*Elicitation — SKILL.md v2.5*
