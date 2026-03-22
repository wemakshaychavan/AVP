# AVP Integration Ecosystem v1.0-draft

**Integration Roadmap for the AI Vendor Protocol**

Status: Draft
Version: 1.0-draft
Date: 2026-03-22
Authors: AVP Working Group

---

## Table of Contents

1. [Overview](#1-overview)
2. [CRM & Sales](#2-crm--sales)
3. [Communication & Notifications](#3-communication--notifications)
4. [E-Signing & Legal](#4-e-signing--legal)
5. [Payments & Billing](#5-payments--billing)
6. [Project Management](#6-project-management)
7. [Cloud Marketplaces](#7-cloud-marketplaces)
8. [Vendor Directories](#8-vendor-directories)
9. [CMS & Website Builders](#9-cms--website-builders)
10. [AI & LLM Platforms](#10-ai--llm-platforms)
11. [Identity & Compliance](#11-identity--compliance)
12. [Analytics & Monitoring](#12-analytics--monitoring)
13. [ERP Systems](#13-erp-systems)
14. [Browser Extensions](#14-browser-extensions)
15. [Building Custom Integrations](#15-building-custom-integrations)

---

## 1. Overview

### 1.1 Purpose

This document maps out the integration ecosystem for the AI Vendor Protocol. Each section describes how AVP connects to a category of third-party platforms, which API hooks are involved, and what configuration is required on the AVP agent side.

The goal is to make AVP useful in real business workflows from day one. An RFP that arrives via the protocol should flow into the tools teams already use — CRMs, Slack channels, billing systems — without manual copy-paste.

### 1.2 Integration Maturity Model

| Status | Meaning |
|--------|---------|
| **Available** | Shipped, tested, documented. Ready for production use. |
| **In Development** | Active implementation underway. Expected within 1-2 release cycles. |
| **Planned** | On the roadmap with a defined scope. Not yet in active development. |
| **Community** | Contributed or maintained by the community. Not officially supported. |

### 1.3 Architecture

All integrations follow a consistent pattern: the AVP agent emits webhook events (see [API Reference, Section 11](API-REFERENCE.md#11-webhook-events)) and exposes REST endpoints. Integrations subscribe to relevant events and call AVP endpoints to read or write data.

```
┌──────────────────────────────────────────────────────┐
│                    AVP Agent                          │
│                                                       │
│  Webhooks out ──►  avp.rfp.received                  │
│                    avp.proposal.accepted              │
│                    avp.settlement.reached             │
│                    avp.negotiation.round              │
│                                                       │
│  Endpoints in ◄──  POST /avp/rfp                     │
│                    POST /avp/negotiate                │
│                    GET  /avp/status/session/{id}      │
│                    GET  /avp/analytics/usage          │
└───────────┬──────────────────────────┬───────────────┘
            │                          │
     ┌──────▼──────┐           ┌──────▼──────┐
     │ CRM / Slack │           │ Stripe /    │
     │ / Jira      │           │ DocuSign    │
     └─────────────┘           └─────────────┘
```

---

## 2. CRM & Sales

### 2.1 Why It Matters

CRM integration turns AVP from a protocol into a sales pipeline tool. When an RFP arrives, it should appear as a deal or opportunity in the CRM the sales team already watches. When a settlement is reached, the CRM record should update automatically.

### 2.2 Platforms

| Platform | Integration Type | Status | Description |
|----------|-----------------|--------|-------------|
| Salesforce | Managed Package | Planned | Maps AVP sessions to Salesforce Opportunities. Auto-creates Contacts from buyer agent metadata. |
| HubSpot | Native App | Planned | Creates HubSpot Deals from inbound RFPs. Syncs negotiation stages to deal pipeline stages. |
| Pipedrive | REST Webhook | Community | Pushes AVP events to Pipedrive via their webhooks API. |
| Zoho CRM | REST Webhook | Community | Maps AVP sessions to Zoho Potentials with custom fields. |
| Microsoft Dynamics 365 | Power Automate Connector | Community | Flow-based integration using Power Automate triggers. |

### 2.3 Key API Hooks

- **`avp.rfp.received`** — Creates a new deal/opportunity in the CRM.
- **`avp.proposal.accepted`** — Moves the deal to "Proposal Accepted" stage.
- **`avp.settlement.reached`** — Moves the deal to "Closed Won" and attaches settlement terms.
- **`avp.proposal.rejected`** — Moves the deal to "Closed Lost" with rejection reason.
- **`GET /avp/status/session/{id}`** — Polled to sync negotiation progress.

### 2.4 Configuration Example

```json
{
  "integration": "crm.salesforce",
  "enabled": true,
  "config": {
    "instance_url": "https://yourorg.my.salesforce.com",
    "auth": {
      "type": "oauth2",
      "client_id": "3MVG9...",
      "client_secret_ref": "$AVP_SF_CLIENT_SECRET"
    },
    "mappings": {
      "rfp_received": {
        "object": "Opportunity",
        "stage": "Qualification",
        "fields": {
          "Name": "{{session.buyer_org}} - {{rfp.title}}",
          "Amount": "{{rfp.budget_max}}",
          "CloseDate": "{{rfp.deadline}}"
        }
      },
      "settlement_reached": {
        "stage": "Closed Won",
        "fields": {
          "Amount": "{{settlement.agreed_price}}"
        }
      }
    },
    "subscribe_events": [
      "avp.rfp.received",
      "avp.proposal.accepted",
      "avp.proposal.rejected",
      "avp.settlement.reached"
    ]
  }
}
```

---

## 3. Communication & Notifications

### 3.1 Why It Matters

Teams need to know when things happen. A high-value RFP that sits unnoticed for hours is a missed opportunity. Communication integrations ensure the right people are notified in real time and can trigger human escalation directly from their messaging tool.

### 3.2 Platforms

| Platform | Integration Type | Status | Description |
|----------|-----------------|--------|-------------|
| Slack | Bot + Slash Commands | In Development | Posts RFP summaries to channels. `/avp status` shows active sessions. Buttons for human escalation. |
| Microsoft Teams | Bot Framework | Community | Adaptive Cards for RFP notifications. Actionable messages for approval workflows. |
| Discord | Bot | Community | Lightweight notifications for smaller teams and open-source vendor communities. |
| Email (SMTP) | Webhook-to-Email | Community | Sends formatted email digests for AVP events. Useful as a fallback channel. |
| Twilio (SMS) | REST API | Community | SMS alerts for high-priority escalations when team members are away from desktop. |

### 3.3 Key API Hooks

- **`avp.rfp.received`** — Posts a formatted summary to the configured channel.
- **`avp.escalation.requested`** — Sends an urgent notification with session context and a link to intervene.
- **`avp.negotiation.round`** — Optional per-round updates (configurable; off by default to reduce noise).
- **`avp.session.expired`** — Alerts the team that a session timed out without settlement.
- **`POST /avp/negotiate/escalate`** — Called when a user clicks "Escalate" in Slack/Teams.

### 3.4 Configuration Example

```json
{
  "integration": "communication.slack",
  "enabled": true,
  "config": {
    "bot_token_ref": "$AVP_SLACK_BOT_TOKEN",
    "channels": {
      "rfp_notifications": "#avp-deals",
      "escalations": "#avp-urgent",
      "settlements": "#avp-wins"
    },
    "notification_rules": [
      {
        "event": "avp.rfp.received",
        "channel": "rfp_notifications",
        "mention": "@sales-team",
        "filter": { "rfp.budget_max": { "$gte": 1000 } }
      },
      {
        "event": "avp.escalation.requested",
        "channel": "escalations",
        "mention": "@channel"
      }
    ],
    "slash_commands": {
      "/avp-status": true,
      "/avp-sessions": true,
      "/avp-escalate": true
    },
    "subscribe_events": [
      "avp.rfp.received",
      "avp.escalation.requested",
      "avp.settlement.reached",
      "avp.session.expired"
    ]
  }
}
```

---

## 4. E-Signing & Legal

### 4.1 Why It Matters

AVP settlements define agreed terms, but many businesses require a signed contract before work begins. E-signing integrations bridge the gap: when a settlement is reached, a signing envelope is automatically generated from the settlement terms and sent to both parties.

### 4.2 Platforms

| Platform | Integration Type | Status | Description |
|----------|-----------------|--------|-------------|
| DocuSign | eSignature API | Planned | Auto-generates envelopes from AVP settlement data. Tracks signing status back to the AVP session. |
| PandaDoc | REST API | Community | Creates documents from templates populated with settlement terms. |
| HelloSign (Dropbox Sign) | REST API | Community | Lightweight signing flow for smaller agreements. |
| Adobe Acrobat Sign | REST API | Community | Enterprise signing with Adobe ecosystem integration. |

### 4.3 Key API Hooks

- **`avp.settlement.reached`** — Triggers envelope creation with settlement terms as merge fields.
- **`GET /avp/status/session/{id}`** — Retrieves full session history to include in the contract appendix.
- **Inbound webhook from DocuSign** — When signing is complete, the integration calls `POST /avp/settle` to update the settlement record with the signed document reference.

### 4.4 Configuration Example

```json
{
  "integration": "esign.docusign",
  "enabled": true,
  "config": {
    "account_id": "ae30ea4e-...",
    "auth": {
      "type": "jwt",
      "integration_key": "412af...",
      "private_key_ref": "$AVP_DOCUSIGN_PRIVATE_KEY"
    },
    "template_id": "tmpl_contract_standard_v2",
    "auto_send": false,
    "signers": {
      "vendor": {
        "name": "{{agent.org_name}}",
        "email": "{{agent.contact_email}}"
      },
      "buyer": {
        "name": "{{session.buyer_org}}",
        "email": "{{session.buyer_contact_email}}"
      }
    },
    "field_mappings": {
      "scope_of_work": "{{settlement.terms.scope}}",
      "total_price": "{{settlement.agreed_price}}",
      "delivery_date": "{{settlement.terms.timeline}}"
    },
    "subscribe_events": [
      "avp.settlement.reached"
    ]
  }
}
```

---

## 5. Payments & Billing

### 5.1 Why It Matters

A settlement without payment collection is just a handshake. Payment integrations let vendors automatically generate invoices or payment links when terms are agreed, closing the loop from discovery to revenue.

### 5.2 Platforms

| Platform | Integration Type | Status | Description |
|----------|-----------------|--------|-------------|
| Stripe | Payment Links + Invoicing API | Planned | Creates Stripe invoices from settlement terms. Supports milestone-based billing. |
| QuickBooks Online | REST API | Community | Generates QBO invoices and syncs payment status back to AVP. |
| Xero | REST API | Community | Creates Xero invoices with line items mapped from settlement terms. |
| PayPal | REST API | Community | Generates PayPal invoices for international transactions. |
| Square | REST API | Community | Invoice and payment link generation for SMB vendors. |

### 5.3 Key API Hooks

- **`avp.settlement.reached`** — Creates an invoice or payment link based on settlement terms.
- **`avp.proposal.accepted`** — Optionally creates a proforma invoice or deposit request.
- **Inbound webhook from Stripe** — `invoice.paid` event updates the AVP session with payment confirmation.
- **`GET /avp/status/session/{id}`** — Used to reconcile payment records with AVP session data.

### 5.4 Configuration Example

```json
{
  "integration": "payments.stripe",
  "enabled": true,
  "config": {
    "api_key_ref": "$AVP_STRIPE_SECRET_KEY",
    "auto_invoice": true,
    "invoice_settings": {
      "days_until_due": 30,
      "currency": "usd",
      "tax_behavior": "exclusive"
    },
    "line_item_mapping": {
      "description": "{{settlement.terms.scope}}",
      "amount": "{{settlement.agreed_price}}",
      "metadata": {
        "avp_session_id": "{{session.id}}",
        "avp_rfp_id": "{{rfp.id}}"
      }
    },
    "milestone_billing": {
      "enabled": false,
      "milestones_field": "settlement.terms.milestones"
    },
    "subscribe_events": [
      "avp.settlement.reached"
    ]
  }
}
```

---

## 6. Project Management

### 6.1 Why It Matters

Once a deal closes, work needs to begin. Project management integrations auto-create projects, tasks, and timelines from settlement terms so delivery teams can start immediately without re-entering scope details.

### 6.2 Platforms

| Platform | Integration Type | Status | Description |
|----------|-----------------|--------|-------------|
| Jira | REST API + Webhooks | Planned | Creates Jira projects/epics from settlement scope. Maps milestones to issues. |
| Asana | REST API | Community | Auto-creates Asana projects with tasks derived from settlement deliverables. |
| Linear | GraphQL API | Community | Creates Linear projects and issues. Popular with engineering-heavy vendors. |
| Trello | REST API | Community | Creates Trello boards with lists mapped to project phases. |
| Monday.com | REST API | Community | Generates Monday boards from settlement terms and timelines. |
| Notion | REST API | Community | Creates Notion database entries and project pages from settlements. |

### 6.3 Key API Hooks

- **`avp.settlement.reached`** — Creates a project with tasks/issues based on agreed deliverables.
- **`avp.proposal.accepted`** — Optionally creates a draft project for pre-work planning.
- **`GET /avp/status/session/{id}`** — Retrieves session and settlement details for project setup.

### 6.4 Configuration Example

```json
{
  "integration": "pm.jira",
  "enabled": true,
  "config": {
    "base_url": "https://yourteam.atlassian.net",
    "auth": {
      "type": "api_token",
      "email": "bot@yourteam.com",
      "token_ref": "$AVP_JIRA_API_TOKEN"
    },
    "project_template": {
      "project_key": "AVP",
      "issue_type": "Epic",
      "summary": "{{session.buyer_org}}: {{rfp.title}}",
      "description": "Settlement terms:\n{{settlement.terms.scope}}",
      "priority": "Medium"
    },
    "subtask_mapping": {
      "source_field": "settlement.terms.deliverables",
      "issue_type": "Task",
      "assignee": "unassigned"
    },
    "subscribe_events": [
      "avp.settlement.reached"
    ]
  }
}
```

---

## 7. Cloud Marketplaces

### 7.1 Why It Matters

Cloud marketplaces are where many SaaS and infrastructure purchasing decisions already happen. Listing AVP-enabled services on these marketplaces expands discovery beyond the AVP registry and lets buyers use existing procurement workflows.

### 7.2 Platforms

| Platform | Integration Type | Status | Description |
|----------|-----------------|--------|-------------|
| AWS Marketplace | SaaS Listing + Metering API | Community | Lists AVP-enabled services. Meters usage for AWS-billed transactions. |
| Azure Marketplace | SaaS Fulfillment API | Community | Integrates AVP agent provisioning with Azure subscription lifecycle. |
| Google Cloud Marketplace | Partner Ingestion API | Community | Lists services and handles GCP-billed procurement. |
| Shopify App Store | Shopify App | Community | Enables Shopify merchants to use AVP for sourcing services. |

### 7.3 Key API Hooks

- **`POST /avp/capabilities`** — Syncs vendor capability data to marketplace listings.
- **`avp.settlement.reached`** — Reports metering/usage events to the marketplace billing API.
- **`GET /avp/analytics/usage`** — Provides usage data for marketplace analytics dashboards.

### 7.4 Configuration Example

```json
{
  "integration": "marketplace.aws",
  "enabled": true,
  "config": {
    "product_code": "prod-abc123xyz",
    "aws_region": "us-east-1",
    "auth": {
      "type": "iam_role",
      "role_arn": "arn:aws:iam::123456789012:role/AVPMarketplaceRole"
    },
    "metering": {
      "enabled": true,
      "dimension": "avp_sessions",
      "unit": "session"
    },
    "sync_capabilities": true,
    "subscribe_events": [
      "avp.settlement.reached"
    ]
  }
}
```

---

## 8. Vendor Directories

### 8.1 Why It Matters

Vendor directories are where buyers go to find service providers. Integrations with these platforms let AVP agents automatically maintain up-to-date profiles, pulling capabilities and trust scores from the AVP registry into directory listings.

### 8.2 Platforms

| Platform | Integration Type | Status | Description |
|----------|-----------------|--------|-------------|
| G2 | REST API | Community | Syncs AVP trust scores and capabilities to G2 vendor profiles. |
| Capterra | Data Feed | Community | Automated profile updates from AVP agent metadata. |
| Clutch | REST API | Community | Pushes AVP-verified review data and project history to Clutch profiles. |
| Product Hunt | REST API | Community | Announces AVP-enabled products with capability metadata. |

### 8.3 Key API Hooks

- **`POST /avp/capabilities`** — Source of truth for what the vendor offers; synced to directory profiles.
- **`avp.trust.updated`** — Triggers a profile update when the vendor's trust score changes.
- **`GET /avp/analytics/conversion`** — Optionally shared with directories as social proof metrics.

### 8.4 Configuration Example

```json
{
  "integration": "directory.g2",
  "enabled": true,
  "config": {
    "api_key_ref": "$AVP_G2_API_KEY",
    "product_id": "g2-prod-12345",
    "sync_fields": {
      "capabilities": true,
      "trust_score": true,
      "response_time": true,
      "settlement_rate": true
    },
    "sync_interval": "24h",
    "subscribe_events": [
      "avp.trust.updated"
    ]
  }
}
```

---

## 9. CMS & Website Builders

### 9.1 Why It Matters

Most vendors have a website. CMS integrations let them add AVP capability to their existing site without writing code — install a plugin, configure credentials, and the site starts serving `/.well-known/avp-agent.json` and handling inbound RFPs.

### 9.2 Platforms

| Platform | Integration Type | Status | Description |
|----------|-----------------|--------|-------------|
| WordPress | Plugin | In Development | Full AVP agent as a WP plugin. Admin panel for capabilities, pricing rules, and negotiation settings. Serves the discovery endpoint automatically. |
| Webflow | Embedded Script | Community | JavaScript widget that adds AVP discovery and an RFP intake form to any Webflow site. |
| Squarespace | Code Injection | Community | Script-based integration via Squarespace's code injection feature. |
| Wix | Velo (Custom Code) | Community | AVP agent running as a Wix Velo backend module. |
| Ghost | Custom Integration | Community | Theme helper and API route for Ghost-based vendor sites. |

### 9.3 Key API Hooks

- **`GET /.well-known/avp-agent.json`** — The plugin serves this discovery endpoint from the CMS.
- **`POST /avp/handshake`** — Handled by the plugin's backend; initiates sessions.
- **`POST /avp/rfp`** — Receives inbound RFPs and stores them in the CMS database.
- **`avp.rfp.received`** — Triggers CMS-side notifications (email to admin, dashboard widget).

### 9.4 Configuration Example

```json
{
  "integration": "cms.wordpress",
  "enabled": true,
  "config": {
    "plugin_version": "0.3.0-beta",
    "agent_settings": {
      "org_name": "Acme Web Services",
      "capabilities": ["web-development", "seo", "hosting"],
      "auto_respond": true,
      "response_mode": "ai_draft_human_approve"
    },
    "discovery_endpoint": {
      "serve_well_known": true,
      "custom_path": null
    },
    "negotiation": {
      "max_rounds": 5,
      "auto_accept_below": 500,
      "require_human_above": 10000
    },
    "notifications": {
      "admin_email": "admin@acme.example.com",
      "dashboard_widget": true
    }
  }
}
```

---

## 10. AI & LLM Platforms

### 10.1 Why It Matters

AVP agents need language understanding to parse RFPs, generate proposals, and negotiate terms. LLM platform integrations let the AVP agent delegate reasoning tasks to the model provider of the vendor's choice, keeping the protocol model-agnostic while enabling sophisticated behavior.

### 10.2 Platforms

| Platform | Integration Type | Status | Description |
|----------|-----------------|--------|-------------|
| OpenAI | API (Chat Completions) | Planned | Uses GPT models for RFP analysis, proposal drafting, and negotiation strategy. |
| Anthropic | API (Messages) | Planned | Uses Claude models for structured reasoning during multi-round negotiations. |
| MCP (Model Context Protocol) | Tool Provider | Planned | Exposes AVP operations as MCP tools so any MCP-compatible agent can participate in the protocol. |
| Google Vertex AI | API | Community | Gemini models for RFP processing and response generation. |
| AWS Bedrock | API | Community | Multi-model access for AVP agents running on AWS infrastructure. |
| Ollama / Local Models | REST API | Community | Self-hosted models for vendors with data sovereignty requirements. |

### 10.3 Key API Hooks

- **`avp.rfp.received`** — Sends the RFP content to the LLM for analysis and proposal generation.
- **`avp.negotiation.round`** — Sends the latest counter-terms to the LLM for strategy evaluation.
- **`POST /avp/rfp`** — The LLM-generated proposal is submitted back through this endpoint.
- **`POST /avp/negotiate`** — The LLM's recommended counter-offer is sent via the negotiation endpoint.

### 10.4 Configuration Example

```json
{
  "integration": "ai.anthropic",
  "enabled": true,
  "config": {
    "api_key_ref": "$AVP_ANTHROPIC_API_KEY",
    "model": "claude-sonnet-4-20250514",
    "tasks": {
      "rfp_analysis": {
        "enabled": true,
        "system_prompt": "You are an AVP vendor agent. Analyze incoming RFPs and determine fit based on our capabilities.",
        "max_tokens": 2048
      },
      "proposal_generation": {
        "enabled": true,
        "system_prompt": "Draft a professional proposal responding to the RFP. Be specific about deliverables and timeline.",
        "max_tokens": 4096
      },
      "negotiation": {
        "enabled": true,
        "system_prompt": "Evaluate the buyer's counter-offer. Recommend accept, counter, or reject with reasoning.",
        "max_tokens": 1024,
        "temperature": 0.3
      }
    },
    "guardrails": {
      "max_discount_percent": 20,
      "never_accept_below": "{{agent.minimum_price}}",
      "require_human_review": ["scope_change", "timeline_under_7d"]
    },
    "subscribe_events": [
      "avp.rfp.received",
      "avp.negotiation.round"
    ]
  }
}
```

---

## 11. Identity & Compliance

### 11.1 Why It Matters

Trust is foundational to AVP. Identity and compliance integrations verify that the businesses behind AVP agents are legitimate, meet regulatory requirements, and can be held accountable. These integrations feed into the AVP trust score system.

### 11.2 Platforms

| Platform | Integration Type | Status | Description |
|----------|-----------------|--------|-------------|
| Auth0 | OIDC / OAuth2 | Community | Federated identity for AVP agent operators. SSO for the AVP admin dashboard. |
| Okta | SAML / OIDC | Community | Enterprise SSO and identity lifecycle management for AVP deployments. |
| Dun & Bradstreet | REST API | Community | Business verification via DUNS number lookup. Feeds trust score. |
| Veriforce / ISNetworld | REST API | Community | Contractor compliance verification for regulated industries. |
| OpenCorporates | REST API | Community | Company registration verification across global jurisdictions. |

### 11.3 Key API Hooks

- **`POST /v1/agents/register`** — Identity verification data is attached during agent registration.
- **`avp.trust.updated`** — Fired when a compliance check result changes the trust score.
- **`GET /v1/trust/{agent_id}`** — Returns the current trust profile including verification status.

### 11.4 Configuration Example

```json
{
  "integration": "identity.dnb",
  "enabled": true,
  "config": {
    "api_key_ref": "$AVP_DNB_API_KEY",
    "duns_number": "123456789",
    "verification_checks": [
      "business_registration",
      "financial_standing",
      "trade_references"
    ],
    "auto_refresh_interval": "30d",
    "trust_score_contribution": {
      "verified_business": 20,
      "financial_standing_good": 10,
      "trade_references_positive": 10
    },
    "subscribe_events": [
      "avp.trust.updated"
    ]
  }
}
```

---

## 12. Analytics & Monitoring

### 12.1 Why It Matters

Operators need visibility into how their AVP agent is performing: how many RFPs are coming in, conversion rates, response times, and error rates. Analytics integrations send AVP telemetry to the monitoring tools teams already use.

### 12.2 Platforms

| Platform | Integration Type | Status | Description |
|----------|-----------------|--------|-------------|
| Datadog | APM + Custom Metrics | Community | Traces AVP request lifecycle. Dashboards for RFP volume, latency, and conversion. |
| Grafana | Prometheus Exporter | Community | Exposes AVP metrics in Prometheus format for Grafana dashboards. |
| New Relic | REST API | Community | Custom events and metrics for AVP session monitoring. |
| Google Analytics | Measurement Protocol | Community | Tracks AVP widget interactions and conversion funnels on vendor websites. |
| PostHog | Event API | Community | Product analytics for AVP agent behavior and buyer interaction patterns. |
| Sentry | SDK | Community | Error tracking and alerting for AVP agent runtime failures. |

### 12.3 Key API Hooks

- **`GET /avp/analytics/usage`** — Polled periodically to push usage metrics to the monitoring platform.
- **`GET /avp/analytics/conversion`** — Conversion funnel data exported to dashboards.
- **All webhook events** — Each event is forwarded as a custom metric/event to the analytics platform.

### 12.4 Configuration Example

```json
{
  "integration": "analytics.datadog",
  "enabled": true,
  "config": {
    "api_key_ref": "$AVP_DD_API_KEY",
    "app_key_ref": "$AVP_DD_APP_KEY",
    "site": "datadoghq.com",
    "metrics_prefix": "avp.",
    "tags": ["env:production", "service:avp-agent"],
    "track_events": {
      "rfp_received": { "metric": "avp.rfp.count", "type": "count" },
      "settlement_reached": { "metric": "avp.settlement.count", "type": "count" },
      "response_time": { "metric": "avp.response.latency", "type": "gauge" }
    },
    "apm_tracing": {
      "enabled": true,
      "sample_rate": 1.0
    },
    "subscribe_events": [
      "avp.rfp.received",
      "avp.rfp.responded",
      "avp.settlement.reached",
      "avp.session.expired"
    ]
  }
}
```

---

## 13. ERP Systems

### 13.1 Why It Matters

Enterprise vendors manage procurement, inventory, and fulfillment through ERP systems. Integrating AVP with ERPs ensures that settlements translate into purchase orders, resource allocations, and delivery schedules within the vendor's operational backbone.

### 13.2 Platforms

| Platform | Integration Type | Status | Description |
|----------|-----------------|--------|-------------|
| SAP S/4HANA | OData API | Community | Creates sales orders from AVP settlements. Syncs pricing from SAP master data. |
| Oracle NetSuite | SuiteTalk REST API | Community | Maps settlements to NetSuite sales orders and customer records. |
| Microsoft Dynamics 365 Business Central | REST API | Community | Integrates AVP with BC sales order and invoicing workflows. |
| Odoo | JSON-RPC API | Community | Open-source ERP integration for SMB vendors using Odoo. |

### 13.3 Key API Hooks

- **`avp.settlement.reached`** — Creates a sales order or purchase order in the ERP.
- **`avp.rfp.received`** — Optionally checks ERP for inventory/capacity before generating a proposal.
- **`POST /avp/capabilities`** — Capability data can be dynamically derived from ERP product catalogs.

### 13.4 Configuration Example

```json
{
  "integration": "erp.sap",
  "enabled": true,
  "config": {
    "base_url": "https://myhost:443/sap/opu/odata/sap/",
    "auth": {
      "type": "basic",
      "username": "AVP_SERVICE",
      "password_ref": "$AVP_SAP_PASSWORD"
    },
    "settlement_mapping": {
      "entity_set": "SalesOrderSet",
      "fields": {
        "SalesOrganization": "1000",
        "DistributionChannel": "10",
        "SoldToParty": "{{session.buyer_org_erp_id}}",
        "PurchaseOrderByCustomer": "AVP-{{session.id}}"
      }
    },
    "capacity_check": {
      "enabled": true,
      "plant": "1000",
      "check_before_proposal": true
    },
    "subscribe_events": [
      "avp.rfp.received",
      "avp.settlement.reached"
    ]
  }
}
```

---

## 14. Browser Extensions

### 14.1 Why It Matters

Browser extensions bring AVP to the buyer side. Instead of requiring buyers to install SDKs or modify their websites, a browser extension can detect AVP-enabled vendors on any webpage, initiate RFPs, and manage negotiations from the browser toolbar.

### 14.2 Platforms

| Platform | Integration Type | Status | Description |
|----------|-----------------|--------|-------------|
| Chrome Extension | Manifest V3 | In Development | Detects `/.well-known/avp-agent.json` on visited sites. Provides a popup UI for initiating and tracking AVP sessions. |
| Firefox Add-on | WebExtension API | Community | Port of the Chrome extension for Firefox users. |
| Safari Extension | Safari Web Extension | Community | Port for Safari / macOS users. |
| Edge Extension | Manifest V3 | Community | Chromium-based; shares codebase with Chrome extension. |

### 14.3 Key API Hooks

- **`GET /.well-known/avp-agent.json`** — The extension probes this endpoint on every visited domain to detect AVP support.
- **`POST /avp/handshake`** — Initiated from the extension when the user clicks "Start RFP".
- **`POST /avp/rfp`** — Submits the buyer's RFP from the extension's form UI.
- **`GET /avp/status/session/{id}`** — Polled to show session progress in the extension popup.
- **`POST /avp/negotiate/escalate`** — Allows the buyer to request human contact from the extension.

### 14.4 Configuration Example

```json
{
  "integration": "browser.chrome",
  "enabled": true,
  "config": {
    "extension_id": "avp-chrome-ext",
    "version": "0.2.0-beta",
    "buyer_agent": {
      "agent_id": "avp:buyer:mycompany:f9e8d7c6",
      "org_name": "My Company Inc.",
      "contact_email": "procurement@mycompany.example.com"
    },
    "detection": {
      "auto_probe": true,
      "probe_on_navigation": true,
      "show_badge_when_detected": true
    },
    "defaults": {
      "budget_currency": "usd",
      "preferred_timeline": "30d",
      "auto_negotiate": false,
      "max_negotiation_rounds": 3
    },
    "ui": {
      "theme": "system",
      "show_trust_score": true,
      "show_capability_badges": true
    }
  }
}
```

---

## 15. Building Custom Integrations

For platforms not covered above, AVP provides three patterns for building custom integrations.

### 15.1 Webhook-Based Integration

The simplest approach. Register a webhook URL with the AVP agent and handle events in your own backend.

**Pattern:**

```
AVP Agent ──webhook──► Your HTTP Endpoint ──► Your System
```

**Setup:**

```bash
# Register a webhook via the AVP API
curl -X POST https://yourdomain.com/avp/webhooks \
  -H "AVP-Version: 1.0" \
  -H "AVP-Agent-ID: avp:vendor:yourorg:abc123" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://your-backend.example.com/avp-events",
    "events": [
      "avp.rfp.received",
      "avp.settlement.reached",
      "avp.escalation.requested"
    ],
    "secret": "whsec_your_signing_secret"
  }'
```

**Handler example (Node.js):**

```javascript
const crypto = require('crypto');

app.post('/avp-events', (req, res) => {
  // Verify webhook signature
  const signature = req.headers['x-avp-signature'];
  const expected = 'sha256=' + crypto
    .createHmac('sha256', process.env.AVP_WEBHOOK_SECRET)
    .update(JSON.stringify(req.body))
    .digest('hex');

  if (signature !== expected) {
    return res.status(401).json({ error: 'Invalid signature' });
  }

  const { event, data } = req.body;

  switch (event) {
    case 'avp.rfp.received':
      // Create a record in your internal system
      yourSystem.createLead(data.session_id, data.rfp);
      break;
    case 'avp.settlement.reached':
      // Trigger your fulfillment workflow
      yourSystem.startProject(data.session_id, data.settlement);
      break;
    case 'avp.escalation.requested':
      // Page the on-call team
      yourSystem.alert(data.session_id, data.reason);
      break;
  }

  res.status(200).json({ received: true });
});
```

### 15.2 SDK Extension Pattern

Extend the AVP SDK with custom middleware that intercepts events before or after they are processed.

**Pattern:**

```
Inbound Event ──► Your Middleware ──► AVP SDK Core ──► Your Middleware ──► Response
```

**Example (Node.js SDK):**

```javascript
const { AVPAgent } = require('@avp-protocol/sdk');

const agent = new AVPAgent({
  orgName: 'Custom Corp',
  capabilities: ['consulting', 'development']
});

// Register pre-processing middleware
agent.use('before:rfp', async (rfp, context) => {
  // Enrich the RFP with data from your internal systems
  const clientHistory = await yourCRM.lookup(rfp.buyer_org);
  context.metadata.client_history = clientHistory;
  context.metadata.is_returning_client = clientHistory.length > 0;
  return rfp;
});

// Register post-processing middleware
agent.use('after:settlement', async (settlement, context) => {
  // Sync to your internal project tracker
  await yourTracker.createProject({
    name: `${settlement.buyer_org} - ${settlement.rfp_title}`,
    budget: settlement.agreed_price,
    deadline: settlement.terms.timeline,
    source: 'avp',
    session_id: context.session_id
  });
});

agent.listen(3000);
```

### 15.3 MCP (Model Context Protocol) Tool Pattern

Expose AVP operations as MCP tools so that any MCP-compatible AI agent can discover vendors, submit RFPs, and negotiate through the protocol.

**Pattern:**

```
MCP Client (e.g., Claude) ──MCP──► AVP MCP Server ──► AVP Protocol
```

**MCP Tool Definitions:**

```json
{
  "tools": [
    {
      "name": "avp_discover",
      "description": "Discover AVP-enabled vendors on a given domain",
      "inputSchema": {
        "type": "object",
        "properties": {
          "domain": {
            "type": "string",
            "description": "The vendor's domain to probe for AVP support"
          }
        },
        "required": ["domain"]
      }
    },
    {
      "name": "avp_submit_rfp",
      "description": "Submit an RFP to an AVP-enabled vendor",
      "inputSchema": {
        "type": "object",
        "properties": {
          "vendor_agent_id": { "type": "string" },
          "title": { "type": "string" },
          "requirements": { "type": "string" },
          "budget_max": { "type": "number" },
          "timeline": { "type": "string" }
        },
        "required": ["vendor_agent_id", "title", "requirements"]
      }
    },
    {
      "name": "avp_negotiate",
      "description": "Send a counter-offer in an active AVP negotiation",
      "inputSchema": {
        "type": "object",
        "properties": {
          "session_id": { "type": "string" },
          "counter_terms": { "type": "object" },
          "message": { "type": "string" }
        },
        "required": ["session_id", "counter_terms"]
      }
    },
    {
      "name": "avp_accept",
      "description": "Accept the current proposal in an AVP negotiation",
      "inputSchema": {
        "type": "object",
        "properties": {
          "session_id": { "type": "string" }
        },
        "required": ["session_id"]
      }
    }
  ]
}
```

**MCP Server Example (Python):**

```python
from mcp import Server, Tool
import httpx

server = Server("avp-mcp-bridge")

@server.tool("avp_discover")
async def discover(domain: str) -> dict:
    """Probe a domain for AVP agent support."""
    async with httpx.AsyncClient() as client:
        url = f"https://{domain}/.well-known/avp-agent.json"
        response = await client.get(url, timeout=10)
        if response.status_code == 200:
            return {"supported": True, "agent": response.json()}
        return {"supported": False}

@server.tool("avp_submit_rfp")
async def submit_rfp(
    vendor_agent_id: str,
    title: str,
    requirements: str,
    budget_max: float = None,
    timeline: str = None
) -> dict:
    """Submit an RFP to a vendor's AVP agent."""
    # 1. Perform handshake
    # 2. Submit RFP
    # 3. Return session_id and initial proposal
    ...

@server.tool("avp_negotiate")
async def negotiate(
    session_id: str,
    counter_terms: dict,
    message: str = None
) -> dict:
    """Send a counter-offer."""
    ...

if __name__ == "__main__":
    server.run()
```

### 15.4 Full Custom Integration Example

Below is a complete example that combines webhooks, the SDK, and a custom internal system — a staffing agency that matches AVP RFPs to available contractors.

```json
{
  "integration": "custom.staffing_matcher",
  "enabled": true,
  "config": {
    "name": "Internal Contractor Matcher",
    "description": "Matches inbound AVP RFPs to available contractors in our staffing database.",
    "webhook_endpoint": "https://internal.staffingco.example.com/avp/events",
    "webhook_secret_ref": "$AVP_CUSTOM_WEBHOOK_SECRET",
    "subscribe_events": [
      "avp.rfp.received",
      "avp.negotiation.round",
      "avp.settlement.reached"
    ],
    "custom_logic": {
      "on_rfp_received": {
        "action": "match_contractors",
        "database": "postgresql://staffing-db:5432/contractors",
        "match_fields": ["skills", "availability", "hourly_rate"],
        "min_match_score": 0.7
      },
      "on_settlement_reached": {
        "action": "assign_contractor",
        "notify_contractor_via": "email",
        "create_timesheet": true
      }
    }
  }
}
```

---

## Appendix A: Event Quick Reference

All webhook events available for integration subscriptions:

| Event | Description | Common Integrations |
|-------|-------------|-------------------|
| `avp.rfp.received` | New RFP arrived | CRM, Slack, LLM, PM |
| `avp.rfp.responded` | Proposal sent back | CRM, Analytics |
| `avp.negotiation.started` | First round began | Slack, Analytics |
| `avp.negotiation.round` | Each subsequent round | Slack, LLM |
| `avp.proposal.accepted` | Buyer accepted terms | CRM, Payments, Slack |
| `avp.proposal.rejected` | Buyer rejected terms | CRM, Analytics |
| `avp.settlement.reached` | Final agreement | CRM, Payments, E-Sign, PM, ERP |
| `avp.session.expired` | Session timed out | Slack, Analytics |
| `avp.trust.updated` | Trust score changed | Directories, Identity |
| `avp.escalation.requested` | Human intervention needed | Slack, Teams, SMS |

## Appendix B: Status Tracker

| Integration | Status | Target Release | Maintainer |
|-------------|--------|----------------|------------|
| WordPress Plugin | In Development | v0.3.0-beta | Core Team |
| Slack Bot | In Development | v0.3.0-beta | Core Team |
| Chrome Extension | In Development | v0.2.0-beta | Core Team |
| Salesforce | Planned | v0.5.0 | Core Team |
| HubSpot | Planned | v0.5.0 | Core Team |
| Stripe | Planned | v0.4.0 | Core Team |
| DocuSign | Planned | v0.5.0 | Core Team |
| Jira | Planned | v0.5.0 | Core Team |
| OpenAI / Anthropic LLM | Planned | v0.4.0 | Core Team |
| MCP Tool Provider | Planned | v0.4.0 | Core Team |
| All others | Community | — | Community Contributors |

---

*This document is part of the [AVP Protocol Documentation](./PROTOCOL-SPEC.md). For implementation details on individual integrations, see the [Integration Guide](./INTEGRATION-GUIDE.md). For API endpoint specifics, see the [API Reference](./API-REFERENCE.md).*
