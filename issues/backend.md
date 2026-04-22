# Backend — MVP issues backlog

This file is the **canonical MVP flaw list** for the Node/Express API (`acbu-backend`). Older notes live under [../PROJECT/issues/BACKEND_ISSUES.md](../PROJECT/issues/BACKEND_ISSUES.md); items there may be **fixed, outdated, or duplicated**—treat this backlog as triage-ready.

**Issue format:** each entry is one flaw or improvement with severity, evidence, impact, fix hint, and acceptance.

---

### B-001 — Mint basket deposit limits use wrong USD proxy
**Severity:** High · **Area:** backend/minting · **Evidence:** `acbu-backend/src/controllers/mintController.ts` (`depositFromBasketCurrency`)
**Impact:** Local currency amounts are treated as USD for `checkDepositLimits`, so limits can be wildly wrong. **Fix direction:** Convert basket→USD using oracle/rates before limit checks; add tests for NGN/KES. **Acceptance check:** Limits match policy for each basket currency in integration tests.

### B-002 — Floating `Number()` on money paths
**Severity:** High · **Area:** backend/money · **Evidence:** Multiple controllers (e.g. `mintController.ts`, `burnController.ts`)
**Impact:** `Number()` on user-supplied strings loses precision vs `Decimal` and can skew fees/limits. **Fix direction:** Parse monetary strings with decimal library end-to-end; only coerce at Soroban boundary with explicit rounding. **Acceptance check:** Golden tests for large fractional inputs and fee boundaries.

### B-003 — USDC convert job infinite requeue on failure
**Severity:** High · **Area:** backend/jobs · **Evidence:** `acbu-backend/src/jobs/usdcConvertAndMintJob.ts` (`ch.nack(..., true)`)
**Impact:** Poison messages can spin forever, duplicating side effects or stalling the queue. **Fix direction:** Add retry caps + DLQ routing; mark `OnRampSwap` failed with reason after N attempts. **Acceptance check:** Failed swap lands in DLQ; swap row reaches terminal state; no infinite nack loop.

### B-004 — Webhook delivery retry semantics
**Severity:** High · **Area:** backend/webhooks · **Evidence:** `acbu-backend/src/services/webhook/webhookService.ts` (per prior audits)
**Impact:** Partners may be hammered or events retried without idempotency, causing duplicate settlements. **Fix direction:** Per-delivery idempotency keys, exponential backoff, max attempts, DLQ + alerting. **Acceptance check:** Webhook consumer stops after max failures; DLQ visible in ops dashboard.

### B-005 — Webhook signature verification fail-open risk
**Severity:** Critical · **Area:** backend/webhooks · **Evidence:** `acbu-backend/src/controllers/webhookController.ts` / `verifyFlutterwaveSignature` (verify pattern)
**Impact:** If secret env vars are missing, forged webhooks could mint credit in downstream handlers. **Fix direction:** Fail closed in production when secret unset; separate dev/stage mocks. **Acceptance check:** Production boot fails or webhooks 401 when secret missing.

### B-006 — Recovery unlock as single-factor account recovery
**Severity:** Critical · **Area:** backend/auth · **Evidence:** `acbu-backend/src/services/recovery/recoveryService.ts` (per `PROJECT/issues/BACKEND_ISSUES.md`)
**Impact:** Passcode + identifier alone may issue a new API key = takeover if passcode leaks. **Fix direction:** Add OTP/email/device proof + rate limits + audit; rotate sessions. **Acceptance check:** E2E test: cannot obtain API key without second factor.

### B-007 — Client-supplied mint recipient vs user wallet
**Severity:** High · **Area:** backend/minting · **Evidence:** `acbu-backend/src/controllers/mintController.ts`, on-ramp controllers (cross-check routes)
**Impact:** If any path still accepts `wallet_address` without binding to `User.stellarAddress`, funds can be sent elsewhere. **Fix direction:** Centralize `assertUserWalletAddress` on every mint/on-ramp path; deny mismatches. **Acceptance check:** Security test: cannot mint to arbitrary `G...` not owned by user.

### B-008 — Transfer service Decimal typing
**Severity:** High · **Area:** backend/transfers · **Evidence:** `acbu-backend/src/services/transfer/transferService.ts`
**Impact:** String amounts into Prisma Decimal fields risk precision/type errors at runtime. **Fix direction:** Construct `Decimal` explicitly; validate with Zod `string` + decimal regex. **Acceptance check:** Unit tests for max precision transfers.

### B-009 — Burn fee policy threshold logic
**Severity:** High · **Area:** backend/fees · **Evidence:** `acbu-backend/src/services/feePolicy/feePolicyService.ts`
**Impact:** Suspected inverted/typo thresholds can charge wrong BPS vs reserve basket state. **Fix direction:** Review math vs spec; add property tests against documented tables. **Acceptance check:** Fee outputs match `OPERATIONS/LIMITS_AND_TIERS` / product spec.

### B-010 — ReserveTracker total supply from DB not chain
**Severity:** High · **Area:** backend/reserves · **Evidence:** `acbu-backend/src/services/reserve/ReserveTracker.ts`
**Impact:** On-chain supply and indexed DB can diverge; circuit breakers may use wrong numerator. **Fix direction:** Periodic chain truth reconciliation; alert on drift. **Acceptance check:** Dashboard shows DB vs on-chain delta under threshold.

### B-011 — Org-scoped transaction limits with null userId
**Severity:** High · **Area:** backend/limits · **Evidence:** `acbu-backend/src/services/limits/limitsService.ts`
**Impact:** Org keys creating `userId: null` rows may be excluded from aggregates → limits bypass. **Fix direction:** Include `organizationId` dimension consistently in limit queries. **Acceptance check:** Org key cannot exceed org daily cap in tests.

### B-012 — Savings controller input validation
**Severity:** Medium · **Area:** backend/savings · **Evidence:** `acbu-backend/src/controllers/savingsController.ts`
**Impact:** `amount` / `term_seconds` are unchecked before contract calls → bad Soroban args or user errors. **Fix direction:** Zod schemas; bounds on term; decimal string for amount. **Acceptance check:** 400 on invalid body with structured errors.

### B-013 — Placeholder / invalid Stellar address on signup
**Severity:** High · **Area:** backend/auth · **Evidence:** `acbu-backend/src/services/auth/authService.ts` (`getPlaceholderStellarAddress`)
**Impact:** Non-`G` placeholders can break downstream Stellar validation and UX. **Fix direction:** Defer DB constraint until wallet created, or use valid funded temp account pattern. **Acceptance check:** No user row ships with invalid `stellarAddress` format in prod.

### B-014 — Wallet activation funding asset choice
**Severity:** Medium · **Area:** backend/wallet · **Evidence:** `acbu-backend/src/services/wallet/walletActivationService.ts`
**Impact:** Product docs mention Pi vs XLM; wrong asset strands users on target network. **Fix direction:** Align with `TESTNET_CUSTODIAL_BOOTSTRAP` / network config; feature flag. **Acceptance check:** Activation succeeds on configured network with correct asset.

### B-015 — Hardcoded Soroban inclusion fees
**Severity:** Medium · **Area:** backend/stellar · **Evidence:** `acbu-backend/src/services/stellar/contractClient.ts` and related services
**Impact:** Fee 100 may fail under congestion or be overpaid; unpredictable confirmation. **Fix direction:** Fetch base fee + soroban resource fees; configurable caps. **Acceptance check:** Transactions succeed under load test without manual fee edits.

### B-016 — Stellar event listener scope and error handling
**Severity:** High · **Area:** backend/stellar · **Evidence:** `acbu-backend/src/services/stellar/eventListener.ts` (per prior audit)
**Impact:** Broad Horizon streams + swallowed errors can miss mint/burn indexing silently. **Fix direction:** Filter to contract IDs; structured retries; DLQ for parse failures. **Acceptance check:** Injected test event always reaches projection store.

### B-017 — KYC machine layer is placeholder
**Severity:** High · **Area:** backend/kyc · **Evidence:** `acbu-backend/src/services/kyc/machineLayer.ts`
**Impact:** Automated checks/redaction not production-grade. **Fix direction:** Integrate vision provider + PII redaction pipeline + manual review queue. **Acceptance check:** Sample IDs pass/fail deterministically in staging with redacted logs.

### B-018 — Government treasury aggregation stub
**Severity:** Medium · **Area:** backend/government · **Evidence:** `acbu-backend/src/controllers/governmentController.ts`
**Impact:** Operators lack consolidated exposure view. **Fix direction:** Aggregate from Prisma + reserve segments; cache with TTL. **Acceptance check:** Endpoint returns non-empty `byCurrency` in seeded env.

### B-019 — Salary APIs not implemented
**Severity:** Medium · **Area:** backend/salary · **Evidence:** `acbu-backend/src/controllers/salaryController.ts`
**Impact:** MVP payroll promises unfulfilled; clients get 501/empty arrays. **Fix direction:** Batch transfers + schedules + idempotency keys. **Acceptance check:** Postman collection green for happy path.

### B-020 — Enterprise bulk transfer not implemented
**Severity:** Medium · **Area:** backend/enterprise · **Evidence:** `acbu-backend/src/controllers/enterpriseController.ts`
**Impact:** Large merchants cannot upload batches. **Fix direction:** Chunked processing + per-row idempotency + reporting. **Acceptance check:** 10k row CSV completes within SLO with partial failure report.

### B-021 — Enterprise treasury view stub
**Severity:** Medium · **Area:** backend/enterprise · **Evidence:** `acbu-backend/src/controllers/enterpriseController.ts` (`getTreasury`)
**Impact:** Treasury nulls block finance workflows. **Fix direction:** Join transfers, reserves, and FX snapshots. **Acceptance check:** Treasury totals reconcile to ledger within tolerance.

### B-022 — Bills catalog and pay not implemented
**Severity:** Medium · **Area:** backend/bills · **Evidence:** `acbu-backend/src/controllers/billsController.ts`
**Impact:** Bill pay segment is non-functional. **Fix direction:** Partner adapter + webhook reconciliation + refunds. **Acceptance check:** Happy path bill payment creates auditable transaction.

### B-023 — Investment deployable allocation ignores deployed
**Severity:** Medium · **Area:** backend/investment · **Evidence:** `acbu-backend/src/services/investment/allocationService.ts`
**Impact:** `deployedUsd` pinned to 0 can over-allocate to strategies. **Fix direction:** Track deployed notional in DB; subtract from deployable. **Acceptance check:** Allocation never exceeds policy with non-zero deployments.

### B-024 — Investment withdrawal notifications for org requests
**Severity:** Medium · **Area:** backend/jobs · **Evidence:** `acbu-backend/src/jobs/investmentWithdrawalJob.ts` (per prior audit)
**Impact:** Org-only requests may never notify approvers. **Fix direction:** Notify org admins via email/push using `organizationId`. **Acceptance check:** Org withdrawal triggers notification in test inbox.

### B-025 — Yield accounting stub
**Severity:** Medium · **Area:** backend/investment · **Evidence:** `acbu-backend/src/services/investment/yieldAccountingService.ts`
**Impact:** Cannot report yield or tax basis. **Fix direction:** Implement accrual postings tied to vault positions. **Acceptance check:** Monthly statement API returns non-zero yield on seeded data.

### B-026 — Audit write failures only logged
**Severity:** High · **Area:** backend/audit · **Evidence:** `acbu-backend/src/services/audit/auditService.ts`
**Impact:** Compliance gap if audit events disappear silently. **Fix direction:** Retry + outbox to Mongo/queue; alert on sustained failure. **Acceptance check:** Chaos test: audit eventually consistent.

### B-027 — Cache `deletePattern` ReDoS risk
**Severity:** Medium · **Area:** backend/cache · **Evidence:** `acbu-backend/src/utils/cache.ts`
**Impact:** User-influenced patterns could hang Node regex engine. **Fix direction:** Escape/limit pattern length; use safe glob or prefix scan. **Acceptance check:** Fuzz test cannot stall event loop beyond budget.

### B-028 — API key rate limiter non-atomic increment
**Severity:** High · **Area:** backend/ratelimit · **Evidence:** `acbu-backend/src/middleware/rateLimiter.ts`
**Impact:** Race allows burst over limit under concurrency. **Fix direction:** Use Mongo atomic `$inc` with document-level cap or Redis INCR. **Acceptance check:** Concurrent load test cannot exceed configured QPS.

### B-029 — Rate limiter Mongo outage fail-open
**Severity:** Medium · **Area:** backend/ratelimit · **Evidence:** `acbu-backend/src/middleware/rateLimiter.ts` + `cacheService`
**Impact:** When cache errors, fallback may weaken per-key limits (see prior audit). **Fix direction:** Stricter IP fallback + circuit breaker + metrics. **Acceptance check:** Simulated Mongo outage triggers degraded but bounded mode.

### B-030 — Standard IP rate limit message mismatch
**Severity:** Low · **Area:** backend/ratelimit · **Evidence:** `acbu-backend/src/middleware/rateLimiter.ts` (`createRateLimiter`)
**Impact:** Message says IP while some routes use API keys. **Fix direction:** Contextual messages; separate limiters. **Acceptance check:** 429 JSON explains which limit tripped.

### B-031 — Deep health and metrics routes are public
**Severity:** Medium · **Area:** backend/ops · **Evidence:** `acbu-backend/src/routes/index.ts` (`/health/deep`, `/health/metrics`)
**Impact:** Dependency topology exposed to unauthenticated callers (reconnaissance). **Fix direction:** Protect with mTLS or admin API key; keep `/health` shallow public. **Acceptance check:** Public cannot scrape dependency failures at scale.

### B-032 — Shallow `/health` does not prove readiness
**Severity:** Low · **Area:** backend/ops · **Evidence:** `acbu-backend/src/routes/index.ts` (`/health`)
**Impact:** LB marks instance ready while DB/queue is down. **Fix direction:** Optional `/health/ready` for k8s; document split. **Acceptance check:** K8s readiness uses deep check when configured.

### B-033 — Swagger UI publicly mounted
**Severity:** Medium · **Area:** backend/docs · **Evidence:** `acbu-backend/src/index.ts` (per prior audit)
**Impact:** Attackers map surface area without auth. **Fix direction:** Gate `/api-docs` behind admin auth or disable in prod. **Acceptance check:** Prod deployment returns 404 for `/api-docs` or requires admin JWT.

### B-034 — OpenAPI drift vs implementation
**Severity:** Medium · **Area:** backend/docs · **Evidence:** `acbu-backend` swagger annotations vs routes
**Impact:** Clients integrate against wrong schemas. **Fix direction:** CI check: swagger build + contract tests vs zod schemas. **Acceptance check:** CI fails when route removes field still documented.

### B-035 — README still references npm only
**Severity:** Low · **Area:** backend/devx · **Evidence:** `acbu-backend/README.md`
**Impact:** Contributors mix npm/pnpm; lockfile drift. **Fix direction:** Document pnpm as canonical; align scripts. **Acceptance check:** Single package manager in README + CI.

### B-036 — `postinstall` runs prisma generate + tsc
**Severity:** Medium · **Area:** backend/devx · **Evidence:** `acbu-backend/package.json` (`postinstall`)
**Impact:** Slow installs; CI surprises; failures on restricted networks. **Fix direction:** Move compile to explicit CI step; keep `postinstall` to `prisma generate` only. **Acceptance check:** `pnpm install` under 2 minutes on clean cache (target).

### B-037 — Jest `passWithNoTests` weak gate
**Severity:** Medium · **Area:** backend/qa · **Evidence:** `acbu-backend/package.json` (`test` script)
**Impact:** Missing tests do not fail CI. **Fix direction:** Remove flag once smoke tests exist; require min coverage for money modules. **Acceptance check:** CI fails if no tests under `src/services/transfer`.

### B-038 — No transfer pagination cursors
**Severity:** Medium · **Area:** backend/api · **Evidence:** `acbu-backend/src/controllers/transferController.ts`
**Impact:** Clients cannot page beyond first 50. **Fix direction:** Cursor pagination + index on `(userId, createdAt)`. **Acceptance check:** Client walks entire history without duplicates.

### B-039 — Investment withdrawal list lacks pagination
**Severity:** Medium · **Area:** backend/api · **Evidence:** `acbu-backend/src/controllers/investmentController.ts`
**Impact:** Same 50-row cap issue. **Fix direction:** Cursor + filters by status. **Acceptance check:** Large org history loads progressively.

### B-040 — Government list `limit` NaN hazard
**Severity:** Low · **Area:** backend/api · **Evidence:** `acbu-backend/src/controllers/governmentController.ts`
**Impact:** `Number(NaN)` can break Prisma take/skip. **Fix direction:** Zod coerce int with min/max defaults. **Acceptance check:** Invalid limit returns 400.

### B-041 — Prisma `onRampSwap` typed via `as any` casts
**Severity:** Low · **Area:** backend/types · **Evidence:** Jobs/controllers casting prisma client
**Impact:** Hides compile-time safety for financial rows. **Fix direction:** Regenerate client; fix schema naming; remove casts. **Acceptance check:** No `(prisma as any)` in `src/jobs`.

### B-042 — Webhook raw body JSON fallback empty object
**Severity:** Medium · **Area:** backend/webhooks · **Evidence:** `acbu-backend/src/index.ts` (per prior audit)
**Impact:** Invalid JSON becomes `{}` → handlers proceed dangerously. **Fix direction:** Reject non-JSON with 400 before routing. **Acceptance check:** Malformed payload never hits business logic.

### B-043 — Notification service placeholder sender
**Severity:** Low · **Area:** backend/notifications · **Evidence:** `acbu-backend/src/services/notification/notificationService.ts`
**Impact:** Emails may bounce or hit spam. **Fix direction:** Configure SES/Sendgrid + verified domain. **Acceptance check:** DNS DKIM/SPF green in staging.

### B-044 — Basket weight computation job missing
**Severity:** Medium · **Area:** backend/ops · **Evidence:** Documented in `PROJECT/issues/BACKEND_ISSUES.md`
**Impact:** Weights drift from policy without automation. **Fix direction:** Scheduled job + audit log + manual approval gate. **Acceptance check:** Weekly job emits diff report.

### B-045 — Fintech multi-provider per country
**Severity:** Medium · **Area:** backend/integrations · **Evidence:** `acbu-backend` webhook + payout modules
**Impact:** Single Flutterwave path = outage risk. **Fix direction:** Router chooses provider by health + fees. **Acceptance check:** Simulated provider outage fails over.

### B-046 — Limits config hardcoded
**Severity:** Low · **Area:** backend/config · **Evidence:** `acbu-backend/src/config/limits.ts`
**Impact:** Ops cannot tune without deploy. **Fix direction:** Load from DB or env with hot reload cache. **Acceptance check:** Limit change applies without redeploy.

### B-047 — Segment guard `requireMinTier` unused
**Severity:** Medium · **Area:** backend/authz · **Evidence:** `acbu-backend/src/middleware/segmentGuard.ts`
**Impact:** Tier field unused → wrong product gating. **Fix direction:** Apply middleware to premium routes; tests. **Acceptance check:** Free tier cannot call SME-only endpoints.

### B-048 — Permissions typed as loose strings
**Severity:** Low · **Area:** backend/types · **Evidence:** `acbu-backend/src/middleware/auth.ts` (`validatePermissions`)
**Impact:** Typos in DB JSON grant wrong access. **Fix direction:** Zod enum list for permissions at write time. **Acceptance check:** Invalid permission rejected at admin API.

### B-049 — CORS policy review for credentialed browsers
**Severity:** High · **Area:** backend/security · **Evidence:** `acbu-backend/src/index.ts` / cors config (verify)
**Impact:** Misconfigured origins with credentials enable token theft via malicious site. **Fix direction:** Explicit allowlist per env; no `*` with credentials. **Acceptance check:** Security headers test passes OWASP baseline.

### B-050 — Helmet / security headers baseline
**Severity:** Medium · **Area:** backend/security · **Evidence:** `acbu-backend/src/index.ts`
**Impact:** Missing headers increase XSS/clickjacking impact for any HTML served. **Fix direction:** Helmet defaults + CSP for any static docs. **Acceptance check:** securityheaders.io A- or documented exceptions.

### B-051 — Request ID propagation
**Severity:** Low · **Area:** backend/observability · **Evidence:** Express middleware stack
**Impact:** Hard to correlate user report with logs. **Fix direction:** Add `x-request-id` middleware; log field. **Acceptance check:** Support can trace single request across services.

### B-052 — Structured logging for money movements
**Severity:** Medium · **Area:** backend/observability · **Evidence:** `acbu-backend/src/config/logger.ts` usage
**Impact:** Insufficient fields for forensic audit (amount, currency, user, idempotency). **Fix direction:** Standard log schema for financial events. **Acceptance check:** Log query returns full transfer lifecycle.

### B-053 — Idempotency-Key header support
**Severity:** High · **Area:** backend/api · **Evidence:** POST transfer/mint/burn routes
**Impact:** Retries double-charge or double-mint. **Fix direction:** Persist keys in `Idempotency` table with response cache. **Acceptance check:** Duplicate POST with same key returns identical body.

### B-054 — Distributed tracing (OpenTelemetry)
**Severity:** Low · **Area:** backend/observability · **Evidence:** N/A
**Impact:** Cross-service latency opaque. **Fix direction:** OTel exporter to vendor; sample rates. **Acceptance check:** Trace links Prisma + Soroban submit span.

### B-055 — Database migration CI gate
**Severity:** Medium · **Area:** backend/ci · **Evidence:** `.github/workflows` (verify)
**Impact:** Broken migrations ship. **Fix direction:** Ensure `prisma migrate diff` or deploy dry-run in CI. **Acceptance check:** CI fails on destructive migration without label.

### B-056 — Prisma Accelerate vs direct URL confusion
**Severity:** Medium · **Area:** backend/config · **Evidence:** `acbu-backend/README.md` + `ENV_VARS.md`
**Impact:** Wrong URL used for migrations vs runtime. **Fix direction:** Document matrix; startup assert roles. **Acceptance check:** Boot logs show correct URL type per purpose.

### B-057 — Mongo index strategy for hot keys
**Severity:** Low · **Area:** backend/data · **Evidence:** Mongo collections for cache/ratelimit
**Impact:** Hot keys slow under load. **Fix direction:** Index design review + TTL policies. **Acceptance check:** Load test p95 under SLO.

### B-058 — Encryption at rest for PII columns
**Severity:** Medium · **Area:** backend/compliance · **Evidence:** `prisma/schema.prisma` (verify fields)
**Impact:** DB leak exposes plaintext government IDs. **Fix direction:** App-level encryption or DB TDE; KMS rotation. **Acceptance check:** Security review sign-off.

### B-059 — GDPR export & delete endpoints
**Severity:** Medium · **Area:** backend/compliance · **Evidence:** User settings (verify coverage)
**Impact:** Regulatory exposure for EU users. **Fix direction:** Data export job + tombstone deletes. **Acceptance check:** DPO checklist satisfied.

### B-060 — Admin role separation
**Severity:** High · **Area:** backend/authz · **Evidence:** Admin vs user API keys
**Impact:** Support staff impersonation too powerful. **Fix direction:** Scoped admin keys + MFA + break-glass. **Acceptance check:** Admin actions attributed separately in audit.

### B-061 — Secrets in environment validation
**Severity:** High · **Area:** backend/config · **Evidence:** `acbu-backend/src/config/env.ts`
**Impact:** Late or partial validation ships half-configured prod. **Fix direction:** Strict schema (zod) at startup; fail fast. **Acceptance check:** Missing `JWT_SECRET` prevents listen.

### B-062 — S3 presigned URL abuse controls
**Severity:** Medium · **Area:** backend/storage · **Evidence:** KYC uploads (if S3)
**Impact:** Open buckets or long TTL enable exfiltration. **Fix direction:** Short TTL + content-type constraint + virus scan hook. **Acceptance check:** Pen test cannot read other users' KYC objects.

### B-063 — OpenAI usage guardrails
**Severity:** Medium · **Area:** backend/ai · **Evidence:** `package.json` includes `openai`
**Impact:** Prompt injection or runaway spend if exposed. **Fix direction:** Auth + per-org budget + prompt allowlist. **Acceptance check:** Spending caps enforced in staging load test.

### B-064 — Dependency vulnerability scanning
**Severity:** Medium · **Area:** backend/security · **Evidence:** `pnpm-lock.yaml`
**Impact:** Known CVEs in transitive deps. **Fix direction:** Enable Dependabot/Snyk + monthly cadence. **Acceptance check:** CI fails on critical CVE without waiver.

### B-065 — Time sync / clock skew handling for JWT
**Severity:** Low · **Area:** backend/auth · **Evidence:** `acbu-backend/src/utils/jwt.ts`
**Impact:** Skew causes confusing 401s. **Fix direction:** Leeway config + NTP monitoring. **Acceptance check:** Clock skew test passes.

### B-066 — 2FA challenge token purpose binding
**Severity:** Medium · **Area:** backend/auth · **Evidence:** `acbu-backend/src/utils/jwt.ts`
**Impact:** Reuse across flows if purpose check incomplete. **Fix direction:** Add `aud`/`iss` claims; rotate secrets. **Acceptance check:** Cannot reuse challenge token for API access.

### B-067 — Account enumeration on sign-in
**Severity:** Medium · **Area:** backend/auth · **Evidence:** `acbu-backend/src/controllers/auth` (verify responses)
**Impact:** Attackers learn which emails exist. **Fix direction:** Uniform error messages + CAPTCHA after N tries. **Acceptance check:** Timing differences below noise threshold.

### B-068 — Brute-force protection on auth endpoints
**Severity:** High · **Area:** backend/auth · **Evidence:** `authRoutes` + rate limiter
**Impact:** Passcodes/API keys guessable at scale. **Fix direction:** Stricter limiter on `/auth/*` + lockout policy. **Acceptance check:** Hydra test blocked after threshold.

### B-069 — Content-Type and body size limits
**Severity:** Medium · **Area:** backend/security · **Evidence:** Express JSON parser
**Impact:** Large JSON DoS. **Fix direction:** Lower default limit; per-route overrides. **Acceptance check:** 10MB payload rejected where inappropriate.

### B-070 — Input sanitization for search endpoints
**Severity:** Low · **Area:** backend/api · **Evidence:** Recipient resolve / user search
**Impact:** SQL/NoSQL injection via regex. **Fix direction:** Parameterized queries only; escape user regex. **Acceptance check:** Security scanner clean.

### B-071 — Versioning and deprecation headers
**Severity:** Low · **Area:** backend/api · **Evidence:** `config.apiVersion`
**Impact:** Clients stuck on breaking changes. **Fix direction:** `Sunset` header + changelog endpoint. **Acceptance check:** Old version returns warning header.

### B-072 — Consistent error JSON schema
**Severity:** Medium · **Area:** backend/api · **Evidence:** `errorHandler.ts` vs legacy `{error: string}`
**Impact:** Frontend parsing brittle. **Fix direction:** Standard envelope `{error:{code,message,details}}` everywhere. **Acceptance check:** OpenAPI examples match runtime.

### B-073 — Transaction status machine tests
**Severity:** High · **Area:** backend/core · **Evidence:** `prisma` `Transaction` model transitions
**Impact:** Illegal transitions corrupt balances reporting. **Fix direction:** Explicit state machine + DB constraints. **Acceptance check:** Invalid transition throws domain error.

### B-074 — Double-submit burn with same blockchain hash
**Severity:** Medium · **Area:** backend/burn · **Evidence:** `burnController.ts` optional `blockchain_tx_hash`
**Impact:** Replay could duplicate redemption records. **Fix direction:** Unique index on hash when present; idempotency. **Acceptance check:** Second submit returns original id.

### B-075 — On-ramp swap race: duplicate processing
**Severity:** High · **Area:** backend/jobs · **Evidence:** `usdcConvertAndMintJob.ts` status transitions
**Impact:** Two workers might process same swap if status check non-atomic. **Fix direction:** Use conditional update `where status=pending` → processing. **Acceptance check:** Concurrency test single winner.

