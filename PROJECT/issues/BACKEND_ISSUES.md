# Backend — Known Issues

This document is the **single, durable long-form reference** for known issues in the ACBU backend (`acbu-backend`, Node.js/Express + Prisma/PostgreSQL + MongoDB + RabbitMQ + Stellar SDK), covering all routes, services, jobs, middleware, utils, config, security, observability, compliance, and ops.

> **How to use**
>
> - Each issue is structured as: **severity · area · evidence · impact · fix direction · acceptance check**.
> - Items are ordered by severity (Critical → Low), then by ID for stable cross-referencing.
> - Provenance: this file consolidates the legacy 56-item annotated list previously at `PROJECT/issues/BACKEND_ISSUES.md` with the canonical 75-item triage-ready list at `issues/backend.md`. The canonical list at `issues/backend.md` remains the **live working backlog**; this file is the durable version-of-record.
> - Legacy items whose content maps to a canonical `B-###` ID are tagged **(moved)**; legacy items that go beyond the canonical list (newer findings) are tagged **(new)** and retained verbatim because they pre-date the canonical rewrite.
> - When an item is fixed, the corresponding entry is removed from the working lists (`issues/backend.md`) and the resolution row flips to ✅ Fixed in Section 8 with a merged-PR link.
> - **Severity legend:** 🔴 Critical · 🟠 High · 🟡 Medium · 🟢 Low

---

## 1. Summary Table

| ID | Severity | Area | Title (canonical) |
|----|----------|------|---------|
| B-001 | 🟠 High | backend/minting | Mint basket deposit limits use wrong USD proxy |
| B-002 | 🟠 High | backend/money | Floating `Number()` on money paths |
| B-003 | 🟠 High | backend/jobs | USDC convert job infinite requeue on failure |
| B-004 | 🟠 High | backend/webhooks | Webhook delivery retry semantics |
| B-005 | 🔴 Critical | backend/webhooks | Webhook signature verification fail-open risk |
| B-006 | 🔴 Critical | backend/auth | Recovery unlock as single-factor account recovery |
| B-007 | 🟠 High | backend/minting | Client-supplied mint recipient vs user wallet |
| B-008 | 🟠 High | backend/transfers | Transfer service Decimal typing |
| B-009 | 🟠 High | backend/fees | Burn fee policy threshold logic |
| B-010 | 🟠 High | backend/reserves | ReserveTracker total supply from DB not chain |
| B-011 | 🟠 High | backend/limits | Org-scoped transaction limits with null userId |
| B-012 | 🟡 Medium | backend/savings | Savings controller input validation |
| B-013 | 🟠 High | backend/auth | Placeholder / invalid Stellar address on signup |
| B-014 | 🟡 Medium | backend/wallet | Wallet activation funding asset choice |
| B-015 | 🟡 Medium | backend/stellar | Hardcoded Soroban inclusion fees |
| B-016 | 🟠 High | backend/stellar | Stellar event listener scope and error handling |
| B-017 | 🟠 High | backend/kyc | KYC machine layer is placeholder |
| B-018 | 🟡 Medium | backend/government | Government treasury aggregation stub |
| B-019 | 🟡 Medium | backend/salary | Salary APIs not implemented |
| B-020 | 🟡 Medium | backend/enterprise | Enterprise bulk transfer not implemented |
| B-021 | 🟡 Medium | backend/enterprise | Enterprise treasury view stub |
| B-022 | 🟡 Medium | backend/bills | Bills catalog and pay not implemented |
| B-023 | 🟡 Medium | backend/investment | Investment deployable allocation ignores deployed |
| B-024 | 🟡 Medium | backend/jobs | Investment withdrawal notifications for org requests |
| B-025 | 🟡 Medium | backend/investment | Yield accounting stub |
| B-026 | 🟠 High | backend/audit | Audit write failures only logged |
| B-027 | 🟡 Medium | backend/cache | Cache `deletePattern` ReDoS risk |
| B-028 | 🟠 High | backend/ratelimit | API key rate limiter non-atomic increment |
| B-029 | 🟡 Medium | backend/ratelimit | Rate limiter Mongo outage fail-open |
| B-030 | 🟢 Low | backend/ratelimit | Standard IP rate limit message mismatch |
| B-031 | 🟡 Medium | backend/ops | Deep health and metrics routes are public |
| B-032 | 🟢 Low | backend/ops | Shallow `/health` does not prove readiness |
| B-033 | 🟡 Medium | backend/docs | Swagger UI publicly mounted |
| B-034 | 🟡 Medium | backend/docs | OpenAPI drift vs implementation |
| B-035 | 🟢 Low | backend/devx | README still references npm only |
| B-036 | 🟡 Medium | backend/devx | `postinstall` runs prisma generate + tsc |
| B-037 | 🟡 Medium | backend/qa | Jest `passWithNoTests` weak gate |
| B-038 | 🟡 Medium | backend/api | No transfer pagination cursors |
| B-039 | 🟡 Medium | backend/api | Investment withdrawal list lacks pagination |
| B-040 | 🟢 Low | backend/api | Government list `limit` NaN hazard |
| B-041 | 🟢 Low | backend/types | Prisma `onRampSwap` typed via `as any` casts |
| B-042 | 🟡 Medium | backend/webhooks | Webhook raw body JSON fallback empty object |
| B-043 | 🟢 Low | backend/notifications | Notification service placeholder sender |
| B-044 | 🟡 Medium | backend/ops | Basket weight computation job missing |
| B-045 | 🟡 Medium | backend/integrations | Fintech multi-provider per country |
| B-046 | 🟢 Low | backend/config | Limits config hardcoded |
| B-047 | 🟡 Medium | backend/authz | Segment guard `requireMinTier` unused |
| B-048 | 🟢 Low | backend/types | Permissions typed as loose strings |
| B-049 | 🟠 High | backend/security | CORS policy review for credentialed browsers |
| B-050 | 🟡 Medium | backend/security | Helmet / security headers baseline |
| B-051 | 🟢 Low | backend/observability | Request ID propagation |
| B-052 | 🟡 Medium | backend/observability | Structured logging for money movements |
| B-053 | 🟠 High | backend/api | Idempotency-Key header support |
| B-054 | 🟢 Low | backend/observability | Distributed tracing (OpenTelemetry) |
| B-055 | 🟡 Medium | backend/ci | Database migration CI gate |
| B-056 | 🟡 Medium | backend/config | Prisma Accelerate vs direct URL confusion |
| B-057 | 🟢 Low | backend/data | Mongo index strategy for hot keys |
| B-058 | 🟡 Medium | backend/compliance | Encryption at rest for PII columns |
| B-059 | 🟡 Medium | backend/compliance | GDPR export & delete endpoints |
| B-060 | 🟠 High | backend/authz | Admin role separation |
| B-061 | 🟠 High | backend/config | Secrets in environment validation |
| B-062 | 🟡 Medium | backend/storage | S3 presigned URL abuse controls |
| B-063 | 🟡 Medium | backend/ai | OpenAI usage guardrails |
| B-064 | 🟡 Medium | backend/security | Dependency vulnerability scanning |
| B-065 | 🟢 Low | backend/auth | Time sync / clock skew handling for JWT |
| B-066 | 🟡 Medium | backend/auth | 2FA challenge token purpose binding |
| B-067 | 🟡 Medium | backend/auth | Account enumeration on sign-in |
| B-068 | 🟠 High | backend/auth | Brute-force protection on auth endpoints |
| B-069 | 🟡 Medium | backend/security | Content-Type and body size limits |
| B-070 | 🟢 Low | backend/api | Input sanitization for search endpoints |
| B-071 | 🟢 Low | backend/api | Versioning and deprecation headers |
| B-072 | 🟡 Medium | backend/api | Consistent error JSON schema |
| B-073 | 🟠 High | backend/core | Transaction status machine tests |
| B-074 | 🟡 Medium | backend/burn | Double-submit burn with same blockchain hash |
| B-075 | 🟠 High | backend/jobs | On-ramp swap race: duplicate processing |

**Totals:** 2 Critical · 21 High · 38 Medium · 14 Low · **75 total catalog items** (all IDs `B-001` through `B-075`, fully aligned with `issues/backend.md`).

---

## 2. Severity Counts

| Severity | Count |
|----------|-------|
| 🔴 Critical | **2** (B-005, B-006) |
| 🟠 High | **21** (B-001–B-004, B-007–B-011, B-013, B-016, B-017, B-026, B-028, B-049, B-053, B-060, B-061, B-068, B-073, B-075) |
| 🟡 Medium | **38** (B-012, B-014, B-015, B-018–B-025, B-027, B-029, B-031, B-033, B-034, B-036–B-039, B-042, B-044, B-045, B-047, B-050, B-052, B-055, B-056, B-058, B-059, B-062–B-064, B-066, B-067, B-069, B-072, B-074) |
| 🟢 Low | **14** (B-030, B-032, B-035, B-040, B-041, B-043, B-046, B-048, B-051, B-054, B-057, B-065, B-070, B-071) |
| **Total** | **75** |

---

## 3. Critical Issues (🔴) — Ship Blockers

### B-005 — Webhook signature verification fail-open risk
- **Area:** backend/webhooks
- **Evidence:** `acbu-backend/src/controllers/webhookController.ts` / `verifyFlutterwaveSignature` (verify pattern)
- **Impact:** If `FLUTTERWAVE_WEBHOOK_SECRET` (or equivalent secrets) are missing in production, verification is skipped and `next()` is called, allowing forged webhooks to mint credit downstream.
- **Fix direction:** Fail closed in production when secret env vars are missing; separate dev/stage mocks; refuse to register the webhook route in prod without a usable secret.
- **Acceptance check:** Production boot fails fast (or webhook handlers return 401) when signing secret is unset; staging retains the mocked-verify behavior under a feature flag.

### B-006 — Recovery unlock as single-factor account recovery
- **Area:** backend/auth
- **Evidence:** `acbu-backend/src/services/recovery/recoveryService.ts` (per prior audits)
- **Impact:** Identifier + passcode alone can issue a new API key; a leaked/stolen passcode enables full account takeover.
- **Fix direction:** Add OTP (email/SMS) and/or device proof as a second factor; rate-limit recovery endpoints; rotate all existing sessions on success; emit audit event.
- **Acceptance check:** E2E test confirms a fresh API key cannot be obtained without the configured second factor; brute-force lockout kicks in after threshold.

---

## 4. High Severity Issues (🟠)

### B-001 — Mint basket deposit limits use wrong USD proxy
- **Area:** backend/minting
- **Evidence:** `acbu-backend/src/controllers/mintController.ts` (`depositFromBasketCurrency`)
- **Impact:** Local currency amounts are treated as USD when calling `checkDepositLimits`; limits can be wildly wrong for NGN/KES/etc.
- **Fix direction:** Convert basket currency → USD using oracle/rates before limit checks; populate per-basket limit constants from config.
- **Acceptance check:** Integration tests for each basket currency enforce the correct USD-denominated limits.

### B-002 — Floating `Number()` on money paths
- **Area:** backend/money
- **Evidence:** Multiple controllers (`mintController.ts`, `burnController.ts`)
- **Impact:** `Number()` over user-supplied monetary strings loses precision versus `Decimal` and can skew fee/limit math.
- **Fix direction:** Parse monetary strings with a decimal library end-to-end; coerce to `Decimal` at every model boundary; only lossy coerce at the Soroban boundary with explicit rounding policy.
- **Acceptance check:** Golden tests for large fractional inputs and boundary fees pass.

### B-003 — USDC convert job infinite requeue on failure
- **Area:** backend/jobs
- **Evidence:** `acbu-backend/src/jobs/usdcConvertAndMintJob.ts` (`ch.nack(..., true)`)
- **Impact:** Poison messages can spin forever, duplicating side effects or stalling the queue; `OnRampSwap` rows never reach terminal state.
- **Fix direction:** Add retry caps + DLQ routing; transition `OnRampSwap.status` atomically to `failed` after N attempts with the failure reason.
- **Acceptance check:** Failed swap lands in DLQ; corresponding DB row reaches terminal state; no infinite nack loop under chaos replay.

### B-004 — Webhook delivery retry semantics
- **Area:** backend/webhooks
- **Evidence:** `acbu-backend/src/services/webhook/webhookService.ts`
- **Impact:** Partners may be hammered; events retried without idempotency can cause duplicate settlements.
- **Fix direction:** Per-delivery idempotency keys; exponential backoff; max attempts; DLQ + alerting on sustained failure.
- **Acceptance check:** Webhook consumer stops after configured failure threshold; DLQ is observable in the ops dashboard.

### B-007 — Client-supplied mint recipient vs user wallet
- **Area:** backend/minting
- **Evidence:** `acbu-backend/src/controllers/mintController.ts`, on-ramp controllers
- **Impact:** If mint / on-ramp paths still accept a body-supplied `wallet_address` without binding to `User.stellarAddress`, minted ACBU can be redirected to an unrelated address.
- **Fix direction:** Centralize `assertUserWalletAddress` on every mint / on-ramp path; deny mismatches with a structured 403.
- **Acceptance check:** Security test fails when minting to a `G…` not owned by the authenticated user.

### B-008 — Transfer service Decimal typing
- **Area:** backend/transfers
- **Evidence:** `acbu-backend/src/services/transfer/transferService.ts`
- **Impact:** String amounts into Prisma Decimal fields risk precision and type drift at runtime.
- **Fix direction:** Construct `Decimal` explicitly; validate with Zod (`string` + decimal regex) before passing to Prisma.
- **Acceptance check:** Unit tests pass for maximum-precision transfers and boundary cases.

### B-009 — Burn fee policy threshold logic
- **Area:** backend/fees
- **Evidence:** `acbu-backend/src/services/feePolicy/feePolicyService.ts`
- **Impact:** Suspected inverted/typo thresholds can charge wrong BPS versus the reserve basket state.
- **Fix direction:** Review the math against `OPERATIONS/LIMITS_AND_TIERS` and product spec; add property tests for the published tier table.
- **Acceptance check:** Fee outputs match the documented tier table for all bracket boundaries.

### B-010 — ReserveTracker total supply from DB not chain
- **Area:** backend/reserves
- **Evidence:** `acbu-backend/src/services/reserve/ReserveTracker.ts`
- **Impact:** On-chain supply and indexed DB can diverge; circuit breakers may compute against the wrong numerator.
- **Fix direction:** Periodic chain-truth reconciliation; alert on drift beyond policy threshold.
- **Acceptance check:** Dashboard surfaces DB vs on-chain delta under threshold; sustained drift alerts.

### B-011 — Org-scoped transaction limits with null userId
- **Area:** backend/limits
- **Evidence:** `acbu-backend/src/services/limits/limitsService.ts`
- **Impact:** Org keys creating `userId: null` rows may be excluded from aggregates; limits can be bypassed at the org level.
- **Fix direction:** Include `organizationId` dimension consistently in limit queries; align with the prisma `Transaction` schema.
- **Acceptance check:** Org-scope key cannot exceed org daily cap in tests.

### B-013 — Placeholder / invalid Stellar address on signup
- **Area:** backend/auth
- **Evidence:** `acbu-backend/src/services/auth/authService.ts` (`getPlaceholderStellarAddress`)
- **Impact:** Non-`G` placeholders break downstream Stellar validation and break UX.
- **Fix direction:** Defer the DB constraint until the real wallet is created, **or** use a valid funded temporary account pattern under a feature flag.
- **Acceptance check:** No user row ships in production with an invalid `stellarAddress` format.

### B-016 — Stellar event listener scope and error handling
- **Area:** backend/stellar
- **Evidence:** `acbu-backend/src/services/stellar/eventListener.ts`
- **Impact:** Broad Horizon streams + swallowed errors cause missed mint/burn indexing silently.
- **Fix direction:** Filter to known contract IDs; structured retries; DLQ routing for parse failures; metrics on event ingestion lag.
- **Acceptance check:** Injected test event reaches projection store within SLA under simulated outages.

### B-017 — KYC machine layer is placeholder
- **Area:** backend/kyc
- **Evidence:** `acbu-backend/src/services/kyc/machineLayer.ts`
- **Impact:** Automated checks / PII redaction are not production-grade; KYC fraud risk increases.
- **Fix direction:** Integrate a vision provider; PII redaction pipeline; manual review queue for low-confidence results.
- **Acceptance check:** Sample IDs pass/fail deterministically in staging with redacted logs.

### B-026 — Audit write failures only logged
- **Area:** backend/audit
- **Evidence:** `acbu-backend/src/services/audit/auditService.ts`
- **Impact:** Compliance gap if audit events disappear silently.
- **Fix direction:** Retry + outbox to Mongo/queue; sustained-failure alert; dashboard on audit lag.
- **Acceptance check:** Chaos test (simulated Mongo/queue outage) demonstrates the audit pipeline recovers and reaches eventual consistency.

### B-028 — API key rate limiter non-atomic increment
- **Area:** backend/ratelimit
- **Evidence:** `acbu-backend/src/middleware/rateLimiter.ts`
- **Impact:** Get-then-set increment pattern is not atomic; concurrent requests can both pass the limit.
- **Fix direction:** Use Mongo atomic `$inc` with a per-key document-level cap **or** Redis `INCR` with TTL.
- **Acceptance check:** Concurrent load test cannot exceed configured QPS for any single API key.

### B-049 — CORS policy review for credentialed browsers
- **Area:** backend/security
- **Evidence:** `acbu-backend/src/index.ts` / CORS config
- **Impact:** Misconfigured origins with credentials enable token theft via malicious site.
- **Fix direction:** Explicit allowlist per environment; never combine `*` with `credentials: true`.
- **Acceptance check:** Security headers test suite (OWASP baseline) passes.

### B-053 — Idempotency-Key header support
- **Area:** backend/api
- **Evidence:** POST transfer/mint/burn routes
- **Impact:** Network retries can double-charge / double-mint.
- **Fix direction:** Persist keys in `Idempotency` table; cache and return identical response bodies for repeated keys.
- **Acceptance check:** Duplicate POST with the same key returns the cached response within the configured TTL.

### B-060 — Admin role separation
- **Area:** backend/authz
- **Evidence:** Admin vs user API keys
- **Impact:** Support-staff impersonation is overpowered; single admin key compromise is catastrophic.
- **Fix direction:** Scoped admin keys + MFA + break-glass workflow; attribute admin actions separately in audit.
- **Acceptance check:** Admin actions are clearly delineated in audit; scoped keys cannot perform unscoped operations.

### B-061 — Secrets in environment validation
- **Area:** backend/config
- **Evidence:** `acbu-backend/src/config/env.ts`
- **Impact:** Late or partial validation lets half-configured prod boot.
- **Fix direction:** Strict Zod schema at startup with `fail-fast`; no `config` access before validation runs.
- **Acceptance check:** Missing `JWT_SECRET` (or any required secret) prevents the process from `app.listen`.

### B-068 — Brute-force protection on auth endpoints
- **Area:** backend/auth
- **Evidence:** `authRoutes` + rate limiter
- **Impact:** Passcodes / API keys are guessable at scale.
- **Fix direction:** Stricter limiter on `/auth/*`; explicit lockout policy; backoff.
- **Acceptance check:** Hydra-style test is blocked after configured threshold.

### B-073 — Transaction status machine tests
- **Area:** backend/core
- **Evidence:** Prisma `Transaction` model transitions
- **Impact:** Illegal transitions corrupt balance reporting.
- **Fix direction:** Explicit state machine + DB constraints; property tests covering every transition pair.
- **Acceptance check:** Invalid transition throws a domain error and is also rejected by a DB-level `CHECK` constraint.

### B-075 — On-ramp swap race: duplicate processing
- **Area:** backend/jobs
- **Evidence:** `acbu-backend/src/jobs/usdcConvertAndMintJob.ts` (`OnRampSwap.status` transitions)
- **Impact:** Two workers may pick the same `pending` swap if the status check is non-atomic.
- **Fix direction:** Use conditional update `WHERE status = 'pending'` to transition to `processing`, scoped by id; verification in concurrency tests.
- **Acceptance check:** Concurrency test confirms exactly one worker processes each swap.

---

## 5. Medium Severity Issues (🟡)

### B-012 — Savings controller input validation
- **Area:** backend/savings · **Evidence:** `acbu-backend/src/controllers/savingsController.ts`
- **Impact:** `amount` / `term_seconds` unchecked before contract calls; bad Soroban args or cryptic errors.
- **Fix direction:** Zod schemas with bounds; decimal-string amount; reject overflow before service dispatch.
- **Acceptance check:** Invalid bodies produce structured 400 errors; valid bodies pass service-layer validation.

### B-014 — Wallet activation funding asset choice
- **Area:** backend/wallet · **Evidence:** `acbu-backend/src/services/wallet/walletActivationService.ts`
- **Impact:** Product docs reference Pi vs XLM; wrong asset strands users on the wrong network.
- **Fix direction:** Align with `TESTNET_CUSTODIAL_BOOTSTRAP` / network config; expose a network-scoped feature flag; clear log of chosen asset.
- **Acceptance check:** Activation succeeds on the configured network with the documented asset.

### B-015 — Hardcoded Soroban inclusion fees
- **Area:** backend/stellar · **Evidence:** `acbu-backend/src/services/stellar/contractClient.ts` and related services
- **Impact:** Default fee `'100'` may fail under network congestion or be overpaid; confirmation timing is unpredictable.
- **Fix direction:** Fetch base fee + soroban resource fees via Horizon / RPC; configurable caps.
- **Acceptance check:** Transactions succeed under load tests with no manual fee intervention.

### B-018 — Government treasury aggregation stub
- **Area:** backend/government · **Evidence:** `acbu-backend/src/controllers/governmentController.ts`
- **Impact:** Operators lack a consolidated exposure view.
- **Fix direction:** Aggregate from Prisma + reserve segments; cache with TTL; populate per-currency breakdown.
- **Acceptance check:** Endpoint returns non-empty `byCurrency` against seeded data.

### B-019 — Salary APIs not implemented
- **Area:** backend/salary · **Evidence:** `acbu-backend/src/controllers/salaryController.ts`
- **Impact:** MVP payroll promises are unfulfilled; clients get 501/empty arrays.
- **Fix direction:** Batch transfers + schedules + idempotency keys; reuse `transferService` for the core accounting.
- **Acceptance check:** Postman collection green for happy path.

### B-020 — Enterprise bulk transfer not implemented
- **Area:** backend/enterprise · **Evidence:** `acbu-backend/src/controllers/enterpriseController.ts`
- **Impact:** Large merchants cannot upload batches.
- **Fix direction:** Chunked processing; per-row idempotency; reporting endpoint for partial failure.
- **Acceptance check:** 10k-row CSV completes within SLO with a partial-failure report.

### B-021 — Enterprise treasury view stub
- **Area:** backend/enterprise · **Evidence:** `acbu-backend/src/controllers/enterpriseController.ts` (`getTreasury`)
- **Impact:** Treasury nulls block finance workflows.
- **Fix direction:** Join transfers, reserves, FX snapshots.
- **Acceptance check:** Totals reconcile to ledger within tolerance.

### B-022 — Bills catalog and pay not implemented
- **Area:** backend/bills · **Evidence:** `acbu-backend/src/controllers/billsController.ts`
- **Impact:** Bill-pay segment is non-functional.
- **Fix direction:** Partner adapter + webhook reconciliation + refund flow.
- **Acceptance check:** Happy-path bill payment creates an auditable `Transaction`.

### B-023 — Investment deployable allocation ignores deployed
- **Area:** backend/investment · **Evidence:** `acbu-backend/src/services/investment/allocationService.ts`
- **Impact:** `deployedUsd` pinned to 0 over-allocates to strategies.
- **Fix direction:** Track deployed notional in DB; subtract from deployable; tie ledger adjustments to allocation events.
- **Acceptance check:** Allocation never exceeds policy once deployed balances are populated.

### B-024 — Investment withdrawal notifications for org requests
- **Area:** backend/jobs · **Evidence:** `acbu-backend/src/jobs/investmentWithdrawalJob.ts`
- **Impact:** Org-only requests may never notify approvers.
- **Fix direction:** Notify org admins via email/push using `organizationId`; mirror for org-only `r.organizationId` rows where `r.userId` is null.
- **Acceptance check:** Org withdrawal request lands a notification in the test inbox.

### B-025 — Yield accounting stub
- **Area:** backend/investment · **Evidence:** `acbu-backend/src/services/investment/yieldAccountingService.ts`
- **Impact:** Cannot report yield or tax basis.
- **Fix direction:** Implement accrual postings tied to vault positions; statement API endpoint.
- **Acceptance check:** Monthly statement API returns non-zero yield against seeded positions.

### B-027 — Cache `deletePattern` ReDoS risk
- **Area:** backend/cache · **Evidence:** `acbu-backend/src/utils/cache.ts`
- **Impact:** User-influenced regex patterns can hang the Node event loop.
- **Fix direction:** Escape / length-cap patterns; prefer safe glob or prefix scan instead of arbitrary regex.
- **Acceptance check:** Fuzz test cannot stall the event loop beyond budget.

### B-029 — Rate limiter Mongo outage fail-open
- **Area:** backend/ratelimit · **Evidence:** `acbu-backend/src/middleware/rateLimiter.ts` + `cacheService`
- **Impact:** When cache errors, fallback may weaken per-key limits.
- **Fix direction:** Stricter IP fallback + circuit breaker + active metrics on degraded mode.
- **Acceptance check:** Simulated Mongo outage triggers the documented degraded-but-bounded mode.

### B-031 — Deep health and metrics routes are public
- **Area:** backend/ops · **Evidence:** `acbu-backend/src/routes/index.ts` (`/health/deep`, `/health/metrics`)
- **Impact:** Dependency topology exposes to unauthenticated callers (reconnaissance).
- **Fix direction:** Protect with mTLS or admin API key; keep `/health` shallow public.
- **Acceptance check:** Public callers cannot scrape dependency failures or reserve ratios at scale.

### B-033 — Swagger UI publicly mounted
- **Area:** backend/docs · **Evidence:** `acbu-backend/src/index.ts`
- **Impact:** Attackers map the surface area without auth.
- **Fix direction:** Gate `/api-docs` behind admin auth, or disable in prod.
- **Acceptance check:** Prod deployment returns 404 (or admin-authenticated response) for `/api-docs`.

### B-034 — OpenAPI drift vs implementation
- **Area:** backend/docs · **Evidence:** `acbu-backend` swagger annotations vs routes
- **Impact:** Clients integrate against stale schemas.
- **Fix direction:** CI check: generate OpenAPI + contract tests against Zod schemas; block merges on drift.
- **Acceptance check:** CI fails when a route removes a field still documented (or adds a field undocumented).

### B-036 — `postinstall` runs `prisma generate` + `tsc`
- **Area:** backend/devx · **Evidence:** `acbu-backend/package.json` (`postinstall`)
- **Impact:** Slow installs; CI surprises; failures on restricted networks.
- **Fix direction:** Move TypeScript compile to an explicit CI step; keep `postinstall` to `prisma generate` only.
- **Acceptance check:** `pnpm install <2 min` on a clean cache (target); CI has a deterministic compile step.

### B-037 — Jest `passWithNoTests` weak gate
- **Area:** backend/qa · **Evidence:** `acbu-backend/package.json` (`test` script)
- **Impact:** Missing tests do not fail CI.
- **Fix direction:** Remove the flag once smoke tests exist; require minimum coverage thresholds in money modules.
- **Acceptance check:** CI fails if no tests under `src/services/transfer` (or other money-present paths).

### B-038 — No transfer pagination cursors
- **Area:** backend/api · **Evidence:** `acbu-backend/src/controllers/transferController.ts`
- **Impact:** Clients cannot paginate beyond the first 50.
- **Fix direction:** Cursor pagination + composite index on `(userId, createdAt)`.
- **Acceptance check:** Client walks entire history without duplicates.

### B-039 — Investment withdrawal list lacks pagination
- **Area:** backend/api · **Evidence:** `acbu-backend/src/controllers/investmentController.ts`
- **Impact:** Same 50-row cap as transfers.
- **Fix direction:** Cursor + filters by status; matching DB index.
- **Acceptance check:** Large org history loads progressively.

### B-042 — Webhook raw body JSON fallback empty object
- **Area:** backend/webhooks · **Evidence:** `acbu-backend/src/index.ts`
- **Impact:** Invalid JSON becomes `{}`; handlers may run on bad data.
- **Fix direction:** Reject non-JSON with 400 before routing; fail closed.
- **Acceptance check:** Malformed payload raises a 400 and never reaches business logic.

### B-044 — Basket weight computation job missing
- **Area:** backend/ops · **Evidence:** Documented in `PROJECT/issues/BACKEND_ISSUES.md`
- **Impact:** Weights drift from policy without automation.
- **Fix direction:** Scheduled job (`metricsService`/`proposedWeightsJob`) + audit log + manual approval gate.
- **Acceptance check:** Weekly job emits a diff report against the active basket config.

### B-045 — Fintech multi-provider per country
- **Area:** backend/integrations · **Evidence:** `acbu-backend` webhook + payout modules
- **Impact:** Single Flutterwave path = outage risk.
- **Fix direction:** Router chooses provider by health + fees + country per `correction.md` (e.g. NGN: Paystack + Flutterwave).
- **Acceptance check:** Simulated provider outage fails over; both providers exercised in tests.

### B-047 — Segment guard `requireMinTier` unused
- **Area:** backend/authz · **Evidence:** `acbu-backend/src/middleware/segmentGuard.ts`
- **Impact:** `User.tier` field unused → wrong product gating at the middleware layer.
- **Fix direction:** Apply middleware to premium routes; add tests for tier-bound routes.
- **Acceptance check:** Free-tier user cannot call SME-only endpoints in tests.

### B-050 — Helmet / security headers baseline
- **Area:** backend/security · **Evidence:** `acbu-backend/src/index.ts`
- **Impact:** Missing headers increase XSS/clickjacking risk.
- **Fix direction:** Helmet defaults + CSP for any server-rendered/static content; documented exceptions.
- **Acceptance check:** `securityheaders.io` reports no worse than `A-` (or documented exceptions).

### B-052 — Structured logging for money movements
- **Area:** backend/observability · **Evidence:** `acbu-backend/src/config/logger.ts` usage
- **Impact:** Insufficient fields for forensic audit (amount, currency, user, idempotency key).
- **Fix direction:** Standard log schema for financial events.
- **Acceptance check:** Log query returns the full transfer lifecycle by user/idempotency key.

### B-055 — Database migration CI gate
- **Area:** backend/ci · **Evidence:** `.github/workflows`
- **Impact:** Destructive migrations ship without warning.
- **Fix direction:** Ensure `prisma migrate diff` or deploy dry-run in CI; require label for destructive migrations.
- **Acceptance check:** CI fails on destructive migration without the required label.

### B-056 — Prisma Accelerate vs direct URL confusion
- **Area:** backend/config · **Evidence:** `acbu-backend/README.md` + `ENV_VARS.md`
- **Impact:** Wrong URL used for migrations vs runtime.
- **Fix direction:** Document matrix; startup assert roles.
- **Acceptance check:** Boot logs show correct URL type per purpose.

### B-058 — Encryption at rest for PII columns
- **Area:** backend/compliance · **Evidence:** `prisma/schema.prisma`
- **Impact:** DB leak exposes plaintext government IDs.
- **Fix direction:** App-level encryption or DB TDE; KMS rotation; documented key management.
- **Acceptance check:** Security review sign-off for the chosen field list and key lifecycle.

### B-059 — GDPR export & delete endpoints
- **Area:** backend/compliance · **Evidence:** User settings (verify coverage)
- **Impact:** Regulatory exposure for EU users.
- **Fix direction:** Data export job + tombstone deletes; documented legal data categories.
- **Acceptance check:** DPO checklist satisfied; sample export contains only documented fields.

### B-062 — S3 presigned URL abuse controls
- **Area:** backend/storage · **Evidence:** KYC uploads (if S3)
- **Impact:** Open buckets / long TTL enable exfiltration.
- **Fix direction:** Short TTL + content-type constraint + virus scan hook.
- **Acceptance check:** Pen test cannot read other users' KYC objects.

### B-063 — OpenAI usage guardrails
- **Area:** backend/ai · **Evidence:** `package.json` includes `openai`
- **Impact:** Prompt injection or runaway spend.
- **Fix direction:** Auth + per-org budget + prompt allowlist; spend alerts.
- **Acceptance check:** Spending caps enforced in staging load test.

### B-064 — Dependency vulnerability scanning
- **Area:** backend/security · **Evidence:** `pnpm-lock.yaml`
- **Impact:** Known CVEs in transitive deps.
- **Fix direction:** Enable Dependabot/Snyk + monthly cadence.
- **Acceptance check:** CI fails on critical CVE without an explicit waiver label.

### B-066 — 2FA challenge token purpose binding
- **Area:** backend/auth · **Evidence:** `acbu-backend/src/utils/jwt.ts`
- **Impact:** Reuse across flows if `aud`/`iss` checks incomplete.
- **Fix direction:** Add `aud`/`iss` claims; rotate secrets.
- **Acceptance check:** Challenge token cannot be used as a general API token.

### B-067 — Account enumeration on sign-in
- **Area:** backend/auth · **Evidence:** `acbu-backend/src/controllers/auth.ts` (verify responses)
- **Impact:** Attackers learn which emails exist via timing / response differences.
- **Fix direction:** Uniform error messages; CAPTCHA after N tries; throttling; normalize timing.
- **Acceptance check:** Timing differences and message differences are below the noise threshold.

### B-069 — Content-Type and body size limits
- **Area:** backend/security · **Evidence:** Express JSON parser
- **Impact:** Large JSON DoS.
- **Fix direction:** Lower default limit; per-route overrides for upload endpoints.
- **Acceptance check:** 10MB payload rejected where inappropriate.

### B-072 — Consistent error JSON schema
- **Area:** backend/api · **Evidence:** `errorHandler.ts` vs legacy `{ error: string }`
- **Impact:** Frontend parsing brittle.
- **Fix direction:** Standard envelope `{ error: { code, message, details } }` everywhere.
- **Acceptance check:** OpenAPI examples match runtime errors.

### B-074 — Double-submit burn with same blockchain hash
- **Area:** backend/burn · **Evidence:** `burnController.ts` optional `blockchain_tx_hash`
- **Impact:** Replay duplicates redemption records.
- **Fix direction:** Unique index on hash when present + idempotency-key handling.
- **Acceptance check:** Second submit returns the original id; no duplicate row created.

---

## 6. Low Severity Issues (🟢)

### B-030 — Standard IP rate limit message mismatch
- **Area:** backend/ratelimit · **Evidence:** `acbu-backend/src/middleware/rateLimiter.ts` (`createRateLimiter`)
- **Impact:** Message says “IP” while some routes use API keys.
- **Fix direction:** Contextual messages; separate limiters.
- **Acceptance check:** 429 JSON explains which limit tripped.

### B-032 — Shallow `/health` does not prove readiness
- **Area:** backend/ops · **Evidence:** `acbu-backend/src/routes/index.ts` (`/health`)
- **Impact:** LB marks instance ready while DB/queue is down.
- **Fix direction:** Optional `/health/ready` for k8s; document split.
- **Acceptance check:** K8s readiness uses the deep check when configured.

### B-035 — README still references npm only
- **Area:** backend/devx · **Evidence:** `acbu-backend/README.md`
- **Impact:** Contributors mix npm/pnpm; lockfile drift.
- **Fix direction:** Document pnpm as canonical; align scripts.
- **Acceptance check:** Single package manager in README + CI.

### B-040 — Government list `limit` NaN hazard
- **Area:** backend/api · **Evidence:** `acbu-backend/src/controllers/governmentController.ts`
- **Impact:** `Number(NaN)` can break Prisma `take`/`skip`.
- **Fix direction:** Zod coerce int with min/max defaults.
- **Acceptance check:** Invalid limit returns 400 with structured error.

### B-041 — Prisma `onRampSwap` typed via `as any` casts
- **Area:** backend/types · **Evidence:** Jobs/controllers casting prisma client
- **Impact:** Hides compile-time safety for financial rows.
- **Fix direction:** Regenerate client; fix schema naming; remove casts.
- **Acceptance check:** No `(prisma as any)` in `src/jobs`.

### B-043 — Notification service placeholder sender
- **Area:** backend/notifications · **Evidence:** `acbu-backend/src/services/notification/notificationService.ts`
- **Impact:** Emails may bounce or hit spam.
- **Fix direction:** Configure SES/SendGrid with verified domain; DKIM/SPF green.
- **Acceptance check:** DNS DKIM/SPF green in staging; smoke test delivers to a known inbox.

### B-046 — Limits config hardcoded
- **Area:** backend/config · **Evidence:** `acbu-backend/src/config/limits.ts`
- **Impact:** Ops cannot tune without redeploy.
- **Fix direction:** Load from DB or env with hot reload cache.
- **Acceptance check:** Limit change applies without redeploy.

### B-048 — Permissions typed as loose strings
- **Area:** backend/types · **Evidence:** `acbu-backend/src/middleware/auth.ts` (`validatePermissions`)
- **Impact:** Typos in DB JSON grant wrong access.
- **Fix direction:** Zod enum list for permissions at write time.
- **Acceptance check:** Invalid permission rejected at admin API.

### B-051 — Request ID propagation
- **Area:** backend/observability · **Evidence:** Express middleware stack
- **Impact:** Hard to correlate user reports with logs.
- **Fix direction:** Add `x-request-id` middleware; log field; pass through downstream calls.
- **Acceptance check:** Support engineers can trace a single request across services end-to-end.

### B-054 — Distributed tracing (OpenTelemetry)
- **Area:** backend/observability · **Evidence:** N/A
- **Impact:** Cross-service latency is opaque.
- **Fix direction:** OTel exporter with sample rates; per-route overrides.
- **Acceptance check:** Trace links Prisma + Soroban submit span under load.

### B-057 — Mongo index strategy for hot keys
- **Area:** backend/data · **Evidence:** Mongo collections for cache/ratelimit
- **Impact:** Hot keys slow under load.
- **Fix direction:** Index design review + TTL policies.
- **Acceptance check:** Load-test p95 under SLO; TTL confirmed for ephemeral collections.

### B-065 — Time sync / clock skew handling for JWT
- **Area:** backend/auth · **Evidence:** `acbu-backend/src/utils/jwt.ts`
- **Impact:** Skew causes confusing 401s.
- **Fix direction:** Leeway config + NTP monitoring; document skew tolerance.
- **Acceptance check:** Clock-skew test passes within the documented window.

### B-070 — Input sanitization for search endpoints
- **Area:** backend/api · **Evidence:** Recipient resolve / user search
- **Impact:** SQL/NoSQL injection via regex.
- **Fix direction:** Parameterized queries only; escape user-supplied regex.
- **Acceptance check:** Security scanner reports clean for all recipient/user search paths.

### B-071 — Versioning and deprecation headers
- **Area:** backend/api · **Evidence:** `config.apiVersion`
- **Impact:** Clients stuck on breaking changes.
- **Fix direction:** `Sunset` header + changelog endpoint.
- **Acceptance check:** Old API version returns the warning header and changelog link.

---

## 7. Legacy 56-Item Origin List (cross-reference)

The legacy list previously alone at this path was a more recent manual audit with slightly different framing. Items are preserved here for traceability; each is tagged `(moved)` (when it cleanly maps to a canonical `B-###` ID) or `(new)` (when it captures a distinct finding that should be merged into the canonical list in a follow-up PR).

### Critical
1. **API key validation broken** – src/middleware/auth.ts: `validateApiKey` uses `bcrypt.hash` for lookup (random salt) → stored hash never matches. Use `bcrypt.compare`. **Distinct finding; not in the canonical 75-item list. Should be promoted to a follow-up canonical ID (expected `B-076`) in a separate PR.** *(new — propose B-076)*
2. **Wrong import in `usdcConvertAndMintJob`** – Imports `db` from `../config/database` but module exports `prisma`. **Maps to a runtime variant of B-003 / B-075; tracked together.** *(moved → B-003 + B-075)*
3. **IDOR / auth bypass in savings controller** – `user` taken from body/query, not authenticated user. **Combine with B-007 + segment guards.** *(moved → B-007)*
4. **Env validation too late** – `requiredEnvVars` checked after `config` is built. **Folds into B-061.** *(moved → B-061)*

### High
5. **Infinite retries for failed webhooks** – *(moved → B-004)*
6. **Organization users never notified for investment withdrawal** – *(moved → B-024)*
7. **Burn limits bypass when `audience` is unset** – Mirror B-007 in the burn path. **Folds into B-005 / B-007 / B-009 audit.** *(moved → B-007 + B-009)*
8. **Mint limits bypass when `audience` is unset** – Same root cause as #7. *(moved → B-007 + B-001)*
9. **Fee thresholds likely wrong** – *(moved → B-009)*
10. **Decimal vs string for `acbuAmount` in transfer** – *(moved → B-008)*
11. **No dead-letter queues** – Folds into B-003, B-004, B-026. *(moved → B-003 + B-004)*
12. **No max retry limit for webhook consumer** – Folds into B-003, B-004, B-026. *(moved → B-004)*

### Medium
13. **Wallet activation uses XLM, not Pi** – *(moved → B-014)*
14. **Remove placeholder Stellar address in auth** – *(moved → B-013)*
15. **Transfer service: verify alias-based flow** – Folds into B-007 + B-008. *(moved → B-007 + B-008)*
16. **Contract client and Stellar fees hardcoded** – *(moved → B-015)*
17. **Multiple fintech providers per nation** – *(moved → B-045)* (also reinforced by `correction.md`)
18. **KYC extraction placeholder** – *(moved → B-017)*
19. **Basket weight script missing** – *(moved → B-044)*
20. **Balance endpoint for frontend missing** – Cross-cutting concern surfaced in `issues/frontend.md` `F-025`; see **Cross-Reference Map**. *(new — cross-cutting)*
21. **No dependency health checks** – Reinforces B-031 / B-032. *(moved → B-031 + B-032)*
22. **Race condition in API key rate limiting** – *(moved → B-028)*
23. **ReDoS risk in `deletePattern`** – *(moved → B-027)*
24. **Webhook verification skipped when secret missing** – *(moved → B-005)*
25. **Placeholder Stellar address format invalid** – *(moved → B-013)*
26. **ACBU supply from DB, not chain** – *(moved → B-010)*
27. **Org-scoped limits may not apply** – *(moved → B-011)*
28. **USDC to XLM conversion is a no-op** – *(moved → B-003 + B-075)*
29. **Hardcoded XLM rate** – *(moved → B-002 + B-015)*
30. **Deposit limits use wrong USD equivalent** – *(moved → B-001)*
31. **Incomplete treasury aggregation** – *(moved → B-018)*
32. **Audit failures are silent** – *(moved → B-026)*

### Low
33. **`any` cast for permissions** – *(moved → B-048)*
34. **`any` usage in segmentGuard** – *(moved → B-047 + B-048)*
35. **`any` usage in Stellar client** – *(moved → B-041)*
36. **`any` cast in burning service** – *(moved → B-041)*
37. **No pagination on transfers** – *(moved → B-038)*
38. **No pagination on investment withdrawals** – *(moved → B-039)*
39. **`limit` query param not validated** – *(moved → B-040)*
40. **No input validation in savings controller** – *(moved → B-012)*
41. **Missing composite index** – Tied to B-038 / B-039. *(moved → B-038 + B-039)*
42. **Webhook JSON parse failure returns empty body** – *(moved → B-042)*
43. **Stack traces in error logs** – Cross-cutting; folded into B-051 / B-052 + a structured error envelope (B-072). *(moved → B-051 + B-052 + B-072)*

### Trivial
44. **`(prisma as any).onRampSwap` casts** – *(moved → B-041)*
45. **Hardcoded limit values** – *(moved → B-046)*
46. **Fallback email placeholder** – *(moved → B-043)*
47. **Yield accounting stub** – *(moved → B-025)*
48. **No automated tests** – *(moved → B-037)* + general backend hygiene
49. **Client-controlled mint recipient** – *(moved → B-007)*
50. **Rate limiting fails open when MongoDB cache is down** – *(moved → B-029)*
51. **Recovery unlock is single-factor** – *(moved → B-006)*
52. **`requireMinTier` middleware is never used** – *(moved → B-047)*
53. **`InvestmentWithdrawalRequest.organizationId` has no FK** – Direct prisma schema follow-up; tracks alongside B-024 and B-011. *(new — schema)*
54. **Unauthenticated reserve metrics on `/health/metrics`** – *(moved → B-031)*
55. **Swagger UI exposed without auth** – *(moved → B-033)*
56. **Stellar event listener: broad stream and swallowed errors** – *(moved → B-016)*

> **Reconciliation policy:** When a follow-up audit pass opens a new canonical ID for items tagged `(new)`, the row is migrated forward and the tagged comment block in this section is updated to point at the new ID. New IDs are expected to land in `B-076+` and should preserve backward-reference to this section so future readers can trace provenance.

---

## 8. Resolution Tracker (Fix Status)

> **Format:** `ID · title · status · linked PR / commit · owner`
>
> **Forward-looking template.** Every item is currently 🟡 Open; uniformity of status reflects the state of the working backlog. When a fix merges, the row transitions to ✅ Fixed with a PR link. Example anchors below show what a real fixed row looks like.

| ID | Title | Status | Notes |
|----|-------|--------|-------|
| _(example)_ | _(example anchor for prior documentary PRs — PR #14 for issue #1, PR #22 for issue #12)_ | ✅ Fixed (PR #22 merged) | Documentary PR — not a code fix; historical anchor |
| B-005 | Webhook signature fail-open | 🟡 Open (ship blocker) | Tracked with auth refactor |
| B-006 | Recovery single-factor | 🟡 Open (ship blocker) | Tracked with auth refactor |
| B-001..B-075 (others) | _see Summary Table_ | 🟡 Open | All canonical IDs pending remediation |

> **Status legend** — 🟢 Trivial/cosmetic · 🟡 Open (tracked) · 🔵 In review · ✅ Fixed (PR linked)
>
> **Reminder when updating:** When an external PR closes an issue, replace `🟡 Open` with `✅ Fixed` and append the merged PR link in the Notes column. Do not rewrite history; the row itself is the audit trail.

---

## 9. Cross-Reference Map

| Document | Relationship to this file |
|----------|---------------------------|
| [`../../issues/backend.md`](../../issues/backend.md) | **Live triage-ready catalog.** Same IDs and severities; entries are condensed for daily use. |
| [`../../issues/MASTER_INDEX.md`](../../issues/MASTER_INDEX.md) | Master index across backend/frontend/contracts backlogs. |
| [`../../issues/contracts.md`](../../issues/contracts.md) | Smart contracts MVP issues (C-001..C-060). |
| [`../../issues/frontend.md`](../../issues/frontend.md) | Frontend MVP issues (F-001..F-065). |
| [`../API_AND_CONTRACTS_REFERENCE.MD`](../API_AND_CONTRACTS_REFERENCE.MD) | Routes ↔ contracts mapping. |
| [`../PROJECT_STRUCTURE.MD`](../PROJECT_STRUCTURE.MD) | Repo layout (where the backend source lives). |
| [`../../TECHNICAL/ARCHITECTURE.MD`](../../TECHNICAL/ARCHITECTURE.MD) | High-level architecture. |
| [`../../TECHNICAL/API_SPECIFICATION.MD`](../../TECHNICAL/API_SPECIFICATION.MD) | REST API endpoints. |
| [`../../TECHNICAL/DATABASE_SCHEMA.MD`](../../TECHNICAL/DATABASE_SCHEMA.MD) | Database tables and relationships. |
| [`../../OPERATIONS/REGULATORY.MD`](../../OPERATIONS/REGULATORY.MD) | Compliance/regulatory rationale for B-058 (PII encryption at rest), B-059 (GDPR export/delete), B-067 (account enumeration guard). |
| [`../../OPERATIONS/FINTECH_PARTNERSHIPS.MD`](../../OPERATIONS/FINTECH_PARTNERSHIPS.MD) | Multi-provider context for B-045 (also referenced in `correction.md`). |
| [`../../OPERATIONS/LIMITS_AND_TIERS.MD`](../../OPERATIONS/LIMITS_AND_TIERS.MD) | Source of truth for B-001, B-009, B-011, B-046 limit tables. |
| [`../../USER_EXPERIENCE/MERCHANT_INTEGRATION.MD`](../../USER_EXPERIENCE/MERCHANT_INTEGRATION.MD) | bill-pay / merchant context for B-022. |
| [`../../correction.md`](../../correction.md) | Standing list of items curated for fast triage (multi-fintech, basket weight). |
| [`../CONTRACTS_ISSUES.md`](../CONTRACTS_ISSUES.md) | Companion long-form reference for smart contracts. |

---

## 10. Top Remediations (ship-safety order)

Order these fixes in the MVP launch runbook before deploying mainnet:

1. **B-005 / B-006** — Webhook fail-closed + recovery second factor (single auth-week PR).
2. **B-007** — Bind all mint / burn / on-ramp recipients to `User.stellarAddress`.
3. **B-001 / B-009** — Correct USD proxy for mint deposits and burn fee math.
4. **B-002 / B-008** — `Decimal` end-to-end on money paths.
5. **B-003 / B-004 / B-075** — RabbitMQ retry caps + DLQ + atomic status transitions.
6. **B-010 / B-011** — Reserve truth reconciliation + org-scoped limits.
7. **B-013 / B-014 / B-015** — Signup Stellar address hygiene + wallet asset choice + dynamic fees.
8. **B-016 / B-028 / B-049** — Listener scope + atomic rate limiter + CORS allowlist.
9. **B-017 / B-020** — KYC machine layer + enterprise bulk transfer.
10. **B-026 / B-053 / B-061 / B-068 / B-073** — Audit durability, idempotency keys, env schema, brute-force guard, transaction state machine.
11. **B-018 / B-021 / B-022 / B-023 / B-024 / B-025** — Segment-API stubs (government, enterprise, bills, investment notifications, yield accounting).
12. **B-031 / B-032 / B-036 / B-037 / B-044 / B-045** — Ops hygiene (deep health, postinstall, jest gate, basket job, multi-fintech).

---

## 11. Maintenance

- **New issues** — Add to `issues/backend.md` first (canonical triage). Mirror to this file with the next stable `B-###` ID.
- **Closed fixes** — Promote the fix to `✅ Fixed (PR linked)` in Section 8 and link to the merged PR.
- **Legacy `(new)` items in Section 7** — Open a follow-up canonical-ID PR; on merge, update both `issues/backend.md` and this file.
- **Cross-doc references** — Update `issues/MASTER_INDEX.md` summary counts when severities change.

---

## Related Documents

- [Live backend issue catalog](../../issues/backend.md)
- [Master backlog index](../../issues/MASTER_INDEX.md)
- [Frontend issues](../../issues/frontend.md)
- [Contracts long-form reference](../CONTRACTS_ISSUES.md)
- [Smart contract spec (root copy)](../SMART_CONTRACT_SPEC.md)
- [Project structure map](../PROJECT_STRUCTURE.md)
- [Tech architecture](../../TECHNICAL/ARCHITECTURE.md)
- [API specification](../../TECHNICAL/API_SPECIFICATION.md)
- [Limits & tiers](../../OPERATIONS/LIMITS_AND_TIERS.md)
- [Fintech partnerships](../../OPERATIONS/FINTECH_PARTNERSHIPS.md)

---

_Last consolidated: June 2026. Source content merged from legacy `PROJECT/issues/BACKEND_ISSUES.md` list (56 items) and canonical [`issues/backend.md`](../../issues/backend.md) (75 items). Severity buckets and IDs fully aligned with canonical catalog; legacy 56-item list preserved as Section 7 with `(moved)` and `(new)` tags for traceability. New audit findings captured under `(new)` should be promoted to canonical IDs (`B-076+`) in a follow-up PR._
