# YAML Generator — Skills Index
## Version 1.0

Routing contract per AGENT_STACK_FORMAT.md §5.2.

---

## Available Skills

| # | Skill | Folder | Type | Mandatory | Purpose |
|---|---|---|---|---|---|
| 1 | PRD YAML Generator | /yaml-generation/ | Domain | Yes | Generate a structured PRD YAML from a locked WIOM OS document |

---

## Execution Order

```
yaml-generation/
```

Sequential. Single skill. Self-contained.

---

## Conflict Resolution

| Conflict | Resolution |
|---|---|
| OS content is ambiguous | PAUSE — surface to human; do not interpret |
| OS has conflicting values | PAUSE — surface to human; do not choose |
| Phase C finding contradicts Phase A finding | Phase C takes precedence — OS is the authority |

---

## Re-Run Triggers

| Trigger | What Re-Runs |
|---|---|
| OS file updated (new locked version) | yaml-generation/ — full run |
| Conformance gap report produced | yaml-generation/ — after human resolves gaps |
| Human corrects a surfaced ambiguity | yaml-generation/ — resume from last valid step |

---

*YAML Generator — Skills Index v1.0*
