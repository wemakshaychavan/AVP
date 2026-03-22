# AVP System Architecture

**Comprehensive Architecture Document for the AI Vendor Protocol**

Version: 1.0-draft
Date: 2026-03-18

---

## Table of Contents

1. [System Overview](#1-system-overview)
2. [Component Architecture](#2-component-architecture)
3. [Data Flow Architecture](#3-data-flow-architecture)
4. [Deployment Architecture](#4-deployment-architecture)
5. [Integration Architecture](#5-integration-architecture)
6. [Network Topology](#6-network-topology)
7. [Data Architecture](#7-data-architecture)
8. [Scalability Architecture](#8-scalability-architecture)

---

## 1. System Overview

### 1.1 High-Level Architecture

```
╔══════════════════════════════════════════════════════════════════════╗
║                        AVP ECOSYSTEM                                ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                      ║
║  ┌──────────────┐    ┌──────────────┐    ┌──────────────────────┐   ║
║  │   BUYER      │    │   AVP        │    │   VENDOR             │   ║
║  │   LAYER      │    │   CORE       │    │   LAYER              │   ║
║  │              │    │   INFRA      │    │                      │   ║
║  │ ┌──────────┐ │    │              │    │ ┌──────────────────┐ │   ║
║  │ │Chrome    │ │    │ ┌──────────┐ │    │ │ Vendor SDK       │ │   ║
║  │ │Extension │ │◀──▶│ │Registry  │ │◀──▶│ │ (Node/Python/Go) │ │   ║
║  │ └──────────┘ │    │ └──────────┘ │    │ └──────────────────┘ │   ║
║  │ ┌──────────┐ │    │ ┌──────────┐ │    │ ┌──────────────────┐ │   ║
║  │ │Buyer SDK │ │◀──▶│ │Trust     │ │◀──▶│ │ Vendor Agent     │ │   ║
║  │ │          │ │    │ │Authority │ │    │ │ (AI-Powered)     │ │   ║
║  │ └──────────┘ │    │ └──────────┘ │    │ └──────────────────┘ │   ║
║  │ ┌──────────┐ │    │ ┌──────────┐ │    │ ┌──────────────────┐ │   ║
║  │ │AI Engine │ │    │ │Schema    │ │    │ │ Proposal Engine  │ │   ║
║  │ │(Analysis)│ │    │ │Registry  │ │    │ │ (AI-Powered)     │ │   ║
║  │ └──────────┘ │    │ └──────────┘ │    │ └──────────────────┘ │   ║
║  │ ┌──────────┐ │    │ ┌──────────┐ │    │ ┌──────────────────┐ │   ║
║  │ │Dashboard │ │    │ │Monitoring│ │    │ │ Dashboard        │ │   ║
║  │ │(Web UI)  │ │    │ │& Logging │ │    │ │ (Web UI)         │ │   ║
║  │ └──────────┘ │    │ └──────────┘ │    │ └──────────────────┘ │   ║
║  └──────────────┘    └──────────────┘    └──────────────────────┘   ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

### 1.2 Core Principles

| Principle | Implementation |
|-----------|---------------|
| Decentralized | No single point of failure; vendors host their own agents |
| Federated Trust | Trust authority validates but doesn't control communication |
| Protocol-First | All communication follows the AVP spec |
| AI-Native | Every layer designed for AI agent interaction |
| Human-in-the-Loop | Humans can override at any point |

---

## 2. Component Architecture

### 2.1 Buyer-Side Components

```
┌─────────────────────────────────────────────────────┐
│                BUYER SYSTEM                          │
│                                                      │
│  ┌────────────────────┐   ┌────────────────────┐    │
│  │  Chrome Extension   │   │  Web Dashboard     │    │
│  │                     │   │                     │    │
│  │  - Search trigger   │   │  - RFP management  │    │
│  │  - Quick view       │   │  - Proposal review │    │
│  │  - Notifications    │   │  - Vendor compare  │    │
│  │  - Context extract  │   │  - Negotiation UI  │    │
│  └────────┬────────────┘   └────────┬────────────┘   │
│           │                          │                │
│           ▼                          ▼                │
│  ┌─────────────────────────────────────────────────┐ │
│  │              Buyer Agent Core                    │ │
│  │                                                   │ │
│  │  ┌─────────────┐  ┌──────────┐  ┌────────────┐ │ │
│  │  │ Discovery   │  │ Comms    │  │ Negotiation│ │ │
│  │  │ Engine      │  │ Manager  │  │ Engine     │ │ │
│  │  │             │  │          │  │            │ │ │
│  │  │ - Registry  │  │ - Send   │  │ - Strategy │ │ │
│  │  │   query     │  │ - Receive│  │ - Counter  │ │ │
│  │  │ - Well-known│  │ - Queue  │  │ - Accept   │ │ │
│  │  │   crawl     │  │ - Retry  │  │ - Reject   │ │ │
│  │  │ - DNS       │  │ - Sign   │  │ - Escalate │ │ │
│  │  └─────────────┘  └──────────┘  └────────────┘ │ │
│  │                                                   │ │
│  │  ┌─────────────┐  ┌──────────┐  ┌────────────┐ │ │
│  │  │ AI Analysis │  │ Auth     │  │ Storage    │ │ │
│  │  │ Engine      │  │ Manager  │  │ Layer      │ │ │
│  │  │             │  │          │  │            │ │ │
│  │  │ - Rank      │  │ - Keys   │  │ - Sessions │ │ │
│  │  │ - Score     │  │ - JWT    │  │ - History  │ │ │
│  │  │ - Compare   │  │ - Sign   │  │ - Cache    │ │ │
│  │  │ - Summarize │  │ - Verify │  │ - Proposals│ │ │
│  │  └─────────────┘  └──────────┘  └────────────┘ │ │
│  └─────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────┘
```

#### 2.1.1 Chrome Extension

| Component | Responsibility |
|-----------|---------------|
| Search Trigger | User initiates vendor search from any webpage |
| Context Extractor | Extracts relevant context from current page |
| Quick View Panel | Shows search results in sidebar |
| Notification Center | Alerts for proposal updates, negotiation status |
| Settings Manager | User preferences, API keys, saved searches |

#### 2.1.2 Buyer Agent Core

| Component | Responsibility |
|-----------|---------------|
| Discovery Engine | Finds vendors via Registry, well-known URIs, DNS |
| Communication Manager | Handles all AVP message send/receive with retry logic |
| Negotiation Engine | AI-driven negotiation strategy and execution |
| AI Analysis Engine | Ranks, scores, and compares proposals |
| Auth Manager | Key generation, JWT handling, signature creation/verification |
| Storage Layer | Persists sessions, proposals, negotiation history |

### 2.2 Vendor-Side Components

```
┌─────────────────────────────────────────────────────┐
│                VENDOR SYSTEM                         │
│                                                      │
│  ┌────────────────────┐   ┌────────────────────┐    │
│  │  Vendor Dashboard   │   │  Admin Panel       │    │
│  │                     │   │                     │    │
│  │  - Incoming RFPs    │   │  - Agent config    │    │
│  │  - Active deals     │   │  - Pricing rules   │    │
│  │  - Revenue analytics│   │  - Team profiles   │    │
│  │  - Response queue   │   │  - Auto-response   │    │
│  └────────┬────────────┘   └────────┬────────────┘   │
│           │                          │                │
│           ▼                          ▼                │
│  ┌─────────────────────────────────────────────────┐ │
│  │              Vendor Agent Core                   │ │
│  │                                                   │ │
│  │  ┌─────────────┐  ┌──────────┐  ┌────────────┐ │ │
│  │  │ Discovery   │  │ Request  │  │ Proposal   │ │ │
│  │  │ Publisher   │  │ Handler  │  │ Generator  │ │ │
│  │  │             │  │          │  │            │ │ │
│  │  │ - avp.json  │  │ - Validate│ │ - Template │ │ │
│  │  │ - Registry  │  │ - Route  │  │ - AI Draft │ │ │
│  │  │   sync      │  │ - Queue  │  │ - Price    │ │ │
│  │  │ - DNS TXT   │  │ - Filter │  │ - Team     │ │ │
│  │  └─────────────┘  └──────────┘  └────────────┘ │ │
│  │                                                   │ │
│  │  ┌─────────────┐  ┌──────────┐  ┌────────────┐ │ │
│  │  │ Negotiation │  │ Auth     │  │ Policy     │ │ │
│  │  │ Responder   │  │ Gate     │  │ Engine     │ │ │
│  │  │             │  │          │  │            │ │ │
│  │  │ - Counter   │  │ - Verify │  │ - Rules    │ │ │
│  │  │ - Accept    │  │ - Token  │  │ - Limits   │ │ │
│  │  │ - Escalate  │  │ - Rate   │  │ - Blacklist│ │ │
│  │  │ - Human     │  │   limit  │  │ - Auto     │ │ │
│  │  └─────────────┘  └──────────┘  └────────────┘ │ │
│  └─────────────────────────────────────────────────┘ │
│                                                      │
│  ┌─────────────────────────────────────────────────┐ │
│  │              Integration Layer                   │ │
│  │  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ │ │
│  │  │ CRM  │ │ ERP  │ │Slack │ │Email │ │ Cal  │ │ │
│  │  └──────┘ └──────┘ └──────┘ └──────┘ └──────┘ │ │
│  └─────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────┘
```

### 2.3 Core Infrastructure Components

```
┌──────────────────────────────────────────────────────────┐
│                  AVP CORE INFRASTRUCTURE                   │
│                                                            │
│  ┌──────────────────┐    ┌──────────────────────────┐     │
│  │   AVP Registry    │    │   Trust Authority         │     │
│  │                    │    │                            │     │
│  │  ┌──────────────┐ │    │  ┌──────────────────────┐ │     │
│  │  │ Agent Index   │ │    │  │ Business Verification│ │     │
│  │  │ - Search      │ │    │  │ - Document verify    │ │     │
│  │  │ - Filter      │ │    │  │ - Domain verify      │ │     │
│  │  │ - Rank        │ │    │  │ - Identity check     │ │     │
│  │  └──────────────┘ │    │  └──────────────────────┘ │     │
│  │  ┌──────────────┐ │    │  ┌──────────────────────┐ │     │
│  │  │ Health Monitor│ │    │  │ Reputation Engine    │ │     │
│  │  │ - Liveness    │ │    │  │ - Score compute      │ │     │
│  │  │ - Compliance  │ │    │  │ - Review aggregate   │ │     │
│  │  │ - SLA track   │ │    │  │ - Dispute resolve    │ │     │
│  │  └──────────────┘ │    │  └──────────────────────┘ │     │
│  │  ┌──────────────┐ │    │  ┌──────────────────────┐ │     │
│  │  │ Webhook Relay │ │    │  │ Certificate Authority│ │     │
│  │  │ - Events      │ │    │  │ - Issue AVP certs    │ │     │
│  │  │ - Notify      │ │    │  │ - Revoke             │ │     │
│  │  │ - Status      │ │    │  │ - OCSP               │ │     │
│  │  └──────────────┘ │    │  └──────────────────────┘ │     │
│  └──────────────────┘    └──────────────────────────┘     │
│                                                            │
│  ┌──────────────────┐    ┌──────────────────────────┐     │
│  │  Schema Registry  │    │  Analytics & Monitoring   │     │
│  │                    │    │                            │     │
│  │  - JSON Schemas   │    │  - Protocol metrics        │     │
│  │  - Validation     │    │  - Network health          │     │
│  │  - Versioning     │    │  - Anomaly detection       │     │
│  │  - Migration      │    │  - Abuse prevention        │     │
│  └──────────────────┘    └──────────────────────────┘     │
└──────────────────────────────────────────────────────────┘
```

---

## 3. Data Flow Architecture

### 3.1 Complete Request Flow

```
User clicks "Find Java Travel Dev Companies"
    │
    ▼
┌──────────────────┐
│ Chrome Extension  │
│ 1. Extract context│
│ 2. Build query    │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐     ┌──────────────────┐
│ Buyer Agent       │────▶│ AVP Registry      │
│ Discovery Engine  │◀────│ Search API        │
│ 3. Search vendors │     │ 4. Return matches │
└────────┬─────────┘     └──────────────────┘
         │
         │ For each matched vendor:
         ▼
┌──────────────────┐     ┌──────────────────┐
│ Buyer Agent       │────▶│ Vendor Website    │
│ Comms Manager     │     │ /.well-known/     │
│ 5. Fetch discovery│     │ avp.json          │
│ 6. Validate       │◀────│ 7. Return doc     │
└────────┬─────────┘     └──────────────────┘
         │
         ▼
┌──────────────────┐     ┌──────────────────┐
│ Buyer Agent       │────▶│ Trust Authority   │
│ Auth Manager      │◀────│                   │
│ 8. Verify vendor  │     │ 9. Trust score    │
└────────┬─────────┘     └──────────────────┘
         │
         │ Filter by trust score threshold
         ▼
┌──────────────────┐     ┌──────────────────┐
│ Buyer Agent       │────▶│ Vendor Agent      │
│ Comms Manager     │     │ /avp/handshake    │
│ 10. Handshake     │◀────│ 11. Session token │
└────────┬─────────┘     └──────────────────┘
         │
         ▼
┌──────────────────┐     ┌──────────────────┐
│ Buyer Agent       │────▶│ Vendor Agent      │
│ Comms Manager     │     │ /avp/rfp          │
│ 12. Send RFP      │◀────│ 13. Proposal      │
└────────┬─────────┘     └──────────────────┘
         │
         │ Collect all proposals
         ▼
┌──────────────────┐
│ AI Analysis       │
│ Engine            │
│ 14. Rank proposals│
│ 15. Score vendors │
│ 16. Generate      │
│     comparison    │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Dashboard / UI    │
│ 17. Show results  │
│ 18. Allow action  │
│     (negotiate/   │
│      accept/      │
│      reject)      │
└──────────────────┘
```

### 3.2 Negotiation Flow

```
┌─────────┐                              ┌─────────┐
│  Buyer   │                              │ Vendor  │
│  Agent   │                              │ Agent   │
└────┬─────┘                              └────┬────┘
     │                                         │
     │  Round 1: Counter on pricing            │
     │────────────────────────────────────────▶│
     │                                         │
     │         Vendor considers...             │
     │                                         │
     │  Round 1: Adjusted proposal             │
     │◀────────────────────────────────────────│
     │                                         │
     │  Buyer AI evaluates adjustment...       │
     │                                         │
     │  Round 2: Counter on timeline           │
     │────────────────────────────────────────▶│
     │                                         │
     │  Round 2: Partial accept + counter      │
     │◀────────────────────────────────────────│
     │                                         │
     │  Buyer AI: terms acceptable             │
     │                                         │
     │  Accept                                 │
     │────────────────────────────────────────▶│
     │                                         │
     │  Settlement confirmation                │
     │◀────────────────────────────────────────│
     │                                         │
     │  ┌─────────────────────────────────┐    │
     │  │ Record on Registry (optional)   │    │
     │  └─────────────────────────────────┘    │
     │                                         │
```

### 3.3 Multi-Vendor Parallel Flow

```
                    ┌──────────┐
                    │  Buyer   │
                    │  Agent   │
                    └────┬─────┘
                         │
              ┌──────────┼──────────┐
              │          │          │
         ┌────▼────┐┌───▼────┐┌───▼────┐
         │Vendor A ││Vendor B││Vendor C│
         │Handshake││Handshak││Handshak│
         └────┬────┘└───┬────┘└───┬────┘
              │         │         │
         ┌────▼────┐┌───▼────┐┌───▼────┐
         │Send RFP ││Send RFP││Send RFP│
         └────┬────┘└───┬────┘└───┬────┘
              │         │         │
              │    (async, parallel)
              │         │         │
         ┌────▼────┐┌───▼────┐┌───▼────┐
         │Proposal ││Proposal││Proposal│
         │ $185K   ││ $210K  ││ $150K  │
         └────┬────┘└───┬────┘└───┬────┘
              │         │         │
              └─────────┼─────────┘
                        │
                   ┌────▼─────┐
                   │AI Compare│
                   │& Rank    │
                   └────┬─────┘
                        │
               Rank: C > A > B
               (best value considering
                price + quality + trust)
```

---

## 4. Deployment Architecture

### 4.1 Vendor Deployment Options

#### Option A: Standalone (Self-Hosted)

```
┌─────────────────────────────────────┐
│         Vendor Infrastructure        │
│                                      │
│  ┌──────────┐    ┌──────────────┐   │
│  │ Existing  │    │ AVP Vendor   │   │
│  │ Website   │    │ Agent        │   │
│  │           │    │              │   │
│  │ nginx/    │───▶│ Node.js /    │   │
│  │ apache    │    │ Python       │   │
│  │           │    │              │   │
│  │ /.well-   │    │ /avp/*       │   │
│  │ known/    │    │              │   │
│  │ avp.json  │    │ ┌──────────┐ │   │
│  │ (static)  │    │ │ SQLite / │ │   │
│  │           │    │ │ Postgres │ │   │
│  └──────────┘    │ └──────────┘ │   │
│                   └──────────────┘   │
└─────────────────────────────────────┘
```

#### Option B: Cloud-Hosted (AVP Cloud)

```
┌──────────────────┐     ┌──────────────────────┐
│ Vendor Website    │     │ AVP Cloud             │
│                   │     │                       │
│ /.well-known/     │────▶│ ┌───────────────────┐│
│ avp.json          │     │ │ Managed Vendor    ││
│ (points to cloud) │     │ │ Agent             ││
│                   │     │ │                   ││
│                   │◀────│ │ - Auto proposals  ││
│                   │     │ │ - AI negotiation  ││
│                   │     │ │ - Dashboard       ││
│                   │     │ └───────────────────┘│
└──────────────────┘     └──────────────────────┘
```

#### Option C: Serverless (Edge)

```
┌──────────────────────────────────────┐
│         Edge Network (Cloudflare)     │
│                                       │
│  ┌─────────────┐  ┌───────────────┐  │
│  │ Worker:      │  │ Worker:       │  │
│  │ avp.json     │  │ /avp/*        │  │
│  │ (discovery)  │  │ (agent logic) │  │
│  └─────────────┘  └───────────────┘  │
│                                       │
│  ┌─────────────┐  ┌───────────────┐  │
│  │ KV Store:    │  │ D1 Database:  │  │
│  │ Config       │  │ Sessions      │  │
│  └─────────────┘  └───────────────┘  │
└──────────────────────────────────────┘
```

### 4.2 Buyer Deployment

```
┌─────────────────────────────────────────────────┐
│              BUYER DEPLOYMENT                    │
│                                                  │
│  ┌──────────────┐                               │
│  │ Chrome        │                               │
│  │ Extension     │◀──────┐                       │
│  └──────────────┘       │                       │
│                          │                       │
│  ┌──────────────┐  ┌────┴──────────────────┐    │
│  │ Web Dashboard │──│ Buyer Backend          │    │
│  │ (React SPA)   │  │                        │    │
│  └──────────────┘  │ ┌────────────────────┐ │    │
│                     │ │ API Server         │ │    │
│                     │ │ (Node.js/Express)  │ │    │
│                     │ └────────────────────┘ │    │
│                     │ ┌────────────────────┐ │    │
│                     │ │ Agent Workers      │ │    │
│                     │ │ (Bull Queue)       │ │    │
│                     │ └────────────────────┘ │    │
│                     │ ┌────────────────────┐ │    │
│                     │ │ AI Engine          │ │    │
│                     │ │ (LLM Integration)  │ │    │
│                     │ └────────────────────┘ │    │
│                     │ ┌────────────────────┐ │    │
│                     │ │ PostgreSQL + Redis  │ │    │
│                     │ └────────────────────┘ │    │
│                     └────────────────────────┘    │
└─────────────────────────────────────────────────┘
```

### 4.3 Registry Deployment

```
┌────────────────────────────────────────────────────────┐
│                  AVP REGISTRY                           │
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │ Load Balancer │  │ CDN          │  │ WAF          │  │
│  │ (HAProxy)     │  │ (Cloudflare) │  │ (Rate limit) │  │
│  └──────┬───────┘  └──────────────┘  └──────────────┘  │
│         │                                                │
│  ┌──────▼───────────────────────────────────────────┐   │
│  │              API Gateway (Kong)                    │   │
│  └──────┬───────────────────────────────────────────┘   │
│         │                                                │
│  ┌──────┴──────────────────────────────────────────┐    │
│  │           Microservices                          │    │
│  │                                                    │    │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────────────┐ │    │
│  │  │ Search   │ │ Trust    │ │ Verification     │ │    │
│  │  │ Service  │ │ Service  │ │ Service          │ │    │
│  │  └──────────┘ └──────────┘ └──────────────────┘ │    │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────────────┐ │    │
│  │  │ Agent    │ │ Webhook  │ │ Analytics        │ │    │
│  │  │ Service  │ │ Service  │ │ Service          │ │    │
│  │  └──────────┘ └──────────┘ └──────────────────┘ │    │
│  └─────────────────────────────────────────────────┘    │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │              Data Layer                            │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────────────┐ │   │
│  │  │PostgreSQL│ │ Redis    │ │ Elasticsearch    │ │   │
│  │  │(Primary) │ │ (Cache)  │ │ (Search Index)   │ │   │
│  │  └──────────┘ └──────────┘ └──────────────────┘ │   │
│  └──────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────┘
```

---

## 5. Integration Architecture

### 5.1 How External Systems Integrate with AVP

```
┌────────────────────────────────────────────────────────────┐
│                    INTEGRATION MAP                          │
│                                                              │
│  ┌──────────────┐                    ┌──────────────────┐   │
│  │ CRM Systems   │                    │ Project Mgmt     │   │
│  │ (Salesforce,  │◀──Webhook/API────▶│ (Jira, Linear,   │   │
│  │  HubSpot)     │                    │  Asana)           │   │
│  └──────────────┘                    └──────────────────┘   │
│         │                                     │              │
│         ▼                                     ▼              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                 AVP AGENT                             │    │
│  │                                                       │    │
│  │  Events:                                              │    │
│  │  - new_rfp_received → Create CRM Lead                │    │
│  │  - proposal_accepted → Create Jira Project           │    │
│  │  - negotiation_started → Slack Notification          │    │
│  │  - settlement_reached → Create Invoice in ERP        │    │
│  └─────────────────────────────────────────────────────┘    │
│         │                                     │              │
│         ▼                                     ▼              │
│  ┌──────────────┐                    ┌──────────────────┐   │
│  │ Communication │                    │ Finance          │   │
│  │ (Slack, Teams,│◀──Events────────▶│ (QuickBooks,     │   │
│  │  Email)       │                    │  Stripe, Razorpay│   │
│  └──────────────┘                    └──────────────────┘   │
│         │                                     │              │
│         ▼                                     ▼              │
│  ┌──────────────┐                    ┌──────────────────┐   │
│  │ AI Platforms  │                    │ Identity         │   │
│  │ (OpenAI,     │◀──LLM Calls──────▶│ (Okta, Auth0,    │   │
│  │  Anthropic)   │                    │  Google)          │   │
│  └──────────────┘                    └──────────────────┘   │
└────────────────────────────────────────────────────────────┘
```

### 5.2 Webhook Events

| Event | Payload | Use Case |
|-------|---------|----------|
| `avp.rfp.received` | RFP details | Create CRM lead, Slack alert |
| `avp.rfp.responded` | Proposal details | Update pipeline |
| `avp.negotiation.started` | Session details | Notify sales team |
| `avp.negotiation.round` | Round details | Log in CRM |
| `avp.proposal.accepted` | Final terms | Create project in PM tool |
| `avp.proposal.rejected` | Rejection reason | Update CRM, analytics |
| `avp.settlement.reached` | Agreement details | Generate contract, invoice |
| `avp.trust.updated` | New trust score | Internal dashboards |

### 5.3 Integration SDK

```javascript
const { AVPIntegrations } = require('@avp/integrations');

const integrations = new AVPIntegrations({
  agent: vendorAgent,

  // CRM Integration
  crm: {
    provider: 'salesforce',
    on_rfp_received: (rfp) => createLead(rfp),
    on_settlement: (deal) => updateOpportunity(deal)
  },

  // Communication
  notifications: {
    slack: { webhook: 'https://hooks.slack.com/...' },
    email: { smtp: '...' }
  },

  // Finance
  billing: {
    provider: 'stripe',
    on_settlement: (deal) => createInvoice(deal)
  }
});
```

---

## 6. Network Topology

### 6.1 Decentralized Network Model

```
     ┌───────┐          ┌───────┐
     │Vendor │          │Vendor │
     │   A   │◀────────▶│   B   │
     └───┬───┘          └───┬───┘
         │                  │
         │    ┌────────┐    │
         └───▶│Registry│◀───┘
              │(Index) │
         ┌───▶│        │◀───┐
         │    └────────┘    │
         │                  │
     ┌───┴───┐          ┌──┴────┐
     │Vendor │          │Buyer  │
     │   C   │          │   X   │
     └───────┘          └───┬───┘
                            │
                        ┌───┴───┐
                        │Buyer  │
                        │   Y   │
                        └───────┘
```

Key properties:
- **Registry is an index, not a gateway** — communication is direct between agents
- **No single point of failure** — if registry is down, direct discovery still works
- **Peer-to-peer communication** — buyer talks directly to vendor
- **Registry federation** — multiple registries can exist and sync

### 6.2 Federation Model

```
┌────────────────────┐     ┌────────────────────┐
│ Registry: Global    │◀──▶│ Registry: India     │
│ registry.avp.org    │     │ in.registry.avp.org │
└────────┬───────────┘     └────────┬───────────┘
         │                          │
         │    ┌────────────────┐    │
         └───▶│ Registry: EU   │◀───┘
              │eu.registry.avp │
              └────────────────┘
```

Registries sync agent indexes but maintain independence for:
- Data residency compliance (GDPR)
- Regional trust authorities
- Local business verification standards

---

## 7. Data Architecture

### 7.1 Core Data Model

```
┌─────────────────────────────────────────────────┐
│                  DATA MODEL                      │
│                                                  │
│  ┌───────────┐     ┌──────────────┐             │
│  │   Agent    │────▶│  Capability  │             │
│  │           │     │              │             │
│  │ id        │     │ technology   │             │
│  │ name      │     │ domain       │             │
│  │ type      │     │ service      │             │
│  │ domain    │     │ proficiency  │             │
│  │ trust_score│     └──────────────┘             │
│  └─────┬─────┘                                  │
│        │                                         │
│        │ 1:N                                     │
│        ▼                                         │
│  ┌───────────┐     ┌──────────────┐             │
│  │  Session   │────▶│   Message    │             │
│  │           │     │              │             │
│  │ id        │     │ id           │             │
│  │ buyer_id  │     │ type         │             │
│  │ vendor_id │     │ payload      │             │
│  │ status    │     │ signature    │             │
│  │ created   │     │ timestamp    │             │
│  └─────┬─────┘     └──────────────┘             │
│        │                                         │
│        │ 1:1                                     │
│        ▼                                         │
│  ┌───────────┐     ┌──────────────┐             │
│  │    RFP     │────▶│  Proposal    │             │
│  │           │     │              │             │
│  │ id        │     │ id           │             │
│  │ title     │     │ pricing      │             │
│  │ reqs      │     │ timeline     │             │
│  │ budget    │     │ team         │             │
│  │ deadline  │     │ confidence   │             │
│  └───────────┘     └──────┬───────┘             │
│                           │                      │
│                           │ 1:N                  │
│                           ▼                      │
│                    ┌──────────────┐              │
│                    │ Negotiation  │              │
│                    │   Round      │              │
│                    │              │              │
│                    │ round_num    │              │
│                    │ changes      │              │
│                    │ status       │              │
│                    └──────────────┘              │
└─────────────────────────────────────────────────┘
```

### 7.2 Storage Strategy

| Data Type | Storage | Reason |
|-----------|---------|--------|
| Agent profiles | PostgreSQL + Elasticsearch | Relational + full-text search |
| Capabilities | PostgreSQL + Vector DB | Semantic similarity matching |
| Sessions | Redis (active) + PostgreSQL (archive) | Speed + persistence |
| Messages | PostgreSQL (append-only) | Audit trail |
| Proposals | PostgreSQL + Object Storage | Structured + large docs |
| Trust scores | PostgreSQL + Redis (cache) | Computed + fast reads |
| Discovery docs | CDN + Redis | High availability |

---

## 8. Scalability Architecture

### 8.1 Scaling Strategy

```
┌──────────────────────────────────────────────┐
│              SCALING LAYERS                    │
│                                                │
│  Layer 1: Edge (CDN)                          │
│  ┌──────────────────────────────────────────┐ │
│  │ Cache: avp.json, public keys, schemas    │ │
│  │ WAF: Rate limiting, DDoS protection      │ │
│  └──────────────────────────────────────────┘ │
│                                                │
│  Layer 2: API Gateway                         │
│  ┌──────────────────────────────────────────┐ │
│  │ Route: /avp/* to agent workers           │ │
│  │ Auth: Verify signatures at gateway       │ │
│  │ Throttle: Per-agent rate limits          │ │
│  └──────────────────────────────────────────┘ │
│                                                │
│  Layer 3: Worker Pool                         │
│  ┌──────────────────────────────────────────┐ │
│  │ Auto-scale: Based on queue depth         │ │
│  │ Isolation: Per-session worker binding    │ │
│  │ Priority: Premium agents get priority    │ │
│  └──────────────────────────────────────────┘ │
│                                                │
│  Layer 4: Data                                │
│  ┌──────────────────────────────────────────┐ │
│  │ Read replicas for search                 │ │
│  │ Write primary for transactions           │ │
│  │ Redis cluster for sessions               │ │
│  └──────────────────────────────────────────┘ │
└──────────────────────────────────────────────┘
```

### 8.2 Performance Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| Discovery response | < 100ms | p99 latency |
| Handshake completion | < 500ms | p99 latency |
| RFP delivery | < 1s | p99 latency |
| Proposal generation | < 30s | p95 (AI-dependent) |
| Registry search | < 200ms | p99 latency |
| Concurrent sessions per vendor | 100+ | Steady state |
| Messages per second (registry) | 10,000+ | Peak capacity |

---

*End of Architecture Document*
