# 16 — RISKS & MITIGATIONS

_KisanSetu Brain · July 2026 · Owner: Alpesh (founder). Siblings: [02-MARKET-RESEARCH.md](02-MARKET-RESEARCH.md) (competitive facts), [13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md) (gates that operationalize kill criteria), [14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md) (SOPs that implement mitigations), [17-FUNDRAISE-FINANCE.md](17-FUNDRAISE-FINANCE.md) (runway math)._

This register exists so that failure modes are decided **in advance, in writing, while calm** — not rationalized in the moment. Each risk has: likelihood × impact, early-warning indicators (leading, not lagging), mitigations already embedded in the plan, a contingency if it fires anyway, and a **kill/pivot criterion** — the pre-committed line at which we stop pushing and change course.

**Review cadence:** monthly, in the extended first-Monday metrics ritual ([14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md) §10); likelihoods re-scored, new risks added, fired risks post-mortemed. The graveyard of 2024–25 (Otipy, Fraazo, ReshaMandi, Greenikk, Deep Rooted) is our memento mori: they died of asset-heaviness, cash burn, and unvalidated demand — the exact three things this register watches hardest.

## Register at a glance

| ID | Risk | Likelihood | Impact | Exposure window |
|---|---|---|---|---|
| R1 | Demand validation fails | Medium | Fatal to model | Phase 0 → pilot week 8 |
| R2 | FPO partner quality/reliability | Medium-High | High | Pilot onward |
| R3 | Cold-chain cost blowout | Medium | High (margin) | Pilot onward |
| R4 | Price war — Ninjacart (or other) re-focuses on our lane | Low-Medium | High | Month 6+ / post-traction |
| R5 | Farmer side-selling | High | Medium-High | Every price-spike day |
| R6 | Perishability & quality disputes | Medium | Medium (trust) | Every delivery |
| R7 | Regulatory — APMC/mandi-fee action | Low-Medium | Medium-High | Launch onward |
| R8 | Funding winter — seed round fails | Medium | High | Month 5–9 |
| R9 | Single-founder execution risk | High | Fatal if unmanaged | Always |
| R10 | Brand/trademark blocked ("KisanSetu") | Medium | Low-Medium | Phase 0 |
| R11 | Payment/fraud incident | Low | Medium | Pilot onward |
| R12 | Supply shock (monsoon/pest/price crash) | Medium | Medium | Seasonal |

---

## R1 — Demand validation fails (buyers won't actually switch)
**The risk:** HoReCa buyers say yes in interviews but won't leave their credit-giving, relationship-based existing suppliers; or will only switch for discounts that destroy the 8–12% take rate.
**Early warnings:** interviewees ask mainly about price, never about quality/traceability; pilot buyers order once and revert; M1 repeat rate <50%; buyers demand 30+ day credit as a precondition.
**Mitigation (already in plan):** Phase 0 interviews before any build (T0.2); sample-crate demos so the product argues for itself; graded-quality + stable-price positioning rather than cheapest-price; net-7 max credit terms at pilot ([14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md) §8.5).
**Contingency:** narrow to the buyer sub-type with best repeat (cloud kitchens are the likely candidate — volume-consistent, chef-less procurement); re-test price positioning ±5%.
**KILL/PIVOT CRITERION (pre-committed):** <3 of 10 Phase-0 buyers show real willingness to switch ⇒ **do not build the marketplace**; pivot to FPO-procurement-SaaS-first wedge (sell grading/procurement software to those already moving produce). Post-launch equivalent: G1 gate failure on M1 with ops metrics passing ⇒ same pivot ([13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md) §6).

## R2 — FPO partner quality/reliability
**The risk:** the single pilot FPO under-delivers — inconsistent grading, politics inside the FPO board, staff turnover, or the FPO treating us as one buyer among many rather than a partner. With 1 FPO we have a single point of supply failure.
**Early warnings:** M8 supply reliability <90%; grading photo audits show chart drift; hub staff rotate without notice; FPO leadership misses two monthly reviews.
**Mitigation:** MoU with explicit service expectations + exit clauses ([18-LEGAL-COMPLIANCE.md](18-LEGAL-COMPLIANCE.md) §6); we train and certify graders ourselves; grading photos create an objective quality record; monthly FPO review in the ritual; FPO gets visible wins early (payout slips, farmer goodwill) so the partnership is worth protecting; second FPO relationship warmed (met, not signed) from month 2.
**Contingency:** activate FPO #2 within 4 weeks (the warm relationship is the insurance premium); in extremis run 2 weeks of direct village-aggregator intake at the same hub while transitioning.
**Kill criterion for the partner (not the company):** M8 <75% for 3 consecutive weeks, or documented grading manipulation ⇒ invoke MoU exit, switch FPOs. Two consecutive failed FPO partnerships ⇒ re-examine whether the FPO-hub model works in this district at all (structural, not partner, problem — escalate to model review).

## R3 — Cold-chain cost blowout
**The risk:** per-trip 3PL costs + shrinkage exceed the logistics fee + take rate; the "asset-light" model just relocates the cost instead of removing it. This killed the capital-heavy players from the other direction — we must not die of the per-trip version.
**Early warnings:** logistics cost >15% of GMV for 2 consecutive weeks; shrinkage (quality loss ledger) >3% of GMV; 3PL raises rates after we're dependent; route utilization <60% of vehicle capacity.
**Mitigation:** per-trip contracts with rate cards, no retainers ([14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md) §6); two trial routes before signing; vendor #2 kept warm for price tension; delivery-window batching (06:00–10:00) to maximize drops per trip; insulated-box standard instead of premature reefer; logistics fee passed to buyer as an explicit line item so cost inflation is visible, not silently margin-eating.
**Contingency:** re-price delivery fee (buyers accept visible fuel-style adjustments better than produce-price games); consolidate delivery days for small buyers (3×/week instead of daily); drop unprofitable routes — a smaller profitable pilot beats a bigger bleeding one.
**Kill criterion:** contribution margin per order still negative at week 16 **with** route utilization >70% and take rate in band ⇒ the unit economics don't work at tier-2 density; halt expansion, shrink to the profitable core, and re-run the model with FPO-SaaS revenue in the mix before raising or scaling.

## R4 — Price war: Ninjacart (or another funded player) re-focuses on our lane
**The risk:** our wedge exists because Ninjacart pivoted away from focused fresh-to-HoReCa ([02-MARKET-RESEARCH.md](02-MARKET-RESEARCH.md)). Traction can invite them back — or invite DeHaat to bolt on buyer-side delivery — with below-cost pricing we cannot match.
**Early warnings:** competitor sales reps visiting our named buyers; sudden below-mandi pricing in Surat HoReCa; competitor job postings for Surat/Gujarat fresh-supply roles; our CAC rising while M1 falls among newest cohorts.
**Mitigation:** the moat is deliberately non-price: traceability, published freshness SLA, farmer-share story, and switching costs via standing orders + WhatsApp habit; B2B relationships defend better than app users (founder knows all ≤25 buyers personally); tier-2 focus is itself protection — Surat is small for their cost structure; FPO SaaS deepens the supply-side lock.
**Contingency:** do not match below-cost prices (pre-committed — subsidy wars are how the 2024–25 graveyard filled); defend on the accounts where we have SLA history; accelerate crop breadth for existing buyers (share-of-wallet beats buyer count in a knife fight); consider the FPO-SaaS flank where a marketplace competitor can't follow easily.
**Kill criterion:** losing >30% of 8-week+ tenured buyers to a subsidizing competitor over 8 weeks despite SLA being met ⇒ the buyers were price-loyal, not quality-loyal; retreat from that segment, refocus on the sub-segment that stayed, and revisit the SaaS pivot rather than out-burning a Walmart-backed balance sheet.

## R5 — Farmer side-selling
**The risk:** on price-spike days farmers sell committed produce to the mandi/local traders instead of delivering; our fulfilment promise to buyers breaks. This is the classic agri-marketplace failure mode and it is *rational farmer behavior*, not betrayal.
**Early warnings:** M8 kg-delivered/kg-committed <90%; shortfalls correlated with mandi price spikes; farmers listing less while attending the hub less.
**Mitigation:** instant same-visit UPI payment (the strongest anti-side-selling tool that exists — cash-now beats maybe-more-later); daily price referenced to the mandi so we never drift below market for long; the movement damper works both ways — we track spikes and can pay up on spike days within the farmer-share constraint; FPO social enforcement (the MoU makes reliability a hub norm); over-commit buffer: confirm to buyers only ~85% of listed supply in week 1–4 until reliability data accumulates (apply at the evening provisional confirm, [14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md) §4.2).
**Contingency:** per-farmer reliability score (delivered/committed) drives allocation priority and demo-day recognition (public trust, not public shaming); on spike days, proactively partial-fill buyers by 19:30 per SOP rather than discovering shortfall at dawn.
**Kill criterion (for the mechanism, not the company):** if even with instant payment and market-tracking prices, M8 stays <75% across 6 weeks and two price regimes ⇒ listing-based day-ahead supply doesn't work here; switch to morning spot-intake model (grade whatever arrives, sell same day) and redesign the promise around it.

## R6 — Perishability & quality disputes
**The risk:** produce degrades between grading and delivery; buyer's "Grade A" expectation diverges from the chart; disputes erode trust in both directions and shrinkage eats margin.
**Early warnings:** M7 dispute rate >5%; disputes clustering on one crop/route/farmer; repeat disputes from the same buyer (expectation mismatch, not quality).
**Mitigation:** photo grade charts at hub **and** with every buyer (same images — one shared truth); grading photo per lot as evidence ([14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md) §2.3.4); 2-hour claim window; pre-dawn harvest instruction + <36h median target keeps degradation physics on our side; crop choice at T0.6 explicitly weighs gradability and hardiness.
**Contingency:** same-day credit-note resolution (speed of resolution is remembered longer than the defect); crop-specific fixes (ice packs, max-fill lines, route resequencing); drop a crop that can't survive the chain — 2 crops was always a starting set, not a commitment to suffer.
**Kill criterion:** dispute rate >10% for 4 consecutive weeks on both crops after chart recalibration and route fixes ⇒ our grading-at-hub model doesn't transfer quality reliably; pause buyer acquisition and redesign QC (e.g., grade-at-dispatch re-check step) before scaling further.

## R7 — Regulatory: APMC/mandi-fee action
**The risk:** direct procurement outside the market yard is challenged — a demand for market fees/cess, a license question, or local trader-lobby pressure via the APMC — especially once we're visible.
**Early warnings:** notice from the APMC; mandi traders raising us at association meetings; our 3PL or FPO getting informal warnings; press coverage naming us before our compliance file is solid.
**Mitigation:** T0.8 legal memo from a Gujarat advocate **before** launch — position on Gujarat APMC Act (as amended), whether/where a unified/trader license or fee applies to hub-gate procurement, filed with evidence ([18-LEGAL-COMPLIANCE.md](18-LEGAL-COMPLIANCE.md) §5); procure at the FPO's hub (FPO's own farmer-member produce — the strongest structural position); obtain whatever license is cheap and available even if arguably unnecessary; keep the farmer-benefit narrative documented (payout slips are political armor: we demonstrably pay farmers more).
**Contingency:** pay disputed fees under protest while operating (never halt deliveries over a recoverable fee); escalate through the FPO federation and SFAC channels; restructure procurement contractually through the FPO as seller-of-record if that resolves the license question.
**Kill criterion (geography-level):** a binding order prohibiting the model in Gujarat with no licensable path within 8 weeks ⇒ relocate the beachhead to a state whose regime clearly permits it — the model is global-first by design; Gujarat is a choice, not a dependency.

## R8 — Funding winter: seed round fails
**The risk:** agritech funding remains selective (~$160M in 2025); even with a working pilot the seed doesn't close; burn (₹2.5–4L/month) outruns bootstrap capital.
**Early warnings:** <20% of targeted funds take a first meeting; meetings praise the mission but question category ("agritech fatigue"); runway <6 months without a term sheet.
**Mitigation:** lean burn by design (2 hires, per-trip logistics, no cloud spend before buyers — [17-FUNDRAISE-FINANCE.md](17-FUNDRAISE-FINANCE.md)); raise on retention + unit economics, not GMV vanity (the selective revivals — Arya.ag, AgroStar — rewarded exactly this); revenue is plan A: contribution-positive orders by month 6 makes the raise optional rather than existential; FPO SaaS creates a second, software-margin revenue story.
**Contingency:** bridge via Gujarat angel/family-office networks (smaller cheques, faster); slow expansion, not the core (Surat alone can be profitable at roughly 40–100 active buyers depending on mix per [03-BUSINESS-MODEL.md](03-BUSINESS-MODEL.md) §7); venture debt/revenue-based financing against recurring buyer revenue if metrics support it.
**Kill criterion:** runway <3 months, no term sheet, and not yet contribution-positive ⇒ shrink to founder-only + best 20 buyers and reach breakeven at micro-scale, or execute an orderly wind-down that honors farmer payouts first. This line is checked monthly against the runway sheet — never discovered in the moment.

## R9 — Single-founder execution risk
**The risk:** one person is CEO, CTO, head of sales, and hub supervisor. Illness, burnout, a family emergency, or plain bandwidth saturation stalls everything; the company has a bus factor of 1.
**Early warnings:** founder in every daily loop after week 4; SOP steps that only the founder can run; backlog decisions pending >1 week; sleep/health degradation (yes, this is a tracked risk indicator — it is the leading indicator of every other one).
**Mitigation:** this Brain is itself the primary mitigation — every SOP, price formula, and gate is written to be executable by someone else; Ops Lead (month 2) cross-trained on price setting and payouts by week 4 ([14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md) §5); field-sales hire (month 3) owns buyer relationships day-to-day; WhatsApp-manual fallback means ops survive a founder-absent day; weekly ritual forces delegation review; passwords/access documented in a sealed contingency note with a trusted person.
**Contingency:** pre-agreed "founder down" protocol: Ops Lead runs the hub on SOPs, field sales holds buyers, deliveries continue, all expansion decisions freeze — the pilot can idle safely for 2–3 weeks on this playbook.
**Kill criterion:** none — this risk is managed, not killed. But a standing rule: if by month 4 the founder is still required for >2 of the 5 daily SOP stations, hiring is behind schedule and expansion gates (G2 onward) are frozen until fixed. Consider a co-founder/founding-team search a standing agenda item at the monthly review.

## R10 — Brand/trademark blocked
**The risk:** "KisanSetu" fails trademark search (crowded "Kisan-" namespace in classes 31/35) or conflicts with a government-scheme name, forcing a rename after materials exist.
**Mitigation:** T0.5 search before any spend on printed matter/icons; treat the name as a working title everywhere (this corpus already does); keep two backup names cleared in the same search session. **Rule:** no signage, packaging, or ad spend on the name until the search memo says clear. Impact if fired: 1–2 weeks of rework — annoying, not dangerous, *if* caught in Phase 0.

## R11 — Payment/fraud incident
**The risk:** payout to a wrong/fraudulent VPA, an ops-device compromise, or a buyer default on credit terms.
**Mitigation:** ₹1 test payout verifies every farmer VPA at onboarding (T6.4); payouts only against graded listings (no free-form transfers); ops device with screen lock + separate Razorpay role credentials (Razorpay is the fixed India PSP — founder decision 14 Jul 2026, [21-AI-EXECUTION-PLAYBOOK.md](21-AI-EXECUTION-PLAYBOOK.md) §10); buyer credit capped at net-7 and only after a buyer's first 4 pay-on-delivery/prepaid orders (canonical credit terms, [13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md) §2.1); weekly reconciliation of payouts vs slips vs bank statement.
**Contingency:** single-incident exposure ~₹10k at pilot AOVs on the payout side; worst buyer-default case is the ₹25,000 per-buyer credit cap ([13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md) §2.1); Razorpay dispute process; buyer defaults handled by supply suspension (the strongest B2B collections tool is tomorrow's delivery). Not a kill-level risk at pilot scale; re-scored before scale-up.

## R12 — Supply shock: monsoon, pest, or price crash
**The risk:** a flood week or pest outbreak wipes out one crop's local supply; or a glut crashes reference prices so far that the 60% farmer-share floor and buyer price stability collide.
**Mitigation:** 2 crops chosen partly for complementary seasonality (T0.6 criterion); FPO field signal gives 48h warning ([14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md) §5 reference data #3); the movement damper + honest partial-fill SOP handles day-scale shocks; glut days are farmer-acquisition days (we absorb volume at market-tracking prices when the mandi punishes them).
**Contingency:** temporary sourcing from a neighboring FPO/aggregator for buyer continuity (disclosed in the trace — traceability is never faked, even in a shock week); crop substitution offers to buyers.
**Kill criterion:** none at pilot scale; a full failed season in the district would trigger the R2/R7-style geography review.

---

## How this register is used
1. **Monthly re-score** in the first-Monday extended ritual; changes logged at the bottom of this file.
2. **Kill criteria are pre-commitments.** Overriding one requires a written memo stating why the criterion was wrong *when written* — not why this week is special.
3. Every gate in [13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md) maps to a risk here: G1↔R1, supply gates↔R2/R5, margin gates↔R3, runway check↔R8.

_Change log: v1.0 (July 2026) — initial register, pre-pilot._
