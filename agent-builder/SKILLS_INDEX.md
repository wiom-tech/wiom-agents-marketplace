# Agent Builder — Skills Index
## Version 2.4

Routing contract per AGENT_STACK_FORMAT.md §5.2.

---

## Available Skills

| # | Skill | Folder | Type | Mandatory | Purpose |
|---|---|---|---|---|---|
| 1 | Elicitation | /elicitation/ | Domain | Yes | Collect and qualify Input, Instructions, Skills from human |
| 2 | Skill Orchestrator | /skill-orchestrator/ | Domain | Yes | Validate, convert, gap-check skills |
| 3 | Assembly | /assembly/ | Domain | Yes | Derive checks, produce agent folder + RUN.md + LOG.md |

---

## Execution Order

```
elicitation → skill-orchestrator → assembly
```

Sequential. All mandatory. Each skill's output feeds the next.

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

---

*Agent Builder — Skills Index v2.4*
