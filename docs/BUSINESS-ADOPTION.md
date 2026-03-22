# AVP Business Adoption Specification v1.0-draft

**Enterprise Readiness Requirements for the AI Vendor Protocol**

Status: Draft
Version: 1.0-draft
Date: 2026-03-22
Authors: AVP Working Group

---

## Table of Contents

1. [Compliance & Regulatory Framework](#1-compliance--regulatory-framework)
2. [SLA Framework](#2-sla-framework)
3. [Webhooks & Events](#3-webhooks--events)
4. [Multi-currency & Localization](#4-multi-currency--localization)
5. [Contract Templates](#5-contract-templates)
6. [Audit Trail](#6-audit-trail)
7. [Sandbox Mode](#7-sandbox-mode)
8. [Analytics API](#8-analytics-api)
9. [Dispute Resolution Protocol](#9-dispute-resolution-protocol)
10. [Data Portability](#10-data-portability)

---

## 1. Compliance & Regulatory Framework

### 1.1 GDPR Compliance

AVP implementations that process data of EU residents MUST satisfy the following:

| Requirement | Implementation |
|-------------|---------------|
| Lawful Basis | Agents MUST declare a lawful basis (`legitimate_interest`, `contract`, `consent`) in every `avp-manifest.json` capability block |
| Data Minimization | Proposals MUST NOT request or transmit personal data beyond what is strictly necessary for the negotiation |
| Right to Erasure | Agents MUST support `DELETE /avp/negotiations/{id}/data` to purge all negotiation-related personal data within 30 days |
| Data Processing Agreement | The `avp-manifest.json` SHOULD link to a machine-readable DPA at `dpa_url` |
| Cross-border Transfers | Agents MUST declare `data_residency` regions; data MUST NOT leave declared regions without explicit consent |

**Erasure Request Format:**

```json
{
  "type": "avp.data.erasure_request",
  "subject_id": "eu-resident-ref-001",
  "negotiation_ids": ["neg_abc123"],
  "reason": "gdpr_article_17",
  "requested_at": "2026-03-22T10:00:00Z"
}
```

Agents MUST respond with a confirmation including an `erasure_completion_eta` (ISO 8601 duration, maximum `P30D`).

### 1.2 SOC 2 Alignment

AVP implementations seeking SOC 2 alignment SHOULD implement:

- **Security**: All requirements from the AVP Security Model (see `SECURITY.md`)
- **Availability**: SLA framework defined in Section 2 of this document
- **Confidentiality**: End-to-end encryption for negotiation payloads using the agent's Ed25519 keys
- **Processing Integrity**: Schema validation on all inbound and outbound messages
- **Privacy**: GDPR controls as defined in Section 1.1

### 1.3 Data Residency

Agents MUST declare supported residency regions in `avp-manifest.json`:

```json
{
  "compliance": {
    "data_residency": ["eu-west", "us-east", "ap-southeast"],
    "certifications": ["soc2-type2", "iso27001", "gdpr"],
    "dpa_url": "https://example.com/.well-known/avp-dpa.json"
  }
}
```

Buyer agents MAY specify a `required_residency` constraint in RFPs. Vendor agents MUST reject the RFP with error code `AVP-4010` (Residency Constraint Unmet) if they cannot comply.

---

## 2. SLA Framework

### 2.1 Response Time Requirements

Conformant implementations MUST meet the following latency targets measured at the 95th percentile:

| Operation | Target | Maximum | Error Code on Timeout |
|-----------|--------|---------|----------------------|
| Discovery (`GET /.well-known/avp-manifest.json`) | 200ms | 500ms | `AVP-5001` |
| Handshake (`POST /avp/handshake`) | 1s | 2s | `AVP-5002` |
| RFP Response (`POST /avp/rfp/{id}/proposals`) | 15s | 30s | `AVP-5003` |
| Negotiation Round | 5s | 10s | `AVP-5004` |
| Settlement Confirmation | 2s | 5s | `AVP-5005` |

### 2.2 Uptime Expectations

Agents SHOULD declare their uptime commitment in the manifest:

```json
{
  "sla": {
    "uptime_target": "99.9",
    "maintenance_window": "sunday/02:00-06:00Z",
    "status_url": "https://status.example.com/avp"
  }
}
```

| Tier | Uptime | Max Monthly Downtime |
|------|--------|---------------------|
| Tier 1 (Enterprise) | 99.99% | 4.3 minutes |
| Tier 2 (Business) | 99.9% | 43.8 minutes |
| Tier 3 (Standard) | 99.5% | 3.6 hours |

### 2.3 Degradation Handling

When an agent cannot meet SLA targets, it MUST return an `AVP-Degraded` response header:

```http
AVP-Degraded: true
AVP-Degraded-Reason: high_load
AVP-Retry-After: 30
```

Agents operating in degraded mode SHOULD:
1. Continue serving discovery requests at reduced fidelity
2. Queue inbound RFPs and acknowledge with `202 Accepted`
3. Pause active negotiations and notify counterparties via the `negotiation.paused` event

---

## 3. Webhooks & Events

### 3.1 Event Types

Agents MAY subscribe to events by registering a webhook URL during handshake. The following event types are defined:

| Event Type | Trigger | Payload Key Fields |
|------------|---------|-------------------|
| `rfp.received` | New RFP arrives | `rfp_id`, `buyer_agent_id`, `requirements` |
| `rfp.expired` | RFP passes deadline | `rfp_id`, `expired_at` |
| `proposal.sent` | Proposal submitted | `proposal_id`, `rfp_id`, `pricing` |
| `proposal.accepted` | Buyer accepts proposal | `proposal_id`, `terms` |
| `proposal.rejected` | Buyer rejects proposal | `proposal_id`, `reason` |
| `negotiation.updated` | Counter-offer or term change | `negotiation_id`, `round`, `changes` |
| `negotiation.paused` | Agent enters degraded mode | `negotiation_id`, `resume_eta` |
| `settlement.completed` | Agreement finalized | `settlement_id`, `final_terms`, `signatures` |
| `dispute.opened` | Dispute filed | `dispute_id`, `negotiation_id`, `grounds` |

### 3.2 Delivery Format

```json
{
  "event_id": "evt_a1b2c3d4",
  "type": "settlement.completed",
  "timestamp": "2026-03-22T14:30:00Z",
  "agent_id": "avp:vendor:acme:x7k9",
  "payload": { },
  "signature": "ed25519:{base64_signature}"
}
```

### 3.3 Delivery Guarantees

- Events are delivered **at least once**. Receivers MUST be idempotent.
- Each event includes a unique `event_id`; receivers SHOULD deduplicate by this field.
- The receiver MUST respond with `200 OK` within 5 seconds to acknowledge receipt.

### 3.4 Retry Policy

On non-2xx response or timeout, the sender MUST retry with exponential backoff:

| Attempt | Delay |
|---------|-------|
| 1 | 30 seconds |
| 2 | 2 minutes |
| 3 | 15 minutes |
| 4 | 1 hour |
| 5 (final) | 4 hours |

After 5 failed attempts, the event is written to a dead-letter queue. The receiving agent MAY poll `GET /avp/events/missed?since={timestamp}` to recover missed events.

---

## 4. Multi-currency & Localization

### 4.1 Currency Support in Proposals

All monetary values in proposals MUST use the following structure:

```json
{
  "pricing": {
    "amount": "15000.00",
    "currency": "USD",
    "fx_rate_source": "ecb",
    "fx_rate_timestamp": "2026-03-22T12:00:00Z",
    "accepted_currencies": ["USD", "EUR", "GBP", "JPY"]
  }
}
```

- `amount` MUST be a string to avoid floating-point precision loss.
- `currency` MUST be an ISO 4217 three-letter code.
- Vendors SHOULD declare `accepted_currencies` in their manifest.
- When a buyer requests a different currency, the vendor MAY include an `fx_rate_source` and `fx_rate_timestamp` so both parties can verify the conversion.

### 4.2 Timezone Handling

- All protocol timestamps MUST be in UTC (ISO 8601 with `Z` suffix).
- Agents MAY declare a `display_timezone` in their manifest for human-facing contexts (IANA timezone identifier, e.g., `America/New_York`).
- Deadline fields (`rfp_deadline`, `proposal_valid_until`) MUST be interpreted as UTC regardless of agent timezone.

### 4.3 Language Preferences

Agents SHOULD declare language capabilities in `avp-manifest.json`:

```json
{
  "localization": {
    "default_language": "en",
    "supported_languages": ["en", "de", "ja", "es"],
    "content_negotiation": true
  }
}
```

When `content_negotiation` is `true`, buyer agents MAY send an `AVP-Language: de` header to request responses in that language. Vendors MUST fall back to `default_language` if the requested language is unsupported.

---

## 5. Contract Templates

### 5.1 Standard Agreement Schema

Settlements SHOULD produce a machine-readable contract object:

```json
{
  "contract": {
    "contract_id": "con_m8n2p4",
    "version": "1.0",
    "parties": [
      { "agent_id": "avp:buyer:globex:a1b2", "legal_entity": "Globex Corp", "jurisdiction": "US-DE" },
      { "agent_id": "avp:vendor:acme:x7k9", "legal_entity": "Acme Ltd", "jurisdiction": "GB" }
    ],
    "terms": {
      "scope": "cloud-infrastructure",
      "deliverables": ["Compute provisioning", "99.9% SLA"],
      "pricing": { "amount": "15000.00", "currency": "USD", "billing_cycle": "monthly" },
      "duration": { "start": "2026-04-01", "end": "2027-03-31" },
      "termination_notice_days": 30,
      "governing_law": "US-DE",
      "dispute_resolution": "avp_arbitration"
    },
    "custom_terms": {},
    "signatures": []
  }
}
```

### 5.2 Terms & Conditions Framework

Agents MAY reference pre-registered terms templates by URI:

```json
{
  "terms_template": "https://registry.avp-protocol.org/v1/terms/standard-saas-v1",
  "overrides": {
    "termination_notice_days": 60
  }
}
```

The AVP registry SHOULD maintain a library of common templates (SaaS, consulting, licensing, procurement). Custom terms MUST be placed in the `custom_terms` object and are not validated by the protocol.

### 5.3 Digital Signature Integration

Contract signatures use the agent's existing Ed25519 keypair:

```json
{
  "signatures": [
    {
      "agent_id": "avp:buyer:globex:a1b2",
      "signed_at": "2026-03-22T15:00:00Z",
      "key_id": "key-2026-03-01",
      "signature": "ed25519:{base64_signature}",
      "human_approved": true
    }
  ]
}
```

The `human_approved` field indicates whether a human reviewed the contract before the agent signed. Implementations MAY require `human_approved: true` for contracts above a configurable value threshold.

---

## 6. Audit Trail

### 6.1 Required Logging Fields

Every AVP interaction MUST produce an audit log entry containing at minimum:

| Field | Type | Description |
|-------|------|-------------|
| `log_id` | string | Unique log entry identifier |
| `timestamp` | ISO 8601 | When the event occurred (UTC) |
| `agent_id` | string | Agent that generated the log |
| `counterparty_id` | string | The other agent involved |
| `action` | string | Action type (e.g., `rfp.submit`, `proposal.respond`) |
| `session_id` | string | AVP session identifier |
| `negotiation_id` | string | Negotiation identifier (if applicable) |
| `request_hash` | string | SHA-256 hash of the request body |
| `response_code` | integer | HTTP status code returned |
| `ip_address` | string | Source IP (hashed for GDPR compliance if needed) |
| `signature` | string | Ed25519 signature of the log entry |

### 6.2 Retention Periods

| Data Category | Minimum Retention | Maximum Retention | Notes |
|---------------|------------------|------------------|-------|
| Negotiation logs | 3 years | 7 years | Subject to governing law |
| Settlement/contract records | 7 years | 10 years | Financial compliance |
| Discovery logs | 90 days | 1 year | Operational |
| Failed authentication | 1 year | 3 years | Security analysis |
| Erased-subject records | Tombstone only | 3 years | GDPR Article 17 proof of deletion |

### 6.3 Compliance Export Formats

Agents MUST support exporting audit data in at least one of the following formats:

- **JSON Lines** (`.jsonl`) — one log entry per line, default
- **CSV** — for spreadsheet-based review workflows
- **SIEM-compatible** — CEF (Common Event Format) for integration with security platforms

Export endpoint:

```
GET /avp/audit/export?from={iso8601}&to={iso8601}&format=jsonl
Authorization: Bearer {admin_token}
```

Exports MUST be signed and include an integrity checksum in the `Content-Digest` response header (RFC 9530).

---

## 7. Sandbox Mode

### 7.1 Test Environment Specification

AVP implementations MUST provide a sandbox mode for integration testing. Sandbox is activated by using test agent IDs (see Section 7.3) or by targeting the sandbox registry.

| Property | Production | Sandbox |
|----------|-----------|---------|
| Registry URL | `registry.avp-protocol.org` | `sandbox.registry.avp-protocol.org` |
| Real negotiations | Yes | No |
| Rate limits | Standard | Relaxed (10x) |
| Data retention | Per Section 6.2 | 30 days auto-purge |
| Webhook delivery | At-least-once | Best-effort |

### 7.2 Mock Responses

The sandbox registry provides built-in mock vendor agents that return predictable responses:

| Mock Agent ID | Behavior |
|---------------|----------|
| `avp:test:always-accept:mock` | Accepts any proposal immediately |
| `avp:test:always-reject:mock` | Rejects with reason `price_too_low` |
| `avp:test:slow-responder:mock` | Responds after 25 seconds (for timeout testing) |
| `avp:test:counter-offer:mock` | Always counters at 80% of proposed price |
| `avp:test:error-500:mock` | Returns server error (for retry testing) |
| `avp:test:multi-round:mock` | Requires 3 negotiation rounds before accepting |

### 7.3 Test Agent IDs

All test agent IDs MUST match the pattern `avp:test:*`. Agents MUST reject any `avp:test:*` agent ID in production environments. Production registries MUST refuse registration of `avp:test:*` identifiers.

### 7.4 Sandbox Registry

The sandbox registry mirrors the production API surface but:
- Resets all data every 30 days
- Does not enforce business verification requirements
- Provides pre-seeded test data for common integration scenarios
- Exposes a `POST /sandbox/reset` endpoint to clear an agent's test data on demand

---

## 8. Analytics API

### 8.1 Usage Metrics

```
GET /avp/analytics/usage?period=30d
```

**Response:**

```json
{
  "period": { "from": "2026-02-20", "to": "2026-03-22" },
  "metrics": {
    "rfps_received": 142,
    "rfps_responded": 138,
    "proposals_sent": 156,
    "negotiations_started": 89,
    "settlements_completed": 34,
    "avg_negotiation_rounds": 2.4,
    "total_contract_value": { "amount": "524000.00", "currency": "USD" }
  }
}
```

### 8.2 Conversion Tracking

```
GET /avp/analytics/conversions?period=30d
```

**Response:**

```json
{
  "funnel": {
    "rfps_received": 142,
    "proposals_submitted": 138,
    "shortlisted": 67,
    "negotiations_entered": 89,
    "settlements_reached": 34
  },
  "conversion_rates": {
    "rfp_to_proposal": "97.2%",
    "proposal_to_shortlist": "48.6%",
    "shortlist_to_negotiation": "100.0%",
    "negotiation_to_settlement": "38.2%",
    "overall": "23.9%"
  }
}
```

### 8.3 Response Time Monitoring

```
GET /avp/analytics/performance?period=7d
```

**Response:**

```json
{
  "latency_ms": {
    "discovery": { "p50": 85, "p95": 190, "p99": 420 },
    "handshake": { "p50": 340, "p95": 890, "p99": 1800 },
    "rfp_response": { "p50": 4200, "p95": 12000, "p99": 26000 },
    "negotiation_round": { "p50": 2100, "p95": 4800, "p99": 8900 }
  },
  "sla_compliance": {
    "within_target": "96.3%",
    "violations": 12,
    "violation_details_url": "/avp/analytics/sla-violations"
  }
}
```

All analytics endpoints require `Authorization: Bearer {agent_token}` with `analytics:read` scope. Data is scoped to the requesting agent only; agents MUST NOT access another agent's analytics.

---

## 9. Dispute Resolution Protocol

### 9.1 Escalation Flow

Disputes follow a three-stage escalation:

```
Stage 1: Direct Resolution (agents negotiate automatically)
    │ 48 hours timeout
    ▼
Stage 2: Mediation (registry-appointed mediator agent)
    │ 7 days timeout
    ▼
Stage 3: Arbitration (human arbitration panel)
    │ 30 days resolution target
    ▼
Final Decision (binding)
```

### 9.2 Filing a Dispute

```
POST /avp/disputes
```

```json
{
  "negotiation_id": "neg_abc123",
  "settlement_id": "stl_def456",
  "grounds": "non_delivery",
  "description": "Vendor failed to deliver agreed services within the contracted timeline.",
  "evidence": [
    {
      "type": "negotiation_log",
      "ref": "/avp/audit/export?negotiation_id=neg_abc123"
    },
    {
      "type": "external_document",
      "url": "https://example.com/evidence/timeline.pdf",
      "hash": "sha256:a1b2c3..."
    }
  ],
  "requested_remedy": "partial_refund",
  "remedy_amount": { "amount": "7500.00", "currency": "USD" }
}
```

Valid `grounds` values: `non_delivery`, `quality_deficiency`, `terms_violation`, `billing_dispute`, `misrepresentation`.

### 9.3 Evidence Submission Format

Evidence attachments MUST include a content hash for integrity verification. Accepted formats:

| Type | Description | Max Size |
|------|-------------|----------|
| `negotiation_log` | AVP audit trail reference | N/A (reference) |
| `external_document` | PDF, signed document | 10 MB |
| `communication_record` | Timestamped message log | 5 MB |
| `performance_data` | Metrics/monitoring export | 5 MB |

All evidence is immutable once submitted. Additional evidence MAY be submitted until the arbitration panel closes the evidence period.

### 9.4 Arbitration Decisions

Arbitration decisions are published as signed AVP events:

```json
{
  "type": "dispute.resolved",
  "dispute_id": "dsp_g7h8i9",
  "decision": "partial_favor_claimant",
  "remedy": { "amount": "5000.00", "currency": "USD" },
  "rationale_url": "https://registry.avp-protocol.org/v1/disputes/dsp_g7h8i9/decision",
  "binding": true
}
```

---

## 10. Data Portability

### 10.1 Export Formats

Agents MUST support full data export for portability. The export package includes:

| Component | Format | File |
|-----------|--------|------|
| Agent manifest | JSON | `manifest.json` |
| Negotiation history | JSON Lines | `negotiations.jsonl` |
| Contracts | JSON | `contracts/*.json` |
| Audit logs | JSON Lines | `audit.jsonl` |
| Cryptographic keys (public) | JWK Set | `keys.jwks` |
| Analytics snapshots | JSON | `analytics.json` |

Export endpoint:

```
POST /avp/data/export
```

Response returns a signed archive URL:

```json
{
  "export_id": "exp_j1k2l3",
  "status": "processing",
  "download_url": "https://example.com/avp/data/export/exp_j1k2l3.tar.gz",
  "available_until": "2026-04-22T00:00:00Z",
  "checksum": "sha256:d4e5f6..."
}
```

### 10.2 Migration Between Registries

Agents MAY migrate from one registry to another. The migration flow:

1. Agent requests export from source registry via `POST /avp/data/export`
2. Agent submits the signed export to the target registry via `POST /registry/v1/import`
3. Target registry verifies signatures and provenance
4. Source registry marks the agent as `migrated` with a redirect pointer for 90 days
5. After 90 days, the source registry MAY remove the agent's records

During migration, both registries MUST recognize the agent's ID. The agent ID itself does not change.

### 10.3 Vendor Data Ownership

The following data ownership rules apply:

| Data Type | Owner | Portability |
|-----------|-------|------------|
| Agent manifest & configuration | Agent operator | Fully portable |
| Negotiation messages (sent) | Sending agent | Exportable |
| Negotiation messages (received) | Both parties | Exportable by either party |
| Settlement contracts | Both parties | Exportable by either party |
| Registry metadata | Registry operator | Not portable (re-generated on import) |
| Trust scores | Registry operator | Not portable |
| Analytics data | Agent operator | Fully portable |

Agents MUST NOT claim exclusive ownership over data generated bilaterally. Both parties to a negotiation retain the right to export their view of the interaction.

---

## Appendix A: Error Codes

| Code | Meaning |
|------|---------|
| `AVP-4010` | Residency Constraint Unmet |
| `AVP-4011` | Currency Not Supported |
| `AVP-4012` | Language Not Supported |
| `AVP-5001` | Discovery Timeout |
| `AVP-5002` | Handshake Timeout |
| `AVP-5003` | RFP Response Timeout |
| `AVP-5004` | Negotiation Round Timeout |
| `AVP-5005` | Settlement Timeout |
| `AVP-5010` | Sandbox Agent in Production |
| `AVP-5020` | Export in Progress |

## Appendix B: Conformance

An implementation is **Business Adoption Conformant** if it satisfies:

1. **MUST** requirements from at least Sections 1, 2, 3, and 6
2. **SHOULD** implement Sections 4, 5, and 7
3. **MAY** implement Sections 8, 9, and 10

Implementations SHOULD declare their conformance level in the manifest:

```json
{
  "business_adoption": {
    "conformance_level": "full",
    "sections_implemented": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
  }
}
```
