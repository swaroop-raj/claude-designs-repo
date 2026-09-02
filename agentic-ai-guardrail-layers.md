# 7 Guardrail Layers Every Agentic AI Stack Needs

> Reference extracted from [Antrixsh Gupta's LinkedIn post](https://lnkd.in/p/dW7FBjXB) and accompanying infographic.
> Captured by `agent:brain` on 2026-09-02.

---

## Why This Matters

Most AI agents are secured at the model. Very few are secured across the entire execution stack. That is why so many enterprise AI projects fail security reviews before they reach production.

The model is only one component. Your real attack surface spans **prompts, memory, retrieval, tools, runtime, and outputs**.

> "Enterprise AI security is no longer about protecting a single LLM. It is about governing every decision an autonomous agent can make across its entire execution lifecycle."
> -- Antrixsh Gupta

---

## Architecture Overview

```
                    +---------------------+
         User ---->|  1. Input Guardrails |
                    +--------+------------+
                             |
                             v
                    +---------------------+
                    |  2. Prompt Guardrails|
                    +--------+------------+
                             |
                             v
                    +---------------------+
                    |  3. Memory Guardrails|
                    +--------+------------+
                             |
                             v
                    +-------------------------+
                    |  4. Retrieval Guardrails |
                    +--------+----------------+
                             |
                             v
                    +----------------------+
                    |  5. Tool Guardrails   |
                    +--------+-------------+
                             |
                             v
                    +------------------------+
                    |  6. Runtime Guardrails  |
                    +--------+---------------+
                             |
                             v
                    +------------------------+
                    |  7. Output Guardrails   |
                    +--------+---------------+
                             |
                             v
                          Response
```

---

## Summary Table

| # | Guardrail Layer | What It Protects Against | Proposed Config File |
|---|---|---|---|
| 1 | **Input Guardrails** | Prompt injection, malicious payloads, schema abuse, unsafe inputs | `guardrails/input_guardrails.yaml` |
| 2 | **Prompt Guardrails** | System prompt exposure, jailbreaks, role boundary violations | `guardrails/prompt_guardrails.yaml` |
| 3 | **Memory Guardrails** | Session leakage, sensitive memory access, retention violations | `guardrails/memory_guardrails.yaml` |
| 4 | **Retrieval Guardrails** | Untrusted RAG sources, metadata poisoning, ungrounded context | `guardrails/retrieval_guardrails.yaml` |
| 5 | **Tool Guardrails** | Over-privileged tool access, unauthorized actions, missing approval gates | `guardrails/tool_guardrails.yaml` |
| 6 | **Runtime Guardrails** | Anomalies, infinite loops, latency spikes, concurrency issues | `guardrails/runtime_guardrails.yaml` |
| 7 | **Output Guardrails** | PII leakage, hallucinations, toxic content, policy violations | `guardrails/output_guardrails.yaml` |

---

## Proposed Directory Structure

```
guardrails/
├── input_guardrails.yaml
├── prompt_guardrails.yaml
├── memory_guardrails.yaml
├── retrieval_guardrails.yaml
├── tool_guardrails.yaml
├── runtime_guardrails.yaml
├── output_guardrails.yaml
└── guardrails_config.yaml          # master config that imports all layers
```

---

## 1. Input Guardrails

**Purpose:** Block prompt injection, malicious payloads, schema abuse, and unsafe inputs before they reach the model.

### Sub-Controls (from infographic)

| Control | Description |
|---|---|
| Language & injection screening | Detect and reject known injection patterns, adversarial Unicode, invisible characters |
| Size and rate limits | Cap input token count and request frequency per user/session |
| Schema and MIME checks | Validate input structure, reject unexpected content types |
| Content sanitization | Strip or escape HTML, scripts, and control characters |
| Input validation | Enforce expected data types, ranges, and formats |
| Malware and file scanning | Scan uploaded files for malware before processing |

### Sample Template: `guardrails/input_guardrails.yaml`

```yaml
input_guardrails:
  enabled: true
  version: "1.0.0"

  injection_screening:
    enabled: true
    patterns:
      - name: "prompt_injection_basic"
        regex: "(?i)(ignore previous|disregard above|forget your instructions|you are now)"
        action: "block"
        severity: "critical"
      - name: "invisible_unicode"
        regex: "[\\u200B-\\u200F\\u2028-\\u202F\\uFEFF]"
        action: "strip"
        severity: "medium"
    on_match: "reject_with_message"
    log_level: "warn"

  size_and_rate_limits:
    max_input_tokens: 4096
    max_request_size_bytes: 1048576  # 1 MB
    rate_limit:
      requests_per_minute: 30
      requests_per_hour: 500
      per: "user"  # user | session | api_key
    on_exceed: "reject_429"

  schema_validation:
    enabled: true
    allowed_mime_types:
      - "text/plain"
      - "application/json"
      - "image/png"
      - "image/jpeg"
      - "application/pdf"
    reject_unknown_fields: true
    max_nesting_depth: 5

  content_sanitization:
    strip_html: true
    strip_scripts: true
    strip_control_characters: true
    normalize_unicode: true
    max_consecutive_whitespace: 3

  input_validation:
    enforce_types: true
    required_fields: ["message"]
    max_field_length:
      message: 10000
      metadata: 2000

  file_scanning:
    enabled: true
    scanner: "clamav"  # clamav | custom_endpoint
    max_file_size_mb: 25
    allowed_extensions: [".pdf", ".png", ".jpg", ".txt", ".csv"]
    on_threat: "quarantine_and_alert"
```

---

## 2. Prompt Guardrails

**Purpose:** Protect system prompts, enforce role boundaries, prevent jailbreaks, and preserve instruction integrity.

### Sub-Controls (from infographic)

| Control | Description |
|---|---|
| Hidden prompt protection | Prevent system prompt extraction via adversarial queries |
| Jailbreak detection | Identify and block known jailbreak techniques (DAN, roleplay escapes) |
| Instruction locking | Ensure system instructions cannot be overridden by user input |
| Context boundaries | Enforce separation between system context and user context |
| System prompt isolation | Prevent user messages from modifying system-level instructions |
| Role separation | Maintain strict boundaries between system, user, and assistant roles |

### Sample Template: `guardrails/prompt_guardrails.yaml`

```yaml
prompt_guardrails:
  enabled: true
  version: "1.0.0"

  hidden_prompt_protection:
    enabled: true
    block_extraction_attempts: true
    detection_patterns:
      - "(?i)(show|reveal|print|repeat|display).*(system prompt|instructions|system message)"
      - "(?i)what (are|were) your (instructions|rules|system)"
    response_on_detect: "I can't share my system instructions."
    log_level: "warn"

  jailbreak_detection:
    enabled: true
    classifier: "rules"  # rules | ml_classifier | hybrid
    known_patterns:
      - name: "dan_prompt"
        pattern: "(?i)(DAN|do anything now|jailbreak|bypass)"
        action: "block"
      - name: "roleplay_escape"
        pattern: "(?i)(pretend you are|act as if you have no|imagine you are free)"
        action: "block"
      - name: "encoding_bypass"
        pattern: "(?i)(base64|rot13|hex encode).*(instructions|system)"
        action: "block"
    on_detect: "reject_and_log"
    escalate_after: 3  # consecutive attempts trigger escalation

  instruction_locking:
    enabled: true
    immutable_system_sections:
      - "role_definition"
      - "safety_rules"
      - "output_constraints"
    override_prevention: true

  context_boundaries:
    system_user_separator: true
    prevent_cross_context_reference: true
    max_user_context_tokens: 8192

  system_prompt_isolation:
    enabled: true
    hash_verification: true  # verify system prompt hasn't been tampered with
    prompt_hash_algorithm: "sha256"

  role_separation:
    enforce_roles: ["system", "user", "assistant", "tool"]
    reject_unknown_roles: true
    prevent_role_spoofing: true
```

---

## 3. Memory Guardrails

**Purpose:** Control session isolation, sensitive memory access, retention policies, and secure long-term memory.

### Sub-Controls (from infographic)

| Control | Description |
|---|---|
| Memory retention rules | Define what gets stored, for how long, and under what conditions |
| Memory expiry | Auto-expire stale or time-limited memory entries |
| Recall filtering | Filter what memory is surfaced during retrieval based on context |
| Sensitive data blocking | Prevent PII, credentials, or secrets from being persisted in memory |
| Session separation | Isolate memory across sessions, users, and tenants |
| Write controls | Restrict what the agent can write to long-term memory |

### Sample Template: `guardrails/memory_guardrails.yaml`

```yaml
memory_guardrails:
  enabled: true
  version: "1.0.0"

  retention_rules:
    default_ttl_days: 90
    max_entries_per_user: 1000
    max_entry_size_bytes: 10240
    categories:
      - name: "conversation_context"
        ttl_days: 7
        auto_archive: true
      - name: "user_preferences"
        ttl_days: 365
        auto_archive: false
      - name: "task_state"
        ttl_days: 30
        auto_archive: true

  memory_expiry:
    enabled: true
    check_interval_hours: 24
    soft_delete: true  # mark as expired vs hard delete
    grace_period_days: 7
    on_expire: "archive"  # archive | delete | notify_and_archive

  recall_filtering:
    enabled: true
    max_recall_entries: 20
    relevance_threshold: 0.7
    recency_weight: 0.3
    exclude_categories: ["system_internal", "debug"]

  sensitive_data_blocking:
    enabled: true
    scan_before_write: true
    blocked_patterns:
      - name: "credit_card"
        regex: "\\b\\d{4}[- ]?\\d{4}[- ]?\\d{4}[- ]?\\d{4}\\b"
      - name: "ssn"
        regex: "\\b\\d{3}-\\d{2}-\\d{4}\\b"
      - name: "api_key"
        regex: "(?i)(api[_-]?key|secret|token|password)\\s*[:=]\\s*\\S+"
      - name: "email_pii"
        regex: "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}"
    on_detect: "redact_and_warn"

  session_separation:
    isolate_by: ["user_id", "session_id"]
    cross_session_access: false
    cross_user_access: false
    tenant_isolation: true

  write_controls:
    require_explicit_consent: true
    max_writes_per_session: 50
    prohibited_write_targets: ["system_config", "other_user_memory"]
    audit_all_writes: true
```

---

## 4. Retrieval Guardrails

**Purpose:** Validate RAG context through source trust scoring, metadata filtering, grounding checks, and chunk validation.

### Sub-Controls (from infographic)

| Control | Description |
|---|---|
| Source filtering | Only retrieve from approved, vetted data sources |
| Trust scoring | Score retrieved documents by source authority and freshness |
| Metadata filtering | Enforce metadata constraints (date range, author, classification) |
| Freshness checks | Reject or downrank stale content beyond a defined threshold |
| Chunk validation | Validate chunk integrity, completeness, and coherence |
| Grounding rules | Ensure responses are grounded in retrieved evidence, not hallucinated |

### Sample Template: `guardrails/retrieval_guardrails.yaml`

```yaml
retrieval_guardrails:
  enabled: true
  version: "1.0.0"

  source_filtering:
    enabled: true
    approved_sources:
      - name: "internal_kb"
        type: "vector_store"
        trust_level: "high"
      - name: "public_docs"
        type: "web_crawl"
        trust_level: "medium"
    blocked_sources:
      - "*.pastebin.com"
      - "*.4chan.org"
    require_source_attribution: true

  trust_scoring:
    enabled: true
    scoring_model: "weighted"  # weighted | ml_classifier
    weights:
      source_authority: 0.4
      content_freshness: 0.3
      relevance_score: 0.2
      citation_count: 0.1
    minimum_trust_score: 0.6
    on_low_trust: "flag_for_review"

  metadata_filtering:
    enabled: true
    required_metadata: ["source", "date", "author"]
    filters:
      max_age_days: 365
      allowed_classifications: ["public", "internal"]
      excluded_authors: []

  freshness_checks:
    enabled: true
    max_content_age_days: 180
    stale_content_action: "downrank"  # downrank | exclude | warn
    freshness_boost_days: 30  # boost content newer than this

  chunk_validation:
    enabled: true
    min_chunk_tokens: 50
    max_chunk_tokens: 1024
    require_complete_sentences: true
    deduplication: true
    coherence_threshold: 0.5

  grounding_rules:
    enabled: true
    require_citation: true
    max_unsupported_claims: 0
    citation_format: "inline"  # inline | footnote | appendix
    on_ungrounded_content: "flag_and_disclaim"
```

---

## 5. Tool Guardrails

**Purpose:** Enforce least-privilege access using allowlists, permission boundaries, transaction limits, and human approval gates.

### Sub-Controls (from infographic)

| Control | Description |
|---|---|
| Tool allowlists | Whitelist which tools the agent can invoke |
| Permission checks | Verify the agent has authorization before tool execution |
| Transaction limits | Cap the scope of actions (dollar amounts, record counts, etc.) |
| Tool sandboxing | Execute tools in isolated environments to contain blast radius |
| Timeout controls | Enforce execution time limits on tool calls |
| Human confirmation | Require human approval for high-risk or irreversible actions |

### Sample Template: `guardrails/tool_guardrails.yaml`

```yaml
tool_guardrails:
  enabled: true
  version: "1.0.0"

  tool_allowlists:
    enabled: true
    default_policy: "deny"  # deny | allow
    allowed_tools:
      - name: "search_knowledge_base"
        risk_level: "low"
        requires_approval: false
      - name: "send_email"
        risk_level: "medium"
        requires_approval: true
      - name: "execute_sql"
        risk_level: "high"
        requires_approval: true
        restricted_operations: ["DROP", "DELETE", "TRUNCATE", "ALTER"]
      - name: "process_payment"
        risk_level: "critical"
        requires_approval: true
    blocked_tools:
      - "shell_execute"
      - "file_system_write"
      - "network_raw_socket"

  permission_checks:
    enabled: true
    check_before_execution: true
    permission_model: "rbac"  # rbac | abac | policy_engine
    roles:
      - name: "read_only_agent"
        allowed_actions: ["read", "search", "summarize"]
      - name: "operator_agent"
        allowed_actions: ["read", "search", "summarize", "create", "update"]
      - name: "admin_agent"
        allowed_actions: ["read", "search", "summarize", "create", "update", "delete"]

  transaction_limits:
    enabled: true
    limits:
      - tool: "process_payment"
        max_amount: 1000
        currency: "USD"
        max_per_session: 3
      - tool: "execute_sql"
        max_rows_affected: 100
        max_queries_per_session: 10
      - tool: "send_email"
        max_recipients: 5
        max_per_hour: 20
    on_exceed: "block_and_escalate"

  tool_sandboxing:
    enabled: true
    execution_environment: "container"  # container | vm | wasm | process
    network_isolation: true
    filesystem_isolation: true
    max_memory_mb: 512
    max_cpu_seconds: 30

  timeout_controls:
    default_timeout_seconds: 30
    per_tool_overrides:
      search_knowledge_base: 10
      execute_sql: 15
      send_email: 20
      process_payment: 45
    on_timeout: "abort_and_log"

  human_confirmation:
    enabled: true
    require_for_risk_levels: ["high", "critical"]
    approval_timeout_seconds: 300
    approval_channels: ["in_app", "slack", "email"]
    auto_deny_on_timeout: true
    audit_all_approvals: true
```

---

## 6. Runtime Guardrails

**Purpose:** Detect anomalies, monitor latency, prevent infinite agent loops, manage concurrency, and enable safe fallbacks.

### Sub-Controls (from infographic)

| Control | Description |
|---|---|
| Session monitoring | Track active sessions, token usage, and agent behavior in real time |
| Anomaly detection | Flag unusual patterns (sudden topic shifts, repetitive outputs, cost spikes) |
| Latency tracking | Monitor response times and flag degradation |
| Loop detection | Detect and break infinite reasoning or tool-call loops |
| Fallback routing | Route to fallback models or human handoff when primary fails |
| Concurrency control | Manage parallel agent executions and resource contention |

### Sample Template: `guardrails/runtime_guardrails.yaml`

```yaml
runtime_guardrails:
  enabled: true
  version: "1.0.0"

  session_monitoring:
    enabled: true
    track_metrics:
      - "tokens_used"
      - "tool_calls_count"
      - "response_times"
      - "error_rate"
    max_session_duration_minutes: 60
    max_tokens_per_session: 100000
    max_tool_calls_per_session: 50
    alert_thresholds:
      tokens_used_pct: 80
      error_rate_pct: 10

  anomaly_detection:
    enabled: true
    detectors:
      - name: "topic_drift"
        description: "Flag sudden topic shifts mid-conversation"
        threshold: 0.7
        action: "warn"
      - name: "repetitive_output"
        description: "Detect repeated phrases or circular reasoning"
        max_similarity: 0.95
        window_size: 5  # last N responses
        action: "interrupt"
      - name: "cost_spike"
        description: "Flag sessions exceeding cost baseline"
        max_cost_multiplier: 3.0
        action: "throttle"

  latency_tracking:
    enabled: true
    p50_threshold_ms: 500
    p95_threshold_ms: 2000
    p99_threshold_ms: 5000
    on_exceed: "log_and_alert"
    degradation_action: "switch_to_smaller_model"

  loop_detection:
    enabled: true
    max_iterations: 10
    max_recursive_depth: 5
    detection_methods:
      - "call_count"       # same tool called N times
      - "output_similarity" # near-identical outputs
      - "state_cycle"      # agent returns to previous state
    on_detect: "break_and_summarize"
    cooldown_seconds: 60

  fallback_routing:
    enabled: true
    primary_model: "gpt-4o"
    fallback_chain:
      - model: "gpt-4o-mini"
        trigger: "primary_timeout"
      - model: "claude-3-haiku"
        trigger: "primary_error"
      - action: "human_handoff"
        trigger: "all_models_failed"
    health_check_interval_seconds: 30

  concurrency_control:
    max_parallel_sessions: 100
    max_parallel_tool_calls: 5
    queue_strategy: "fifo"  # fifo | priority | fair_share
    backpressure_threshold: 80  # pct of capacity
    on_overload: "queue_with_timeout"
    queue_timeout_seconds: 30
```

---

## 7. Output Guardrails

**Purpose:** Validate responses with PII detection, hallucination checks, toxicity filtering, policy enforcement, and response validation.

### Sub-Controls (from infographic)

| Control | Description |
|---|---|
| Content moderation | Filter harmful, offensive, or inappropriate content |
| Toxicity checks | Score output for toxic language and reject above threshold |
| PII masking | Detect and redact personally identifiable information in responses |
| Hallucination checks | Verify claims against source material and flag unsupported statements |
| Response validation | Validate output format, length, and schema compliance |
| Restricted topic blocking | Block responses on prohibited topics (legal, medical, financial advice) |

### Sample Template: `guardrails/output_guardrails.yaml`

```yaml
output_guardrails:
  enabled: true
  version: "1.0.0"

  content_moderation:
    enabled: true
    classifier: "openai_moderation"  # openai_moderation | custom | perspective_api
    categories:
      - "hate"
      - "harassment"
      - "self_harm"
      - "sexual"
      - "violence"
    threshold: 0.7
    on_flag: "block_and_rephrase"

  toxicity_checks:
    enabled: true
    max_toxicity_score: 0.3
    check_per_sentence: true
    on_toxic: "remove_sentence_and_warn"

  pii_masking:
    enabled: true
    scan_output: true
    entity_types:
      - "PERSON_NAME"
      - "EMAIL"
      - "PHONE"
      - "ADDRESS"
      - "SSN"
      - "CREDIT_CARD"
      - "DATE_OF_BIRTH"
    masking_strategy: "redact"  # redact | hash | generalize
    redact_format: "[REDACTED:{entity_type}]"

  hallucination_checks:
    enabled: true
    method: "source_comparison"  # source_comparison | nli_model | hybrid
    require_source_for_claims: true
    confidence_threshold: 0.8
    on_hallucination: "flag_and_disclaim"
    disclaimer_text: "Note: This claim could not be verified against available sources."

  response_validation:
    enabled: true
    max_output_tokens: 4096
    enforce_json_schema: false  # set true for structured outputs
    required_sections: []
    prohibited_phrases:
      - "as an AI language model"
      - "I cannot and will not"
    format_checks:
      valid_utf8: true
      no_null_bytes: true

  restricted_topic_blocking:
    enabled: true
    blocked_topics:
      - name: "legal_advice"
        keywords: ["legal advice", "sue", "lawsuit", "attorney recommendation"]
        action: "redirect_to_professional"
      - name: "medical_diagnosis"
        keywords: ["diagnose", "prescription", "you have", "symptoms indicate"]
        action: "redirect_to_professional"
      - name: "financial_advice"
        keywords: ["invest in", "buy stock", "financial advice", "guaranteed returns"]
        action: "redirect_to_professional"
    redirect_message: "For {topic}, please consult a qualified professional."
```

---

## Master Configuration: `guardrails/guardrails_config.yaml`

```yaml
guardrails_config:
  version: "1.0.0"
  description: "Master guardrail configuration for production agentic AI systems"

  layers:
    - file: "input_guardrails.yaml"
      enabled: true
      order: 1
    - file: "prompt_guardrails.yaml"
      enabled: true
      order: 2
    - file: "memory_guardrails.yaml"
      enabled: true
      order: 3
    - file: "retrieval_guardrails.yaml"
      enabled: true
      order: 4
    - file: "tool_guardrails.yaml"
      enabled: true
      order: 5
    - file: "runtime_guardrails.yaml"
      enabled: true
      order: 6
    - file: "output_guardrails.yaml"
      enabled: true
      order: 7

  global_settings:
    log_level: "info"  # debug | info | warn | error
    audit_trail: true
    metrics_export: "prometheus"  # prometheus | datadog | cloudwatch
    alert_channel: "slack"
    environment: "production"  # development | staging | production
```

---

## Community Insights (from LinkedIn discussion)

**Weakest layers according to practitioners:**

- **Retrieval Guardrails**: Most commonly cited blind spot. Teams lock down tool access and inputs but trust whatever comes back from a vector search without scoring the source (Gaurav Sharma).
- **Tool Governance**: Where the agent moves from generating answers to actually changing things. Once agents call APIs, access cloud resources, or trigger workflows, least privilege and identity controls become critical (Rohini Vanga).
- **Cross-layer propagation**: Guardrails in isolation are insufficient. A suspicious retrieval should narrow tool authority; an ambiguous tool effect should block memory promotion (Aditya Singh).

**Key design principles raised:**
1. Security shifts from one-time review to ongoing discipline (Lakshay Dhiman)
2. Small gaps in access, validation, or monitoring create disproportionate downstream risks (Dr. Drijesh P.)
3. Deciding what an agent is allowed to do may be as important as securing the technology (Ariane Adjagbodjou)
4. Good security should protect progress without creating unnecessary friction (Carlos Shoji)

---

## Cross-References

- **Source post**: [Antrixsh Gupta on LinkedIn](https://lnkd.in/p/dW7FBjXB)
- **Author**: Antrixsh Gupta, Senior Solution & AI Architect @ Genzeon Platform
- **ClickUp reference task**: Capture 7 guardrail layers for production-ready agentic AI systems reference

---

## License

Content extracted and restructured for reference purposes. Original content by Antrixsh Gupta.