# 07 — Payments & Payouts (30/70 split)

## 1. Money model (assumption A1: platform collects, then splits)

```
Partner pays CrewLink the FULL job value
       |
       +--> 30%  platform fee (revenue)
       +--> 70%  crew pool  -> split by (hours x weight) across assigned movers
                   -> per-mover PAYOUT via Stripe Connect (T+2 ACH, or 1% instant)
```

### Worked example (2 movers, 3 hrs, $390 gross)
| component | compute | amount |
|---|---|---|
| gross job value | 2 x $65/hr x 3 | $390.00 |
| platform fee (30%) | 390 x 0.30 | $117.00 |
| crew pool (70%) | 390 x 0.70 | $273.00 |
| Lead Mover weight | 1.15 | 273 x (1.15/(1.15+1.0)) | $146.04 |
| Mover weight | 1.00 | 273 x (1.00/(2.15)) | $126.96 |
| sum | | $273.00 (balanced) |

### Hourly-effective check
A $65/hr crew rate with 2 movers on-site means each mover's gross ~ $32.5/hr before split; after the 70% pool it lands ~ $21-$23/hr effective — competitive enough for students.

## 2. Stripe Connect topology (recommended: **destination charges**)
- **Each mover = Stripe Connect Express account** created at onboarding; KYC gating before payouts.
- Job is charged on the **partner's** Stripe account as a `destination charge` with `transfer_data` to the Lead Mover's account, then internal splits via on-behalf-of or a connected-account transfer.
- If a partner can't pay by card: use **invoicing** (ACH) and reconcile to the ledger, then pay out crews from our own settlement.

### Decision matrix
| Partner payment method | Topology | Notes |
|---|---|---|
| Card, amount captured per job | Destination charge + transfers | automatic; recommended for SCS/Muvr |
| ACH / invoice (net 30) | Platform-in-escrow then payout | we carry float; need credit risk |
| Partner pays movers directly (A1 upside down) | Account-hold + invoiced 30% fee | legal + simpler but different

## 3. Split & payout flow (server-side)
```
job COMPLETE -> FINALIZE
  -> lock job
  -> compute crew pool (gross x 0.70) using actual minutes per mover
  -> weighted split by role weight
  -> write per-mover PAYOUT (PENDING) idempotently
  -> write double-entry ledger (platform fee CREDIT, mover payable CREDIT, AR DEBIT)
  -> push payouts to Stripe Connect (standard T+2) with per-payout idempotency
  -> on webhook success mark PAID; on failure retry w/ backoff; never double-pay
```

### Payout schedule
| method | timeline | fee |
|---|---|---|
| Standard ACH | T+2 business days | $0.25 |
| Instant | minutes | 1% of amount; capped cap e.g. $25 |

### Payout holds
- First 2 jobs: earnings held until review (fraud control).
- No payout while `w9_verified = false` (tax gate).

## 4. Ledger (double-entry)
Accounts: `AR_RECEIVABLE`, `PLATFORM_FEE`, `CREW_PAYABLE`, `MOVER_PAYABLE`, `CLAIM_RESERVE`, `ESCROW`.
- Every job posts fully balanced debits/credits in one DB transaction.
- Integer cents only; `CHECK >= 0` on amounts; direction by side.
- Immutable: ledger rows never updated/deleted; corrections are reversing entries.
- Audit: every entry has `reference` (payout/invoice id), `created_by`.

## 5. Refunds, cancellations & no-shows
| scenario | money effect |
|---|---|
| Cancel > 48h | no charge; nothing captured |
| Cancel 24-48h | 2 hours at booked rate billed (customer) |
| Cancel < 1h | full job billed |
| No-show (assigned mover) | payout reversed, penalty policy applied |
| Partial completion | settle on actual hours; refund difference |
| Damage claim (V2) | hold reserve, adjudicate, recover from gross negligence |

All refunds route through Stripe and mirror a reversing ledger entry in the same transaction.

## 6. Tax
- **1099-NEC** at year end for all paid movers (contractor status; assumption A2).
- **1099-MISC** for partners/companies if applicable.
- Annual filing via provider (e.g. Track1099 / Stripe Tax) using payout records.
- W-9 required and e-signed before any payout.

## 7. Reconciliation & fraud
- Daily job: `sum(gross captured) == sum(ledger sums) == sum(payouts + fees + reserves)`.
- Alert on: payout dup, capture/ledger mismatch, unbalanced job, payout > configurable high-water mark.
- **Anomaly detection**: crew pair with same customer repeatedly; check-in far from geofence; job created and cancelled instantly.
- **Money correctness** is the top NFR: no floats, idempotent everywhere, single-writer ledger.

## 8. Escrow consideration
Since partners pay after completion (net terms) rather than upfront, `ESCROW` is optional in MVP. Introduce escrow/trust accounting when a partner demands prepayment or when we carry inbound float. Documented here so the ledger has the account ready.
