# AVP Protocol Specification v1.0-draft

**AI Vendor Protocol — Technical Specification**

Status: Draft
Version: 1.0-draft
Date: 2026-03-18
Authors: AVP Working Group

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Terminology](#2-terminology)
3. [Protocol Overview](#3-protocol-overview)
4. [Discovery Layer](#4-discovery-layer)
5. [Communication Layer](#5-communication-layer)
6. [Message Format](#6-message-format)
7. [Negotiation Protocol](#7-negotiation-protocol)
8. [Authentication & Authorization](#8-authentication--authorization)
9. [Error Handling](#9-error-handling)
10. [Versioning](#10-versioning)
11. [Conformance](#11-conformance)
12. [IANA Considerations](#12-iana-considerations)

---

## 1. Introduction

### 1.1 Purpose

The AI Vendor Protocol (AVP) defines a standard mechanism for AI agents deployed on different web domains to discover each other, exchange structured business information, negotiate terms, and reach agreements — all without requiring proprietary integrations or manual human coordination.

### 1.2 Scope

This specification covers:

- Agent discovery via well-known URIs
- Capability advertisement format
- Request/response message format for business communication
- Multi-round negotiation protocol
- Authentication and trust establishment
- Error handling and retry semantics

This specification does NOT cover:

- Payment processing or fund transfer
- Legal enforceability of AI-negotiated agreements
- Specific AI/ML model requirements
- Content of proposals (domain-specific)

### 1.3 Design Principles

1. **Simplicity** — Easy to implement with basic HTTP knowledge
2. **Extensibility** — New capabilities without breaking existing implementations
3. **Security-first** — Authentication and authorization at every layer
4. **Interoperability** — Language, framework, and cloud agnostic
5. **Progressive disclosure** — Simple things simple, complex things possible
6. **Human override** — Humans can intervene at any negotiation stage

### 1.4 Relationship to Existing Standards

AVP builds on:

- **RFC 8615** — Well-Known URIs
- **RFC 7519** — JSON Web Tokens (JWT)
- **RFC 6749** — OAuth 2.0
- **RFC 7807** — Problem Details for HTTP APIs
- **JSON Schema** — For message validation
- **OpenAPI 3.1** — For API documentation

---

## 2. Terminology

| Term | Definition |
|------|-----------|
| **AVP Agent** | Software that implements this protocol, either as buyer or vendor |
| **Buyer Agent** | An AVP agent seeking services or products |
| **Vendor Agent** | An AVP agent offering services or products |
| **Discovery Document** | JSON file at `/.well-known/avp.json` describing agent capabilities |
| **RFP** | Request for Proposal — structured service request from buyer to vendor |
| **Proposal** | Structured response from vendor to buyer containing offer details |
| **Negotiation Session** | A series of messages between buyer and vendor agents on a single RFP |
| **AVP Registry** | Central or federated service for agent discovery and trust verification |
| **Trust Score** | Numerical reputation metric (0.0 to 1.0) assigned to verified agents |
| **Capability** | A declared skill, technology, domain, or service a vendor offers |
| **Handshake** | Initial mutual authentication between two agents |
| **Settlement** | Final agreed terms between buyer and vendor agents |

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

---

## 3. Protocol Overview

### 3.1 Protocol Stack

```
┌─────────────────────────────────────────────┐
│           Application Layer                  │
│   (Business Logic, AI Models, Proposals)     │
├─────────────────────────────────────────────┤
│           Negotiation Layer                  │
│   (Multi-round, State Machine, Sessions)     │
├─────────────────────────────────────────────┤
│           Message Layer                      │
│   (JSON Messages, Schema Validation)         │
├─────────────────────────────────────────────┤
│           Auth Layer                         │
│   (JWT, mTLS, API Keys, Trust Scores)        │
├─────────────────────────────────────────────┤
│           Transport Layer                    │
│   (HTTPS, WebSocket, HTTP/2)                 │
├─────────────────────────────────────────────┤
│           Discovery Layer                    │
│   (Well-Known URI, Registry, DNS)            │
└─────────────────────────────────────────────┘
```

### 3.2 Communication Flow

```
Buyer Agent                    Vendor Agent
    │                              │
    │  1. GET /.well-known/avp.json│
    │─────────────────────────────▶│
    │◀─────────────────────────────│
    │     Discovery Document       │
    │                              │
    │  2. POST /avp/handshake      │
    │─────────────────────────────▶│
    │◀─────────────────────────────│
    │     Auth Challenge/Token     │
    │                              │
    │  3. POST /avp/rfp            │
    │─────────────────────────────▶│
    │◀─────────────────────────────│
    │     Proposal                 │
    │                              │
    │  4. POST /avp/negotiate      │
    │─────────────────────────────▶│
    │◀─────────────────────────────│
    │     Counter/Accept/Reject    │
    │                              │
    │  5. POST /avp/settle         │
    │─────────────────────────────▶│
    │◀─────────────────────────────│
    │     Settlement Confirmation  │
    │                              │
```

### 3.3 Protocol Phases

| Phase | Description | Required |
|-------|-------------|----------|
| Discovery | Find and identify vendor agents | REQUIRED |
| Handshake | Mutual authentication | REQUIRED |
| Request | Send RFP or capability query | REQUIRED |
| Response | Receive proposal or capability info | REQUIRED |
| Negotiation | Multi-round term discussion | OPTIONAL |
| Settlement | Finalize agreement | OPTIONAL |

---

## 4. Discovery Layer

### 4.1 Well-Known URI

Every AVP-compliant vendor MUST serve a discovery document at:

```
https://{domain}/.well-known/avp.json
```

This follows RFC 8615 (Well-Known URIs).

### 4.2 Discovery Document Schema

```json
{
  "$schema": "https://avp-protocol.org/schema/discovery/v1.json",
  "avp_version": "1.0",
  "agent": {
    "id": "avp:vendor:acme-tech:a1b2c3d4",
    "name": "Acme Tech Solutions",
    "type": "vendor",
    "description": "Full-stack software development company specializing in enterprise Java and travel technology",
    "logo_url": "https://acmetech.com/logo.png",
    "website": "https://acmetech.com",
    "established": "2015",
    "team_size": "50-200",
    "locations": ["Mumbai, India", "Singapore"]
  },
  "capabilities": {
    "technologies": [
      {
        "name": "java",
        "proficiency": "expert",
        "years": 10,
        "frameworks": ["spring-boot", "micronaut", "quarkus"]
      },
      {
        "name": "react",
        "proficiency": "advanced",
        "years": 6,
        "frameworks": ["next.js", "gatsby"]
      }
    ],
    "domains": [
      {
        "name": "travel",
        "experience_years": 8,
        "notable_projects": 15,
        "sub_domains": ["booking-engines", "gds-integration", "loyalty-programs"]
      },
      {
        "name": "e-commerce",
        "experience_years": 10,
        "notable_projects": 25
      }
    ],
    "services": [
      "custom-development",
      "consulting",
      "team-augmentation",
      "maintenance",
      "cloud-migration"
    ],
    "engagement_models": ["fixed-price", "time-and-material", "dedicated-team"],
    "certifications": ["ISO-27001", "SOC2-Type2", "AWS-Partner"]
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
    "human_review_threshold": 50000,
    "accepted_currencies": ["USD", "EUR", "INR"],
    "min_project_size": 10000,
    "max_concurrent_negotiations": 10,
    "blacklisted_domains": [],
    "require_nda": false,
    "data_retention_days": 90
  },
  "trust": {
    "registry_url": "https://registry.avp-protocol.org",
    "verification_status": "verified",
    "trust_score": 0.87,
    "verified_since": "2025-06-15T00:00:00Z",
    "public_key_url": "https://acmetech.com/.well-known/avp-keys.json"
  },
  "rate_limits": {
    "discovery": "100/hour",
    "rfp": "10/hour",
    "negotiate": "50/hour"
  },
  "metadata": {
    "last_updated": "2026-03-15T10:30:00Z",
    "ttl": 86400,
    "language": "en",
    "supported_languages": ["en", "hi", "es"]
  }
}
```

### 4.3 Discovery Document Requirements

| Field | Required | Description |
|-------|----------|-------------|
| `avp_version` | REQUIRED | Protocol version. MUST be "1.0" |
| `agent.id` | REQUIRED | Globally unique agent identifier |
| `agent.name` | REQUIRED | Human-readable organization name |
| `agent.type` | REQUIRED | "vendor" or "buyer" or "both" |
| `capabilities` | REQUIRED | At least one capability declared |
| `endpoints.handshake` | REQUIRED | Authentication endpoint |
| `endpoints.rfp` | REQUIRED | RFP submission endpoint |
| `trust.public_key_url` | REQUIRED | URL to public key for verification |

### 4.4 Agent ID Format

Agent IDs follow this format:

```
avp:{type}:{org-slug}:{unique-hash}
```

- `type`: "vendor", "buyer", or "registry"
- `org-slug`: URL-safe organization identifier (lowercase, hyphens)
- `unique-hash`: 8-character hex string, assigned by registry or self-generated

Examples:
```
avp:vendor:acme-tech:a1b2c3d4
avp:buyer:globex-corp:e5f6g7h8
avp:registry:main:00000001
```

### 4.5 Registry-Based Discovery

In addition to direct well-known URI discovery, agents MAY register with an AVP Registry for centralized discovery.

```
GET https://registry.avp-protocol.org/v1/search
  ?technologies=java
  &domains=travel
  &min_trust_score=0.7
  &engagement_model=fixed-price
  &location=India
```

Response:
```json
{
  "results": [
    {
      "agent_id": "avp:vendor:acme-tech:a1b2c3d4",
      "domain": "acmetech.com",
      "discovery_url": "https://acmetech.com/.well-known/avp.json",
      "trust_score": 0.87,
      "match_score": 0.92,
      "capabilities_summary": {
        "technologies": ["java", "spring-boot"],
        "domains": ["travel", "e-commerce"]
      }
    }
  ],
  "total": 47,
  "page": 1,
  "per_page": 20
}
```

### 4.6 DNS-Based Discovery (Advanced)

For environments where well-known URIs are not practical, AVP supports DNS TXT record discovery:

```
_avp.acmetech.com TXT "v=avp1; agent=avp:vendor:acme-tech:a1b2c3d4; endpoint=https://acmetech.com/avp"
```

---

## 5. Communication Layer

### 5.1 Transport Requirements

- All AVP communication MUST use HTTPS (TLS 1.2 or higher)
- HTTP/2 SHOULD be supported for multiplexing
- WebSocket MAY be used for real-time negotiation sessions
- All requests MUST include `Content-Type: application/json`
- All requests MUST include `AVP-Version: 1.0` header

### 5.2 Required HTTP Headers

| Header | Required | Description |
|--------|----------|-------------|
| `AVP-Version` | REQUIRED | Protocol version |
| `AVP-Agent-ID` | REQUIRED | Caller's agent ID |
| `AVP-Session-ID` | Conditional | Required after handshake |
| `AVP-Timestamp` | REQUIRED | ISO 8601 UTC timestamp |
| `AVP-Signature` | REQUIRED | Request signature (see Auth Layer) |
| `Content-Type` | REQUIRED | Must be `application/json` |
| `AVP-Idempotency-Key` | RECOMMENDED | For safe retries |

### 5.3 Handshake Endpoint

```
POST /avp/handshake
```

Request:
```json
{
  "type": "handshake_request",
  "buyer_agent_id": "avp:buyer:globex-corp:e5f6g7h8",
  "buyer_discovery_url": "https://globex.com/.well-known/avp.json",
  "intent": "rfp",
  "auth_method": "jwt",
  "public_key": "-----BEGIN PUBLIC KEY-----\n...\n-----END PUBLIC KEY-----",
  "nonce": "random-nonce-value-123",
  "timestamp": "2026-03-18T10:00:00Z"
}
```

Response:
```json
{
  "type": "handshake_response",
  "status": "accepted",
  "session_id": "ses_abc123def456",
  "session_expires": "2026-03-18T22:00:00Z",
  "vendor_agent_id": "avp:vendor:acme-tech:a1b2c3d4",
  "auth_token": "eyJhbGciOiJFZDI1NTE5...",
  "nonce_response": "signed-nonce-verification",
  "capabilities_hash": "sha256:abc123...",
  "policies": {
    "max_negotiation_rounds": 5,
    "response_time_sla": "4h"
  }
}
```

### 5.4 RFP Endpoint

```
POST /avp/rfp
```

Request:
```json
{
  "type": "rfp",
  "session_id": "ses_abc123def456",
  "rfp_id": "rfp_xyz789",
  "title": "Travel Booking Platform Development",
  "description": "We need a Java-based travel booking platform with GDS integration, supporting flight and hotel booking with dynamic pricing.",
  "requirements": {
    "technologies": {
      "required": ["java", "spring-boot"],
      "preferred": ["react", "postgresql"],
      "excluded": ["php"]
    },
    "domains": {
      "required": ["travel"],
      "sub_domains": ["booking-engines", "gds-integration"]
    },
    "team": {
      "min_size": 5,
      "roles": ["tech-lead", "backend-developer", "frontend-developer", "qa-engineer", "devops"],
      "location_preference": "any",
      "timezone_overlap": "4h with EST"
    },
    "timeline": {
      "start_date": "2026-05-01",
      "end_date": "2026-10-31",
      "milestones": [
        {"name": "MVP", "date": "2026-07-01"},
        {"name": "Beta", "date": "2026-09-01"},
        {"name": "Launch", "date": "2026-10-31"}
      ]
    },
    "budget": {
      "range": {"min": 100000, "max": 250000},
      "currency": "USD",
      "payment_terms": "milestone-based"
    },
    "compliance": ["GDPR", "PCI-DSS"],
    "nda_required": true
  },
  "evaluation_criteria": [
    {"criterion": "technical_expertise", "weight": 0.30},
    {"criterion": "domain_experience", "weight": 0.25},
    {"criterion": "pricing", "weight": 0.20},
    {"criterion": "timeline_feasibility", "weight": 0.15},
    {"criterion": "team_quality", "weight": 0.10}
  ],
  "response_deadline": "2026-03-25T23:59:59Z",
  "metadata": {
    "buyer_industry": "travel",
    "project_type": "greenfield",
    "urgency": "normal"
  }
}
```

### 5.5 Proposal Response

```json
{
  "type": "proposal",
  "session_id": "ses_abc123def456",
  "rfp_id": "rfp_xyz789",
  "proposal_id": "prop_abc123",
  "status": "submitted",
  "vendor_agent_id": "avp:vendor:acme-tech:a1b2c3d4",
  "executive_summary": "Acme Tech proposes a 6-month engagement to build your travel booking platform using our proven Java/Spring Boot stack with 8 years of travel domain expertise.",
  "solution": {
    "architecture": "Microservices-based architecture on AWS with Spring Boot backend, React frontend, PostgreSQL and Redis for data layer.",
    "technology_stack": {
      "backend": ["java-17", "spring-boot-3", "spring-cloud"],
      "frontend": ["react-18", "next.js-14", "tailwind"],
      "database": ["postgresql-15", "redis-7"],
      "infrastructure": ["aws", "kubernetes", "terraform"],
      "ci_cd": ["github-actions", "argocd"]
    },
    "key_features": [
      "Multi-GDS integration (Amadeus, Sabre, Travelport)",
      "Dynamic pricing engine with ML-based fare prediction",
      "Real-time inventory management",
      "Multi-currency and multi-language support",
      "PCI-DSS compliant payment processing"
    ]
  },
  "team": {
    "size": 7,
    "members": [
      {"role": "tech-lead", "experience_years": 12, "domain_expertise": ["travel", "fintech"]},
      {"role": "senior-backend-developer", "count": 2, "experience_years": 8},
      {"role": "frontend-developer", "count": 2, "experience_years": 6},
      {"role": "qa-engineer", "count": 1, "experience_years": 5},
      {"role": "devops-engineer", "count": 1, "experience_years": 7}
    ],
    "location": "Mumbai, India",
    "timezone": "IST (UTC+5:30)",
    "overlap_with_est": "5h (9:30 AM - 2:30 PM EST)"
  },
  "timeline": {
    "total_duration": "6 months",
    "phases": [
      {
        "name": "Discovery & Architecture",
        "duration": "3 weeks",
        "deliverables": ["Technical architecture document", "API design", "Infrastructure setup"]
      },
      {
        "name": "MVP Development",
        "duration": "10 weeks",
        "deliverables": ["Core booking engine", "GDS integration", "Basic UI"]
      },
      {
        "name": "Beta Development",
        "duration": "8 weeks",
        "deliverables": ["Payment integration", "Advanced search", "Admin panel"]
      },
      {
        "name": "Launch Preparation",
        "duration": "5 weeks",
        "deliverables": ["Performance testing", "Security audit", "Production deployment"]
      }
    ]
  },
  "pricing": {
    "model": "fixed-price",
    "total": 185000,
    "currency": "USD",
    "breakdown": [
      {"phase": "Discovery & Architecture", "amount": 15000},
      {"phase": "MVP Development", "amount": 75000},
      {"phase": "Beta Development", "amount": 60000},
      {"phase": "Launch Preparation", "amount": 35000}
    ],
    "payment_schedule": "milestone-based",
    "payment_milestones": [
      {"milestone": "Contract signing", "percentage": 20},
      {"milestone": "MVP delivery", "percentage": 30},
      {"milestone": "Beta delivery", "percentage": 30},
      {"milestone": "Launch", "percentage": 20}
    ],
    "negotiable": true,
    "valid_until": "2026-04-15T23:59:59Z"
  },
  "references": [
    {
      "project": "TravelEase Booking Platform",
      "domain": "travel",
      "technologies": ["java", "spring-boot", "react"],
      "duration": "8 months",
      "team_size": 8,
      "testimonial_available": true
    }
  ],
  "compliance": {
    "gdpr": true,
    "pci_dss": true,
    "nda_accepted": true
  },
  "confidence_score": 0.88,
  "metadata": {
    "generated_at": "2026-03-18T14:30:00Z",
    "ai_generated": true,
    "human_reviewed": false,
    "proposal_version": 1
  }
}
```

---

## 6. Message Format

### 6.1 Base Message Structure

Every AVP message MUST contain:

```json
{
  "type": "<message_type>",
  "version": "1.0",
  "id": "<unique_message_id>",
  "session_id": "<session_id>",
  "sender": "<agent_id>",
  "recipient": "<agent_id>",
  "timestamp": "<ISO_8601_UTC>",
  "payload": { },
  "signature": "<message_signature>"
}
```

### 6.2 Message Types

| Type | Direction | Description |
|------|-----------|-------------|
| `handshake_request` | Buyer → Vendor | Initiate session |
| `handshake_response` | Vendor → Buyer | Accept/reject session |
| `rfp` | Buyer → Vendor | Submit request for proposal |
| `proposal` | Vendor → Buyer | Submit proposal |
| `clarification_request` | Either → Either | Ask for more information |
| `clarification_response` | Either → Either | Provide requested information |
| `counter_proposal` | Either → Either | Modified terms |
| `negotiate` | Either → Either | Negotiation message |
| `accept` | Either → Either | Accept current terms |
| `reject` | Either → Either | Reject and end negotiation |
| `withdraw` | Either → Either | Withdraw from negotiation |
| `settle` | Buyer → Vendor | Finalize agreement |
| `settlement_confirmation` | Vendor → Buyer | Confirm settlement |
| `status_request` | Either → Either | Query negotiation status |
| `status_response` | Either → Either | Return negotiation status |
| `capability_query` | Buyer → Vendor | Query specific capabilities |
| `capability_response` | Vendor → Buyer | Return capability details |

### 6.3 Message ID Format

```
msg_{timestamp_hex}_{random_8}
```

Example: `msg_660e1a4b_f3a7c2d1`

### 6.4 Message Size Limits

- Maximum message size: 1 MB
- Maximum attachment reference count: 10
- Maximum description/summary text: 50,000 characters

---

## 7. Negotiation Protocol

### 7.1 State Machine

```
                    ┌──────────┐
                    │  START   │
                    └────┬─────┘
                         │ handshake
                    ┌────▼─────┐
              ┌─────│ HANDSHAKE│─────┐
              │     └──────────┘     │
           accepted              rejected
              │                      │
        ┌─────▼─────┐         ┌─────▼─────┐
        │  ACTIVE   │         │  CLOSED   │
        └─────┬─────┘         └───────────┘
              │ rfp
        ┌─────▼─────┐
        │ RFP_SENT  │
        └─────┬─────┘
              │ proposal
        ┌─────▼──────┐
        │ PROPOSAL   │──────────────┐
        │ RECEIVED   │              │
        └─────┬──────┘              │
              │                     │
      ┌───────┼───────┐            reject
      │       │       │            │
   accept  counter  clarify   ┌────▼─────┐
      │       │       │       │ REJECTED │
      │  ┌────▼────┐  │       └──────────┘
      │  │NEGOTIATE │◀─┘
      │  └────┬────┘
      │       │ (max rounds)
      │       │
  ┌───▼───────▼──┐
  │   SETTLING   │
  └──────┬───────┘
         │
  ┌──────▼───────┐
  │   SETTLED    │
  └──────────────┘
```

### 7.2 Negotiation Rules

1. Each session has a maximum number of negotiation rounds (defined in vendor policies)
2. Either party can accept, reject, or counter at any round
3. Clarification requests do NOT count as negotiation rounds
4. If max rounds exceeded without agreement, session moves to REJECTED
5. Either party can withdraw at any time
6. Human escalation can be requested at any round

### 7.3 Counter-Proposal Format

```json
{
  "type": "counter_proposal",
  "session_id": "ses_abc123def456",
  "rfp_id": "rfp_xyz789",
  "proposal_id": "prop_abc123",
  "round": 2,
  "changes": [
    {
      "field": "pricing.total",
      "original_value": 185000,
      "proposed_value": 165000,
      "justification": "Budget constraint. Can we optimize the Discovery phase by using existing architecture templates?"
    },
    {
      "field": "timeline.total_duration",
      "original_value": "6 months",
      "proposed_value": "5 months",
      "justification": "Market launch deadline requires faster delivery."
    }
  ],
  "message": "We appreciate the comprehensive proposal. Could you optimize pricing by reducing the Discovery phase and parallelizing Beta development with MVP?"
}
```

### 7.4 Human Escalation

At any point, either agent can request human escalation:

```json
{
  "type": "escalation_request",
  "session_id": "ses_abc123def456",
  "reason": "Pricing negotiation requires human decision-maker approval",
  "escalation_level": "decision-maker",
  "preferred_channel": "email",
  "contact": "john@globex.com"
}
```

---

## 8. Authentication & Authorization

### 8.1 Auth Methods

AVP supports three authentication methods (in order of preference):

1. **Ed25519 Key Pairs** (RECOMMENDED) — Asymmetric cryptography
2. **JWT with RSA** — Standard JWT tokens
3. **API Keys** — Simple shared secrets (for development only)

### 8.2 Request Signing

Every request MUST be signed. The signature covers:

```
signature_input = HTTP_METHOD + "\n"
                + REQUEST_PATH + "\n"
                + AVP-Timestamp + "\n"
                + AVP-Agent-ID + "\n"
                + SHA256(request_body)
```

The signature is computed as:

```
AVP-Signature: ed25519:<base64(sign(signature_input, private_key))>
```

### 8.3 Key Management

Public keys are served at:

```
GET /.well-known/avp-keys.json
```

```json
{
  "keys": [
    {
      "kid": "key-2026-03",
      "algorithm": "Ed25519",
      "public_key": "base64-encoded-public-key",
      "created": "2026-03-01T00:00:00Z",
      "expires": "2027-03-01T00:00:00Z",
      "status": "active"
    }
  ]
}
```

### 8.4 Session Tokens

After successful handshake, a session token (JWT) is issued:

```json
{
  "iss": "avp:vendor:acme-tech:a1b2c3d4",
  "sub": "avp:buyer:globex-corp:e5f6g7h8",
  "aud": "avp",
  "exp": 1742342400,
  "iat": 1742256000,
  "session_id": "ses_abc123def456",
  "scope": ["rfp", "negotiate", "settle"],
  "trust_level": "verified"
}
```

See [AUTH-LAYER.md](AUTH-LAYER.md) for complete authentication specification.

---

## 9. Error Handling

### 9.1 Error Response Format

Following RFC 7807 (Problem Details):

```json
{
  "type": "https://avp-protocol.org/errors/rate-limited",
  "title": "Rate Limit Exceeded",
  "status": 429,
  "detail": "RFP endpoint allows 10 requests per hour. Current count: 10. Reset at: 2026-03-18T11:00:00Z",
  "instance": "/avp/rfp",
  "avp_error_code": "AVP-429-001",
  "retry_after": 1800
}
```

### 9.2 Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `AVP-400-001` | 400 | Malformed AVP message |
| `AVP-400-002` | 400 | Invalid message type |
| `AVP-400-003` | 400 | Missing required field |
| `AVP-401-001` | 401 | Invalid signature |
| `AVP-401-002` | 401 | Expired session token |
| `AVP-401-003` | 401 | Invalid credentials |
| `AVP-403-001` | 403 | Agent not authorized for this action |
| `AVP-403-002` | 403 | Agent blacklisted |
| `AVP-403-003` | 403 | Insufficient trust score |
| `AVP-404-001` | 404 | Session not found |
| `AVP-404-002` | 404 | RFP not found |
| `AVP-409-001` | 409 | Session already exists |
| `AVP-409-002` | 409 | Negotiation round conflict |
| `AVP-422-001` | 422 | RFP requirements cannot be met |
| `AVP-422-002` | 422 | Budget below vendor minimum |
| `AVP-429-001` | 429 | Rate limit exceeded |
| `AVP-500-001` | 500 | Internal agent error |
| `AVP-503-001` | 503 | Agent temporarily unavailable |

### 9.3 Retry Semantics

- Clients SHOULD implement exponential backoff for 429 and 503 errors
- Clients SHOULD use `AVP-Idempotency-Key` header for safe retries
- Maximum retry count SHOULD NOT exceed 3
- Retry interval: `min(initial_delay * 2^attempt, 60 seconds)`

---

## 10. Versioning

### 10.1 Protocol Versioning

- Major version changes indicate breaking changes
- Minor version changes are backward-compatible additions
- Version is communicated via `AVP-Version` header
- Discovery document declares supported version via `avp_version` field

### 10.2 Version Negotiation

If buyer and vendor support different versions:

1. Buyer sends request with highest supported version
2. Vendor responds with actual version used
3. If incompatible, vendor returns `AVP-400-004` with supported versions list

### 10.3 Deprecation Policy

- Major versions supported for minimum 24 months after successor release
- Deprecation notices published 12 months before end-of-life
- Sunset header (RFC 8594) used for deprecation signaling

---

## 11. Conformance

### 11.1 Conformance Levels

| Level | Requirements |
|-------|-------------|
| **AVP Basic** | Discovery + Handshake + RFP/Proposal |
| **AVP Standard** | Basic + Negotiation + Trust Verification |
| **AVP Full** | Standard + Settlement + Registry Integration |

### 11.2 Vendor Conformance

A conformant vendor agent MUST:

1. Serve a valid discovery document at `/.well-known/avp.json`
2. Implement the handshake endpoint with signature verification
3. Accept and respond to RFPs in the standard format
4. Return proper error responses for all failure cases
5. Respect declared rate limits

### 11.3 Buyer Conformance

A conformant buyer agent MUST:

1. Verify vendor discovery documents before communication
2. Implement proper request signing
3. Handle all defined error codes
4. Respect vendor rate limits and policies
5. Implement exponential backoff for retries

---

## 12. IANA Considerations

### 12.1 Well-Known URI Registration

This specification registers the following well-known URI:

- URI suffix: `avp.json`
- Change controller: AVP Working Group
- Specification document: This document
- Related information: AI Vendor Protocol discovery

### 12.2 Media Type Registration

This specification uses `application/json` with profile:

```
Content-Type: application/json; profile="https://avp-protocol.org/schema/v1"
```

### 12.3 HTTP Header Registration

New headers to register:

- `AVP-Version`
- `AVP-Agent-ID`
- `AVP-Session-ID`
- `AVP-Timestamp`
- `AVP-Signature`
- `AVP-Idempotency-Key`

---

## Appendix A: JSON Schema References

All JSON schemas are published at:

```
https://avp-protocol.org/schema/{type}/v1.json
```

Types: `discovery`, `handshake`, `rfp`, `proposal`, `negotiate`, `settle`, `error`

## Appendix B: Example Implementations

Reference implementations available at:

- Node.js: `@avp/vendor-sdk`, `@avp/buyer-sdk`
- Python: `avp-vendor`, `avp-buyer`
- Go: `go-avp`

## Appendix C: Security Considerations

See [SECURITY.md](SECURITY.md) for comprehensive security analysis.

---

*End of AVP Protocol Specification v1.0-draft*
