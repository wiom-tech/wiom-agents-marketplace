# Skill: PRD YAML Generator
## SKILL.md — Version 1.0

Conforms to AGENT_STACK_FORMAT.md §5.4 (9-section skill format).

---

## Purpose

Generate a structured `{service-name}-prd.yaml` from a locked WIOM OS (Operating Specification) document. The YAML is a machine-readable companion to the OS — it adds structured engineering sections for use in code generation. It never duplicates OS prose and never invents information not present in the OS.

---

## When

Triggered when: user asks to create a PRD, extract engineering sections from an OS, generate a YAML from an OS document, or convert a locked OS to machine-readable form. Also triggered when user says "generate YAML", "PRD from OS", "codegen spec", or references converting a locked specification into a machine-readable companion file. Input must be a LOCKED OS markdown file.

Ends when: YAML passes full conformance (zero CRITICAL/OS_CRITICAL findings after up to 3 C→A→B retry attempts), or conformance report is written and user acknowledgment is required before proceeding.

---

## Required Inputs

| Input | Why |
|---|---|
| Locked OS markdown file (filename contains "locked") | SOURCE_OF_TRUTH — all YAML content derived exclusively from this document |

---

## Framework

```
Step 1: Pre-flight — Read entire OS end-to-end. Identify: service name, OS version,
  all states, transitions, invariants, parameters, events, dependencies, failure
  modes, and assumptions. → Named output: OS element inventory

Step 2: Confirm OS is LOCKED — verify filename contains "locked".
  → Named output: lock confirmation (or K0b fires → ABORT if not locked)

Step 3: Generate Section 1 — Header:

  service_id: csp-{service-name}-service
  version: "1.0"
  owner: {team}-team
  status: approved
  os_ref: {OS_filename_LOCKED}.md
  consistency_requirement: strong
  # Excluded (irrelevant to WIOM):
  #   - multi_tenancy
  #   - internationalisation (INR only)
  #   - cost_model
  #   - versioning_lifecycle
  #   - compliance_gdpr_pci

  → Named output: header block

Step 4: Generate Section 1a — Service Identity:

  service_identity:
    service_name: csp-{service-name}-service
    kubernetes_service_name: csp-{service-name}-service
    api_prefix: {service-short-name}
    package: io.wiom.csp.{service_snake}
    port: 8080
    database: {service_snake}_db
    source_service_header: csp-{service-name}-service
    openapi_spec: docs/api-specs/{service-name}-svc.yaml

  → Named output: service_identity block

Step 5: Generate Section 2 — Interfaces:

  interfaces:
    - type: REST
      spec_ref: /{service-name}-svc.yaml
      format: OpenAPI 3.1
      note: "Brief description of API surface"
    - type: CEF_EVENTS
      format: CEF v2.0 routed
      note: "Brief description of event surface"

  → Named output: interfaces block

Step 6: Generate Section 3 — Dependencies. Extract from OS dependency, wave
  planning, and interface contract sections. For each dependency:

  dependencies:
    - service: {service-name}
      type: event-consumer | event-producer | query | http | auth-provider |
            datastore | scheduler | secrets-provider | config-consumer
      direction: upstream | downstream
      events: [EVENT_A, EVENT_B]          # if event-based
      data: [field_a, field_b]            # if query-based
      criticality: critical | high | medium | low
      os_ref: {Source_OS_LOCKED}.md       # when known
      timing: "When the dependency is exercised"
      note: "Clarifying context"

  Always include: postgresql, aws-parameter-store, csp-auth-service,
  registry-service. Include aws-eventbridge if timers exist.
  → Named output: dependencies block

Step 7: Generate Section 4 — State Machines. Extract from OS state/lifecycle
  sections. One entry per state machine entity:

  state_machines:
    - entity: {entity}.{field}
      owner: {service}
      write_authority: exclusive
      consistency: strong
      initial_state: STATE_A
      terminal_states: [STATE_X, STATE_Y]
      states:
        - { id: STATE_A, terminal: false, description: "..." }
      transitions:
        - from: STATE_A
          to: STATE_B
          trigger: "What causes this"
          guard: "Condition that must be true (INV-XXX-NN)"
          actor: system | csp | ops-team | {service-name}
          reversible: false
          note: "Optional clarification"
      invariant_ref: [INV-XXX-01, INV-XXX-02]

  → Named output: state_machines block

Step 8: Generate Section 5 — Upstream Signal Mapping (CONDITIONAL). If OS defines
  upstream signal to local state mapping, extract:

  upstream_signal_mapping:
    - upstream_signal: SIGNAL_VALUE_A
      local_state: LOCAL_STATE_A
      allowed_operations: [op_1, op_2]
      blocked_operations: [op_3]
      auto_adjustments: true | false
      note: "Optional behavioral annotation from OS"

  If no upstream-driven state mapping → OMIT section entirely.
  → Named output: upstream_signal_mapping block or section omitted

Step 9: Generate Section 6 — Derived-Field Model (CONDITIONAL). If OS defines
  named computed fields with explicit formulas, extract:

  derived_field_model:
    fields:
      - name: FIELD_NAME
        definition: "Human-readable definition"
        formula: "FIELD_A - FIELD_B"
        mutability: "Updated on every X mutation" | "Derived (computed, not stored)"
        stored: true | false
    configurable_modes:
      - parameter: {PARAMETER_ID}
        purpose: "What this mode controls"
        v1_default: VALUE
    gates:
      - rule: "condition expression; else REJECT"
        note: "What this gate enforces"

  If no derived-field model → OMIT section entirely.
  → Named output: derived_field_model block or section omitted

Step 10: Generate Section 7 — Structural Assumptions (CONDITIONAL). Extract from
  OS assumptions sections — only those with direct code impact (DB constraints,
  guard logic, scope boundaries):

  structural_assumptions:
    - id: ASM-{SVC}-NN
      assumption: "Description of the structural constraint"
      code_impact: "How this affects implementation"

  Skip pure business context assumptions. If no code-relevant assumptions →
  OMIT section entirely.
  → Named output: structural_assumptions block or section omitted

Step 11: Generate Section 8 — Write-Back Prohibition (CONDITIONAL). If OS
  prohibits writes to specific services:

  write_back_prohibition:
    - service: {service-name}
      reason: "Why writes are prohibited"

  One entry per prohibited target. If no write-back prohibition → OMIT section.
  → Named output: write_back_prohibition block or section omitted

Step 12: Generate Section 9 — Trigger-to-Mutation Matrix (CONDITIONAL). If OS
  contains a matrix, mapping table, or appendix showing triggers to data store
  mutations:

  trigger_mutation_matrix:
    stores: [store_a, store_b, store_c]
    entries:
      - trigger: "Human-readable trigger description"
        source: {source-service}
        mutations:
          store_a: "CREDIT | DEBIT | INCREMENT | DECREMENT | null"
          store_b: "Effect description or null"
          store_c: "Effect description or null"
        overflow_rules: "Cascade/overflow logic if any"
        note: "Optional clarification"

  Use null for unaffected stores. Include direction and overflow/cascade rules
  inline. If no trigger-to-mutation matrix → OMIT section.
  → Named output: trigger_mutation_matrix block or section omitted

Step 13: Generate Section 10 — Guard Specifications. Enumerate every guard
  referenced anywhere in events, flows, or state transitions:

  guard_specifications:
    - id: {guard_name}
      description: "What the guard checks"
      error_code: {UPPER_SNAKE_CASE}
      http_status: 422 | 404 | 409 | 500
      invariant_ref: INV-{SVC}-NN | null
      fm_ref: FM-{SVC}-NN
      response_fields: [field_a]
      note: "Optional clarification"

  → Named output: guard_specifications block

Step 14: Generate Section 11 — Endpoint Permissions. Map every REST endpoint to
  its @CheckPermission action/resource pair and auth mechanism. One entry per
  distinct endpoint (method + path):

  endpoint_permissions:
    - path: "GET /example/resource"
      action: VIEW_RESOURCE
      resource: RESOURCE_{SERVICE_UPPER}
      role: {ROLE_NAME}
      auth: "JWT (csp-auth-service)" | "dispatch-token" |
            "Bearer token from Parameter Store" | "none"
      note: "Optional clarification"

  Cross-reference with security.authorization.roles to confirm role mapping
  consistency.
  → Named output: endpoint_permissions block

Step 15: Generate Section 12 — Failure Modes. Extract from OS failure mode,
  rollback, and error handling sections:

  failure_modes:
    - id: FM-{SVC}-NN
      failure: "What breaks"
      response: "System behavior and recovery (INV-ref)"
      severity: high | medium | low

  → Named output: failure_modes block

Step 16: Generate Section 13 — Rollback:

  rollback:
    strategy: forward-correction
    schema_migration: backward-compatible
    rollback_trigger: "Condition"
    rollback_steps:
      - "Step 1"
    data_migration:
      approach: flyway-versioned

  → Named output: rollback block

Step 17: Generate Section 14 — Data Retention. One entry per data entity:

  data_retention:
    - entity: {entity_name}
      retention_days: N
      archive_strategy: "cold-storage after N days" | null
      immutable: true | false
      reason: "Why retained (INV-ref)"

  → Named output: data_retention block

Step 18: Generate Section 15 — Flows. Extract from OS decision flow, interface
  contract, and procedure sections. Each flow models a complete workflow from
  trigger to terminal:

  flows:
    - id: {snake_case_id}
      title: "Human-readable title"
      owner: {service}
      trigger: "What starts this flow"
      sla_ms: N
      steps:
        - id: {step_id}
          label: "What happens"
          type: start | process | end_success | end_error
          next: {next_step_id}
          # OR conditional branching:
          next:
            - condition: {condition_name}
              to: {step_id}

  → Named output: flows block

Step 19: Generate Section 16 — Events using Enhanced Event Discovery Methodology.
  Two groups: consumed (inbound) and emitted (outbound):

  events:
    - id: EVENT_NAME
      direction: consumed | emitted
      version: "1"
      emitted_by: {source}                 # consumed only
      entity: {entity}                     # emitted only
      event_class: FACT                    # emitted only
      delivery: at-least-once
      ordering_guarantee: per-partition
      partition_key: "{type}-{id_field}"
      retention_days: 365                  # emitted only
      consumers: [svc-a, svc-b]           # emitted only
      schema_fields:
        fieldName: { type: uuid|string|int|timestamp|boolean|enum,
                     required: true|false, values: [...], note: "..." }
      idempotency_key: eventId             # consumed only
      state_mutation: "description"        # consumed only

  Enhanced Event Discovery — execute all 4 sub-steps:

  19a: Extract from structured sections: OS interface contract tables, appendix
       event schemas, dependency tables listing specific events.

  19b: Prose scan for missing events using patterns:
       - Timer/Schedule: _LOCKED|window.*close|cycle.*end|evaluation.*trigger|
         timer.*fire|schedule.*emit|batch.*trigger
       - Health/Status: _CONFIRMED|_UPDATED|health.*restore|posture.*emit|
         verification.*status|health.*gate|status.*change
       - Operational refs: emits.*event|fires.*event|triggers.*event|
         publishes.*event|sends.*signal

  19c: Cross-reference validation:
       - Every event in prose must appear in events[]
       - Every event in events[] must have corresponding dependencies
       - Timer triggers require aws-eventbridge dependency and
         /internal/batch/ endpoints

  19d: Common missing event patterns check:
       - Monthly/Periodic triggers (settlement locks, evaluation triggers,
         cycle closures)
       - Health restoration (posture updates, system health signals)
       - Cycle confirmations (health cycle completions, evaluation completions)
       - Scheduler events (usually /internal/batch/ endpoints, not /events/)

  → Named output: events block (structured + prose-discovered)

Step 20: Generate Section 17 — Idempotency. One entry per event handler or batch
  operation:

  idempotency:
    - operation: handle_{event_name_snake}
      key_field: eventId
      deduplication_window_ms: 604800000
      safe_to_retry: true
      strategy: processed_events
      on_duplicate: return_200_silently | return_202_silently

  → Named output: idempotency block

Step 21: Generate Section 18 — Concurrency:

  concurrency:
    - entity: {entity}
      strategy: optimistic-locking | append-only
      version_field: version
      conflict_resolution: retry-with-backoff | none
      max_concurrent_writers: 1
      retry:
        max_attempts: 3
        backoff_base_ms: 50
        backoff_max_ms: 500
        exception: ObjectOptimisticLockingFailureException

  → Named output: concurrency block

Step 22: Generate Section 19 — Security:

  security:
    authentication:
      - interface: "/events/**"
        mechanism: CEF routed dispatch
        required_headers: [Authorization, X-Correlation-Id,
          X-Causation-Event-Id, X-Source-Service, X-Schema-Version]
        validation: dispatch-token from registry-service
      - interface: "/api/**"
        mechanism: JWT
        issuer: csp-auth-service
      - interface: "/internal/task/**"
        mechanism: Bearer token
        token_source: AWS Parameter Store
      - interface: "/public/health"
        mechanism: none
    authorization:
      model: RBAC via Permit.io (csp-auth-service SDK)
      write_authority: {service} (exclusive)
      roles: [...]
    secrets:
      manager: AWS Parameter Store
      rotation_days: 90
      secrets_list: [...]

  → Named output: security block

Step 23: Generate Section 20 — Observability. Extract from OS where available;
  otherwise derive from state machines and flows:

  observability:
    metrics: [...]
    traces: [...]
    alerts: [...]

  → Named output: observability block

Step 24: Generate Section 21 — Capacity:

  capacity:
    expected_rps: { baseline: N, peak: N, peak_window: "description" }
    scaling: { strategy: horizontal, min_replicas: 2, max_replicas: 8 }
    db_connection_pool: { min: 5, max: 30 }
    resource_limits: { cpu: "1", memory: 1Gi, cpu_request: 250m,
                       memory_request: 512Mi }

  → Named output: capacity block

Step 25: Generate Section 22 — Data Model. Extract entity structure from state
  machines, events, and OS text. Always include processed_events entity:

  data_model:
    entities:
      - name: {entity}
        owner: {service}
        write_authority: exclusive
        table: {table_name}
        primary_key: {field}
        primary_key_type: uuid | composite
        invariants: [INV-XXX-01]
        append_only: true | false
        fields:
          - { name: field, type: uuid|varchar|int|bigint|boolean|
              "timestamp with time zone"|"varchar[]", nullable: false,
              default: 1, enum: [...], unique: true, note: "..." }
        indexes:
          - { columns: [col_a, col_b], unique: true,
              where: "partial index condition", note: "..." }

  → Named output: data_model block

Step 26: Generate Section 23 — Testing:

  testing:
    unit_tests:
      guards: [...]
      state_machine:
        valid_transitions: [...]
        invalid_transitions: [...]
    repository_integration_tests:
      framework: "TestContainers + PostgreSQL"
      required_tests: [...]
    full_integration_tests:
      framework: "@SpringBootTest + TestContainers"
      happy_path: [...]
      failure_paths: [...]
    load_tests: [...]
    chaos_tests: [...]

  → Named output: testing block

Step 27: Generate Section 24 — Disaster Recovery:

  disaster_recovery:
    rto_minutes: 30
    rpo_minutes: 5
    backup: { frequency: continuous,
              method: "PostgreSQL WAL streaming + daily snapshots" }

  → Named output: disaster_recovery block

Step 28: Generate Section 25 — Operational Configuration:

  operational_config:
    jwt_identity:
      enabled: true | false
      claims:
        - name: walletId
          config_key: wallet-id-claim
          purpose: "Resolve CSP wallet from JWT"
        - name: cspId
          config_key: csp-id-claim
          purpose: "Resolve CSP identity from JWT"
        - name: role
          config_key: role-claim
          purpose: "Resolve caller role from JWT"
    outbox:
      enabled: true | false
      defaults:
        poll-interval-ms: 2000
        batch-size: 10
        max-retries: 5
        initial-delay-ms: 1000
        max-delay-ms: 60000
        multiplier: 2.0
    optimistic_locking:
      enabled: true | false
      defaults:
        max-retries: 3
        backoff-initial: PT0.1S
        backoff-max: PT5S
        backoff-multiplier: 2.0
    service_specific_params:
      - key: "{service}.parameters.{param-name}"
        type: "Duration | int | String | List"
        default: "value"
        env_var: "ENV_VAR_NAME"
        purpose: "What this parameter controls"
    upi_config:
      enabled: true | false
      fields:
        - key: payee-vpa
          env_var: UPI_PAYEE_VPA
          default: "wiom@upi"
        - key: payee-name
          env_var: UPI_PAYEE_NAME
          default: "Wiom"

  → Named output: operational_config block

Step 29: Generate Section 26 — Registry Event Mapping. Mechanically derivable
  from events[]:

  registry_event_mapping:
    producers:
      - event_id: EVENT_NAME
        event_type_name: EventNameEvent
        schema_version: "1.0"
    subscriptions:
      - event_id: EVENT_NAME
        event_type_name: EventNameEvent
        endpoint_path: /events/event-name
        schema_version: "1.0"

  → Named output: registry_event_mapping block

Step 30: Run Phase C — OS Internal Coherence (read-only on LOCKED OS). 7 checks:
  C1: Closed vocabulary — build set of defined states/enums from OS; flag any
      token in later sections not in that set (undefined alias or typo).
  C2: Cross-section contradiction — compare behavioral rules across sections;
      flag incompatible mechanics unless one explicitly supersedes the other.
  C3: Transition completeness — verify each declared state has defined entry/exit;
      flag orphan or unreachable states.
  C4: Dependency readiness — flag OS_WARNING for Pending/TBD dependencies;
      flag OS_CRITICAL if core flows cannot be implemented without that contract.
  C5: Invariant operability — flag invariants with vague terms if no operational
      definition testable in code exists in OS or YAML flows/capacity.
  C6: Dual authority — flag if multiple upstreams affect same aggregate without
      OS stating precedence or single source of truth.
  C7: Parameter reference integrity — collect parameter IDs from prose; flag any
      not defined in OS parameter sections or appendix.

  Document Phase C gaps as os_clarifications_needed or engineering_assumptions
  tail sections in YAML. NEVER modify LOCKED OS.
  → Named output: Phase C findings list (C1–C7)

Step 31: Run Phase A — OS-to-YAML Conformance. 16 checks:
  A1:  Every state from OS in state_machines[].states.
  A2:  Every transition from OS in state_machines[].transitions.
  A3:  Every invariant ID from OS cited in YAML.
  A4:  Every upstream dependency from OS in dependencies.
  A5:  Every downstream signal from OS in events (emitted).
  A6:  Every consumed signal from OS in events (consumed).
  A6a: Enhanced event discovery prose scan for missing timer/health/lifecycle
       events.
  A7:  Every failure mode from OS in failure_modes.
  A8:  Every parameter from OS in capacity/config or referenced.
  A9:  No invented state, transition, or event name.
  A10: Upstream signal mapping present if OS defines it.
  A11: Derived-field model present if OS defines it.
  A12: Structural assumptions present if OS defines them.
  A13: Write-back prohibition present if OS defines it.
  A14: Trigger-to-mutation matrix present if OS has it.
  A15: Every guard name referenced anywhere has guard_specifications entry with
       error_code and http_status.
  A16: Every REST endpoint in flows or security has endpoint_permissions entry.
  → Named output: Phase A findings list (A1–A16)

Step 32: Run Phase B — YAML Internal Cross-Section Consistency. 21 checks:
  B1:  State enum alignment — state_machines[].states must match
       data_model.entities[].fields[state].enum exactly.
  B2:  Event-to-handler alignment — every consumed event in events[] has entry
       in idempotency[].
  B3:  Event-to-dependency alignment — every consumed event's emitted_by in
       dependencies[].
  B4:  Dependency-to-event alignment — every event in dependencies[].events
       exists in events[].
  B5:  Flow step connectivity — every next reference points to existing step.id.
  B6:  Data model completeness — every state_machines[].entity in
       data_model.entities[].
  B7:  Concurrency-to-entity alignment — every concurrency[] entity in
       data_model.entities[].
  B8:  Terminal state consistency — terminal_states[] equals set of states where
       terminal: true.
  B9:  processed_events entity present in data_model.entities[].
  B10: Traceability columns — every non-append-only domain entity has
       correlation_id and causation_id fields.
  B11: Version column — every optimistic-locking entity has version field in
       data_model.
  B12: Upstream signal mapping alignment — every local_state value in
       state_machines[].states.
  B13: Trigger-mutation matrix alignment — every source in
       dependencies[].service; every trigger with mutation has corresponding
       consumed event.
  B14: Guard-to-failure-mode alignment — every guard_specifications[].fm_ref
       references existing failure_modes[].id.
  B15: Endpoint-to-security alignment — every endpoint_permissions[].role in
       security.authorization.roles[].
  B16: Guard completeness — every guard name in events[].guards has entry in
       guard_specifications[].
  B17: Registry producers alignment — every producers[].event_id in events[]
       (direction: emitted, not event_class: INTERNAL).
  B18: Registry subscriptions alignment — every subscriptions[].event_id in
       events[] (direction: consumed).
  B19: Outbox config — if namastack-outbox dependency exists,
       operational_config.outbox.enabled: true.
  B20: Optimistic lock config — if any concurrency[].strategy: optimistic-locking,
       operational_config.optimistic_locking.enabled: true.
  B21: JWT identity config — if security.authentication[] includes mechanism: JWT,
       operational_config.jwt_identity.enabled: true.
  → Named output: Phase B findings list (B1–B21)

Step 33: Execute retry-and-fix loop:
  Attempt 1: Fix YAML per all C, A, B findings. Re-run full C → A → B.
  Attempt 2: If failures remain, fix and re-run full C → A → B.
  Attempt 3: If failures remain, fix and re-run full C → A → B.
  If failures persist after 3 attempts: STOP — write conformance gap report.
  Require user acknowledgment before proceeding.
  → Named output: corrected YAML or conformance gap report

Step 34: Run cross-PRD validator after single-PRD conformance passes:
  Execute: python docs/os/validate-prd-cross-refs.py --dir docs/os/
  Checks: event schema alignment, dependency symmetry, partition key agreement,
  consumer listings, schema version consistency across all PRD YAMLs.
  CRITICAL findings block code generation. WARNING findings do not block.
  → Named output: cross-PRD validation result
```

---

## Rules

```
R1:  IF OS filename does not contain "locked" → ABORT (K0b fires)
R2:  IF mandatory YAML section has no data from OS → include section header
     with empty list and # TBD comment; NEVER omit mandatory section
R3:  IF section is CONDITIONAL and OS does not provide relevant data →
     OMIT section entirely
R4:  IF service_id is generated → MUST follow csp-{service-name}-service
     pattern; no -svc, -srv, -app suffixes
R5:  IF os_ref is generated → MUST match exact OS filename
R6:  IF service_name is generated → MUST match Header service_id exactly
R7:  IF api_prefix is generated → MUST be first path segment after / in REST
     endpoints
R8:  IF dependencies block is generated → MUST include postgresql,
     aws-parameter-store, csp-auth-service, registry-service
R9:  IF OS contains timer-driven events or transitions → MUST include
     aws-eventbridge in dependencies
R10: IF state machine is generated → every state listed in OS MUST appear in
     states[]
R11: IF state machine is generated → every transition (including timer-driven)
     MUST appear in transitions[]
R12: IF state machine guard references an invariant → guard field MUST cite
     invariant ID in INV-XXX-NN format
R13: IF state has no outgoing transitions → terminal: true
R14: IF OS defines behavioral metadata for a state (counters, fees, timers) →
     include description field with that metadata
R15: IF upstream_signal_mapping is generated → every upstream signal value from
     OS MUST appear in mapping
R16: IF OS states a signal "blocks X but not Y" → capture allowed_operations
     and blocked_operations explicitly
R17: IF derived_field_model has configurable modes referencing SPR parameters →
     reference SPR parameter IDs only; NEVER copy default values (SPR owns them)
R18: IF OS assumption has no code_impact → SKIP; do not include in
     structural_assumptions
R19: IF error_code is generated → MUST be UPPER_SNAKE_CASE
R20: IF http_status is assigned → 422 for business rule violations; 404 for
     missing entities; 409 for conflict/already-completed; 500 for
     infrastructure atomicity failures
R21: IF guard_specifications entry has fm_ref → fm_ref MUST reference an
     existing failure_modes[].id
R22: IF guard fails at runtime → MUST return result object with error_code;
     NEVER throw exceptions directly
R23: IF endpoint_permissions entry is generated → one entry per distinct
     endpoint (method + path)
R24: IF action is generated → MUST be operation name in UPPER_SNAKE_CASE
R25: IF resource is generated → MUST follow {LAST_PATH_SEGMENT}_{SERVICE_NAME}
     in UPPER_SNAKE_CASE
R26: IF endpoint is /events/** → auth is dispatch-token validated by
     InboundEventFilter; still include @CheckPermission
R27: IF endpoint is /internal/task/** → auth is Bearer token from Parameter
     Store; document filter-based validation
R28: IF endpoint is /public/health → action, resource, and role are null; auth
     is none; listed in public-endpoints.yml
R29: IF flow is generated → MUST have exactly one start step
R30: IF flow is generated → MUST have at least one terminal step (end_success
     or end_error)
R31: IF flow is an event-handling flow → idempotency check (processed_events)
     MUST be an explicit step
R32: IF OS describes timer-driven transitions → EventBridge timer scheduling
     MUST be an explicit step in the relevant flow
R33: IF flow step includes retry, timeout_ms, or owner fields → REMOVE; use
     operational_config.outbox and operational_config.optimistic_locking instead
R34: IF event partition_key is generated → MUST follow {aggregateType}-{id}
     convention (e.g. netbox-{device_id}, csp-{csp_id})
R35: IF OS mentions "when X fires" or "X event triggers" but X not in events[]
     → Phase A6a MUST fail
R36: IF entity has state mutations → MUST have version BIGINT NOT NULL DEFAULT 1
R37: IF entity is generated → MUST have created_at and updated_at timestamps
R38: IF entity is a domain aggregate or audit table → MUST have correlation_id
     VARCHAR(255) and causation_id VARCHAR(255) (both nullable)
R39: IF data_model is generated → MUST include processed_events entity
R40: IF service reads caller identity from JWT → jwt_identity.enabled: true;
     claims MUST be externalized via @ConfigurationProperties; never hardcoded
R41: IF namastack-outbox-starter-jdbc is a dependency → outbox.enabled: true
R42: IF any entity uses @Version → optimistic_locking.enabled: true; all four
     retry parameters MUST be generated
R43: IF service generates UPI deep links or payment intents →
     upi_config.enabled: true
R44: IF registry_event_mapping producer is generated → event_type_name MUST be
     PascalCase + Event suffix (DEPOSIT_BALANCE_UPDATED →
     DepositBalanceUpdatedEvent)
R45: IF event has event_class: INTERNAL → MUST NOT appear in
     registry_event_mapping.producers
R46: IF registry subscription is generated → endpoint_path MUST be kebab-case
     path of inbound event controller endpoint
R47: IF YAML is generated → conformance check MUST run immediately; order
     always C → A → B; no exceptions
R48: IF Phase C gap is resolvable within YAML → document in
     os_clarifications_needed or engineering_assumptions tail sections;
     NEVER modify LOCKED OS
R49: IF conformance fails after 3 full C → A → B attempts → STOP; write
     conformance gap report; require user acknowledgment before proceeding
R50: IF conformance report has OS_CRITICAL findings → do NOT treat PRD as
     codegen-ready until user acknowledges or LOCKED OS is amended
R51: IF Phase A or B fails within 3 attempts → fix YAML and re-run FULL
     C → A → B sequence (not just failing phase)
R52: IF OS content is ambiguous → PAUSE; surface to human; do not interpret
     or complete
R53: IF OS has conflicting values → PAUSE; surface to human; do not choose
R54: IF any YAML content is not traceable to OS → ABORT (K0d); never invent
```

---

## Output

**Primary:** `docs/os/{service-name}-prd.yaml` — single YAML file. Sections 1–26 (all mandatory) plus applicable conditional sections (5–9). Excluded sections comment block in header.

**Secondary (on conformance failure after 3 attempts):** `docs/os/{service-name}-prd-conformance-gaps.md`:

```
# PRD YAML Conformance Gaps — {Service Name}
Generated: {date}
OS Reference: {os_filename}
PRD YAML: {yaml_filename}
Attempts: 3 (did not converge)

## OS Coherence Issues (LOCKED OS)
Phase C findings — defects or ambiguities in the OS text itself.

| # | Check ID | OS section | Description | Severity | Suggested resolution |

- OS_CRITICAL: Block codegen-ready status until user acknowledges or OS amended.
- OS_WARNING: Document; codegen may proceed only if assumptions explicit.

## Unresolved YAML Issues (Phases A & B)
| # | Check ID | Description | Expected | Actual | Severity |

## Notes
- CRITICAL / OS_CRITICAL block code generation until resolved or accepted.
- WARNING / OS_WARNING may produce incomplete code; document before building.
```

---

## Fail Conditions

| Condition | Severity | Action |
|---|---|---|
| OS filename does not contain "locked" | S1 | STOP — K0b fires |
| Phase A or B CRITICAL finding after 3 C→A→B attempts | S1 | STOP — write conformance report; require user acknowledgment |
| Phase C OS_CRITICAL finding after 3 C→A→B attempts | S1 | STOP — write conformance report; block codegen-ready until user acknowledges or OS amended |
| YAML content not traceable to OS | S1 | STOP — K0d fires |
| OS content is ambiguous | S1 | PAUSE — surface to human; do not interpret |
| OS has conflicting values | S1 | PAUSE — surface to human; do not choose |
| Phase A or B WARNING finding | S2 | FLAG — document; codegen may proceed with explicit assumptions |
| Phase C OS_WARNING finding | S2 | FLAG — document in report or YAML assumptions |

*Severity mapping: CRITICAL/OS_CRITICAL → S1. WARNING/OS_WARNING → S2.*

---

## Does Not Do

- Does not invent information not present in the OS
- Does not interpret ambiguous OS content — surfaces to human
- Does not choose between conflicting OS values — surfaces to human
- Does not modify the LOCKED OS file (not even to resolve Phase C findings)
- Does not proceed to codegen when CRITICAL/OS_CRITICAL findings remain unresolved
- Does not skip the conformance check — C → A → B mandatory on every generation
- Does not run only a subset of conformance phases — always full C → A → B
- Does not copy OS prose into YAML — generates structured YAML from OS content only
- Does not duplicate OS prose (YAML is a companion, not a copy)
- Does not include multi_tenancy, internationalisation, cost_model, versioning_lifecycle, or compliance_gdpr_pci sections
- Does not include per-step retry, timeout_ms, or owner fields in flows (operational_config owns these)
- Does not copy default values for SPR parameter IDs (SPR owns defaults)
- Does not emit a codegen-ready PRD without passing or documenting conformance

---

## Traceability

| This Skill Enforces | Authority |
|---|---|
| OS is sole source of truth — no invented content | AGENT_STACK_FORMAT.md §4.4 K0d |
| OS lock check before generation | AGENT_STACK_FORMAT.md §4.4 K0b |
| Ambiguous/conflicting OS content surfaced to human | AGENT_STACK_FORMAT.md §7.1 #1, §7.5 |
| No placeholders where JUDGMENT required | AGENT_STACK_FORMAT.md §7.6 |
| service_id pattern: csp-{service-name}-service | WIOM Platform Naming Convention |
| package pattern: io.wiom.csp.{service_snake} | WIOM Platform Naming Convention |
| Conformance check C→A→B mandatory on every generation | PRD YAML Generator Conformance Spec (this skill §Framework Steps 30–33) |
| Codegen blocked on unresolved CRITICAL/OS_CRITICAL findings | PRD YAML Generator Conformance Spec (this skill §Rules R49, R50) |
| processed_events entity mandatory | WIOM Event Idempotency Standard |
| correlation_id/causation_id mandatory on domain aggregates | WIOM Distributed Tracing Standard |
| version field mandatory on state-mutating entities | WIOM Concurrency Standard |
| outbox.enabled when namastack-outbox-starter-jdbc present | WIOM Namastack Standard |
| No per-step retry/timeout_ms/owner in flows | WIOM Namastack Outbox Routing Standard |
| SPR owns parameter defaults | WIOM Parameter Management Standard |

---

*PRD YAML Generator — SKILL.md v1.0*
