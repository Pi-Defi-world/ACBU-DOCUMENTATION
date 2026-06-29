# MVP issues — MASTER INDEX

This folder consolidates **~200** MVP flaws and improvements across:

- Backend (`acbu-backend`): **75** items — [backend.md](./backend.md)
- Frontend (`acbu-frontend`): **77** items — [frontend.md](./frontend.md)
- Smart contracts (`acbu-smart-contract`): **60** items — [contracts.md](./contracts.md)

**Total:** 212

## How to use this backlog

1. Triage **Critical** items first (loss of funds / auth bypass / false confirmations).
2. Create tickets in your tracker; keep the **Evidence** path as the source anchor.
3. When an item is fixed, prefer **linking the PR** in your tracker (avoid editing this backlog as a changelog).

## Catalog sources & deduping

- **Live triage-ready catalogs in this folder (canonical working backlog):**
  - [backend.md](./backend.md) — 75 items (B-001..B-075)
  - [frontend.md](./frontend.md) — 77 items (F-001..F-077)
  - [contracts.md](./contracts.md) — 60 items (C-001..C-060)
- **Durable long-form references in `PROJECT/issues/`** (consolidated mirrors, see CHANGELOG):
  - [../PROJECT/issues/CONTRACTS_ISSUES.md](../PROJECT/issues/CONTRACTS_ISSUES.md) — consolidated into the canonical format in [PR #22](https://github.com/Pi-Defi-world/ACBU-DOCUMENTATION/pull/22); single source of truth, with Resolution Tracker (Section 8), Top Remediations ship-safety list, and Cross-Reference Map.
  - [../PROJECT/issues/BACKEND_ISSUES.md](../PROJECT/issues/BACKEND_ISSUES.md) — consolidated into the canonical format in [PR #23](https://github.com/Pi-Defi-world/ACBU-DOCUMENTATION/pull/23).
  - [../PROJECT/issues/FRONTEND_ISSUES.md](../PROJECT/issues/FRONTEND_ISSUES.md) — consolidated into the canonical format in [PR #25](https://github.com/Pi-Defi-world/ACBU-DOCUMENTATION/pull/25); expanded to all 77 canonical entries (F-001..F-077) via follow-up canonical-ID migrations in [PR #28](https://github.com/Pi-Defi-world/ACBU-DOCUMENTATION/pull/28) (F-066 Shared Skeleton primitive, Medium), [PR #29](https://github.com/Pi-Defi-world/ACBU-DOCUMENTATION/pull/29) (F-071 Toast removal delay, Medium), [PR #30](https://github.com/Pi-Defi-world/ACBU-DOCUMENTATION/pull/30) (F-076 Frontend `request()` no default timeout, High), and [PR #31](https://github.com/Pi-Defi-world/ACBU-DOCUMENTATION/pull/31) (F-067..F-077 batch: 3 Medium + 6 Low). Severity distribution: **5 Critical / 15 High / 28 Medium / 29 Low** = 77. Single source of truth for frontend issues, mirroring the contracts and backend catalogs.
- **Truly historical (read-only):** Repo-root `ISSUES.md` / `acbu-smart-contract/ISSUES.md` may also exist; treat as historical unless actively maintained.

## Top 20 (ship-safety order)

_Per-item fix status is tracked in the [Resolution Tracker](../PROJECT/issues/CONTRACTS_ISSUES.md#resolution-tracker-fix-status) added in [PR #22](https://github.com/Pi-Defi-world/ACBU-DOCUMENTATION/pull/22), the analogous [backend Resolution Tracker](../PROJECT/issues/BACKEND_ISSUES.md#resolution-tracker-fix-status) in [PR #23](https://github.com/Pi-Defi-world/ACBU-DOCUMENTATION/pull/23), and the [frontend Resolution Tracker](../PROJECT/issues/FRONTEND_ISSUES.md#resolution-tracker-fix-status) in [PR #25](https://github.com/Pi-Defi-world/ACBU-DOCUMENTATION/pull/25). The Top 20 below is the ship-safety triage ordering; the corresponding canonical IDs in each catalog's Resolution Tracker are the canonical record of open/fixed status._

1. **C-001** Escrow `release` missing auth (contracts) — [tracker](../PROJECT/issues/CONTRACTS_ISSUES.md#resolution-tracker-fix-status)
2. **C-005** Savings vault missing `require_auth` (contracts) — [tracker](../PROJECT/issues/CONTRACTS_ISSUES.md#resolution-tracker-fix-status)
3. **C-006** Lending pool missing `require_auth` (contracts) — [tracker](../PROJECT/issues/CONTRACTS_ISSUES.md#resolution-tracker-fix-status)
4. **C-002** `mint_from_fiat` unrestricted (contracts) — [tracker](../PROJECT/issues/CONTRACTS_ISSUES.md#resolution-tracker-fix-status)
5. **F-002** API key in `sessionStorage` (frontend auth) — [tracker](../PROJECT/issues/FRONTEND_ISSUES.md#resolution-tracker-fix-status)
6. **F-005** Plaintext wallet storage path exists (frontend wallet) — [tracker](../PROJECT/issues/FRONTEND_ISSUES.md#resolution-tracker-fix-status)
7. **F-001** Next.js `ignoreBuildErrors` (frontend build safety) — [tracker](../PROJECT/issues/FRONTEND_ISSUES.md#resolution-tracker-fix-status)
8. **F-014** Send success dialog clears amount before dialog (frontend ux-money) — [tracker](../PROJECT/issues/FRONTEND_ISSUES.md#resolution-tracker-fix-status)
9. **F-025** Balances shown as em dash placeholders (frontend data) — [tracker](../PROJECT/issues/FRONTEND_ISSUES.md#resolution-tracker-fix-status)
10. **F-019** Currency page simulates operations (frontend feature) — [tracker](../PROJECT/issues/FRONTEND_ISSUES.md#resolution-tracker-fix-status)
11. **F-022** Security settings placeholder (frontend settings) — [tracker](../PROJECT/issues/FRONTEND_ISSUES.md#resolution-tracker-fix-status)
12. **F-051** Signup passcode minimum too weak (frontend auth) — [tracker](../PROJECT/issues/FRONTEND_ISSUES.md#resolution-tracker-fix-status)
13. **F-063** Environment base URL misconfiguration footgun (frontend config) — [tracker](../PROJECT/issues/FRONTEND_ISSUES.md#resolution-tracker-fix-status)
14. **F-076** Frontend `request()` helper has no default timeout (frontend api) — [tracker](../PROJECT/issues/FRONTEND_ISSUES.md#resolution-tracker-fix-status)
15. **F-020** Bills payment console-only (frontend feature) — [tracker](../PROJECT/issues/FRONTEND_ISSUES.md#resolution-tracker-fix-status)
16. **F-015** Savings deposit not wired (frontend feature) — [tracker](../PROJECT/issues/FRONTEND_ISSUES.md#resolution-tracker-fix-status)
17. **B-005** Webhook signature verification fail-open risk (backend) — [tracker](../PROJECT/issues/BACKEND_ISSUES.md#resolution-tracker-fix-status)
18. **B-006** Recovery unlock single-factor risk (backend) — [tracker](../PROJECT/issues/BACKEND_ISSUES.md#resolution-tracker-fix-status)
19. **C-004** `verify_reserves` wrong source of truth (contracts) — [tracker](../PROJECT/issues/CONTRACTS_ISSUES.md#resolution-tracker-fix-status)
20. **C-003** Escrow ID collision (contracts) — [tracker](../PROJECT/issues/CONTRACTS_ISSUES.md#resolution-tracker-fix-status)

---

## CHANGELOG

This is the master index's own changelog. Catalog-level changes that affect this index are recorded here in **strict reverse-chronological order** (newest first); per-item fix tickets claim their own PR history via the linked Resolution Trackers.

- **2026-06 — F-067..F-077 batch canonical-ID migration ([PR #31](https://github.com/Pi-Defi-world/ACBU-DOCUMENTATION/pull/31)).** Single follow-up PR absorbed the remaining 9 `(new)` Section 7 findings (legacy #55, #56, #68, #69, #72, #73, #74, #75, #77) into canonical F-067..F-077 in `issues/frontend.md`:
  - **3 → 🟡 Medium** (data-validation correctness / privacy): F-067 (Contact inputs no maxLength, frontend/forms), F-068 (Profile inputs no maxLength/format validation, frontend/forms), F-074 (Silent username normalization on signup, frontend/auth).
  - **6 → 🟢 Low** (perf / nav-pattern / hygiene): F-069 (Icons recreated each render, frontend/perf), F-070 (Menu items use `router.push` instead of Link, frontend/nav), F-072 (Mobile detection treats "unknown" as desktop, frontend/compat), F-073 (Post-KYC upload uncleaned setTimeout, frontend/hygiene), F-075 (Auto-fill heuristic requires length ≥ 56, frontend/ux), F-077 (`/p2p` client-side redirect only, frontend/nav).
  - Severity distribution bumped **5 / 15 / 25 / 23 = 68** (post-#28..#30) → **5 / 15 / 28 / 29 = 77** — completing the `(new)` Section 7 backlog for `PROJECT/issues/FRONTEND_ISSUES.md`.
  - Top 20 (ship-safety order) is intentionally **unchanged** in this follow-up: the F-067..F-077 batch added 3 Medium + 6 Low entries — none would rank above the existing Critical / High entries already on the ship-safety list, so re-sorting would reduce clarity for marginal gain.
  - All 12 Section 7 `(new)` markers have now been adopted into canonical IDs F-066..F-077; the backlog of `_proposed_` follow-ups is now zero. Future audit findings will create fresh `(new)` rows for the next canonical-ID promotion cycle.
- **2026-06 — F-066/F-071/F-076 follow-up canonical-ID migrations ([PR #28](https://github.com/Pi-Defi-world/ACBU-DOCUMENTATION/pull/28) · [PR #29](https://github.com/Pi-Defi-world/ACBU-DOCUMENTATION/pull/29) · [PR #30](https://github.com/Pi-Defi-world/ACBU-DOCUMENTATION/pull/30)).** Three follow-up canonical-ID PRs migrated legacy Section 7 `(new)` findings out of the nested `(new — propose F-XXX)` tags and into the canonical `/issues/frontend.md` triage-ready catalog:
  - [PR #28](https://github.com/Pi-Defi-world/ACBU-DOCUMENTATION/pull/28) promoted Section 7 legacy #49 ("Empty and loading states inconsistent — missing `Skeleton` primitive") into canonical **F-066** — _Inconsistent loading / empty state skeletons across pages_ — 🟡 Medium, `frontend/components`.
  - [PR #29](https://github.com/Pi-Defi-world/ACBU-DOCUMENTATION/pull/29) promoted Section 7 legacy #71 ("Toast removal delay is ~17 minutes; `TOAST_REMOVE_DELAY = 1000000`") into canonical **F-071** — _Toast removal delay is ~17 minutes_ — 🟡 Medium, `frontend/ux`.
  - [PR #30](https://github.com/Pi-Defi-world/ACBU-DOCUMENTATION/pull/30) promoted Section 7 legacy #76 ("API fetch has no timeout; Frontend `request()` uses `fetch` with no default timeout") into canonical **F-076** — _Frontend `request()` helper has no default timeout_ — 🟠 High, `frontend/api`.
  - Severity distribution for frontend bumped from **5 / 14 / 23 / 23 = 65** to **5 / 15 / 25 / 23 = 68**.
  - **Top 20 (ship-safety order) reorder:** F-076 admitted to the High frontend cluster between F-063 and F-020 (its evidence — hung `fetch` in a financial app causing indefinite spinners, mobile battery pressure, and retry amplification — warrants ship-safety triage alongside F-063); F-004 — which until now carried the closing slot _without_ a per-item tracker pointer, in order to keep the front end's 5-Critical cluster visibly accounted-for on the master index — was dropped to preserve one-pointer-per-row. All 20 Top-20 entries now carry an `…#resolution-tracker-fix-status` pointer.
  - Master index intro line updated: `Frontend (`acbu-frontend`)` 65 → 68 items; `Total` 200 → 203.
  - Outstanding `(new)` items in Section 7 at this snapshot: 9 (legacy #55, #56, #68, #69, #72, #73, #74, #75, #77) proposed for canonical F-067–F-070 / F-072–F-075 / F-077 promotion in a single batched follow-up (see PR #31).
- **2026-06 — Frontend catalog consolidation ([PR #25](https://github.com/Pi-Defi-world/ACBU-DOCUMENTATION/pull/25)).** `PROJECT/issues/FRONTEND_ISSUES.md` is now the **single source of truth** for frontend issues, mirroring the contracts and backend catalogs. Coverage aligned to all 65 canonical entries (F-001..F-065) with stable IDs in the severity/area/evidence/impact/fix/acceptance format used by `issues/frontend.md`. Severity distribution: **5 Critical / 14 High / 23 Medium / 23 Low**.
  - New structural sections in `PROJECT/issues/FRONTEND_ISSUES.md`: Summary Table, Severity Counts, **Resolution Tracker (Section 8)**, Top Remediations (ship-safety order), Cross-Reference Map, legacy 77-item cross-reference, Maintenance notes.
  - The Top 20 list below now anchors per-item frontend trackers (Critical `F-002`/`F-005`/`F-001` for wallet + auth, plus top-High `F-014`/`F-025`/`F-019`/`F-022`/`F-051`/`F-063`/`F-020`/`F-015`) to the [frontend Resolution Tracker](../PROJECT/issues/FRONTEND_ISSUES.md#resolution-tracker-fix-status) so reviewers can confirm open/fixed status at a glance. Legacy `F-004` critical retains its slot for completeness without a per-item pointer (next PR can add full per-item coverage on remaining Critical IDs).
- **2026-06 — MASTER_INDEX.md refresh ([PR #24](https://github.com/Pi-Defi-world/ACBU-DOCUMENTATION/pull/24)).** Replaced the older `Deduping against older docs` section with a `Catalog sources & deduping` section that distinguishes three states — **canonical triage-ready** (this folder), **durable long-form** (`PROJECT/issues/`, consolidated), and **truly historical** (read-only). Catalog links now cover contracts ([PR #22](https://github.com/Pi-Defi-world/ACBU-DOCUMENTATION/pull/22)), backend ([PR #23](https://github.com/Pi-Defi-world/ACBU-DOCUMENTATION/pull/23)), and (as of [PR #25](https://github.com/Pi-Defi-world/ACBU-DOCUMENTATION/pull/25)) frontend; forward link for the frontend catalog was raised from "pending follow-up PR" to a live link in PR #25. The Top 20 list anchors per-item trackers to the **Resolution Tracker (Section 8) of each consolidated catalog**.
- **2026-06 — Backend catalog consolidation ([PR #23](https://github.com/Pi-Defi-world/ACBU-DOCUMENTATION/pull/23)).** `PROJECT/issues/BACKEND_ISSUES.md` consolidated into the canonical severity/area/evidence/impact/fix/acceptance format with all 75 entries (B-001..B-075); severity distribution 2 Critical / 21 High / 38 Medium / 14 Low. Added an equivalent [Resolution Tracker](../PROJECT/issues/BACKEND_ISSUES.md#resolution-tracker-fix-status) (Section 8), Top Remediations ship-safety list, Cross-Reference Map, and legacy 56-item cross-reference. The legacy `(new)` items in Section 7 propose follow-up canonical IDs (`B-076+`) once a separate PR promotes them.
- **2026-06 — Contracts catalog consolidation ([PR #22](https://github.com/Pi-Defi-world/ACBU-DOCUMENTATION/pull/22)).** `PROJECT/issues/CONTRACTS_ISSUES.md` is the **single source of truth** for smart-contract issues. Coverage expanded from a 34-item flat list to all 60 canonical entries (C-001..C-060) with stable IDs in the severity/area/evidence/impact/fix/acceptance format used by `issues/contracts.md`. Severity distribution: 6 Critical / 21 High / 24 Medium / 9 Low. New structural sections: Summary Table, Severity Counts, **Resolution Tracker (Section 8)**, Top Remediations (ship-safety order), Cross-Reference Map, legacy 34-item cross-reference, Maintenance notes.

