# 06 — Integrations

## 1. Integration principles
- Every external dependency is behind an **adapter interface** so a vendor swap changes one file, not the codebase.
- **Webhooks are retried** with exponential backoff and an outbox for reliable delivery.
- **Mirrors**: screening and job data from partners are periodically mirrored locally so core ops never hard-depend on an external system being up.
- All outbound calls are **idempotency-keyed** and time-boxed (timeout + circuit breaker).

## 2. STRONG Sales Admin (screening of record)
`STRONG` is the external screening application. Status: **unknown whether it exposes a public API/webhooks; assumption A3 — it does.** If it doesn't, use the CSV fallback.

### Capabilities we require from STRONG
1. Create a screening candidate for a mover (ID verif + structured interviews/assessments).
2. Retrieve a verdict (`pass` / `fail` / `under review`) with a reason.
3. Notify us on verdict change (webhook) OR we poll/import.

### Adapter pattern
```
interface ScreeningProvider {
  createCandidate(mover): ScreeningId
  checkVerdict(id): ScreeningResult
  onVerdict(webhook): void
}
class StrongHttpProvider implements ScreeningProvider { ... }   // A3
class StrongCsvProvider   implements ScreeningProvider { ... }   // fallback
```

### Data flow
```
Mover ACTIVATED
  -> POST idempotent to STRONG (candidate + PII)
  -> STRONG validates via its dashboard (STRONG side)
  -> verdict webhook received -> screening.strong_status updated
  -> if pass -> proceed to Checkr; if fail -> REJECTED
Mirror: nightly job syncs all screening rows into local `screening` table.
Ops admin can view STRONG status read-only through the mirror (no round-trip on every screen).
```

### Open item to confirm with STRONG owner
- Auth method (API key / OAuth) and endpoints.
- Whether assessments are synchronous or async.
- Data-retention and PII export/delete (GDPR-style rights even for US as a safeguard).
- Sandbox availability for staging.

## 3. SCS website (demand)
| item | detail |
|---|---|
| Nature | primary demand source |
| Integration | webhook push OR periodic poll (whichever SCS offers); normalize to canonical Job |
| Fields mapped | external_ref, service_type, schedule, crew, hours, pricing, addresses, customer contact, notes |
| Failure handling | outbox + retries; reconcile via `partner_job_ref` dedupe |
| Contract | minimums + notice + fee structure confirmed (see 07) |

## 4. Muvr (demand)
Same canonical `Job` ingestion as SCS through a shared `JobIntake` port.

```
interface JobIntakePort {
  subscribe(): void            // webhook path
  poll(): JobDraft[]            // fallback path
  confirm(jobId, ref): void
}
class ScsIntake  implements JobIntakePort {}
class MuvrIntake implements JobIntakePort {}
```

## 5. Checkr (background checks)
- On STRONG pass, raise a Checkr candidate and run standard + county criminal package.
- Statuses mapped: `clear` / `consider` / `suspend`.
- Webhook completion updates `mover.status`; restricted report accessible only to Finance/Ops roles.
- Sandbox used in staging; production uses Checkr prod keys scoped per environment.

## 6. Stripe Connect (money) — see 07 for topology
- **Express accounts** for movers (KYC on onboarding, payouts ready).
- Destination charges to capture partner funds, or invoiced platform-fee topology (see 07 choice)
- Instant payout product for the 1% fee.

## 7. Twilio (notifications)
- SMS for dispatch offers (with TTL warning), confirmations, no-show escalations.
- Onboarding OTPs, verification, and support routing.
- Delivery receipts feed the fill-rate telemetry.

## 8. Map APIs
- Geocoding of job addresses to lat/lon (stored once at ingest).
- Geofence validation for check-in/out.
- ETA to job site for movers (fire-and-forget notification).

## 9. Payroll / tax (V2, if W2 in ABC-test states)
- Gusto or Check integration for jurisdictions requiring W2 payroll; workers' comp module per state.
- Mirrors the classification outputs of 08.

## 10. Outbox / event bus
All integrations emit domain events to an internal outbox (transactionally with the DB write), published to BullMQ, ensuring at-least-once delivery and replay-ability for: notifications, payouts, partner updates, STRONG sync.
