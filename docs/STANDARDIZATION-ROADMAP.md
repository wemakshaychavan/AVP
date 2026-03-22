# AVP Standardization Roadmap

**Path from Open-Source Protocol to Internet Standard**

Version: 1.0-draft
Date: 2026-03-18

---

## Table of Contents

1. [Vision](#1-vision)
2. [Standardization Strategy](#2-standardization-strategy)
3. [Phase 1: Community Standard](#3-phase-1-community-standard)
4. [Phase 2: Industry Adoption](#4-phase-2-industry-adoption)
5. [Phase 3: De Facto Standard](#5-phase-3-de-facto-standard)
6. [Phase 4: Formal Standard](#6-phase-4-formal-standard)
7. [Governance for Standardization](#7-governance-for-standardization)
8. [Comparison with Existing Standards](#8-comparison-with-existing-standards)
9. [Adoption Incentive Framework](#9-adoption-incentive-framework)

---

## 1. Vision

AVP aims to become the universal standard for AI agent business communication, similar to how:

| Analogy | What It Standardized | AVP Equivalent |
|---------|---------------------|---------------|
| HTTP | Web page transfer | AI agent communication |
| OAuth | Authorization | Agent authentication |
| robots.txt | Crawler permissions | Agent policies |
| OpenAPI | API documentation | Agent capability declaration |
| DNS | Domain name resolution | Agent discovery |
| SMTP | Email exchange | Proposal exchange |

**End state:** Every business website has `/.well-known/avp.json` — just like every website has `robots.txt`.

---

## 2. Standardization Strategy

### 2.1 The Path

```
                    STANDARDIZATION PATH

Year 1              Year 2              Year 3              Year 4+
┌───────────┐      ┌───────────┐      ┌───────────┐      ┌───────────┐
│ Community  │─────▶│ Industry  │─────▶│ De Facto  │─────▶│ Formal    │
│ Standard   │      │ Adoption  │      │ Standard  │      │ Standard  │
│            │      │           │      │           │      │           │
│ - Open spec│      │ - 1000+   │      │ - 10K+    │      │ - IETF    │
│ - GitHub   │      │   vendors │      │   vendors │      │   RFC     │
│ - 10-20    │      │ - Platform│      │ - Platform│      │ - W3C     │
│   adopters │      │   support │      │   default │      │   spec    │
│ - Feedback │      │ - SDK in  │      │ - Multiple│      │ - ISO     │
│   loop     │      │   5+ langs│      │   registrs│      │           │
└───────────┘      └───────────┘      └───────────┘      └───────────┘
```

### 2.2 Why This Order

1. **Community first** — Prove the protocol works with real users
2. **Adoption second** — Get critical mass before formalizing
3. **Formalization last** — Standards bodies respect existing adoption

This is how HTTP, JSON, OAuth, and WebSocket all became standards.

---

## 3. Phase 1: Community Standard (Year 1)

### 3.1 Goals

- Publish protocol spec as open document
- Build reference implementations
- Get 10-20 real-world adopters
- Iterate based on feedback
- Establish community governance

### 3.2 Actions

| Action | Timeline | Details |
|--------|----------|---------|
| Publish spec on GitHub | Month 1 | CC BY 4.0 license |
| Create RFC-style process | Month 2 | AVP Enhancement Proposals (AEPs) |
| Open community forum | Month 2 | GitHub Discussions + Discord |
| First AEP cycle | Month 3-4 | Community proposes changes |
| Spec v1.0 stable | Month 6 | Based on alpha feedback |
| Spec v1.1 | Month 9 | Based on beta feedback |
| Spec v1.2 | Month 12 | Based on public launch feedback |

### 3.3 AVP Enhancement Proposals (AEPs)

```
AEP Process:

1. Author writes AEP (markdown document)
   ├── Problem statement
   ├── Proposed solution
   ├── Backward compatibility analysis
   └── Reference implementation (optional)

2. Community review (14 days minimum)
   ├── GitHub PR discussion
   ├── Discord/forum debate
   └── Implementer feedback

3. Working Group vote
   ├── Requires 2/3 majority
   └── At least 3 implementations must agree

4. If accepted:
   ├── Merged into spec
   ├── Reference implementation updated
   └── SDKs updated

5. If rejected:
   ├── Feedback provided
   └── Author can revise and resubmit
```

### 3.4 Success Criteria

- [ ] 500+ GitHub stars
- [ ] 20+ contributors
- [ ] 10+ AEPs submitted
- [ ] 3+ independent implementations
- [ ] Active Discord community (100+ members)

---

## 4. Phase 2: Industry Adoption (Year 2)

### 4.1 Goals

- 1,000+ vendors using AVP
- Platform integrations (Clutch, WordPress, etc.)
- SDKs in 5+ languages
- Industry recognition

### 4.2 Platform Partnership Strategy

| Platform | Integration Type | Priority | Value |
|----------|-----------------|----------|-------|
| WordPress | Plugin | High | 40% of web runs WordPress |
| Clutch | API bridge | High | Largest vendor directory |
| GoodFirms | API bridge | High | Second largest directory |
| LinkedIn | Data enrichment | Medium | Professional network |
| Shopify | App | Medium | E-commerce vendors |
| HubSpot | Integration | Medium | CRM for SMBs |
| Salesforce | AppExchange | High | Enterprise CRM |
| Webflow | Plugin | Medium | Modern website builder |

### 4.3 Developer Ecosystem

```
SDK Priority:

Phase 1 (Year 1):
├── JavaScript/TypeScript (npm)
└── Python (pip)

Phase 2 (Year 2 Q1-Q2):
├── Go
├── Java/Kotlin
└── Ruby

Phase 3 (Year 2 Q3-Q4):
├── PHP
├── C#/.NET
├── Rust
└── Swift
```

### 4.4 Industry Event Strategy

| Event | Purpose | Timeline |
|-------|---------|----------|
| Web Summit | Launch announcement | Year 1 Q4 |
| KubeCon | Developer adoption | Year 2 Q1 |
| API World | Protocol showcase | Year 2 Q2 |
| IETF Meeting | Standards engagement | Year 2 Q3 |
| W3C TPAC | Web standards alignment | Year 2 Q4 |
| AI Summit | AI-native protocol positioning | Year 2 Q2 |

### 4.5 Success Criteria

- [ ] 1,000+ registered vendors
- [ ] 5+ platform integrations live
- [ ] SDKs in 5+ languages
- [ ] 2+ blog posts from major tech companies mentioning AVP
- [ ] Invited to speak at major conference

---

## 5. Phase 3: De Facto Standard (Year 3)

### 5.1 Goals

- 10,000+ vendors
- Multiple competing registries
- Protocol recognized by industry analysts
- Begin formal standardization process

### 5.2 Network Effects

```
                 NETWORK EFFECT FLYWHEEL

                      ┌──────────┐
                      │ More     │
                 ┌───▶│ Vendors  │───┐
                 │    └──────────┘   │
                 │                   │
          ┌──────┴──┐          ┌────▼─────┐
          │ More    │          │ More     │
          │ Value   │          │ Buyers   │
          │ for All │          │          │
          └──────┬──┘          └────┬─────┘
                 │                   │
                 │    ┌──────────┐   │
                 └────│ More     │◀──┘
                      │ Deals    │
                      └──────────┘
```

### 5.3 Federation

By Year 3, multiple registries should exist:

```
Global Registry (avp-protocol.org)
    │
    ├── EU Registry (eu.avp-protocol.org)
    │   └── GDPR-compliant data residency
    │
    ├── India Registry (in.avp-protocol.org)
    │   └── DPDPA-compliant
    │
    ├── Industry Registry: Healthcare (health.avp-protocol.org)
    │   └── HIPAA-compliant vendors only
    │
    └── Private Registries (enterprise.company.com)
        └── Internal vendor management
```

### 5.4 Success Criteria

- [ ] 10,000+ vendors
- [ ] 3+ independent registries
- [ ] Gartner/Forrester mentions AVP in reports
- [ ] 100+ deals per month
- [ ] Formal standardization process begun

---

## 6. Phase 4: Formal Standard (Year 4+)

### 6.1 Standards Bodies

| Body | What To Submit | Timeline |
|------|---------------|----------|
| **IETF** | RFC for protocol (transport, auth, discovery) | Year 3-4 |
| **W3C** | Recommendation for well-known URI + discovery | Year 3-4 |
| **OASIS** | Business communication standard | Year 4 |
| **ISO** | ISO standard (if sufficient adoption) | Year 5+ |

### 6.2 IETF RFC Path

```
Step 1: Internet-Draft (I-D)
│   Submit draft-avp-protocol-00
│   Duration: 6 months
│
▼
Step 2: Working Group Adoption
│   IETF creates AVP WG (or fits into existing WG)
│   Duration: 3-6 months
│
▼
Step 3: WG Review + Iteration
│   Multiple draft revisions
│   Duration: 12-18 months
│
▼
Step 4: IETF Last Call
│   Community-wide review
│   Duration: 2-4 weeks
│
▼
Step 5: IESG Review
│   Area Directors review
│   Duration: 1-3 months
│
▼
Step 6: RFC Publication
│   Informational or Standards Track
│   Duration: 1-2 months
```

### 6.3 IANA Registrations

| Registration | Type | Details |
|-------------|------|---------|
| `/.well-known/avp.json` | Well-Known URI | RFC 8615 |
| `AVP-Version` header | HTTP Header | RFC 7230 |
| `AVP-Agent-ID` header | HTTP Header | RFC 7230 |
| `AVP-Signature` header | HTTP Header | RFC 7230 |
| `application/avp+json` | Media Type | RFC 6838 |

---

## 7. Governance for Standardization

### 7.1 Governance Evolution

```
Year 1: Benevolent Dictator (founder-led)
    │
    ▼
Year 2: Steering Committee (5-7 members)
    │   - 2 founders
    │   - 3 community-elected
    │   - 2 corporate sponsors
    │
    ▼
Year 3: Foundation
    │   - AVP Foundation (non-profit)
    │   - Formal governance charter
    │   - Corporate membership tiers
    │
    ▼
Year 4+: Standards Body Working Group
    - IETF WG or W3C CG
    - Open participation
    - Consensus-based
```

### 7.2 Corporate Membership

| Tier | Annual Fee | Benefits |
|------|-----------|---------|
| Platinum | $100K | Board seat, logo on spec, early access |
| Gold | $50K | Advisory role, logo on website |
| Silver | $10K | Listed as supporter |
| Community | Free | Participation in AEPs and discussions |

---

## 8. Comparison with Existing Standards

### 8.1 How AVP Relates to Other Protocols

| Protocol | Relationship to AVP |
|----------|-------------------|
| HTTP/HTTPS | Transport layer — AVP uses HTTPS |
| OAuth 2.0 | Inspiration for auth model — AVP has custom auth |
| OpenAPI | Complementary — AVP agents can also publish OpenAPI docs |
| robots.txt | Precedent — `/.well-known/avp.json` follows similar convention |
| Schema.org | Complementary — AVP capabilities could map to Schema.org vocabulary |
| MCP (Model Context Protocol) | Different scope — MCP is for tool-model communication; AVP is for agent-agent business communication |
| ActivityPub | Inspiration — Federated social protocol; AVP is federated business protocol |

### 8.2 What Makes AVP Unique

1. **Business-native** — Not just communication; understands RFPs, proposals, pricing
2. **AI-native** — Designed for AI agent interaction, not human APIs
3. **Trust-integrated** — Business verification and trust scoring are core, not add-ons
4. **Negotiation-aware** — Multi-round negotiation with state machine is built-in
5. **Decentralized** — No single authority controls the network

---

## 9. Adoption Incentive Framework

### 9.1 For Vendors

| Incentive | Phase | Details |
|-----------|-------|---------|
| Free premium verification | Alpha/Beta | First 100 vendors get lifetime premium |
| Lead generation | All | AVP brings qualified leads |
| SEO benefit | Post-launch | `avp.json` as a signal for business legitimacy |
| Ecosystem visibility | All | Listed in registry, discoverable by AI |
| Integration support | Alpha | Free engineering support for integration |

### 9.2 For Buyers

| Incentive | Phase | Details |
|-----------|-------|---------|
| Free Chrome extension | All | Always free for buyers |
| Better vendor matching | All | AI-powered, not manual |
| Time savings | All | Hours → minutes for vendor discovery |
| Transparent pricing | All | Comparable proposals |
| Negotiation AI | Beta+ | AI handles negotiation |

### 9.3 For Platforms

| Incentive | Phase | Details |
|-----------|-------|---------|
| Revenue share | Launch+ | Commission on deals originating from platform |
| Differentiation | All | "AVP-enabled" as a selling point |
| Data enrichment | All | AVP data enriches platform profiles |
| Network access | All | Access to AVP vendor network |

---

*End of Standardization Roadmap*
