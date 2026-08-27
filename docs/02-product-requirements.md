# 02 — Product Requirements

## 1. Personas

| ID | Persona | Primary app | Needs |
|---|---|---|---|
| P1 | **Mover** — student / gig worker | Mover app (PWA, later native) | See jobs in my city/availability; accept; check in/out; know my pay; withdraw |
| P2 | **Lead Mover / Crew Chief** — experienced mover leading a crew | Mover app (crew lead role) | Crew roster for the day; coordinate; confirm completion; handle issues |
| P3 | **Partner Dispatcher** (SCS / Muvr ops person) | Partner portal / API | Post job; see status; approve completion; invoice; get reports |
| P4 | **CrewLink Ops / Support Agent** | Admin console (web) | Review screening; resolve no-shows/disputes; monitor live jobs; override |
| P5 | **Compliance / Finance** | Admin console | Background check review; W-9 status; payout audits; 1099 exports |

## 2. Epic map (MVP scope)

```
[M] Onboarding & Screening (MVP)  ->  Mover app signup -> STRONG screen -> bg check -> W-9 -> ACTIVE
[M] Availability & Scoring                  (MVP) - single weekly grid; reliability signals
[M] Job Feed / Broadcast                    (MVP) - geo+sched matched, first-accept-wins
[F] Job Lifecycle & Execution               (MVP) - assign -> reminders -> check-in/out -> completion
[M] Payments                                (MVP) - 70/30 ledger, Stripe payout, payout history
[A] Partner API & Job Intake                (MVP via SCS) - webhook/PM, normalize to Job
[A] STRONG Admin Sync                       (MVP) - webhook mirror, status gate
[P] Job Gating & Deductions                 (core) - ownership
[A] Notifications (Twilio push)             (core)
[A] Admin console                          (core but incremental)
--- below is V2 ---
[V2] Partner self-serve portal + public API docs
[V2] Dispute resolution / claims / photos workflow
[V2] Consumer booking (Phase D)
```

## 3. User stories (MoSCoW)

### P1 (MUST), epic: Onboarding & Screening
- **US-01** As a Mover I complete a journey: verify identity (ID + live selfie), then assessments. **Accept:** profile enters PENDING_SCREEN.
- **US-02** As STRONG rigger I can push a screening verdict; system converts profile to SCREENED or REJECTED with reason. **Acccept:** status transition visible to partner and admin; audit log.
- **US-03** As Ops I trigger a background check via Checkr; completion webhook moves Mover to BACKGROUND_PENDING then ACTIVE. **Acccept:** Checkr candidate id stored, status not orphaned.
- **US-04** As Finance I must see and sign a W-9; Mover cannot accept paid jobs without executed W-9. **Acccept:** payout disabled until e-sign returned.
- **US-05** As a Mover I upload a photo, list my interests (weight/skills/equipment), set base hourly asking rate (capped). **Acccept:** saved to profile.

### P1, epic Availability
- **US-10** As a Mover I set a weekly availability grid (day/hour + radius + max hours + max jobs). **Acccept:** system respects all constraints in offers.
- **US-11** As Ops I can auto-deactivate a Mover with >3 no-shows in 90 days (policy flag). **Acccept:** flow jumps to REJECTED/INACTIVE.

### P1, epic Job intake (partner)
- **US-20** As a Dispatcher I post a job (payload that fits my schema) OR call `POST /v1/jobs`; it is validated and accepted with a `job_id`; rejected if >72h sourcing window with no coverage. **Accept:** idempotent (same payload same id), creates Job in RECEIVED.
- **US-21** As an integrator I call `GET /v1/partners/{id}/jobs?since=` for missed events; server replays. **Acccept:** cursor-based.

### P1, epic matching/dispatch
- **US-30** As the system I score eligible Movers (geo + availability + earnings) and surface a tiered broadcast. **Accept:** score fields logged with their weightings.
- **US-31** As a Mover I accept an offer; first acceptance wins; others receive a REGRET notice; job moves to ASSIGNED.
- **US-32** Same-day jobs are broadcast to currently-online Movers (presence) with job-score >= threshold, with a shorter offer TTL.

### P1, epic execution
- **US-40** As a Mover I view/pull job details (addresses, door notes, photos).
- **US-41** As a Mover I geofence-check-in at START; app records time/loc; timer starts. **geofence radius** per config.
- **US-42** As a Mover I check out at END; completion photo uploaded; both check-ins geotagged; timeline persists.
- **US-43** As Crew Lead I confirm handover & select ANY issue flag; if a claim, an incident record is auto-created.

### P1, epic payments
- **US-50** As Finance I see computed crew split (70/30) per job with per-mover hours&weights; every ledger entry immutable.
- **US-51** As a Mover I see my accrued balance; standard payout T+2 ACH; instant for 1%. **Acc.** amount cap; idempotency keys.
- **US-52** As Ops I initiate on-demand payout; Stripe Connect handles refund setup; dispute flow routes to old job.

### Core / V2
- **US-60** As Support I reverse-lookup any payout (ledger id -> Stripe tx).
- **US-70 (V2)** As a Dispatcher I self-serve via portal: schedule, cancel, reschedule, see live map of assigned crew, approve.
- **US-80 (V2)** Claims: photo intake, notes, party review, reserve deduction, final.

## 4. MVP vs V2 cut

| Capability | MVP | V2 |
|---|---|---|
| 1 demand partner, HTTP + webhook intake | YES | partners self-serve |
| STRONG integration via webhook + CSV fallback | YES | n/a |
| Mover app (PWA) screens | YES | native Expo |
| Payments (check-in/out, 70/30, Stripe payout) | YES | instant payout, withholdings |
| Notification chain (dispatch->accept->reminders) | YES | richer push |
| Geofence GPS check-in/out | YES (coarse) | precise + timeline |
| Admin console | read-mostly | full ops toolkit |
| Partner self-serve portal | NO | YES |
| Disputes/claims workflow | MAIL (manual) | self-serve |
| Consumer-facing booking | NO | Phase D |
| Public partner API with docs | NO | YES |

## 5. Core acceptance (end-to-end)

**Given** a partner posts a 2x3 job (2 movers, 3 hrs, $65/hr) at 10:00, ZIP 78704, Sat.
**And** two ACTIVE Movers have weekend availability over Austin with rating >= 4.0.
**When** the ingestion normalizes and the matcher scores & broadcasts.
**Then** each gets offer; both accept; job ASSIGNED; crew locked; partner notified.
**And** at job time both geofence-check in/out; $390 captured; 30% fee ($117) recorded to platform; 70% ($273) split (Lead 1.15x = $146.04, other $126.96) as pending payouts.
**And** both receive T+2 payout; both records show correct amounts; ledger balances 0 (no float money).

## 6. Important non-functional requirements
- **Idempotency everywhere:** every POST has `Idempotency-Key`; Stripe & our ledger both use it.
- **Correctness over speed in money:** immutable ledger; no in-memory floats for cash (use cents integers).
- **Fill SLA telemetry**: time-to-fill metric on every job; ops alert > 60 min.
- **Multi-region**: US-only; use UTC internally, render in mover local TZ.
- **Private of PII**: DC enforceable; screening data access-scoped.
