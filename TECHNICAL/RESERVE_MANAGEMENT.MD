# Reserve Management

## Buffer Structure

**Total Reserves: 105% of Outstanding ACBU**

**Breakdown:**
- **100%:** Exact 1:1 backing (untouchable)
- **3%:** Operational buffer (daily rebalancing)
- **2%:** Emergency reserve (crisis situations)
- Regular audit to maintain overcollateralization

## Reserve Setup Mechanisms

### Option 1: Fintech Float Model
Fintechs like Flutterwave and Paystack maintain "float accounts" - pre-funded balances in local currencies across Africa for their payment operations.

**Approach:**
- Partner agreements where fintechs allocate a portion of their float as stablecoin reserves
- Fintechs already hold licenses to hold customer funds in each country
- Stablecoin backed by their obligated float, verified via API
- Settlement happens within their existing infrastructure

### Option 2: Escrow with Licensed Fintech Custodians
- Use fintechs that have e-money licenses (Chipper Cash, Wave, M-Pesa)
- They hold segregated reserve accounts on your behalf
- View-only API access for proof of reserves
- Smart contracts verify balances before minting

### Option 3: Hybrid Reserve Model (Recommended)

**Reserve Structure:**
```
├── 80% in Fintech Partner Float/Custody
│   └── Distributed across multiple fintechs per country
└── 20% in Liquid Crypto Assets (USDC/USDT)
    └── Emergency liquidity buffer for redemptions
```

## Reserve Verification System

- Real-time API integration with each fintech partner
- Merkle tree proofs published on-chain weekly
- Third-party attestation monthly (audit firm)
- Public dashboard showing reserve breakdown

## Geographic Distribution Model

Reserve distribution matches user base:

**Nigeria (40M users):**
- Primarily deposit NGN
- Primarily withdraw NGN
- Reserve naturally accumulates NGN
- Weight: 18% is appropriate

**Kenya (15M users):**
- Primarily deposit/withdraw KES
- Reserve accumulates KES
- Weight: 12% is appropriate

**Cross-border transactions (20% of volume):**
- These create natural rebalancing
- Nigerian in Ghana withdraws GHS
- Ghanaian in Nigeria withdraws NGN

## Monthly Flow Example

**Nigeria:**
- Deposits: 10B NGN
- Withdrawals: 9.5B NGN (local users)
- Cross-border withdrawals: 0.3B NGN (foreigners)
- Net: +0.2B NGN accumulation

**Ghana:**
- Deposits: 600M GHS
- Withdrawals: 580M GHS (local users)
- Cross-border withdrawals: 10M GHS (travelers)
- Net: +10M GHS accumulation

**Natural Balance:** ✓ Both reserves growing proportionally, no major rebalancing needed

## Deposit rule: pool (basket) currencies and USDC/XLM

- **Basket (pool) only in reserves:** Only **basket currencies** (NGN, KES, RWF, ZAR, GHS, EGP, MAD, TZS, UGX, XOF, etc.) are held in the pool. **No USDC and no USDT** are held in the pool.
- **User deposits:** Users can deposit **basket currency** (POST /mint/deposit), **USDC** (POST /mint/usdc), or **XLM** (POST /onramp/register). For USDC we convert USDC→XLM in the backend; once conversion succeeds we approve the mint. For XLM we use backend pools/swaps and mint. Pools (e.g. ACBU/XLM or P2P) and swap services run as **independent** backend processes—users do not wait for them.
- **API:** POST /mint/deposit accepts only basket currency. POST /mint/usdc accepts USDC (we convert then mint). POST /onramp/register accepts XLM deposit (we process and mint).

## USDC and XLM flows (backend conversion and jobs)

1. **USDC deposit (POST /mint/usdc):** User deposits USDC to us. Backend job converts USDC→XLM (via our LP/swap service, which runs independently). Once conversion succeeds, we mint ACBU to the user's wallet. User does not wait for LPs.
2. **XLM deposit (POST /onramp/register):** User deposits XLM. Backend job sells XLM (or uses equivalent value) and mints ACBU to the user. We may use ACBU/XLM LP or P2P in the backend; these run independently.
3. **Smart contracts:** ACBU mint/burn and savings use on-chain smart contracts (acbu_minting, acbu_burning, acbu_savings_vault, etc.). USDC/XLM and ACBU/XLM liquidity and swaps can be **backend-only** (custody + swap APIs or our own P2P); no new smart contracts are required for those. On-chain AMM contracts can be added later if we want on-chain liquidity.

## Reserve segments (quarters)

Reserves are tracked in two segments:

- **Transactions quarter:** Backs current mint/burn and P2P activity (1:1 backing for circulating ACBU). Updated by mint, burn, and rebalancing. Reserve health and overcollateralization use this segment by default.
- **Investment/savings quarter:** Allocated to savings vault and investment products. Updated by savings/investment flows; not used for same-day transaction redemptions.

Reporting and dashboard can show both segments. Movement between segments is defined in ops docs.

## Reserve Health Monitoring

**Daily Checks:**
- Monitor reserve ratios (alert if < 102%)
- Track actual vs target weights
- Verify 1:1 backing ratio maintained

**Public Dashboard:**
- Total ACBU supply
- Total reserve value (USD equivalent)
- Overcollateralization ratio (target: 105%)
- Breakdown by currency
- Last updated timestamp

---

**Related Documents:**
- [Rebalancing](../TECHNICAL/REBALANCING.MD)
- [Architecture](../TECHNICAL/ARCHITECTURE.MD)
