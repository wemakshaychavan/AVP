# AVP — AI Vendor Protocol

**The Open Standard for AI-to-AI Business Discovery, Communication, and Negotiation**

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![Protocol Version](https://img.shields.io/badge/Protocol-v1.0--draft-orange.svg)](docs/PROTOCOL-SPEC.md)

---

## What is AVP?

AVP (AI Vendor Protocol) is an open protocol that enables AI agents on different websites to discover each other, exchange capabilities, negotiate proposals, and transact — without human intermediation.

Think of it as **DNS + HTTP + OAuth, but for AI business agents**.

### The Problem

Today, finding and evaluating technology vendors is manual, slow, and opaque:

- Businesses spend weeks researching vendors on Clutch, GoodFirms, LinkedIn
- RFPs are sent via email, responses are inconsistent and hard to compare
- Pricing is hidden behind "contact us" walls
- There's no way for buyer AI agents to talk to vendor AI agents

### The Solution

AVP defines a standard way for any website to expose an AI agent endpoint that can:

1. **Declare capabilities** — what services, technologies, and domains a vendor covers
2. **Receive structured requests** — RFPs, capability queries, pricing inquiries
3. **Respond with proposals** — structured, comparable, machine-readable proposals
4. **Negotiate terms** — multi-round AI-to-AI negotiation with guardrails
5. **Establish trust** — verified business identity, reputation scores, transaction history

---

## How It Works

```
┌─────────────────┐         ┌─────────────────────────┐
│  Buyer Website   │         │   Vendor Website        │
│                  │         │                         │
│  ┌────────────┐  │  AVP    │  ┌───────────────────┐  │
│  │ AVP Buyer  │──┼────────▶│  │ AVP Vendor Agent  │  │
│  │   Agent    │◀─┼────────┤│  │                   │  │
│  └────────────┘  │Protocol │  └───────────────────┘  │
│                  │         │                         │
└─────────────────┘         └─────────────────────────┘
        │                              │
        │         ┌──────────┐         │
        └────────▶│ AVP      │◀────────┘
                  │ Registry │
                  │ (Trust)  │
                  └──────────┘
```

1. **Discovery** — Buyer agent finds vendors via `/.well-known/avp.json`
2. **Handshake** — Mutual authentication via AVP Auth Layer
3. **Request** — Buyer sends structured RFP to vendor's `/avp/agent` endpoint
4. **Response** — Vendor agent returns structured proposal
5. **Negotiation** — Multi-round communication until terms are agreed or rejected
6. **Settlement** — Agreement finalized, optionally recorded on AVP Registry

---

## Documentation

| Document | Description |
|----------|-------------|
| [Protocol Specification](docs/PROTOCOL-SPEC.md) | Complete AVP protocol spec (v1.0-draft) |
| [Architecture](docs/ARCHITECTURE.md) | System architecture, components, data flow |
| [Auth Layer](docs/AUTH-LAYER.md) | Authentication, authorization, key management |
| [Business Verification](docs/BUSINESS-VERIFICATION.md) | Identity verification, trust scoring |
| [Integration Guide](docs/INTEGRATION-GUIDE.md) | How to integrate AVP into existing systems |
| [Go-Live Plan](docs/GO-LIVE-PLAN.md) | Deployment, rollout, and launch strategy |
| [Standardization Roadmap](docs/STANDARDIZATION-ROADMAP.md) | Path to becoming an internet standard |
| [API Reference](docs/API-REFERENCE.md) | Complete API documentation |
| [Security](docs/SECURITY.md) | Security model, threat analysis, mitigations |
| [Governance](docs/GOVERNANCE.md) | Protocol governance, decision-making, evolution |
| [Project Plan](docs/PROJECT-PLAN.md) | Comprehensive project plan with phases and milestones |

---

## Quick Start

### For Vendors (Expose your AI agent)

```bash
npm install @avp/vendor-sdk
```

```javascript
const { createVendorAgent } = require('@avp/vendor-sdk');

const agent = createVendorAgent({
  name: 'Acme Tech Solutions',
  capabilities: {
    technologies: ['java', 'spring-boot', 'react'],
    domains: ['travel', 'e-commerce', 'fintech'],
    services: ['custom-development', 'consulting', 'maintenance']
  },
  proposalHandler: async (rfp) => {
    // Your AI generates proposals based on incoming RFPs
    return generateProposal(rfp);
  }
});

agent.listen(3000);
// Now discoverable at /.well-known/avp.json
```

### For Buyers (Discover and negotiate)

```bash
npm install @avp/buyer-sdk
```

```javascript
const { createBuyerAgent } = require('@avp/buyer-sdk');

const buyer = createBuyerAgent({ apiKey: 'your-avp-key' });

const results = await buyer.search({
  technologies: ['java'],
  domains: ['travel'],
  budget: { min: 50000, max: 200000, currency: 'USD' },
  timeline: '3 months'
});

// results = ranked list of verified vendors with proposals
```

---

## Protocol Version

Current: **v1.0-draft** (Pre-RFC)

See [STANDARDIZATION-ROADMAP.md](docs/STANDARDIZATION-ROADMAP.md) for the path to v1.0 stable.

---

## License

Apache License 2.0 — See [LICENSE](LICENSE) for details.

The protocol specification itself is released under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) to ensure maximum adoption.

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

**AVP is not just software — it's infrastructure for the AI-native economy.**
