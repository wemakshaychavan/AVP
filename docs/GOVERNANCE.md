# AVP Protocol Governance

**Decision-Making, Community Management, and Protocol Evolution**

Version: 1.0-draft
Date: 2026-03-18

---

## Table of Contents

1. [Governance Model](#1-governance-model)
2. [Organizational Structure](#2-organizational-structure)
3. [Decision-Making Process](#3-decision-making-process)
4. [Protocol Evolution](#4-protocol-evolution)
5. [Community Guidelines](#5-community-guidelines)
6. [Intellectual Property](#6-intellectual-property)
7. [Conflict Resolution](#7-conflict-resolution)

---

## 1. Governance Model

### 1.1 Evolution of Governance

```
Year 1: Founder-Led
│  Fast decisions, clear vision
│  Risk: bus factor = 1
│
▼
Year 2: Steering Committee
│  5-7 members, balanced representation
│  Community input via AEPs
│
▼
Year 3+: Foundation
│  AVP Foundation (non-profit)
│  Corporate sponsors
│  Formal charter
│
▼
Year 4+: Standards Body WG
   IETF WG or W3C CG
   Open participation
   Consensus-based decisions
```

### 1.2 Core Principles

1. **Openness** — All decisions, discussions, and specifications are public
2. **Meritocracy** — Influence comes from contribution, not title
3. **Consensus** — Major decisions require broad agreement
4. **Transparency** — Meeting notes, votes, and rationale published
5. **Inclusivity** — Anyone can participate, regardless of affiliation

---

## 2. Organizational Structure

### 2.1 Roles

```
┌──────────────────────────────────────────────┐
│            GOVERNANCE STRUCTURE                │
│                                                │
│  ┌──────────────────────────────────────────┐ │
│  │ Steering Committee (5-7 members)          │ │
│  │  - Protocol direction                     │ │
│  │  - Major version decisions                │ │
│  │  - Governance changes                     │ │
│  └──────────────────────────────────────────┘ │
│                                                │
│  ┌──────────────────────────────────────────┐ │
│  │ Working Groups                            │ │
│  │  ┌────────────┐  ┌────────────────────┐  │ │
│  │  │ Protocol WG│  │ Security WG        │  │ │
│  │  │ (spec)     │  │ (auth, trust)      │  │ │
│  │  └────────────┘  └────────────────────┘  │ │
│  │  ┌────────────┐  ┌────────────────────┐  │ │
│  │  │ SDK WG     │  │ Ecosystem WG       │  │ │
│  │  │ (impls)    │  │ (integrations)     │  │ │
│  │  └────────────┘  └────────────────────┘  │ │
│  └──────────────────────────────────────────┘ │
│                                                │
│  ┌──────────────────────────────────────────┐ │
│  │ Community                                 │ │
│  │  - Contributors (code, docs, reviews)     │ │
│  │  - Users (vendors, buyers, platforms)     │ │
│  │  - Advocates (evangelists, writers)        │ │
│  └──────────────────────────────────────────┘ │
└──────────────────────────────────────────────┘
```

### 2.2 Steering Committee Composition

| Seat | Selection | Term |
|------|-----------|------|
| Founder (1) | Permanent (Year 1-3), then elected | N/A |
| Technical Lead (1) | Appointed by founder, then elected | 2 years |
| Community Elected (3) | Voted by contributors | 2 years, staggered |
| Corporate Sponsor (2) | Nominated by top sponsors | 1 year |

### 2.3 Working Group Structure

Each WG has:
- **Chair** — Facilitates meetings, drives agenda
- **Members** — Active contributors (minimum 3)
- **Scope** — Defined area of responsibility
- **Meeting cadence** — Bi-weekly (minimum)
- **Decision authority** — Within scope, escalate to SC if cross-cutting

---

## 3. Decision-Making Process

### 3.1 Decision Types

| Type | Who Decides | Process |
|------|-------------|---------|
| Bug fix | Any maintainer | PR review + merge |
| Minor feature | Working Group | WG consensus |
| AEP (protocol change) | Steering Committee | AEP process (see below) |
| Major version | Steering Committee | Supermajority (5/7) |
| Governance change | Steering Committee + Community | Supermajority + 30-day comment |
| License change | All contributors | Unanimous consent |

### 3.2 AEP (AVP Enhancement Proposal) Process

```
Phase 1: Draft (Author)
│  Write AEP document
│  Submit as PR to avp-spec repo
│
▼
Phase 2: Discussion (14 days minimum)
│  Community reviews and comments
│  Author addresses feedback
│  WG provides technical assessment
│
▼
Phase 3: Working Group Review
│  Relevant WG evaluates feasibility
│  Security WG reviews security impact
│  Provides recommendation to SC
│
▼
Phase 4: Steering Committee Vote
│  Requires 2/3 majority
│  At least 2 independent implementations must exist
│  (or commit to implement within 90 days)
│
▼
Phase 5: Implementation
│  Merged into spec
│  Reference implementation updated
│  SDK updates coordinated
│
▼
Phase 6: Release
│  Included in next spec version
│  Migration guide if breaking
```

### 3.3 AEP Document Template

```markdown
# AEP-{number}: {Title}

- **Author:** {name}
- **Status:** Draft | Discussion | Accepted | Rejected | Withdrawn
- **Created:** {date}
- **Updated:** {date}

## Summary
One paragraph summary.

## Motivation
Why is this change needed?

## Specification
Detailed technical specification.

## Backward Compatibility
Impact on existing implementations.

## Security Considerations
Security implications.

## Reference Implementation
Link to implementation (or planned timeline).

## Alternatives Considered
What other approaches were evaluated?
```

---

## 4. Protocol Evolution

### 4.1 Versioning Policy

- **Patch (1.0.x):** Bug fixes, clarifications — no AEP needed
- **Minor (1.x.0):** Backward-compatible additions — requires AEP
- **Major (x.0.0):** Breaking changes — requires AEP + migration guide + 12-month deprecation

### 4.2 Release Schedule

| Release Type | Frequency | Notice Period |
|-------------|-----------|---------------|
| Patch | As needed | None |
| Minor | Quarterly | 30 days |
| Major | Annually (max) | 12 months |

### 4.3 Backward Compatibility

Every protocol change MUST:
1. Include backward compatibility analysis
2. Provide migration guide for breaking changes
3. Support previous major version for 24 months minimum
4. Include version negotiation mechanism

---

## 5. Community Guidelines

### 5.1 Code of Conduct

AVP follows the [Contributor Covenant](https://www.contributor-covenant.org/) v2.1.

Key points:
- Be respectful and inclusive
- Focus on constructive feedback
- No harassment, discrimination, or personal attacks
- Disagree on ideas, not people

### 5.2 Contribution Guidelines

1. **Code contributions** — PR to relevant repo, follow coding standards
2. **Spec contributions** — AEP process for changes, PR for typos/clarifications
3. **Documentation** — PR to docs repo, reviewed by WG
4. **Bug reports** — GitHub Issues with reproduction steps
5. **Feature requests** — GitHub Discussions or AEP

### 5.3 Recognition

| Contribution Level | Recognition |
|-------------------|-------------|
| First PR merged | Listed in CONTRIBUTORS.md |
| 10+ meaningful contributions | "Core Contributor" badge |
| WG membership | Listed on website |
| Steering Committee | Listed on website + governance docs |
| Major feature/AEP accepted | Acknowledged in release notes |

---

## 6. Intellectual Property

### 6.1 License Structure

| Asset | License | Rationale |
|-------|---------|-----------|
| Protocol specification | CC BY 4.0 | Maximum adoption, attribution required |
| Reference implementations | Apache 2.0 | Permissive, patent grant |
| SDKs | Apache 2.0 | Permissive, patent grant |
| Website/docs | CC BY 4.0 | Open documentation |
| Brand/logo | Trademark | Protect protocol identity |

### 6.2 Patent Policy

- All contributors grant royalty-free patent license for AVP implementations
- No contributor may assert patents against AVP implementations
- RAND-Z (Reasonable and Non-Discriminatory, Zero-royalty) commitment

### 6.3 Trademark Policy

- "AVP" and "AI Vendor Protocol" are trademarks of AVP Foundation (once established)
- Use permitted for:
  - Referring to the protocol ("implements AVP")
  - Compatibility claims ("AVP-compatible")
- Use requires permission for:
  - Implying endorsement
  - Modified protocol versions
  - Commercial products using AVP branding

---

## 7. Conflict Resolution

### 7.1 Technical Disagreements

```
Step 1: Discussion in PR/Issue (public)
    │
    ▼
Step 2: Working Group mediation
    │
    ▼
Step 3: Steering Committee decision (binding)
```

### 7.2 Community Conflicts

```
Step 1: Private discussion with Code of Conduct team
    │
    ▼
Step 2: Mediation by neutral community member
    │
    ▼
Step 3: Steering Committee decision
    │   Possible outcomes: warning, temporary ban, permanent ban
```

### 7.3 Corporate Disputes

```
Step 1: Contact Steering Committee
    │
    ▼
Step 2: Mediation by independent party
    │
    ▼
Step 3: Binding arbitration (if foundation established)
```

---

*End of Governance Document*
