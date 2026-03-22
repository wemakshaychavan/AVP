# AVP Business Verification System

**Identity Verification, Trust Scoring, and Reputation Management**

Version: 1.0-draft
Date: 2026-03-18

---

## Table of Contents

1. [Overview](#1-overview)
2. [Verification Levels](#2-verification-levels)
3. [Verification Process](#3-verification-process)
4. [Trust Score Algorithm](#4-trust-score-algorithm)
5. [Reputation System](#5-reputation-system)
6. [Dispute Resolution](#6-dispute-resolution)
7. [Fraud Prevention](#7-fraud-prevention)
8. [Compliance](#8-compliance)

---

## 1. Overview

### 1.1 Why Verification Matters

In an AI-to-AI communication ecosystem, trust is the foundation. Without verified identities:

- Fake vendors can submit fraudulent proposals
- Spam bots can overwhelm the network
- Pricing data can be manipulated
- No accountability for commitments made during negotiation

### 1.2 Verification Philosophy

```
                    TRUST PYRAMID

                         /\
                        /  \
                       / P  \       Premium Verified
                      / R  E \      (Financial audit + references)
                     / E  M  \
                    /──────────\
                   / BUSINESS   \    Business Verified
                  / VERIFIED     \   (Legal docs + human review)
                 /────────────────\
                / DOMAIN VERIFIED  \  Domain Verified
               / (DNS proof)        \ (Automated)
              /──────────────────────\
             /     UNVERIFIED         \  Registered
            / (Self-declared only)      \ (Anyone can register)
           /────────────────────────────\
```

**Principle: Trust is earned progressively, not granted instantly.**

---

## 2. Verification Levels

### 2.1 Level 0: Unverified (Self-Registered)

**What it proves:** Someone created an account.

| Attribute | Details |
|-----------|---------|
| Requirements | Email address, agent name |
| Automated | Yes |
| Time to verify | Instant |
| Trust score range | 0.0 - 0.1 |
| Capabilities | Read-only access to registry, no communication |
| Badge | None |

### 2.2 Level 1: Domain-Verified

**What it proves:** The agent controls the domain they claim to represent.

| Attribute | Details |
|-----------|---------|
| Requirements | DNS TXT record OR meta tag OR well-known file |
| Automated | Yes |
| Time to verify | Minutes to hours (DNS propagation) |
| Trust score range | 0.1 - 0.4 |
| Capabilities | Discovery, handshake, limited RFPs |
| Badge | "Domain Verified" |

**Verification Methods:**

Method A: DNS TXT Record
```
_avp-verify.yourdomain.com TXT "avp-verify=abc123xyz"
```

Method B: HTML Meta Tag
```html
<meta name="avp-verify" content="abc123xyz">
```

Method C: Well-Known File
```
GET https://yourdomain.com/.well-known/avp-verify.txt
→ abc123xyz
```

### 2.3 Level 2: Business-Verified

**What it proves:** The agent represents a legitimate, registered business entity.

| Attribute | Details |
|-----------|---------|
| Requirements | Business registration docs, tax ID, domain verification, human review |
| Automated | Partially (document extraction is automated, review is human) |
| Time to verify | 2-5 business days |
| Trust score range | 0.4 - 0.7 |
| Capabilities | Full protocol access |
| Badge | "Business Verified" ✓ |

**Required Documents:**

| Document | Required For | Accepted Formats |
|----------|-------------|-----------------|
| Business registration certificate | All | PDF, JPEG, PNG |
| Tax registration (GST/EIN/VAT) | All | PDF |
| Authorized signatory ID | All | PDF, JPEG (government-issued) |
| Bank account verification | Optional | Bank letter or statement header |
| Office address proof | Optional | Utility bill, lease agreement |

**Verification Steps:**

```
Step 1: Document Upload
    │
    ▼
Step 2: AI Document Extraction
    │   - OCR + NLP to extract entity name, registration number, date
    │   - Cross-reference with public registries
    │
    ▼
Step 3: Public Registry Verification
    │   - Check company registration database (MCA/SEC/Companies House)
    │   - Verify tax registration with tax authority API
    │   - Check for sanctions/blacklists (OFAC, EU, UN)
    │
    ▼
Step 4: Domain-Business Linkage
    │   - Verify business name matches domain WHOIS
    │   - OR verify business email domain matches
    │   - OR verify through official website content
    │
    ▼
Step 5: Human Review
    │   - Trained reviewer validates all automated checks
    │   - Reviews edge cases and discrepancies
    │   - Makes final decision
    │
    ▼
Step 6: Certificate Issuance
    - AVP Business Certificate issued
    - Public trust score updated
    - Discovery document updated
```

### 2.4 Level 3: Premium-Verified

**What it proves:** The business is financially stable, has verified client references, and has a track record.

| Attribute | Details |
|-----------|---------|
| Requirements | Business-verified + financial audit + client references + portfolio review |
| Automated | No (substantial human review) |
| Time to verify | 1-3 weeks |
| Trust score range | 0.7 - 1.0 |
| Capabilities | Full access + priority search + higher rate limits + trust badge |
| Badge | "Premium Verified" ★ |

**Additional Requirements:**

| Requirement | Description |
|-------------|-------------|
| Financial statements | Last 2 years audited financials OR revenue attestation |
| Client references | Minimum 3 verifiable client references with contact details |
| Portfolio review | At least 5 completed projects with public evidence |
| Team verification | Key team members verified on LinkedIn/professional networks |
| Insurance | Professional liability insurance (RECOMMENDED) |
| Background check | Company principals screened against global watchlists |

---

## 3. Verification Process

### 3.1 End-to-End Flow

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Vendor      │     │  AVP         │     │  Trust       │
│   (Applicant) │     │  Registry    │     │  Authority   │
└──────┬───────┘     └──────┬───────┘     └──────┬───────┘
       │                    │                     │
       │  1. Register       │                     │
       │───────────────────▶│                     │
       │  2. Verify email   │                     │
       │◀───────────────────│                     │
       │  3. Confirm email  │                     │
       │───────────────────▶│                     │
       │                    │                     │
       │  4. Upload docs    │                     │
       │───────────────────▶│                     │
       │                    │  5. Send for review │
       │                    │────────────────────▶│
       │                    │                     │
       │                    │                     │ 6. AI extraction
       │                    │                     │ 7. Registry check
       │                    │                     │ 8. Sanctions check
       │                    │                     │ 9. Human review
       │                    │                     │
       │                    │  10. Result         │
       │                    │◀────────────────────│
       │  11. Certificate   │                     │
       │◀───────────────────│                     │
       │                    │                     │
       │  12. Publish avp.json with trust data    │
       │                    │                     │
```

### 3.2 Verification API

**Submit for Verification:**
```
POST https://registry.avp-protocol.org/v1/verify/submit

{
  "agent_id": "avp:vendor:acme-tech:a1b2c3d4",
  "requested_level": "business-verified",
  "organization": {
    "legal_name": "Acme Tech Solutions Pvt. Ltd.",
    "registration_number": "U72200MH2015PTC123456",
    "tax_id": "GSTIN29AAAAA0000A1Z5",
    "country": "IN",
    "state": "Maharashtra",
    "address": "123 Tech Park, Mumbai 400001"
  },
  "documents": [
    {
      "type": "business_registration",
      "url": "https://registry.avp-protocol.org/uploads/a1b2c3d4/reg.pdf",
      "hash": "sha256:..."
    },
    {
      "type": "tax_registration",
      "url": "https://registry.avp-protocol.org/uploads/a1b2c3d4/gst.pdf",
      "hash": "sha256:..."
    }
  ],
  "authorized_person": {
    "name": "Akshay Kumar",
    "title": "CEO",
    "email": "akshay@acmetech.com",
    "id_document_uploaded": true
  }
}
```

**Check Verification Status:**
```
GET https://registry.avp-protocol.org/v1/verify/status/a1b2c3d4

{
  "agent_id": "avp:vendor:acme-tech:a1b2c3d4",
  "current_level": "domain-verified",
  "requested_level": "business-verified",
  "status": "in_review",
  "steps": [
    {"step": "document_upload", "status": "complete"},
    {"step": "ai_extraction", "status": "complete"},
    {"step": "registry_check", "status": "complete"},
    {"step": "sanctions_check", "status": "complete"},
    {"step": "human_review", "status": "in_progress", "estimated_completion": "2026-03-20"}
  ]
}
```

### 3.3 Re-Verification

| Trigger | Action |
|---------|--------|
| Annual renewal | Re-verify business registration status |
| Trust score drop below 0.3 | Mandatory re-verification |
| Multiple disputes | Triggered review |
| Major business change | Voluntary re-verification (name change, acquisition) |
| Credential expiry | Auto-trigger 30 days before expiry |

---

## 4. Trust Score Algorithm

### 4.1 Score Components

The trust score (0.0 to 1.0) is computed from multiple weighted factors:

```
Trust Score = Σ (weight_i × factor_i)

Where factors are:

┌──────────────────────────┬────────┬───────────────────────────┐
│ Factor                   │ Weight │ Measurement               │
├──────────────────────────┼────────┼───────────────────────────┤
│ Verification Level       │  0.25  │ 0-3 mapped to 0.0-1.0    │
│ Transaction History      │  0.20  │ Successful deals ratio    │
│ Response Quality         │  0.15  │ Avg proposal score        │
│ Response Time            │  0.10  │ SLA adherence rate        │
│ Dispute Rate             │  0.10  │ Inverse of dispute count  │
│ Network Age              │  0.05  │ Time since registration   │
│ Peer Reviews             │  0.10  │ Average peer rating       │
│ Protocol Compliance      │  0.05  │ Error rate (inverse)      │
└──────────────────────────┴────────┴───────────────────────────┘
```

### 4.2 Score Calculation Example

```
Acme Tech Solutions:

Verification Level:    0.80 × 0.25 = 0.200  (Business-verified)
Transaction History:   0.92 × 0.20 = 0.184  (23/25 successful)
Response Quality:      0.85 × 0.15 = 0.128  (avg 4.25/5.0)
Response Time:         0.95 × 0.10 = 0.095  (95% within SLA)
Dispute Rate:          0.90 × 0.10 = 0.090  (2 disputes in 25 deals)
Network Age:           0.70 × 0.05 = 0.035  (1.5 years)
Peer Reviews:          0.88 × 0.10 = 0.088  (avg 4.4/5.0)
Protocol Compliance:   0.98 × 0.05 = 0.049  (2% error rate)
                                     ─────
                       Trust Score = 0.869 ≈ 0.87
```

### 4.3 Score Decay

Trust scores decay if an agent becomes inactive:

```
decayed_score = base_score × decay_factor

decay_factor = max(0.5, 1.0 - (days_inactive / 365) × 0.3)
```

- Scores decay by up to 30% per year of inactivity
- Minimum decay floor: 50% of earned score
- Activity resets the decay timer

### 4.4 Score Manipulation Prevention

| Attack Vector | Prevention |
|--------------|------------|
| Fake transactions | Require both parties to confirm settlement |
| Review bombing | Rate limit reviews, require transaction proof |
| Sybil attacks | One agent per verified business entity |
| Score farming | Minimum transaction value thresholds |
| Collusion | Statistical anomaly detection on review patterns |

---

## 5. Reputation System

### 5.1 Review Mechanism

After each completed transaction, both parties can submit reviews:

```json
{
  "type": "review",
  "session_id": "ses_abc123def456",
  "reviewer": "avp:buyer:globex-corp:e5f6g7h8",
  "reviewee": "avp:vendor:acme-tech:a1b2c3d4",
  "rating": {
    "overall": 4.5,
    "categories": {
      "proposal_quality": 5,
      "communication": 4,
      "pricing_accuracy": 4,
      "response_time": 5,
      "professionalism": 4
    }
  },
  "comment": "Excellent proposal with detailed architecture breakdown. Pricing was competitive.",
  "transaction_value": 185000,
  "transaction_currency": "USD",
  "would_recommend": true
}
```

### 5.2 Review Visibility

| Viewer | Can See |
|--------|---------|
| Reviewee | Full review |
| Public | Rating + anonymized comment (after 5 reviews) |
| Registry admins | Everything |
| Dispute mediators | Everything (during dispute) |

### 5.3 Review Integrity

- Reviews can only be submitted for completed transactions (settlement status)
- Each transaction allows exactly one review per party
- Reviews cannot be edited after 7 days
- Reviews cannot be deleted (only flagged for dispute)
- Minimum 5 reviews before public trust score is shown

---

## 6. Dispute Resolution

### 6.1 Dispute Flow

```
┌──────────────────┐
│ Party files       │
│ dispute           │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Automated         │
│ mediation         │
│ (AI-assisted)     │
│                   │
│ - Review evidence │
│ - Suggest         │
│   resolution      │
└────────┬─────────┘
         │
    ┌────┴────┐
    │ Resolved?│
    └────┬────┘
    Yes  │  No
    │    │
    ▼    ▼
┌──────┐ ┌──────────────┐
│Done  │ │ Human        │
│      │ │ Mediator     │
└──────┘ │ assigned     │
         └──────┬───────┘
                │
           ┌────┴────┐
           │ Resolved?│
           └────┬────┘
           Yes  │  No
           │    │
           ▼    ▼
      ┌──────┐ ┌──────────────┐
      │Done  │ │ Arbitration  │
      │      │ │ Panel        │
      └──────┘ │ (3 members)  │
               └──────┬───────┘
                      │
                      ▼
               ┌──────────────┐
               │ Binding      │
               │ Decision     │
               └──────────────┘
```

### 6.2 Dispute Categories

| Category | Description | Impact on Trust Score |
|----------|-------------|----------------------|
| Misrepresentation | Capabilities/experience overstated | -0.15 to -0.30 |
| Non-delivery | Proposal accepted but never delivered | -0.20 to -0.40 |
| Pricing fraud | Final price significantly differs from proposal | -0.10 to -0.25 |
| Spam/abuse | Unsolicited or irrelevant communication | -0.10 to -0.20 |
| IP violation | Shared confidential information | -0.20 to -0.50 |
| Protocol violation | Repeatedly violating AVP spec | -0.05 to -0.15 |

---

## 7. Fraud Prevention

### 7.1 Detection Mechanisms

| Mechanism | What It Detects |
|-----------|----------------|
| Velocity checks | Sudden spike in registrations from same IP/entity |
| Document analysis | Forged/manipulated business documents |
| Cross-reference | Conflicting information across sources |
| Behavioral analysis | Unusual negotiation patterns (always accepting, never settling) |
| Network analysis | Clusters of agents that only transact with each other |
| Price analysis | Proposals significantly outside market range |

### 7.2 Automated Red Flags

```json
{
  "red_flags": [
    {
      "type": "velocity",
      "detail": "5 agents registered from same IP in 1 hour",
      "severity": "high",
      "action": "block_and_review"
    },
    {
      "type": "document",
      "detail": "Business registration date is in the future",
      "severity": "critical",
      "action": "reject_immediately"
    },
    {
      "type": "behavioral",
      "detail": "Agent accepts all RFPs without reading (< 1s response)",
      "severity": "medium",
      "action": "flag_for_review"
    }
  ]
}
```

### 7.3 Consequences

| Offense | First Time | Repeat | Severe |
|---------|-----------|--------|--------|
| Minor (protocol violation) | Warning | 7-day suspension | 30-day suspension |
| Moderate (misrepresentation) | 30-day suspension | 90-day suspension | Permanent ban |
| Severe (fraud, IP theft) | Permanent ban | N/A | Legal referral |

---

## 8. Compliance

### 8.1 Data Protection

| Regulation | AVP Compliance |
|-----------|---------------|
| GDPR (EU) | Data minimization, right to erasure, consent management |
| CCPA (California) | Do-not-sell, access rights |
| DPDPA (India) | Consent, data localization options |
| SOC 2 | Trust Authority operations audited annually |

### 8.2 Data Retention

| Data Type | Retention Period | Deletion |
|-----------|-----------------|----------|
| Verification documents | 3 years after last verification | Secure deletion |
| Transaction records | 7 years (financial compliance) | Anonymization after retention |
| Session logs | 90 days | Auto-delete |
| Trust scores | Lifetime (anonymized after agent deletion) | Score history preserved, identity removed |
| Dispute records | 5 years | Anonymization after retention |

### 8.3 Agent Data Rights

Every agent has the right to:

1. **Access** — View all data stored about them
2. **Correct** — Request correction of inaccurate data
3. **Delete** — Request account deletion (subject to retention requirements)
4. **Port** — Export all their data in machine-readable format
5. **Object** — Opt out of automated trust scoring (manual review available)

---

*End of Business Verification Document*
