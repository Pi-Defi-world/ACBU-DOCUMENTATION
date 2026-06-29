# MVP issues — MASTER INDEX

This folder consolidates **~200** MVP flaws and improvements across:

- Backend (`acbu-backend`): **75** items — [backend.md](./backend.md)
- Frontend (`acbu-frontend`): **65** items — [frontend.md](./frontend.md)
- Smart contracts (`acbu-smart-contract`): **60** items — [contracts.md](./contracts.md)

**Total:** 200

## How to use this backlog

1. Triage **Critical** items first (loss of funds / auth bypass / false confirmations).
2. Create tickets in your tracker; keep the **Evidence** path as the source anchor.
3. When an item is fixed, prefer **linking the PR** in your tracker (avoid editing this backlog as a changelog).

## Catalog sources & deduping

- **Live triage-ready catalogs in this folder (canonical working backlog):**
  - [backend.md](./backend.md) — 75 items (B-001..B-075)
  - [frontend.md](./frontend.md) — 65 items (F-001..F-065)
  - [contracts.md](./contracts.md) — 60 items (C-001..C-060)
- **Durable long-form references in `PROJECT/issues/`** (consolidated mirrors, see CHANGELOG):
  - [../PROJECT/issues/CONTRACTS_ISSUES.md](../PROJECT/issues/CONTRACTS_ISSUES.md) — consolidated into the canonical format in [PR #22](https://github.com/Pi-Defi-world/ACBU-DOCUMENTATION/pull/22); single source of truth, with Resolution Tracker (Section 8), Top Remediations ship-safety list, and Cross-Reference Map.
  - [../PROJECT/issues/BACKEND_ISSUES.md](../PROJECT/issues/BACKEND_ISSUES.md) — consolidated into the canonical format in [PR #23](https://github.com/Pi-Defi-world/ACBU-DOCUMENTATION/pull/23).
  - [../PROJECT/issues/FRONTEND_ISSUES.md](../PROJECT/issues/FRONTEND_ISSUES.md) — legacy list; same pattern pending a follow-up PR.
- **Truly historical (read-only):** Repo-root `ISSUES.md` / `acbu-smart-contract/ISSUES.md` may also exist; treat as historical unless actively maintained.

## Top 20 (ship-safety order)

_Per-item fix status is tracked in the [Resolution Tracker](../PROJECT/issues/CONTRACTS_ISSUES.md#resolution-tracker-fix-status) added in [PR #22](https://github.com/Pi-Defi-world/ACBU-DOCUMENTATION/pull/22) (plus the analogous [backend Resolution Tracker](../PROJECT/issues/BACKEND_ISSUES.md#resolution-tracker-fix-status) in [PR #23](https://github.com/Pi-Defi-world/ACBU-DOCUMENTATION/pull/23)). The Top 20 below is the ship-safety triage ordering; the corresponding canonical IDs in each contract/PR's Resolution Tracker are the canonical record of open/fixed status._

1. **C-001** Escrow `release` missing auth (contracts) — [tracker](../PROJECT/issues/CONTRACTS_ISSUES.md#resolution-tracker-fix-status)
2. **C-005** Savings vault missing `require_auth` (contracts) — [tracker](../PROJECT/issues/CONTRACTS_ISSUES.md#resolution-tracker-fix-status)
3. **C-006** Lending pool missing `require_auth` (contracts) — [tracker](../PROJECT/issues/CONTRACTS_ISSUES.md#resolution-tracker-fix-status)
4. **C-002** `mint_from_fiat` unrestricted (contracts) — [tracker](../PROJECT/issues/CONTRACTS_ISSUES.md#resolution-tracker-fix-status)
5. **F-001** Next.js `ignoreBuildErrors` (frontend build safety)
6. **F-002** API key in `sessionStorage` (frontend)
7. **F-003** Passcode stored in `sessionStorage` for wallet decrypt path (frontend)
8. **F-004** Wallet secret 'encryption' is reversible (frontend)
9. **F-005** Plaintext wallet storage path exists (frontend)
10. **B-005** Webhook signature verification fail-open risk (backend) — [tracker](../PROJECT/issues/BACKEND_ISSUES.md#resolution-tracker-fix-status)
11. **B-006** Recovery unlock single-factor risk (backend) — [tracker](../PROJECT/issues/BACKEND_ISSUES.md#resolution-tracker-fix-status)
12. **B-003** USDC job infinite requeue risk (backend) — [tracker](../PROJECT/issues/BACKEND_ISSUES.md#resolution-tracker-fix-status)
13. **B-001** Basket deposit USD limit proxy wrong (backend) — [tracker](../PROJECT/issues/BACKEND_ISSUES.md#resolution-tracker-fix-status)
14. **B-002** Floating `Number()` on money paths (backend) — [tracker](../PROJECT/issues/BACKEND_ISSUES.md#resolution-tracker-fix-status)
15. **C-004** `verify_reserves` wrong source of truth (contracts) — [tracker](../PROJECT/issues/CONTRACTS_ISSUES.md#resolution-tracker-fix-status)
16. **C-003** Escrow ID collision (contracts) — [tracker](../PROJECT/issues/CONTRACTS_ISSUES.md#resolution-tracker-fix-status)
17. **B-010** ReserveTracker DB vs chain supply drift (backend) — [tracker](../PROJECT/issues/BACKEND_ISSUES.md#resolution-tracker-fix-status)
18. **B-009** Burn fee policy threshold correctness (backend) — [tracker](../PROJECT/issues/BACKEND_ISSUES.md#resolution-tracker-fix-status)
19. **F-015** Savings deposit not wired (frontend)
20. **B-022** Bills catalog/pay not implemented (backend) & **F-020** Bills payment UI console-only (frontend)

---

## CHANGELOG

This is the master index's own changelog. Catalog-level changes that affect this index are recorded here in **strict reverse-chronological order** (newest first); per-item fix tickets claim their own PR history via the linked Resolution Trackers.

- **2026-06 (PR TBD) — MASTER_INDEX.md refresh.** Replaced the older `Deduping against older docs` section with a `Catalog sources & deduping` section that distinguishes three states — **canonical triage-ready** (this folder), **durable long-form** (`PROJECT/issues/`, consolidated), and **truly historical** (read-only). Forward links to [PR #22](https://github.com/Pi-Defi-world/ACBU-DOCUMENTATION/pull/22) (contracts consolidation — what formally elevates `PROJECT/issues/CONTRACTS_ISSUES.md` to single-source-of-truth) and [PR #23](https://github.com/Pi-Defi-world/ACBU-DOCUMENTATION/pull/23) (backend consolidation) are now provided. The Top 20 list below anchors per-item trackers to the **Resolution Tracker (Section 8) of each consolidated catalog**, so reviewers can confirm open/fixed status at a glance. `PROJECT/issues/FRONTEND_ISSUES.md` is the only remaining legacy list pending its follow-up PR.
- **2026-06 — Backend catalog consolidation ([PR #23](https://github.com/Pi-Defi-world/ACBU-DOCUMENTATION/pull/23)).** `PROJECT/issues/BACKEND_ISSUES.md` consolidated into the canonical severity/area/evidence/impact/fix/acceptance format with all 75 entries (B-001..B-075); severity distribution 2 Critical / 21 High / 38 Medium / 14 Low. Added an equivalent [Resolution Tracker](../PROJECT/issues/BACKEND_ISSUES.md#resolution-tracker-fix-status) (Section 8), Top Remediations ship-safety list, Cross-Reference Map, and legacy 56-item cross-reference. The legacy `(new)` items in Section 7 propose follow-up canonical IDs (`B-076+`) once a separate PR promotes them.
- **2026-06 — Contracts catalog consolidation ([PR #22](https://github.com/Pi-Defi-world/ACBU-DOCUMENTATION/pull/22)).** `PROJECT/issues/CONTRACTS_ISSUES.md` is now the **single source of truth** for smart-contract issues. Coverage expanded from a 34-item flat list to all 60 canonical entries (C-001..C-060) with stable IDs in the severity/area/evidence/impact/fix/acceptance format used by `issues/contracts.md`. Severity distribution: 6 Critical / 21 High / 24 Medium / 9 Low.
  - New structural sections in `PROJECT/issues/CONTRACTS_ISSUES.md`: Summary Table, Severity Counts, **Resolution Tracker (Section 8)**, Top Remediations (ship-safety order), Cross-Reference Map, legacy 34-item cross-reference, Maintenance notes.
  - The Top 20 list below references the [Resolution Tracker](../PROJECT/issues/CONTRACTS_ISSUES.md#resolution-tracker-fix-status) so reviewers can confirm open/fixed status at a glance; the same pattern is duplicated to the consolidated backend catalog (above) and (planned) frontend catalog.
- **_(Future) Frontend catalog consolidation._** `PROJECT/issues/FRONTEND_ISSUES.md` is currently still a legacy flat list. The same consolidation pattern (canonical format + Summary Table + Severity Counts + Resolution Tracker + Top Remediations + Cross-Reference Map) is planned as a follow-up PR; once merged, this CHANGELOG entry will be updated with the PR number and frontend tracker pointers will be added to the Top 20 list above.

