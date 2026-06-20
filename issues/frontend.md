# Frontend — MVP issues backlog

Canonical UI/app backlog for `acbu-frontend`. Older list: [../PROJECT/issues/FRONTEND_ISSUES.md](../PROJECT/issues/FRONTEND_ISSUES.md) and repo root `ISSUES.md` (if present)—verify which items remain open.

**Issue format:** severity, area, evidence, impact, fix hint, acceptance.

---

### F-001 — Next.js production build ignores TypeScript errors
**Severity:** Critical · **Area:** frontend/build · **Evidence:** `acbu-frontend/next.config.mjs` (`typescript.ignoreBuildErrors`)
**Impact:** Type errors ship to prod → runtime crashes and security holes. **Fix direction:** Set `ignoreBuildErrors: false`; fix surfaced errors; add `typecheck` CI. **Acceptance check:** `next build` fails on any TS error in CI.

### F-002 — API key stored in sessionStorage
**Severity:** Critical · **Area:** frontend/auth · **Evidence:** `acbu-frontend/contexts/auth-context.tsx`
**Impact:** XSS exfiltration steals long-lived API keys. **Fix direction:** Move to httpOnly cookie session or short-lived JWT + refresh; CSP strict. **Acceptance check:** No secrets in `sessionStorage` in prod build.

### F-003 — Passcode stored in sessionStorage for wallet decrypt path
**Severity:** Critical · **Area:** frontend/wallet · **Evidence:** `acbu-frontend/lib/wallet-storage.ts` (`KEY_STORE_PASSPHRASE` via sessionStorage in `getWalletSecretAnyLocal`)
**Impact:** Passcode + encrypted secret becomes trivially stealable together under XSS. **Fix direction:** Use WebCrypto + non-extractable keys; never store passcode in session storage. **Acceptance check:** Security review: passcode only in memory during operation.

### F-004 — Wallet secret 'encryption' is base64 obfuscation
**Severity:** Critical · **Area:** frontend/wallet · **Evidence:** `acbu-frontend/lib/wallet-storage.ts` (`encryptSecret`)
**Impact:** Stellar secret key is not confidential at rest. **Fix direction:** Implement AES-GCM with PBKDF2/Argon2 derived keys; hardware wallet path for prod. **Acceptance check:** Pen test cannot recover secret from IndexedDB dump without passcode.

### F-005 — Plaintext wallet secret storage path exists
**Severity:** Critical · **Area:** frontend/wallet · **Evidence:** `acbu-frontend/lib/wallet-storage.ts` (`storeWalletSecretLocalPlaintext`)
**Impact:** Dev helper enables catastrophic prod misconfiguration. **Fix direction:** Strip from prod bundles behind `NODE_ENV===development` only; runtime assert. **Acceptance check:** Production build throws if plaintext store called.

### F-006 — 2FA challenge token in URL query
**Severity:** High · **Area:** frontend/auth · **Evidence:** `acbu-frontend/app/auth/2fa/page.tsx` (per root `ISSUES.md`)
**Impact:** Leak via Referer/history/logs. **Fix direction:** POST body one-time token or encrypted state cookie. **Acceptance check:** Token never appears in URL bar or server access logs.

### F-007 — Dual Authorization + x-api-key headers
**Severity:** Medium · **Area:** frontend/api · **Evidence:** `acbu-frontend/lib/api/client.ts`
**Impact:** Redundant secret channels increase accidental logging by proxies. **Fix direction:** Send only one header per backend contract; document. **Acceptance check:** Proxy logs never contain duplicate secret lines.

### F-008 — Client-side global fetch timeout without abort sharing
**Severity:** Low · **Area:** frontend/api · **Evidence:** `acbu-frontend/lib/api/client.ts` (`AbortController` per call)
**Impact:** Long hung requests waste battery; duplicated controllers if callers pass signal incorrectly. **Fix direction:** Document pattern; optional default timeout env. **Acceptance check:** Hung request cancels at configured deadline.

### F-009 — CSRF cookie read without backend pairing guarantee
**Severity:** Medium · **Area:** frontend/security · **Evidence:** `acbu-frontend/lib/api/client.ts` (`getCsrfToken`)
**Impact:** If backend never sets `XSRF-TOKEN`, comment in root issues is misleading. **Fix direction:** Align with backend double-submit cookie; else remove dead code. **Acceptance check:** Cross-site POST cannot succeed against prod API.

### F-010 — Duplicate Stellar SDK dependencies
**Severity:** High · **Area:** frontend/deps · **Evidence:** `acbu-frontend/package.json` (`@stellar/stellar-sdk` + `stellar-sdk`)
**Impact:** Two major versions inflate bundle and cause subtle type/runtime mismatches. **Fix direction:** Remove legacy `stellar-sdk`; migrate imports. **Acceptance check:** Only one stellar sdk in lockfile.

### F-011 — Package name still `my-v0-project`
**Severity:** Low · **Area:** frontend/devx · **Evidence:** `acbu-frontend/package.json` (`name`)
**Impact:** Confusing telemetry, npm metadata, support tickets. **Fix direction:** Rename to `acbu-frontend`. **Acceptance check:** Published docs reference correct package name.

### F-012 — No `test` / `typecheck` scripts in frontend
**Severity:** Medium · **Area:** frontend/qa · **Evidence:** `acbu-frontend/package.json`
**Impact:** No default CI quality gate beyond lint (if run). **Fix direction:** Add `pnpm typecheck` + Playwright smoke + component tests for money forms. **Acceptance check:** CI runs typecheck + smoke on PR.

### F-013 — Both `package-lock.json` and `pnpm-lock.yaml`
**Severity:** Medium · **Area:** frontend/devx · **Evidence:** `acbu-frontend/` lockfiles
**Impact:** Dependency drift between contributors. **Fix direction:** Pick one package manager; delete the other lockfile. **Acceptance check:** Single lockfile policy enforced in CI.

### F-014 — Send success dialog clears amount before dialog
**Severity:** High · **Area:** frontend/ux-money · **Evidence:** `acbu-frontend/app/(app)/send/page.tsx` (per `ISSUES.md`)
**Impact:** Users cannot confirm how much was sent. **Fix direction:** Keep amount in state until dialog dismissed. **Acceptance check:** Visual regression or unit test asserts non-empty amount.

### F-015 — Savings deposit handler is console-only
**Severity:** High · **Area:** frontend/feature · **Evidence:** `acbu-frontend/app/(app)/savings/page.tsx`
**Impact:** Core savings MVP non-functional. **Fix direction:** Wire to `POST /savings/...` endpoints; handle errors. **Acceptance check:** Deposit creates pending/completed UI state from API.

### F-016 — Savings New Goal button inert
**Severity:** Medium · **Area:** frontend/feature · **Evidence:** `acbu-frontend/app/(app)/savings/page.tsx`
**Impact:** Broken CTA erodes trust. **Fix direction:** Implement goal model or hide until ready. **Acceptance check:** Button either works or is absent.

### F-017 — SME CTAs inert
**Severity:** Medium · **Area:** frontend/feature · **Evidence:** `acbu-frontend/app/(app)/sme/page.tsx`
**Impact:** Lead gen dead end. **Fix direction:** Hook to application API or remove. **Acceptance check:** Click leads to real flow or mailto with tracking.

### F-018 — Lending application console-only
**Severity:** High · **Area:** frontend/feature · **Evidence:** `acbu-frontend/app/(app)/lending/page.tsx`
**Impact:** No loan intake for MVP. **Fix direction:** Create lending application API + form. **Acceptance check:** Submitted application visible in admin/backoffice stub.

### F-019 — Currency page simulates operations
**Severity:** High · **Area:** frontend/feature · **Evidence:** `acbu-frontend/app/(app)/currency/page.tsx`
**Impact:** Users think trades executed. **Fix direction:** Replace timeouts with real API calls + toasts. **Acceptance check:** Network tab shows POST to backend on confirm.

### F-020 — Bills payment console-only
**Severity:** High · **Area:** frontend/feature · **Evidence:** `acbu-frontend/app/(app)/bills/page.tsx`
**Impact:** Bill pay is fake. **Fix direction:** Integrate bills endpoints when ready; else gate feature flag off. **Acceptance check:** Feature flag off shows honest copy.

### F-021 — Wallet confirm disabled / incomplete
**Severity:** High · **Area:** frontend/wallet · **Evidence:** `acbu-frontend/app/me/settings/wallet/page.tsx` (per `ISSUES.md`)
**Impact:** Users cannot finish wallet setup. **Fix direction:** Complete confirm flow + backend sync. **Acceptance check:** E2E completes wallet activation.

### F-022 — Security settings placeholder
**Severity:** High · **Area:** frontend/settings · **Evidence:** `acbu-frontend/app/me/settings/security/page.tsx`
**Impact:** 2FA management missing for financial app. **Fix direction:** Wire rotate device, sessions list, API keys. **Acceptance check:** User can disable compromised sessions.

### F-023 — Mint page burn tab fake success
**Severity:** High · **Area:** frontend/mint · **Evidence:** `acbu-frontend/app/(app)/mint/page.tsx`
**Impact:** False confirmation risk. **Fix direction:** Call burn API or deep-link with params. **Acceptance check:** Burn tab hits network or routes to `/burn` with prefilled state.

### F-024 — AFK vs ACBU naming drift
**Severity:** Medium · **Area:** frontend/copy · **Evidence:** Multiple pages (`mint`, `send`, `home`)
**Impact:** Confusing brand + reconciliation errors. **Fix direction:** Single glossary constant exported to UI. **Acceptance check:** Grep shows no `AFK` in user-visible prod strings (if policy is ACBU-only).

### F-025 — Balances shown as em dash placeholders
**Severity:** High · **Area:** frontend/data · **Evidence:** Home/mint/send/savings pages
**Impact:** Users cannot trust balances for decisions. **Fix direction:** Use `/users/me/balance` (or equivalent) everywhere with skeletons. **Acceptance check:** No long-lived `—` placeholders in signed-in dashboard.

### F-026 — Savings page mixes mock cards with API data
**Severity:** Medium · **Area:** frontend/data · **Evidence:** `acbu-frontend/app/(app)/savings/page.tsx`
**Impact:** Totals disagree with chain truth. **Fix direction:** Remove mock constants; derive from API only. **Acceptance check:** Totals equal sum(positions).

### F-027 — Savings withdraw recipient editable
**Severity:** High · **Area:** frontend/ux-risk · **Evidence:** Withdraw flow (per `ISSUES.md`)
**Impact:** User can fat-finger funds to wrong ID. **Fix direction:** Lock recipient to resolved user; explicit 'change' flow. **Acceptance check:** Cannot submit withdraw to different ID without extra confirmation.

### F-028 — Hardcoded fees on mint UI
**Severity:** Medium · **Area:** frontend/mint · **Evidence:** `acbu-frontend/app/(app)/mint/page.tsx`
**Impact:** Misleading fee → compliance complaints. **Fix direction:** Fetch fee schedule from backend. **Acceptance check:** Fee matches backend quote within rounding.

### F-029 — Me page KYC badge static
**Severity:** Medium · **Area:** frontend/me · **Evidence:** `acbu-frontend/app/(app)/me/page.tsx`
**Impact:** Users misjudge what they can do. **Fix direction:** Bind badge to KYC status endpoint. **Acceptance check:** Badge updates after KYC webhook simulation.

### F-030 — Me page hardcoded stats
**Severity:** Medium · **Area:** frontend/me · **Evidence:** `acbu-frontend/app/(app)/me/page.tsx`
**Impact:** Trust-breaking mock wealth numbers. **Fix direction:** Replace with API-driven aggregates. **Acceptance check:** Stats match backend for seeded user.

### F-031 — Business page hardcoded stats
**Severity:** Low · **Area:** frontend/business · **Evidence:** `acbu-frontend/app/(app)/business/page.tsx`
**Impact:** Same trust issue for SMB story. **Fix direction:** Wire SME metrics API or mark demo mode clearly. **Acceptance check:** Demo mode banner when using mock numbers.

### F-032 — International/Gateway stubs
**Severity:** Medium · **Area:** frontend/nav · **Evidence:** `international` + `gateway` pages
**Impact:** Dead nav items frustrate users. **Fix direction:** Hide behind feature flags until backend ready. **Acceptance check:** Nav only lists shipped features.

### F-033 — Help page too thin
**Severity:** Low · **Area:** frontend/support · **Evidence:** `acbu-frontend/app/me/help/page.tsx`
**Impact:** Support load spikes. **Fix direction:** FAQ + status page links + ticket form. **Acceptance check:** Users can self-serve top 10 questions.

### F-034 — Sign-in label says Username only
**Severity:** Low · **Area:** frontend/auth · **Evidence:** `acbu-frontend/app/auth/signin/page.tsx`
**Impact:** Users try wrong identifier format. **Fix direction:** Copy: username/email/phone per backend. **Acceptance check:** Support tickets about login drop in UAT.

### F-035 — Signup success query param ignored
**Severity:** Low · **Area:** frontend/auth · **Evidence:** `acbu-frontend/app/auth/signin/page.tsx`
**Impact:** Missed positive reinforcement. **Fix direction:** Toast/banner on `?created=1`. **Acceptance check:** New user sees confirmation once.

### F-036 — Send note omitted from confirm + API
**Severity:** Medium · **Area:** frontend/transfers · **Evidence:** `acbu-frontend/app/(app)/send/page.tsx`
**Impact:** Compliance reference lost for transfers. **Fix direction:** Plumb `note` to API + receipts. **Acceptance check:** PDF/email receipt shows note.

### F-037 — Inconsistent API error UX across pages
**Severity:** Medium · **Area:** frontend/quality · **Evidence:** Many `app/(app)/**` pages
**Impact:** Users see generic failures on money errors. **Fix direction:** Shared `useApiError` hook mirroring burn page. **Acceptance check:** All money pages map 429/503/402 consistently.

### F-038 — Silent catch on send loads
**Severity:** Medium · **Area:** frontend/reliability · **Evidence:** `send/page.tsx` `.catch(() => {})` patterns (per `ISSUES.md`)
**Impact:** Broken lists look empty vs errored. **Fix direction:** Surface toast + retry. **Acceptance check:** Failed load shows error state with retry.

### F-039 — No React error boundary
**Severity:** Medium · **Area:** frontend/reliability · **Evidence:** `acbu-frontend/app/layout.tsx`
**Impact:** Single component error whitescreens app. **Fix direction:** Add route-level error boundaries + reporting. **Acceptance check:** Forced error shows fallback UI not blank page.

### F-040 — useEffect dependency hazards (send/contacts/guardians/kyc)
**Severity:** Medium · **Area:** frontend/react · **Evidence:** Multiple pages (per `ISSUES.md`)
**Impact:** Stale data or infinite loops when deps fixed naively. **Fix direction:** Refactor to `useCallback` + exhaustive-deps lint as error. **Acceptance check:** ESLint react-hooks passes on money routes.

### F-041 — Savings deposit URI length filter too strict
**Severity:** Low · **Area:** frontend/savings · **Evidence:** `acbu-frontend/app/(app)/savings/deposit/page.tsx`
**Impact:** Blocks non-Stellar IDs incorrectly. **Fix direction:** Align validation with backend recipient resolver. **Acceptance check:** Phone-based IDs work where supported.

### F-042 — `as any` tabs in bills/currency
**Severity:** Low · **Area:** frontend/types · **Evidence:** `bills/page.tsx`, `currency/page.tsx`
**Impact:** Weak typing hides invalid tab states. **Fix direction:** Discriminated unions for tab state. **Acceptance check:** TS strict mode clean for those files.

### F-043 — console.log in money handlers
**Severity:** Low · **Area:** frontend/hygiene · **Evidence:** Savings/bills/lending/currency pages
**Impact:** Leaks PII to browser consoles in prod. **Fix direction:** Replace with structured logger behind `DEBUG` flag. **Acceptance check:** Prod bundle has no `console.log` in transfer paths.

### F-044 — Unstable list keys (rates/mint)
**Severity:** Low · **Area:** frontend/react · **Evidence:** `rates/page.tsx`, `mint/page.tsx`
**Impact:** UI state bugs on reorder. **Fix direction:** Use stable ids from API. **Acceptance check:** No `key={index}` for dynamic lists.

### F-045 — No i18n / localization
**Severity:** Low · **Area:** frontend/product · **Evidence:** Whole app
**Impact:** Africa MVP needs multi-locale formatting. **Fix direction:** Adopt next-intl or similar; extract strings. **Acceptance check:** At least NG/KE locale number formats supported.

### F-046 — Accessibility: labels without htmlFor
**Severity:** Medium · **Area:** frontend/a11y · **Evidence:** Mint/send/burn/withdraw forms
**Impact:** Screen reader users cannot operate forms reliably. **Fix direction:** Pair labels+ids; run axe in CI. **Acceptance check:** axe-core critical violations = 0 on money forms.

### F-047 — Mobile nav icon-only links lack aria-label
**Severity:** Medium · **Area:** frontend/a11y · **Evidence:** `acbu-frontend/components/mobile-nav.tsx`
**Impact:** Navigation unusable via VoiceOver. **Fix direction:** Add concise `aria-label` per route. **Acceptance check:** VoiceOver reads destination per icon.

### F-048 — Dark mode badge colors hardcoded light palette
**Severity:** Low · **Area:** frontend/theme · **Evidence:** `send/page.tsx` status badges (per `ISSUES.md`)
**Impact:** Poor contrast in dark theme. **Fix direction:** Use semantic tokens from design system. **Acceptance check:** WCAG AA contrast on badges in dark mode.

### F-049 — Back navigation pattern inconsistent
**Severity:** Low · **Area:** frontend/ux · **Evidence:** Multiple pages mix `router.back` vs `Link`
**Impact:** Users land on unexpected off-app pages. **Fix direction:** Prefer explicit `Link` targets within app shell. **Acceptance check:** Back never exits app shell unintentionally.

### F-050 — Burn form lacks strong validation
**Severity:** Medium · **Area:** frontend/forms · **Evidence:** `acbu-frontend/app/(app)/burn/page.tsx`
**Impact:** Bad bank details fail late or confuse users. **Fix direction:** Client-side format hints + server-driven validation messages. **Acceptance check:** Invalid bank codes blocked before submit.

### F-051 — Signup passcode minimum too weak
**Severity:** High · **Area:** frontend/auth · **Evidence:** `acbu-frontend/app/auth/signup/page.tsx` (per `ISSUES.md`)
**Impact:** 4-digit passcode easy to brute locally. **Fix direction:** Align with backend policy; enforce entropy meter. **Acceptance check:** Cannot sign up with weak passcode in prod config.

### F-052 — Send amount missing min attribute
**Severity:** Low · **Area:** frontend/forms · **Evidence:** `send/page.tsx`
**Impact:** Negative amounts via keyboard. **Fix direction:** InputMode decimal + min + backend validation mirror. **Acceptance check:** Negative amounts impossible in UI.

### F-053 — Activity links wrong route target
**Severity:** Low · **Area:** frontend/nav · **Evidence:** `activity/page.tsx` (per `ISSUES.md`)
**Impact:** Users open wrong detail screen. **Fix direction:** Link to canonical transaction detail route. **Acceptance check:** Deep link opens correct transfer id.

### F-054 — Me page links 2FA to auth flow not settings
**Severity:** Low · **Area:** frontend/nav · **Evidence:** `me/page.tsx`
**Impact:** Confusing security UX loop. **Fix direction:** Point to settings-first flow. **Acceptance check:** User manages 2FA without re-auth loop.

### F-055 — Nested interactive elements (Link in Button)
**Severity:** Medium · **Area:** frontend/html · **Evidence:** `send/page.tsx` Add Contact (per `ISSUES.md`)
**Impact:** Invalid HTML confuses assistive tech + click targets. **Fix direction:** Use `Button asChild` + `Link` or style `Link` as button. **Acceptance check:** HTML validator clean for that control.

### F-056 — Amount formatting inconsistent
**Severity:** Low · **Area:** frontend/formatting · **Evidence:** Home/transfers lists
**Impact:** Users misread decimals especially for 7dp assets. **Fix direction:** Central `formatAcbu` helper with locale. **Acceptance check:** All lists use same formatter.

### F-057 — Reserves page lacks unit explanations
**Severity:** Low · **Area:** frontend/education · **Evidence:** `reserves/page.tsx` (per `ISSUES.md`)
**Impact:** Users misinterpret health metrics. **Fix direction:** Tooltips linking to docs `RESERVE_MANAGEMENT`. **Acceptance check:** User testing shows improved comprehension.

### F-058 — Rates page number formatting
**Severity:** Low · **Area:** frontend/formatting · **Evidence:** `rates/page.tsx`
**Impact:** Rates as raw strings hard to scan. **Fix direction:** Format with significant digits policy. **Acceptance check:** Rates match spec precision.

### F-059 — Burn page currency display locale
**Severity:** Low · **Area:** frontend/formatting · **Evidence:** `burn/page.tsx`
**Impact:** NGN formatting not locale-aware. **Fix direction:** Use Intl.NumberFormat. **Acceptance check:** Locale switch changes grouping separators.

### F-060 — Performance: send page callbacks not memoized
**Severity:** Low · **Area:** frontend/perf · **Evidence:** `send/page.tsx`
**Impact:** Extra rerenders on large contact lists. **Fix direction:** Memoize handlers + virtualize list. **Acceptance check:** React profiler shows acceptable commit times.

### F-061 — Clipboard API failure on HTTP
**Severity:** Low · **Area:** frontend/compat · **Evidence:** `receive/page.tsx` (per `ISSUES.md`)
**Impact:** Copy address fails silently on non-secure origins. **Fix direction:** Detect + fallback prompt. **Acceptance check:** Copy works on HTTPS and shows message on HTTP.

### F-062 — Stellar burning client uses `as any` on send response
**Severity:** Medium · **Area:** frontend/stellar · **Evidence:** `acbu-frontend/lib/stellar/burning.ts`
**Impact:** Missed error branches hide failed txs. **Fix direction:** Type Soroban response properly; surface `errorResultXdr`. **Acceptance check:** Failed tx always shows human readable Soroban error.

### F-063 — Environment base URL misconfiguration footgun
**Severity:** High · **Area:** frontend/config · **Evidence:** `acbu-frontend/lib/api/client.ts` error message
**Impact:** POSTs hit Next.js and return 405 confusingly. **Fix direction:** Add startup runtime check in layout for missing `NEXT_PUBLIC_API_BASE_URL`. **Acceptance check:** Dev console shows loud misconfig banner if unset.

### F-064 — No Content-Security-Policy for Next app
**Severity:** Medium · **Area:** frontend/security · **Evidence:** `next.config.mjs` headers (missing)
**Impact:** XSS impact amplified when keys live in sessionStorage. **Fix direction:** Add strict CSP nonces via Next middleware. **Acceptance check:** CSP report-only phase then enforce.

### F-065 — No Subresource Integrity for third-party scripts (if any)
**Severity:** Low · **Area:** frontend/security · **Evidence:** `app/layout` / analytics
**Impact:** Supply-chain risk if external scripts added. **Fix direction:** Prefer first-party hosting; SRI for CDNs. **Acceptance check:** Security review checklist item signed off.

### F-066 — Inconsistent loading / empty state skeletons across pages
**Severity:** Medium · **Area:** frontend/components · **Evidence:** Per-page `loading.tsx` / `empty.tsx` wrappers across `app/(app)/**`
**Impact:** Loading and empty states are inconsistent across pages (mix of spinners, dimmed text, blank space). Cognitive load + a maintenance burden, plus a11y regressions (no shared `aria-busy` semantics). **Fix direction:** Introduce a single `<Skeleton>` / `<EmptyState>` shared primitive in `components/ui/`; replace ad-hoc loading/empty wrappers across `app/(app)/**` with this primitive. **Acceptance check:** All `app/(app)/**/loading.tsx` files import the shared `<Skeleton>`; design QA spot-check shows visual consistency across send/home/mint/savings/currency; axe-core reports no missing busy-state violations.

### F-071 — Toast removal delay is ~17 minutes
**Severity:** Medium · **Area:** frontend/ux · **Evidence:** `acbu-frontend/lib/toast.ts` constant `TOAST_REMOVE_DELAY = 1000000`
**Impact:** Default toast auto-dismiss is ~17 minutes. Stale toasts pollute the toast log and may display transaction context that should have been cleared (privacy / shoulder-surfer risk in a financial app; trust issue for users who don’t understand why old toasts persist). **Fix direction:** Use a sane default dismiss window (e.g. 5–10 s); gate “sticky” toasts behind an explicit `sticky: true` option for in-progress actions. **Acceptance check:** Default toast auto-dismiss ≤ 10 s under default config; sticky / progress toasts continue to work for genuinely long-running operations (e.g. async Stellar deposit confirmation).

### F-076 — Frontend `request()` helper has no default timeout
**Severity:** High · **Area:** frontend/api · **Evidence:** `acbu-frontend/lib/api/client.ts` `request()` (uses raw `fetch`, no `AbortController` / timeout)
**Impact:** Hung requests spin forever; users see indefinite spinners; tabs accumulate fetch pile-up; mobile users hit memory pressure; retries amplify the issue; users may force-close during pending operations, losing state. **Fix direction:** Wrap `request()` with `AbortController` + configurable default timeout (e.g. 30 s); honor caller-supplied `signal` for cancellation; type timeouts as a distinct `RequestTimeoutError`. **Acceptance check:** Network-throttled test surfaces typed timeout error to UI with retry; integration test asserts `request()` aborts after configured deadline even if backend is unreachable.

