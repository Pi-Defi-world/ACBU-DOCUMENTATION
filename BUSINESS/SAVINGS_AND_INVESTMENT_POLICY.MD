# Savings and Investment Policy

Policy for ACBU savings (lock calendar, returns) and investments (withdrawal rules by audience). Aligns with [Business Model](BUSINESS_MODEL.MD) and [Revenue Streams](REVENUE_STREAMS.MD).

---

## Savings

### Lock calendar

- **Monthly lock dates:** Savings withdrawals are **not allowed** on specific calendar days each month (e.g. 1st and 15th, or configurable list).
- **Behaviour:** On a lock date, the API returns an error and the **next available withdrawal date**.
- **Implementation:** Backend and (if needed) contract check the lock calendar before processing savings withdrawal. Calendar is stored in config or DB for ops changes.

### Returns (parameter-driven)

- **Parameters:** Term (e.g. 3/6/12 months), amount tier, and/or investment/savings segment yield rate. Optional: configurable APY table per term/tier.
- **Formula (example):** `accrued_yield = principal * (apy_from_params / 100) * (days_locked / 365)`. APY can come from config, segment yield, or oracle.
- **Transparency:** Parameter set and formula are documented and configurable so user returns are predictable.

---

## Investments

### Retail (everyday users)

- User submits an **investment withdrawal request**. Funds become available **24 hours** after request (T+24h).
- When funds are available, a **notification** is sent (in-app, push, email, or SMS): “Your investment withdrawal of X ACBU is now available.”
- No fee for normal 24h redemption (unless a standard fee is added later).
we need to add a fee in the future [junman sam]

### Business

- **Scheduled withdrawal dates:** Normal withdrawals only on **specific calendar dates** (e.g. monthly or bi-weekly). No early-withdrawal fee on those dates.
- **Forced (early) removal:** If a business requests withdrawal **off** the allowed dates, it is treated as early: funds are available in **24 hours** but a **1% fee** is applied (deducted from amount or charged separately).

---

## Reserve segment

Savings and investment liquidity use the **investment/savings quarter** of reserves (see [Reserve Management – Reserve segments](../TECHNICAL/RESERVE_MANAGEMENT.MD)). Only this segment is used for yield-bearing allocation and for the above products.

---

## Related documents

- [Reserve Management](../TECHNICAL/RESERVE_MANAGEMENT.MD)
- [Limits and Tiers](../OPERATIONS/LIMITS_AND_TIERS.MD)
- [Risk Management](../OPERATIONS/RISK_MANAGEMENT.MD)
