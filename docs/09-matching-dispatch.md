# 09 — Matching & Dispatch

Goal: turn a canonical job into an assigned, confirmed, reliable crew — fast, with the highest fill and lowest no-show probability per market.

## 1. Eligibility filter (cheap, SQL/PostGIS first)
Given a job at `(lat, lon)` starting at time T, candidate pool = ACTIVE movers who:
- Have availability covering that weekday/time window (local timezone).
- Live within their declared commute radius of the job ZIP (PostGIS `ST_DWithin`).
- Are not at their daily jobs/hours cap.
- Meet minimum reliability and rating thresholds.
- Are not on an ops leave/blackout override.
Pool capped at ~50 per market.

## 2. Scoring (rank candidates)
```
score =
     0.40 * reliability      # historical show-up reliability
   + 0.25 * rating           # customer/ops rating
   + 0.15 * proximity decay  # closer is better (distance in km)
   + 0.10 * offer-fairness   # lift movers who got fewer recent offers
   + 0.10 * skill/equipment  # e.g. +bonus if owns dolly/straps or marks heavy-item skill
   - 0.05 * recent-declines  # modest discouragement
   + lead_credit             # bump for LEAD slots when picking a crew chief
```
Weights tuned via A/B in the field. Store the final score and its component breakdown in `job_assignments.meta` for audit and tuning.

## 3. Broadcast & negotiation
- **Top-N broadcast**: push/SMS offer to top N (default 6) eligible movers with a TTL (default 90s).
- **First-accept-wins**: first acceptance for a role locks it; remaining offered movers receive a REGRET notification.
- **Roles**: broadcast LEAD and MOVER positions; a mover accepts a single slot per job.
- If a role is not filled within the TTL, widen radius, raise N, or relax the reliability tier; escalate to ops (manual sourcing) after ~90 minutes total.
- **Standby**: for jobs requiring >= 3 movers, auto-engage a standby mover at low pay-if-unused to hedge no-shows.

### Same-day acceleration
For jobs with under 4 hours notice, prioritise currently-online (presence) movers and shorten TTL to ~45s. These jobs carry the strictest fill SLA, so part more aggressively.

## 4. Confirmation & reminders
| time | action |
|---|---|
| assignment | send job brief (addresses, timing, crew, notes, photos) |
| T-24h | confirmation request; non-responders flagged |
| T-6h | reminder + ETA link |
| T-2h | final confirmation; detect risk -> engage standby |
| T-0 | geofenced check-in; miss = NO_SHOW path |

## 5. No-show & escalation policy
- No-show = assigned, confirmed, absent at start window.
- Auto-actions: engage standby, re-broadcast the role, notify ops, mark job MISSED if unfillable.
- Mover effects: reliability penalty + warning; 3 in 90 days -> SUSPENDED (threshold configurable per market).
- Partner gets a fast compensated rebooking.

## 6. Ops tools (admin)
- Live job board: today's jobs, fill status, crew, ETA.
- Manual assign / broadcast override per job.
- Market telemetry: time-to-fill, fill-rate, no-show rate, and a PostGIS supply-gap heat map (demand vs ACTIVE movers per ZIP).
- **Recruit-on-demand** trigger: a job lands in a ZIP below supply threshold -> auto-open a geo-targeted recruiting campaign (01, Phase C).

## 7. Fairness & supply health
- Rotate top offers so strong movers do not monopolise the best jobs (offer-fairness term above).
- Track per-mover offers, accepts, declines, no-shows, and earnings; show a personal portfolio.
- Weekly ops review of fill-rate by market, hour-of-day, and skill gaps.

## 8. Metrics (feed 01 section 9)
- Time-to-fill (median, split by market and lead-time bucket).
- Broadcast -> accept conversion.
- Fill rate by lead-time (>=48h vs same-day).
- No-show rate by mover segment, market, day.
