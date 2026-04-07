# PRD YAML Generator
## SKILL.md — Version 2.0

Conforms to AGENT_STACK_FORMAT.md §5.4 (9-section skill format).

---

## Purpose

Generate a structured `{service}-prd.yaml` from a WIOM OS document and produce a conformance report. The YAML is a machine-readable companion to the OS — it adds engineering sections the OS does not cover and never duplicates OS prose.

---

## When

Triggered when user asks to create a PRD, extract engineering sections from an OS, generate a YAML from an OS document, or convert an OS to machine-readable form. Ends when conformance check passes (YAML emitted) or fails after 3 attempts (conformance gap report emitted).

---

## Required Inputs

| Input | Why |
|---|---|
| OS file (LOCKED, versioned) | SOURCE_OF_TRUTH — all YAML content derives from this |
| yaml-prd-template.yaml v1.0 | Schema — defines all sections, field types, constraints |

---

## Framework

```
Step 1: Pre-flight
  → Read entire OS document end-to-end before generating anything.
  → Identify: service name, OS version, all states, transitions, invariants,
    parameters, events, dependencies, failure modes, and assumptions.
  → Confirm OS is LOCKED. Do not generate from a draft OS.
  → Set output path: docs/os/{service-name}-prd.yaml
    Naming convention:
      "Asset Custody OS"       → asset-custody-prd.yaml
      "Payment & Settlement OS" → payment-settlement-prd.yaml
  → Output: extracted OS inventory (all elements identified)

Step 2: Generate Header + Service Identity
  Header fields:
    service_id: csp-{service-name}-service (no -svc, -srv, -app suffix)
    version: "1.0"
    owner: {team}-team
    status: approved
    os_ref: {OS_filename_LOCKED}.md (must match exact OS filename)
    consistency_requirement: strong
  Service Identity fields:
    service_name: must equal service_id
    kubernetes_service_name: csp-{service-name}-service
    api_prefix: {service-short-name}
      (first path segment after / in REST endpoints, e.g. "payment" → /payment/...)
    package: io.wiom.csp.{service_snake}
      (Java root package for generated code — used by Pass 1 scaffold)
    port: 8080
    database: {service_snake}_db
      (PostgreSQL database name — used by Pass 2 migrations and application.yml)
    source_service_header: csp-{service-name}-service
      (sent in X-Source-Service header on all outbound HTTP calls and emitted events)
    openapi_spec: docs/api-specs/{service-name}-svc.yaml
  → Output: header + service_identity blocks

Step 3: Generate Interfaces
  Always include REST and CEF_EVENTS entries.
  Type values: REST | CEF_EVENTS | GRPC | GRAPHQL | WEBSOCKET
  Per entry: type, spec_ref, format, note
  → Output: interfaces block

Step 4: Generate Dependencies
  Extract from OS §6 (Wave & Dependencies) and §7 (Interface Contract).
  Per entry: service, type, direction, events (if event-based), data (if query-based),
             criticality, os_ref (when known), timing, note
  Type values: event-consumer | event-producer | query | http | auth-provider |
               datastore | scheduler | secrets-provider | config-consumer
  Direction values: upstream | downstream
  Always include: postgresql, aws-parameter-store, csp-auth-service, registry-service
  Include aws-eventbridge only if OS defines timer-driven transitions (per R3).
  → Output: dependencies block

Step 5: Generate State Machines [CONDITIONAL — include if service owns stateful entities]
  Extract from OS §4 (States & Transitions). One entry per state machine entity.
  Per state machine:
    entity: {aggregate}.{field}
    owner: {service}
    write_authority: exclusive
    consistency: strong
    initial_state, terminal_states
    states: every OS state
      (id: UPPER_SNAKE_CASE, terminal: true/false, description with behavioral metadata)
    transitions: every transition including timer-driven
      (from, to, trigger, guard with INV-XXX-NN reference, actor, reversible, note)
    invariant_ref: invariant IDs from OS §8
  → Output: state_machines block OR omit if no stateful entities

Step 6: Generate State Mapping Tables [CONDITIONAL — include if internal state
  is derived from an external authority's state]
  Name: {upstream_concept}_{local_entity}_mapping
  Every upstream posture/state value from OS must appear.
  Include behavioral annotations (credits, withdrawals, liability_auto_adjust, etc.)
  when OS defines them.
  If OS states a particular state "blocks X but not Y", capture that explicitly —
  this drives guard logic.
  → Output: state mapping block OR omit if no upstream-driven state mapping

Step 7: Generate Balance Model [CONDITIONAL — include if service manages
  financial balances or derived fields with formulas]
  Per balance: name, definition, mutability, stored (true/false)
  Reference SPR parameter IDs for configurable modes — do not copy default values
  (SPR owns them).
  Include gate/invariant rules that reference these balances.
  → Output: balance_model block OR omit if no multi-balance or derived-field model

Step 8: Generate Structural Assumptions [CONDITIONAL — include if OS §10 has
  assumptions with code_impact]
  Extract from OS §10 only those with code_impact (DB constraints, guard logic,
  scope boundaries). Skip pure business context assumptions.
  Per entry: id (ASM-{SVC}-NN), assumption (short label), code_impact
  Reference the OS assumption ID.
  → Output: structural_assumptions block OR omit if no code-relevant assumptions

Step 9: Generate Write-Back Prohibition [CONDITIONAL — include if OS explicitly
  prohibits writes to specific services]
  If OS §7.3 or equivalent defines write-back prohibitions:
  One entry per prohibited target service (service, reason).
  → Output: write_back_prohibition block OR omit if no write-back prohibition

Step 10: Generate Economic Application Matrix [CONDITIONAL — include if OS
  contains economic matrix or event-to-ledger mapping]
  If OS has economic application matrix or appendix showing trigger → ledger effects:
  One entry per economic trigger (trigger, source, wallet, liability, deposit, note).
  Use null for unaffected stores.
  Include CREDIT/DEBIT direction and overflow/cascade rules inline.
  → Output: economic_application_matrix block OR omit if no economic matrix

Step 11: Generate Guard Specifications
  Enumerate every guard referenced in events[], flows[], or state_machines[].transitions.
  Per entry:
    id: snake_case
    description: what the guard checks
    error_code: UPPER_SNAKE_CASE
      (value in structured error response {error_code, message, correlation_id})
    http_status:
      422 = business rule violation
      404 = missing entity
      409 = conflict or already-completed
      500 = infrastructure atomicity failure
    invariant_ref: INV-{SVC}-NN | null
    fm_ref: FM-{SVC}-NN | null (when a failure mode maps to this guard)
    response_fields: list of extra fields beyond standard triple | null
    note: OS ref, edge cases
  Guards return result object with error_code — never throw exceptions directly
  (per guards skill).
  → Output: guard_specifications block

Step 12: Generate Endpoint Permissions
  Map every REST endpoint to @CheckPermission action/resource pair and auth mechanism.
  Per entry:
    path: "METHOD /path/pattern"
    action: UPPER_SNAKE_CASE operation name | null for public endpoints
    resource: UPPER_SNAKE_CASE — convention: {LAST_PATH_SEGMENT}_{SERVICE_NAME} | null
    role: required caller role | null
    auth: "JWT (csp-auth-service)" | "dispatch-token" |
          "Bearer token from Parameter Store" | "none"
    note: endpoint purpose, guard references
  /events/**: auth is dispatch-token validated by InboundEventFilter.
    Include @CheckPermission per platform skill.
  /internal/task/**: auth is Bearer token from Parameter Store.
    Document the filter-based validation.
  /public/health: action, resource, role all null. Auth none.
    Listed in public-endpoints.yml.
  Cross-reference with security.authorization.roles to confirm role mapping consistency.
  → Output: endpoint_permissions block

Step 13: Generate Failure Modes
  Extract from OS §12 (Failure Modes & Rollback).
  Per entry:
    id: FM-{SVC}-NN
    failure: short label
    response: system behavior and recovery (include INV-ref)
    severity: high | medium | low
  → Output: failure_modes block

Step 14: Generate Rollback
  strategy: forward-correction
  schema_migration: backward-compatible
  rollback_trigger: condition from OS
  rollback_steps: ordered correction actions from OS
  data_migration.approach: flyway-versioned
  → Output: rollback block

Step 15: Generate Data Retention
  One entry per data entity.
  Per entry:
    entity: name
    retention_days: N
    archive_strategy: "cold-storage after N days" | null
    immutable: true | false
    reason: why retained (with INV-ref)
  → Output: data_retention block

Step 16: Generate Events — includes Enhanced Event Discovery (mandatory)

  Step 16a: Extract from structured sections
    → OS §7 Interface Contract tables
    → Appendix A event schemas
    → Dependency tables listing specific events

  Step 16b: Prose scan for missing events
    Timer/Schedule: scan for _LOCKED, window.*close, cycle.*end,
                    evaluation.*trigger, timer.*fire, schedule.*emit
    Health/Status: scan for _CONFIRMED, _UPDATED, health.*restore,
                   posture.*emit, verification.*status, health.*gate
    Operational references: scan for
      "emits.*event|fires.*event|triggers.*event|publishes.*event|sends.*signal"
    Common missed patterns:
      SETTLEMENT_LOCKED: monthly evaluation window closure
      ENFORCEMENT_POSTURE_UPDATED: health restoration after outages
      *_CYCLE_CONFIRMED: periodic health checks
      Timer-triggered batch operations

  Step 16c: Cross-reference validation
    Every prose-referenced event must appear in events[]
    Every events[] entry must have corresponding dependency
    Timer triggers require aws-eventbridge dependency and /internal/batch/ endpoints

  Per consumed event:
    id: EVENT_NAME (UPPER_SNAKE_CASE)
    direction: consumed
    version: "1"
    emitted_by: source service (kebab-case)
    delivery: at-least-once
    ordering_guarantee: per-partition
    partition_key: "{aggregateType}-{id_field}" (e.g. "csp-{csp_id}")
    schema_fields: one entry per field
      {type: uuid|string|int|bigint|boolean|timestamp|enum, required: true|false,
       values: [...] (if enum), note: "..."}
    idempotency_key: eventId
    state_mutation: description of state change caused

  Per emitted event:
    id: EVENT_NAME (UPPER_SNAKE_CASE)
    direction: emitted
    version: "1"
    entity: aggregate name
    event_class: FACT | COMMAND | INTERNAL
    delivery: at-least-once
    ordering_guarantee: per-partition
    partition_key: "{aggregateType}-{id_field}"
    retention_days: 365
    consumers: list of known consumer services
    schema_fields: one entry per field (same structure as consumed)

  → Output: events block

Step 17: Generate Idempotency
  One entry per event handler or batch operation.
  Per entry:
    operation: handle_{event_name_snake}
    key_field: eventId
    deduplication_window_ms: 604800000 (7 days default)
    safe_to_retry: true
    strategy: processed_events
    on_duplicate: return_200_silently | return_202_silently
  → Output: idempotency block

Step 18: Generate Concurrency
  Per entity:
    strategy: optimistic-locking | append-only
    version_field: version (if optimistic-locking) | null (for append-only)
    conflict_resolution: retry-with-backoff | none
    max_concurrent_writers: 1
    retry block (only for retry-with-backoff):
      max_attempts: 3
      backoff_base_ms: 50
      backoff_max_ms: 500
      exception: ObjectOptimisticLockingFailureException
  → Output: concurrency block

Step 19: Generate Security
  Authentication interfaces:
    /events/**: mechanism: CEF routed dispatch
      required_headers: [Authorization, X-Correlation-Id, X-Causation-Event-Id,
                         X-Source-Service, X-Schema-Version]
      validation: dispatch-token from registry-service
    /api/**: mechanism: JWT, issuer: csp-auth-service
    /internal/task/**: mechanism: Bearer token, token_source: AWS Parameter Store
    /public/health: mechanism: none
  Authorization:
    model: RBAC via Permit.io (csp-auth-service SDK)
    write_authority: {service} (exclusive)
    roles: extract from OS
  Secrets:
    manager: AWS Parameter Store
    rotation_days: 90
    secrets_list: extract from OS
  → Output: security block

Step 20: Generate Observability
  Extract from OS where available; otherwise derive from state machines and flows.
  metrics: snake_case names with _gauge/_count/_duration/_rate suffix as appropriate
  traces: trace span names
  alerts: alert rule names
  → Output: observability block

Step 21: Generate Capacity
  expected_rps: baseline, peak, peak_window
  scaling: strategy: horizontal, min_replicas: 2, max_replicas: 8
  db_connection_pool: min: 5, max: 30
  resource_limits: cpu: "1", memory: 1Gi, cpu_request: 250m, memory_request: 512Mi
  → Output: capacity block

Step 22: Generate Data Model
  Extract entity structure from state machines, events, and OS text.
  Per entity:
    name: snake_case
    owner: service
    write_authority: exclusive
    table: table_name (snake_case)
    primary_key: field name
    primary_key_type: uuid | composite
    invariants: [INV-XXX-NN]
    append_only: true | false
    fields: one entry per column
      (name, type, nullable, unique, default, enum, note)
    indexes: one entry per index
      (columns, unique, where for partial index, note)
  Mandatory fields per entity:
    Every entity with state mutations: version BIGINT NOT NULL DEFAULT 1
    Every entity: created_at, updated_at timestamps
    Domain aggregates + audit tables:
      correlation_id VARCHAR(255) nullable
      causation_id VARCHAR(255) nullable
  Always include: processed_events entity
  → Output: data_model block

Step 23: Generate Testing
  unit_tests:
    guards: all guard IDs from guard_specifications
    state_machine: valid_transitions list, invalid_transitions list
  repository_integration_tests:
    framework: "TestContainers + PostgreSQL"
    required_tests: list from OS
  full_integration_tests:
    framework: "@SpringBootTest + TestContainers"
    happy_path: list of scenarios
    failure_paths: list of failure scenarios
  load_tests: list of scenarios
  chaos_tests: list of scenarios
  → Output: testing block

Step 24: Generate Disaster Recovery
  rto_minutes: 30
  rpo_minutes: 5
  backup: frequency: continuous, method: "PostgreSQL WAL streaming + daily snapshots"
  → Output: disaster_recovery block

Step 25: Generate Operational Configuration
  jwt_identity: enabled if service reads caller identity from JWT
    claims: walletId (config-key: wallet-id-claim, purpose: "Resolve CSP wallet from JWT"),
            cspId (config-key: csp-id-claim, purpose: "Resolve CSP identity from JWT"),
            role (config-key: role-claim, purpose: "Resolve caller role from JWT")
  outbox: enabled if namastack-outbox-starter-jdbc is a dependency
    defaults: poll-interval-ms: 2000, batch-size: 10, max-retries: 5,
              initial-delay-ms: 1000, max-delay-ms: 60000, multiplier: 2.0
  optimistic_locking: enabled if any entity uses @Version
    defaults: max-retries: 3, backoff-initial: PT0.1S, backoff-max: PT5S,
              backoff-multiplier: 2.0
  service_specific_params: list any OS parameters needing YAML config keys
    Per entry: key "{service}.parameters.{param-name}", type, default,
               env_var (UPPER_SNAKE_CASE), purpose (include parameter ID if known)
  upi_config: enabled if service generates UPI deep links or payment intents
    If enabled: payee-vpa (env: UPI_PAYEE_VPA, default: "wiom@upi"),
                payee-name (env: UPI_PAYEE_NAME, default: "Wiom")
  Codegen reads this section to produce namastack:, {service}.app.identity:,
  optimistic-lock-*, aws.scheduler:, and service-specific param blocks in application.yml.
  → Output: operational_config block

Step 26: Generate Registry Event Mapping
  producers: one entry per non-INTERNAL emitted event
    event_id: must match events[] entry with direction: emitted
    event_type_name: PascalCase + Event suffix
      (e.g. DEPOSIT_BALANCE_UPDATED → DepositBalanceUpdatedEvent)
    schema_version: "1.0"
  subscriptions: one entry per consumed event
    event_id: must match events[] entry with direction: consumed
    event_type_name: PascalCase + Event suffix
    endpoint_path: kebab-case path of inbound event controller endpoint
    schema_version: "1.0"
  INTERNAL events (e.g. LIABILITY_AUTO_ADJUSTED) excluded from producers.
  Codegen uses this to produce registry-service.producers and .subscriptions
  blocks in application.yml.
  → Output: registry_event_mapping block

Step 27: Add Excluded Sections comment block to YAML header
  # Excluded (irrelevant to WIOM):
  #   - multi_tenancy
  #   - internationalisation (INR only)
  #   - cost_model
  #   - versioning_lifecycle
  #   - compliance_gdpr_pci
  → Output: excluded comment block in YAML header

Step 28: Run Phase C — OS Internal Coherence (read-only on LOCKED OS)
  Execute against source OS only. Do not edit LOCKED OS to pass a check.
  C1: Closed vocabulary — build state/status/enum set from OS §4 + appendix.
      Flag tokens in later sections not in that set (undefined alias or typo).
  C2: Cross-section contradiction — compare behavioral rules across amended sections.
      Flag incompatible mechanics unless one explicitly supersedes (version note).
  C3: Transition completeness — for each §4 state, verify how it is entered and exited.
      Flag orphan or unreachable states.
  C4: Dependency readiness — list Pending/TBD/provisional dependencies in §6/§7.
      OS_WARNING for documentation risk.
      OS_CRITICAL if core flows cannot be implemented without that contract.
  C5: Invariant operability — for invariants using vague terms, flag if neither OS
      nor YAML gives operationally testable definition.
  C6: Dual authority — where multiple upstreams affect same aggregate, flag if OS
      does not state precedence or single source of truth.
  C7: Parameter reference integrity — collect parameter IDs (e.g. P###) cited in prose.
      Flag any not defined in OS §5, appendix, or parameter tables.
  Phase C remediation: capture resolvable ambiguity in os_clarifications_needed: or
  engineering_assumptions: tail sections. Cite conflicting OS passages. Never invent
  OS facts. Never modify LOCKED OS.
  → Output: Phase C findings (OS_CRITICAL / OS_WARNING)

Step 29: Run Phase A — OS-to-YAML Conformance
  A1:  Every OS §4 state appears in state_machines[].states
  A2:  Every OS §4 transition appears in state_machines[].transitions
  A3:  Every OS §8 invariant ID cited somewhere in YAML
       (invariant_ref, guard, note, or invariants field)
  A4:  Every OS §6/§7 upstream dependency appears in dependencies
  A5:  Every OS §7+prose downstream signal appears in events (emitted)
  A6:  Every OS §7+prose consumed signal appears in events (consumed)
  A6a: Enhanced event discovery — prose scan for missing timer/health events
  A7:  Every OS §12 failure mode appears in failure_modes
  A8:  Every OS §5 parameter appears in capacity/config or is referenced
  A9:  No state/transition/event name invented that OS does not define
  A10: OS §4 posture/classification→state mapping → appears in posture_*_mapping
  A11: OS §4 named balances with formulas → appear in balance_model
  A12: OS §10 structural assumptions with code_impact → appear in structural_assumptions
  A13: OS §7.3 write-back prohibitions → appear in write_back_prohibition
  A14: OS economic matrix (Appendix) → appears in economic_application_matrix
  A15: Every guard in events[].guards or flows[].steps → entry in guard_specifications
       with error_code and http_status
  A16: Every REST endpoint in flows[].trigger or security → entry in endpoint_permissions
       with action/resource/role
  → Output: Phase A findings (CRITICAL / WARNING)

Step 30: Run Phase B — YAML Internal Consistency
  B1:  state_machines[].states IDs match data_model entities[].fields[state].enum exactly
  B2:  Every consumed event has matching idempotency entry
  B3:  Every consumed event's emitted_by appears in dependencies
  B4:  Every dependencies[].events[] entry exists in events[]
  B5:  Every next: reference in flows points to existing step.id in same flow
  B6:  Every entity in state_machines[].entity has entry in data_model.entities[]
  B7:  Every concurrency[] entity exists in data_model.entities[]
  B8:  terminal_states exactly equals set of states where terminal: true
  B9:  processed_events entity present in data_model.entities[]
  B10: Every non-append-only domain entity has correlation_id and causation_id fields
  B11: Every optimistic-locking entity has version field in data_model
  B12: Every posture_*_mapping local state value appears in state_machines[].states
  B13: Every economic_application_matrix source appears in dependencies[].service;
       every trigger with wallet/liability/deposit mutation has corresponding consumed event
  B14: Every guard_specifications[].fm_ref references existing failure_modes[].id
  B15: Every endpoint_permissions[].role appears in security.authorization.roles[].name
  B16: Every guard name in events[].guards has entry in guard_specifications[].id
  B17: Every registry_event_mapping.producers[].event_id appears in events[] as emitted
       and is NOT INTERNAL class
  B18: Every registry_event_mapping.subscriptions[].event_id appears in events[] as consumed
  B19: If namastack-outbox dependency → operational_config.outbox.enabled: true
  B20: If any optimistic-locking in concurrency[] →
       operational_config.optimistic_locking.enabled: true
  B21: If mechanism: JWT in security → operational_config.jwt_identity.enabled: true
  → Output: Phase B findings (CRITICAL / WARNING)

Step 31: Fix loop (up to 3 full C→A→B passes)
  Attempt 1: Collect all C*/A*/B* failures. Fix YAML. Re-run C→A→B.
  Attempt 2: If failures remain, fix and re-run C→A→B.
  Attempt 3: If failures remain, fix and re-run C→A→B.
  Fix priority: OS text > state_machines section > events section > data_model section
  Phase C items: fix via os_clarifications_needed or engineering_assumptions tail sections
  → Output: fixed YAML after each attempt

Step 32: Emit output
  IF all checks pass → emit docs/os/{service-name}-prd.yaml
  IF failures persist after 3 attempts → emit conformance gap report:
    docs/os/{service-name}-prd-conformance-gaps.md
    Structure:
      Header: service name, date, OS reference, YAML filename, attempts: 3
      OS Coherence Issues (Phase C): table of findings
        columns: Check ID, OS section, Description, Severity, Suggested resolution
        OS_CRITICAL: block codegen-ready status until user acknowledges or OS amended
        OS_WARNING: document; codegen may proceed only if assumptions explicit
                    and user accepts risk
      Unresolved YAML Issues (Phases A+B): table of findings
        columns: Check ID, Description, Expected, Actual, Severity
        CRITICAL: block codegen until resolved or explicitly accepted by user
        WARNING: produce incomplete but functional code; document before building
```

---

## Rules

```
R1:  IF mandatory section has no OS data → include section header with empty
     list and # TBD comment. Never omit a mandatory section.
R2:  IF OS title → service_id follows csp-{name}-service pattern
     (no -svc, -srv, or -app suffix).
R3:  IF OS defines timer-driven transitions → include aws-eventbridge in dependencies.
R4:  IF OS mentions "when X fires" or "X event triggers" but X not in events[]
     → Phase A6a fails.
R5:  IF state has no outgoing transitions → terminal: true.
R6:  IF service reads caller identity from JWT
     → operational_config.jwt_identity.enabled: true.
R7:  IF namastack-outbox-starter-jdbc is a dependency
     → operational_config.outbox.enabled: true.
R8:  IF any entity uses @Version
     → operational_config.optimistic_locking.enabled: true.
R9:  IF service generates UPI deep links or payment intents
     → operational_config.upi_config.enabled: true.
R10: IF emitted event has event_class: INTERNAL
     → exclude from registry_event_mapping.producers.
R11: IF conformance fails after 3 full C→A→B passes
     → emit conformance-gaps.md. Do not emit PRD YAML.
R12: IF Phase C gap is resolvable → capture in os_clarifications_needed or
     engineering_assumptions. Never edit LOCKED OS.
R13: IF flow handles an event → include processed_events idempotency check as
     an explicit step in that flow.
R14: IF OS describes timer-driven transitions → include EventBridge timer
     scheduling as an explicit step in the relevant flow.
R15: IF flow step has next: reference → target step.id must exist in same flow.
     Never dangle.
R16: Conformance check runs every time YAML is generated or regenerated.
     No exceptions. Order always: C → A → B.
```

---

## Output

```
docs/os/{service-name}-prd.yaml
OR (if conformance fails after 3 attempts):
docs/os/{service-name}-prd-conformance-gaps.md
```

---

## Fail Conditions

| Condition | Severity | Action |
|---|---|---|
| OS not LOCKED | S1 | STOP — do not generate |
| Mandatory YAML section missing from output | S1 | STOP |
| State/enum mismatch between state_machines and data_model (B1) | S1 | STOP |
| Missing entity, missing event, or flow graph break | S1 | STOP (CRITICAL) |
| Invented state/transition/event not in OS (A9) | S1 | STOP |
| OS_CRITICAL Phase C finding unresolved | S1 | STOP — block codegen-ready |
| Missing invariant references | S2 | FLAG (WARNING) |
| Missing traceability columns (correlation_id, causation_id) | S2 | FLAG (WARNING) |
| Incomplete capacity data | S2 | FLAG (WARNING) |
| OS_WARNING Phase C finding | S2 | FLAG — document in assumptions |

---

## Does Not Do

- Does not validate event names or parameters against ESR/SPR
  (ESR/SPR Validator does that)
- Does not duplicate OS prose in YAML
- Does not add per-step retry/timeout_ms/owner in flows
  (operational_config.outbox and operational_config.optimistic_locking own this)
- Does not run cross-PRD validation across multiple PRDs
- Does not edit the LOCKED OS file
- Does not invent OS states, transitions, events, or parameters
- Does not resolve Phase C OS ambiguity by modifying the OS
- Does not proceed to codegen-ready status with unresolved CRITICAL findings

---

## Traceability

| This Skill Enforces | Authority |
|---|---|
| YAML does not duplicate OS prose | AGENT_SPEC — Constraints |
| Every OS state/rule/logic block in YAML | AGENT_SPEC — M1 |
| Zero invented logic, rules, or entities | AGENT_SPEC — M4 |
| All REQUIRED sections present | AGENT_SPEC — M5 |
| CONDITIONAL sections only when OS warrants | AGENT_SPEC — M6 |
| Zero `<<...>>` placeholder strings in output | AGENT_SPEC — M7 |
| OS must be LOCKED before generation | AGENT_SPEC — K1 |

---

*PRD YAML Generator — SKILL.md v2.0*
