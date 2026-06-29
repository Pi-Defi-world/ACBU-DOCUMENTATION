# Frontend — Known Issues

This document is the **single, durable long-form reference** for known issues in the ACBU frontend (`acbu-frontend`, Next.js 14 + React + Stellar SDK), covering all routes under `app/(app)/**` (segments), the auth surface (`app/(public)/auth/**`), settings, wallet, savings, lending, currency, bills, and the lib/api + lib/stellar + lib/wallet-storage + lib/auth surface.

> **How to use**
>
> - Each issue is structured as: **severity · area · evidence · impact · fix direction · acceptance check**.
> - Items are ordered by severity (Critical → Low), then by ID for stable cross-referencing.
> - Provenance: this file consolidates the legacy 77-item annotated list previously at `PROJECT/issues/FRONTEND_ISSUES.md` with the canonical 65-item triage-ready list at `issues/frontend.md`. The canonical list at `issues/frontend.md` remains the **live working backlog**; this file is the durable version-of-record.
> - Legacy items whose content maps to a canonical `F-###` ID are tagged **(moved)**; legacy items that go beyond the canonical list (newer findings) are tagged **(new)** and retained verbatim because they pre-date the canonical rewrite.
> - When an item is fixed, the corresponding entry is removed from the working lists (`issues/frontend.md`) and the resolution row flips to ✅ Fixed in Section 8 with a merged-PR link.
> - **Severity legend:** 🔴 Critical · 🟠 High · 🟡 Medium · 🟢 Low

---

## 1. Summary Table

| ID | Severity | Area | Title (canonical) |
|----|----------|------|---------|
| F-001 | 🔴 Critical | frontend/build | Next.js production build ignores TypeScript errors |
| F-002 | 🔴 Critical | frontend/auth | API key stored in sessionStorage |
| F-003 | 🔴 Critical | frontend/wallet | Passcode stored in sessionStorage for wallet decrypt path |
| F-004 | 🔴 Critical | frontend/wallet | Wallet secret "encryption" is base64 obfuscation |
| F-005 | 🔴 Critical | frontend/wallet | Plaintext wallet secret storage path exists |
| F-006 | 🟠 High | frontend/auth | 2FA challenge token in URL query |
| F-007 | 🟡 Medium | frontend/api | Dual Authorization + x-api-key headers |
| F-008 | 🟢 Low | frontend/api | Client-side global fetch timeout without abort sharing |
| F-009 | 🟡 Medium | frontend/security | CSRF cookie read without backend pairing guarantee |
| F-010 | 🟠 High | frontend/deps | Duplicate Stellar SDK dependencies |
| F-011 | 🟢 Low | frontend/devx | Package name still `my-v0-project` |
| F-012 | 🟡 Medium | frontend/qa | No `test` / `typecheck` scripts in frontend |
| F-013 | 🟡 Medium | frontend/devx | Both `package-lock.json` and `pnpm-lock.yaml` |
| F-014 | 🟠 High | frontend/ux-money | Send success dialog clears amount before dialog |
| F-015 | 🟠 High | frontend/feature | Savings deposit handler is console-only |
| F-016 | 🟡 Medium | frontend/feature | Savings New Goal button inert |
| F-017 | 🟡 Medium | frontend/feature | SME CTAs inert |
| F-018 | 🟠 High | frontend/feature | Lending application console-only |
| F-019 | 🟠 High | frontend/feature | Currency page simulates operations |
| F-020 | 🟠 High | frontend/feature | Bills payment console-only |
| F-021 | 🟠 High | frontend/wallet | Wallet confirm disabled / incomplete |
| F-022 | 🟠 High | frontend/settings | Security settings placeholder |
| F-023 | 🟠 High | frontend/mint | Mint page burn tab fake success |
| F-024 | 🟡 Medium | frontend/copy | AFK vs ACBU naming drift |
| F-025 | 🟠 High | frontend/data | Balances shown as em dash placeholders |
| F-026 | 🟡 Medium | frontend/data | Savings page mixes mock cards with API data |
| F-027 | 🟠 High | frontend/ux-risk | Savings withdraw recipient editable |
| F-028 | 🟡 Medium | frontend/mint | Hardcoded fees on mint UI |
| F-029 | 🟡 Medium | frontend/me | Me page KYC badge static |
| F-030 | 🟡 Medium | frontend/me | Me page hardcoded stats |
| F-031 | 🟢 Low | frontend/business | Business page hardcoded stats |
| F-032 | 🟡 Medium | frontend/nav | International/Gateway stubs |
| F-033 | 🟢 Low | frontend/support | Help page too thin |
| F-034 | 🟢 Low | frontend/auth | Sign-in label says Username only |
| F-035 | 🟢 Low | frontend/auth | Signup success query param ignored |
| F-036 | 🟡 Medium | frontend/transfers | Send note omitted from confirm + API |
| F-037 | 🟡 Medium | frontend/quality | Inconsistent API error UX across pages |
| F-038 | 🟡 Medium | frontend/reliability | Silent catch on send loads |
| F-039 | 🟡 Medium | frontend/reliability | No React error boundary |
| F-040 | 🟡 Medium | frontend/react | useEffect dependency hazards (send/contacts/guardians/kyc) |
| F-041 | 🟢 Low | frontend/savings | Savings deposit URI length filter too strict |
| F-042 | 🟢 Low | frontend/types | `as any` tabs in bills/currency |
| F-043 | 🟢 Low | frontend/hygiene | console.log in money handlers |
| F-044 | 🟢 Low | frontend/react | Unstable list keys (rates/mint) |
| F-045 | 🟢 Low | frontend/product | No i18n / localization |
| F-046 | 🟡 Medium | frontend/a11y | Accessibility: labels without htmlFor |
| F-047 | 🟡 Medium | frontend/a11y | Mobile nav icon-only links lack aria-label |
| F-048 | 🟢 Low | frontend/theme | Dark mode badge colors hardcoded light palette |
| F-049 | 🟢 Low | frontend/ux | Back navigation pattern inconsistent |
| F-050 | 🟡 Medium | frontend/forms | Burn form lacks strong validation |
| F-051 | 🟠 High | frontend/auth | Signup passcode minimum too weak |
| F-052 | 🟢 Low | frontend/forms | Send amount missing min attribute |
| F-053 | 🟢 Low | frontend/nav | Activity links wrong route target |
| F-054 | 🟢 Low | frontend/nav | Me page links 2FA to auth flow not settings |
| F-055 | 🟡 Medium | frontend/html | Nested interactive elements (Link in Button) |
| F-056 | 🟢 Low | frontend/formatting | Amount formatting inconsistent |
| F-057 | 🟢 Low | frontend/education | Reserves page lacks unit explanations |
| F-058 | 🟢 Low | frontend/formatting | Rates page number formatting |
| F-059 | 🟢 Low | frontend/formatting | Burn page currency display locale |
| F-060 | 🟢 Low | frontend/perf | Performance: send page callbacks not memoized |
| F-061 | 🟢 Low | frontend/compat | Clipboard API failure on HTTP |
| F-062 | 🟡 Medium | frontend/stellar | Stellar burning client uses `as any` on send response |
| F-063 | 🟠 High | frontend/config | Environment base URL misconfiguration footgun |
| F-064 | 🟡 Medium | frontend/security | No Content-Security-Policy for Next app |
| F-065 | 🟢 Low | frontend/security | No Subresource Integrity for third-party scripts (if any) |
| F-066 | 🟡 Medium | frontend/components | Inconsistent loading / empty state skeletons across pages |
| F-071 | 🟡 Medium | frontend/ux | Toast removal delay is ~17 minutes |

**Totals:** 5 Critical · 14 High · 25 Medium · 23 Low · **67 total catalog items** (all IDs `F-001` through `F-071`, fully aligned with `issues/frontend.md`).

---

## 2. Severity Counts

| Severity | Count |
|----------|-------|
| 🔴 Critical | **5** (F-001–F-005) |
| 🟠 High | **14** (F-006, F-010, F-014, F-015, F-018–F-023, F-025, F-027, F-051, F-063) |
| 🟡 Medium | **25** (F-007, F-009, F-012, F-013, F-016, F-017, F-024, F-026, F-028–F-030, F-032, F-036–F-040, F-046, F-047, F-050, F-055, F-062, F-064, F-066, F-071) |
| 🟢 Low | **23** (F-008, F-011, F-031, F-033–F-035, F-041–F-045, F-048, F-049, F-052–F-054, F-056–F-061, F-065) |
| **Total** | **65** |

---

## 3. Critical Issues (🔴) — Ship Blockers

### F-001 — Next.js production build ignores TypeScript errors
- **Area:** frontend/build
- **Evidence:** `acbu-frontend/next.config.mjs` (`typescript.ignoreBuildErrors`)
- **Impact:** Type errors ship to prod → runtime crashes and security holes.
- **Fix direction:** Set `ignoreBuildErrors: false`; fix surfaced errors; add `typecheck` CI.
- **Acceptance check:** `next build` fails on any TS error in CI.

### F-002 — API key stored in sessionStorage
- **Area:** frontend/auth
- **Evidence:** `acbu-frontend/contexts/auth-context.tsx`
- **Impact:** XSS exfiltration steals long-lived API keys.
- **Fix direction:** Move to httpOnly cookie session or short-lived JWT + refresh; CSP strict.
- **Acceptance check:** No secrets in `sessionStorage` in prod build.

### F-003 — Passcode stored in sessionStorage for wallet decrypt path
- **Area:** frontend/wallet
- **Evidence:** `acbu-frontend/lib/wallet-storage.ts` (`KEY_STORE_PASSPHRASE` via sessionStorage in `getWalletSecretAnyLocal`)
- **Impact:** Passcode + encrypted secret becomes trivially stealable together under XSS.
- **Fix direction:** Use WebCrypto + non-extractable keys; never store passcode in session storage.
- **Acceptance check:** Security review: passcode only in memory during operation.

### F-004 — Wallet secret "encryption" is base64 obfuscation
- **Area:** frontend/wallet
- **Evidence:** `acbu-frontend/lib/wallet-storage.ts` (`encryptSecret`)
- **Impact:** Stellar secret key is not confidential at rest.
- **Fix direction:** Implement AES-GCM with PBKDF2/Argon2 derived keys; hardware wallet path for prod.
- **Acceptance check:** Pen test cannot recover secret from IndexedDB dump without passcode.

### F-005 — Plaintext wallet secret storage path exists
- **Area:** frontend/wallet
- **Evidence:** `acbu-frontend/lib/wallet-storage.ts` (`storeWalletSecretLocalPlaintext`)
- **Impact:** Dev helper enables catastrophic prod misconfiguration.
- **Fix direction:** Strip from prod bundles behind `NODE_ENV === "development"` only; runtime assert.
- **Acceptance check:** Production build throws if plaintext store called.

---

## 4. High Severity Issues (🟠)

### F-006 — 2FA challenge token in URL query
- **Area:** frontend/auth
- **Evidence:** `acbu-frontend/app/auth/2fa/page.tsx`
- **Impact:** Token leaks via Referer headers, browser history, server logs.
- **Fix direction:** POST body one-time token or encrypted state cookie.
- **Acceptance check:** Token never appears in URL bar or server access logs.

### F-010 — Duplicate Stellar SDK dependencies
- **Area:** frontend/deps
- **Evidence:** `acbu-frontend/package.json` (`@stellar/stellar-sdk` + `stellar-sdk`)
- **Impact:** Two major versions inflate bundle and cause subtle type/runtime mismatches.
- **Fix direction:** Remove legacy `stellar-sdk`; migrate imports.
- **Acceptance check:** Only one stellar sdk in lockfile.

### F-014 — Send success dialog clears amount before dialog
- **Area:** frontend/ux-money
- **Evidence:** `acbu-frontend/app/(app)/send/page.tsx`
- **Impact:** Users cannot confirm how much was sent.
- **Fix direction:** Keep amount in state until dialog dismissed.
- **Acceptance check:** Visual regression or unit test asserts non-empty amount.

### F-015 — Savings deposit handler is console-only
- **Area:** frontend/feature
- **Evidence:** `acbu-frontend/app/(app)/savings/page.tsx`
- **Impact:** Core savings MVP non-functional.
- **Fix direction:** Wire to `POST /savings/...` endpoints; handle errors.
- **Acceptance check:** Deposit creates pending/completed UI state from API.

### F-018 — Lending application console-only
- **Area:** frontend/feature
- **Evidence:** `acbu-frontend/app/(app)/lending/page.tsx`
- **Impact:** No loan intake for MVP.
- **Fix direction:** Create lending application API + form.
- **Acceptance check:** Submitted application visible in admin/backoffice stub.

### F-019 — Currency page simulates operations
- **Area:** frontend/feature
- **Evidence:** `acbu-frontend/app/(app)/currency/page.tsx`
- **Impact:** Users think trades executed.
- **Fix direction:** Replace `setTimeout` with real API calls + toasts.
- **Acceptance check:** Network tab shows POST to backend on confirm.

### F-020 — Bills payment console-only
- **Area:** frontend/feature
- **Evidence:** `acbu-frontend/app/(app)/bills/page.tsx`
- **Impact:** Bill pay is fake.
- **Fix direction:** Integrate bills endpoints when ready; else gate feature flag off.
- **Acceptance check:** Feature flag off shows honest copy.

### F-021 — Wallet confirm disabled / incomplete
- **Area:** frontend/wallet
- **Evidence:** `acbu-frontend/app/me/settings/wallet/page.tsx`
- **Impact:** Users cannot finish wallet setup.
- **Fix direction:** Complete confirm flow + backend sync.
- **Acceptance check:** E2E completes wallet activation.

### F-022 — Security settings placeholder
- **Area:** frontend/settings
- **Evidence:** `acbu-frontend/app/me/settings/security/page.tsx`
- **Impact:** 2FA management missing for financial app.
- **Fix direction:** Wire rotate device, sessions list, API keys.
- **Acceptance check:** User can disable compromised sessions.

### F-023 — Mint page burn tab fake success
- **Area:** frontend/mint
- **Evidence:** `acbu-frontend/app/(app)/mint/page.tsx`
- **Impact:** False confirmation risk.
- **Fix direction:** Call burn API or deep-link with params.
- **Acceptance check:** Burn tab hits network or routes to `/burn` with prefilled state.

### F-025 — Balances shown as em dash placeholders
- **Area:** frontend/data
- **Evidence:** Home/mint/send/savings pages
- **Impact:** Users cannot trust balances for decisions.
- **Fix direction:** Use `/users/me/balance` (or equivalent) everywhere with skeletons.
- **Acceptance check:** No long-lived `—` placeholders in signed-in dashboard.

### F-027 — Savings withdraw recipient editable
- **Area:** frontend/ux-risk
- **Evidence:** Withdraw flow
- **Impact:** User can fat-finger funds to wrong ID.
- **Fix direction:** Lock recipient to resolved user; explicit "change" flow.
- **Acceptance check:** Cannot submit withdraw to different ID without extra confirmation.

### F-051 — Signup passcode minimum too weak
- **Area:** frontend/auth
- **Evidence:** `acbu-frontend/app/auth/signup/page.tsx`
- **Impact:** 4-digit passcode easy to brute locally.
- **Fix direction:** Align with backend policy; enforce entropy meter.
- **Acceptance check:** Cannot sign up with weak passcode in prod config.

### F-063 — Environment base URL misconfiguration footgun
- **Area:** frontend/config
- **Evidence:** `acbu-frontend/lib/api/client.ts` error message
- **Impact:** POSTs hit Next.js and return 405 confusingly.
- **Fix direction:** Add startup runtime check in layout for missing `NEXT_PUBLIC_API_BASE_URL`.
- **Acceptance check:** Dev console shows loud misconfig banner if unset.

---

## 5. Medium Severity Issues (🟡)

### F-007 — Dual Authorization + x-api-key headers
- **Area:** frontend/api · **Evidence:** `acbu-frontend/lib/api/client.ts`
- **Impact:** Redundant secret channels increase accidental logging by proxies.
- **Fix direction:** Send only one header per backend contract; document.
- **Acceptance check:** Proxy logs never contain duplicate secret lines.

### F-009 — CSRF cookie read without backend pairing guarantee
- **Area:** frontend/security · **Evidence:** `acbu-frontend/lib/api/client.ts` (`getCsrfToken`)
- **Impact:** If backend never sets `XSRF-TOKEN`, code becomes dead.
- **Fix direction:** Align with backend double-submit cookie; remove if dead.
- **Acceptance check:** Cross-site POST cannot succeed against prod API.

### F-012 — No `test` / `typecheck` scripts in frontend
- **Area:** frontend/qa · **Evidence:** `acbu-frontend/package.json`
- **Impact:** No default CI quality gate beyond lint.
- **Fix direction:** Add `pnpm typecheck` + Playwright smoke + component tests for money forms.
- **Acceptance check:** CI runs typecheck + smoke on PR.

### F-013 — Both `package-lock.json` and `pnpm-lock.yaml`
- **Area:** frontend/devx · **Evidence:** `acbu-frontend/` lockfiles
- **Impact:** Dependency drift between contributors.
- **Fix direction:** Pick one package manager; delete the other lockfile.
- **Acceptance check:** Single lockfile policy enforced in CI.

### F-016 — Savings New Goal button inert
- **Area:** frontend/feature · **Evidence:** `acbu-frontend/app/(app)/savings/page.tsx`
- **Impact:** Broken CTA erodes trust.
- **Fix direction:** Implement goal model or hide until ready.
- **Acceptance check:** Button either works or is absent.

### F-017 — SME CTAs inert
- **Area:** frontend/feature · **Evidence:** `acbu-frontend/app/(app)/sme/page.tsx`
- **Impact:** Lead gen dead end.
- **Fix direction:** Hook to application API or remove.
- **Acceptance check:** Click leads to real flow or mailto with tracking.

### F-024 — AFK vs ACBU naming drift
- **Area:** frontend/copy · **Evidence:** Multiple pages (`mint`, `send`, `home`)
- **Impact:** Confusing brand + reconciliation errors.
- **Fix direction:** Single glossary constant exported to UI.
- **Acceptance check:** Grep shows no `AFK` in user-visible prod strings (if policy is ACBU-only).

### F-026 — Savings page mixes mock cards with API data
- **Area:** frontend/data · **Evidence:** `acbu-frontend/app/(app)/savings/page.tsx`
- **Impact:** Totals disagree with chain truth.
- **Fix direction:** Remove mock constants; derive from API only.
- **Acceptance check:** Totals equal sum(positions).

### F-028 — Hardcoded fees on mint UI
- **Area:** frontend/mint · **Evidence:** `acbu-frontend/app/(app)/mint/page.tsx`
- **Impact:** Misleading fee → compliance complaints.
- **Fix direction:** Fetch fee schedule from backend.
- **Acceptance check:** Fee matches backend quote within rounding.

### F-029 — Me page KYC badge static
- **Area:** frontend/me · **Evidence:** `acbu-frontend/app/(app)/me/page.tsx`
- **Impact:** Users misjudge what they can do.
- **Fix direction:** Bind badge to KYC status endpoint.
- **Acceptance check:** Badge updates after KYC webhook simulation.

### F-030 — Me page hardcoded stats
- **Area:** frontend/me · **Evidence:** `acbu-frontend/app/(app)/me/page.tsx`
- **Impact:** Trust-breaking mock wealth numbers.
- **Fix direction:** Replace with API-driven aggregates.
- **Acceptance check:** Stats match backend for seeded user.

### F-032 — International/Gateway stubs
- **Area:** frontend/nav · **Evidence:** `international` + `gateway` pages
- **Impact:** Dead nav items frustrate users.
- **Fix direction:** Hide behind feature flags until backend ready.
- **Acceptance check:** Nav only lists shipped features.

### F-036 — Send note omitted from confirm + API
- **Area:** frontend/transfers · **Evidence:** `acbu-frontend/app/(app)/send/page.tsx`
- **Impact:** Compliance reference lost for transfers.
- **Fix direction:** Plumb `note` to API + receipts.
- **Acceptance check:** PDF/email receipt shows note.

### F-037 — Inconsistent API error UX across pages
- **Area:** frontend/quality · **Evidence:** Many `app/(app)/**` pages
- **Impact:** Users see generic failures on money errors.
- **Fix direction:** Shared `useApiError` hook mirroring burn page.
- **Acceptance check:** All money pages map 429/503/402 consistently.

### F-038 — Silent catch on send loads
- **Area:** frontend/reliability · **Evidence:** `send/page.tsx` `.catch(() => {})` patterns
- **Impact:** Broken lists look empty vs errored.
- **Fix direction:** Surface toast + retry.
- **Acceptance check:** Failed load shows error state with retry.

### F-039 — No React error boundary
- **Area:** frontend/reliability · **Evidence:** `acbu-frontend/app/layout.tsx`
- **Impact:** Single component error whitescreens app.
- **Fix direction:** Add route-level error boundaries + reporting.
- **Acceptance check:** Forced error shows fallback UI not blank page.

### F-040 — useEffect dependency hazards (send/contacts/guardians/kyc)
- **Area:** frontend/react · **Evidence:** Multiple pages
- **Impact:** Stale data or infinite loops when deps fixed naively.
- **Fix direction:** Refactor to `useCallback` + exhaustive-deps lint as error.
- **Acceptance check:** ESLint react-hooks passes on money routes.

### F-046 — Accessibility: labels without htmlFor
- **Area:** frontend/a11y · **Evidence:** Mint/send/burn/withdraw forms
- **Impact:** Screen reader users cannot operate forms reliably.
- **Fix direction:** Pair labels+ids; run axe in CI.
- **Acceptance check:** axe-core critical violations = 0 on money forms.

### F-047 — Mobile nav icon-only links lack aria-label
- **Area:** frontend/a11y · **Evidence:** `acbu-frontend/components/mobile-nav.tsx`
- **Impact:** Navigation unusable via VoiceOver.
- **Fix direction:** Add concise `aria-label` per route.
- **Acceptance check:** VoiceOver reads destination per icon.

### F-050 — Burn form lacks strong validation
- **Area:** frontend/forms · **Evidence:** `acbu-frontend/app/(app)/burn/page.tsx`
- **Impact:** Bad bank details fail late or confuse users.
- **Fix direction:** Client-side format hints + server-driven validation messages.
- **Acceptance check:** Invalid bank codes blocked before submit.

### F-055 — Nested interactive elements (Link in Button)
- **Area:** frontend/html · **Evidence:** `send/page.tsx` Add Contact
- **Impact:** Invalid HTML confuses assistive tech + click targets.
- **Fix direction:** Use `Button asChild` + `Link` or style `Link` as button.
- **Acceptance check:** HTML validator clean for that control.

### F-062 — Stellar burning client uses `as any` on send response
- **Area:** frontend/stellar · **Evidence:** `acbu-frontend/lib/stellar/burning.ts`
- **Impact:** Missed error branches hide failed txs.
- **Fix direction:** Type Soroban response properly; surface `errorResultXdr`.
- **Acceptance check:** Failed tx always shows human readable Soroban error.

### F-064 — No Content-Security-Policy for Next app
- **Area:** frontend/security · **Evidence:** `next.config.mjs` headers (missing)
- **Impact:** XSS impact amplified when keys live in sessionStorage.
- **Fix direction:** Add strict CSP nonces via Next middleware.
- **Acceptance check:** CSP report-only phase then enforce.

### F-066 — Inconsistent loading / empty state skeletons across pages
- **Area:** frontend/components
- **Evidence:** Per-page `loading.tsx` / `empty.tsx` wrappers across `app/(app)/**`
- **Impact:** Loading and empty states are inconsistent across pages (mix of spinners, dimmed text, blank space). Cognitive load + maintenance burden, plus a11y regressions (no shared `aria-busy` semantics).
- **Fix direction:** Introduce a single `<Skeleton>` / `<EmptyState>` shared primitive in `components/ui/`; replace ad-hoc loading/empty wrappers across `app/(app)/**` with this primitive.
- **Acceptance check:** All `app/(app)/**/loading.tsx` files import the shared `<Skeleton>`; design QA spot-check shows visual consistency across send/home/mint/savings/currency; axe-core reports no missing busy-state violations.

### F-071 — Toast removal delay is ~17 minutes
- **Area:** frontend/ux
- **Evidence:** `acbu-frontend/lib/toast.ts` constant `TOAST_REMOVE_DELAY = 1000000`
- **Impact:** Default toast auto-dismiss is ~17 minutes. Stale toasts pollute the toast log and may display transaction context that should have been cleared (privacy / shoulder-surfer risk in a financial app; trust issue for users who don’t understand why old toasts persist).
- **Fix direction:** Use a sane default dismiss window (e.g. 5–10 s); gate “sticky” toasts behind an explicit `sticky: true` option for in-progress actions.
- **Acceptance check:** Default toast auto-dismiss ≤ 10 s under default config; sticky / progress toasts continue to work for genuinely long-running operations (e.g. async Stellar deposit confirmation).

---

## 6. Low Severity Issues (🟢)

### F-008 — Client-side global fetch timeout without abort sharing
- **Area:** frontend/api · **Evidence:** `acbu-frontend/lib/api/client.ts` (`AbortController` per call)
- **Impact:** Long hung requests waste battery; duplicated controllers if callers pass signal incorrectly.
- **Fix direction:** Document pattern; optional default timeout env.
- **Acceptance check:** Hung request cancels at configured deadline.

### F-011 — Package name still `my-v0-project`
- **Area:** frontend/devx · **Evidence:** `acbu-frontend/package.json` (`name`)
- **Impact:** Confusing telemetry, npm metadata, support tickets.
- **Fix direction:** Rename to `acbu-frontend`.
- **Acceptance check:** Published docs reference correct package name.

### F-031 — Business page hardcoded stats
- **Area:** frontend/business · **Evidence:** `acbu-frontend/app/(app)/business/page.tsx`
- **Impact:** Same trust issue for SMB story.
- **Fix direction:** Wire SME metrics API or mark demo mode clearly.
- **Acceptance check:** Demo mode banner when using mock numbers.

### F-033 — Help page too thin
- **Area:** frontend/support · **Evidence:** `acbu-frontend/app/me/help/page.tsx`
- **Impact:** Support load spikes.
- **Fix direction:** FAQ + status page links + ticket form.
- **Acceptance check:** Users can self-serve top 10 questions.

### F-034 — Sign-in label says Username only
- **Area:** frontend/auth · **Evidence:** `acbu-frontend/app/auth/signin/page.tsx`
- **Impact:** Users try wrong identifier format.
- **Fix direction:** Copy: username/email/phone per backend.
- **Acceptance check:** Support tickets about login drop in UAT.

### F-035 — Signup success query param ignored
- **Area:** frontend/auth · **Evidence:** `acbu-frontend/app/auth/signin/page.tsx`
- **Impact:** Missed positive reinforcement.
- **Fix direction:** Toast/banner on `?created=1`.
- **Acceptance check:** New user sees confirmation once.

### F-041 — Savings deposit URI length filter too strict
- **Area:** frontend/savings · **Evidence:** `acbu-frontend/app/(app)/savings/deposit/page.tsx`
- **Impact:** Blocks non-Stellar IDs incorrectly.
- **Fix direction:** Align validation with backend recipient resolver.
- **Acceptance check:** Phone-based IDs work where supported.

### F-042 — `as any` tabs in bills/currency
- **Area:** frontend/types · **Evidence:** `bills/page.tsx`, `currency/page.tsx`
- **Impact:** Weak typing hides invalid tab states.
- **Fix direction:** Discriminated unions for tab state.
- **Acceptance check:** TS strict mode clean for those files.

### F-043 — console.log in money handlers
- **Area:** frontend/hygiene · **Evidence:** Savings/bills/lending/currency pages
- **Impact:** Leaks PII to browser consoles in prod.
- **Fix direction:** Replace with structured logger behind `DEBUG` flag.
- **Acceptance check:** Prod bundle has no `console.log` in transfer paths.

### F-044 — Unstable list keys (rates/mint)
- **Area:** frontend/react · **Evidence:** `rates/page.tsx`, `mint/page.tsx`
- **Impact:** UI state bugs on reorder.
- **Fix direction:** Use stable ids from API.
- **Acceptance check:** No `key={index}` for dynamic lists.

### F-045 — No i18n / localization
- **Area:** frontend/product · **Evidence:** Whole app
- **Impact:** Africa MVP needs multi-locale formatting.
- **Fix direction:** Adopt next-intl or similar; extract strings.
- **Acceptance check:** At least NG/KE locale number formats supported.

### F-048 — Dark mode badge colors hardcoded light palette
- **Area:** frontend/theme · **Evidence:** `send/page.tsx` status badges
- **Impact:** Poor contrast in dark theme.
- **Fix direction:** Use semantic tokens from design system.
- **Acceptance check:** WCAG AA contrast on badges in dark mode.

### F-049 — Back navigation pattern inconsistent
- **Area:** frontend/ux · **Evidence:** Multiple pages mix `router.back` vs `Link`
- **Impact:** Users land on unexpected off-app pages.
- **Fix direction:** Prefer explicit `Link` targets within app shell.
- **Acceptance check:** Back never exits app shell unintentionally.

### F-052 — Send amount missing min attribute
- **Area:** frontend/forms · **Evidence:** `send/page.tsx`
- **Impact:** Negative amounts via keyboard.
- **Fix direction:** `inputMode="decimal"` + `min` + backend validation mirror.
- **Acceptance check:** Negative amounts impossible in UI.

### F-053 — Activity links wrong route target
- **Area:** frontend/nav · **Evidence:** `activity/page.tsx`
- **Impact:** Users open wrong detail screen.
- **Fix direction:** Link to canonical transaction detail route.
- **Acceptance check:** Deep link opens correct transfer id.

### F-054 — Me page links 2FA to auth flow not settings
- **Area:** frontend/nav · **Evidence:** `me/page.tsx`
- **Impact:** Confusing security UX loop.
- **Fix direction:** Point to settings-first flow.
- **Acceptance check:** User manages 2FA without re-auth loop.

### F-056 — Amount formatting inconsistent
- **Area:** frontend/formatting · **Evidence:** Home/transfers lists
- **Impact:** Users misread decimals especially for 7dp assets.
- **Fix direction:** Central `formatAcbu` helper with locale.
- **Acceptance check:** All lists use same formatter.

### F-057 — Reserves page lacks unit explanations
- **Area:** frontend/education · **Evidence:** `reserves/page.tsx`
- **Impact:** Users misinterpret health metrics.
- **Fix direction:** Tooltips linking to docs `RESERVE_MANAGEMENT`.
- **Acceptance check:** User testing shows improved comprehension.

### F-058 — Rates page number formatting
- **Area:** frontend/formatting · **Evidence:** `rates/page.tsx`
- **Impact:** Rates as raw strings hard to scan.
- **Fix direction:** Format with significant digits policy.
- **Acceptance check:** Rates match spec precision.

### F-059 — Burn page currency display locale
- **Area:** frontend/formatting · **Evidence:** `burn/page.tsx`
- **Impact:** NGN formatting not locale-aware.
- **Fix direction:** Use `Intl.NumberFormat`.
- **Acceptance check:** Locale switch changes grouping separators.

### F-060 — Performance: send page callbacks not memoized
- **Area:** frontend/perf · **Evidence:** `send/page.tsx`
- **Impact:** Extra rerenders on large contact lists.
- **Fix direction:** Memoize handlers + virtualize list.
- **Acceptance check:** React profiler shows acceptable commit times.

### F-061 — Clipboard API failure on HTTP
- **Area:** frontend/compat · **Evidence:** `receive/page.tsx`
- **Impact:** Copy address fails silently on non-secure origins.
- **Fix direction:** Detect + fallback prompt.
- **Acceptance check:** Copy works on HTTPS and shows message on HTTP.

### F-065 — No Subresource Integrity for third-party scripts (if any)
- **Area:** frontend/security · **Evidence:** `app/layout` / analytics
- **Impact:** Supply-chain risk if external scripts added.
- **Fix direction:** Prefer first-party hosting; SRI for CDNs.
- **Acceptance check:** Security review checklist item signed off.

---

## 7. Legacy 77-Item Origin List (cross-reference)

The legacy list previously alone at this path was a more recent manual review with slightly different framing. Items are preserved here for traceability; each is tagged `(moved)` (when it cleanly maps to a canonical `F-###` ID) or `(new)` (when it captures a distinct finding that should be merged into the canonical list in a follow-up PR).

### Critical

1. **Send success dialog shows empty amount** – app/(app)/send/page.tsx: `setAmount('')` runs before the success dialog. *(moved → F-014)*

### High – Security

2. **API key stored in sessionStorage** – *(moved → F-002)*
3. **Challenge token in URL** – *("challenge_token" query param).* *(moved → F-006)*

### High – Non-functional Flows

4. **Savings deposit only logs to console** – *(moved → F-015)*
5. **Savings "New Goal" button does nothing** – *(moved → F-016)*
6. **SME "Get Started" / "Apply Now" no handlers** – *(moved → F-017)*
7. **Lending application only logs to console** – *(moved → F-018)*
8. **Currency operations simulate with setTimeout** – *(moved → F-019)*
9. **Bill payment only logs to console** – *(moved → F-020)*
10. **Wallet confirm button disabled forever** – *(moved → F-021)*
11. **Security settings placeholder** – *(moved → F-022)*
12. **Mint page Burn tab non-functional** – *(moved → F-023)*

### Medium – UX/Design

13. **AFK vs ACBU naming inconsistency** – *(moved → F-024)*
14. **Balance always placeholder** – *(moved → F-025)*
15. **Savings: mock data mixed with API** – *(moved → F-026)*
16. **Savings withdraw: user field confusing** – *(moved → F-027)*
17. **Hardcoded fees and placeholder copy** – *(moved → F-028)*
18. **Me page KYC badge always "Pending"** – *(moved → F-029)*
19. **Me page balance and stats hardcoded** – *(moved → F-030)*
20. **Business page stats hardcoded** – *(moved → F-031)*
21. **Burn tab shows unhelpful text** – app/(app)/mint/page.tsx: copy on the Burn tab. The fake-success flow is captured by F-023; this entry covers the unimplemented / unclear copy. *(see F-023; F-023 covers the fake-success side, this covers the copy side)*
22. **International / gateway pages are stubs** – *(moved → F-032)*
23. **Help page minimal** – *(moved → F-033)*
24. **Sign-in identifier label misleading** – *(moved → F-034)*
25. **`?created=1` from signup ignored** – *(moved → F-035)*
26. **Confirm dialog does not show note** – *(moved → F-036)*
27. **Send note never sent to API** – *(moved → F-036)*

### Medium – Error Handling

28. **Consistent API error handling missing** – *(moved → F-037)*
29. **loadTransfers / loadContacts swallow errors** – *(moved → F-038)*
30. **getReceive errors ignored** – Cross-cuts F-038 + F-037. *(moved → F-038 + F-037)*
31. **No error boundary** – *(moved → F-039)*
32. **No CSRF protection** – *(moved → F-009)*

### Medium – State Management

33. **useEffect missing opts in dependency array (send)** – *(moved → F-040)*
34. **Infinite loop risk in contacts page** – *(moved → F-040)*
35. **Infinite loop risk in guardians page** – *(moved → F-040)*
36. **useEffect missing opts in KYC page** – *(moved → F-040)*
37. **Savings deposit user filter too strict** – *(moved → F-041 + F-075)*

### Medium – Code Quality

38. **`as any` usage in currency page** – *(moved → F-042)*
39. **`as any` usage in bills page** – *(moved → F-042)*
40. **console.log left in production code** – *(moved → F-043)*
41. **Index used as key in lists** – *(moved → F-044)*
42. **No i18n / localization** – *(moved → F-045)*

### Low – Accessibility

43. **Form labels and accessibility** – *(moved → F-046)*
44. **Nav links lack aria-labels** – *(moved → F-047)*
45. **Password toggle has no aria-label** – Cross-cuts F-047. *(moved → F-047)*
46. **Burn back link icon-only** – Cross-cuts F-047. *(moved → F-047)*
47. **Send detail back link icon-only** – Cross-cuts F-047. *(moved → F-047)*
48. **Settings cards missing aria-label** – Cross-cuts F-047. *(moved → F-047)*

### Low – Design Consistency

49. **Empty and loading states inconsistent** – Cross-cuts F-049 + a missing `Skeleton` primitive. **Distinct finding, adopted into canonical catalog as F-066** (Medium · frontend/components). *(new → adopted as F-066 in companion canonical-ID PR; see Section 5 and Section 8 for canonical rows)*
50. **Status badge colours in dark mode** – *(moved → F-048)*
51. **Back button pattern inconsistent** – *(moved → F-049)*

### Low – Forms

52. **Burn form fields missing maxLength and validation** – *(moved → F-050)*
53. **Signup passcode minimum too weak** – *(moved → F-051)*
54. **Send amount allows negative via keyboard** – *(moved → F-052)*
55. **Contact inputs have no maxLength** – Cross-cuts F-052 + new maxLength checks; **distinct finding, propose F-067.** *(new — propose F-067)*
56. **Profile inputs have no maxLength or format validation** – **Distinct finding, propose F-068.** *(new — propose F-068)*
57. **Recipient input has no maxLength** – Cross-cuts F-067. *(moved → F-067)*

### Low – Navigation

58. **Activity links to /send/${id} instead of /transactions/${id}** – *(moved → F-053)*
59. **"Two-Factor Auth" links to auth flow** – *(moved → F-054)*
60. **Nested interactive elements** – *(moved → F-055)*

### Low – Data Display

61. **Transfer amounts not formatted** – *(moved → F-056)*
62. **Mock "Total Savings" mixed with API balance** – *(moved → F-026)*
63. **Currency balance misleading** – Cross-cuts F-026. *(moved → F-026)*
64. **Reserve data lacks context** – *(moved → F-057)*
65. **Rate displayed without formatting** – *(moved → F-058)*
66. **Currency shown without locale** – *(moved → F-059)*

### Low – Performance

67. **loadTransfers / loadContacts not memoized** – *(moved → F-060)*
68. **Icons recreated each render** – **Distinct finding, propose F-069.** *(new — propose F-069)*
69. **Menu items use router.push instead of Link** – **Distinct finding, propose F-070.** *(new — propose F-070)*

### Trivial

70. **Clipboard API may fail on HTTP** – *(moved → F-061)*
71. **Toast removal delay is ~17 minutes** – `TOAST_REMOVE_DELAY = 1000000`. **Distinct finding, adopted into canonical catalog as F-071** (Medium · frontend/ux). *(new → adopted as F-071 in companion canonical-ID PR; see Section 5 and Section 8 for canonical rows)*
72. **Mobile detection treats "unknown" as desktop** – **Distinct finding, propose F-072.** *(new — propose F-072)*
73. **Post-KYC upload navigation uses uncleaned setTimeout** – **Distinct finding, propose F-073.** *(new — propose F-073)*
74. **Silent username normalization on signup** – `lib/api/auth.ts` lowercases + strips whitespace without warning UI. **Distinct finding, propose F-074.** *(new — propose F-074)*
75. **Auto-fill heuristic requires length >= 56** – **Distinct finding, propose F-075.** *(new — propose F-075)* — note: this overlaps the older F-041 but addresses a different code path (lending + savings auto-fill, not URI-length filter).
76. **API fetch has no timeout** – Frontend `request()` uses `fetch` with no default timeout. **Distinct finding, propose F-076.** *(new — propose F-076)*
77. **/p2p is client-side redirect only** – `app/(app)/p2p/page.tsx` does `useEffect -> router.replace('/send')`. **Distinct finding, propose F-077.** *(new — propose F-077)*

> **Reconciliation policy:** When a follow-up audit pass opens a new canonical ID for items tagged `(new)`, the row is migrated forward and the tagged comment block in this section is updated to point at the new ID. New IDs are expected to land in `F-066+` and `F-071..F-076` should preserve backward-reference to this section so future readers can trace provenance.

---

## 8. Resolution Tracker (Fix Status)

> **Format:** `ID · title · status · linked PR / commit · owner`
>
> **Forward-looking template.** Every item is currently 🟡 Open; uniformity of status reflects the state of the working backlog. When a fix merges, the row transitions to ✅ Fixed with a PR link. Example anchors below show what a real fixed row looks like.

| ID | Title | Status | Notes |
|----|-------|--------|-------|
| _(example)_ | _(example anchor for prior documentary PRs — PR #14 closed issue #1, PR #22 closed #12, PR #23 backend, PR #24 master-index)_ | ✅ Fixed (PR #24 merged) | Documentary PR — not a code fix; historical anchor |
| F-001..F-005 | _Critical cluster_ | 🟡 Open (ship blockers) | Track with TypeScript / wallet / auth refactor |
| F-006, F-010, F-014..F-023, F-025, F-027, F-051, F-063 | _High cluster (14 items)_ | 🟡 Open | Next.js auth + feature-flag workstream |
| F-007, F-009, F-012, F-013, F-016, F-017, F-024, F-026, F-028–F-030, F-032, F-036–F-040, F-046, F-047, F-050, F-055, F-062, F-064, F-066, F-071 | _Medium cluster (25 items)_ | 🟡 Open | Quality / a11y / hardening workstream |
| F-008, F-011, F-031, F-033..F-035, F-041..F-045, F-048..F-049, F-052..F-054, F-056..F-061, F-065 | _Low cluster (23 items)_ | 🟡 Open | Polish / formatting / docs workstream |

> **Status legend** — 🟢 Trivial/cosmetic · 🟡 Open (tracked) · 🔵 In review · ✅ Fixed (PR linked)
>
> **Reminder when updating:** When an external PR closes an issue, replace `🟡 Open` with `✅ Fixed` and append the merged PR link in the Notes column. Do not rewrite history; the row itself is the audit trail.

---

## 9. Cross-Reference Map

| Document | Relationship to this file |
|----------|---------------------------|
| [`../../issues/frontend.md`](../../issues/frontend.md) | **Live triage-ready catalog.** Same IDs and severities; entries are condensed for daily use. |
| [`../../issues/MASTER_INDEX.md`](../../issues/MASTER_INDEX.md) | Master index across backend/frontend/contracts backlogs; Top 20 ship-safety list with per-row tracker pointers. |
| [`../../issues/contracts.md`](../../issues/contracts.md) | Smart contracts MVP issues (C-001..C-060). |
| [`../../issues/backend.md`](../../issues/backend.md) | Backend MVP issues (B-001..B-075). |
| [`../CONTRACTS_ISSUES.md`](../CONTRACTS_ISSUES.md) | Companion long-form reference for smart contracts (PR #22). |
| [`../BACKEND_ISSUES.md`](../BACKEND_ISSUES.md) | Companion long-form reference for backend (PR #23). |
| [`../API_AND_CONTRACTS_REFERENCE.MD`](../API_AND_CONTRACTS_REFERENCE.MD) | API surface map; front-end integration anchors. |
| [`../PROJECT_STRUCTURE.MD`](../PROJECT_STRUCTURE.MD) | Repo layout (where the frontend source lives). |
| [`../../USER_EXPERIENCE/USER_FLOWS.MD`](../../USER_EXPERIENCE/USER_FLOWS.MD) | UX flows context for F-015, F-018–F-020 (console-only features). |
| [`../../USER_EXPERIENCE/MOBILE_OPTIMIZATION_COMPLETE.md`](../../USER_EXPERIENCE/MOBILE_OPTIMIZATION_COMPLETE.md) | Mobile-first context for F-047 (mobile nav a11y) and F-072. |
| [`../../USER_EXPERIENCE/DIASPORA_REMITTANCES.MD`](../../USER_EXPERIENCE/DIASPORA_REMITTANCES.MD) | Diaspora remittance context for F-019, F-020, F-032. |
| [`../../SECURITY_GUIDE.MD`](../../SECURITY_GUIDE.MD) | Security posture context for F-001..F-005 + F-064 + F-065. |
| [`../../correction.md`](../../correction.md) | Standing list of items curated for fast triage (frontend visibility gaps). |

---

## 10. Top Remediations (ship-safety order)

Order these fixes in the MVP launch runbook before deploying production:

1. **F-001..F-005** — Hard wallet + auth + build pipeline overhaul: turn off `ignoreBuildErrors`, move secrets off sessionStorage, replace base64 with AES-GCM, block plaintext wallet path.
2. **F-002 / F-003 / F-005** — Single PR for the wallet secrets stack (migrate to WebCrypto keys, kill plaintext store, add sessionStorage lint rule).
3. **F-022 / F-039 / F-064** — Settings-first security surface + global error boundary + CSP baseline.
4. **F-014 / F-015 / F-018 / F-019 / F-020 / F-022 / F-023 / F-021** — Stubbed feature clean-up: savings deposit, lending application, currency operations, bills payment, security settings, mint/burn tab, wallet confirm.
5. **F-025 / F-027 / F-051 / F-063** — Data integrity + auth strength + environment footgun.
6. **F-006 / F-010 / F-046 / F-047** — Auth/secrets hygiene + accessibility on money routes.
7. **F-040 / F-039 / F-038 / F-037** — Quality + reliability pass (react-hooks lint, error boundaries, retry UI, useApiError hook).
8. **F-007 / F-009 / F-012 / F-013** — Dev infrastructure (single header, CSRF pairing, typecheck/smoke CI, single lockfile).
9. **F-016 / F-017 / F-032 / F-033 / F-034 / F-035** — Feature-flag and copy cleanups (inert CTAs, dead nav, sign-in label, signup confirmation).
10. **F-008 / F-011 / F-031 / F-041..F-065** — Polish + tooling (timeouts, package rename, demo-mode banners, format/locale helpers, accessibility polish).

---

## 11. Maintenance

- **New issues** — Add to `issues/frontend.md` first (canonical triage). Mirror to this file with the next stable `F-###` ID.
- **Closed fixes** — Promote the fix to `✅ Fixed (PR linked)` in Section 8 and link to the merged PR.
- **Legacy `(new)` items in Section 7** — Open a follow-up canonical-ID PR; on merge, update both `issues/frontend.md` and this file.
- **Cross-doc references** — Update `issues/MASTER_INDEX.md` summary counts (Top 20 pointer list) when severities change.

---

## Related Documents

- [Live frontend issue catalog](../../issues/frontend.md)
- [Master backlog index](../../issues/MASTER_INDEX.md)
- [Contracts long-form reference](../CONTRACTS_ISSUES.md)
- [Backend long-form reference](../BACKEND_ISSUES.md)
- [Smart contract spec (root copy)](../SMART_CONTRACT_SPEC.md)
- [Project structure map](../PROJECT_STRUCTURE.md)
- [User flows](../../USER_EXPERIENCE/USER_FLOWS.md)
- [Security guide](../../SECURITY_GUIDE.md)

---

_Last consolidated: June 2026. Source content merged from legacy `PROJECT/issues/FRONTEND_ISSUES.md` list (77 items) and canonical [`issues/frontend.md`](../../issues/frontend.md) (65 items). Severity buckets and IDs fully aligned with canonical catalog; legacy 77-item list preserved as Section 7 with `(moved)` and `(new)` tags for traceability. New audit findings captured under `(new)` should be promoted to canonical IDs in the `F-066..F-077` range in a follow-up PR._
