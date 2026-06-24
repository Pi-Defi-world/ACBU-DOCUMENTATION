# Smart Contracts — Known Issues

This document is the **single source of truth** for known issues in the ACBU Soroban smart contracts (`acbu_minting`, `acbu_burning`, `acbu_oracle`, `acbu_reserve_tracker`, `acbu_savings_vault`, `acbu_lending_pool`, `acbu_escrow`, plus the `shared` crate).

> **How to use**
>
> - Each issue is structured as: **severity · area · evidence · impact · fix direction · acceptance check**.
> - Issues are ordered by severity, then by ID for stable cross-referencing.
> - Provenance: this file consolidates the legacy 34-item list previously at `PROJECT/issues/CONTRACTS_ISSUES.md` and the canonical 60-item catalog at `issues/contracts.md` into one document. Tracking duplicates the working catalog (`issues/contracts.md`) which remains the live triage-ready list; this file is the durable long-form reference.
> - When an item is fixed, prefer **linking the PR** in your tracker and removing the entry here rather than rewriting history.
> - **Severity legend:** 🔴 Critical · 🟠 High · 🟡 Medium · 🟢 Low · ⚪ Trivial

---

## 1. Summary Table

| ID | Severity | Area | Contract / Crate | Title |
|----|----------|------|------------------|-------|
| C-001 | 🔴 Critical | contracts/escrow | `acbu_escrow` | `release` missing authorization |
| C-002 | 🔴 Critical | contracts/minting | `acbu_minting` | `mint_from_fiat` lacks off-chain settlement proof |
| C-003 | 🔴 Critical | contracts/escrow | `acbu_escrow` | Escrow ID key collision |
| C-004 | 🔴 Critical | contracts/reserves | `acbu_reserve_tracker` | `verify_reserves` uses wrong supply source |
| C-005 | 🔴 Critical | contracts/savings | `acbu_savings_vault` | Missing `require_auth` on deposit/withdraw |
| C-006 | 🔴 Critical | contracts/lending | `acbu_lending_pool` | Missing `require_auth` on deposit/withdraw |
| C-007 | 🟠 High | contracts/savings | `acbu_savings_vault` | Term not enforced |
| C-008 | 🟠 High | contracts/minting | `acbu_minting` | `mint_from_fiat` missing max mint bound |
| C-009 | 🟠 High | contracts/burning | `acbu_burning` | Burn basket integer division dust loss |
| C-010 | 🟠 High | contracts/escrow | `acbu_escrow` | Escrow create lacks duplicate guard |
| C-011 | 🟠 High | contracts/minting | `acbu_minting` | Oracle/reserve clients loaded but not called |
| C-012 | 🟠 High | contracts/burning | `acbu_burning` | Oracle/reserve clients loaded but not called |
| C-021 | 🟠 High | contracts/integration | All | Event payload schema drift vs backend listeners |
| C-027 | 🟠 High | contracts/lending | `acbu_lending_pool` | Loan events never emitted / loan logic missing |
| C-028 | 🟠 High | contracts/qa | `acbu_minting/tests` | Missing tests for core mint flows |
| C-034 | 🟠 High | contracts/ops | Multiple | No contract versioning / upgrade story |
| C-035 | 🟠 High | contracts/governance | Mint/burn/savings/lending | Admin pause flags not verified on all state changes |
| C-037 | 🟠 High | contracts/security | All `*_amount` fields | Integer overflow review for all i128 money math |
| C-038 | 🟠 High | contracts/security | Mint/burn | Authorization invocation tree for cross-contract calls |
| C-039 | 🟠 High | contracts/minting | `acbu_minting` | Strict input validation for string IDs |
| C-040 | 🟠 High | contracts/tokenomics | Mint/burn/oracle | Decimal scaling consistency (7 decimals) |
| C-041 | 🟠 High | contracts/oracle | `acbu_oracle` | Staleness: max ledger age for rates |
| C-042 | 🟠 High | contracts/oracle | `acbu_oracle` | Minimum oracle sources for quorum |
| C-050 | 🟠 High | contracts/qa | `acbu_escrow/tests` | Integration tests: escrow full lifecycle |
| C-051 | 🟠 High | contracts/qa | `acbu_savings_vault/tests` | Integration tests: savings lock + interest |
| C-052 | 🟠 High | contracts/qa | `acbu_lending_pool/tests` | Integration tests: lending borrow/repay |
| C-055 | 🟠 High | contracts/authz | `acbu_minting` and others | Access control naming audit (`check_admin_or_user`) |
| C-013 | 🟡 Medium | contracts/build | Mint/burn/reserve tracker | Token WASM import uses zero SHA256 placeholder |
| C-014 | 🟡 Medium | contracts/escrow | `acbu_escrow` | Double `.unwrap().unwrap()` pattern in storage reads |
| C-015 | 🟡 Medium | contracts/savings | `acbu_savings_vault` | Double `.unwrap().unwrap()` pattern |
| C-016 | 🟡 Medium | contracts/lending | `acbu_lending_pool` | Double `.unwrap().unwrap()` pattern |
| C-017 | 🟡 Medium | contracts/oracle | `acbu_oracle` | Outlier detection is a no-op |
| C-018 | 🟡 Medium | contracts/oracle | `acbu_oracle/tests` | Oracle unit test assertion incorrect vs median behavior |
| C-020 | 🟡 Medium | contracts/shared | `shared` | `median` allocates via `to_vec()` in shared util |
| C-024 | 🟡 Medium | contracts/savings | `acbu_savings_vault` | Savings withdraw yield always zero |
| C-025 | 🟡 Medium | contracts/lending | `acbu_lending_pool` | Lending fee rate unused |
| C-026 | 🟡 Medium | contracts/savings | `acbu_savings_vault` | Savings fee rate unused |
| C-032 | 🟡 Medium | contracts/minting | `acbu_minting` | Predictable `transaction_id` in minting |
| C-033 | 🟡 Medium | contracts/reserves | `acbu_reserve_tracker` | Reserve tracker emits no events on updates |
| C-036 | 🟡 Medium | contracts/security | Token transfers inside contracts | Reentrancy class issues via token callbacks |
| C-043 | 🟡 Medium | contracts/governance | Admin-only methods | Emergency multisig for admin operations |
| C-044 | 🟡 Medium | contracts/qa | CI pipeline | Contract size/compute budget regression tests |
| C-046 | 🟡 Medium | contracts/security | Rust dependencies | `cargo audit` / supply chain monitoring |
| C-047 | 🟡 Medium | contracts/docs | `INTEGRATION.md` / `SMART_CONTRACT_SPEC` | Formalize invariants documentation per contract |
| C-048 | 🟡 Medium | contracts/ops | `dummy_token` etc. | Testnet vs mainnet feature flags in code |
| C-054 | 🟡 Medium | contracts/api-ux | All contracts | Consistent error codes for clients |
| C-056 | 🟡 Medium | contracts/token | Mint/burn pulling user tokens | Token allowance assumptions |
| C-057 | 🟡 Medium | contracts/burning | `acbu_burning` `burn_for_basket` | Validate recipient accounts non-empty + distinct |
| C-058 | 🟡 Medium | contracts/minting | Recipient address parameter | Mint path: validate recipient is a real account |
| C-059 | 🟡 Medium | contracts/reserves | `acbu_reserve_tracker` init | Initialization parameters validation |
| C-060 | 🟡 Medium | contracts/ops | `acbu_oracle` admin | Oracle admin rotation workflow |
| C-019 | 🟢 Low | contracts/burning | `acbu_burning` | Redundant DECIMALS multiply/divide in burn path |
| C-022 | 🟢 Low | contracts/fees | `acbu_savings_vault`, `acbu_lending_pool` | Magic number vs `BASIS_POINTS` constant inconsistency |
| C-023 | 🟢 Low | contracts/burning | `acbu_burning` events | Per-account BurnEvent fee reporting mismatch |
| C-029 | 🟢 Low | contracts/hygiene | `acbu_minting` | Unused import noise in minting crate |
| C-030 | 🟢 Low | contracts/burning | `acbu_burning` | Empty recipient list style / checks |
| C-031 | 🟢 Low | contracts/oracle | `acbu_oracle` | Oracle RateUpdateEvent validators empty |
| C-045 | 🟢 Low | contracts/devx | `acbu-smart-contract/Cargo.lock` | Deterministic builds: `Cargo.lock` policy |
| C-049 | 🟢 Low | contracts/build | Root `soroban_token_contract.wasm` | Wasm artifact in repo |
| C-053 | 🟢 Low | contracts/perf | Events with large vectors | Gas griefing: unbounded vectors in events |

**Totals:** 6 Critical · 21 High · 24 Medium · 9 Low · **60 total catalog items** (all IDs from C-001 through C-060, fully aligned with `issues/contracts.md`).

---

## 2. Severity Counts

| Severity | Count |
|----------|-------|
| 🔴 Critical | **6** (C-001 – C-006) |
| 🟠 High | **21** (C-007–C-012, C-021, C-027, C-028, C-034–C-042, C-050–C-052, C-055) |
| 🟡 Medium | **24** (C-013–C-018, C-020, C-024–C-026, C-032, C-033, C-036, C-043, C-044, C-046–C-048, C-054, C-056–C-060) |
| 🟢 Low | **9** (C-019, C-022, C-023, C-029–C-031, C-045, C-049, C-053) |
| **Total** | **60** |

These six critical items are **ship-blockers**. They each enable direct loss of funds and must be remediated before the ACBU MVP can be deployed to Stellar mainnet.

---

## 3. Critical Issues (🔴) — Ship Blockers

### C-001 — Escrow `release` missing authorization
- **Area:** contracts/escrow
- **Evidence:** `acbu_escrow/src/lib.rs` (`release`)
- **Impact:** Any address can call `release(escrow_id)` and send funds held in escrow to the payee, enabling theft of any escrow created by an honest payer.
- **Fix direction:** Require payer (`payer.require_auth()`) or admin auth on `release` and `refund`; add explicit error for unauthorized caller. Maintain CEI ordering for token transfer.
- **Acceptance check:** Unauthorized `release` returns `ContractError::Unauthorized` at contract unit-test level; only payer or admin can settle an escrow.

### C-002 — `mint_from_fiat` lacks off-chain settlement proof
- **Severity:** 🔴 Critical
- **Area:** contracts/minting
- **Evidence:** `acbu_minting/src/lib.rs` (`mint_from_fiat`)
- **Impact:** `mint_from_fiat` does not validate `fintech_tx_id` or that an off-chain fiat deposit has actually settled. Combined with `check_admin_or_user` semantics, recipients can mint ACBU without backing.
- **Fix direction:** Require admin-only minting for the fiat path (or verified settlement proof object), enforce uniqueness of `fintech_tx_id` in storage to prevent replay, and add an integration test that mints from a known-good provider.
- **Acceptance check:** Cannot mint from `mint_from_fiat` without a valid (admin, fintech_tx_id) tuple; replay of an existing `fintech_tx_id` returns the original transaction id and never mints twice.

### C-003 — Escrow ID key collision
- **Severity:** 🔴 Critical
- **Area:** contracts/escrow
- **Evidence:** `acbu_escrow/src/lib.rs` storage keying
- **Impact:** Escrow keys are `(ESCROW, escrow_id)` only. Two payers using the same `escrow_id` cause the second `create` to overwrite the first, locking the first payer's funds in someone else's escrow.
- **Fix direction:** Include `payer` in the key tuple — `(ESCROW, payer, escrow_id)` — and reject duplicates with a domain error.
- **Acceptance check:** Property test confirms that two different payers calling `create` with the same `escrow_id` cannot collide and each can independently settle their escrow.

### C-004 — `verify_reserves` uses wrong supply source
- **Severity:** 🔴 Critical
- **Area:** contracts/reserves
- **Evidence:** `acbu_reserve_tracker/src/lib.rs` (`verify_reserves`)
- **Impact:** `verify_reserves` reads `acbu_client.balance(&env.current_contract_address())` as the total minted supply. The reserve tracker does not hold ACBU, so this is always zero, and the function returns `true` early. Reserve checks trivially pass on every mint.
- **Fix direction:** Read the authoritative token supply via the token contract’s `total_supply` (or another trusted source stored at init), then compute `reserves_total_usd / minted_supply` against `min_ratio_bps` / `target_ratio_bps`.
- **Acceptance check:** Unit test asserts `verify_reserves(true_min)` returns `false` when the contract is undercollateralized and `true` when it is overcollateralized.

### C-005 — Savings vault missing `require_auth` on deposit/withdraw
- **Severity:** 🔴 Critical
- **Area:** contracts/savings
- **Evidence:** `acbu_savings_vault/src/lib.rs` (deposit/withdraw)
- **Impact:** Anyone can call `withdraw(user=X, amount=Z)` and move X's funds, until the redeploy fixes auth.
- **Fix direction:** Add `user.require_auth()` to deposit and withdraw; emit explicit events; redeploy.
- **Acceptance check:** Withdraw requires the authenticated user’s address; non-owner calls fail with `ContractError::Unauthorized`.

### C-006 — Lending pool missing `require_auth` on deposit/withdraw
- **Severity:** 🔴 Critical
- **Area:** contracts/lending
- **Evidence:** `acbu_lending_pool/src/lib.rs` (deposit/withdraw)
- **Impact:** Anyone can call `withdraw(lender=X, amount=Y)` and drain X's lender position.
- **Fix direction:** Add `lender.require_auth()` to deposit, withdraw, and any other lender-only methods; redeploy.
- **Acceptance check:** Withdraw requires the authenticated lender’s address; non-lender calls fail with `ContractError::Unauthorized`.

---

## 4. High Severity Issues (🟠)

### C-007 — Savings term not enforced
- **Area:** contracts/savings · **Evidence:** `acbu_savings_vault/src/lib.rs`
- **Impact:** `term_seconds` is stored but never checked on withdraw. Users can deposit and withdraw immediately, breaking the savings product semantics.
- **Fix direction:** Compare `env.ledger().timestamp()` to deposit timestamp + `term_seconds`. Allow early withdrawal only with an explicit penalty applied via `calculate_fee` policy.
- **Acceptance check:** Time-manipulation tests in the Soroban harness show that withdrawing before the term either fails or pays the documented penalty.

### C-008 — `mint_from_fiat` missing max mint bound
- **Area:** contracts/minting · **Evidence:** `acbu_minting/src/lib.rs` (`mint_from_fiat`)
- **Impact:** `mint_from_usdc` enforces `MAX_MINT_AMOUNT`, but `mint_from_fiat` only enforces `MIN_MINT_AMOUNT`, allowing unbounded minting.
- **Fix direction:** Mirror the `MAX_MINT_AMOUNT` constant check from `mint_from_usdc`; expose the bound through storage for admin updates.
- **Acceptance check:** A mint request of `MAX_MINT_AMOUNT + 1` fails with `ContractError::InvalidAmount`.

### C-009 — Burn basket integer division dust loss
- **Area:** contracts/burning · **Evidence:** `acbu_burning/src/lib.rs` (`burn_for_basket`)
- **Impact:** `amount_per_account = acbu_after_fee / recipient_accounts.len()` truncates. With many recipients, dust accumulates to user loss or an accounting mismatch off-chain.
- **Fix direction:** Allocate the remainder to the last recipient (or to the treasury in a `DustReleased` event) such that `sum(recipient_amounts) + fee == input` within 1 stroop.
- **Acceptance check:** Fuzz test against randomized recipient counts; total reconciles to `acbu_after_fee + fee` exactly.

### C-010 — Escrow create lacks duplicate guard
- **Area:** contracts/escrow · **Evidence:** `acbu_escrow/src/lib.rs` (`create`)
- **Impact:** Even with C-003 fixed (per-payer key), the same payer calling `create` twice with the same `escrow_id` will overwrite their own escrow and lock prior funds.
- **Fix direction:** Reject with `ContractError::AlreadyExists` if the escrow already exists in storage; or use a monotonic per-payer sequence instead of caller-supplied ids.
- **Acceptance check:** A second `create` from the same payer with the same id returns an explicit error and does not modify storage.

### C-011 — Minting loads oracle/reserve clients but does not call
- **Area:** contracts/minting · **Evidence:** `acbu_minting/src/lib.rs`
- **Impact:** `oracle` and `reserve_tracker` addresses are stored at init, but rates are hardcoded to 1:1 and reserve checks are skipped. Economic design is bypassed.
- **Fix direction:** Invoke the oracle to fetch the basket-weighted ACBU/USD rate; call `reserve_tracker.get_reserve_ratio()` before minting; reject if below `min_ratio_bps` or if oracle rate is stale beyond `UPDATE_INTERVAL_SECONDS`.
- **Acceptance check:** Mint fails when oracle is stale beyond threshold or when reserves are below the minimum.

### C-012 — Burning loads oracle/reserve clients but does not call
- **Area:** contracts/burning · **Evidence:** `acbu_burning/src/lib.rs`
- **Impact:** Same as C-011 — oracle/reserve loaded but never invoked; burn FX rate is hardcoded.
- **Fix direction:** Use oracle medians + bounds checks before burn. Validate the redemption currency has sufficient reserves via `reserve_tracker.get_reserves()`.
- **Acceptance check:** Burn rejects out-of-range FX and insufficient reserves with explicit errors.

### C-021 — Event payload schema drift vs backend listener
- **Area:** contracts/integration · **Evidence:** Mint/Burn events vs `acbu-backend` listener parsers
- **Impact:** Indexer silently drops events when field names, types, or decimals diverge between contract-emitted events and backend parser expectations.
- **Fix direction:** Generate a shared JSON schema or Rust types in a shared crate consumed by both contracts and the backend listener; cross-check fields in CI.
- **Acceptance check:** Golden vector tests across contracts and backend listeners pass against the same fixture.

### C-027 — Loan events never emitted / loan logic missing
- **Area:** contracts/lending · **Evidence:** `acbu_lending_pool/src/lib.rs`
- **Impact:** `LoanCreatedEvent` and `RepaymentEvent` are defined but never emitted. Borrow/repay state transitions are not implemented on-chain.
- **Fix direction:** Implement `borrow(borrower, amount, interest_rate, duration_seconds)` and `repay(borrower, loan_id, amount)` with proper state machine; emit events at every transition. Coordinate with `acbu-lending-service` on backend consumption.
- **Acceptance check:** Loan lifecycle integration test (create → accrue → repay) emits every event in the right order with correct fields.

### C-028 — Missing tests for core mint flows
- **Area:** contracts/qa · **Evidence:** `acbu_minting/tests/test.rs` coverage gaps
- **Impact:** Regressions in the most critical module are not caught; CI confidence in mint is low.
- **Fix direction:** Add unit + integration tests for `mint_from_usdc` and `mint_from_fiat` happy paths, error paths, and edge cases (zero, max, min, pause, oracle stale, reserves low). Enforce coverage threshold for `acbu_minting` in CI.
- **Acceptance check:** `cargo test -p acbu_minting` covers both flows; coverage gate set in CI.

### C-034 — No contract versioning / upgrade story
- **Area:** contracts/ops · **Evidence:** Multiple crates
- **Impact:** No `version` field in storage, no documented migration playbook. Future upgrades cannot migrate user data safely.
- **Fix direction:** Add `version: u32` storage key at init; document upgrade + data migration playbook; pin invariant that user migration is opt-in and gated.
- **Acceptance check:** Upgrade runbook tested on testnet for at least one contract.

### C-035 — Admin pause flags: verify on all state changes
- **Area:** contracts/governance · **Evidence:** Mint/burn/savings/lending
- **Impact:** Emergency pause may not cover all mutating entrypoints, leaving a window for continued mint/burn during an incident.
- **Fix direction:** Audit each `pub fn` for pause guard; add fuzz/property tests that paused contracts reject all mutating calls.
- **Acceptance check:** Fuzz test asserts no state change can occur while `paused=true`.

### C-037 — Integer overflow review for all i128 money math
- **Area:** contracts/security · **Evidence:** All `*_amount` fields
- **Impact:** `i128` overflow can wrap in panic paths or produce incorrect comparisons downstream.
- **Fix direction:** Use checked math helpers in `shared`; explicit bounds on every multiplication; document per-field ranges. Add property tests covering extremes.
- **Acceptance check:** Property tests with extreme values never overflow silently.

### C-038 — Authorization invocation tree for cross-contract calls
- **Area:** contracts/security · **Evidence:** Mint/burn calling token client
- **Impact:** Wrong invoker can break assumptions about who authorized a token transfer.
- **Fix direction:** Require auth on every external entrypoint; document the trust model in `INTEGRATION.md`; review token client call sites for `require_auth` propagation.
- **Acceptance check:** Threat model document updated and reviewed.

### C-039 — Strict input validation for string IDs (`fintech_tx_id`)
- **Area:** contracts/minting · **Evidence:** `mint_from_fiat` parameters
- **Impact:** Garbage IDs pollute indexers, complicate partner reconciliation.
- **Fix direction:** Enforce length, charset, and uniqueness in storage; reject invalid ids at the contract boundary.
- **Acceptance check:** Invalid ids (oversize, non-printable, reused) are rejected with `ContractError::InvalidInput`.

### C-040 — Decimal scaling consistency (7 decimals) across contracts
- **Area:** contracts/tokenomics · **Evidence:** Mint/burn/oracle
- **Impact:** Off-by-1e7 errors between contracts are catastrophic for users.
- **Fix direction:** Centralize `DECIMALS`, `stroop`, and unit-test helpers in `shared`; add cross-contract integration test for rounding.
- **Acceptance check:** Full mint→burn round trip reconciles within 1 stroop.

### C-041 — Oracle staleness: max ledger age for rates
- **Area:** contracts/oracle · **Evidence:** `acbu_oracle/src/lib.rs`
- **Impact:** Old rates may be used in volatile markets, enabling arbitrage.
- **Fix direction:** Reject rates older than `STALENESS_THRESHOLD_LEDGERS` unless an admin override flag is present.
- **Acceptance check:** A stale rate cannot be used for new mints.

### C-042 — Minimum oracle sources for quorum
- **Area:** contracts/oracle · **Evidence:** `acbu_oracle/src/lib.rs`
- **Impact:** A single-source feed can be exploited to push the median.
- **Fix direction:** Require `sources.len() >= K` for an update to be accepted; reject updates with `K-1` or fewer sources.
- **Acceptance check:** Cannot update with one manipulated feed.

### C-050 — Integration tests: escrow full lifecycle
- **Area:** contracts/qa · **Evidence:** `acbu_escrow/tests` (add)
- **Impact:** Regressions go undetected on the escrow fixes for C-001/C-003/C-010.
- **Fix direction:** End-to-end tests for create → fund → release/dispute/refund covering both happy and adversarial paths.
- **Acceptance check:** Property tests cover at least 8 distinct flows.

### C-051 — Integration tests: savings vault lock + interest
- **Area:** contracts/qa · **Evidence:** `acbu_savings_vault/tests` (expand)
- **Impact:** Lock and yield logic regressions silently corrupt user balances.
- **Fix direction:** Time manipulation tests in Soroban harness; cover under-term, exactly-at-term, and post-term cases.
- **Acceptance check:** Withdraw before term fails (or pays penalty); after-term withdraw succeeds with yield event matching spec.

### C-052 — Integration tests: lending borrow/repay
- **Area:** contracts/qa · **Evidence:** `acbu_lending_pool/tests` (expand)
- **Impact:** Lending is high risk; needs coverage of liquidation if applicable.
- **Fix direction:** Cover borrow → accrue → repay → default scenarios; verify liquidation behavior per spec.
- **Acceptance check:** Loan default scenario matches documented behavior.

### C-055 — Access control naming audit (`check_admin_or_user`)
- **Area:** contracts/authz · **Evidence:** Minting and other modules
- **Impact:** Misnamed helper may end up being too permissive; unintuitive caller set.
- **Fix direction:** Rename to a name that reflects actual semantics; add unit tests that cover all negative authorization cases.
- **Acceptance check:** Negative authorization tests catch every disallowed caller pattern in CI.

## 5. Medium Severity Issues (🟡)

### C-013 — Token WASM import uses zero SHA256 placeholder
- **Area:** contracts/build · **Evidence:** Mint/burn/reserve tracker token imports
- **Impact:** Integrity unverified; supply chain risk in production builds.
- **Fix direction:** Pin real wasm hash for `soroban_token_contract.wasm`; verify in CI against the deployed Stellar Asset Contract (SAC) artifact.
- **Acceptance check:** Build fails if the hash does not match the artifact.

### C-014 — Double `.unwrap().unwrap()` pattern in escrow storage reads
- **Area:** contracts/escrow · **Evidence:** `acbu_escrow/src/lib.rs`
- **Impact:** Potential panics or undefined behavior if storage is malformed.
- **Fix direction:** Replace with a clean Option-handling helper that returns `ContractError::NotFound` on missing keys.
- **Acceptance check:** No `.unwrap().unwrap()` remains in production paths; missing storage returns a clean error.

### C-015 — Double `.unwrap().unwrap()` pattern in savings vault
- **Area:** contracts/savings · **Evidence:** `acbu_savings_vault/src/lib.rs`
- **Impact:** Same as C-014.
- **Fix direction:** Replace with the same helper used in C-014.
- **Acceptance check:** No panic paths in normal operation; missing storage returns a structured error.

### C-016 — Double `.unwrap().unwrap()` pattern in lending pool
- **Area:** contracts/lending · **Evidence:** `acbu_lending_pool/src/lib.rs`
- **Impact:** Same as C-014.
- **Fix direction:** Replace with the same helper used in C-014.
- **Acceptance check:** Consistent error handling across all three contracts.

### C-017 — Oracle outlier detection is a no-op
- **Area:** contracts/oracle · **Evidence:** `acbu_oracle/src/lib.rs`
- **Impact:** Poisoned sources can move the median, undermining price integrity.
- **Fix direction:** Implement real outlier rejection or quarantine (e.g. `dropped_source` event) above `OUTLIER_THRESHOLD_BPS`. Emit an `OutlierDetected` event for downstream alerting.
- **Acceptance check:** Outlier source cannot move median beyond the configured bound; events emit on detection.

### C-018 — Oracle unit test assertion incorrect vs median behavior
- **Area:** contracts/oracle · **Evidence:** `acbu_oracle/tests/test.rs`
- **Impact:** CI red or false confidence depending on changes.
- **Fix direction:** Fix the expected value to match `median(sources)`; add tests for multiple fixtures.
- **Acceptance check:** `cargo test -p acbu_oracle` passes reliably across all median cases.

### C-020 — `median` allocates via `to_vec()` in shared util
- **Area:** contracts/shared · **Evidence:** `shared/src/lib.rs` (`median`)
- **Impact:** `no_std`/budget issues on Soroban.
- **Fix direction:** Replace with an in-place selection algorithm (`select_nth_unstable` style) within gas budget.
- **Acceptance check:** Contract budget measurements stay within limits on mainnet settings.

### C-024 — Savings withdraw yield always zero
- **Area:** contracts/savings · **Evidence:** `acbu_savings_vault/src/lib.rs` (`WithdrawEvent`)
- **Impact:** Cannot build yield product UX honestly.
- **Fix direction:** Either implement a real accrual model or remove yield claims from the product spec.
- **Acceptance check:** Event `yield_amount` matches computed yield in tests; if accrual is removed, the field is deleted and the docs are updated.

### C-025 — Lending fee rate unused
- **Area:** contracts/lending · **Evidence:** `acbu_lending_pool/src/lib.rs`
- **Impact:** Economic model incomplete.
- **Fix direction:** Apply fees on interest/repayments as designed; emit fee events.
- **Acceptance check:** Fee events are non-zero in the happy path; off-chain reconciliation agrees.

### C-026 — Savings fee rate unused
- **Area:** contracts/savings · **Evidence:** `acbu_savings_vault/src/lib.rs`
- **Impact:** Same as C-025 for savings deposits.
- **Fix direction:** Apply deposit/withdraw fee policy consistently.
- **Acceptance check:** Fee events reflected in balances and off-chain reporting.

### C-032 — Predictable `transaction_id` in minting
- **Area:** contracts/minting · **Evidence:** `acbu_minting/src/lib.rs`
- **Impact:** Correlation attacks and partner id collisions become trivial.
- **Fix direction:** Compute `transaction_id = sha256(user || amount || external_ref || ledger_seq)` (or similar), and ensure uniqueness via storage or sequence counter.
- **Acceptance check:** Generated IDs are globally unique with high probability; fuzz test confirms no collision over a million samples.

### C-033 — Reserve tracker emits no events on updates
- **Area:** contracts/reserves · **Evidence:** `acbu_reserve_tracker/src/lib.rs`
- **Impact:** Harder to index reserve history for transparency dashboards.
- **Fix direction:** Emit `ReserveUpdateEvent` on every `update_reserve` call.
- **Acceptance check:** Listener can reconstruct complete reserve timeline from emitted events.

### C-036 — Reentrancy class issues via token callbacks
- **Area:** contracts/security · **Evidence:** Token transfers inside contracts
- **Impact:** Soroban reentrancy differs from EVM but token contract behavior still needs audit.
- **Fix direction:** Verify CEI ordering; document any reliance on token contract non-reentrancy; check Soroban SDK guarantees.
- **Acceptance check:** External reviewer audit notes checked in.

### C-043 — Emergency multisig for admin operations
- **Area:** contracts/governance · **Evidence:** Admin-only methods
- **Impact:** Single key compromise is catastrophic.
- **Fix direction:** Use Soroban auth pattern with M-of-N multisig threshold; harden against single-point compromise.
- **Acceptance check:** Admin action requires configured M-of-N in tests.

### C-044 — Contract size/compute budget regression tests
- **Area:** contracts/qa · **Evidence:** CI pipeline
- **Impact:** Near-limit contracts fail after small edits.
- **Fix direction:** Track contract size and compute budget metrics in CI; alert on regression beyond threshold.
- **Acceptance check:** CI fails the PR if budget exceeds threshold.

### C-046 — `cargo audit` / supply chain monitoring
- **Area:** contracts/security · **Evidence:** Rust dependencies
- **Impact:** Transitive CVEs in Soroban ecosystem.
- **Fix direction:** Add audit step in CI; pin critical dependencies.
- **Acceptance check:** CI fails on critical advisories (or documented waiver).

### C-047 — Formalize invariants documentation per contract
- **Area:** contracts/docs · **Evidence:** `INTEGRATION.md` / `SMART_CONTRACT_SPEC`
- **Impact:** Auditors slow; engineers drift.
- **Fix direction:** Add `INVARIANTS.md` per crate linking each public fn to its pre/post conditions; review in PRs.
- **Acceptance check:** Each public fn lists pre/post conditions in code comments or invariants doc.

### C-048 — Testnet vs mainnet feature flags in code
- **Area:** contracts/ops · **Evidence:** `dummy_token` etc.
- **Impact:** Accidental mainnet deploy of test code paths.
- **Fix direction:** Compile-time features or explicit network checks at init; mainnet builds must not reference test-only tokens.
- **Acceptance check:** Mainnet build cannot reference dummy token.

### C-054 — Consistent error codes for clients
- **Area:** contracts/api-ux · **Evidence:** All contracts
- **Impact:** Clients cannot map errors to UX flows.
- **Fix direction:** Standard `ContractError` with stable discriminants across the workspace; document in OpenAPI/Soroban spec.
- **Acceptance check:** Spec lists error codes and clients can map them to user-facing messages.

### C-056 — Token allowance assumptions if using allowance
- **Area:** contracts/token · **Evidence:** Mint/burn pulling user tokens
- **Impact:** Wrong allowance patterns block users or allow drains.
- **Fix direction:** Document pull vs push model; align with frontend allowance UX; add matrix in `INTEGRATION.md`.
- **Acceptance check:** Manual test matrix for allowances passes for at least the three flows mint, burn, savings deposit.

### C-057 — Burn path: validate recipient accounts non-empty + distinct
- **Area:** contracts/burning · **Evidence:** `burn_for_basket` recipients
- **Impact:** Duplicate recipients could double-pay off-chain mapping.
- **Fix direction:** Enforce non-empty and deduplicate (or reject duplicates).
- **Acceptance check:** Duplicates rejected deterministically and emit a clear error.

### C-058 — Mint path: validate recipient is a real account
- **Area:** contracts/minting · **Evidence:** Recipient address parameter
- **Impact:** Minting to contract-only addresses strands ACBU.
- **Fix direction:** Optional claim flow (mint to claim PDA redeemed by user) or accounts allowlist pattern.
- **Acceptance check:** Stranded mint rate is negligible / documented.

### C-059 — Reserve tracker initialization parameters validation
- **Area:** contracts/reserves · **Evidence:** `acbu_reserve_tracker` init
- **Impact:** Bad init can brick the policy.
- **Fix direction:** Validate ranges (`min_ratio < target_ratio < BASIS_POINTS`); freeze critical params after launch.
- **Acceptance check:** Init cannot set impossible targets.

### C-060 — Oracle admin rotation workflow
- **Area:** contracts/ops · **Evidence:** `acbu_oracle` admin
- **Impact:** Lost admin key bricks updates.
- **Fix direction:** Two-step transfer + timelock.
- **Acceptance check:** Rotation runbook tested on testnet.

---

## 6. Low Severity Issues (🟢)

### C-019 — Redundant DECIMALS multiply/divide in burn path
- **Area:** contracts/burning · **Evidence:** `acbu_burning/src/lib.rs`
- **Impact:** Noise / audit confusion.
- **Fix direction:** Simplify; keep explicit where needed for readability.
- **Acceptance check:** Diff is purely refactor with identical behavior under fuzz.

### C-022 — Magic number vs `BASIS_POINTS` constant inconsistency
- **Area:** contracts/fees · **Evidence:** `acbu_savings_vault`, `acbu_lending_pool`
- **Impact:** Fee math harder to audit consistently.
- **Fix direction:** Import `shared::BASIS_POINTS` everywhere.
- **Acceptance check:** Grep shows single `BASIS_POINTS` definition.

### C-023 — Per-account BurnEvent fee reporting mismatch
- **Area:** contracts/burning · **Evidence:** `acbu_burning/src/lib.rs` events
- **Impact:** Off-chain accounting reconciliation breaks.
- **Fix direction:** Emit totals + per-recipient net amounts consistently so the indexer can reconcile to the original input.
- **Acceptance check:** Indexer balances to events within 1 stroop.

### C-029 — Unused import noise in minting crate
- **Area:** contracts/hygiene · **Evidence:** `acbu_minting/src/lib.rs`
- **Impact:** Clutter.
- **Fix direction:** Remove unused imports; add `cargo clippy --deny warnings` in CI.
- **Acceptance check:** `cargo clippy` clean.

### C-030 — Empty recipient list style / checks
- **Area:** contracts/burning · **Evidence:** `acbu_burning/src/lib.rs`
- **Impact:** Minor readability.
- **Fix direction:** Use `is_empty()`; add explicit error.
- **Acceptance check:** Clear error when recipients empty.

### C-031 — Oracle RateUpdateEvent validators empty
- **Area:** contracts/oracle · **Evidence:** `acbu_oracle/src/lib.rs`
- **Impact:** Downstream cannot audit which validators contributed.
- **Fix direction:** Populate validator set in event from the active signers for that update.
- **Acceptance check:** Event includes non-empty validator metadata.

### C-045 — Deterministic builds: `Cargo.lock` policy
- **Area:** contracts/devx · **Evidence:** `acbu-smart-contract/Cargo.lock`
- **Impact:** Non-reproducible deployments.
- **Fix direction:** CI verifies `cargo build --locked`.
- **Acceptance check:** Locked builds pass in CI.

### C-049 — Wasm artifact `soroban_token_contract.wasm` in repo
- **Area:** contracts/build · **Evidence:** Root `soroban_token_contract.wasm`
- **Impact:** Binary blobs in git complicate review.
- **Fix direction:** Store as release artifacts; pin hash in source.
- **Acceptance check:** Repo size reduced (or documented policy inline).

### C-053 — Gas griefing: unbounded vectors in events
- **Area:** contracts/perf · **Evidence:** Events with large vectors
- **Impact:** Large transactions fail or become expensive.
- **Fix direction:** Cap `recipients.len()` per call; paginate off-chain with a follow-up event for continuation flows.
- **Acceptance check:** Maximum recipients enforced with clear error.

---

## 7. Legacy 34-Item Origin List

The following list was previously the only contents of this file. Items are kept here as a stable cross-reference; their detailed entries are in the sections above (or in `issues/contracts.md` if explicitly listed there). Items marked **(moved)** have been superseded by the equivalent entry in the detailed catalog; items marked **(archived)** describe issues already considered and not requiring separate tracking.

### Critical
1. **Missing access control on escrow `release`** – C-001 *(moved)*
2. **Unrestricted minting in `mint_from_fiat`** – C-002 *(moved)*
3. **Escrow ID collision** – C-003 *(moved)*
4. **Incorrect total supply in `verify_reserves`** – C-004 *(moved)*
5. **Missing auth checks in savings vault** – C-005 *(moved)*
6. **Missing auth checks in lending pool** – C-006 *(moved)*

### High
7. **Term not enforced in savings vault** – C-007 *(moved)*
8. **No max amount check in `mint_from_fiat`** – C-008 *(moved)*
9. **Integer division truncation in `burn_for_basket`** – C-009 *(moved)*
10. **No duplicate escrow check** – C-010 *(moved)*
11. **Oracle and reserve tracker unused in minting** – C-011 *(moved)*
12. **Oracle and reserve tracker unused in burning** – C-012 *(moved)*

### Medium
13. **Token WASM import uses zero SHA256** – C-013 *(moved)*
14. **Double `.unwrap().unwrap()` in escrow** – C-014 *(moved)*
15. **Double `.unwrap().unwrap()` in savings vault** – C-015 *(moved)*
16. **Double `.unwrap().unwrap()` in lending pool** – C-016 *(moved)*
17. **Outlier detection has no effect in oracle** – C-017 *(moved)*
18. **Incorrect oracle test assertion** – C-018 *(moved)*
19. **Redundant calculation in burning** – C-019 *(moved)*
20. **`median` uses `to_vec()` in no_std** – C-020 *(moved)*
21. **Contract events vs backend listeners** – C-021 *(moved)*

### Low
22. **Magic number for fee cap in savings and lending** – C-022 *(moved)*
23. **Incorrect fee in per-account BurnEvent** – C-023 *(moved)*
24. **`yield_amount` always 0 in savings** – C-024 *(moved)*
25. **Fee rate stored but unused in lending pool** – C-025 *(moved)*
26. **Fee rate stored but unused in savings vault** – C-026 *(moved)*
27. **Loan events never emitted** – C-027 *(moved)*
28. **No tests for core minting flows** – C-028 *(moved)*

### Trivial
29. **Unused import** in `acbu_minting/src/lib.rs` – C-029 *(moved)*
30. **`len() == 0` style** in `acbu_burning/src/lib.rs` – C-030 *(moved)*
31. **Empty validators in event** in `acbu_oracle/src/lib.rs` – C-031 *(moved)*
32. **Weak `transaction_id` generation** in `acbu_minting/src/lib.rs` – C-032 *(moved)*
33. **No events in reserve tracker** in `acbu_reserve_tracker/src/lib.rs` – C-033 *(moved)*
34. **No upgrade or versioning** across multiple contracts – C-034 *(moved)*

---

## 8. Resolution Tracker (Fix Status)

> **Format:** `ID · title · status · linked PR / commit · owner`
>
> **Forward-looking template.** This is a placeholder grid designed for the day a fix lands. Every item is currently 🟡 Open; reviewers should not read uniformity of status as stagnation. When a fix merges, the row flips to ✅ Fixed and a PR/commit link is added. Example anchors below show what a real fixed row looks like.

| ID | Title | Status | Notes |
|----|-------|--------|-------|
| _(example)_ | _(example anchor: prior PR #14 closed issue #1)_ | ✅ Fixed (PR #14 merged) | Historical anchor — not a contracts item |
| C-001 | Escrow `release` missing authorization | 🟡 Open (tracked separately) | Blocked on auth refactor for C-005/C-006 |
| C-002 | `mint_from_fiat` lacks off-chain settlement proof | 🟡 Open | Admin-only path + uniqueness required |
| C-003 | Escrow ID key collision | 🟡 Open | Payer+id tuple planned |
| C-004 | `verify_reserves` uses wrong supply source | 🟡 Open | Read `total_supply` from token contract |
| C-005 | Savings vault missing `require_auth` | 🟡 Open | Redeploy gating MVP |
| C-006 | Lending pool missing `require_auth` | 🟡 Open | Redeploy gating MVP |
| C-007 | Term not enforced | 🟡 Open | Coupled to C-005 |
| C-008 | `mint_from_fiat` missing max bound | 🟡 Open | Coupled to C-002 |
| C-009 | Burn basket dust loss | 🟡 Open | Allocate remainder to last |
| C-010 | Escrow create duplicate guard | 🟡 Open | Coupled to C-003 |
| C-011 | Minting oracle/reserve unused | 🟡 Open | Rate fetching required |
| C-012 | Burning oracle/reserve unused | 🟡 Open | Rate fetching required |
| C-013 | WASM zero SHA256 | 🟡 Open | Pin hash in CI |
| C-014..C-016 | Double unwrap patterns | 🟡 Open | Standardize helper |
| C-017 | Oracle outlier detection no-op | 🟡 Open | Implement quarantine |
| C-018 | Oracle test assertion | 🟡 Open | Fix expected value |
| C-019 | Redundant DECIMALS | 🟢 Trivial | Refactor PR |
| C-020 | `median` to_vec in no_std | 🟡 Open | In-place algorithm |
| C-021 | Event schema drift | 🟡 Open | Shared types crate |
| C-022 | Magic number fees | 🟢 Trivial | Use shared constant |
| C-023 | Per-account BurnEvent fee | 🟡 Open | Reconcile events |
| C-024 | Yield always 0 | 🟡 Open | Accrual model |
| C-025 | Lending fee rate unused | 🟡 Open | Apply policy |
| C-026 | Savings fee rate unused | 🟡 Open | Apply policy |
| C-027 | Loan events missing | 🟡 Open | Implement borrow/repay |
| C-028 | No core mint tests | 🟡 Open | Coverage gate CI |
| C-029 | Unused import | 🟢 Trivial | Lint PR |
| C-030 | Len()==0 style | 🟢 Trivial | Refactor PR |
| C-031 | Empty validators in event | 🟢 Trivial | Fill in event |
| C-032 | Predictable transaction_id | 🟡 Open | Hash-based id |
| C-033 | Reserve tracker no events | 🟡 Open | Emit on update |
| C-034 | No versioning/upgrade | 🟡 Open | Runbook + storage |
| C-035 | Pause not all-encompassing | 🟡 Open | Audit + fuzz |
| C-036 | Reentrancy via token | 🟡 Open | External review |
| C-037 | i128 overflow | 🟡 Open | Checked math |
| C-038 | Auth invocation tree | 🟡 Open | Threat model doc |
| C-039 | fintech_tx_id validation | 🟡 Open | Reject garbage |
| C-040 | 7-decimals consistency | 🟡 Open | Centralize constant |
| C-041 | Oracle staleness threshold | 🟡 Open | Reject stale |
| C-042 | Min oracle sources | 🟡 Open | Quorum required |
| C-043 | Emergency multisig | 🟡 Open | M-of-N pattern |
| C-044 | Budget regression tests | 🟡 Open | CI metric |
| C-045 | Deterministic builds | 🟢 Trivial | `cargo build --locked` |
| C-046 | cargo audit | 🟡 Open | CI step |
| C-047 | INVARIANTS per contract | 🟡 Open | Doc per crate |
| C-048 | Testnet/mainnet flags | 🟡 Open | Compile-time |
| C-049 | WASM blob in repo | 🟢 Trivial | Release artifact |
| C-050..C-052 | End-to-end integration tests | 🟡 Open | Property tests |
| C-053 | Gas griefing | 🟢 Trivial | Cap + paginate |
| C-054 | Consistent error codes | 🟡 Open | Standardize enum |
| C-055 | Auth helper naming | 🟡 Open | Rename + tests |
| C-056 | Token allowance assumptions | 🟡 Open | Document matrix |
| C-057 | Recipient uniqueness | 🟡 Open | Reject duplicates |
| C-058 | Mint to real account | 🟡 Open | Claim flow |
| C-059 | Reserve init validation | 🟡 Open | Range checks |
| C-060 | Oracle admin rotation | 🟡 Open | Two-step + timelock |

> **Status legend** — 🟢 Trivial/cosmetic · 🟡 Open (tracked) · 🔵 In review · ✅ Fixed (PR linked)
>
> **Reminder when updating:** when an external PR closes an issue, replace `🟡 Open` with `✅ Fixed` and append the merged PR link in the Notes column. Do not rewrite history; the row itself is the audit trail.

---

## 9. Cross-Reference Map

| Document | Relationship to this file |
|----------|---------------------------|
| [`../../issues/contracts.md`](../../issues/contracts.md) | **Live triage-ready catalog.** Same IDs and severities; entries are condensed for daily use. |
| [`../../issues/MASTER_INDEX.md`](../../issues/MASTER_INDEX.md) | Master index across backend/frontend/contracts backlogs. |
| [`../../issues/backend.md`](../../issues/backend.md) | Backend MVP issues (B-001..B-075). |
| [`../../issues/frontend.md`](../../issues/frontend.md) | Frontend MVP issues (F-001..F-065). |
| [`../SMART_CONTRACT_SPEC.MD`](../SMART_CONTRACT_SPEC.MD) | Detailed on-chain spec (interfaces, types, events, invariants, security model). |
| [`../../DOCS/SMART_CONTRACT_SPEC.MD`](../../DOCS/SMART_CONTRACT_SPEC.MD) | Spec doc used in MVPs; duplicates the contract-side spec. |
| [`../../DOCS/ARCHITECTURE_DIAGRAMS.MD`](../../DOCS/ARCHITECTURE_DIAGRAMS.MD) | Mint/burn/reserve/oracle diagrams. |
| [`../API_AND_CONTRACTS_REFERENCE.MD`](../API_AND_CONTRACTS_REFERENCE.MD) | Routes ↔ contracts mapping. |
| [`../../TECHNICAL/ARCHITECTURE.MD`](../../TECHNICAL/ARCHITECTURE.MD) | High-level architecture. |
| [`../../TECHNICAL/ORACLE_SYSTEM.MD`](../../TECHNICAL/ORACLE_SYSTEM.MD) | Oracle multi-source design context for C-011/012/017/018/041/042. |
| [`../../TECHNICAL/RESERVE_MANAGEMENT.MD`](../../TECHNICAL/RESERVE_MANAGEMENT.MD) | Reserve model context for C-004/C-059. |
| [`../../TECHNICAL/PRICING_MECHANISM.MD`](../../TECHNICAL/PRICING_MECHANISM.MD) | Pricing/basket-weight context for C-040/C-024/C-025/C-026. |
| [`../PROJECT_STRUCTURE.MD`](../PROJECT_STRUCTURE.MD) | Repo layout (where the contract source lives). |
| [`../TASK_BREAKDOWN.MD`](../TASK_BREAKDOWN.MD) | Stage-by-stage tasks (where fixes are scheduled). |
| [`../../DOCS/IMPLEMENTATION_PLAN.MD`](../../DOCS/IMPLEMENTATION_PLAN.MD) | Phase-by-phase remediation timeline. |
| [`../../correction.md`](../../correction.md) | Standing-list of additional items curated for fast triage. |

---

## 10. Top Remediations (ship-safety order)

Order these fixes in the MVP launch runbook before deploying mainnet:

1. **C-001 / C-005 / C-006** — Auth on escrow/savings/lending (single PR or coordinated).
2. **C-002** — Mint only with verified settlement proof.
3. **C-003 / C-010** — Per-payer escrow keys + duplicate guard.
4. **C-004** — Real token-supply source in reserve tracker.
5. **C-011 / C-012** — Wire oracle + reserve checks into mint/burn.
6. **C-007 / C-024** — Term + yield honoring in savings.
7. **C-008** — Mirror mint bound on `mint_from_fiat`.
8. **C-009 / C-023** — Reconcile burn accounting + events.
9. **C-013** — Pin token WASM hash.
10. **C-014 / C-015 / C-016** — Replace double-unwrap helpers.
11. **C-017 / C-021** — Outlier enforcement + shared event schemas.
12. **C-027 / C-050 / C-051 / C-052** — Lifecycle + integration tests.

---

## 11. Maintenance

- **New issues** — Add to `issues/contracts.md` first (canonical triage). Mirror to this file's Section 4–6 with the next stable ID.
- **Closed fixes** — Promote the fix to `✅ Fixed (PR linked)` in Section 8 and link to the merged PR.
- **Audit-changing decisions** — Update Section 3 first (Critical) so reviewers see the most consequential revisions.
- **Cross-doc references** — Update `issues/MASTER_INDEX.md` summary counts when severities change.

---

## Related Documents

- [Live contracts issue catalog](../../issues/contracts.md)
- [Master backlog index](../../issues/MASTER_INDEX.md)
- [Backend issues](../../issues/backend.md)
- [Frontend issues](../../issues/frontend.md)
- [Smart contract spec (root copy)](../SMART_CONTRACT_SPEC.md)
- [Smart contract spec (DOCS copy)](../../DOCS/SMART_CONTRACT_SPEC.MD)
- [Project structure map](../PROJECT_STRUCTURE.md)
- [Architecture (TECHNICAL)](../../TECHNICAL/ARCHITECTURE.MD)
- [Oracle system design](../../TECHNICAL/ORACLE_SYSTEM.MD)

---

_Last consolidated: June 2026. Source content merged from legacy `PROJECT/issues/CONTRACTS_ISSUES.md` list (34 items) and canonical [`issues/contracts.md`](../../issues/contracts.md) (60 items). Severity buckets, IDs, and totals fully aligned with canonical catalog; legacy list is preserved in Section 7 for reference but every item now resolves to a canonical C-### ID._
