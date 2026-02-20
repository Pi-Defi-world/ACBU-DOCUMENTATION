# API Routes and Smart Contracts Reference

Single reference for all ACBU API routes and on-chain protocols. Base URL uses the API version (e.g. `https://api.acbu.io/v1`). All segment and core routes require an API key unless noted.

---

## 1. Structure effectiveness

The current layout is **smart-contract-first** and **segment-based**:

- **On-chain first:** Business logic (mint, burn, savings, lending, escrow) lives in Soroban contracts; the backend calls them and reacts to events. No duplicate source of truth off-chain.
- **Naming:** Contracts use `acbu_<domain>_<capability>` (e.g. `acbu_minting`, `acbu_savings_vault`). Backend services match with camelCase (e.g. `acbuMinting.service.ts`, `acbuSavingsVaultService`).
- **Segments:** Routes are grouped by use case (p2p, sme, international, salary, enterprise, savings, lending, gateway, bills). Each segment is protected by `requireSegmentScope`, so API keys must have the right permissions (e.g. `p2p:read`, `p2p:write`).
- **Access:** User/organization model supports `actorType` (retail, sme, enterprise, merchant) and `tier` (free, verified, sme, enterprise). ApiKey can be user- or organization-scoped with a `permissions` array for segment scopes.

**Strengths:** Clear separation of protocols vs backend; one service per contract; event queues per contract; segment routes make product boundaries explicit. **To improve:** Complete stub controllers (salary, enterprise, bills) with real logic.

**Audience:** Everyday users use **retail** paths (`/retail/mint/deposit`, `/retail/burn/acbu`) or core `/mint/deposit`, `/burn/acbu`. Businesses use **business** paths with sme/enterprise scope. Governments use **government** paths with government scope. Limits and fees vary by audience (see [Limits and Tiers](../OPERATIONS/LIMITS_AND_TIERS.MD)).

---

## 2. API routes (all)

Prefix: `/{apiVersion}` (e.g. `/v1`). Auth: API key in `Authorization: Bearer <key>` or `X-Api-Key`, except where noted.

### 2.1 Health (no auth)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/health` | Liveness |
| GET | `/health/metrics` | Reserve ratio and health |

### 2.2 Core (API key; no segment scope)

| Method | Path | Description |
|--------|------|-------------|
| POST | `/auth/signin` | Sign in (rate-limited; no API key) |
| POST | `/auth/signin/verify-2fa` | 2FA verification |
| POST | `/auth/signout` | Sign out (requires API key) |
| GET | `/reserves` | Reserve status and composition |
| POST | `/reserves/track` | Trigger reserve tracking |
| GET | `/rates` | ACBU exchange rates |
| GET | `/rates/quote` | Quote for amount and optional currency |
| POST | `/mint/usdc` | USDC deposit: we convert USDC→XLM in backend, then mint (API key required) |
| POST | `/mint/deposit` | Deposit in basket currency only (NGN, KES, …); rejects USDC/USDT |
| POST | `/burn/acbu` | Burn ACBU for fiat |
| POST | `/onramp/register` | XLM deposit: register XLM; backend job mints ACBU to user (API key required) |
| POST | `/transfers` | Create P2P transfer |
| GET | `/transfers` | List transfers |
| GET | `/transfers/:id` | Get transfer by id |
| GET | `/transactions/:id` | Get transaction by id |
| GET | `/recipient` | Resolve recipient (e.g. alias → Stellar) |
| GET | `/users/me` | Current user profile |
| PATCH | `/users/me` | Update profile |
| GET/POST/DELETE | `/users/me/contacts` | Contacts |
| GET/POST/DELETE | `/users/me/guardians` | Guardians |
| POST | `/users/me/wallet/confirm` | Confirm wallet |
| GET | `/users/me/receive`, `/users/me/receive/qrcode` | Receive details / QR |
| POST | `/kyc/applications` | Submit KYC application |
| GET | `/kyc/applications`, `/kyc/applications/:id` | List / get KYC application |
| GET | `/kyc/applications/upload-urls` | Document upload URLs |
| PATCH | `/kyc/applications/:id/documents` | Patch documents |
| POST | `/kyc/validator/register` | Register as KYC validator |
| GET | `/kyc/validator/tasks`, `/kyc/validator/me` | Validator tasks / me |
| POST | `/kyc/validator/tasks/:id` | Submit validation result |
| POST | `/recovery/unlock` | Account recovery (no API key) |

### 2.2a Audience-specific mint/burn and deposit rule

- **Deposit rule (basket):** Only **basket (pool) currencies** may be deposited into the pool (NGN, KES, RWF, ZAR, GHS, EGP, MAD, TZS, UGX, XOF). Use **POST /mint/deposit** with `currency` and `amount` (basket only).
- **USDC and XLM deposits:** Users can deposit **USDC** via **POST /mint/usdc** (usdc_amount, wallet_address) or **XLM** via **POST /onramp/register** (stellar_address, xlm_amount). We run USDC→XLM conversion and/or ACBU/XLM pools and swap services in the backend; these run independently. The mint is approved once we have successfully converted USDC to XLM (for USDC deposits). Users do not wait for LPs or P2P markets.
- **Audience paths:** Mint and burn are also exposed per audience with shared behaviour and per-audience limits:

| Audience   | Scope / gate        | Mint (deposit)              | Burn                    |
|-----------|----------------------|-----------------------------|--------------------------|
| Retail    | API key (no segment) | POST `/retail/mint/deposit` | POST `/retail/burn/acbu` |
| Business  | sme:write or enterprise:write | POST `/business/mint/deposit` | POST `/business/burn/acbu` |
| Government| government:read/write| POST `/government/mint/deposit` | POST `/government/burn/acbu` |

- **POST /retail/mint/usdc**, **POST /business/mint/usdc**, **POST /government/mint/usdc** accept USDC deposit (same as core **POST /mint/usdc**); we convert in backend and mint.
- Government-only: GET `/government/treasury`, GET `/government/statements`.

### 2.3 Segment routes (API key + segment scope)

Each segment requires one of the listed scopes (e.g. `p2p:read` or `p2p:write`).

| Segment | Scope(s) | Method | Path | Description |
|---------|----------|--------|------|-------------|
| **P2P** | p2p:read, p2p:write | POST | `/p2p/send` | Send P2P (same as POST /transfers) |
| | | GET | `/p2p/history` | Transfer history |
| | | GET | `/p2p/:id` | Transfer by id |
| **SME** | sme:read, sme:write | POST | `/sme/transfers` | Create transfer |
| | | GET | `/sme/transfers` | List transfers |
| | | GET | `/sme/transfers/:id` | Transfer by id |
| | | GET | `/sme/statements` | Statements (same as transfers list) |
| **International** | international:read, international:write | GET | `/international/quote` | Rates quote |
| | | POST | `/international/mint/usdc` | Mint from USDC |
| | | POST | `/international/burn/acbu` | Burn for fiat |
| **Salary** | salary:read, salary:write | POST | `/salary/disburse` | Disburse salary batch (stub) |
| | | POST | `/salary/schedule` | Schedule recurring (stub) |
| | | GET | `/salary/batches` | List batches |
| **Enterprise** | enterprise:read, enterprise:write | POST | `/enterprise/bulk-transfer` | Bulk transfer (stub) |
| | | GET | `/enterprise/treasury` | Treasury view (stub) |
| **Savings** | savings:read, savings:write | POST | `/savings/deposit` | Deposit into vault |
| | | POST | `/savings/withdraw` | Withdraw from vault (blocked on monthly lock dates) |
| | | GET | `/savings/positions` | Positions (balance by user/term) |
| | | GET | `/savings/next-withdrawal-date` | Next date withdrawals are allowed |
| **Investment** | investment:read, investment:write | POST | `/investment/withdraw/request` | Request withdrawal (24h + notification; business: calendar or 1% forced) |
| | | GET | `/investment/withdraw/requests` | List withdrawal requests |
| **Government** | government:read, government:write | GET | `/government/treasury` | Treasury view |
| | | GET | `/government/statements` | Statements |
| **Lending** | lending:read, lending:write | POST | `/lending/deposit` | Deposit to pool |
| | | POST | `/lending/withdraw` | Withdraw from pool |
| | | GET | `/lending/balance` | Lender balance |
| **Gateway** | gateway:read, gateway:write | POST | `/gateway/charges` | Create escrow charge |
| | | POST | `/gateway/confirm` | Release or refund escrow |
| **Bills** | bills:read, bills:write | GET | `/bills/catalog` | Billers catalog (stub) |
| | | POST | `/bills/pay` | Pay bill (stub) |

### 2.4 Webhooks (inbound)

| Method | Path | Description |
|--------|------|-------------|
| POST | `/webhooks/flutterwave` | Flutterwave webhook (signature verified; raw body) |

---

## 3. Segment scopes (API key permissions)

API key `permissions` is a JSON array of strings. For segment routes, at least one of the segment scopes must be present.

| Scope | Segment |
|-------|---------|
| p2p:read, p2p:write | P2P / mobile money |
| sme:read, sme:write | SME |
| international:read, international:write | International (mint/burn/quote) |
| salary:read, salary:write | Salary / payroll |
| enterprise:read, enterprise:write | Enterprise |
| savings:read, savings:write | Savings |
| lending:read, lending:write | Lending |
| bills:read, bills:write | Bills |
| gateway:read, gateway:write | Merchant gateway |
| government:read, government:write | Government |
| investment:read, investment:write | Investment withdrawal |

---

## 4. Smart contracts (on-chain)

All contracts are Soroban (Stellar); crate names and backend services use the naming below.

| # | Crate / folder | Purpose | Backend service | Env var |
|---|----------------|---------|------------------|---------|
| 1 | acbu_minting | Mint ACBU from USDC/fiat | acbuMinting.service.ts / acbuMintingService | CONTRACT_MINTING |
| 2 | acbu_burning | Burn ACBU, trigger redemption | acbuBurning.service.ts / acbuBurningService | CONTRACT_BURNING |
| 3 | acbu_oracle | Aggregated rates, ACBU/USD | acbuOracle.service.ts / acbuOracleService | CONTRACT_ORACLE |
| 4 | acbu_reserve_tracker | Reserve amounts per currency | acbuReserveTracker.service.ts / acbuReserveTrackerService | CONTRACT_RESERVE_TRACKER |
| 5 | acbu_savings_vault | Lock ACBU, terms, yield | acbuSavingsVault.service.ts / acbuSavingsVaultService | CONTRACT_SAVINGS_VAULT |
| 6 | acbu_lending_pool | Deposits, withdraws (lending) | acbuLendingPool.service.ts / acbuLendingPoolService | CONTRACT_LENDING_POOL |
| 7 | acbu_escrow | Hold ACBU; release/refund | acbuEscrow.service.ts / acbuEscrowService | CONTRACT_ESCROW |

Optional (Stage 3+): **acbu_rebalancing** for on-chain weight proposals/execution.

**Do we need smart contracts for USDC/XLM or ACBU/XLM pools?** ACBU mint, burn, and savings flows use the smart contracts above. USDC/XLM conversion and ACBU/XLM liquidity (or P2P) can be **backend-only** (custody + swap APIs or our own P2P); no new smart contracts are required. On-chain AMM (e.g. Stellar AMM) can be added later if we want on-chain liquidity.

### 4.1 Contract events and backend queues

| Contract | Event listener job | Queue |
|----------|---------------------|-------|
| acbu_minting | acbu_minting_event_listener.ts | (enqueues USDC_CONVERSION) |
| acbu_burning | acbu_burning_event_listener.ts | (enqueues WITHDRAWAL_PROCESSING) |
| acbu_savings_vault | acbu_savings_vault_event_listener.ts | ACBU_SAVINGS_VAULT_EVENTS |
| acbu_lending_pool | acbu_lending_pool_event_listener.ts | ACBU_LENDING_POOL_EVENTS |
| acbu_escrow | acbu_escrow_event_listener.ts | ACBU_ESCROW_EVENTS |

---

## 5. Related docs

- [API Specification](MD/TECHNICAL/API_SPECIFICATION.MD) – Request/response shapes, errors
- [Smart Contract Spec](SMART_CONTRACT_SPEC.MD) – Contract interfaces and behaviour
- [Integration (contracts)](../acbu-smart-contract/INTEGRATION.md) – Backend integration with contracts
- [Project Structure](PROJECT_STRUCTURE.MD) – Repo layout and responsibilities
