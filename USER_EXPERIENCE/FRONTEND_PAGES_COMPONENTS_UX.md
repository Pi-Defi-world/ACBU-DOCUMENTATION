# Frontend Pages, Components & UI/UX Map

Map of backend endpoints and smart-contract flows to **frontend pages**, **components**, and **UI/UX ideas**. Use this to build the ACBU web app against the existing API and contracts.

**Backend base:** `GET/POST /v1/...` (see [API and Contracts Reference](PROJECT/API_AND_CONTRACTS_REFERENCE.MD)).  
**UI standards:** [Frontend & UI/UX Guide](FRONTEND_AND_UI.md).

---

## 1. Auth

| Backend | Frontend page(s) | Components | UI/UX ideas |
|--------|-------------------|------------|-------------|
| `POST /auth/signin` | **Sign in** | `SignInForm`, `EmailInput`, `PasswordInput`, `SubmitButton` | Single form: email + password. Clear “Forgot password?” link. Error inline under form. No clutter. |
| `POST /auth/signin/verify-2fa` | **Verify 2FA** (after signin when 2FA enabled) | `OtpInput`, `Verify2FAButton` | 6-digit OTP input with auto-focus next. Short copy: “Enter code from your app.” Resend code countdown. |
| `POST /auth/signout` | (No dedicated page) | — | Call on “Sign out” in header/settings. Optional confirmation for shared devices. |

**Flows:** Sign in → dashboard or 2FA step → dashboard. Sign out from header or Settings.

---

## 2. User management & profile

| Backend | Frontend page(s) | Components | UI/UX ideas |
|--------|-------------------|------------|-------------|
| `GET /users/me` | **Profile** (or **Account**) | `ProfileCard`, `Avatar`, `ProfileField` | Show name, email, Stellar address, KYC badge, tier. Clear hierarchy: identity first, then wallet/status. |
| `PATCH /users/me` | **Profile** (edit) | `ProfileForm`, `TextInput`, `SaveButton` | Inline or modal edit. Only editable fields (e.g. display name, phone). Success toast after save. |
| `DELETE /users/me` | **Settings** or **Account** | `DeleteAccountSection`, `ConfirmDialog` | “Delete account” in danger zone. Two-step: confirm password + type “DELETE”. Clear warning text. |

---

## 3. Settings (user)

| Backend | Frontend page(s) | Components | UI/UX ideas |
|--------|-------------------|------------|-------------|
| `GET /users/me/receive` | **Receive** (or under Settings/Wallet) | `ReceiveDetails`, `AddressDisplay`, `CopyButton` | Show Stellar address and optional memo. One-tap copy. Short explanation: “Share this to receive ACBU.” |
| `GET /users/me/receive/qrcode` | **Receive** | `QrCodeDisplay` | QR for address (and memo if used). Same page as receive details. |
| `POST /users/me/wallet/confirm` | **Wallet** or **Settings** | `ConfirmWalletButton`, `StatusBadge` | After first funding, user confirms wallet active. Button + short copy. |
| `GET/POST/DELETE /users/me/contacts` | **Contacts** (or **Settings → Contacts**) | `ContactList`, `ContactCard`, `AddContactForm`, `ContactRow` | List with search. Add: alias + Stellar (or resolve). Delete with row action + confirm. |
| `GET/POST/DELETE /users/me/guardians` | **Settings → Recovery** (or **Guardians**) | `GuardianList`, `AddGuardianForm`, `GuardianCard` | List guardians. Add: email or Stellar. Explain purpose (recovery). Delete with confirm. |

**Settings page idea:** Tabs or sections: Profile, Wallet & receive, Contacts, Guardians, Security (2FA), Danger zone (delete account). Each section uses the components above.

---

## 4. KYC

| Backend | Frontend page(s) | Components | UI/UX ideas |
|--------|-------------------|------------|-------------|
| `POST /kyc/applications` | **KYC – Start** | `KycStartForm`, `CountrySelect`, `ConsentCheckbox` | Minimal form: country, accept terms. Then redirect to “Upload documents”. |
| `GET /kyc/applications` | **KYC – My applications** | `KycApplicationList`, `KycStatusBadge`, `ApplicationCard` | List with status: pending, in_review, approved, rejected. Click → detail. |
| `GET /kyc/applications/:id` | **KYC – Application detail** | `KycApplicationDetail`, `DocumentStatusList`, `StatusTimeline` | Show status, submitted docs, and next steps (e.g. “Upload ID if missing”). |
| `GET /kyc/applications/upload-urls` | **KYC – Upload documents** | `DocumentUploadZone`, `UploadUrlRequestButton`, `FileInput` | Per doc type: get upload URL from backend, then upload file (or use pre-signed flow). Show “Upload ID”, “Upload proof of address” etc. |
| `PATCH /kyc/applications/:id/documents` | **KYC – Application detail** | `DocumentPatchForm`, `DocumentTypeSelect` | After upload, patch application with doc references. Inline or same page as upload. |

**KYC flow:** Start application → Upload documents (with upload-urls + patch) → View application status → Optional “Validator” path below.

**Validator (KYC reviewer):**

| Backend | Frontend page(s) | Components | UI/UX ideas |
|--------|-------------------|------------|-------------|
| `POST /kyc/validator/register` | **Become validator** (onboarding) | `ValidatorRegisterForm` | One-time: terms, identity. After approval, access task list. |
| `GET /kyc/validator/tasks` | **Validator – Tasks** | `ValidatorTaskList`, `TaskCard`, `TaskFilters` | Queue of applications to review. Status filters. Click → task detail. |
| `GET /kyc/validator/me` | **Validator – Profile** | `ValidatorProfileCard` | Stats, status, rewards (if shown). |
| `POST /kyc/validator/tasks/:id` | **Validator – Task detail** | `TaskReviewForm`, `ApproveRejectButtons`, `DocumentViewer` | View applicant docs, add notes, submit approve/reject. Clear CTA and confirmation. |

---

## 5. Account recovery

| Backend | Frontend page(s) | Components | UI/UX ideas |
|--------|-------------------|------------|-------------|
| `POST /recovery/unlock` | **Recovery / Unlock account** (no API key) | `RecoveryForm`, `GuardianCodeInput`, `UnlockButton` | User enters recovery code (from guardians). Simple form, clear success/error. Link from sign-in: “Locked out?” |

---

## 6. Reserves & rates (public / dashboard)

| Backend | Frontend page(s) | Components | UI/UX ideas |
|--------|-------------------|------------|-------------|
| `GET /reserves` | **Reserves** (public or dashboard) | `ReserveSummary`, `ReserveCurrencyList`, `ReserveHealthBadge` | Total supply, reserve value, health. Table or cards per currency (target vs actual weight). Calm, trust-oriented. |
| `POST /reserves/track` | **Reserves** (admin/ops) | `TriggerTrackButton` | Optional “Refresh” for manual run. Feedback: “Tracking started.” |
| `GET /rates` | **Rates** (dashboard or header) | `RatesTable`, `RateRow`, `RatesLastUpdated` | ACBU vs USD and basket currencies. Optional filter by currency. |
| `GET /rates/quote` | **Quote** (used in Mint/Burn/International) | `QuoteDisplay`, `QuoteInput` (amount + currency) | User enters amount (and optional currency); show equivalent. Reuse in mint/burn flows. |

---

## 7. Mint & burn (core)

| Backend | Frontend page(s) | Components | UI/UX ideas |
|--------|-------------------|------------|-------------|
| `POST /mint/usdc` | **Mint** (or **Add funds**) | `MintForm`, `AmountInput`, `WalletAddressInput`, `MintConfirmStep` | Step 1: amount (USDC) + wallet. Step 2: quote (from rates/quote) + confirm. Success: tx id + “ACBU will arrive shortly.” |
| `POST /burn/acbu` | **Burn** (or **Withdraw**) | `BurnForm`, `AmountInput`, `RecipientBankForm`, `BurnConfirmStep` | Amount + currency + recipient (account number, bank code, name). Quote + fee. Confirm step with clear summary. |

**Contract link:** Mint uses `acbu_minting`; burn uses `acbu_burning`. Backend handles contract calls; frontend only shows status and result.

---

## 8. Transfers (P2P)

| Backend | Frontend page(s) | Components | UI/UX ideas |
|--------|-------------------|------------|-------------|
| `POST /transfers` | **Send** (P2P) | `SendForm`, `RecipientPicker` (alias or Stellar), `AmountInput`, `SendConfirm` | Recipient (resolve via GET /recipient), amount, optional memo. Confirm step. Success: transfer id. |
| `GET /transfers` | **Transfer history** | `TransferList`, `TransferRow`, `TransferFilters`, `Pagination` | List with date, recipient, amount, status. Filter by direction/status. |
| `GET /transfers/:id` | **Transfer detail** | `TransferDetailCard`, `StatusTimeline` | Full details; link from list. |
| `GET /recipient` | (Used inside Send) | `RecipientResolver`, `RecipientSearch` | Resolve alias → Stellar. Autocomplete or search in Send form. |

Same endpoints are used by **P2P segment** (`POST /p2p/send`, `GET /p2p/history`, `GET /p2p/:id`). Same pages; optionally different nav (e.g. “P2P” vs “Transfers”).

---

## 9. Transactions

| Backend | Frontend page(s) | Components | UI/UX ideas |
|--------|-------------------|------------|-------------|
| `GET /transactions/:id` | **Transaction detail** (from list or notification) | `TransactionDetailCard`, `TransactionTypeBadge`, `TxStatusStepper` | Type (mint/burn/transfer), amount, status, timestamps, blockchain link if available. |

**Idea:** Global “Activity” or “Transactions” page that lists all (mint, burn, transfer) using transfers + transaction endpoints where applicable.

---

## 10. International (mint / burn / quote)

Same as **Mint & burn** and **Rates**, but under segment **International** (scope `international:read` / `international:write`).

| Backend | Frontend page(s) | Components | UI/UX ideas |
|--------|-------------------|------------|-------------|
| `GET /international/quote` | **International – Quote** | Same `QuoteDisplay` / `QuoteInput` | Dedicated “Get a quote” before mint/burn. |
| `POST /international/mint/usdc` | **International – Mint** | Same as Mint | Same flow; segment-specific nav/label. |
| `POST /international/burn/acbu` | **International – Withdraw** | Same as Burn | Same flow; segment-specific nav. |

**Page idea:** One “International” section: Quote, Mint, Withdraw (tabs or steps).

---

## 11. P2P segment

Same backend as **Transfers**; segment routes: `POST /p2p/send`, `GET /p2p/history`, `GET /p2p/:id`. Use same **Send** and **Transfer history** pages; label as “P2P” or “Send” in nav.

---

## 12. SME segment

| Backend | Frontend page(s) | Components | UI/UX ideas |
|--------|-------------------|------------|-------------|
| `POST /sme/transfers` | **SME – Send** | Same `SendForm` + `RecipientPicker`, `AmountInput` | Same as P2P send; SME branding/sidebar. |
| `GET /sme/transfers` | **SME – Transfers** | Same `TransferList`, `TransferRow` | List SME transfers. |
| `GET /sme/transfers/:id` | **SME – Transfer detail** | Same `TransferDetailCard` | Detail view. |
| `GET /sme/statements` | **SME – Statements** | `StatementList`, `StatementFilters`, `ExportButton` | Same data as transfers; present as “Statements” (e.g. date range, export). |

**Page idea:** SME dashboard: Send, Transfers, Statements (tabs or sidebar).

---

## 13. Salary segment

| Backend | Frontend page(s) | Components | UI/UX ideas |
|--------|-------------------|------------|-------------|
| `POST /salary/disburse` | **Salary – Disburse** | `DisburseForm`, `BatchRecipientsInput`, `AmountPerRecipient`, `DisburseConfirm` | Upload or paste recipients (e.g. Stellar + amount). Summary + confirm. (Backend may be stub.) |
| `POST /salary/schedule` | **Salary – Schedule** | `ScheduleForm`, `RecurrenceSelect`, `RecipientListSelect` | Set recurring batch (e.g. monthly). (Backend may be stub.) |
| `GET /salary/batches` | **Salary – Batches** | `BatchList`, `BatchCard`, `BatchStatusBadge` | List past and scheduled batches; click for detail. |

**Page idea:** Payroll section: Disburse now, Schedule, Batch history.

---

## 14. Enterprise segment

| Backend | Frontend page(s) | Components | UI/UX ideas |
|--------|-------------------|------------|-------------|
| `POST /enterprise/bulk-transfer` | **Enterprise – Bulk transfer** | `BulkTransferForm`, `CsvUpload`, `BulkPreviewTable`, `ConfirmBulk` | Upload CSV (recipient, amount); preview; confirm. (Backend may be stub.) |
| `GET /enterprise/treasury` | **Enterprise – Treasury** | `TreasuryDashboard`, `TreasurySummary`, `BalanceByCurrency`, `RecentMovements` | High-level balances, positions, recent activity. (Backend may be stub.) |

**Page idea:** Enterprise dashboard: Treasury overview, Bulk transfer.

---

## 15. Savings (acbu_savings_vault)

| Backend | Frontend page(s) | Components | UI/UX ideas |
|--------|-------------------|------------|-------------|
| `POST /savings/deposit` | **Savings – Deposit** | `SavingsDepositForm`, `TermSelect`, `AmountInput`, `DepositConfirm` | Amount + term (e.g. 30/90/180 days). Show expected yield if API returns it. Confirm. |
| `POST /savings/withdraw` | **Savings – Withdraw** | `SavingsWithdrawForm`, `PositionSelect`, `WithdrawConfirm` | Choose position (or auto if one). Show early-withdrawal penalty if any. Confirm. |
| `GET /savings/positions` | **Savings – Positions** | `PositionsList`, `PositionCard`, `PositionProgress` | List positions: amount, term, maturity, accrued yield. Clear “Deposit” CTA. |

**Page idea:** Savings hub: Positions (default), Deposit, Withdraw. Contract: `acbu_savings_vault`.

---

## 16. Lending (acbu_lending_pool)

| Backend | Frontend page(s) | Components | UI/UX ideas |
|--------|-------------------|------------|-------------|
| `POST /lending/deposit` | **Lending – Deposit** | `LendingDepositForm`, `AmountInput`, `DepositConfirm` | Deposit ACBU into pool. Summary + confirm. |
| `POST /lending/withdraw` | **Lending – Withdraw** | `LendingWithdrawForm`, `AmountInput`, `WithdrawConfirm` | Withdraw from pool (subject to liquidity). |
| `GET /lending/balance` | **Lending – Dashboard** | `LenderBalanceCard`, `LendingStats`, `DepositWithdrawCtas` | Show lender balance and optional APY; CTAs for deposit/withdraw. |

**Page idea:** Lending section: Balance/dashboard, Deposit, Withdraw. Contract: `acbu_lending_pool`.

---

## 17. Gateway (merchant, acbu_escrow)

| Backend | Frontend page(s) | Components | UI/UX ideas |
|--------|-------------------|------------|-------------|
| `POST /gateway/charges` | **Gateway – Create charge** | `ChargeForm`, `AmountInput`, `ChargeMetadata`, `ChargeConfirm` | Create escrow charge (amount, reference, optional metadata). Return charge id / link for customer. |
| `POST /gateway/confirm` | **Gateway – Confirm** | `ConfirmForm`, `ChargeIdInput`, `ReleaseRefundChoice`, `ConfirmButton` | Release or refund an escrow. Merchant flow. |

**Page idea:** Merchant dashboard: Create charge, List charges (if backend adds later), Release/Refund. Contract: `acbu_escrow`.

---

## 18. Bills

| Backend | Frontend page(s) | Components | UI/UX ideas |
|--------|-------------------|------------|-------------|
| `GET /bills/catalog` | **Bills – Catalog** | `BillerList`, `BillerCard`, `BillerSearch` | List billers (e.g. utilities, airtime). Search/filter. (Backend may be stub.) |
| `POST /bills/pay` | **Bills – Pay** | `BillPayForm`, `BillerSelect`, `AccountRefInput`, `AmountInput`, `PayConfirm` | Select biller, enter customer ref, amount. Confirm. (Backend may be stub.) |

**Page idea:** Bills: Catalog, Pay bill.

---

## 19. Health (optional for frontend)

| Backend | Frontend page(s) | Components | UI/UX ideas |
|--------|-------------------|------------|-------------|
| `GET /health` | (Ops / status page) | — | Optional public status widget. |
| `GET /health/metrics` | (Ops dashboard) | `ReserveRatioBadge` | Optional internal dashboard. |

---

## 20. Shared components (cross-cutting)

Use these across the pages above.

| Component | Purpose | UI/UX |
|-----------|---------|--------|
| **Layout** | `AppLayout`, `Sidebar`, `Header`, `Footer` | Consistent nav; user menu (profile, settings, sign out). |
| **Navigation** | `NavItem`, `SegmentNav` (P2P, SME, International, etc.) | Segment routes gated by scope; hide nav user can’t access. |
| **Auth guard** | Wrapper that checks auth and redirects to sign-in | Use on all authenticated routes. |
| **Amount display** | `Amount`, `CurrencyLabel` | Format ACBU and fiat consistently; optional tooltip for full precision. |
| **Status** | `StatusBadge`, `StatusPill` | pending, completed, failed, etc. One style across transfers, transactions, KYC. |
| **Empty state** | `EmptyState`, `EmptyIllustration` | Icon + message + optional CTA (e.g. “No transfers yet” → “Send”). |
| **Loading** | `PageSkeleton`, `ButtonSpinner`, `InlineLoader` | Skeletons for lists/detail; spinner in buttons during submit. |
| **Errors** | `InlineError`, `ToastError`, `ErrorBanner` | Inline for fields; toast or banner for global errors. |
| **Modals** | `ConfirmModal`, `FormModal` | Confirm destructive actions; small forms (e.g. add contact). |
| **Forms** | `Input`, `Select`, `Checkbox`, `RadioGroup` | From design tokens; consistent validation and error display. |

---

## 21. Suggested app structure (routes / nav)

- **Public:** Sign in, Recovery unlock, (optional) Reserves/Rates.
- **Authenticated:**  
  - **Home/Dashboard:** Summary (balance, recent activity), quick actions (Send, Mint, Burn).  
  - **Profile:** GET/PATCH /users/me.  
  - **Settings:** Receive, Wallet confirm, Contacts, Guardians, Delete account.  
  - **KYC:** Start, My applications, Application detail, Upload documents.  
  - **Validator:** (if user is validator) Tasks, Task detail, Me.  
  - **Transfers / P2P:** Send, History, Transfer detail.  
  - **Transactions:** List (mint/burn/transfer), Transaction detail.  
  - **Mint / Burn:** Mint, Burn (or under “Funds” / “International”).  
  - **Rates / Reserves:** Rates, Reserves.  
  - **SME:** Send, Transfers, Statements.  
  - **International:** Quote, Mint, Withdraw.  
  - **Salary:** Disburse, Schedule, Batches.  
  - **Enterprise:** Treasury, Bulk transfer.  
  - **Savings:** Positions, Deposit, Withdraw.  
  - **Lending:** Balance, Deposit, Withdraw.  
  - **Gateway:** Create charge, Confirm (release/refund).  
  - **Bills:** Catalog, Pay.

Nav can be role/tier-based: show only segments the user’s API key (or role) can access.

---

## 22. API usage summary

- **Auth:** Store token/API key securely; send `Authorization: Bearer <key>` (or header per backend).
- **Segment routes:** Require scope (e.g. `p2p:read`). Backend returns 403 if missing; frontend can hide nav for segments user can’t access.
- **Errors:** Use backend error shape (code, message, details) for inline and toast messages.
- **Lists:** Use query params for filters/pagination if backend supports them; otherwise client-side filter.

This map is the single source for “what to build” for the frontend; implementation should follow the [Frontend & UI/UX Guide](FRONTEND_AND_UI.md) for look and feel.


---

## 23. Dashboard & Home page

**Purpose:** User entry point post-login. Shows quick stats, recent activity, and navigation shortcuts.

| Component | Purpose | Details |
|-----------|---------|---------|
| `DashboardLayout` | Main container | Header + sidebar + main content area. |
| `BalanceSummary` | Display balance | Total ACBU balance in primary currency. Show equivalent in USD if applicable. |
| `RecentActivityList` | Last 5-10 txs | Mint, burn, transfer, savings deposit/withdraw. Paginated or "view all" link. |
| `QuickActionBar` | CTAs | Send, Mint, Burn, Deposit (savings), Withdraw. Toggle based on user scope. |
| `ExchangeRateWidget` | Live rates | ACBU/USD and basket. Auto-refresh or manual. |
| `PortfolioBreakdown` | Holdings | If user has savings/lending positions, show breakdown. |
| `UpcomingMaturities` | Savings/lending | Show next maturity dates if relevant. |

**User flow:**
1. Sign in → Dashboard
2. Tap quick action → relevant page (Send, Mint, etc.)
3. Or navigate via sidebar to other sections.

---

## 24. Component library & design tokens

All components should follow a unified design language. Reference [FRONTEND_AND_UI.md](FRONTEND_AND_UI.md) for:

- **Color palette:** Primary (brand), secondary, success, warning, error, neutral grays.
- **Typography:** Headings (H1–H4), body, labels, captions. Font sizes and weights.
- **Spacing:** 4px, 8px, 16px, 24px, 32px grid.
- **Borders & radius:** 4px, 8px, 12px for standard corners.
- **Shadows:** Elevation levels (none, low, medium, high).
- **Icons:** Consistent icon set (e.g. Feather, Material, custom).

**Component conventions:**
- Prefix all components: `Acbu*` or match project naming.
- Props: `className`, `isLoading`, `isDisabled`, `error`, `success`.
- Events: `onChange`, `onSubmit`, `onCancel`, `onDelete`.
- Accessibility: ARIA labels, semantic HTML, keyboard navigation.

---

## 25. Forms & validation

| Pattern | Implementation |
|---------|-----------------|
| **Text input** | Controlled component; debounce if calling API (e.g. resolve recipient). |
| **Amount input** | Validate: >= 0, max decimals, max amount (if limit exists). Show error inline. |
| **Select/dropdown** | Options fetched from backend or static. Searchable if >5 items. |
| **Checkbox** | Single or multi-select. Use for terms acceptance, filters. |
| **Radio** | Single choice from list. Use for burn recipient options, etc. |
| **File upload** | Accept image/PDF per KYC doc type. Show preview. Validate size before upload. |
| **OTP input** | 6-digit auto-focus and auto-submit on complete. |
| **Multi-step form** | Wizard: step indicator, "back" button, validate before "next". |

**Error handling:**
- Show error text below field in red. Persist until corrected.
- Highlight input border in error color.
- Disable submit button if form has errors.
- Show success toast after successful submit.

---

## 26. Modals & overlays

| Use case | Component | Behavior |
|----------|-----------|----------|
| **Confirmation** | `ConfirmModal` | Title, message, "Cancel" + "Confirm" buttons. Red CTA for destructive actions. |
| **Error** | `ErrorModal` or `ErrorBanner` | Show error title and description. "Dismiss" or "Retry" button. |
| **Loading** | `LoadingOverlay` | Spinner + optional message. Prevent interaction until complete. |
| **Form modal** | `FormModal` | Embed form inside modal. "Save" + "Cancel". Close on ESC or backdrop click. |
| **Drawer** | `SideDrawer` | For larger content (e.g. contact edit, batch preview). Slide from right/left. |

---

## 27. Notifications & feedback

| Type | When | Example |
|------|------|---------|
| **Toast (success)** | Action completed | "Transfer sent successfully." |
| **Toast (error)** | Action failed | "Insufficient balance. Please mint more ACBU." |
| **Toast (info)** | FYI | "Exchange rate updated." |
| **In-line error** | Form validation | Red text under input field. |
| **Banner** | Page-level warning | "KYC under review. Limits reduced." |
| **Skeleton** | Loading data | Gray placeholder shapes while fetching. |

**Duration:**
- Success toasts: 3–4 seconds.
- Error toasts: Persist until dismissed or 6 seconds.
- Banners: Persist until dismissed.

---

## 28. Navigation & routing

**Route structure (React Router example):**

```
/
├── /auth
│   ├── /signin
│   └── /recovery
├── /dashboard (protected)
│   ├── / (home)
│   ├── /profile
│   ├── /settings
│   │   ├── /receive
│   │   ├── /contacts
│   │   ├── /guardians
│   │   └── /security
│   ├── /kyc
│   │   ├── /start
│   │   ├── /applications
│   │   ├── /applications/:id
│   │   └── /upload
│   ├── /validator (if role = validator)
│   │   ├── /tasks
│   │   └── /tasks/:id
│   ├── /transfers
│   │   ├── /send
│   │   ├── /history
│   │   └── /:id
│   ├── /transactions
│   │   ├── / (list)
│   │   └── /:id
│   ├── /mint
│   ├── /burn
│   ├── /rates
│   ├── /reserves
│   ├── /p2p (if scope p2p:read)
│   │   ├── /send
│   │   ├── /history
│   │   └── /:id
│   ├── /international (if scope international:read)
│   │   ├── /quote
│   │   ├── /mint
│   │   └── /withdraw
│   ├── /sme (if role SME)
│   │   ├── /send
│   │   ├── /transfers
│   │   ├── /statements
│   │   └── /:id
│   ├── /salary (if scope salary:write)
│   │   ├── /disburse
│   │   ├── /schedule
│   │   └── /batches
│   ├── /enterprise (if role Enterprise)
│   │   ├── /bulk-transfer
│   │   └── /treasury
│   ├── /savings
│   │   ├── / (positions)
│   │   ├── /deposit
│   │   └── /withdraw
│   ├── /lending
│   │   ├── / (balance)
│   │   ├── /deposit
│   │   └── /withdraw
│   ├── /gateway (if role Merchant)
│   │   ├── /create-charge
│   │   └── /manage
│   └── /bills
│       ├── /catalog
│       └── /pay
└── /error (optional catch-all for 404, 500, etc.)
```

**Route guards:**
- All `/dashboard/*` routes require authentication.
- Segment routes (e.g. `/p2p`, `/sme`) check user scope and redirect to dashboard if denied.
- Validator routes check if user has validator role.

---

## 29. State management

**Recommended approach:**

- **Auth state:** Token, user profile, scopes. Use context or Redux.
- **UI state:** Loading, error messages, modals. Use local state or Zustand.
- **Data fetching:** React Query (TanStack Query) for caching and sync.
- **Form state:** React Hook Form + Zod/Yup for validation.

**Example pattern:**
```javascript
// Auth context
const AuthContext = React.createContext();

// Dashboard page
function Dashboard() {
  const { user, token } = useContext(AuthContext);
  const { data: balance, isLoading } = useQuery(
    ['balance'],
    () => fetchBalance(token),
    { staleTime: 60 * 1000 }
  );
  // ...
}
```

---

## 30. API integration

**Base URL:** `https://api.acbu.app/v1` (or test endpoint for dev).

**Headers:**
```javascript
{
  'Authorization': `Bearer ${token}`,
  'Content-Type': 'application/json',
  'X-Request-ID': generateUUID() // optional, for tracing
}
```

**Handling responses:**
- 2xx: Success. Return data.
- 4xx: Client error. Show user-friendly message from backend error.message.
- 5xx: Server error. Show "Something went wrong" + retry option.

**Rate limiting:**
- Check `X-RateLimit-*` headers if returned.
- Show toast if rate limited: "Too many requests. Please try again in X seconds."

---

## 31. Security & best practices

| Area | Practice |
|------|----------|
| **Token storage** | Store in secure httpOnly cookie or secure storage (not localStorage if possible). |
| **CSRF** | Use CSRF tokens if backend requires. Send in headers or hidden form fields. |
| **XSS prevention** | Sanitize user input before display. Use DOMPurify or framework escaping. |
| **API calls** | Use HTTPS only. Validate SSL certificates. |
| **Sensitive data** | Never log API keys, tokens, or PII. Use redaction in logs. |
| **Password fields** | Use `type="password"`. Mask input. No autocomplete for sensitive fields unless necessary. |
| **2FA** | OTP input masking. No copy/paste if possible (but allow if UX demands). |

---

## 32. Accessibility (WCAG 2.1 AA)

| Requirement | Implementation |
|-------------|-----------------|
| **Semantic HTML** | Use `<button>`, `<form>`, `<label>`, `<input>`, etc. Avoid `<div>` for interactive elements. |
| **ARIA labels** | Add `aria-label` to icons, icon-only buttons. Use `aria-describedby` for error text. |
| **Keyboard navigation** | Tab order logical. Focus visible (outline). Trap focus in modals. |
| **Color contrast** | Text: 4.5:1 (normal), 3:1 (large). Icons: 3:1 if meaningful. |
| **Alt text** | Images used for content (not decoration) have alt text. |
| **Form labels** | All inputs have `<label>` with `for` or wrapped input. |
| **Error messages** | Associated with input via `aria-describedby`. |
| **Loading states** | Use `aria-busy` or `aria-live` region for dynamic updates. |
| **Skip links** | Optional: skip to main content link for keyboard users. |

---

## 33. Performance & optimization

| Technique | Why |
|-----------|-----|
| **Code splitting** | Load route-specific bundles on demand. Reduces initial load. |
| **Image optimization** | Compress, use WebP, lazy load below fold. |
| **Caching** | Use React Query cache + localStorage for rarely-changing data. |
| **Debouncing** | Recipient search, rate quote input to reduce API calls. |
| **Memoization** | Memoize expensive components (e.g. large lists) with `React.memo`. |
| **Tree shaking** | Import only what's needed (e.g. specific lodash functions). |
| **Minification & compression** | gzip/brotli compression on server. Minify JS/CSS. |

**Metrics to monitor:**
- Core Web Vitals: LCP (<2.5s), FID (<100ms), CLS (<0.1).
- Time to Interactive (TTI).
- Lighthouse score (target: >85).

---

## 34. Testing strategy

| Layer | Tools | Examples |
|-------|-------|----------|
| **Unit** | Jest, React Testing Library | Component rendering, form validation, utility functions. |
| **Integration** | React Testing Library, MSW | Form submission flow, data fetching, API mocking. |
| **E2E** | Cypress or Playwright | Full user journeys: sign in → transfer → confirm. |
| **Visual** | Percy, Chromatic | Screenshot regression testing. |
| **Accessibility** | axe, jest-axe | Automated a11y checks in unit/integration tests. |

**Coverage target:** >80% for critical paths (auth, transactions, KYC).

---

## 35. Error scenarios & edge cases

| Scenario | Handling |
|----------|----------|
| **No internet** | Show banner: "No connection." Retry button. Auto-retry when online. |
| **API down** | Show error: "Service unavailable. Try again later." |
| **Auth expired** | Redirect to sign-in. Offer to re-authenticate. |
| **Insufficient balance** | On mint/burn/transfer form: show error + link to mint/burn. |
| **Rate limit hit** | Show countdown timer. Disable submit button until reset. |
| **File upload failed** | Show error reason (e.g. "File too large"). Retry option. |
| **Form timeout** | After 5 mins: show "Session expired" + offer to retry. |
| **Invalid recipient** | Show error: "Recipient not found." Suggest search. |
| **KYC rejected** | Show reason (if provided by backend). Link to reapply or support. |
| **Empty lists** | Show empty state illustration + CTA (e.g. "No transfers yet. Send one now."). |

---

## 36. Onboarding flow

**First-time user:**
1. Sign in → Dashboard (empty or with info banner).
2. Prompt: "Welcome to ACBU! Next step: Complete KYC."
3. Link to KYC start page.
4. After KYC approved: Prompt to mint ACBU.
5. After mint: Show tutorial or guide to send/receive/savings.

**Optional tutorial pages:**
- "What is ACBU?" (brief explanation, rationale).
- "How to receive" (show address, QR, memo).
- "How to send" (fill form, confirm, done).
- "Savings explained" (terms, yield, withdrawal).

**Suggested components:**
- `OnboardingBanner` (dismissible).
- `TutorialStep` (step indicator + content + buttons).
- `InfoTooltip` (hover/click for more context).

---

## 37. Mobile-first considerations

*See also [MOBILE_FIRST_REFACTOR.md](MOBILE_FIRST_REFACTOR.md) and [MOBILE_OPTIMIZATION_COMPLETE.md](MOBILE_OPTIMIZATION_COMPLETE.md).*

| Issue | Solution |
|-------|----------|
| **Small screens** | Stack vertically. Use full width for inputs. Collapse nav to hamburger. |
| **Touch targets** | Buttons ≥44x44px. Spacing between taps ≥8px. |
| **Scrolling** | Long forms: break into steps or expandable sections. Modals: full-height on mobile. |
| **Copy-paste** | QR code for Stellar addresses. One-tap copy buttons. |
| **Keyboard** | Virtual keyboard on mobile: use `type="tel"` for amounts, `type="email"` for emails. |
| **Offline** | Cache last state. Show "offline" banner. Queue actions if connectivity returns. |
| **Performance** | Lazy load images. Limit animations. Use system fonts. |

---

## 38. Internationalization (i18n)

**Approach:** Use i18next or similar.

**Key areas:**
- **UI text:** All labels, buttons, messages translated.
- **Dates/times:** Locale-aware formatting (e.g. DD/MM/YYYY vs MM/DD/YYYY).
- **Numbers:** Decimal separators, thousands grouping.
- **Currency:** Display ACBU, USD, GHS, etc. with local formatting.
- **RTL:** If supporting Arabic, Amharic, etc., mirror layout.

**Language files structure:**
```
/locales
├── en.json
├── es.json
├── fr.json
└── am.json
```

**Usage:**
```javascript
import { useTranslation } from 'i18next';

function SendForm() {
  const { t } = useTranslation();
  return <label>{t('send.recipientLabel')}</label>;
}
```

---

## 39. Analytics & monitoring

**Recommended tools:** Mixpanel, Amplitude, or custom backend.

| Event | Tracking |
|-------|----------|
| **User sign-in** | Success, failure reason. |
| **KYC submission** | Started, completed, rejected. |
| **Mint/Burn** | Amount, currency, success/failure. |
| **Transfer sent** | Recipient type (alias/address), amount, success/failure. |
| **Savings deposit** | Term, amount, success/failure. |
| **Page view** | Route, time spent. |
| **Error** | Error code, page context, user action. |

**Goals:**
- Track funnel completion (sign-up → KYC → first mint → first transfer).
- Identify drop-off points.
- Monitor error rates.
- Measure feature adoption (e.g. savings vs lending vs P2P).

---

## 40. Deployment & environments

| Environment | Purpose | URL |
|-------------|---------|-----|
| **Development** | Local dev & feature branches. | `localhost:3000` |
| **Staging** | QA & pre-production testing. | `staging.acbu.app` |
| **Production** | Live user-facing app. | `app.acbu.app` |

**Deployment checklist:**
- [ ] All tests passing.
- [ ] No console errors or warnings.
- [ ] Lighthouse score >85.
- [ ] Accessibility audit passed.
- [ ] Environment variables set correctly.
- [ ] API endpoints verified for environment.
- [ ] Error logging configured.
- [ ] Analytics events firing.
- [ ] Performance monitoring active.

**Rollback plan:**
- Keep previous version deployed or in registry.
- If critical bug detected, revert to last stable version.
- Use feature flags for gradual rollouts.

---

## 41. Known gaps & next steps

**Partially implemented:**
- ✅ Auth (signin, 2FA, signout).
- ✅ User profile & settings.
- ✅ KYC flow (basic).
- ✅ Mint & burn (core).
- ✅ P2P transfers.
- ✅ Rates & reserves.
- ⚠️ Savings (vault contract exists; UI needs implementation).
- ⚠️ Lending (pool contract exists; UI needs implementation).
- ⚠️ Salary (backend stub; disburse/schedule UI needed).
- ⚠️ Enterprise (backend stub; bulk transfer + treasury UI needed).
- ⚠️ Gateway (escrow implemented; charge UI needs testing).
- ⚠️ Bills (backend stub; catalog + pay UI needed).
- ⚠️ Validator KYC (backend implemented; task queue UI needs work).

**Next priorities:**
1. **Savings & Lending UX:** Finalize deposit/withdraw flows. Test with contracts.
2. **Validator dashboard:** Implement task queue, document viewer, approval workflow.
3. **SME & International:** Build segment-specific dashboards and routing.
4. **Merchant gateway:** Test charge creation and release/refund flows.
5. **Mobile optimization:** Conduct mobile QA. Optimize performance on 3G.
6. **Analytics:** Deploy event tracking. Analyze user funnels.
7. **Accessibility:** Full WCAG 2.1 AA audit with assistive tech testing.
8. **Documentation:** Keep this map up-to-date as features launch.

---

## 42. Maintenance & versioning

**Frontend versioning:** Semantic versioning (e.g. 1.0.0, 1.1.0, 2.0.0).

**Release cycle:**
- **Patch (1.0.x):** Bug fixes, minor UX tweaks. Released as needed.
- **Minor (1.x.0):** New features, non-breaking API changes. Monthly or biweekly.
- **Major (x.0.0):** Breaking changes, major redesign. Quarterly or as needed.

**Deprecation policy:**
- Announce deprecated features 2–4 weeks before removal.
- Show in-app banner or warning in docs.
- Provide migration guide.

**Change log:**
- Maintain CHANGELOG.md with all releases.
- Format: date, version, features added, bugs fixed, breaking changes.

---

## 43. FAQ & common patterns

**Q: How do I handle concurrent requests?**  
A: Use React Query or similar to deduplicate requests. Ignore duplicate requests while one is in flight.

**Q: How do I validate OTP input?**  
A: Use `react-otp-input` or custom: 6-digit masked input, auto-focus next, auto-submit on complete.

**Q: How do I show real-time balance updates?**  
A: Poll with `setInterval` every 10–30s, or use WebSocket if backend supports it. Use React Query to cache.

**Q: How do I handle errors from the API in forms?**  
A: Display backend error.message in a toast or banner. Highlight relevant fields if error specifies field.

**Q: How do I support offline mode?**  
A: Cache recent data in localStorage. Show "offline" banner. Queue actions (transfers, KYC submissions) and retry when online.

**Q: How do I track user engagement?**  
A: Emit analytics events for key actions (sign-in, KYC, mint, transfer, etc.). Use Mixpanel or custom logger.

**Q: Should I use Redux or Context?**  
A: Context for auth/theme. Redux or Zustand for complex app state. React Query for server state.

---

## 44. Resources & links

- **API Reference:** [API_SPECIFICATION.MD](../../TECHNICAL/API_SPECIFICATION.MD)
- **Contracts:** [SMART_CONTRACT_SPEC.MD](../../DOCS/SMART_CONTRACT_SPEC.MD)
- **Design Guide:** [FRONTEND_AND_UI.md](FRONTEND_AND_UI.md)
- **Architecture:** [ARCHITECTURE.MD](../../TECHNICAL/ARCHITECTURE.MD)
- **Contributing:** [CONTRIBUTING_GUIDE.MD](../../CONTRIBUTING_GUIDE.MD)
- **Database Schema:** [DATABASE_SCHEMA.MD](../../TECHNICAL/DATABASE_SCHEMA.MD)

---

**Last updated:** June 18, 2026  
**Status:** Ongoing (sections 1–22 stable; 23–44 framework for implementation)  
**Maintained by:** Frontend team

