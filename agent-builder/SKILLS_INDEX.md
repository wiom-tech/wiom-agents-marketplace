# Agent Builder — Skills Index
## Version 2.5

Routing contract per AGENT_STACK_FORMAT.md §5.2.

---

## Available Skills

| # | Skill | Folder | Type | Mandatory | Purpose |
|---|---|---|---|---|---|
| 1 | Elicitation | /elicitation/ | Domain | Yes | Collect and qualify Input, Instructions, Skills + amendment mode decision from human |
| 2 | Skill Orchestrator | /skill-orchestrator/ | Domain | Yes | Validate, convert, gap-check skills |
| 3 | Assembly | /assembly/ | Domain | Yes | Derive checks, produce agent folder + RUN.md + LOG.md + amendment skills if enabled |
| 4 | Input Diff (template) | /input-diff/ | Domain | Conditional | Template skill — copied into target agents when amendment_mode = YES |
| 5 | Output Comparison (template) | /output-comparison/ | Domain | Conditional | Template skill — copied into target agents when amendment_mode = YES |

---

## Execution Order

```
elicitation → skill-orchestrator → assembly
                                      ↓
                              (if amendment_mode = YES)
                              assembly copies /input-diff/ and /output-comparison/
                              into target agent package
```

Sequential for skills 1-3. All mandatory. Each skill's output feeds the next.
Skills 4-5 are template skills — not executed by the builder. They are copied into target agent packages by Assembly (Step 8) when amendment mode is enabled.

---

## Conflict Resolution

| Conflict | Resolution |
|---|---|
| Human contradicts earlier answer | PAUSE. Surface both. Human picks. Log. |
| Two uploaded skills overlap | Skill Orchestrator surfaces: "which owns this?" |
| Skill Orchestrator gap proposal rejected | Accepted. Gap noted in LOG.md. |
| Assembly-derived check unclear traceability | Assembly surfaces for human confirmation. |

---

## Re-Run Triggers

| Trigger | What Re-Runs |
|---|---|
| Human modifies a completed layer | That layer forward. Log. Version bump if structural. |
| Human adds skill after orchestration | skill-orchestrator + assembly. Minor version bump. |
| Human changes objective or constraints | All 3 skills. Major version bump. |
| JUDGMENT decision modifies a skill | skill-orchestrator + assembly. |
| Human changes amendment mode decision | assembly only. Minor version bump. |

---

*Agent Builder — Skills Index v2.5*
