# Risk Management

ACBU is exposed to financial, operational, regulatory, and technical risks because it
holds real reserves, supports redemptions in multiple currencies, and relies on
partners across several jurisdictions. This document is the operating playbook for
identifying, measuring, reducing, and responding to those risks.

---

## Objectives

- Protect the 1:1 backing of circulating ACBU before growth, yield, or expansion.
- Keep single-currency redemption available when reserves are healthy, but never at
  the expense of the full basket.
- Detect reserve, liquidity, oracle, partner, fraud, and compliance stress early.
- Give operations, engineering, compliance, and leadership clear escalation rules.
- Maintain transparent communication with users, partners, auditors, and regulators.

## Core Principles

1. **Backing first:** Minting, burning, yield, and rebalancing must preserve the
   reserve ratio and published reserve policy.
2. **Basket fairness:** Individual users may redeem in a single currency only within
   liquidity, fee, and risk controls. Basket redemption remains the baseline right.
3. **No silent risk transfer:** Any fee change, withdrawal delay, pause, reserve
   adjustment, or reweighting must be recorded and explainable.
4. **Defense in depth:** No single oracle, custodian, fintech partner, admin key, or
   monitoring system should be able to create an unrecoverable failure.
5. **Proportional response:** Controls become stricter as risk rises and are relaxed
   only after objective recovery criteria are met.

---

## Risk Governance

| Function | Primary responsibility | Escalates to |
| --- | --- | --- |
| Risk owner | Maintains risk register, thresholds, and mitigation plans | Executive committee |
| Treasury operations | Reserve allocation, liquidity buffers, partner limits | Risk owner |
| Compliance | KYC/AML reviews, sanctions screening, regulator notices | General counsel |
| Engineering | Circuit breakers, monitoring, oracle controls, incident fixes | CTO |
| Security | Key management, exploit response, access reviews | CTO / executive committee |
| Executive committee | Emergency pauses, structural changes, public disclosures | Board / regulators |

### Review cadence

- **Daily:** Reserve ratio, currency weights, withdrawal queues, partner health,
  oracle freshness, failed jobs, suspicious transaction alerts.
- **Weekly:** Risk dashboard review, open incidents, limit exceptions, failed
  compliance checks, partner settlement reconciliation.
- **Monthly:** Reserve attestation, stress test report, incident trend review,
  access review, regulatory change scan.
- **Quarterly:** Basket weight review, partner concentration review, crisis
  simulation, board-level risk appetite review.

---

## Risk Appetite and Thresholds

| Risk area | Green | Amber | Red | Hard stop |
| --- | --- | --- | --- | --- |
| Total reserve ratio | >= 105% | 102% to 104.99% | 100% to 101.99% | < 100% |
| Single-currency reserve | 90% to 115% of target | 75% to 89% or 116% to 130% | 50% to 74% or > 130% | < 50% |
| Oracle freshness | < 6h | 6h to 12h | 12h to 24h | > 24h |
| Oracle source deviation | <= 3% | > 3% | > 5% | > 10% unexplained |
| Daily net withdrawals | < 10% of reserves | 10% to 20% | 20% to 30% | > 30% |
| Partner concentration | <= 30% with one partner | 30% to 40% | 40% to 50% | > 50% |
| Failed settlement jobs | < 1% | 1% to 3% | 3% to 5% | > 5% or stuck queue |
| Suspicious transaction rate | Baseline | 2x baseline | 3x baseline | Active exploit pattern |

Hard stops require executive notification and a written recovery decision before the
affected function is resumed.

---

## Risk Register

| Risk | Impact | Primary controls | Monitoring |
| --- | --- | --- | --- |
| Reserve undercollateralization | ACBU no longer fully backed | 105% target, mint pause below 102%, audits | Reserve ratio, partner balances |
| Single-currency depletion | Users cannot redeem requested currency | Dynamic fees, currency caps, pause below threshold | Reserve weight by currency |
| Mass withdrawal event | Liquidity strain and settlement delays | Buffers, queues, credit lines, communication plan | Burn volume, queue age |
| Currency shock | Basket value or liquidity disruption | Emergency reserve, structural reweighting, TWAP | FX volatility, reserve drift |
| Oracle manipulation or outage | Wrong mint/burn pricing | Multi-source median, validator quorum, stale-rate pause | Source deviation, update age |
| Custodian/fintech outage | Settlement failure or balance uncertainty | Multiple partners, view-only proof, exposure limits | API health, reconciliation gaps |
| Smart contract failure | Loss of funds or invalid accounting | Audits, multisig, pauses, timelocks, tests | On-chain events, alerts |
| Backend job failure | Delayed mint/burn, duplicate processing | Idempotency, retry limits, DLQs, alerting | Queue depth, retry count |
| Fraud and AML abuse | Regulatory exposure and reserve drain | KYC tiers, transaction monitoring, EDD | Pattern alerts, SAR queue |
| Key compromise | Unauthorized admin or settlement action | HSM/key vaults, rotation, least privilege | Access logs, signature anomalies |
| Regulatory restriction | Forced service changes in a country | Local partners, legal review, country controls | Regulator notices, policy scans |
| Savings/investment liquidity stress | Yield products impair redemption liquidity | Reserve segmentation, withdrawal calendar, caps | Segment liquidity ratio |
| Partner insolvency | Reserve impairment | Diversification, legal segregation, attestations | Credit reviews, failed settlement |
| Reputation and communication failure | User panic and liquidity spiral | Status page, proof of reserves, incident templates | Support volume, social signals |

---

## Control Strategy

### Preventive controls

- Maintain 105% total reserve target with 100% backing, 3% operational buffer, and
  2% emergency reserve.
- Enforce per-actor deposit and withdrawal limits from
  [Limits and Tiers](LIMITS_AND_TIERS.MD).
- Use multiple fintech/custody partners per major country and cap partner exposure.
- Require KYC tier checks before raising transaction limits.
- Separate transactions reserves from savings/investment reserves.
- Require multisig approval for privileged contract and treasury actions.
- Use 24-hour TWAP or longer averages for volatile redemption pricing.
- Maintain 72-hour single-currency redemption lock on newly minted ACBU.

### Detective controls

- Monitor reserve ratio, reserve drift, oracle freshness, price deviation, partner
  API health, failed jobs, withdrawal queue age, and suspicious patterns.
- Reconcile partner balances against internal ledger daily.
- Publish proof-of-reserves data weekly and obtain monthly third-party attestations.
- Alert compliance on mint-to-redeem cycling, linked beneficial-owner activity, and
  transactions inconsistent with KYC tier.
- Track unresolved control exceptions until an owner closes them.

### Responsive controls

- Raise dynamic fees or bonuses based on reserve weights.
- Queue, delay, or pause single-currency withdrawals when thresholds are breached.
- Pause minting if total reserves fall below the required threshold.
- Move traffic to healthy payment partners during provider outages.
- Rotate compromised keys and suspend affected admin functions.
- Issue user, partner, auditor, and regulator communications based on severity.

### Recovery controls

- Resume only after objective recovery criteria are met and documented.
- Backfill audit logs and reconcile every affected transaction.
- Publish incident summaries for material events.
- Update limits, tests, runbooks, or partner exposure caps after each post-mortem.

---

## Rebalancing and Withdrawal Risk

This is where most basket-backed stablecoins fail. ACBU addresses withdrawal
pressure before it becomes a reserve crisis.

### Single-currency withdrawal pressure

**Problem scenario:** NGN strengthens versus the basket.

- Day 1: NGN = 1,550/USD
- Day 30: NGN = 1,450/USD, strengthened 6.5%

**Arbitrage opportunity:**

- Users deposit weaker currencies.
- Users redeem only NGN because it is now stronger.
- The arbitrage becomes profitable.
- NGN reserves are depleted if controls are absent.

### Strategy A: Dynamic redemption fees

Fees change by currency reserve health. The fee policy service should consume
current reserve levels and return the applicable fee before burn execution.

| Condition | Withdrawal fee | Deposit incentive | User message |
| --- | --- | --- | --- |
| Currency reserve below 85% of target | Higher fee, example 2.0% | Deposit bonus, example 0.3% | "Reserves low; higher fee applies." |
| Currency reserve within target band | Normal fee, example 1.5% | No incentive | Standard quote |
| Currency reserve above 115% of target | Lower fee, example 1.0% | Higher deposit fee or no bonus | "Lower fee available." |

The backend should quote the fee before the user confirms and should store the
fee version used for the transaction.

### Strategy B: Withdrawal limits by currency

**Individual users**

- Basket withdrawal: Unlimited, subject to availability of normal operations.
- Single currency: $10,000 equivalent per day.
- Window: Rolling 24 hours.

**Institutional users**

- Basket withdrawal: Unlimited, subject to normal operations.
- Single currency: $100,000 per day with 24-hour notice.
- Larger amounts: 3-day advance notice and treasury approval.

**System-wide circuit breaker**

- If any currency reserve is below 10% of its target weight, pause
  single-currency withdrawals for that currency.
- Basket withdrawals continue unless total reserve backing is impaired.

### Strategy C: Market-making incentives

The liquidity provider program should encourage natural rebalancing:

- Deposit under-weight currencies to earn a configurable bonus.
- Withdraw over-weight currencies with reduced fees.
- Provide standing orders that treasury can use for rebalancing.
- Offer institution-specific quotes for large, scheduled conversions.

---

## Arbitrage and Abuse Controls

### Institutional arbitrage risk

Sophisticated actors could monitor currency trends, mint ACBU with weak
currencies, redeem for stronger currencies, and drain specific reserves.

### Strategy A: Time-based redemption locks

New ACBU minted by deposit has these rules:

- Peer-to-peer transfers: Available immediately.
- Merchant payments: Available immediately.
- Basket redemption: Available immediately.
- Single-currency redemption: Locked for 72 hours.

The lock prevents flash arbitrage while preserving payment utility.

### Strategy B: Redemption curves

Single-currency redemption fees should rise with trade size and reserve stress.

| Withdrawal size | Example fee |
| --- | --- |
| 0 to 1,000 ACBU | 1.5% |
| 1,000 to 5,000 ACBU | 2.0% |
| 5,000 to 10,000 ACBU | 2.5% |
| 10,000+ ACBU | 3.0% plus slippage |

Large withdrawals can also require manual review, scheduled settlement, or basket
redemption only.

### Strategy C: KYC/AML monitoring

Flag suspicious patterns:

- Mint followed by immediate single-currency redemption.
- Multiple accounts with the same beneficial owner.
- Rapid cycling through multiple currencies.
- Activity coordinated with large forex movements.
- Transactions inconsistent with stated business purpose or KYC tier.

Available actions:

- Enhanced due diligence.
- Basket-redemption-only restriction.
- Temporary withdrawal limits.
- Account freeze where legally required.
- Suspicious activity report to the relevant jurisdiction.

---

## Circuit Breakers

| Trigger | Action | Owner | Resume criteria |
| --- | --- | --- | --- |
| Total reserve ratio < 102% | Pause new minting | Treasury + engineering | Ratio >= 105% or executive-approved plan |
| Total reserve ratio < 100% | Pause minting and discretionary withdrawals | Executive committee | Full backing restored and reconciled |
| Currency reserve < 75% of target | Raise redemption fee and deposit incentive | Treasury | Currency returns to >= 90% of target |
| Currency reserve < 50% of target | Pause single-currency withdrawal | Treasury + compliance | Currency returns to >= 75% and queue cleared |
| Daily net withdrawals > 20% | Add 24h queue for new withdrawals | Treasury | 48h below 10% daily withdrawals |
| Oracle stale > 12h | Use last valid TWAP and alert | Engineering | Fresh quorum update accepted |
| Oracle deviation > 10% unexplained | Pause mint/burn for affected currency | Engineering + risk owner | Source review complete and valid rate published |
| Partner API unavailable > 30m | Route to backup partner | Treasury operations | Partner stable for 2h and reconciliation clean |
| Settlement failure > 5% | Stop sending new settlement jobs to provider | Engineering + treasury | Failed jobs reconciled and root cause fixed |
| Critical security incident | Pause affected contracts/services | Security + executive committee | Incident contained, keys rotated if needed |

---

## Crisis Playbooks

### Scenario 1: Single-currency collapse

**Example:** Egypt devalues EGP by 40% overnight.

**Response:**

- Trigger emergency risk meeting.
- Freeze automatic structural reweighting until rates are verified.
- Use emergency reserve only under executive approval.
- Temporarily reduce affected currency weight if governance permits.
- Increase single-currency redemption fees or pause affected withdrawals.
- Communicate expected ACBU impact and next review time.

**User impact framing:**

- ACBU holders absorb only the affected currency's basket weight impact.
- A 40% drop in a currency with an 11% basket weight implies a much smaller basket
  effect than holding the currency directly.

### Scenario 2: Mass withdrawal event

**Example:** 30% of ACBU redeemed in 48 hours.

**Response:**

- Use operational buffer first.
- Activate approved partner credit lines or scheduled liquidity swaps.
- Queue new withdrawals for 24 to 48 hours if daily thresholds are breached.
- Keep reserve accounting proportional and auditable.
- Update the public dashboard and support scripts at fixed intervals.
- Pause new minting if reserves fall below 102%.

**Non-negotiable:** Do not compromise 1:1 backing.

### Scenario 3: Regional currency crisis

**Example:** Three East African currencies weaken simultaneously.

**Response:**

- Confirm oracle data across official, fintech, and market sources.
- Review regional concentration and liquidity by partner.
- Temporarily tighten affected currency redemption limits.
- Keep normal operations available where reserves remain healthy.
- Prepare a quarterly reweighting proposal if the shock is structural.

### Scenario 4: Oracle outage or manipulation

**Response:**

- Reject outlier feeds and require validator quorum.
- Use the last valid TWAP for display only if mint/burn is paused.
- Pause affected mint/burn routes if the rate is stale or disputed.
- Rotate validator keys if compromise is suspected.
- Publish a post-incident rate correction if user quotes were affected.

### Scenario 5: Fintech or custody partner outage

**Response:**

- Disable new settlement routing to the unhealthy partner.
- Move eligible volume to backup partners.
- Reconcile all in-flight settlements before retrying.
- Cap exposure until the partner passes a health review.
- If insolvency is suspected, notify counsel and auditors immediately.

### Scenario 6: Regulatory restriction

**Response:**

- Pause affected country onboarding, deposits, or withdrawals as required.
- Preserve user redemption paths that remain legally allowed.
- Notify affected users with country-specific instructions.
- Move operations to licensed local partners where possible.
- Update terms, limits, and compliance rules before reopening.

### Scenario 7: Savings or investment liquidity stress

**Response:**

- Keep transaction reserves separate from investment/savings reserves.
- Suspend new yield allocations before touching transaction backing.
- Enforce withdrawal calendars, early withdrawal fees, and T+24h windows.
- Stop marketing yield products until liquidity coverage returns to target.
- Report segment-level reserve health separately.

---

## Stress Testing

Treasury and engineering should run stress tests at least monthly and after major
market events.

Required tests:

- 20%, 30%, and 50% redemption shocks over 24 and 72 hours.
- 10%, 25%, and 40% devaluation of each basket currency.
- Simultaneous outage of the largest fintech partner and largest oracle source.
- 3x normal suspicious transaction volume.
- Failed queue replay with duplicate webhook deliveries.
- Savings/investment withdrawal requests exceeding liquid segment capacity.

Each test must produce:

- Reserve ratio before and after.
- Currency weights before and after.
- Amount of buffer used.
- User-facing delay estimate.
- Circuit breakers triggered.
- Recommended parameter changes.

---

## Reporting and Transparency

### Internal dashboard

Show these metrics to operations, risk, and engineering:

- Total reserve ratio and trend.
- Currency reserve actual versus target.
- Partner exposure by country and provider.
- Oracle freshness and source deviation.
- Mint, burn, and withdrawal queue volumes.
- Failed jobs, retry counts, and dead-letter queues.
- AML alerts by severity and age.
- Savings/investment segment liquidity.

### Public dashboard

Show user-safe transparency data:

- Total ACBU supply.
- Total reserve value and overcollateralization ratio.
- Reserve breakdown by currency.
- Last oracle update.
- Last proof-of-reserves publication.
- Active service notices or pauses.

### Incident communications

For material events, publish:

- What happened.
- Which functions or countries are affected.
- Whether user funds remain fully backed.
- What controls were activated.
- Next update time.
- Final post-mortem when resolved.

---

## Implementation Checklist

- [ ] ReserveTracker reports total reserve ratio, reserve drift, and segment health.
- [ ] Fee policy service uses reserve bands and records fee version per quote.
- [ ] Burn flow enforces single-currency limits even when audience is absent.
- [ ] Mint flow enforces deposit limits even when audience is absent.
- [ ] Oracle service blocks stale or disputed rates from mint/burn paths.
- [ ] Queue workers use idempotency keys, retry caps, and dead-letter queues.
- [ ] Partner router supports health-based failover.
- [ ] Risk dashboard exposes the internal metrics listed above.
- [ ] Compliance queue captures suspicious patterns and review outcomes.
- [ ] Runbooks exist for each crisis playbook.
- [ ] Monthly stress test report is stored with assumptions and results.
- [ ] Post-incident action items are tracked to closure.

---

## Related Documents

- [Limits and Tiers](LIMITS_AND_TIERS.MD)
- [Reserve Management](../TECHNICAL/RESERVE_MANAGEMENT.MD)
- [Rebalancing](../TECHNICAL/REBALANCING.MD)
- [Oracle System](../TECHNICAL/ORACLE_SYSTEM.MD)
- [Savings and Investment Policy](../BUSINESS/SAVINGS_AND_INVESTMENT_POLICY.MD)
- [Security Guide](../SECURITY_GUIDE.MD)
