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

## Deduping against older docs

- Legacy consolidated lists: [../PROJECT/issues/BACKEND_ISSUES.md](../PROJECT/issues/BACKEND_ISSUES.md), [../PROJECT/issues/FRONTEND_ISSUES.md](../PROJECT/issues/FRONTEND_ISSUES.md), [../PROJECT/issues/CONTRACTS_ISSUES.md](../PROJECT/issues/CONTRACTS_ISSUES.md).
- Repo-root `ISSUES.md` / `acbu-smart-contract/ISSUES.md` may also exist; treat them as **historical** unless actively maintained.

## Top 20 (ship-safety order)

1. C-001 Escrow `release` missing auth (contracts)
2. C-005 Savings vault missing `require_auth` (contracts)
3. C-006 Lending pool missing `require_auth` (contracts)
4. C-002 `mint_from_fiat` unrestricted (contracts)
5. F-001 Next.js `ignoreBuildErrors` (frontend build safety)
6. F-002 API key in `sessionStorage` (frontend)
7. F-003 Passcode stored in `sessionStorage` for wallet decrypt path (frontend)
8. F-004 Wallet secret 'encryption' is reversible (frontend)
9. F-005 Plaintext wallet storage path exists (frontend)
10. B-005 Webhook signature verification fail-open risk (backend)
11. B-006 Recovery unlock single-factor risk (backend)
12. B-003 USDC job infinite requeue risk (backend)
13. B-001 Basket deposit USD limit proxy wrong (backend)
14. B-002 Floating `Number()` on money paths (backend)
15. C-004 `verify_reserves` wrong supply source (contracts)
16. C-003 Escrow ID collision (contracts)
17. B-010 ReserveTracker DB vs chain supply drift (backend)
18. B-009 Burn fee policy threshold correctness (backend)
19. F-015 Savings deposit not wired (frontend)
20. B-022 Bills catalog/pay not implemented (backend) + F-020 Bills payment UI console-only (frontend)

