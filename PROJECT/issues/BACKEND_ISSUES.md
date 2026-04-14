# Backend – Known Issues

Issues found in the Node/Express API, services, and jobs. Add new items below as numbered list entries.

---

## Critical

1. **API key validation broken** – src/middleware/auth.ts: `validateApiKey` uses `keyHash: await hashApiKey(apiKey)` for lookup. `hashApiKey` calls `bcrypt.hash()`, which produces a different hash each time (random salt). Stored hashes will never match. Use `bcrypt.compare(plaintextApiKey, storedKeyHash)` instead of hashing the input for lookup.
2. **Wrong import in usdcConvertAndMintJob** – src/jobs/usdcConvertAndMintJob.ts: Imports `db` from `../config/database`, but that module exports `prisma`. The code uses `prisma`, which is never imported, causing a `ReferenceError` at runtime.
3. **IDOR / auth bypass in savings controller** – src/controllers/savingsController.ts: `user` is taken from `req.body` or `req.query` without checking it against the authenticated user. Any authenticated user can operate on another user's savings by supplying a different `user` ID.
4. **Env validation too late** – src/config/env.ts: `requiredEnvVars` is checked after `config` is built. If `DATABASE_URL` is missing, the app may still start and fail later. Validation should run before any use of `config`.

## High

5. **Infinite retries for failed webhooks** – src/services/webhook/webhookService.ts: When `status === 'failed'`, `deliverWebhook` still attempts delivery and returns `false`, so the consumer nacks and requeues indefinitely. Add early return for failed status.
6. **Organization users never notified for investment withdrawal** – src/jobs/investmentWithdrawalJob.ts: `publishInvestmentWithdrawalReady` only fires when `r.userId` is set. For organization requests (`r.organizationId` set, `r.userId` null), no notification is sent.
7. **Burn limits bypass when `audience` is unset** – src/controllers/burnController.ts: `checkWithdrawalLimits` and `isCurrencyWithdrawalPaused` run only when `req.audience` is set. `/burn/acbu` via `burnRoutes` does not set `audience`, so limits and circuit breakers are skipped.
8. **Mint limits bypass when `audience` is unset** – src/controllers/mintController.ts: `isMintingPaused` and `checkDepositLimits` run only when `req.audience` is set. `/mint/usdc` and `/mint/deposit` via `mintRoutes` do not set `audience`, so limits are bypassed.
9. **Fee thresholds likely wrong** – src/services/feePolicy/feePolicyService.ts: `getBurnFeeBps` uses `pctOfTarget < 15` and `pctOfTarget > 21`. With `pctOfTarget = (actualWeight / targetWeight) * 100`, these look like typos (should be 85/115). Logic is unlikely to behave as intended.
10. **Decimal vs string for `acbuAmount` in transfer** – src/services/transfer/transferService.ts: `acbuAmount` is passed as a string to `prisma.transaction.create`. Prisma expects `Decimal` for that field; this can cause type or precision issues.
11. **No dead-letter queues** – src/config/rabbitmq.ts: Queues are declared without `deadLetterExchange` or `deadLetterRoutingKey`. Messages that repeatedly fail are requeued forever instead of being moved to a DLQ.
12. **No max retry limit for webhook consumer** – src/jobs/webhookConsumer.ts: Failed webhooks are nacked with requeue indefinitely. There is no per-message retry cap.

## Medium

13. **Wallet activation uses XLM, not Pi** – src/services/wallet/walletActivationService.ts: Sends XLM to activate Stellar accounts; comment says "using pi instead of xlm" is desired. Switch to Pi when the Pi bridge/chain is in use.
14. **Remove placeholder Stellar address in auth** – src/services/auth/authService.ts: Uses `getPlaceholderStellarAddress()` on signup to satisfy schema before the real wallet exists. Replace with a proper flow.
15. **Transfer service: verify alias-based flow** – Transfer supports sending by username/phone (via recipientResolver); confirm the current resolve-to-Stellar flow is correct, secure, and documented.
16. **Contract client and Stellar fees hardcoded** – src/services/stellar/contractClient.ts defaults fee to `'100'`; walletActivationService.ts and transferService.ts also use hardcoded `'100'`. Make fees configurable or network-derived.
17. **Multiple fintech providers per nation** – Only Flutterwave is wired in webhookRoutes; add support for multiple providers per country (e.g. Nigeria: Paystack and Flutterwave) to avoid single point of failure.
18. **KYC extraction placeholder** – src/services/kyc/machineLayer.ts: Uses a placeholder for "real impl would call Vision API and redact"; replace with actual implementation.
19. **Basket weight script missing** – Create a backend/tooling script or job that computes and reports current basket weights based on market conditions and factors considered.
20. **Balance endpoint for frontend missing** – Frontend shows balance as placeholder "—" everywhere; add or expose GET /users/me/balance so the UI can display real balances.
21. **No dependency health checks** – /health/metrics does not verify PostgreSQL, MongoDB, or RabbitMQ. A "healthy" response can be returned even when core dependencies are down.
22. **Race condition in API key rate limiting** – src/middleware/rateLimiter.ts: The get-then-set flow for rate limit counters is not atomic. Concurrent requests can both pass the limit before either updates the cache.
23. **ReDoS risk in `deletePattern`** – src/utils/cache.ts: `new RegExp(pattern)` with user-controlled or unsanitized input can lead to ReDoS. Patterns should be validated or escaped.
24. **Webhook verification skipped when secret missing** – src/controllers/webhookController.ts: If `FLUTTERWAVE_WEBHOOK_SECRET` is unset, `verifyFlutterwaveSignature` calls `next()` and skips verification, allowing forged webhooks.
25. **Placeholder Stellar address format invalid** – src/services/auth/authService.ts: `getPlaceholderStellarAddress()` returns `'P' + ...` (56 chars). Stellar addresses start with `G`. This may violate schema or downstream validation.
26. **ACBU supply from DB, not chain** – src/services/reserve/ReserveTracker.ts: `getTotalAcbuSupply` uses transaction aggregates instead of on-chain supply. This can diverge from the real supply if there are off-chain mints/burns.
27. **Org-scoped limits may not apply** – src/services/limits/limitsService.ts: For org-level API keys (`userId` null), transactions created with `userId: null` have no user relation, so they are excluded from limit aggregates.
28. **USDC to XLM conversion is a no-op** – src/jobs/usdcConvertAndMintJob.ts: `convertUsdcToXlm` is empty. USDC is never converted; the job still mints ACBU as if conversion succeeded.
29. **Hardcoded XLM rate** – src/jobs/xlmToAcbuJob.ts: `XLM_USD_RATE` defaults to `0.2` from env. A wrong or stale rate can cause incorrect ACBU mint amounts.
30. **Deposit limits use wrong USD equivalent** – src/controllers/mintController.ts: `amountUsdPlaceholder = amountNum` treats local currency amount as USD for limit checks. Limits should use a proper USD conversion.
31. **Incomplete treasury aggregation** – src/controllers/governmentController.ts: TODO: aggregate from transactions for org/user and reserve exposure. `byCurrency` is always `[]`.
32. **Audit failures are silent** – src/services/audit/auditService.ts: Audit write failures are only logged. There is no retry, alerting, or fallback, so audit data can be lost without visibility.

## Low

33. **`any` cast for permissions** – src/middleware/auth.ts: `permissions: permissions as any` bypasses type safety. Use a proper type.
34. **`any` usage in segmentGuard** – src/middleware/segmentGuard.ts: `(req as any).userTier` and `tier as any` weaken type safety.
35. **`any` usage in Stellar client** – src/services/stellar/client.ts: `submitTransaction(transaction: any)` and `(b: any)` in balance lookups.
36. **`any` cast in burning service** – src/services/contracts/acbuBurning.service.ts: `localAmounts as any[]` bypasses type checking.
37. **No pagination on transfers** – src/controllers/transferController.ts: `getTransfers` uses `take: 50` with no `skip` or cursor. No way to page through older transfers.
38. **No pagination on investment withdrawals** – src/controllers/investmentController.ts: `getInvestmentWithdrawRequests` uses `take: 50` with no pagination parameters.
39. **`limit` query param not validated** – src/controllers/governmentController.ts: `Number(req.query.limit)` can be `NaN`; `Math.max(1, NaN)` is `NaN`.
40. **No input validation in savings controller** – src/controllers/savingsController.ts: `amount`, `term_seconds`, and `user` are not validated with Zod or similar. Invalid values can reach the service layer.
41. **Missing composite index** – prisma/schema.prisma: `OnRampSwap` is often queried by `(status, createdAt)`. A composite index would help those queries.
42. **Webhook JSON parse failure returns empty body** – src/index.ts: If `raw.toString()` is invalid JSON, `body` is set to `{}`. The webhook handler may run with incomplete data instead of returning 400.
43. **Stack traces in error logs** – src/errorHandler.ts: `logger.error('Unexpected error', { error: err, ... })` can log full error objects. Ensure logs are not exposed to clients and that PII is not included.

## Trivial

44. **`(prisma as any).onRampSwap` casts** – src/jobs/usdcConvertAndMintJob.ts, xlmToAcbuJob.ts, mintController.ts, onrampController.ts: Prisma client is cast to `any` to access `onRampSwap`. Run `npx prisma generate` and fix the schema/client so `onRampSwap` is properly typed.
45. **Hardcoded limit values** – src/config/limits.ts: Deposit/withdrawal limits are hardcoded. Consider loading from env or config for easier tuning.
46. **Fallback email placeholder** – src/services/notification/notificationService.ts: `noreply@acbu.example.com` is a placeholder and may not be valid in production.
47. **Yield accounting stub** – src/services/investment/yieldAccountingService.ts: Placeholder only; yield accounting is not implemented.
48. **No automated tests** – No `*.test.ts` or `*.spec.ts` files found. Critical paths (auth, transfers, limits, webhooks) lack automated tests.
49. **Client-controlled mint recipient** – src/controllers/mintController.ts, onrampController.ts: Mint/on-ramp flows take a body-supplied Stellar address (`wallet_address` / `stellar_address`) and pass it to `mintFromUsdcInternal` as the chain recipient with no check that it matches the authenticated user's `User.stellarAddress`. A caller could direct minted ACBU to any address.
50. **Rate limiting fails open when MongoDB cache is down** – src/middleware/rateLimiter.ts, src/utils/cache.ts: `apiKeyRateLimiter` relies on Mongo-backed `cacheService`. On `get`/`set` errors the cache returns `null` or swallows failures, so under Mongo outages per-API-key limits do not accumulate, weakening abuse protection.
51. **Recovery unlock is single-factor** – src/services/recovery/recoveryService.ts: `POST /recovery/unlock` returns a new API key after only identifier + passcode — no OTP, email link, or second step. Weak or stolen passcodes enable account takeover.
52. **`requireMinTier` middleware is never used** – src/middleware/segmentGuard.ts: `requireMinTier` is defined but no route imports or applies it. The `User.tier` field is therefore not enforced at the middleware layer.
53. **`InvestmentWithdrawalRequest.organizationId` has no FK** – prisma/schema.prisma: `organizationId` is optional `String?` with no `Organization` relation or `@relation`. The DB cannot enforce that it references a real organization, enabling orphan rows.
54. **Unauthenticated reserve metrics on `/health/metrics`** – src/routes/index.ts: `GET /health/metrics` is not behind `validateApiKey`. It returns reserve ratio, overcollateralization ratio, and reserve health — operationally sensitive data — without authentication.
55. **Swagger UI exposed without auth** – src/index.ts: `app.use('/api-docs', swaggerUi.serve, …)` is mounted without API-key middleware, so the full OpenAPI surface is available for unauthenticated reconnaissance.
56. **Stellar event listener: broad stream and swallowed errors** – src/services/stellar/eventListener.ts: Listener walks global Horizon `effects()` (not scoped to known contract IDs), which is heavy and error-prone. Per-handler failures are logged and swallowed, so contract-dependent jobs can miss state with no retry or dead-letter.
