# Smart contracts — MVP issues backlog

Canonical on-chain backlog for `acbu-smart-contract` (Soroban). Older list: [../PROJECT/issues/CONTRACTS_ISSUES.md](../PROJECT/issues/CONTRACTS_ISSUES.md) and `acbu-smart-contract/ISSUES.md`.

**Issue format:** severity, area, evidence, impact, fix hint, acceptance.

---

### C-001 — Escrow `release` missing authorization
**Severity:** Critical · **Area:** contracts/escrow · **Evidence:** `acbu-smart-contract/acbu_escrow/src/lib.rs` (`release`)
**Impact:** Anyone can release escrows → loss of funds. **Fix direction:** Require payer/auth pattern; add tests for unauthorized caller. **Acceptance check:** Unauthorized `release` fails at contract test level.

### C-002 — `mint_from_fiat` lacks off-chain settlement proof
**Severity:** Critical · **Area:** contracts/minting · **Evidence:** `acbu-smart-contract/acbu_minting/src/lib.rs` (`mint_from_fiat`)
**Impact:** Self-mint without fiat backing. **Fix direction:** Require admin-only or verified proof object + id uniqueness. **Acceptance check:** Cannot mint without valid proof in tests.

### C-003 — Escrow ID key collision
**Severity:** Critical · **Area:** contracts/escrow · **Evidence:** `acbu_escrow/src/lib.rs` storage keying
**Impact:** Second create overwrites first → funds stuck/lost. **Fix direction:** Include payer address in key; reject duplicates. **Acceptance check:** Two payers cannot collide in property test.

### C-004 — `verify_reserves` uses wrong supply source
**Severity:** Critical · **Area:** contracts/reserves · **Evidence:** `acbu_reserve_tracker/src/lib.rs` (`verify_reserves`)
**Impact:** Reserve checks always trivially pass. **Fix direction:** Read total minted supply from token contract or authoritative store. **Acceptance check:** Unit test proves false when undercollateralized.

### C-005 — Savings vault missing `require_auth` on deposit/withdraw
**Severity:** Critical · **Area:** contracts/savings · **Evidence:** `acbu_savings_vault/src/lib.rs`
**Impact:** Anyone can withdraw others' balances (until contract fixed). **Fix direction:** Add `user.require_auth()` + role checks; redeploy. **Acceptance check:** Withdraw requires authenticated user address.

### C-006 — Lending pool missing `require_auth` on deposit/withdraw
**Severity:** Critical · **Area:** contracts/lending · **Evidence:** `acbu_lending_pool/src/lib.rs`
**Impact:** Anyone can drain lender positions. **Fix direction:** Add `lender.require_auth()`; redeploy. **Acceptance check:** Withdraw requires authenticated lender address.

### C-007 — Savings term not enforced
**Severity:** High · **Area:** contracts/savings · **Evidence:** `acbu_savings_vault/src/lib.rs`
**Impact:** No lock period → not a savings product. **Fix direction:** Enforce `term_seconds` against ledger timestamp. **Acceptance check:** Early withdraw fails or pays penalty per policy.

### C-008 — `mint_from_fiat` missing max mint bound
**Severity:** High · **Area:** contracts/minting · **Evidence:** `acbu_minting/src/lib.rs`
**Impact:** Unbounded inflation vs USDC path. **Fix direction:** Mirror `max_mint_amount` checks from `mint_from_usdc`. **Acceptance check:** Huge mint rejected at contract level.

### C-009 — Burn basket integer division dust loss
**Severity:** High · **Area:** contracts/burning · **Evidence:** `acbu_burning/src/lib.rs` (`burn_for_basket`)
**Impact:** Rounding dust accumulates to user loss / accounting mismatch. **Fix direction:** Allocate remainder to last recipient or emit dust event to treasury. **Acceptance check:** Sum(recipient amounts)+fee == input within 1 stroop policy.

### C-010 — Escrow create lacks duplicate guard
**Severity:** High · **Area:** contracts/escrow · **Evidence:** `acbu_escrow/src/lib.rs` (`create`)
**Impact:** Overwrite risk on same id. **Fix direction:** Reject if exists; or key includes payer. **Acceptance check:** Second create errors deterministically.

### C-011 — Minting loads oracle/reserve clients but does not call
**Severity:** High · **Area:** contracts/minting · **Evidence:** `acbu_minting/src/lib.rs`
**Impact:** Hardcoded 1:1 rates bypass economic design. **Fix direction:** Invoke oracle + enforce reserve checks. **Acceptance check:** Mint fails when oracle stale beyond threshold.

### C-012 — Burning loads oracle/reserve clients but does not call
**Severity:** High · **Area:** contracts/burning · **Evidence:** `acbu_burning/src/lib.rs`
**Impact:** Same as minting. **Fix direction:** Use oracle medians + bounds. **Acceptance check:** Burn rejects out-of-range FX.

### C-013 — Token WASM import uses zero SHA256 placeholder
**Severity:** Medium · **Area:** contracts/build · **Evidence:** `acbu_minting`, `acbu_burning`, `acbu_reserve_tracker` token imports
**Impact:** Integrity not verified; supply chain risk. **Fix direction:** Pin real wasm hash; verify in CI. **Acceptance check:** Build fails if hash mismatches artifact.

### C-014 — Double unwrap pattern in escrow storage reads
**Severity:** Medium · **Area:** contracts/escrow · **Evidence:** `acbu_escrow/src/lib.rs`
**Impact:** Panics / undefined behavior risk. **Fix direction:** Use correct Option handling; add negative tests. **Acceptance check:** No `.unwrap().unwrap()` in production paths.

### C-015 — Double unwrap pattern in savings vault
**Severity:** Medium · **Area:** contracts/savings · **Evidence:** `acbu_savings_vault/src/lib.rs`
**Impact:** Same as escrow. **Fix direction:** Replace with safe pattern. **Acceptance check:** Missing storage returns clean errors.

### C-016 — Double unwrap pattern in lending pool
**Severity:** Medium · **Area:** contracts/lending · **Evidence:** `acbu_lending_pool/src/lib.rs`
**Impact:** Same as escrow. **Fix direction:** Replace with safe pattern. **Acceptance check:** No panic paths in normal operation.

### C-017 — Oracle outlier detection is a no-op
**Severity:** Medium · **Area:** contracts/oracle · **Evidence:** `acbu_oracle/src/lib.rs`
**Impact:** Poisoned sources can move median. **Fix direction:** Reject/quarantine outliers; emit alert events. **Acceptance check:** Outlier source cannot move median beyond bound.

### C-018 — Oracle unit test assertion incorrect vs median behavior
**Severity:** Medium · **Area:** contracts/oracle · **Evidence:** `acbu_oracle/tests/test.rs`
**Impact:** CI red or gives false confidence depending on changes. **Fix direction:** Fix expected value; add more cases. **Acceptance check:** `cargo test -p acbu_oracle` passes reliably.

### C-019 — Redundant DECIMALS multiply/divide in burn path
**Severity:** Low · **Area:** contracts/burning · **Evidence:** `acbu_burning/src/lib.rs`
**Impact:** Noise / audit confusion. **Fix direction:** Simplify; keep explicit where needed for readability. **Acceptance check:** Diff is purely refactor with identical behavior.

### C-020 — `median` allocates via `to_vec()` in shared util
**Severity:** Medium · **Area:** contracts/shared · **Evidence:** `acbu-smart-contract/shared/src/lib.rs` (`median`)
**Impact:** `no_std`/budget issues on Soroban. **Fix direction:** In-place selection algorithm within gas budget. **Acceptance check:** Contract budget measurements within limits.

### C-021 — Event payload schema drift vs backend listener
**Severity:** High · **Area:** contracts/integration · **Evidence:** Mint/Burn events vs `acbu-backend` listener parsers
**Impact:** Indexer silently drops events. **Fix direction:** Generate shared JSON schema or Rust types shared crate. **Acceptance check:** Golden vector tests across services.

### C-022 — Magic number vs `BASIS_POINTS` constant inconsistency
**Severity:** Low · **Area:** contracts/fees · **Evidence:** `acbu_savings_vault`, `acbu_lending_pool`
**Impact:** Fee math harder to audit consistently. **Fix direction:** Import shared constant everywhere. **Acceptance check:** Grep shows single basis-points definition.

### C-023 — Per-account BurnEvent fee reporting mismatch
**Severity:** Low · **Area:** contracts/burning · **Evidence:** `acbu_burning/src/lib.rs` events
**Impact:** Off-chain accounting reconciliation breaks. **Fix direction:** Emit totals + per-recipient net amounts consistently. **Acceptance check:** Indexer balances to events within 1 unit.

### C-024 — Savings withdraw yield always zero
**Severity:** Medium · **Area:** contracts/savings · **Evidence:** `acbu_savings_vault/src/lib.rs` (`WithdrawEvent`)
**Impact:** Cannot build yield product UX honestly. **Fix direction:** Implement accrual model or remove yield claims. **Acceptance check:** Event yield matches computed yield in tests.

### C-025 — Lending fee rate unused
**Severity:** Medium · **Area:** contracts/lending · **Evidence:** `acbu_lending_pool/src/lib.rs`
**Impact:** Economic model incomplete. **Fix direction:** Apply fees on interest/repayments as designed. **Acceptance check:** Fee events non-zero in happy path.

### C-026 — Savings fee rate unused
**Severity:** Medium · **Area:** contracts/savings · **Evidence:** `acbu_savings_vault/src/lib.rs`
**Impact:** Same as lending. **Fix direction:** Apply deposit/withdraw fee policy. **Acceptance check:** Fees reflected in balances.

### C-027 — Loan events never emitted / loan logic missing
**Severity:** High · **Area:** contracts/lending · **Evidence:** `acbu_lending_pool/src/lib.rs`
**Impact:** Lending MVP is non-functional on-chain. **Fix direction:** Implement borrow/repay state transitions or remove lending marketing. **Acceptance check:** Loan lifecycle integration test passes.

### C-028 — Missing tests for core mint flows
**Severity:** High · **Area:** contracts/qa · **Evidence:** `acbu_minting/tests/test.rs` coverage gaps
**Impact:** Regressions in most critical module. **Fix direction:** Add tests for `mint_from_usdc` / `mint_from_fiat` happy + sad paths. **Acceptance check:** Coverage threshold enforced in CI.

### C-029 — Unused import noise in minting crate
**Severity:** Low · **Area:** contracts/hygiene · **Evidence:** `acbu_minting/src/lib.rs`
**Impact:** Clutter; warnings ignored hide real issues. **Fix direction:** Deny warnings in CI for contracts workspace. **Acceptance check:** `cargo clippy` clean.

### C-030 — Empty recipient list style / checks
**Severity:** Low · **Area:** contracts/burning · **Evidence:** `acbu_burning/src/lib.rs`
**Impact:** Minor readability. **Fix direction:** Use `is_empty()`; add explicit error. **Acceptance check:** Clear error when recipients empty.

### C-031 — Oracle RateUpdateEvent validators empty
**Severity:** Low · **Area:** contracts/oracle · **Evidence:** `acbu_oracle/src/lib.rs`
**Impact:** Downstream cannot audit which validators contributed. **Fix direction:** Populate validator set in event. **Acceptance check:** Event includes non-empty validator metadata.

### C-032 — Predictable `transaction_id` in minting
**Severity:** Medium · **Area:** contracts/minting · **Evidence:** `acbu_minting/src/lib.rs`
**Impact:** Correlation attacks / partner id collisions. **Fix direction:** Include hash(user, amount, external_ref) + sequence uniqueness. **Acceptance check:** IDs are globally unique with high probability.

### C-033 — Reserve tracker emits no events on updates
**Severity:** Medium · **Area:** contracts/reserves · **Evidence:** `acbu_reserve_tracker/src/lib.rs`
**Impact:** Harder to index reserve history. **Fix direction:** Emit structured events on each update. **Acceptance check:** Listener can reconstruct reserve timeline.

### C-034 — No contract versioning / upgrade story
**Severity:** High · **Area:** contracts/ops · **Evidence:** Multiple crates
**Impact:** Cannot migrate users safely as logic evolves. **Fix direction:** Add version storage + admin migration playbook. **Acceptance check:** Runbook documents upgrade + data migration.

### C-035 — Admin pause flags: verify on all state changes
**Severity:** High · **Area:** contracts/governance · **Evidence:** Mint/burn/savings/lending
**Impact:** Emergency pause might not cover all entrypoints. **Fix direction:** Audit each `pub fn` for pause guard. **Acceptance check:** Fuzz/property: paused contract rejects mutating calls.

### C-036 — Reentrancy class issues via token callbacks
**Severity:** Medium · **Area:** contracts/security · **Evidence:** Token transfers inside contracts
**Impact:** Soroban reentrancy differs; still verify token contract behavior. **Fix direction:** Checks-effects-interactions ordering audit. **Acceptance check:** Audit notes checked in by external reviewer.

### C-037 — Integer overflow review for all i128 money math
**Severity:** High · **Area:** contracts/security · **Evidence:** All `*_amount` fields
**Impact:** Overflow could wrap in panic paths or incorrect comparisons. **Fix direction:** Use checked math helpers; explicit bounds. **Acceptance check:** Property tests never overflow silently.

### C-038 — Authorization invocation tree for cross-contract calls
**Severity:** High · **Area:** contracts/security · **Evidence:** Mint/burn calling token client
**Impact:** Wrong invoker can break assumptions. **Fix direction:** Require auth on external entrypoints; document trust model. **Acceptance check:** Threat model doc updated.

### C-039 — Strict input validation for string IDs (fintech_tx_id)
**Severity:** High · **Area:** contracts/minting · **Evidence:** `mint_from_fiat` parameters
**Impact:** Garbage IDs pollute indexers. **Fix direction:** Enforce length + charset + uniqueness storage. **Acceptance check:** Invalid ids rejected at contract boundary.

### C-040 — Decimal scaling consistency (7 decimals) across contracts
**Severity:** High · **Area:** contracts/tokenomics · **Evidence:** Mint/burn/oracle
**Impact:** Off-by-1e7 errors catastrophic. **Fix direction:** Centralize `DECIMALS` constants in `shared`. **Acceptance check:** Cross-contract integration test for rounding.

### C-041 — Oracle staleness: max ledger age for rates
**Severity:** High · **Area:** contracts/oracle · **Evidence:** `acbu_oracle/src/lib.rs`
**Impact:** Old rates used in volatile markets. **Fix direction:** Reject updates older than N ledgers unless admin override. **Acceptance check:** Stale rate cannot be used for new mints.

### C-042 — Minimum oracle sources for quorum
**Severity:** High · **Area:** contracts/oracle · **Evidence:** `acbu_oracle/src/lib.rs`
**Impact:** Single-source manipulation. **Fix direction:** Require `sources.len() >= K` for update. **Acceptance check:** Cannot update with one manipulated feed.

### C-043 — Emergency multisig for admin operations
**Severity:** Medium · **Area:** contracts/governance · **Evidence:** Admin-only methods
**Impact:** Single key compromise is catastrophic. **Fix direction:** Use Soroban auth pattern with multisig threshold. **Acceptance check:** Admin action requires M-of-N in tests.

### C-044 — Contract size / compute budget regression tests
**Severity:** Medium · **Area:** contracts/qa · **Evidence:** CI pipeline
**Impact:** Near-limit contracts fail after small edits. **Fix direction:** Track budget metrics in CI. **Acceptance check:** Budget under threshold on mainnet settings.

### C-045 — Deterministic builds: `Cargo.lock` policy
**Severity:** Low · **Area:** contracts/devx · **Evidence:** `acbu-smart-contract/Cargo.lock`
**Impact:** Reproducible deployments. **Fix direction:** CI verifies `cargo build --locked`. **Acceptance check:** Locked builds pass in CI.

### C-046 — `cargo audit` / supply chain monitoring
**Severity:** Medium · **Area:** contracts/security · **Evidence:** Rust dependencies
**Impact:** Transitive CVEs in Soroban ecosystem. **Fix direction:** Add audit step in CI. **Acceptance check:** CI fails on critical advisories.

### C-047 — Formalize invariants documentation per contract
**Severity:** Medium · **Area:** contracts/docs · **Evidence:** `INTEGRATION.md` / `SMART_CONTRACT_SPEC`
**Impact:** Auditors slow; engineers drift. **Fix direction:** Add `INVARIANTS.md` per crate linking code anchors. **Acceptance check:** Each public fn lists pre/post conditions.

### C-048 — Testnet vs mainnet feature flags in code
**Severity:** Medium · **Area:** contracts/ops · **Evidence:** `dummy_token` etc.
**Impact:** Accidental mainnet deploy of test token paths. **Fix direction:** Compile-time features or explicit network checks. **Acceptance check:** Mainnet build cannot reference dummy token.

### C-049 — Wasm artifact `soroban_token_contract.wasm` in repo
**Severity:** Low · **Area:** contracts/build · **Evidence:** Root `soroban_token_contract.wasm`
**Impact:** Binary blobs in git complicate review. **Fix direction:** Store in release artifacts; pin hash only in source. **Acceptance check:** Repo size and review policy improved.

### C-050 — Integration tests: escrow full lifecycle
**Severity:** High · **Area:** contracts/qa · **Evidence:** `acbu_escrow/tests` (add)
**Impact:** Regression on escrow fixes. **Fix direction:** End-to-end create/fund/release/dispute patterns. **Acceptance check:** Happy path + adversarial tests pass.

### C-051 — Integration tests: savings vault lock + interest
**Severity:** High · **Area:** contracts/qa · **Evidence:** `acbu_savings_vault/tests` (expand)
**Impact:** Ensure lock + yield logic remains correct. **Fix direction:** Time manipulation tests in Soroban harness. **Acceptance check:** Withdraw before term fails.

### C-052 — Integration tests: lending borrow/repay
**Severity:** High · **Area:** contracts/qa · **Evidence:** `acbu_lending_pool/tests` (expand)
**Impact:** Lending is high risk; needs coverage. **Fix direction:** Cover liquidation path if applicable. **Acceptance check:** Loan default scenario behaves per spec.

### C-053 — Gas griefing: unbounded vectors in events
**Severity:** Low · **Area:** contracts/perf · **Evidence:** Events with large vectors
**Impact:** Large txs fail or become expensive. **Fix direction:** Cap recipients length; paginate off-chain. **Acceptance check:** Max recipients enforced with clear error.

### C-054 — Consistent error codes for clients
**Severity:** Medium · **Area:** contracts/api-ux · **Evidence:** All contracts
**Impact:** Clients cannot map errors to UX. **Fix direction:** Standard error enum codes per contract. **Acceptance check:** OpenAPI/Soroban spec lists error codes.

### C-055 — Access control naming audit (`check_admin_or_user`)
**Severity:** High · **Area:** contracts/authz · **Evidence:** Minting and other modules
**Impact:** Misnamed helper might allow unintended caller set. **Fix direction:** Rename + unit tests for negative authorization. **Acceptance check:** Unauthorized roles cannot call restricted entrypoints.

### C-056 — Token allowance assumptions if using allowance
**Severity:** Medium · **Area:** contracts/token · **Evidence:** Mint/burn pulling user tokens
**Impact:** Wrong allowance patterns block users or allow drains. **Fix direction:** Document pull vs push model; align with frontend. **Acceptance check:** Manual test matrix for allowances passes.

### C-057 — Burn path: validate recipient accounts non-empty + distinct
**Severity:** Medium · **Area:** contracts/burning · **Evidence:** `burn_for_basket` recipients
**Impact:** Duplicate recipients could double-pay off-chain mapping. **Fix direction:** Enforce uniqueness policy explicitly. **Acceptance check:** Duplicates rejected or merged deterministically.

### C-058 — Mint path: validate recipient is a real account
**Severity:** Medium · **Area:** contracts/minting · **Evidence:** Recipient address parameter
**Impact:** Minting to contract-only addresses could strand funds. **Fix direction:** Optional claim flow or allowlist pattern. **Acceptance check:** Stranded mint rate is negligible / documented.

### C-059 — Reserve tracker initialization parameters validation
**Severity:** Medium · **Area:** contracts/reserves · **Evidence:** `acbu_reserve_tracker` init
**Impact:** Bad init can brick policy. **Fix direction:** Validate ranges; freeze critical params after launch. **Acceptance check:** Init cannot set impossible targets.

### C-060 — Oracle admin rotation workflow
**Severity:** Medium · **Area:** contracts/ops · **Evidence:** `acbu_oracle` admin
**Impact:** Lost admin key bricks updates. **Fix direction:** Two-step transfer + timelock. **Acceptance check:** Runbook tested on testnet.

