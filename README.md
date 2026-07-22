# Universal Transfer Protocol (UTP)

**Open Framework for Bounded Agent Authorization in Regulated Industries**

![UTP Badge](https://img.shields.io/badge/UTP-v1.0-blue) ![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-green) ![Status: Production](https://img.shields.io/badge/Status-Production-brightgreen)

---

## What is UTP?

The **Universal Transfer Protocol (UTP)** is an open-source framework for agents to request and receive **bounded, auditable authorization** for accessing regulated data.

In regulated industries—financial services, healthcare, legal, travel, insurance—data access is controlled. When an AI agent needs to access customer data, it must:

1. **Specify exactly what it wants** (not vague intent)
2. **Receive explicit authorization** (with full traceability)
3. **Operate within bounded scope** (time limits, data limits, action limits)
4. **Produce an immutable audit trail** (suitable for regulatory proceedings)

UTP standardizes this flow across industries.

### Core Innovation

Traditional API security assumes human-like behavior: *An API client gets a scope and honors it.*

AI agents don't behave that way. An agent's understanding of what it's allowed to do can drift over time. UTP solves this by:

- **Enumerating specific actions**, not vague scopes
- **Evaluating requests through 9 independent signal stages** (not a single probabilistic model)
- **Degrading gracefully** when uncertain (escalate to human, don't guess)
- **Sealing every decision** with a cryptographic audit trail

---

## Core Features

✅ **Bounded Actions** — Agents request specific, enumerated actions (READ, WRITE, QUERY, CONSENT, etc.), not vague data access

✅ **Multi-Signal Evaluation** — 9 independent stages evaluate authorization signals in parallel, resolving conflicts deterministically

✅ **Deterministic Fallback** — If signals are weak, the system transparently degrades (e.g., "approve READ-only, deny WRITE") instead of probabilistically guessing

✅ **Immutable Audit Trail** — Every decision is cryptographically sealed and includes full stage-by-stage findings, timestamps, and actor identity

✅ **Domain-Agnostic** — Same framework works for financial services, healthcare, legal, travel, insurance, and more

✅ **Multi-Vendor Interoperability** — Banks, fintechs, aggregators, and regulators all speak the same language

✅ **Regulatory-Ready** — Designed to satisfy GLBA, HIPAA, GDPR, ABA, and other compliance frameworks

---

## Quick Start

### 1. Read the Specification

Start here: ./SPEC.md

Takes ~30 minutes to understand the core concepts.

### 2. Explore the API

Review the OpenAPI spec: [Gateway API v1.0](./UTP_Gateway_API_v1.yaml)

Can be imported into Swagger UI, Postman, or any OpenAPI tool.

### 3. See It In Action

Financial domain example:
```json
{
  "domain": "FIN",
  "requestor": {
    "agentId": "fintech-credit-scorer-v3",
    "agentOperator": "acme-fintech@example.com",
    "humanPrincipal": "john.doe@acme.com"
  },
  "dataSubject": {
    "subjectId": "customer-12345",
    "subjectType": "individual"
  },
  "scope": {
    "dataElements": ["transaction_date", "amount", "merchant_category"],
    "actions": ["READ", "QUERY"],
    "duration": { "durationDays": 90 },
    "readOnly": true
  },
  "signals": {
    "signal1": "7372",  // Merchant code (professional services)
    "signal2": "SALA"   // ISO purpose (salary/payroll)
  }
}
```

Response:
```json
{
  "decision": "approved",
  "signalQuality": "rich",
  "utpCode": "UTP-FIN-541219-7372-SALA-READ",
  "overallConfidence": 94,
  "authorizedActions": ["READ", "QUERY"],
  "enforcement": {
    "enforceStrictScope": true,
    "maskSensitiveFields": false
  },
  "utAuthRecord": {
    // Full audit trail with all stage findings
  }
}
```

---

## Architecture

### The 9-Stage Waterfall

Every authorization request flows through 9 independent evaluators:

```
Stage 0: B.A.S.E.      → User-defined rules
Stage 1: S1E           → Primary signal (merchant code, service type, etc.)
Stage 2: S2E           → Secondary signal (ISO purpose code, etc.)
Stage 3: ICE           → Industry context (NAICS, facility type, etc.)
Stage 4: MDE           → Metadata enrichment (entity resolution)
Stage 5: HPE           → Historical patterns (prior decisions, anomaly detection)
Stage 6: CA            → Confidence aggregation (weighted score)
Stage 7: DR            → Deterministic resolution (final action code)
Stage 8: FBH           → Fallback handler (if stages 1-7 insufficient)
```

Each stage:
- Operates independently (can be swapped out)
- Returns a finding + confidence score
- Passes downstream for conflict resolution
- Is logged in the final audit trail

### Signal Quality Tiers

| Tier | Definition | Confidence | Approval | Use Case |
|------|-----------|-----------|----------|----------|
| **RULED** | User rule matched (Stage 0) | 100% | Auto-approved | "I explicitly allow Adobe → Software" |
| **RICH** | Primary + secondary signals | 90-100% | Auto-approved | Full ACH code + ISO purpose + MCC |
| **STANDARD** | One rich signal | 70-85% | Auto-approved (masked) | SEC code present, ISO missing |
| **DEGRADED** | Context + fallback only | 50-70% | Requires escalation | Limited signal, conservative access |
| **MINIMUM** | All signals weak/absent | 30-50% | Human escalation | Unknown agent, no signals |

---

## Domain Profiles

UTP supports multiple regulated domains. Each domain defines:
- **Context Axis** — Industry classification (NAICS, facility type, etc.)
- **Signal-1 & Signal-2** — Primary and secondary authorization signals
- **Action Codes** — What agents can do in that domain
- **Scope Boundaries** — Data limits, time limits, rate limits
- **Enforcement Rules** — Validation, masking, escalation policies
- **Fallback Actions** — Default behavior when uncertain

### Proposed Profiles (in progress)

- **FIN** (Financial Services)
- **HLT** (Healthcare)
- **LGL** (Legal) 
- **TRV** (Travel)
- **INS** (Insurance)
- **RET** (Retail / e-Commerce)
- **EMP** (Employment / HR)
- **TAX** (Tax / Compliance)
- **EDU** (Education)

---

## Implementations

### Reference Implementation

**UTP Agent Authorization Gateway** — Production-ready REST API for evaluating agent authorization requests.

- Language: Node.js + TypeScript
- Status: Production (v1.0.0)
- Repo: [utp-gateway](https://github.com/utp-foundation/utp-gateway)
- Deploy: Docker, Kubernetes, AWS Lambda

### Known Implementations

- **iCOA Pro** (Financial) — Production system using UTP in accounting classification
- [Your implementation here? Open a PR!]

---

## Use Cases

### Financial Services

Bank needs to authorize a fintech agent requesting transaction history:

```
Agent: "I need 90 days of checking account transactions"
Bank: "For what purpose?"
Agent: "Credit scoring"
Bank: Evaluates → UTP-FIN-...-READ approved @ 94% confidence
Bank: Agent now has READ-only access for 90 days
Agent: [Uses data]
Bank: [Audit log shows exactly what agent did]
```

### Healthcare

Care coordinator bot requesting patient labs:

```
Agent: "I need lab results for patient-98765"
Clinic: "On whose authorization?"
Agent: "Nurse Alice, for treatment coordination"
Clinic: Evaluates → UTP-HLT-...-READ approved @ 78% confidence
Clinic: Agent now has READ access to labs only, for 30 days
Agent: [Uses data]
Clinic: [Audit log satisfies HIPAA requirements]
```

### Legal

Discovery agent requesting attorney-client privileged documents:

```
Agent: "I need all documents marked 'privileged' for case-2026-001"
Firm: "Is this from an authorized opposing counsel agent?"
Agent: "No, I'm internal"
Firm: Evaluates → UTP-LGL-...-DISC approved @ 85% confidence
Firm: Agent has access to privileged docs for litigation
Agent: [Uses data]
Firm: [Audit log proves access was authorized]
```

---

## Standards Alignment

UTP can be integrated with or referenced by:

- **FDX** (Financial Data Exchange) — Agentic AI & Open Finance standards
- **HIPAA** — Healthcare access audit and data minimization
- **GDPR** — Purpose limitation and data subject rights
- **ABA Model Rules** — Attorney-client privilege and discovery
- **FIDO** — Agent identity and verification
- **ISO 27001** — Information security management
- **SOC 2** — Audit logging and access controls

---

## Contributing

We welcome contributions:

1. **Domain Profiles** — Propose or refine profiles for new industries
2. **Action Code Dictionaries** — Help define action codes for your domain
3. **Implementations** — Build UTP in your language/platform
4. **Feedback** — Report issues, suggest improvements

See [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines.

---

## License

This project is licensed under the **Apache License 2.0**. You are free to:
- Use UTP in commercial products
- Modify UTP for your needs
- Distribute UTP (with attribution)

---

## Roadmap

**Q3 2026 (Now)**
- ✅ Specification v1.0 published
- ✅ Financial domain profile finalized
- ✅ Gateway reference implementation in production
- ✅ Input to FDX AI Task Force

**Q4 2026**
- Healthcare & legal domain profiles published
- Agent registry MVP launched
- Multi-tenant policy engine
- Integration with major data aggregators

**Q1 2027**
- Travel & insurance domain profiles
- Broader standards alignment (HIPAA, GDPR, ABA)
- Community implementations published

---

## FAQ

### Is UTP a replacement for OAuth?

No. OAuth handles *authentication* ("Who are you?"). UTP handles *authorization* ("What are you allowed to do?"). They work together.

### Can I build on UTP without open-sourcing my code?

Yes. UTP is Apache 2.0 licensed. You can build proprietary implementations. We just ask that you reference the open standard.

### How does UTP prevent an agent from exceeding its scope?

UTP doesn't prevent it; it *records it*. If an agent exceeds its authorized scope, the audit trail proves the violation. Enforcement (blocking, escalation, etc.) is the responsibility of the data provider (bank, healthcare system, etc.).

### Is UTP ready for production?

Yes. iCOA's financial implementation has been production-tested with 45M+ transactions. Deploy the reference Gateway and adapt for your domain.

### Can UTP work with open-source or untrusted agents?

Yes. UTP evaluates requests from any agent and determines appropriate authorization level. Unknown agents typically receive "degraded" signal quality and may require human escalation.

### What if I want to use UTP but build my own stages?

Excellent. The waterfall architecture is designed for this. You can replace S1E with your own signal evaluator, or add new stages entirely. Contribute your stage back to the community!

---

## Contact & Community

- **GitHub Discussions** — Ask questions, share ideas: [utp-foundation/discussions](https://github.com/utp-foundation)
- **Email** — hello@utp.io
- **Standards Bodies** — input@financialdataexchange.org (FDX)
- **Website** — https://utp.io
- **Blog** — https://blog.utp.io

---

## Citation

If you reference UTP in academic work, standards, or publications:

```
Heem McMillan (iCOA Labs). "Universal Transfer Protocol (UTP): 
Bounded Agent Authorization for Open Finance and Regulated Industries." 
Specification v1.0, July 2026. https://utp.io
```

---

## Acknowledgments

UTP was conceived and first implemented by iCOA Labs in collaboration with the Financial Data Exchange (FDX). Special thanks to:

- FDX Community (700+ participants in July 2026 webinar)
- Panelists: FIDO Alliance, NIST, Stripe, JPMorgan Chase, Mastercard, Prove, Skyfire
- Early adopters and testers
- Open-source community feedback

---

**Status: Actively Maintained | Last Updated: July 2026**

