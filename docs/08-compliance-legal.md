# 08 — Compliance & Legal

> This is a planning document, **not legal advice**. Engage employment and tax counsel in each state before launch. The 70/30 split plus dispatch control is the single highest-risk assumption in the plan.

## 1. The core question
Are movers **independent contractors (1099)** or **employees (W2)**? For a labor-supply network that screens, schedules and dispatches crews, the ABC test and CA AB5 put contractors under heavy pressure.

### The three prongs (e.g. California ABC test / AB5)
A mover is an independent contractor only if ALL of:
- **(A)** Free from our control and direction in performing the work.
- **(B)** Performs work outside the usual course of our business (we're in the moving-labor business... so this prong often fails).
- **(C)** Customarily engaged in an independently established trade/occupation.

Prong B is the classic killer for gig platforms. Federal DOL's Independent Contractor Rule (2021) uses an **economic-realities (totality-of-circumstances)** test, while 2024 rescission restored a broader approach. State rules supersede for in-state work.

## 2. Classification posture by state
| Group | Examples | Posture |
|---|---|---|
| ABC/AB5 states | CA, NJ, MA, CT, IL (fare), WA, NY (some) | **Treat as W2** unless counsel confirms independent-contractor compliance |
| Broad IC-friendly | TX, FL, GA, OH, TN, AZ | 1099 with a strong, documented independence paper trail |
| Middle | most others | 1099 with tight legal review; document prongs A & C explicitly |

**Recommended default:** run a small **geo-classification engine** keyed to the mover's work state that routes each mover into the correct employment model (1099 or W2) — which is why [06 — Integrations](06-integrations.md) includes payroll (Gusto/Check) as V2.

## 3. Contractor independence guardrails (to keep 1099 defensible)
- **No mandated acceptance**: movers can decline any offer; no penalty for declining.
- **Substitution & multi-homing allowed** (mover may work for others/use a substitute).
- **Rate set by mover** within a capped band; we don't fix an employment wage.
- **Mover is not exclusively ours** and can hold their own insurance/racing identity.
- Access to the app doesn't require a lock-in contract.
- Avoid: uniforms/badges with our branding, mandatory shifts, performance discipline that amounts to control, non-compete.
- The 70/30 % split should be framed as a **service/placement fee**, not wage-withholding.

## 4. If W2 in some states (V2)
- Payroll via Gusto/Check; wage & hour (FLSA + state); overtime; meal/rest breaks; worker's comp in every state; payroll tax filing (SUTA/FUTA); time-clock compliance integrated with check-in/out.
- Anti-fraud: this materially changes economics; model it per-region before committing (see 01 §5).

## 5. Insurance (operational)
- **Occupational accident** coverage for contractors (medical + AD&D) — non-negotiable.
- **Commercial General Liability** $1M / $2M aggregate; **cargo** coverage for goods handled; **excess umbrella** $5M for enterprise partners.
- Convened certificates of insurance (COI) to partner on request.

## 6. Worker identity & eligibility
- **E-Verify / I-9**: confirm work authorization for all movers (MVP: I-9 attestation + SSN trace).
- **W-9** with TIN validation before first payout.
- Age: 18+ for moving labor.

## 7. Screening & background
- FCRA-compliant: consumer-report background check only with signed disclosure + authorization; provide pre-adverse/adverse action notices before adverse action.
- STRONG screening data and Checkr reports are consumer-report-backed: access-scoped to Finance/Ops; retention per FCRA (max 7 years, shorter by statute).
- Document the screening standard publicly (01 §7) and in TOS.

## 8. Consumer/partner terms
- Platform **ToS**: marketplace-facilitation disclaimer; liability cap; dispute process; cancellation policy (mirrors industry norms so partners can resell).
- **Payroll 1099** disclaimers: we're not an employer; providers are independent.
- Privacy Policy: PII collection, dataset map, US residency, deletion rights; vendor list (Stripe, Checkr, Twilio, Push).
- Partner agreement: fill-rate SLA, payment terms (net 30 / per-job card), insurance coverage, indemnification, DPA.

## 9. Tax
- 1099-NEC annual for contractor movers above threshold.
- Sales/use tax: generally labour services not taxable, but confirm state-by-state (no nexus issues from PII storage? data residency in US).
- State-by-state **payroll/withholding** only where W2 applicable.

## 10. Team & owner
| role | responsibility |
|---|---|
| Ops/Compliance lead | policy, state matrix owner, incident |
| Employment counsel | classification per state, contractor agreements |
| Tax counsel | 1099, nexus, W2 states |
| Insurance broker | bind COIs, claims |
| Finance | W-9/TIN, reserves, audits |

### Immediate legal checklist (pre-launch, Phase A state)
1. Employment-law opinion for the chosen launch state (contractor vs employee).
2. Execute form contractor agreement (prongs A & C documented).
3. Bind GL, cargo, occupational accident, excess policies.
4. FCRA disclosure + authorization flow in STRONG/onboarding.
5. W-9 + I-9 flows.
6. TOS + Privacy + DPA + partner agreement.
7. Confirm STRONG data handling + retention for screening mirror.
