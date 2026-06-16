# ACBU Security Guide

**Version:** 1.0  
**Last Updated:** June 16, 2026  
**Status:** Active

---

## Overview

ACBU handles real financial assets across 10 African jurisdictions on the Stellar blockchain. Security is critical at every layer: smart contracts, backend API, reserve custody, KYC/identity, and operational processes. This guide covers security architecture, secure development practices, and the process for reporting vulnerabilities.

---

## 1. Security Architecture

### 1.1 Smart Contract Security

**Multisig Administration (3-of-5)**  
All privileged contract operations require 3 of 5 administrator signatures:
- Pause / unpause contracts
- Update fee rates
- Add or remove oracle validators
- Emergency rebalancing
- Weight adjustment execution

**Circuit Breakers**  
Contracts automatically halt operations under adverse conditions:

| Trigger | Action |
|--------|--------|
| Reserves < 102% collateralization | Pause new minting |
| Single currency reserve < 10% target | Pause single-currency withdrawals |
| Oracle price deviation > 10% | Pause minting and burning |
| Suspicious activity detected | Full pause, alert compliance |

**Time Locks**  
Delay windows prevent rushed or malicious changes:
- Basket weight adjustments: 30-day implementation period
- Fee changes: 7-day notice period
- Emergency admin functions: 24-hour time lock

**Rate Limits (on-chain)**  
- Maximum mint per transaction
- Maximum burn per transaction
- Daily withdrawal limits per currency
- Oracle update frequency: minimum 6 hours between updates

### 1.2 Oracle Security

The oracle (`acbu_oracle`) aggregates rates from 5 independent validators. A minimum of 3 validator signatures is required to publish a rate update. Rate sources (central bank, fintech partner, forex market) are aggregated via median to resist manipulation.

Operators must monitor for:
- Stale rates (last update > 12 hours)
- Validator key compromise
- Coordinated manipulation by colluding validators

### 1.3 API Security

**Authentication**  
All API requests require a Bearer API key:
```
Authorization: Bearer YOUR_API_KEY
```

**Segment Scopes**  
API keys carry granular permissions. Access to a segment route requires the matching scope:

| Segment | Required scope |
|---------|----------------|
| P2P | `p2p:read`, `p2p:write` |
| Savings | `savings:read`, `savings:write` |
| Lending | `lending:read`, `lending:write` |
| Gateway | `gateway:read`, `gateway:write` |
| (and so on per segment) | |

The `requireSegmentScope` middleware enforces this on every segment route.

**Rate Limiting**

| Tier | Limit |
|------|-------|
| Standard | 100 req/min |
| Premium | 1,000 req/min |
| Enterprise | 10,000 req/min |

**Webhook Signature Verification**  
All inbound webhooks (e.g., from Flutterwave) must be verified using HMAC-SHA256 before processing:

```javascript
const crypto = require('crypto');

function verifyWebhook(payload, signature, secret) {
  const hash = crypto
    .createHmac('sha256', secret)
    .update(JSON.stringify(payload))
    .digest('hex');
  return hash === signature;
}
```

Never process a webhook payload without verifying its signature.

### 1.4 KYC and Identity Security

- Document blobs are stored in S3/MinIO only; the database stores `storageRef` keys and metadata — never raw PII blobs.
- Human validators see only `machineRedactedPayload`. Raw documents and `machineExtractedPayload` are never exposed to validators.
- KYC fee verification is enforced server-side before any application is accepted (via completed mint transaction or Stellar payment to the fee collector address).
- Validator accounts require prior KYC approval (`kycStatus === 'verified'`) and same-country matching before they can be assigned tasks.

### 1.5 Reserve Custody Security

Reserves are distributed across multiple fintech partners (Ecobank, MTN MoMo, Flutterwave, OPay, UBA) to eliminate single-point-of-failure risk. The system uses view-only API integrations for reserve proof — no write access to custody accounts from the ACBU backend.

Proof-of-reserves is maintained via:
- Real-time API balance checks
- Weekly on-chain Merkle tree proofs
- Monthly third-party attestations (Big 4 auditors)
- Public reserve dashboard

---

## 2. Secure Development Practices

### 2.1 Smart Contracts (Stellar/Soroban Rust)

- Follow the canonical naming convention (`acbu_<domain>_<capability>`) to avoid contract address confusion.
- All privileged functions must check `admin` address or require multisig before execution.
- Use checked arithmetic — overflow/underflow in fee or rate calculations can lead to incorrect minting.
- Emit events for every state-changing operation (mint, burn, rate update, weight change, pause).
- Never store private keys or secrets in contract state.
- Write unit tests for boundary conditions: zero amounts, max amounts, paused state, invalid addresses.

### 2.2 Backend API (Node.js / TypeScript)

- Use parameterized queries (via Prisma) for all database access. Never interpolate user input into SQL.
- Validate and sanitize all request inputs before processing or forwarding to contracts.
- Reject requests with unexpected or undeclared fields.
- Do not log full API keys, private keys, or sensitive PII. Log key prefixes/IDs only.
- Environment secrets (`DATABASE_URL`, `STELLAR_SECRET_KEY`, `OPENAI_API_KEY`, object store credentials) must be loaded from environment variables — never hardcoded.
- Rotate API keys and Stellar operation keys regularly. Revoke compromised keys immediately.
- The `STELLAR_SECRET_KEY` used for backend operations must be a hot wallet with minimal balance. Fund it only for operational needs.

### 2.3 Dependency Management

- Pin exact dependency versions in `package.json` and lockfiles.
- Run `npm audit` (or equivalent) regularly and before each release.
- Review new dependencies for known vulnerabilities or suspicious provenance before adding them.

### 2.4 Infrastructure

- Object storage (S3/MinIO) buckets holding KYC documents must not be publicly accessible.
- Use bucket lifecycle policies to enforce document retention limits (e.g., 7 years per regulatory requirement, then automatic deletion).
- Restrict RabbitMQ access to internal services only; do not expose queue ports publicly.
- Use TLS for all inter-service communication and all external API calls.
- Deploy the backend with the principle of least privilege — each service account has only the permissions it needs.

### 2.5 Key Management

| Key type | Storage | Rotation |
|----------|---------|----------|
| Admin multisig keys (contract) | Hardware security modules (HSM) | Annual or after any suspected compromise |
| Oracle validator keys | HSM or isolated key vaults | Annual |
| Backend Stellar operation key | Encrypted secrets manager | Quarterly |
| API keys (user/integration) | Hashed in database | On demand / on suspected compromise |
| Webhook secrets | Encrypted secrets manager | Quarterly |

---

## 3. AML / Transaction Monitoring

The backend's automated monitoring system flags the following patterns for compliance review:

- Mint followed immediately by single-currency redemption (within 72-hour lockup)
- Multiple accounts sharing the same beneficial owner performing coordinated currency cycling
- Rapid cycling through multiple currencies that correlates with forex market movements
- Deposit or withdrawal volumes inconsistent with the user's KYC tier

**Actions available:**
- Require basket-only redemption
- Impose temporary withdrawal limits
- Escalate for enhanced due diligence
- File suspicious activity report (SAR) with the relevant jurisdiction

---

## 4. Incident Response

### Severity Levels

| Level | Description | Response time |
|-------|-------------|---------------|
| **Critical** | Active exploit, funds at risk, contract paused | Immediate (< 1 hour) |
| **High** | Oracle manipulation, reserve undercollateralization | < 4 hours |
| **Medium** | API key compromise, unauthorized data access | < 24 hours |
| **Low** | Misconfiguration, minor vulnerability, informational | < 7 days |

### Response Steps

1. **Contain** — Use the contract `pause` function (admin multisig) to halt minting/burning if funds are at risk.
2. **Assess** — Determine affected users, assets, and data. Preserve logs.
3. **Notify** — Alert internal security and compliance teams. For Critical/High incidents, notify affected users and relevant regulators per jurisdiction requirements.
4. **Remediate** — Deploy a fix. For contract bugs, a new contract deployment with migration may be required.
5. **Post-mortem** — Document root cause, timeline, impact, and preventive measures. Publish a transparency disclosure where appropriate.

---

## 5. Vulnerability Reporting

ACBU operates a responsible disclosure program. If you discover a security vulnerability, please report it privately before public disclosure.

### How to Report

**Email:** security@acbu.io  
**PGP Key:** Available at https://acbu.io/.well-known/security.txt  
**Response SLA:** Initial acknowledgment within 48 hours; status update within 7 days.

### What to Include

- Description of the vulnerability and affected component (contract name, API endpoint, etc.)
- Steps to reproduce or proof-of-concept
- Potential impact (funds at risk, data exposure, denial of service, etc.)
- Your contact information (optional for anonymous reports)

### Scope

**In scope:**
- Smart contracts (`acbu_minting`, `acbu_burning`, `acbu_oracle`, `acbu_reserve_tracker`, `acbu_savings_vault`, `acbu_lending_pool`, `acbu_escrow`, `acbu_rebalancing`)
- Backend API (`https://api.acbu.io/v1`)
- KYC layer and document storage
- Reserve verification and oracle systems
- Authentication and API key management

**Out of scope:**
- Third-party fintech partner platforms (Flutterwave, Paystack, MTN MoMo, etc.) — report to them directly
- Social engineering attacks against ACBU staff
- Denial-of-service attacks against infrastructure
- Theoretical vulnerabilities without a demonstrated impact

### Responsible Disclosure Policy

- Do not access, modify, or exfiltrate user data or funds beyond what is necessary to demonstrate the vulnerability.
- Do not disclose the vulnerability publicly until ACBU has released a fix or acknowledged that no fix is needed.
- ACBU will not pursue legal action against researchers who act in good faith under these terms.
- Researchers who responsibly disclose valid Critical or High severity issues may be acknowledged publicly (with permission) and may be eligible for a reward at ACBU's discretion.

---

## 6. Compliance and Regulatory Security Requirements

| Requirement | Implementation |
|-------------|---------------|
| KYC/AML | Two-step KYC (machine + human validator pool); transaction monitoring |
| Data minimization | Validators see only redacted payloads; DB stores refs not blobs |
| Document retention | S3 lifecycle policies per jurisdiction (minimum 7 years) |
| Reserve auditability | Public dashboard + monthly Big 4 attestation + on-chain Merkle proofs |
| Regulatory reporting | Transaction volume and reserve reports to regulators per country |
| Sanctions screening | Integrated into KYC onboarding and ongoing transaction monitoring |

---

## Related Documents

- [Architecture](TECHNICAL/ARCHITECTURE.MD)
- [KYC Layer](TECHNICAL/KYC_LAYER.md)
- [Smart Contract Spec](PROJECT/SMART_CONTRACT_SPEC.MD)
- [Risk Management](OPERATIONS/RISK_MANAGEMENT.MD)
- [Regulatory Framework](OPERATIONS/REGULATORY.MD)
- [Reserve Management](TECHNICAL/RESERVE_MANAGEMENT.MD)
- [API Specification](TECHNICAL/API_SPECIFICATION.MD)
