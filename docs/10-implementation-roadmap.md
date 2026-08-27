# 10 — Implementation Roadmap

## 1. Phases overview
| Phase | Name | Duration | Goal | Exit gate |
|---|---|---|---|---|
| 0 | Foundations | weeks 1-4 | repo, CI, envs, auth, DB, ledger skeleton | green CI + dev env |
| 1 | Core loop (single metro, 1 partner) | weeks 5-16 | ingest -> match -> execute -> pay round-trip | 200 jobs, fill >=90%, CSAT >=4.6 |
| 2 | Reliability & scale | weeks 17-32 | standby, no-show mgmt, admin, analytics, second partner | 12-18 metros, positive contribution |
| 3 | National & partner platform | months 9-18 | 50 states, self-serve portal, public API, native mover app | 50-state coverage, 95% fill >=48h |
| 4 | Own-demand (consumer) | months 18+ | consumer booking + SEO (optional/strategic) | Phase D of 01 |

## 2. Phase 0 — Foundations (weeks 1-4)
\[s] - Setup monorepo, TS, lint, format, CI (GitHub Actions).
\[s] - Envs dev/staging/prod; secret manager.
\[s] - Postgres + PostGIS provisioning (Neon MVP) + Flyway baseline migrations.
\[s] - NestJS API skeleton with RBAC + Clerk auth.
\[s] - Ledger schema + money utilities (integer cents) + balance invariant tests.
\[s] - Testing: Jest + Pact (contracts), Playwright for web smoke.
\n## 3. Phase 1 — Core loop (weeks 5-16)
Bulletin sprint-by-sprint (2-wk sprints by default):
- **S1**: Mover signup journey (ID/KBA stub) + identity + basic profile.
- **S2**: STRONG integration (adapter + webhook/CSV) + Checkr; status machine + mirror.
- **S3**: Availability grid + PostGIS geo; W-9 e-sign + contractor agreement.
- **S4**: Job ingestion (canonical Job + SCS adapter + idempotency) + validation.
- **S5**: Matching + broadcast (offer TTL, first-accept) + notifications (Twilio/push).
- **S6**: Execution (assignment, reminders, geofenced check-in/out, incident).
- **S7**: Payments (capture -> 70/30 split -> payout -> ledger) + payout holds.
- **S8**: Mover app (PWA) polish: job feed, my payouts, live status.
- **S9**: Admin console read-mostly: job board, screening review, payouts.
- **S10**: Hardening: reconciliation, retries, load test, error budgets; launch pilot metro.

## 4. Phase 2 — Reliability & scale (weeks 17-32)
- Standby mover engine + no-show auto-response.
- Time-to-fill telemetry + ops alerts.
- Claims/photo documentation (V2 claims flow).
- Admin: full ops toolkit, live map, market heat map, recruit-on-demand.
- Second demand partner (Muvr) via shared JobIntake port.
- Regional captains onboarding; referral engine; winter off-peak recruiting mix.
- Instant payout (1% Stripe) + payout anomaly detection.

## 5. Phase 3 — National & partner platform (months 9-18)
- All 50 states via geo supply; recruit-on-demand at scale.
- **Native Expo mover app** (background GPS, reliable push, offline queue).
- Partner self-serve portal + **public partner API + docs** (Redoc/Postman).
- Enterprise partners: SLAs, premium fill SKU, COIs on demand.
- Compliance engine rollout (1099/W2 routing by state, see 08).
- Migration to AWS ECS Fargate + RDS + ElastiCache for control-plane needs.

## 6. Phase 4 — Own-demand (strategic, months 18+)
- Consumer-facing booking front end with state/city SEO pages (HireAHelper playbook).
- Capture 100% of job value on direct bookings; keep 30% on partner jobs.
- Requires building demand on top of defensible supply density first (05 moat).

## 7. Team & skills
| role | FTE (Phase 1) | notes |
|---|---|---|
| Engineering lead / full-stack | 1 | Next.js + NestJS + infra |
| Full-stack / mobile | 1 | PWA -> Expo in Phase 3 |
| Ops / support lead | 1 | dispatch, screening review, partners |
| Marketplace / growth (part-time) | 0.5 | campus recruiting, partner liaison |
| Compliance / legal counsel (external) | as needed | classification + contracts |
| Design (fractional) | 0.25 | app + admin UX |

## 8. Budget estimate (Phase 1, 4 months)
| item | est. |
|---|---|
| Engineering (1.5-2 FTE) | $60k-$90k |
| Design (fractional) | $6k |
| Infra (Render/Neon/Upstash, staging+prod) | $800/mo |
| STRONG + Checkr (sandbox -> prod) | $1.5k/mo |
| Stripe processing | variable (see 01) |
| Insurance (GL, cargo, occ-acc base) | $3k/mo |
| Twilio / push | $300/mo |
| Legal (classification opinion + contracts) | $15k-$30k one-off |
| Campus recruiting + channels (Phase A) | $3k/mo |

## 9. Immediate next steps (this week)
1. **Confirm the 4 assumptions** (A1-A4) with business owner — especially money flow (A1) and worker classification (A2); they change Stripe and legal design.
2. Contact STRONG owner for API/ webhook availability and sandbox access.
3. Get employment-law opinion for the pilot metro (likely Austin TX, contractor-friendly).
4. Sign partner LOI with SCS with fill-rate + payment terms.
5. Stand up monorepo + CI + dev DB per Phase 0.

## 10. Definition of "done" (Phase 1 pilot)
- Mover can go APPLICANT -> ACTIVE (STRONG + Checkr + W-9) in < 72h.
- One partner posts jobs; job moves RECEIVED -> SETTLED with money in ledger and payouts T+2.
- 60-100 active movers; 200 completed jobs; no-show < 3%; CSAT >= 4.6.
- Automated reconciliation passes nightly with zero drift; audit query answers any payout.
- Second partner adapter is a 2-week task (proof of partner-agnostic design).
