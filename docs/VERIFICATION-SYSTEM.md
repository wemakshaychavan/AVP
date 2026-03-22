# AVP Verification System v1.0-draft

**AI Vendor Protocol — Proof of Capability Specification**

Status: Draft
Version: 1.0-draft
Date: 2026-03-22
Authors: AVP Working Group

---

## Table of Contents

1. [Overview](#1-overview)
2. [Verification Layers](#2-verification-layers)
3. [Trust Score Computation](#3-trust-score-computation)
4. [Verification Levels](#4-verification-levels)
5. [Anti-Gaming Measures](#5-anti-gaming-measures)
6. [API Endpoints](#6-api-endpoints)
7. [Integration with Discovery](#7-integration-with-discovery)
8. [Privacy Considerations](#8-privacy-considerations)

---

## 1. Overview

The Proof of Capability system ensures vendor claims are verifiable. Without verification, any vendor can claim expertise they don't have. AVP solves this with multiple verification layers that produce a transparent, breakdown trust score.

The system is designed around the principle that trust must be **earned and evidenced**, not simply declared. Each verification layer adds a different dimension of proof, from basic domain ownership to cryptographically signed client attestations.

---

## 2. Verification Layers

### 2.1 Domain & Business Verification (Tier 1 — Basic)

This tier establishes that the vendor is a real, identifiable business entity.

- **DNS TXT record verification** — vendors add a registry-provided TXT record to their domain's DNS to prove ownership.
- **Business registration document upload** — incorporation certificates, trade licenses, or equivalent documents are submitted and reviewed.
- **Physical address verification** — address validation against postal databases or via postcard verification.
- **Website existence and age check** — WHOIS lookup confirms domain age; automated crawl verifies a functioning website.

**Result:** `"domain-verified"` status.

**avp.json credential declaration:**

```json
{
  "verification": {
    "domain": {
      "status": "verified",
      "method": "dns-txt",
      "verified_at": "2026-01-10T12:00:00Z",
      "domain": "acme-consulting.com",
      "business_name": "Acme Consulting Ltd.",
      "business_registration": "verified"
    }
  }
}
```

### 2.2 Credential Verification (Tier 2 — Certified)

Credentials are auto-verified against issuing authority APIs wherever possible:

- **AWS Partner Network API** — confirm AWS partner tier and competencies.
- **Google Cloud Partner Directory** — verify GCP partner status and specializations.
- **Microsoft Partner Center** — validate Microsoft partner designations.
- **ISO certification bodies** — check ISO 27001, ISO 9001, etc. against accredited registrar databases.
- **SOC2 attestation letters** — verify SOC2 Type I/II reports via auditor confirmation.

Verified credentials receive a **cryptographic stamp** from the registry — a signed assertion that the registry has independently confirmed the credential.

**avp.json credential schema:**

```json
{
  "credentials": [
    {
      "type": "partner_certification",
      "issuer": "aws",
      "credential_id": "aws-partner:advanced-tier:data-analytics",
      "name": "AWS Advanced Tier Partner — Data & Analytics",
      "issued_date": "2025-06-01",
      "expiry_date": "2026-06-01",
      "verification": {
        "status": "verified",
        "verified_at": "2026-01-12T08:30:00Z",
        "method": "issuer-api",
        "registry_stamp": "ed25519:base64..."
      }
    },
    {
      "type": "compliance_certification",
      "issuer": "bsi-group",
      "credential_id": "iso-27001:2022:BSI-12345",
      "name": "ISO 27001:2022",
      "issued_date": "2025-03-15",
      "expiry_date": "2028-03-15",
      "verification": {
        "status": "verified",
        "verified_at": "2026-01-12T09:00:00Z",
        "method": "registrar-lookup",
        "registry_stamp": "ed25519:base64..."
      }
    }
  ]
}
```

### 2.3 Portfolio & Project Verification (Tier 3 — Proven)

Vendors submit portfolio evidence that the registry independently verifies:

- **Live project URLs** — registry crawler verifies they exist, are accessible, and link back to the vendor's verified domain.
- **GitHub repositories** — verify ownership via GitHub API, check commit history depth, language/framework statistics, and contributor activity.
- **Published case studies with verifiable client references** — case studies must include a client contact or reference that can be independently confirmed.
- **App store listings** — iOS App Store and Google Play listings are verified via store APIs, confirming the vendor is the listed developer.

The registry scores portfolio items based on:
- **Quantity** — number of verified projects.
- **Recency** — how recently projects were active.
- **Relevance** — alignment with claimed capabilities.
- **Verifiability** — strength of evidence provided.

**avp.json portfolio schema:**

```json
{
  "portfolio": [
    {
      "type": "live_project",
      "title": "Enterprise Analytics Dashboard",
      "url": "https://analytics.clientcorp.com",
      "domain": "data-analytics",
      "technologies": ["react", "python", "aws-redshift", "d3"],
      "completed_date": "2025-09-01",
      "verification": {
        "status": "verified",
        "url_accessible": true,
        "backlink_found": true,
        "verified_at": "2026-01-15T10:00:00Z"
      }
    },
    {
      "type": "github_repo",
      "title": "ML Pipeline Framework",
      "url": "https://github.com/acme-consulting/ml-pipeline",
      "domain": "machine-learning",
      "technologies": ["python", "tensorflow", "kubernetes"],
      "verification": {
        "status": "verified",
        "ownership_confirmed": true,
        "commit_count": 1247,
        "contributors": 8,
        "last_commit": "2026-01-10T14:30:00Z",
        "primary_languages": {"python": 72, "dockerfile": 15, "yaml": 13},
        "verified_at": "2026-01-15T10:05:00Z"
      }
    },
    {
      "type": "app_store",
      "title": "HealthTrack Mobile App",
      "ios_url": "https://apps.apple.com/app/id123456789",
      "android_url": "https://play.google.com/store/apps/details?id=com.acme.healthtrack",
      "domain": "mobile-health",
      "technologies": ["react-native", "node", "firebase"],
      "verification": {
        "status": "verified",
        "ios_listing_confirmed": true,
        "android_listing_confirmed": true,
        "developer_match": true,
        "verified_at": "2026-01-15T10:10:00Z"
      }
    }
  ]
}
```

### 2.4 Client Attestations (Tier 4 — Attested)

Past clients use their **own AVP agent** to cryptographically sign attestations about vendor performance. This is the strongest form of verification because it requires an independent, verified party to vouch for the vendor.

**Attestation format:**

```json
{
  "type": "client_attestation",
  "attestor_agent_id": "avp:buyer:client-company:xyz",
  "vendor_agent_id": "avp:vendor:acme:abc",
  "project": {
    "title": "Data Science Platform",
    "domain": "data-science",
    "technologies": ["python", "tensorflow", "aws-sagemaker"],
    "duration_months": 6,
    "team_size": 5,
    "delivered": true,
    "satisfaction_score": 4.5
  },
  "attestation_date": "2026-01-15T00:00:00Z",
  "signature": "ed25519:base64..."
}
```

**Key properties:**

- Attestations are stored on the registry, **tamper-proof via Ed25519 signatures**.
- Buyers can verify attestation signatures against the attestor's registered public key.
- The attestor MUST be a verified buyer agent — vendors cannot attest for themselves.

**API endpoints:**

- `POST /v1/attestations` — create a new client attestation (requires authenticated buyer agent).
- `GET /v1/attestations/{vendor_id}` — retrieve all public attestations for a given vendor.

### 2.5 Employee & Team Verification (Tier 5 — Team Verified)

This tier is **optional** and provides additional confidence in vendor team claims.

**LinkedIn API integration:**
- Verify the company page exists and matches the claimed team size (within a reasonable range).
- Confirm that employee skill endorsements align with the vendor's capability claims.
- Validate that key team members listed in proposals exist and have relevant experience.

**GitHub organization verification:**
- Public repositories show sustained activity in the technologies the vendor claims.
- Contributor count is consistent with team size claims.
- Commit patterns indicate an active, productive engineering team.

### 2.6 On-Protocol Track Record

This data is built **automatically** from AVP protocol usage and cannot be faked:

- **RFPs received count** — how many buyer RFPs the vendor has been matched with.
- **Proposals sent count** — how many proposals the vendor has submitted.
- **Acceptance rate** — percentage of proposals accepted by buyers.
- **Response time** — average and p95 time from RFP receipt to proposal submission.
- **Buyer ratings** — post-acceptance ratings from buyers.
- **Repeat buyer rate** — percentage of buyers who engage the vendor more than once.

This data is objective, protocol-native, and provides the most reliable signal of vendor quality over time.

---

## 3. Trust Score Computation

The trust score is **NOT** a single opaque number. It is a transparent breakdown that buyers can inspect, filter on, and interpret according to their own priorities.

```json
{
  "trust_score": 0.82,
  "breakdown": {
    "domain_verified": true,
    "business_verified": true,
    "credentials": {
      "score": 0.9,
      "verified": ["aws-partner", "iso-27001"],
      "claimed_unverified": ["soc2"]
    },
    "portfolio": {
      "score": 0.75,
      "verified_projects": 8,
      "total_claimed": 12,
      "avg_recency_months": 14
    },
    "client_attestations": {
      "score": 0.85,
      "count": 5,
      "avg_satisfaction": 4.3,
      "verified_attestors": 4
    },
    "on_protocol": {
      "score": 0.78,
      "rfps_received": 45,
      "proposals_sent": 38,
      "acceptance_rate": 0.34,
      "avg_response_time_hours": 2.1,
      "repeat_buyer_rate": 0.22
    }
  },
  "last_updated": "2026-03-22T10:00:00Z",
  "verification_level": "attested"
}
```

**Score computation principles:**

- Each layer contributes independently — a vendor can be strong in one area and weak in another.
- The composite `trust_score` is a weighted average, but the weights are published and deterministic.
- Buyers can override weights in their own agent configuration to prioritize what matters to them (e.g., a buyer may weight client attestations higher than credentials).

---

## 4. Verification Levels

| Level | Requirements | Badge |
|-------|-------------|-------|
| Unverified | Just registered | None |
| Domain Verified | DNS + business docs | Bronze |
| Certified | 1+ verified credential | Silver |
| Proven | 3+ verified portfolio items | Gold |
| Attested | 2+ client attestations | Platinum |

Levels are cumulative — achieving "Attested" implies the vendor has also passed Domain Verification and has verified credentials and portfolio items. Each level unlocks additional visibility in the registry:

- **Unverified** vendors appear in search results but are clearly marked.
- **Domain Verified** vendors can receive RFPs from basic-tier buyers.
- **Certified** vendors appear in filtered searches for specific certifications.
- **Proven** vendors are eligible for premium placement in discovery results.
- **Attested** vendors receive the highest trust signal and priority in matching algorithms.

---

## 5. Anti-Gaming Measures

The verification system includes multiple safeguards against manipulation:

- **Self-attestation prevention** — attestations require the attestor to be a verified buyer agent with its own independent registration. A vendor cannot attest for itself.
- **Backlink requirement** — portfolio URLs MUST link back to the vendor's verified domain, preventing vendors from claiming others' work.
- **Re-verification triggers** — sudden changes in claimed capabilities (e.g., adding 10 new domains overnight) trigger mandatory re-verification.
- **Trust score decay** — scores decrease if verification data is not refreshed within 12 months. Stale credentials and portfolio items lose weight over time.
- **Anomaly detection** — unusual patterns (e.g., 50 attestations received in a single day, or attestations from newly created buyer agents) trigger automated review and potential holds.
- **Attestor rate limiting** — each buyer agent is limited to a maximum of 10 attestations per month, preventing attestation farms.
- **Cross-reference validation** — attestation claims are cross-referenced (e.g., if a client attests to a 6-month project, the vendor should have corresponding portfolio evidence).

---

## 6. API Endpoints

### 6.1 Domain Verification

**`POST /v1/verify/domain`**

Submit a domain for verification. The registry returns a DNS TXT record that must be added to the domain.

Request:
```json
{
  "agent_id": "avp:vendor:acme:abc",
  "domain": "acme-consulting.com",
  "business_name": "Acme Consulting Ltd.",
  "business_registration_country": "US",
  "business_registration_id": "EIN:12-3456789"
}
```

Response:
```json
{
  "verification_id": "ver_domain_abc123",
  "status": "pending",
  "dns_txt_record": "avp-verify=abc123def456",
  "instructions": "Add the above TXT record to your domain's DNS. Verification will be attempted every 10 minutes for 72 hours.",
  "expires_at": "2026-03-25T10:00:00Z"
}
```

### 6.2 Credential Verification

**`POST /v1/verify/credentials`**

Submit a credential for automated verification.

Request:
```json
{
  "agent_id": "avp:vendor:acme:abc",
  "credential": {
    "type": "partner_certification",
    "issuer": "aws",
    "credential_id": "aws-partner:advanced-tier:data-analytics",
    "name": "AWS Advanced Tier Partner — Data & Analytics",
    "issued_date": "2025-06-01",
    "expiry_date": "2026-06-01",
    "evidence_url": "https://partners.amazonaws.com/partners/acme-consulting"
  }
}
```

Response:
```json
{
  "verification_id": "ver_cred_xyz789",
  "status": "verifying",
  "method": "issuer-api",
  "estimated_completion": "2026-03-22T10:30:00Z"
}
```

### 6.3 Portfolio Verification

**`POST /v1/verify/portfolio`**

Submit portfolio items for verification.

Request:
```json
{
  "agent_id": "avp:vendor:acme:abc",
  "items": [
    {
      "type": "live_project",
      "title": "Enterprise Analytics Dashboard",
      "url": "https://analytics.clientcorp.com",
      "domain": "data-analytics",
      "technologies": ["react", "python", "aws-redshift"],
      "completed_date": "2025-09-01"
    },
    {
      "type": "github_repo",
      "url": "https://github.com/acme-consulting/ml-pipeline",
      "domain": "machine-learning",
      "technologies": ["python", "tensorflow"]
    }
  ]
}
```

Response:
```json
{
  "verification_id": "ver_port_def456",
  "status": "verifying",
  "items": [
    {
      "url": "https://analytics.clientcorp.com",
      "status": "crawling",
      "estimated_completion": "2026-03-22T10:15:00Z"
    },
    {
      "url": "https://github.com/acme-consulting/ml-pipeline",
      "status": "verifying",
      "estimated_completion": "2026-03-22T10:05:00Z"
    }
  ]
}
```

### 6.4 Client Attestations

**`POST /v1/attestations`**

Create a client attestation (requires authenticated buyer agent).

Request:
```json
{
  "attestor_agent_id": "avp:buyer:client-company:xyz",
  "vendor_agent_id": "avp:vendor:acme:abc",
  "project": {
    "title": "Data Science Platform",
    "domain": "data-science",
    "technologies": ["python", "tensorflow", "aws-sagemaker"],
    "duration_months": 6,
    "team_size": 5,
    "delivered": true,
    "satisfaction_score": 4.5
  },
  "signature": "ed25519:base64..."
}
```

Response:
```json
{
  "attestation_id": "att_ghi789",
  "status": "verified",
  "attestor_verified": true,
  "signature_valid": true,
  "stored_at": "2026-03-22T10:00:00Z"
}
```

**`GET /v1/attestations/{vendor_id}`**

Retrieve all public attestations for a vendor.

Response:
```json
{
  "vendor_agent_id": "avp:vendor:acme:abc",
  "attestations": [
    {
      "attestation_id": "att_ghi789",
      "attestor_agent_id": "avp:buyer:client-company:xyz",
      "project": {
        "title": "Data Science Platform",
        "domain": "data-science",
        "delivered": true,
        "satisfaction_score": 4.5
      },
      "attestation_date": "2026-01-15T00:00:00Z",
      "signature_valid": true
    }
  ],
  "total_count": 5,
  "avg_satisfaction": 4.3
}
```

### 6.5 Trust Breakdown

**`GET /v1/trust/{agent_id}/breakdown`**

Retrieve the full trust score breakdown for an agent.

Response:
```json
{
  "agent_id": "avp:vendor:acme:abc",
  "trust_score": 0.82,
  "verification_level": "attested",
  "breakdown": {
    "domain_verified": true,
    "business_verified": true,
    "credentials": {
      "score": 0.9,
      "verified": ["aws-partner", "iso-27001"],
      "claimed_unverified": ["soc2"]
    },
    "portfolio": {
      "score": 0.75,
      "verified_projects": 8,
      "total_claimed": 12,
      "avg_recency_months": 14
    },
    "client_attestations": {
      "score": 0.85,
      "count": 5,
      "avg_satisfaction": 4.3,
      "verified_attestors": 4
    },
    "on_protocol": {
      "score": 0.78,
      "rfps_received": 45,
      "proposals_sent": 38,
      "acceptance_rate": 0.34,
      "avg_response_time_hours": 2.1,
      "repeat_buyer_rate": 0.22
    }
  },
  "last_updated": "2026-03-22T10:00:00Z"
}
```

### 6.6 Skill Challenge (Optional)

**`POST /v1/verify/challenge`**

Request an optional skill challenge to further demonstrate capability.

Request:
```json
{
  "agent_id": "avp:vendor:acme:abc",
  "domain": "data-science",
  "challenge_type": "technical_assessment",
  "technologies": ["python", "tensorflow"]
}
```

Response:
```json
{
  "challenge_id": "chl_jkl012",
  "status": "issued",
  "challenge_type": "technical_assessment",
  "instructions_url": "https://registry.avp.dev/challenges/chl_jkl012",
  "deadline": "2026-03-29T10:00:00Z",
  "estimated_duration_hours": 4
}
```

---

## 7. Integration with Discovery

The trust breakdown SHOULD be included in the `/.well-known/avp.json` discovery document so buyers can filter vendors before even initiating a handshake.

Example `avp.json` trust section:

```json
{
  "agent": {
    "id": "avp:vendor:acme:abc",
    "type": "vendor",
    "name": "Acme Consulting"
  },
  "trust": {
    "score": 0.82,
    "verification_level": "attested",
    "badge": "platinum",
    "credentials_verified": ["aws-partner", "iso-27001"],
    "portfolio_verified_count": 8,
    "attestation_count": 5,
    "on_protocol_since": "2025-06-01",
    "breakdown_url": "https://registry.avp.dev/v1/trust/avp:vendor:acme:abc/breakdown"
  }
}
```

Buyers' agents can use these fields in discovery queries to filter vendors by minimum verification level, required credentials, or minimum trust score — all before initiating any communication.

---

## 8. Privacy Considerations

- **Vendor control over attestation visibility** — vendors choose which attestations are public and which remain private (still counted in score, but details hidden from public queries).
- **Aggregated employee data** — employee and team verification data is always aggregated. Individual employee names, profiles, or details are never exposed through the API.
- **Confidential portfolio items** — portfolio items can be marked `"confidential": true`, meaning the registry verifies them and counts them toward the score, but project details (title, URL, client name) are hidden from public queries.
- **GDPR compliance** — all verification data is subject to data erasure requests under GDPR Article 17. Vendors can request full deletion of their verification data, which will reset their trust score to zero.
- **Data retention** — verification evidence (uploaded documents, screenshots) is retained only as long as needed for verification and is purged after 90 days. Only the verification result (pass/fail + timestamp) is retained long-term.

---

*This specification is a living document. Submit feedback and proposals via the [AVP GitHub repository](https://github.com/wemakshaychavan/AVP).*
