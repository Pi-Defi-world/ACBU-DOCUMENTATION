# Limits and Tiers

Single reference for deposit/withdrawal limits by actor type, circuit breakers, and tier behaviour. Used by backend limits service and API validation.

---

## Audience and actor types

- **Retail (individual):** `actorType === 'retail'`; tier `free` or `verified`. Uses `/v1/retail/*` mint/burn.
- **Business:** `actorType === 'sme'` or `enterprise` (or org); tier `sme` or `enterprise`. Uses `/v1/business/*` mint/burn.
- **Government:** `actorType === 'government'` (user or org). Uses `/v1/government/*` mint/burn.

---

## Limits matrix

| Actor      | Deposit (daily) | Deposit (monthly) | Withdrawal – single currency (daily) | Withdrawal – single currency (monthly) | Withdrawal – basket | Notes                    |
| ---------- | --------------- | ----------------- | ------------------------------------- | -------------------------------------- | ------------------- | ------------------------ |
| Individual | $5,000          | $50,000           | $10,000 equivalent/day                | $80,000                                 | Unlimited           | 24h rolling window       |
| Business   | $50,000         | $500,000          | $100,000/day with 24h notice          | $800,000                                | Unlimited           | SME/enterprise tier      |
| Government | $500,000        | $5,000,000        | $500,000/day; 24h/72h notice for large | $4,000,000                            | Unlimited           | Dedicated caps; advance notice |

- **Deposit limits:** Per actor type; apply to mint/deposit (basket currencies only). Enforced in mint service before creating transaction.
- **Withdrawal limits:** Per actor type, per currency (single-currency redemption). Rolling 24h and calendar-month windows. Enforced in burn flow and withdrawal-processing job.
- **Basket withdrawal:** No cap (unlimited) for redeeming ACBU for basket proportion; single-currency redemption is capped as above.

---

## Circuit breakers (system-wide)

- If any currency reserve **&lt; 10%** of its target weight: **pause single-currency withdrawals** for that currency. Basket withdrawals continue.
- If total reserve ratio **&lt; 102%**: Pause **new minting** until replenished.
- If withdrawals **&gt; 20%** of daily volume threshold: Optional 24h delay on new withdrawal requests (configurable).

Circuit breakers are evaluated in the burn path and in the withdrawal-processing job using reserve data from ReserveTracker.

---

## Spread and dynamic fees (reference)

- **Spread:** Buy/sell spread 0.2–0.3% (configurable). Applied in rates/quote and mint/burn via fee policy service.
- **Dynamic fees:** Reserve-based fee adjustment per [RISK_MANAGEMENT.MD](RISK_MANAGEMENT.MD) (e.g. low reserve → higher withdrawal fee). Implemented by fee policy service consuming reserve levels.

---

## Related documents

- [Risk Management](RISK_MANAGEMENT.MD)
- [Revenue Streams](../BUSINESS/REVENUE_STREAMS.MD)
- [API and Contracts Reference](../PROJECT/API_AND_CONTRACTS_REFERENCE.MD)
