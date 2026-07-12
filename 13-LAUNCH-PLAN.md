# 13 — LAUNCH PLAN

_KisanSetu Brain · July 2026 · Owner: Alpesh (founder). Siblings: [12-DEVELOPMENT-PLAN.md](12-DEVELOPMENT-PLAN.md) (what gets built), [14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md) (how the hub runs daily), [15-TASKS-BACKLOG.md](15-TASKS-BACKLOG.md) (task-level tracking), [16-RISKS-MITIGATIONS.md](16-RISKS-MITIGATIONS.md) (what kills us and when we call it)._

This document is the go-live sequence for the Surat pilot and the expansion playbook beyond it. It is written to be executed by the founder plus two hires. Everything here respects the Golden Rules in [00-GOLDEN-RULES.md](00-GOLDEN-RULES.md) — especially Rule 3 (**plan before code**: nothing in this plan authorizes development; that gate is PRD approval, see [15-TASKS-BACKLOG.md](15-TASKS-BACKLOG.md)) and Rule 5 (**the two metrics**: median hours harvest→door <36h target / <48h promise, and farmer share ≥60% published).

---

## 1. Launch preconditions (all must be TRUE before the dry run starts)

| # | Precondition | Evidence required | Source doc |
|---|---|---|---|
| P1 | Phase 0 validation passed | ≥5 of 10 interviewed Surat HoReCa buyers said they would switch; real AOV data captured; kill criterion (<3 of 10) not triggered | [02-MARKET-RESEARCH.md](02-MARKET-RESEARCH.md), T0.2 |
| P2 | 1 FPO pilot MoU signed | Signed MoU per outline in [18-LEGAL-COMPLIANCE.md](18-LEGAL-COMPLIANCE.md) §6 | T0.3 |
| P3 | 2 beachhead crops frozen | Decision memo naming the 2 crops, chosen from interview demand data | T0.6 |
| P4 | Unit economics replaced with real numbers | ₹2,000 AOV / 10% take rate placeholders replaced by interview + trial-route data in [03-BUSINESS-MODEL.md](03-BUSINESS-MODEL.md) | T0.9 |
| P5 | Legal minimum in place | Pvt Ltd incorporated; FSSAI + GST applied for (registration number or ARN in hand); Gujarat APMC position documented by CA/advocate | [18-LEGAL-COMPLIANCE.md](18-LEGAL-COMPLIANCE.md) |
| P6 | PRDs approved & MVP accepted | Founder sign-off on 06/07/08/09 PRDs; MVP acceptance tests green per [12-DEVELOPMENT-PLAN.md](12-DEVELOPMENT-PLAN.md) | Dev gate |
| P7 | Ops readiness | Hub SOP rehearsed once with FPO staff; 3PL per-trip contract signed after 2 trial routes; 60 crates + QR labels on site; stamped weighing scale at hub | [14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md) |
| P8 | Manual fallback kit printed | Paper intake forms, payout slips, WhatsApp broadcast templates, laminated grade charts — the software-down kit of [14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md) §9 | T6.7 |

**One deliberate exception:** P6 (software) may lag P1–P5. If validation is done and buyers are hungry, we launch on WhatsApp + paper and let software catch up. The reverse is never allowed — software readiness never substitutes for demand validation.

---

## 2. Pilot configuration (frozen numbers)

### 2.1 Canonical pilot parameters — SINGLE SOURCE OF TRUTH

**This table is the single source of truth for KisanSetu's core pilot operating numbers (CEO-set).** Read it before quoting any of these values anywhere:

- **(a)** These are the authoritative canonical values. Where any other document states a different number for a parameter below, **this table wins** and the other document is wrong until reconciled.
- **(b)** The docs that carry operational promises — **03-BUSINESS-MODEL, 04-GTM-SALES-MARKETING, 05, 07, 08, 14-OPS-PLAYBOOK, and 18-LEGAL-COMPLIANCE** — must reference or match this table exactly (no restating a conflicting cutoff time, delivery window, minimum order, fee, credit term, SLA, or farmer-share number).
- **(c)** The app's `GET /config` values in [06-PRD-BACKEND.md](06-PRD-BACKEND.md) are **seeded FROM this table, per region** (region → config), not hand-set in the client. Changing a value here is the only sanctioned way to change what the app serves.

| Parameter | Canonical value |
|---|---|
| Launch geography | **CMS-managed via Market Targeting** ([19-PRD-CMS-ANALYTICS.md](19-PRD-CMS-ANALYTICS.md) C-6): pilot default = **targeted mode with Surat active (seeded)**. Retargeting to another city (e.g. Ahmedabad) is a CMS action, not a code change. Auto/worldwide mode is a **post-pilot founder action with its own go/no-go review** — never flipped casually |
| Order cutoff | 18:00 (6 PM) day before delivery (D-1), local time |
| Delivery window | 06:00–10:00 (target majority of drops before 09:00) |
| Minimum order value | ₹1,000 |
| Delivery / handling fee | ₹50 flat per order, waived for orders ≥ ₹4,000 |
| Buyer credit terms | Pay-on-delivery (or prepaid) for a buyer's first 4 orders; then net-7, per-buyer exposure cap ₹25,000, auto-suspend supply at 10 days overdue |
| OTP lockout | 1 hour per phone after 5 failed attempts |
| Freshness SLA | Promise <48h harvest→door; internal target <36h median |
| Farmer share | Target 65% of platform price, hard floor 60% |
| Farmer price rule | `farmer_price = platform_price × 0.65` (floor 0.60) |
| Pilot cohort | 1 FPO, 25–50 farmers, 15–25 HoReCa buyers, 2 frozen crops |
| Go/no-go gates | ≥70% weekly buyer repeat, ≥95% fulfilment, ≥60% documented farmer share |

### 2.2 Operational configuration

Operational context for the pilot. Any numeric value here that also appears in §2.1 is governed by §2.1.

| Parameter | Value | Rationale |
|---|---|---|
| Geography | Surat city + one FPO catchment in Surat district (single hub). Serviceability itself is CMS-managed per §2.1's launch-geography row — Surat is a seeded default, not a hardcode | Beachhead; founder can physically be at the hub every morning |
| FPO partners | **1** | One relationship done deeply beats three done badly; second FPO only after go/no-go gate |
| Farmers | **25–50** via the FPO | Enough supply for 2 crops without over-promising offtake |
| Buyers | **15–25** HoReCa (mix: ~10 restaurants, 3 hotels, 3 caterers, 2–4 cloud kitchens) | Matches founder-led sales capacity of 5 visits/day ([04-GTM-SALES-MARKETING.md](04-GTM-SALES-MARKETING.md)) |
| Crops | **2** (frozen at T0.6) | Grading charts, price feeds and quality disputes are per-crop work; 2 is the max we can run well |
| Delivery window | 06:00–10:00 daily, target majority of drops before 09:00 (HoReCa prep window); 6 days/week (hub closed one fixed weekday) — per §2.1 | Matches restaurant procurement rhythm from mandi-shadow data |
| Order cutoff | 18:00 (D-1) via app/WhatsApp, late orders best-effort — per §2.1 | Gives FPO evening harvest instruction time |
| Minimum order value | ₹1,000 — per §2.1 | Keeps per-drop economics viable for asset-light 3PL |
| Logistics | 3PL per-trip, insulated vehicle; no owned assets | Golden Rule 4 |
| Pilot duration before gate review | **8 weeks** of live trading (after the dry-run week) | Enough weekly cohorts to measure repeat rate honestly |

---

## 3. Countdown: T-minus schedule

"G" = first live commercial delivery day. Target: G within 4 weeks of P1–P5 turning green.

### T-4 to T-2 weeks
- Onboard 25–50 farmers at an FPO **village demo day**: live demo of listing flow (or paper listing at pilot start), grading chart walkthrough, sample instant-UPI payout of ₹1 to every attendee's phone to prove the rail works. Owner: founder + FPO CEO.
- Founder sales sprint: convert interviewed buyers to committed pilot buyers with a signed one-page buyer agreement ([18-LEGAL-COMPLIANCE.md](18-LEGAL-COMPLIANCE.md) §7) and a standing delivery-day preference. Target: 15 committed before dry run, backfill to 25 during weeks 1–4.
- 3PL trial routes #1 and #2 completed and scored ([14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md) §6); per-trip contract signed.
- Price-setting SOP rehearsed for 5 consecutive evenings against live reference-market data — prices published to a test WhatsApp list nightly.

### T-1 week — THE DRY RUN WEEK (mandatory, never skipped)
Five consecutive operating days, real produce, real money, real trucks — but **friendly buyers only** (5 buyers who know it's a rehearsal; produce sold at cost, no take rate).

| Day | Drill | Pass condition |
|---|---|---|
| D1 | 1 order, full happy path: farmer listing → hub intake → weigh → grade → QR crate → dispatch → delivery → instant farmer payout → buyer invoice | All 4 SLA timestamps captured (`harvest_ts`, `hub_in_ts`, `dispatch_ts`, `delivered_ts`); payout UTR on printed slip |
| D2 | 3 orders in parallel, mixed grades | ≤1 grading dispute; all deliveries in window |
| D3 | **Software-down drill**: run the entire day on the paper + WhatsApp fallback kit, backfill data into the system by 20:00 | Zero deliveries blocked; backfill complete and reconciled |
| D4 | Failure drills: one simulated UPI payout failure (exercise NEFT fallback), one simulated buyer rejection at door (exercise credit-note protocol), one 3PL late-arrival escalation call | Each drill resolved per [14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md) §7–8 scripts |
| D5 | Full 5-order day + first weekly metrics ritual run on dry-run data | Dashboard (or spreadsheet fallback) produces all 8 pilot metrics of §5 |

**Dry-run exit review (D5 evening):** founder + ops lead + FPO CEO. Any failed pass condition ⇒ fix and re-run that drill before G. Two consecutive failed re-runs of the same drill ⇒ push G by one week; do not launch broken.

### T-0: Go-live week (Week 1)
- Day 1–2: 5 buyers live, ~8–10 orders total. Founder personally rides one delivery route each morning.
- Day 3–6: ramp to all committed buyers. Daily 18:30 stand-up (founder + ops lead, 15 min): yesterday's SLA hours, today's exceptions, tomorrow's supply confirmation from FPO.
- Day 7: Week-1 metrics ritual; call every buyer personally for feedback (all ≤25 of them — this is a feature of being small, use it).

---

## 4. The two non-negotiable operating rules from day 1

### 4.1 Freshness SLA instrumented from day 1
Every crate carries the four timestamps on its allocation record (`harvest_ts → hub_in_ts → dispatch_ts → delivered_ts`, per [06-PRD-BACKEND.md](06-PRD-BACKEND.md)). During any manual-fallback period the same four timestamps are written on the paper crate card and backfilled same-day. **A crate without a harvest timestamp does not ship.** The SLA report (median + p90 hours harvest→door, per crop, per route) is reviewed daily in week 1–2, then weekly. Targets: median <36h, p90 <48h. The <48h number is the public promise on the website and buyer agreement — instrument it before you advertise it, never after.

### 4.2 The WhatsApp-manual fallback rule: software never blocks a delivery
Codified triggers — fall back immediately, no debugging in the delivery window:
- App/API down or unusable at the hub for >10 minutes during intake (06:00–10:00) ⇒ paper intake forms + WhatsApp photos to the ops number.
- Buyer can't order in the app ⇒ take the order on WhatsApp Business, ops enters it into the dashboard (or the paper order log) before cutoff.
- Payout API failure ⇒ manual UPI from the ops phone against the printed slip; NEFT same-day if UPI rail itself is down. Farmer is paid **on pickup day, no exceptions** — this promise is the brand.
- Dashboard down for allocation ⇒ allocate on the printed allocation board (whiteboard photo to WhatsApp group is the record).

Every fallback event is logged (time, cause, duration, orders affected) and backfilled into the system within 24h. Fallback frequency is itself a week-ritual metric — if we're falling back >2×/week in week 4+, that's a software defect review, not an ops heroics story.

---

## 5. Pilot metrics & the weekly ritual

Weekly ritual: **Monday 18:00, 60 minutes**, founder + ops lead (+ FPO CEO monthly). Full agenda and dashboard spec in [14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md) §10. The eight pilot metrics:

| # | Metric | Definition | Week-8 gate | Red line |
|---|---|---|---|---|
| M1 | Weekly buyer repeat rate | % of buyers who ordered last week that ordered again this week | **≥70%** | <50% any 2 consecutive weeks |
| M2 | Fulfilment rate | order_items delivered in full & in window ÷ order_items confirmed | **≥95%** | <85% any week |
| M3 | Farmer share | Σ farmer payouts ÷ Σ buyer produce value (excl. delivery fee), documented per payout slip | **≥60%** | <55% any week |
| M4 | Freshness | Median hours `harvest_ts→delivered_ts` | <36h median, <48h p90 | p90 >48h any week |
| M5 | GMV & orders | Weekly GMV (₹), order count, AOV | Trend up W1→W8 | — |
| M6 | Realized take rate | Net platform revenue ÷ GMV | 8–12% band | <6% (we're subsidizing) |
| M7 | Quality dispute rate | Disputed crates ÷ delivered crates | <5% | >10% any week |
| M8 | Supply reliability | kg delivered by farmers ÷ kg committed at listing | ≥90% | <75% (side-selling signal → [16-RISKS-MITIGATIONS.md](16-RISKS-MITIGATIONS.md) R5) |

---

## 6. Go/no-go gates

### Gate G1 — Week-8 scale-up review (the pilot verdict)
**GO** (expand within Surat) requires ALL of:
1. Weekly buyer repeat rate **≥70%** (M1), averaged over weeks 5–8;
2. Fulfilment **≥95%** (M2), weeks 5–8;
3. Farmer share **≥60% documented** (M3) — documented means: every payout slip archived, the % computed from slips not from intentions, and publishable.

Plus no red-line breach open on M4, M7, M8.

**NO-GO** outcomes (pick one, decided in a written memo within 1 week of the review):
- *Fix-and-extend*: one metric narrowly missed with a known cause ⇒ 4-week extension with a named fix. Allowed once.
- *Pivot*: demand-side metrics failed (M1) while ops metrics passed ⇒ pivot to FPO-procurement-SaaS-first wedge (per the standing kill/pivot criterion in [01-VISION-MISSION.md](01-VISION-MISSION.md) and [16-RISKS-MITIGATIONS.md](16-RISKS-MITIGATIONS.md) R1).
- *Kill*: both demand and ops metrics failed and no credible single cause ⇒ stop, preserve capital, write the post-mortem.

### Gate G2 — Surat densification (weeks 9–20, after G1 GO)
Grow to 50–75 buyers, 100+ farmers, 4–6 crops, second hub with same FPO or FPO #2. Entry criteria: ops lead can run the hub without the founder present 4 days/week; contribution margin per order ≥ ₹0 (path to positive per [03-BUSINESS-MODEL.md](03-BUSINESS-MODEL.md)); M1–M3 held during ramp.

---

## 7. Expansion playbook

The product is global-first ([00-GOLDEN-RULES.md](00-GOLDEN-RULES.md) Rule 2); the expansion motion is deliberately sequential. Each stage has entry criteria — we do not enter a stage because the previous one is boring, only because it is *passed*.

### Stage E1 — City #2 (Gujarat: shortlist Vadodara, Rajkot, Ahmedabad)
**Entry criteria (all):**
- Surat: G1 passed + G2 targets held for 8 further weeks; contribution-margin positive per order in Surat.
- A named city where: ≥100 HoReCa targets listed, ≥1 FPO within 60 km of the city with an MoU in negotiation, ≥2 3PL cold-logistics vendors quoting per-trip.
- A repeatable launch kit exists: this document's §1–§5 executed in Surat becomes the template; city launch owner identified (first city-GM hire or founder relocates focus for 6 weeks).
- Same-state advantage banked: same APMC regime, same language pack (GU), same GST registration (additional place of business), same 3PL vendors where possible.

**City #2 pilot = a compressed copy of the Surat pilot:** 1 FPO, 2–3 crops, 15–25 buyers, dry-run week mandatory, same G1 gate at week 8. Target: city #2 reaches G1 in ≤60% of the calendar time Surat took.

### Stage E2 — State-wide Gujarat + first non-Gujarat metro
**Entry criteria:** 2 cities passed G1; playbook executed by someone other than the founder end-to-end in city #2; central price-setting + regional overrides working in the ops dashboard ([09-PRD-OPS-DASHBOARD.md](09-PRD-OPS-DASHBOARD.md)); seed raise closed or 12-month runway ([17-FUNDRAISE-FINANCE.md](17-FUNDRAISE-FINANCE.md)). New-state checklist: that state's APMC rules memo, new mandi reference-data sources, language pack if needed, 3PL coverage map.

### Stage E3 — Country #2 (Southeast Asia / Africa / LATAM shortlist)
**Entry criteria (all):**
- India: ≥3 cities at G1-passed, company-level contribution positive, the two metrics published publicly for ≥6 months.
- Product: zero India-hardcoding audit passed — currency, tax, language, phone, and `reference_market_price` abstractions verified against [06-PRD-BACKEND.md](06-PRD-BACKEND.md); a "new region" can be configured (region → hub → farm, tax profile, currency, language pack, payment provider) without schema change.
- Country selection scored on: smallholder density, HoReCa concentration in 1–2 metros, existence of FPO-equivalent producer organizations (co-ops), UPI-equivalent instant-payment rail (e.g. PromptPay, PIX, M-Pesa), cold-3PL availability, regulatory feasibility per the [18-LEGAL-COMPLIANCE.md](18-LEGAL-COMPLIANCE.md) global checklist, and a local founding partner/hire.
- Entry mode: one city, one producer-org partner, two crops — the Surat playbook verbatim, localized. No country #2 before the playbook has survived contact with three Indian cities.

### What stays central vs what localizes
| Central (never forked) | Localized per region |
|---|---|
| Codebase, schema, design system, SLA definitions, the two metrics | Language packs, currency, tax profile, payment provider instance |
| Grading discipline (photo chart method), payout-slip concept | Crop list, grade charts per crop, reference-market data source |
| Weekly ritual + gate structure | 3PL vendors, producer-org partners, price multipliers |

---

## 8. Launch communications
- **Farmers:** WhatsApp broadcast (opt-in, per [18-LEGAL-COMPLIANCE.md](18-LEGAL-COMPLIANCE.md) §9) in HI/GU: tomorrow's prices + hub timing, every evening after price-set.
- **Buyers:** WhatsApp price list 19:00 daily; week-1 personal call from founder; payout-slip farmer-share % shared monthly as proof-of-mission content.
- **Public:** website goes from lead-gen to live claims only after week 4 of real SLA data; "hours from harvest" counter ships only with real numbers behind it. PR ("asset-light survivor" angle, [04-GTM-SALES-MARKETING.md](04-GTM-SALES-MARKETING.md)) waits for G1-passed.

---

## 9. Decisions taken in this document (flagged for founder review)
1. Dry-run week is mandatory and has named pass conditions — launching without it is prohibited even under buyer pressure.
2. G1 review sits at week 8 of live trading (not month 2 calendar) to guarantee enough weekly cohorts for M1.
3. Fix-and-extend after a failed G1 is allowed exactly once.
4. City #2 must be in Gujarat (regulatory + language + 3PL reuse) — Mumbai/Pune wait for E2.
5. Country #2 requires three Indian cities passed, not one — this is deliberately conservative; revisit at seed raise.

Open question (owner: founder, decide by end of Phase 0): whether hub operates 6 or 7 days/week — depends on FPO staffing and buyer weekend demand from interviews.
