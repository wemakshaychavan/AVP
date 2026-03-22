# AVP Authentication & Authorization Layer

**Complete Security Architecture for Agent Identity, Authentication, and Access Control**

Version: 1.0-draft
Date: 2026-03-18

---

## Table of Contents

1. [Overview](#1-overview)
2. [Identity Model](#2-identity-model)
3. [Authentication Methods](#3-authentication-methods)
4. [Key Management](#4-key-management)
5. [Session Management](#5-session-management)
6. [Authorization Model](#6-authorization-model)
7. [Request Signing](#7-request-signing)
8. [Token Specifications](#8-token-specifications)
9. [Trust Delegation](#9-trust-delegation)
10. [Revocation](#10-revocation)

---

## 1. Overview

### 1.1 Design Goals

- **Zero-trust by default** — Every request is authenticated and verified
- **Decentralized identity** — Agents own their identity, not a central authority
- **Minimal friction** — Simple API key for development, full crypto for production
- **Auditability** — Every action is signed and traceable
- **Revocability** — Compromised agents can be instantly blocked

### 1.2 Auth Flow Summary

```
┌──────────┐         ┌──────────┐         ┌──────────────┐
│  Buyer   │         │  Vendor  │         │    Trust      │
│  Agent   │         │  Agent   │         │   Authority   │
└────┬─────┘         └────┬─────┘         └──────┬───────┘
     │                    │                       │
     │ 1. Generate key pair                       │
     │─────────────────────────────────────────▶  │
     │                    │    2. Register agent   │
     │ 3. Receive AVP cert◀──────────────────────│
     │                    │                       │
     │ 4. GET /.well-known/avp.json               │
     │───────────────────▶│                       │
     │ 5. Discovery doc   │                       │
     │◀───────────────────│                       │
     │                    │                       │
     │ 6. Fetch vendor public key                 │
     │───────────────────▶│                       │
     │ 7. Public key      │                       │
     │◀───────────────────│                       │
     │                    │                       │
     │ 8. Verify with Trust Authority             │
     │─────────────────────────────────────────▶  │
     │ 9. Trust score + validity                  │
     │◀─────────────────────────────────────────  │
     │                    │                       │
     │ 10. POST /avp/handshake (signed)           │
     │───────────────────▶│                       │
     │                    │ 11. Verify buyer sig   │
     │                    │ 12. Verify buyer cert  │
     │                    │───────────────────────▶│
     │                    │ 13. Confirmation       │
     │                    │◀───────────────────────│
     │ 14. Session token  │                       │
     │◀───────────────────│                       │
     │                    │                       │
     │ 15. All subsequent requests use            │
     │     session token + request signing        │
     │                    │                       │
```

---

## 2. Identity Model

### 2.1 Agent Identity Structure

Every AVP agent has a globally unique identity:

```json
{
  "agent_id": "avp:vendor:acme-tech:a1b2c3d4",
  "display_name": "Acme Tech Solutions",
  "domain": "acmetech.com",
  "organization": {
    "legal_name": "Acme Tech Solutions Pvt. Ltd.",
    "registration_number": "U72200MH2015PTC123456",
    "country": "IN",
    "verified": true
  },
  "keys": {
    "current": "kid:key-2026-03",
    "algorithm": "Ed25519"
  },
  "trust": {
    "score": 0.87,
    "level": "verified",
    "verified_by": "avp:registry:main:00000001"
  },
  "created": "2025-06-15T00:00:00Z",
  "status": "active"
}
```

### 2.2 Identity Levels

| Level | Requirements | Capabilities |
|-------|-------------|--------------|
| **Unverified** | Self-registration only | Discovery only, cannot receive RFPs |
| **Domain-Verified** | DNS TXT record + domain ownership proof | Can participate in communication, limited trust |
| **Business-Verified** | Legal documents + domain + human review | Full protocol access, higher trust score |
| **Premium-Verified** | Business-verified + financial audit + references | Priority in search, trust badge, higher limits |

### 2.3 Identity Registration

```
POST https://registry.avp-protocol.org/v1/agents/register

{
  "agent_type": "vendor",
  "organization_name": "Acme Tech Solutions",
  "domain": "acmetech.com",
  "public_key": "base64-encoded-ed25519-public-key",
  "contact_email": "admin@acmetech.com",
  "verification_level": "business-verified"
}
```

Response:
```json
{
  "agent_id": "avp:vendor:acme-tech:a1b2c3d4",
  "status": "pending_verification",
  "verification_steps": [
    {
      "step": "domain_verification",
      "method": "dns_txt",
      "record": "_avp-verify.acmetech.com TXT avp-verify=xyz789",
      "status": "pending"
    },
    {
      "step": "business_verification",
      "method": "document_upload",
      "required_documents": ["business_registration", "tax_id"],
      "upload_url": "https://registry.avp-protocol.org/v1/verify/upload/a1b2c3d4",
      "status": "pending"
    }
  ]
}
```

---

## 3. Authentication Methods

### 3.1 Method 1: Ed25519 Key Pairs (RECOMMENDED)

The primary authentication method. Each agent generates an Ed25519 key pair.

**Key Generation:**
```javascript
const { generateKeyPair } = require('@avp/auth');

const keys = await generateKeyPair();
// keys.publicKey  — share via /.well-known/avp-keys.json
// keys.privateKey — keep secret, use for signing
```

**Request Authentication:**
```
POST /avp/rfp HTTP/1.1
Host: vendor.com
AVP-Version: 1.0
AVP-Agent-ID: avp:buyer:globex-corp:e5f6g7h8
AVP-Timestamp: 2026-03-18T10:00:00Z
AVP-Signature: ed25519:base64(sign(canonical_request, private_key))
AVP-Key-ID: key-2026-03
Content-Type: application/json

{ ... request body ... }
```

### 3.2 Method 2: JWT with RSA-256

For environments where Ed25519 is not available.

```
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
```

JWT Claims:
```json
{
  "iss": "avp:buyer:globex-corp:e5f6g7h8",
  "sub": "avp:vendor:acme-tech:a1b2c3d4",
  "aud": "avp",
  "exp": 1742259600,
  "iat": 1742256000,
  "jti": "unique-token-id",
  "avp_version": "1.0",
  "scope": ["rfp", "negotiate"]
}
```

### 3.3 Method 3: API Keys (Development Only)

For local development and testing:

```
AVP-API-Key: avpk_test_abc123def456
```

**API keys MUST NOT be used in production.** They are provided only for:
- Local development
- Testing environments
- Initial integration testing

### 3.4 Method Negotiation

During handshake, agents negotiate the auth method:

```json
{
  "type": "handshake_request",
  "auth_methods_supported": ["ed25519", "jwt-rsa256"],
  "preferred_method": "ed25519"
}
```

```json
{
  "type": "handshake_response",
  "auth_method_selected": "ed25519"
}
```

---

## 4. Key Management

### 4.1 Key Lifecycle

```
Generate → Register → Active → Rotate → Deprecated → Revoked
    │                    │                     │
    │                    │ (overlap period)     │
    │                    ▼                     │
    │              New Key Active              │
    │                                          │
    └── Emergency Revoke ──────────────────────┘
```

### 4.2 Key Publication

Public keys are served at:

```
GET https://acmetech.com/.well-known/avp-keys.json
```

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
    },
    {
      "kid": "key-2025-09",
      "algorithm": "Ed25519",
      "public_key": "MCowBQYDK2VwAyEA...",
      "status": "deprecated",
      "created": "2025-09-01T00:00:00Z",
      "expires": "2026-06-01T00:00:00Z",
      "deprecation_note": "Rotated to key-2026-03"
    }
  ]
}
```

### 4.3 Key Rotation Policy

| Requirement | Policy |
|-------------|--------|
| Rotation frequency | Every 12 months (RECOMMENDED), every 24 months (REQUIRED) |
| Overlap period | Minimum 30 days where old and new key both valid |
| Emergency rotation | Immediate; old key revoked, new key published |
| Key size | Ed25519: 256-bit; RSA: minimum 2048-bit |
| Storage | Private keys MUST be stored in HSM or secure vault |

### 4.4 Key Caching

Verifiers SHOULD cache public keys with:
- Cache duration: Minimum of `Cache-Control` header or 1 hour
- Maximum cache: 24 hours
- Cache invalidation: On signature verification failure, re-fetch keys

---

## 5. Session Management

### 5.1 Session Creation

Sessions are created during the handshake:

```json
{
  "session_id": "ses_abc123def456",
  "buyer_agent_id": "avp:buyer:globex-corp:e5f6g7h8",
  "vendor_agent_id": "avp:vendor:acme-tech:a1b2c3d4",
  "created": "2026-03-18T10:00:00Z",
  "expires": "2026-03-18T22:00:00Z",
  "auth_method": "ed25519",
  "scope": ["rfp", "negotiate", "settle"],
  "status": "active"
}
```

### 5.2 Session Properties

| Property | Value |
|----------|-------|
| Default TTL | 12 hours |
| Maximum TTL | 7 days |
| Renewable | Yes, up to maximum TTL |
| Concurrent sessions per pair | Maximum 5 |
| Session binding | Bound to agent IDs + IP range (optional) |

### 5.3 Session Token (JWT)

```json
{
  "header": {
    "alg": "EdDSA",
    "typ": "JWT",
    "kid": "key-2026-03"
  },
  "payload": {
    "iss": "avp:vendor:acme-tech:a1b2c3d4",
    "sub": "avp:buyer:globex-corp:e5f6g7h8",
    "aud": "avp",
    "exp": 1742299200,
    "iat": 1742256000,
    "session_id": "ses_abc123def456",
    "scope": ["rfp", "negotiate", "settle"],
    "trust_level": "business-verified",
    "ip_binding": "203.0.113.0/24"
  }
}
```

### 5.4 Session Renewal

```
POST /avp/session/renew
AVP-Session-ID: ses_abc123def456

{
  "extend_by": 3600
}
```

---

## 6. Authorization Model

### 6.1 Permission Matrix

| Action | Unverified | Domain-Verified | Business-Verified | Premium |
|--------|-----------|----------------|-------------------|---------|
| Discovery (read) | ✅ | ✅ | ✅ | ✅ |
| Handshake | ❌ | ✅ | ✅ | ✅ |
| Send RFP | ❌ | ✅ (5/day) | ✅ (50/day) | ✅ (unlimited) |
| Receive RFP | ❌ | ✅ | ✅ | ✅ |
| Negotiate | ❌ | ✅ | ✅ | ✅ |
| Settle | ❌ | ❌ | ✅ | ✅ |
| Registry search | ✅ (10/day) | ✅ (100/day) | ✅ (1000/day) | ✅ (unlimited) |
| Trust score access | ❌ | ✅ (own) | ✅ (all) | ✅ (all + history) |

### 6.2 Scope-Based Access

Each session has a declared scope. Agents MUST NOT perform actions outside their session scope.

Available scopes:
- `discovery` — Read discovery documents
- `capabilities` — Query detailed capabilities
- `rfp` — Submit and receive RFPs
- `negotiate` — Participate in negotiation rounds
- `settle` — Finalize agreements
- `trust` — Access trust information
- `admin` — Administrative operations (registry only)

### 6.3 Policy Enforcement

Vendor agents can define fine-grained access policies:

```json
{
  "policies": {
    "require_minimum_trust": 0.5,
    "require_verification_level": "business-verified",
    "blacklisted_agents": ["avp:buyer:spam-corp:xxxxxxxx"],
    "whitelisted_domains": [],
    "geo_restrictions": ["US", "EU", "IN"],
    "business_hours_only": {
      "enabled": true,
      "timezone": "Asia/Kolkata",
      "hours": "09:00-18:00",
      "days": ["Mon", "Tue", "Wed", "Thu", "Fri"]
    },
    "human_approval_required_above": {
      "amount": 50000,
      "currency": "USD"
    }
  }
}
```

---

## 7. Request Signing

### 7.1 Canonical Request Format

To create a signature, construct the canonical request:

```
canonical_request = HTTP_METHOD + "\n"
                  + CANONICAL_PATH + "\n"
                  + CANONICAL_QUERY + "\n"
                  + AVP_TIMESTAMP + "\n"
                  + AVP_AGENT_ID + "\n"
                  + AVP_SESSION_ID + "\n"
                  + SHA256(REQUEST_BODY)
```

### 7.2 Signing Process

```
1. Construct canonical request
2. Compute SHA-256 hash of canonical request
3. Sign hash with Ed25519 private key
4. Base64-encode signature
5. Set header: AVP-Signature: ed25519:<base64_signature>
```

### 7.3 Verification Process

```
1. Extract AVP-Signature header
2. Extract AVP-Key-ID header
3. Fetch signer's public key (from cache or /.well-known/avp-keys.json)
4. Reconstruct canonical request from received request
5. Verify signature using public key
6. Check timestamp is within 5-minute window (clock skew tolerance)
7. Check agent ID matches registered identity
```

### 7.4 Example

```python
import hashlib
import base64
from nacl.signing import SigningKey

# Construct canonical request
method = "POST"
path = "/avp/rfp"
query = ""
timestamp = "2026-03-18T10:00:00Z"
agent_id = "avp:buyer:globex-corp:e5f6g7h8"
session_id = "ses_abc123def456"
body_hash = hashlib.sha256(request_body.encode()).hexdigest()

canonical = f"{method}\n{path}\n{query}\n{timestamp}\n{agent_id}\n{session_id}\n{body_hash}"

# Sign
signing_key = SigningKey(private_key_bytes)
signature = signing_key.sign(canonical.encode()).signature
signature_b64 = base64.b64encode(signature).decode()

# Header
headers["AVP-Signature"] = f"ed25519:{signature_b64}"
```

---

## 8. Token Specifications

### 8.1 Registration Token

Issued by registry after agent registration:

```json
{
  "type": "registration_token",
  "agent_id": "avp:vendor:acme-tech:a1b2c3d4",
  "issued_by": "avp:registry:main:00000001",
  "issued_at": "2026-03-18T00:00:00Z",
  "verification_level": "business-verified",
  "capabilities_hash": "sha256:abc123...",
  "expires": "2027-03-18T00:00:00Z"
}
```

### 8.2 Session Token

See Section 5.3.

### 8.3 Delegation Token

For when a vendor delegates negotiation to a third party:

```json
{
  "type": "delegation_token",
  "delegator": "avp:vendor:acme-tech:a1b2c3d4",
  "delegate": "avp:vendor:acme-sales:b2c3d4e5",
  "scope": ["negotiate"],
  "constraints": {
    "max_discount": 0.15,
    "min_price": 100000,
    "sessions": ["ses_abc123def456"]
  },
  "expires": "2026-03-25T00:00:00Z"
}
```

---

## 9. Trust Delegation

### 9.1 Delegation Chain

```
Trust Authority (Root)
    │
    ├── Registry (Intermediate)
    │       │
    │       ├── Vendor Agent (Leaf)
    │       │       │
    │       │       └── Sales Agent (Delegated)
    │       │
    │       └── Buyer Agent (Leaf)
    │
    └── Regional Registry (Intermediate)
            │
            └── Vendor Agent (Leaf)
```

### 9.2 Trust Verification

To verify an agent's identity:

1. Fetch agent's public key
2. Verify registration token signature (signed by registry)
3. Verify registry's certificate (signed by trust authority)
4. Check revocation status (OCSP or CRL)
5. Check trust score is above threshold

---

## 10. Revocation

### 10.1 Revocation Methods

| Method | Speed | Use Case |
|--------|-------|----------|
| OCSP (Online Certificate Status Protocol) | Real-time | Standard verification |
| CRL (Certificate Revocation List) | Batch (hourly) | Offline verification |
| Registry blacklist | Immediate | Emergency block |
| Key rotation | Gradual | Planned key change |

### 10.2 Emergency Revocation

```
POST https://registry.avp-protocol.org/v1/agents/revoke

{
  "agent_id": "avp:vendor:compromised-corp:xxxxxxxx",
  "reason": "key_compromise",
  "effective": "immediate",
  "reported_by": "avp:vendor:compromised-corp:xxxxxxxx",
  "evidence": "..."
}
```

### 10.3 Revocation Propagation

```
Registry publishes revocation
    │
    ├──▶ OCSP responder updated (< 1 minute)
    ├──▶ CRL updated (next cycle, < 1 hour)
    ├──▶ Webhook sent to active session partners
    └──▶ DNS-based revocation record published
```

---

*End of Authentication & Authorization Layer Document*
