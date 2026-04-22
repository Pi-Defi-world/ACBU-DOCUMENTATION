# Testnet custodial MVP bootstrap

After deploying Soroban contracts (`acbu-smart-contract/scripts/deploy_testnet.sh`), complete these steps so `mint_from_demo_fiat`, `mint_from_usdc`, and `admin_drip_demo_fiat` do not fail on oracle, reserve, or empty custody.

## 1. Capture contract IDs

Record IDs in backend `.env` as `CONTRACT_ORACLE_TESTNET`, `CONTRACT_RESERVE_TRACKER_TESTNET`, `CONTRACT_MINTING_TESTNET`, `CONTRACT_BURNING_TESTNET`, or the generic `CONTRACT_*` keys (see `ACBU-DOCUMENTATION/ENV_VARS.md`).

## 2. Oracle: basket weights, rates, and `set_s_token_address`

- Ensure `initialize` / basket setup matches your basket currencies (same codes as `acbu-backend` `BASKET_CURRENCIES`).
- For **each** basket currency, call `submit_rate` (or your admin path) so `get_rate` succeeds.
- For **each** basket currency, call `set_s_token_address` with the deployed **demo SAC** contract ID for that currency.

`get_acbu_usd_rate` returns a safe default (`DECIMALS` = 1:1 in fixed-point) when no weighted rates exist yet, but you should still seed real rates before relying on economics.

## 3. Reserve tracker

`is_reserve_sufficient` compares total on-chain USD reserves to minted supply. After deploy, run **`update_reserve`** per currency with non-zero `value_usd` (7-decimal fixed point, same as `shared::DECIMALS`) so the first mint passes. Example shape (invoke via `stellar contract invoke` or your admin script):

- `currency`: `CurrencyCode` as a single-element string vector in XDR (see `acbu-backend/scripts/initContracts.ts` for examples).
- `value_usd`: large enough vs expected ACBU supply × `min_reserve_ratio`.

## 4. Demo fiat custody

1. Deploy one **SAC / token contract** per basket currency (cap supply per your policy, e.g. 1e9 whole units).
2. Mint the **entire demo supply to the minting contract’s contract address** (custodial pool on the minter).
3. Optionally call **`set_operator`** on the minting contract so the backend signing key matches the authorized operator for `mint_from_demo_fiat` (defaults to **admin** if unset).

## 5. Backend operator key

`STELLAR_SECRET_KEY` must be the key that signs custodial txs:

- **`admin_drip_demo_fiat`**: must be **admin** on the minting contract.
- **`mint_from_demo_fiat`**: first argument `operator` is the same account as the tx source; it must equal `get_operator()` (default admin).

## 6. Smoke tests

- `admin_drip_demo_fiat` → user wallet receives demo SAC.
- `POST /v1/fiat/onramp` → `mint_from_demo_fiat` completes; `Transaction` row `status=completed` with real `blockchain_tx_hash`.
- Horizon: user ACBU balance increases.

## 7. USDC path

`mint_from_usdc` is unchanged. Ensure oracle + reserves are seeded and USDC SAC matches minting `initialize` as documented in `ENV_VARS.md`.

## Troubleshooting

### `Error(WasmVm, MissingValue)` — “trying to invoke non-existent contract function”, `admin_drip_demo_fiat` or `mint_from_demo_fiat`

The **contract id in `.env` points at an old deployment** whose WASM predates custodial entrypoints.

**`cargo build` alone does not change testnet.** After building (`cargo build --release -p acbu_minting --target wasm32-unknown-unknown`), you must **upload the WASM and upgrade the contract** with Soroban CLI (e.g. `soroban contract build` / `install` / `invoke ... --update` per your workflow), **or** deploy a new minting contract and point `CONTRACT_MINTING_*` at it. Then seed demo SAC supply to the minter address again if the contract id changed.
