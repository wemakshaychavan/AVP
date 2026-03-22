# AVP API Reference

**Complete API Documentation for All AVP Endpoints**

Version: 1.0-draft
Date: 2026-03-18

---

## Table of Contents

1. [Base URL & Headers](#1-base-url--headers)
2. [Discovery Endpoints](#2-discovery-endpoints)
3. [Handshake Endpoints](#3-handshake-endpoints)
4. [RFP Endpoints](#4-rfp-endpoints)
5. [Negotiation Endpoints](#5-negotiation-endpoints)
6. [Settlement Endpoints](#6-settlement-endpoints)
7. [Capability Endpoints](#7-capability-endpoints)
8. [Status Endpoints](#8-status-endpoints)
9. [Registry API](#9-registry-api)
10. [Trust API](#10-trust-api)
11. [Webhook Events](#11-webhook-events)
12. [Rate Limits](#12-rate-limits)
13. [SDK Quick Reference](#13-sdk-quick-reference)

---

## 1. Base URL & Headers

### 1.1 Vendor Endpoints

```
Base URL: https://{vendor-domain}/avp/
```

### 1.2 Registry Endpoints

```
Base URL: https://registry.avp-protocol.org/v1/
```

### 1.3 Required Headers (All Requests)

```http
AVP-Version: 1.0
AVP-Agent-ID: avp:{type}:{org}:{hash}
AVP-Timestamp: 2026-03-18T10:00:00Z
AVP-Signature: ed25519:{base64_signature}
Content-Type: application/json
```

### 1.4 Conditional Headers

```http
AVP-Session-ID: ses_{id}          # Required after handshake
AVP-Idempotency-Key: idk_{uuid}   # Recommended for mutations
AVP-Key-ID: key-{date}             # Required for signature verification
```

---

## 2. Discovery Endpoints

### 2.1 GET /.well-known/avp.json

Retrieve vendor's discovery document.

**Request:**
```http
GET /.well-known/avp.json HTTP/1.1
Host: vendor.com
Accept: application/json
```

**Response: 200 OK**
```json
{
  "avp_version": "1.0",
  "agent": {
    "id": "avp:vendor:acme-tech:a1b2c3d4",
    "name": "Acme Tech Solutions",
    "type": "vendor",
    "description": "Enterprise software development",
    "website": "https://acmetech.com"
  },
  "capabilities": {
    "technologies": [
      {"name": "java", "proficiency": "expert", "years": 10}
    ],
    "domains": [
      {"name": "travel", "experience_years": 8}
    ],
    "services": ["custom-development", "consulting"]
  },
  "endpoints": {
    "handshake": "/avp/handshake",
    "rfp": "/avp/rfp",
    "negotiate": "/avp/negotiate",
    "settle": "/avp/settle",
    "capabilities": "/avp/capabilities",
    "status": "/avp/status"
  },
  "policies": {
    "auto_respond": true,
    "max_negotiation_rounds": 5,
    "response_time_sla": "4h",
    "min_project_size": 10000
  },
  "trust": {
    "verification_status": "verified",
    "trust_score": 0.87
  },
  "rate_limits": {
    "rfp": "10/hour",
    "negotiate": "50/hour"
  }
}
```

**Response Headers:**
```http
Cache-Control: public, max-age=3600
ETag: "abc123"
```

**Error Responses:**

| Status | Description |
|--------|-------------|
| 404 | Vendor does not have AVP enabled |
| 503 | Agent temporarily unavailable |

### 2.2 GET /.well-known/avp-keys.json

Retrieve vendor's public keys for signature verification.

**Response: 200 OK**
```json
{
  "agent_id": "avp:vendor:acme-tech:a1b2c3d4",
  "keys": [
    {
      "kid": "key-2026-03",
      "algorithm": "Ed25519",
      "public_key": "MCowBQYDK2VwAyEA...",
      "status": "active",
      "created": "2026-03-01T00:00:00Z",
      "expires": "2027-03-01T00:00:00Z"
    }
  ]
}
```

---

## 3. Handshake Endpoints

### 3.1 POST /avp/handshake

Initiate an authenticated session between buyer and vendor agents.

**Request:**
```json
{
  "type": "handshake_request",
  "buyer_agent_id": "avp:buyer:globex-corp:e5f6g7h8",
  "buyer_discovery_url": "https://globex.com/.well-known/avp.json",
  "intent": "rfp",
  "auth_method": "ed25519",
  "public_key": "MCowBQYDK2VwAyEA...",
  "nonce": "random-nonce-123",
  "timestamp": "2026-03-18T10:00:00Z"
}
```

**Response: 200 OK**
```json
{
  "type": "handshake_response",
  "status": "accepted",
  "session_id": "ses_abc123def456",
  "session_expires": "2026-03-18T22:00:00Z",
  "vendor_agent_id": "avp:vendor:acme-tech:a1b2c3d4",
  "auth_token": "eyJhbGciOiJFZDI1NTE5...",
  "nonce_response": "signed-nonce-response",
  "policies": {
    "max_negotiation_rounds": 5,
    "response_time_sla": "4h"
  }
}
```

**Error Responses:**

| Status | Code | Description |
|--------|------|-------------|
| 401 | AVP-401-001 | Invalid signature |
| 403 | AVP-403-001 | Agent not authorized |
| 403 | AVP-403-002 | Agent blacklisted |
| 403 | AVP-403-003 | Insufficient trust score |
| 429 | AVP-429-001 | Rate limit exceeded |

---

## 4. RFP Endpoints

### 4.1 POST /avp/rfp

Submit a Request for Proposal.

**Headers:**
```http
AVP-Session-ID: ses_abc123def456
```

**Request:**
```json
{
  "type": "rfp",
  "session_id": "ses_abc123def456",
  "rfp_id": "rfp_xyz789",
  "title": "Travel Booking Platform Development",
  "description": "Java-based travel booking platform with GDS integration",
  "requirements": {
    "technologies": {
      "required": ["java", "spring-boot"],
      "preferred": ["react", "postgresql"]
    },
    "domains": {
      "required": ["travel"]
    },
    "team": {
      "min_size": 5,
      "roles": ["tech-lead", "backend-developer", "frontend-developer"]
    },
    "timeline": {
      "start_date": "2026-05-01",
      "end_date": "2026-10-31"
    },
    "budget": {
      "range": {"min": 100000, "max": 250000},
      "currency": "USD"
    },
    "compliance": ["GDPR", "PCI-DSS"]
  },
  "evaluation_criteria": [
    {"criterion": "technical_expertise", "weight": 0.30},
    {"criterion": "domain_experience", "weight": 0.25},
    {"criterion": "pricing", "weight": 0.20},
    {"criterion": "timeline_feasibility", "weight": 0.15},
    {"criterion": "team_quality", "weight": 0.10}
  ],
  "response_deadline": "2026-03-25T23:59:59Z"
}
```

**Response: 202 Accepted**
```json
{
  "type": "rfp_acknowledgement",
  "rfp_id": "rfp_xyz789",
  "status": "received",
  "estimated_response_time": "2026-03-18T14:00:00Z",
  "message": "RFP received. Our AI agent is preparing a proposal."
}
```

### 4.2 GET /avp/rfp/{rfp_id}/proposal

Retrieve the proposal for a specific RFP (polling).

**Response: 200 OK**
```json
{
  "type": "proposal",
  "rfp_id": "rfp_xyz789",
  "proposal_id": "prop_abc123",
  "status": "submitted",
  "executive_summary": "...",
  "solution": { },
  "team": { },
  "timeline": { },
  "pricing": {
    "model": "fixed-price",
    "total": 185000,
    "currency": "USD",
    "breakdown": [ ],
    "negotiable": true
  },
  "confidence_score": 0.88
}
```

**Response: 202 Accepted (still processing)**
```json
{
  "type": "proposal_pending",
  "rfp_id": "rfp_xyz789",
  "status": "processing",
  "estimated_ready": "2026-03-18T14:00:00Z"
}
```

---

## 5. Negotiation Endpoints

### 5.1 POST /avp/negotiate

Submit a negotiation message (counter-proposal, accept, reject).

**Counter-Proposal Request:**
```json
{
  "type": "counter_proposal",
  "session_id": "ses_abc123def456",
  "rfp_id": "rfp_xyz789",
  "proposal_id": "prop_abc123",
  "round": 1,
  "changes": [
    {
      "field": "pricing.total",
      "original_value": 185000,
      "proposed_value": 165000,
      "justification": "Budget constraint"
    }
  ],
  "message": "Can you optimize pricing by reducing Discovery phase scope?"
}
```

**Response: 200 OK**
```json
{
  "type": "counter_proposal",
  "round": 1,
  "status": "counter",
  "changes": [
    {
      "field": "pricing.total",
      "original_value": 185000,
      "proposed_value": 175000,
      "justification": "Optimized Discovery phase, reduced from 3 weeks to 2 weeks."
    }
  ],
  "message": "We've optimized the Discovery phase. Best we can offer is $175K.",
  "remaining_rounds": 4
}
```

### 5.2 POST /avp/negotiate/accept

Accept the current proposal terms.

**Request:**
```json
{
  "type": "accept",
  "session_id": "ses_abc123def456",
  "rfp_id": "rfp_xyz789",
  "proposal_id": "prop_abc123",
  "accepted_terms": {
    "pricing": {"total": 175000, "currency": "USD"},
    "timeline": {"start": "2026-05-01", "end": "2026-10-31"}
  }
}
```

**Response: 200 OK**
```json
{
  "type": "acceptance_confirmation",
  "status": "accepted",
  "settlement_url": "/avp/settle",
  "message": "Terms accepted. Please proceed to settlement."
}
```

### 5.3 POST /avp/negotiate/reject

Reject and end negotiation.

**Request:**
```json
{
  "type": "reject",
  "session_id": "ses_abc123def456",
  "rfp_id": "rfp_xyz789",
  "reason": "budget_exceeded",
  "message": "Unable to reach agreement within our budget constraints.",
  "allow_future_contact": true
}
```

### 5.4 POST /avp/negotiate/escalate

Request human involvement.

**Request:**
```json
{
  "type": "escalation_request",
  "session_id": "ses_abc123def456",
  "reason": "Complex requirements need human discussion",
  "preferred_channel": "video_call",
  "contact": {
    "name": "John Smith",
    "email": "john@globex.com",
    "availability": "Mon-Fri 9am-5pm EST"
  }
}
```

---

## 6. Settlement Endpoints

### 6.1 POST /avp/settle

Finalize the agreement.

**Request:**
```json
{
  "type": "settlement",
  "session_id": "ses_abc123def456",
  "rfp_id": "rfp_xyz789",
  "proposal_id": "prop_abc123",
  "agreed_terms": {
    "pricing": {
      "total": 175000,
      "currency": "USD",
      "payment_schedule": [
        {"milestone": "Contract signing", "amount": 35000, "due": "2026-05-01"},
        {"milestone": "MVP delivery", "amount": 52500, "due": "2026-07-01"},
        {"milestone": "Beta delivery", "amount": 52500, "due": "2026-09-01"},
        {"milestone": "Launch", "amount": 35000, "due": "2026-10-31"}
      ]
    },
    "timeline": {
      "start": "2026-05-01",
      "end": "2026-10-31"
    },
    "team_size": 7,
    "nda_signed": true
  },
  "buyer_signature": "ed25519:base64...",
  "record_on_registry": true
}
```

**Response: 200 OK**
```json
{
  "type": "settlement_confirmation",
  "settlement_id": "stl_def456ghi789",
  "status": "confirmed",
  "vendor_signature": "ed25519:base64...",
  "registry_record_id": "rec_123456",
  "next_steps": {
    "contract_url": "https://acmetech.com/contracts/stl_def456ghi789",
    "onboarding_url": "https://acmetech.com/onboard/stl_def456ghi789",
    "contact": {
      "name": "Priya Sharma",
      "role": "Project Manager",
      "email": "priya@acmetech.com"
    }
  }
}
```

---

## 7. Capability Endpoints

### 7.1 POST /avp/capabilities

Query specific capabilities in detail.

**Request:**
```json
{
  "type": "capability_query",
  "session_id": "ses_abc123def456",
  "queries": [
    {
      "category": "technology",
      "name": "java",
      "detail_level": "full"
    },
    {
      "category": "domain",
      "name": "travel",
      "detail_level": "full"
    }
  ]
}
```

**Response: 200 OK**
```json
{
  "type": "capability_response",
  "capabilities": [
    {
      "category": "technology",
      "name": "java",
      "proficiency": "expert",
      "years": 10,
      "team_members": 15,
      "frameworks": ["spring-boot", "micronaut", "quarkus"],
      "certifications": ["Oracle Certified Professional"],
      "recent_projects": [
        {
          "name": "E-Commerce Platform",
          "year": 2025,
          "team_size": 8,
          "technologies": ["java-17", "spring-boot-3", "kafka"]
        }
      ]
    },
    {
      "category": "domain",
      "name": "travel",
      "experience_years": 8,
      "projects_completed": 15,
      "sub_domains": ["booking-engines", "gds-integration", "loyalty-programs"],
      "gds_partnerships": ["amadeus", "sabre"],
      "case_studies_available": true
    }
  ]
}
```

---

## 8. Status Endpoints

### 8.1 GET /avp/status

Agent health check.

**Response: 200 OK**
```json
{
  "status": "healthy",
  "agent_id": "avp:vendor:acme-tech:a1b2c3d4",
  "uptime": "99.97%",
  "active_sessions": 3,
  "queue_depth": 2,
  "response_time_avg": "2.3s",
  "version": "1.0"
}
```

### 8.2 GET /avp/status/session/{session_id}

Get negotiation status for a session.

**Response: 200 OK**
```json
{
  "session_id": "ses_abc123def456",
  "status": "negotiating",
  "rfp_id": "rfp_xyz789",
  "current_round": 2,
  "max_rounds": 5,
  "last_action": "counter_proposal",
  "last_action_by": "vendor",
  "last_action_at": "2026-03-18T12:30:00Z",
  "session_expires": "2026-03-18T22:00:00Z"
}
```

---

## 9. Registry API

### 9.1 POST /v1/agents/register

Register a new agent.

```json
{
  "agent_type": "vendor",
  "organization_name": "Acme Tech Solutions",
  "domain": "acmetech.com",
  "public_key": "MCowBQYDK2VwAyEA...",
  "contact_email": "admin@acmetech.com"
}
```

### 9.2 GET /v1/search

Search for agents.

```
GET /v1/search?technologies=java&domains=travel&min_trust_score=0.7&page=1&per_page=20
```

**Response:**
```json
{
  "results": [
    {
      "agent_id": "avp:vendor:acme-tech:a1b2c3d4",
      "name": "Acme Tech Solutions",
      "domain": "acmetech.com",
      "trust_score": 0.87,
      "match_score": 0.92,
      "capabilities_summary": {
        "technologies": ["java", "spring-boot"],
        "domains": ["travel", "e-commerce"]
      },
      "discovery_url": "https://acmetech.com/.well-known/avp.json"
    }
  ],
  "total": 47,
  "page": 1,
  "per_page": 20
}
```

### 9.3 GET /v1/agents/{agent_id}

Get agent details.

### 9.4 PUT /v1/agents/{agent_id}

Update agent profile.

### 9.5 GET /v1/agents/{agent_id}/trust

Get trust score details.

### 9.6 POST /v1/verify/submit

Submit for business verification.

### 9.7 GET /v1/verify/status/{agent_id}

Check verification status.

---

## 10. Trust API

### 10.1 GET /v1/trust/{agent_id}

Get trust details for an agent.

**Response:**
```json
{
  "agent_id": "avp:vendor:acme-tech:a1b2c3d4",
  "trust_score": 0.87,
  "verification_level": "business-verified",
  "factors": {
    "verification": 0.80,
    "transaction_history": 0.92,
    "response_quality": 0.85,
    "response_time": 0.95,
    "dispute_rate": 0.90,
    "network_age": 0.70,
    "peer_reviews": 0.88,
    "compliance": 0.98
  },
  "stats": {
    "total_transactions": 25,
    "successful_transactions": 23,
    "average_rating": 4.4,
    "total_reviews": 18,
    "disputes": 2,
    "member_since": "2025-06-15"
  }
}
```

### 10.2 POST /v1/trust/{agent_id}/review

Submit a review after transaction.

```json
{
  "session_id": "ses_abc123def456",
  "rating": {
    "overall": 4.5,
    "categories": {
      "proposal_quality": 5,
      "communication": 4,
      "pricing_accuracy": 4,
      "response_time": 5
    }
  },
  "comment": "Excellent proposal quality.",
  "would_recommend": true
}
```

---

## 11. Webhook Events

### 11.1 Configuring Webhooks

```
POST /v1/webhooks

{
  "url": "https://your-app.com/avp-webhook",
  "events": ["rfp.received", "proposal.sent", "negotiation.round", "settlement.reached"],
  "secret": "your-webhook-secret"
}
```

### 11.2 Event Payloads

All webhook payloads follow this structure:

```json
{
  "event": "avp.rfp.received",
  "timestamp": "2026-03-18T10:00:00Z",
  "agent_id": "avp:vendor:acme-tech:a1b2c3d4",
  "data": { },
  "signature": "sha256=..."
}
```

### 11.3 Event Types

| Event | Trigger |
|-------|---------|
| `avp.rfp.received` | New RFP received |
| `avp.rfp.responded` | Proposal sent |
| `avp.negotiation.started` | First negotiation round |
| `avp.negotiation.round` | Each negotiation round |
| `avp.proposal.accepted` | Terms accepted |
| `avp.proposal.rejected` | Terms rejected |
| `avp.settlement.reached` | Agreement finalized |
| `avp.session.expired` | Session timed out |
| `avp.trust.updated` | Trust score changed |
| `avp.escalation.requested` | Human escalation requested |

---

## 12. Rate Limits

### 12.1 Default Rate Limits

| Endpoint | Limit | Window |
|----------|-------|--------|
| `/.well-known/avp.json` | 100 requests | per hour |
| `/avp/handshake` | 20 requests | per hour |
| `/avp/rfp` | 10 requests | per hour |
| `/avp/negotiate` | 50 requests | per hour |
| `/avp/capabilities` | 30 requests | per hour |
| `/avp/status` | 60 requests | per hour |
| Registry search | 100 requests | per hour |

### 12.2 Rate Limit Headers

```http
X-RateLimit-Limit: 10
X-RateLimit-Remaining: 7
X-RateLimit-Reset: 1742259600
Retry-After: 1800
```

---

## 13. SDK Quick Reference

### 13.1 Node.js (Vendor)

```javascript
const { AVPVendor } = require('@avp/vendor-sdk');
const vendor = new AVPVendor(config);
vendor.on('rfp', handler);
vendor.on('negotiate', handler);
vendor.on('settle', handler);
vendor.listen(3000);
```

### 13.2 Node.js (Buyer)

```javascript
const { AVPBuyer } = require('@avp/buyer-sdk');
const buyer = new AVPBuyer(config);
const vendors = await buyer.search(criteria);
const proposals = await buyer.sendRfp(rfp, vendors);
const ranked = await buyer.compareProposals(proposals);
await buyer.negotiate(ranked[0], strategy);
await buyer.settle(ranked[0]);
```

### 13.3 Python (Vendor)

```python
from avp_vendor import AVPVendor
vendor = AVPVendor(config)

@vendor.on_rfp
async def handle_rfp(rfp, session): ...

@vendor.on_negotiate
async def handle_negotiate(counter, session): ...

vendor.run(port=3000)
```

### 13.4 cURL

```bash
# Discover
curl https://vendor.com/.well-known/avp.json

# Handshake
curl -X POST https://vendor.com/avp/handshake \
  -H "AVP-Version: 1.0" \
  -H "AVP-Agent-ID: avp:buyer:you:xxx" \
  -H "AVP-Timestamp: $(date -u +%Y-%m-%dT%H:%M:%SZ)" \
  -H "AVP-Signature: ed25519:..." \
  -H "Content-Type: application/json" \
  -d '{"type":"handshake_request",...}'
```

---

*End of API Reference*
