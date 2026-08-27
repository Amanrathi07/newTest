# 01 — Business Strategy

## 1. The opportunity

US household moving is a ~$18B/yr industry, and the fastest-growing slice is **labor-only** (also called "moving help", "load/unload only"). The driver is the rise of DIY transport: U-Haul, Penske, PODS, U-Pack, 1-800-PACK-RAT and public storage. The customer already has the truck or container — what they lack is **two to four strong, reliable bodies for three hours**.

The scarce resource in this market is not demand, and not software. It is **screened, insured, reliably-showing-up labor, available in a specific ZIP code on a specific Saturday morning**. Marketplaces that solved discovery (HireAHelper) still suffer provider no-shows. Ops companies that solved reliability (Caddy) cannot scale beyond the metros where they personally recruit.

**CrewLink attacks the supply side.** We build the screened labor pool and rent it to whoever owns the demand.

---

## 2. Competitor teardown

### 2.1 HireAHelper (Porch Group)

| Dimension | Detail |
|---|---|
| Model | Consumer marketplace / aggregator |
| Supply unit | ~1,700 **provider companies** (not individuals) |
| Pricing | Hourly, `helpers x hours`, 2-hour minimum typical; provider sets rate, platform shows upfront quote |
| Revenue | Commission on booking value |
| Trust | $1,000 damage coverage free; $.60/lb up to $10,000; optional Full Value via MovingInsurance.com for 4.5-star+ providers |
| Cancellation | Free to 48 hrs; inside 24 hrs = 2 hrs billed; inside 1 hr = full price |
| Service norms | One flight of stairs included in all rates; providers list equipment and surcharges on profile |
| Moat | SEO. Fifty state landing pages plus city pages, 1M+ completed moves, 9,100+ reviews at 4.6 stars |
| Provider tooling | Provider Sign Up, Provider Login, "Provider Software", "Movers Academy" |

**Read:** HireAHelper is a demand-generation and trust machine. Its weakness is that supply is *companies* it does not control. A provider who takes a better job elsewhere on Saturday simply cancels, and HireAHelper eats the reputational cost.

### 2.2 Caddy Moving

| Dimension | Detail |
|---|---|
| Model | Labor-on-demand, ops-led (vertically integrated crews) |
| Positioning | Verbatim: *"Movers for your U-Haul, POD, or storage unit."* Explicitly labor-only. |
| Speed | **2-hour booking lead time** — the headline differentiator |
| Trust | Background-checked, **$1M insured** |
| Support | "Real humans on the phone, 7 days a week", (888) 818-8049 |
| Scope | Load, unload, pack, in-home moves ("wrestle that sleeper sofa down four flights") |

**Read:** Caddy competes on reliability and speed, not on breadth. Because it controls its crews, it can promise a 2-hour lead time. The cost is that every new metro requires a local recruiting and ops push — linear, expensive growth.

### 2.3 Where CrewLink sits

```
                 Controls supply
                        ^
                        |
            Caddy  o    |    o CrewLink
                        |      (control + API-scalable)
   ---------------------+--------------------->  Scales without local ops
                        |
     TaskRabbit o       |    o HireAHelper
                        |
```

We take Caddy's **control over individual workers** and HireAHelper's **nationwide reach**, and we skip the most expensive part of both — consumer acquisition — by taking demand wholesale from SCS and Muvr.

---

## 3. Positioning statement

> For **moving demand owners** (SCS, Muvr, and later container and truck-rental brands) who cannot reliably staff jobs across all 50 states, **CrewLink** is a labor fulfilment network that supplies screened, scheduled, insured moving crews on demand via API. Unlike a marketplace, we screen and control every individual worker; unlike an ops company, we scale to any ZIP code without opening an office.

---

## 4. Two-sided value proposition

### Supply side — Movers (students, gig workers, off-shift tradespeople)

- **Real money for physical work.** 70% of job value split across the crew, typically $22-$35/hr effective.
- **Zero customer acquisition.** No bidding, no profile marketing, no sales. Jobs are pushed to your phone.
- **Fits a class schedule.** Weekly availability grid; you are only offered jobs inside your declared windows.
- **Fast payout.** Standard 2-day ACH free; instant payout for a 1% fee.
- **Career ladder.** Mover -> Lead Mover (crew chief premium) -> Regional Captain (recruiting bonus).

Our beachhead is **university towns**. Students are strong, cheap to reach (campus job boards, RA networks, Greek life, athletics), naturally clustered, and want weekend work. Their weakness — churn every May — is precisely why an automated screening and onboarding pipeline (STRONG) is the core asset.

### Demand side — Partners (SCS, Muvr)

- **50-state coverage from one integration.** One `POST /jobs`, we handle the rest.
- **Fill-rate SLA.** Target 95% fill on >=48 hrs notice, 85% on same-day.
- **No employment overhead.** Screening, insurance, 1099s, payouts are ours.
- **Transparent economics.** Flat 30% of gross job value. No hidden lead fees.

---

## 5. Business model & unit economics

**Revenue:** 30% of gross job value on every completed job. That is the only Phase-1 revenue line.

### Representative job

| Line | Amount |
|---|---|
| Job: 2 movers x 3 hours @ $65/hr crew rate | **$390.00** gross |
| Platform fee (30%) | $117.00 |
| Crew pool (70%) | $273.00 |
| -> Lead Mover (1.15x weight, 3 hrs) | $146.04 |
| -> Mover (1.0x weight, 3 hrs) | $126.96 |

### Contribution margin per job

| Line | Amount | Note |
|---|---|---|
| Platform fee | $117.00 | |
| Stripe processing (2.9% + $0.30 on $390) | ($11.61) | Only if we collect from partner by card; ACH invoicing drops this to ~$0.80 |
| Stripe Connect payout (2 x $0.25) | ($0.50) | Standard ACH |
| Occurrence insurance allocation | ($9.00) | See section 7 |
| SMS/push/notifications | ($0.35) | Twilio, ~10 messages incl. dispatch broadcast |
| Support allocation (0.18 tickets x $6) | ($1.08) | |
| Screening amortisation | ($2.10) | $42 all-in screening / 20 jobs avg mover lifetime |
| **Contribution margin** | **$92.36** | **23.7% of gross, 79% of revenue** |

### Mover acquisition & payback

| Metric | Value |
|---|---|
| Blended CAC per applicant (campus + paid social) | $18 |
| Applicant -> screened & activated conversion | 34% |
| **CAC per activated Mover** | **~$53** |
| Screening + background check cost | $42 |
| **Fully-loaded acquisition cost** | **$95** |
| Jobs per Mover lifetime (student, ~2 semesters) | 20 |
| Lifetime contribution | $92.36 x 20 = **$1,847** |
| **Payback** | **~1.1 jobs** |

*The economics are extremely favourable because we do not pay for consumer demand. The whole model rests on partner demand being real and durable — which is also its largest single risk (section 8).*

### Scale scenario, Year 2

| | Conservative | Base | Aggressive |
|---|---|---|---|
| Active metros | 18 | 40 | 75 |
| Jobs/month | 1,800 | 6,000 | 14,000 |
| GMV/month | $702K | $2.34M | $5.46M |
| Revenue/month (30%) | $210K | $702K | $1.64M |
| Contribution/month | $166K | $554K | $1.29M |
| Fixed opex/month (team, infra, insurance base) | $185K | $310K | $520K |
| **EBITDA/month** | **($19K)** | **$244K** | **$770K** |

Break-even sits at roughly **2,100 jobs/month**, or about 22 active metros at base productivity.

---

## 6. Go-to-market

### Phase A — One metro, prove the loop (Months 0-3)
Pick a single dense university metro (Austin, Columbus, Raleigh or Tempe). Recruit 60 Movers through campus channels. Take live jobs from **one** partner only. Objective is not revenue; it is proving fill rate >= 90% and CSAT >= 4.6 with real money moving through the ledger.

**Gate to Phase B:** 200 completed jobs, <=3% no-show rate, >=4.6 stars, positive contribution margin.

### Phase B — Regional cluster (Months 4-9)
Expand to 12-18 metros in two contiguous regions. Regions, not scattered cities — it lets one Regional Captain and one support shift cover several markets, and lets Movers travel between nearby markets. Turn on the second demand partner. Launch the referral engine ($50 per referred Mover who completes 3 jobs) — in a student population this normally becomes the cheapest channel by month 6.

### Phase C — National + self-serve demand (Months 10-18)
All 50 states via the \"recruit-on-demand\" pattern: when a job lands in a ZIP with no coverage, the system auto-opens a geo-targeted recruiting campaign for that ZIP and holds the job in `SOURCING` for up to 72 hours. Launch the public partner API and a self-serve moving-company portal so any local moving company can buy overflow labor.

### Phase D — Own the demand (Months 18+)
Only after supply density is defensible do we build a consumer-facing booking front end with state and city SEO pages, the HireAHelper playbook. At that point we capture 100% of job value instead of 30%. This is deliberately last: building consumer demand before supply is the classic marketplace failure.

### Recruiting channel mix (supply)

| Channel | CAC | Volume | Notes |
|---|---|---|---|
| University job boards (Handshake) | $8 | High | Best ROI; requires per-school employer account |
| Campus ambassadors | $22 | Medium | 1 paid ambassador per campus, commission per activation |
| Athletics / Greek / ROTC group deals | $12 | Medium | Bulk sign-ups, strong physique profile |
| Meta / TikTok geo-targeted | $31 | Elastic | The lever for on-demand ZIP coverage |
| Indeed / Craigslist gigs | $27 | Medium | Older, more reliable, less seasonal |
| Mover referral | $50 flat | Grows | Best retention; #1 channel by month 6 |

---

## 7. Insurance & trust strategy

Trust is the product. Both competitors lead with it and we must match or beat them.

| Coverage | Spec | Rationale |
|---|---|---|
| General Liability | $1M per occurrence / $2M aggregate | Matches Caddy's $1M headline |
| Cargo / goods in care, custody & control | $25K per job | Labor-only exposure is damage during handling |
| Standard damage coverage to customer | $0.60/lb, up to $10,000 | Industry norm, matches HireAHelper |
| Interior damage to customer property | $0.60/lb, up to $10,000 | |
| Free basic coverage | $1,000 per job | Matches HireAHelper headline |
| Occupational accident (1099 crews) | $500K medical / $50K AD&D | Non-negotiable for contractor labor |
| Excess / umbrella | $5M | Required by enterprise partners at scale |

Budget ~$9 per job allocated at Phase-B volume; higher per-job early because minimum premiums dominate.

### Screening standard (published, per mover)
1. Government ID verification + selfie liveness
2. SSN trace, national criminal database, sex offender registry, county criminal (7 yrs)
3. STRONG structured interview + reliability scorecard
4. Physical capability attestation (50 lb repeated lift, no limitations)
5. Safety module + quiz (>=80% to pass): lifting, stair/doorway navigation, wrap & pad, property protection
6. W-9 and contractor agreement executed

### Service policy (mirrors market norms so partners can resell us without re-papering)
- 2-hour minimum per booking; one flight of stairs included; surcharge beyond.
- Free cancellation to 48 hrs; inside 24 hrs bills 2 hours; inside 1 hr bills full.
- Crews do NOT drive customer vehicles in Phase 1 (different insurance class).
- Crews do NOT supply boxes, pads, or packing materials; they will pack customer-supplied materials.

---

## 8. Risks & mitigations

| # | Risk | Sev | Mitigation |
|---|---|---|---|
| R1 | **Demand concentration.** SCS or Muvr is >60% of volume; they can cut us off or squeeze the 30%. | Critical | Contracted minimums with notice; partner #3/#4 by month 9; accelerate Phase D consumer channel; keep integration layer partner-agnostic (new partner = ~2-week adapter) |
| R2 | **Worker misclassification.** 70/30 split + dispatch control can look like employment under CA AB5 / ABC test. | Critical | See 08-compliance. Geo-state classification engine; W2 in ABC-test states; never mandate acceptance; allow substitution and multi-homing |
| R3 | **No-shows destroy partner SLA.** | High | Reliability score with auto-deactivation; T-24h and T-2h confirmations; auto-assigned standby mover on jobs >=3 movers; no-show penalty + 3-strike removal |
| R4 | **Seasonality.** Moving peaks May-Sep; students vanish at semester end. | High | Recruit non-student segments (40% target mix) for winter; winter surge pay; expand into adjacent labor (junk removal, event setup, warehouse day labor) on the same pool |
| R5 | **Injury on a job.** | High | Occupational accident policy; mandatory safety module; 50+ single-person limit with two-person lift rule; in-app incident reporting with photos |
| R6 | **Damage claims eroding margin.** | Medium | Mandatory pre/post-job photo documentation (geotag + timestamp); claims reserve at 1.5% of GMV; deductible recovery only in cases of proven gross negligence |
| R7 | **STRONG dependency.** Screening sits in a third-party tool we own. | Medium | `ScreeningProvider` adapter interface; nightly mirror of all screening records; read-only operand if STRONG is down; CSV fallback path |
| R8 | **Payment/punch fraud, fake jobs, crew-customer collusion.** | Medium | Geofenced check-in, device fingerprinting, payout hold on first 2 jobs, anomaly detection |
| R9 | **Flight to cheaper margins.** | Low-Med | Move up-stack: fill-rate analytics, crew quality tiers, guaranteed-fill SKU for premium partners |

---

## 9. KPIs

**North Star metric: Filled Job Hours per Month.** Captures both sides — demand won and supply delivered.

| Tier | Metric | Target (steady state) |
|---|---|---|
| Marketplace | Fill rate, >=48 hrs notice | >=95% |
| Marketplace | Fill rate, same-day | >=85% |
| Marketplace | Time-to-fill (median) | <25 min |
| Marketplace | Broadcast -> accept conversion | >=18% |
| Quality | Customer CSAT | >=4.7 / 5 |
| Quality | No-show rate | <2% |
| Quality | Damage claim rate | <1.2% of jobs |
| Quality | On-time arrival (+/- 15 min) | >=92% |
| Supply | Active Movers (>=1 job in 30 days) | tracked per metro |
| Supply | Applicant -> activated conversion | >=34% |
| Supply | Screening cycle time | <72 hrs |
| Supply | Mover 90-day retention | >=45% |
| Financial | Take rate (realised) | >=28.5% |
| Financial | Contribution margin / job | >=$85 |
| Financial | Payout accuracy | 99.99% |

---

## 10. Strategic moat, in order of durability

1. **Screened supply density per ZIP.** The only asset that compounds and cannot be replicated with money alone. Density is what makes a 2-hour lead possible.
2. **Reliability data.** Thousands of jobs of per-mover no-show, on-time and CSAT history make our matching materially better than random dispatch.
3. **Partner API lock-in.** Once SCS's dispatch system posts to our endpoint, switching cost is an engineering project, not a procurement decision.
4. **Compliance infrastructure.** The 50-state classification, insurance and 1099 machinery is unglamorous, expensive, and a genuine barrier.
5. **Campus recruiting network.** Ambassador relationships at 200 schools are slow to replicate.
