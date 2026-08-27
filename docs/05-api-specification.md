# 05 — API Specification
Base URL: `https://api.crewlink.example/v1`. All responses `application/json`. Partners authenticate with a signed API key (`X-API-Key`) + `Idempotency-Key` header on every mutation.

## 1. Conventions
- Dates/times: ISO-8601 UTC.
- Money: integer **cents** (USC) unless `_usd` suffix implies cents-as-integers; document per field.
- Errors: `{ "error": { "code": "...", "message": "...", "details": {} } }`.
- Pagination: `?cursor=` + `?limit=` (max 100); responses return `next_cursor`.
- Idempotency: replay-safe; store idempotency key 24h; same key+payload returns original body.

## 2. Error codes
| code | http | meaning |
|---|---|---|
| invalid_request | 400 | validation |
| unauthorized | 401 | bad/missing key |
| forbidden | 403 | role insufficient |
| not_found | 404 | |
| conflict | 409 | state conflict (e.g. job already assigned) |
| rate_limited | 429 | |
| upstream_error | 502 | partner/Stripe/STRONG failed |
| duplicate | 409 | idempotent dup (returns original) |

## 3. Partner-facing endpoints

### POST /v1/jobs (create/validate)
Body:
```json
{
  "external_ref": "SCS-88213",
  "service_type": "load_unload",
  "start_at": "2026-09-12T14:00:00Z",
  "end_at": "2026-09-12T17:00:00Z",
  "crew_required": 2,
  "hours": 3.0,
  "pickup": { "address": "111 Example St", "zip": "78704", "lat": 30.24, "lon": -97.78 },
  "destination": { "address": "222 Elm Ave", "zip": "78705", "lat": 30.28, "lon": -97.74 },
  "gross_amount_usd": 39000,
  "customer_name": "Jordan P.",
  "customer_phone": "+15551234567",
  "notes": "3rd floor walkup, no elevator, two-person lift for sofa"
}
```
Response `201`:
```json
{ "job_id": "8f3c...", "status": "RECEIVED", "eta_fill_minutes": 18 }}
```
Errors: invalid payload (400), unmatched taxonomy (422 via 400 details), surprise duplicate (409 returns original body).

### GET /v1/jobs/{id} — fetch canonical job + status + crew

### GET /v1/partners/{id}/jobs?status=ACTIVE&cursor=&limit=
### PATCH /v1/jobs/{id}/cancel  — partner cancels (billing rules per policy)
### POST /v1/jobs/{id}/confirm — partner confirms completion -> triggers settle
### GET /v1/partners/{id}/invoices?period=

## 4. Partner webhooks (we send)
| event | payload note |
|---|---|
| `job.accepted` | job_id, assigned crew ids |
| `job.started` | assignment ids, timestamps |
| `job.completed` | final crew, actual hours, earnings |
| `job.completed.no_show` | markers |
| `job.cancelled` | reason + fee |
| `invoice.issued` | amount, due date |
Signed: `X-Webhook-Signature: HMAC(secret, body)`; respond 200 quickly, retry with backoff on failure.

## 5. Mover-facing endpoints (mover app)
| method | path | purpose |
|---|---|---|
| GET | /v1/mover/me | profile + scope |
| PATCH | /v1/mover/me | update availability, base rate, skills |
| GET | /v1/mover/me/offers?status= | list offers/broadcasts |
| POST | /v1/mover/me/offers/{id}/accept | accept (first wins) |
| POST | /v1/mover/me/offers/{id}/decline | decline |
| GET | /v1/mover/me/jobs | my assigned jobs |
| POST | /v1/jobs/{job_id}/checkin | geofenced check-in |
| POST | /v1/jobs/{job_id}/checkout | geofenced check-out + photo |
| POST | /v1/jobs/{job_id}/incident | report incident (photos) |
| GET | /v1/mover/me/payouts | payout history + status |
| POST | /v1/mover/me/payouts/instant | instant payout (1% fee) |

Mover bodies (e.g. check-in):
```json
{ "lat": 30.241, "lon": -97.781, "photo_ids": ["p1"] }
```

## 6. Admin/ops endpoints
| method | path | purpose |
|---|---|---|
| GET | /v1/admin/dashboard | ops KPI snapshot |
| POST | /v1/admin/jobs/{id}/escalate | force sourcing |
| POST | /v1/admin/jobs/{id}/noshows | mark no-show, apply policy |
| POST | /v1/admin/movers/{id}/status | suspend/reactivate |
| POST | /v1/admin/payouts/{id}/retry | retry failed payout |
| POST | /v1/admin/screening/{id}/refresh | re-sync STRONG + Checkr |
| GET | /v1/admin/ledger?account=&from=&to= | audit ledger |

## 7. Screening (STRONG) integration surface
If STRONG exposes REST: we act as **client** on `POST /screening` and receive a verdict webhook. Adapter maps STRONG status -> our `screening.strong_status`. If no API: nightly CSV import via `POST /v1/admin/screening/import`. Design pattern in 06.
