# Monitoring and Observability

## Overview

ACBU depends on real reserves, live partner integrations, oracle updates, and
stateful settlement flows. Monitoring must therefore cover not only uptime, but
also business-critical health signals such as reserve accuracy, partner balance
freshness, queue backlogs, and contract execution success.

This document defines the monitoring, alerting, and health-check standards for
the ACBU platform.

---

## Objectives

- Detect service degradation before users are affected.
- Protect mint, burn, redemption, and reconciliation flows.
- Verify that reserves, oracle feeds, and partner integrations are fresh and
  consistent.
- Provide engineers, treasury, compliance, and operations with the same source
  of truth.
- Keep a public-facing status view separate from internal operational telemetry.

## Monitoring Scope

### Application layer

- API availability and latency.
- Authentication and authorization failures.
- Request error rates by route and segment.
- Queue processing success, retry, and dead-letter rates.
- Background job execution time and idempotency conflicts.

### Financial and treasury layer

- Total reserve ratio.
- Currency-level reserve weights versus target.
- Mint, burn, and redemption volumes.
- Settlement success and failure rates.
- Reconciliation gaps between internal ledger and partner balances.

### Oracle and pricing layer

- Oracle freshness.
- Source deviation between validators and market feeds.
- Stale rate usage attempts.
- Emergency fallback activation.

### Partner and infrastructure layer

- Fintech partner API health.
- Payment processor latency and response codes.
- Database availability, replication lag, and storage growth.
- Contract event ingestion delays.
- Worker saturation and queue depth.

---

## Health Check Model

Health checks should be layered so that a single failure does not hide a broader
system issue.

### Liveness checks

Liveness confirms the process is running and responsive.

- Process heartbeat.
- Event loop or worker heartbeat.
- Container or host health.

### Readiness checks

Readiness confirms the service can safely receive traffic.

- Database connectivity.
- Cache availability if required.
- Required secrets and configs loaded.
- Queue and partner dependencies reachable where needed.

### Domain checks

Domain checks confirm business-critical functions are safe.

- Oracle rate age below the allowed threshold.
- Reserve ratio above the minimum threshold.
- Settlement backlog within tolerance.
- Mint and burn services are not paused unexpectedly.

### Suggested endpoints

- `GET /health`
- `GET /ready`
- `GET /metrics`
- `GET /status`

The public `status` endpoint should expose only user-safe service state, not
internal secrets, balances, or operational details.

---

## Metrics

### Golden signals

- Latency.
- Traffic.
- Errors.
- Saturation.

### Business-critical metrics

- `reserve_ratio_total`
- `reserve_ratio_by_currency`
- `oracle_age_seconds`
- `oracle_source_deviation_percent`
- `mint_request_count`
- `burn_request_count`
- `withdrawal_queue_age_seconds`
- `reconciliation_gap_amount`
- `partner_api_error_rate`
- `dead_letter_queue_depth`

### Recommended alert dimensions

- Route or segment.
- Currency.
- Partner.
- Environment.
- Severity.

---

## Logging

Logs should be structured, searchable, and consistent across services.

### Required log fields

- Timestamp.
- Service name.
- Environment.
- Request ID or correlation ID.
- User or account reference when permitted.
- Route or job name.
- Severity.
- Error code or exception class.
- Partner or currency context when relevant.

### Logging rules

- Never log private keys, secrets, full card data, or sensitive user credentials.
- Use correlation IDs across API requests, background jobs, and partner callbacks.
- Prefer structured JSON logs over free-form text.
- Record state transitions for mint, burn, pause, resume, and reconciliation
  events.

---

## Tracing

Distributed tracing should be enabled for:

- API gateway requests.
- Mint and burn transactions.
- Partner settlement calls.
- Oracle refresh jobs.
- Queue workers and webhook handlers.

Tracing should answer:

- Which service failed first?
- Where did latency accumulate?
- Which partner or dependency caused the delay?
- Did retries create duplicate processing?

---

## Alerting

Alerts should be actionable, deduplicated, and tied to an owner.

### Severity levels

- **Info:** Non-urgent operational events.
- **Warning:** Degradation that is still within tolerance.
- **Critical:** User-facing impact or financial risk.

### Alert examples

- Reserve ratio falls below the minimum operating threshold.
- Oracle data is stale beyond the configured limit.
- Partner API fails repeatedly or becomes unreachable.
- Queue backlog exceeds processing capacity.
- Reconciliation gap exceeds the allowed tolerance.
- Mint or burn success rate drops below acceptable levels.

### Alert rules

- Page only for conditions that require immediate action.
- Route warnings to dashboards and chat channels, not the pager, unless the
  issue persists.
- Each alert should have an owner, runbook link, and expected response window.

---

## Dashboards

### Internal dashboard

Used by engineering, treasury, and operations.

- Reserve ratio trend.
- Oracle freshness and deviation.
- Partner health by country and provider.
- Queue depth and job failures.
- Reconciliation exceptions.
- Mint and burn throughput.
- Active incidents and open alerts.

### Public status page

Used by customers and partners.

- Service availability by major subsystem.
- Active incident notices.
- Maintenance windows.
- Last update timestamp.

The public page should stay user-safe and avoid exposing internal balances,
account data, or operational thresholds.

---

## Incident Workflow

1. Detect the anomaly through alerting or manual review.
2. Confirm scope and severity.
3. Check domain-specific guardrails first: reserves, oracle freshness, and
   partner status.
4. Apply circuit breakers or feature pauses if needed.
5. Communicate the issue internally and, when material, on the public status page.
6. Reconcile affected jobs and balances after recovery.
7. Record the root cause, timeline, and follow-up actions.

---

## Retention And Review

- Keep high-resolution metrics long enough for incident reconstruction.
- Retain logs and traces according to compliance and operational needs.
- Review dashboards and alert thresholds monthly.
- Revisit alert noise after every major incident.

---

## Implementation Checklist

- [ ] Health endpoints exist for liveness, readiness, and domain checks.
- [ ] Metrics are emitted for reserve, oracle, partner, and queue health.
- [ ] Logs contain correlation IDs and business context.
- [ ] Tracing covers settlement, minting, burning, and reconciliation flows.
- [ ] Alerts have owners, severity, and runbooks.
- [ ] Internal and public dashboards are separated.
- [ ] Status page update process is documented.
- [ ] Monitoring thresholds are reviewed regularly.

---

## Related Documents

- [Technical Architecture](ARCHITECTURE.MD)
- [Reserve Management](RESERVE_MANAGEMENT.MD)
- [Oracle System](ORACLE_SYSTEM.MD)
- [Risk Management](../OPERATIONS/RISK_MANAGEMENT.MD)
- [Backup and Recovery Plan](../OPERATIONS/BACKUP_AND_RECOVERY_PLAN.MD)
