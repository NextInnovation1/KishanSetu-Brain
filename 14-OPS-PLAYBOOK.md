# 14 — OPS PLAYBOOK

_KisanSetu Brain · July 2026 · Owner: Alpesh (founder) until the Ops Lead hire (month 2), then Ops Lead with founder review. Siblings: [13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md) (when this playbook activates), [09-PRD-OPS-DASHBOARD.md](09-PRD-OPS-DASHBOARD.md) (the software that mirrors this playbook), [06-PRD-BACKEND.md](06-PRD-BACKEND.md) (data fields referenced here), [18-LEGAL-COMPLIANCE.md](18-LEGAL-COMPLIANCE.md) (licenses, scale stamping, agreements)._

This is the standard operating procedure for running one KisanSetu hub day. It is written so a new Ops Lead can run the hub after one supervised week. Principles: the paper version of every step exists (software never blocks a delivery — [13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md) §4.2); every step that touches money or quality produces a physical artifact (slip, tag, photo); the two metrics (freshness hours, farmer share) are captured as a by-product of doing the work, never as extra admin.

---

## 1. Roles & the daily rhythm

| Role | Who (pilot) | Owns |
|---|---|---|
| Ops Lead | Hire #1 (founder until month 2) | Hub floor: intake→dispatch, 3PL coordination, payout execution |
| Grader | 1–2 FPO staff (trained by us, per MoU) | Weigh + grade + crate tagging |
| Price setter | Founder (Ops Lead as backup) | Daily price feed |
| Field sales | Hire #2 (founder until month 3) | Buyer orders, escalations, new buyer onboarding |
| 3PL driver | Vendor staff | Pickup, cold transit, doorstep handover, delivery timestamps |

**Daily timeline (frozen for pilot; timestamps are hub local time, stored timezone-aware per [06-PRD-BACKEND.md](06-PRD-BACKEND.md)):**

| Time | Action | SOP |
|---|---|---|
| 18:00 (D-1) | Set tomorrow's prices; publish price feed + WhatsApp broadcasts | §5 |
| 18:00 (D-1) | Order cutoff; supply check → orders **confirmed with provisional quantities** communicated to buyers by 19:30 (no binding allocation yet) | §4.1–4.2 |
| 19:00 (D-1) | Harvest instructions to FPO/farmers (what to harvest, how many kg — capped at demand +15% per §4.6; harvest early morning not previous afternoon) | §4.3 |
| 05:00–07:30 | Farmer produce arrives at hub: intake → weigh → grade → payout → crate/QR → staging | §2–3, §7 |
| 06:30 | Price sanity check vs live reference data; correct only if reference moved >10% | §5.4 |
| 06:30–07:15 | **Binding allocation run**: graded listings → order_items → order_allocations; crate serials linked to allocations at morning staging; buyers told of any change from provisional | §4.4–4.5 |
| 07:30 | 3PL vehicle loads; dispatch scan (`dispatch_ts`) | §2.5 |
| 06:00–10:00 | Deliveries (canonical window, [13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md) §2.1; target majority of drops before 09:00); `delivered_ts` per drop | §8 |
| 10:00 | Exceptions log: disputes, shortfalls, fallback events entered in dashboard | §8–9 |
| 18:30 | 15-min stand-up: founder + Ops Lead (week 1–2 daily, then Mon/Thu) | — |
| Monday 18:00 | Weekly metrics ritual (60 min) | §10 |

---

## 2. Hub SOP: intake → weigh → grade → crate/QR → dispatch

The hub is the FPO's existing collection point. KisanSetu adds: a stamped scale, a grading table with the photo chart, crates + QR labels, a printer (thermal, battery-backed), and one Android device running the ops views.

### 2.1 Station 1 — Intake
1. Farmer arrives with produce against yesterday's listing (or walk-in — accepted during pilot, logged as a same-day listing).
2. Grader pulls the listing on the device (or paper intake form #F1): farmer name, produce, committed kg, `harvest_ts` **asked and recorded** ("कब तोड़ा? / When was this harvested?"). Default assumption if farmer is unsure: 05:00 today for morning arrivals — never leave it blank; a crate without `harvest_ts` does not ship.
3. Visual pre-check: obviously spoiled/damaged produce set aside immediately (see 2.3 rejection rules) — reject at intake, not at grading, so the farmer sees why.

### 2.2 Station 2 — Weigh
1. Weigh on the **Legal-Metrology-stamped platform scale** (verification sticker current — [18-LEGAL-COMPLIANCE.md](18-LEGAL-COMPLIANCE.md) §5). Tare the crate (all crates same model, tare weight printed on the chart).
2. Record `graded_qty` per grade after grading (2.3); the intake gross weight goes on the intake form.
3. Weight discrepancy rule: platform pays on hub-scale weight, full stop. This is stated at farmer onboarding and printed on the payout slip. Farmer may watch the weighing — the scale display faces the farmer.

### 2.3 Station 3 — Grade (A/B with photo chart)
1. Grading is done against the **laminated photo grade chart** — one chart per crop, A2 size, posted at the table, same images in the app. Chart per crop contains: 4 photos of Grade A, 4 of Grade B, 4 of REJECT, plus the 3–5 objective criteria per grade (size band in cm/gm, color stage, firmness, defect tolerance %). Charts for the 2 beachhead crops are produced at T6.1 ([15-TASKS-BACKLOG.md](15-TASKS-BACKLOG.md)) after crop freeze at T0.6; the chart *format* is fixed here.
   - Illustrative (tomato, to be replaced by the frozen crops' charts): **A** = 80–120 g, uniform red-ripe (stage 5–6), firm, zero cuts/cracks, ≤5% blemish surface; **B** = 60–150 g, stage 4–6, minor blemish ≤15%, slight softness OK; **REJECT** = cracked, rotting, pest-holed, or overripe-soft.
2. Grader sorts into A / B / reject piles, weighs each, records `graded_qty` per grade on the listing (status `posted → graded`).
3. Rejects: returned to the farmer on the spot (they keep it, sell it locally, or FPO composts). Reject % recorded. Reject rate >20% for a farmer twice in a week ⇒ Ops Lead conversation, retraining against the chart, before it becomes a dispute pattern (see [16-RISKS-MITIGATIONS.md](16-RISKS-MITIGATIONS.md) R6).
4. **Grading photo (dispute evidence, required from day 1):** one photo per farmer-per-crop-per-day of the graded piles next to the laminated chart, taken on the hub Android device at Station 3 and **uploaded on the listing via the grade endpoint** ([06-PRD-BACKEND.md](06-PRD-BACKEND.md) — the grade call carries a minimal local-disk photo path in the MVP, not a full media pipeline). This photo is the evidence base for any downstream buyer dispute (§8.3) — 10 seconds now saves an hour of he-said-she-said later.
   - **Manual fallback:** if the upload fails (no signal / app down), the photo is still taken and filed to a dated folder on the hub device named by listing ID (`YYYY-MM-DD/<listing_id>/`), and linked to the listing when connectivity returns. Grading never waits on the upload.

### 2.4 Station 4 — Crate + QR tag
1. Graded produce goes into KisanSetu crates: one grade per crate, never mixed. Fill to the crop's max-fill line (printed on chart; overfilled crates crush the bottom layer — a freshness promise killed by physics).
2. Each crate gets a **QR label** (thermal sticker into the label holder): QR encodes the **`listing_id` + the crate's serial number** (the serial embossed on the crate, §3) — **not** an allocation ID. At tagging time the binding allocation has not necessarily run yet (it is a morning step, §4.4), so the crate is identified by which graded listing it came from; the crate is bound to its `order_allocation` later, at morning staging (§4.5). Printed human-readable line: crop, grade (A in forest green band, B in amber band per [10-DESIGN-SYSTEM.md](10-DESIGN-SYSTEM.md)), kg, harvest date, farmer first name + village, hub code.
3. Scanning the QR at tagging writes `hub_in_ts`. On paper-fallback days: handwritten crate card #F2 with the same fields (listing ID + crate serial), timestamps included.

### 2.5 Station 5 — Dispatch
1. Crates staged by delivery route (route sheet from the allocation run, §4.5). Load in reverse-drop order.
2. Driver + Ops Lead count crates against the route sheet, both sign. QR batch-scan at loading writes `dispatch_ts`.
3. Cold discipline at pilot scale: pre-dawn operation + insulated vehicle/insulated boxes with ice packs for leafy crops (see §6.3). Ops Lead records vehicle departure temp check (fridge thermometer in one crate, photo) — crude but honest instrumentation until volumes justify reefer.

### 2.6 Self-signup farmer linking (before a farmer can list or reach Station 1)
A farmer signed up in person on a demo day is linked to this hub then and there (`hub_id` set by ops). But a farmer who **self-signs-up in the app without a `hub_id`** — word-of-mouth, a referral, anyone outside a demo day — must be vetted before they can list:
1. Unlinked signups land in the **"Partners / unlinked farmers" worklist** in the ops dashboard ([09-PRD-OPS-DASHBOARD.md](09-PRD-OPS-DASHBOARD.md)).
2. Ops **calls the farmer**, verifies they are a member of a partner FPO (or otherwise eligible per the partner MoU), captures village + registered VPA, and answers questions.
3. Ops **sets the farmer's `hub_id`.** Only then does the farmer become a listable partner; until then the app shows "pending verification" and blocks listing.
4. Farmers who cannot be verified (not an FPO member, outside serviceable villages) are marked declined with a reason — no silent limbo.

This keeps every kg traceable to a known, hub-linked farmer and stops off-hub produce entering the traceability spine.

**Hygiene & safety (posted at hub):** no smoking/paan at the grading table; hands washed before grading shift; crates washed weekly (rota on the wall); floor swept between crop lots; first-aid box; scale area dry.

---

## 3. Crate & QR sourcing

| Item | Spec | Qty (pilot) | Est. cost | Notes |
|---|---|---|---|---|
| Plastic ventilated crates | 600×400×285 mm, ~20–25 kg capacity, stackable, food-grade HDPE, KisanSetu green if MOQ allows | 60 (≈3 rotation sets) | ₹300–380/crate → ~₹20k | Local Surat plastics wholesaler; emboss/stencil "KISANSETU + serial number" |
| Label holders | Clip-on label window per crate | 60 | ~₹15 ea | Survives washing |
| Thermal label printer | Bluetooth, 2-inch, battery | 1 + 1 backup | ~₹8k | Prints QR labels at Station 4 |
| Thermal labels | 50×50 mm rolls | 5k labels | ~₹2k | Water-resistant stock |
| Crate rotation | Buyer keeps crate at delivery; driver collects previous day's empties at each drop | — | — | Crate deposit clause in buyer agreement: ₹350/crate charged if not returned in 3 deliveries |
| Loss rule | >10% crates unreturned in a month ⇒ deposit enforcement + serial audit | — | — | Crates walking away is a classic silent margin leak |

---

## 4. Order-to-allocation (two-phase: evening provisional → morning binding)

The allocation is deliberately split. In the **evening** we can only match orders against *ungraded* open listings, so evening quantities are a **provisional supply match**, not a commitment of specific crates. The **binding** allocation runs the next **morning**, once produce is actually weighed and graded — only then do real `order_allocations` rows exist. This split is what lets us pay farmers on graded weight (§7) and still promise buyers exact quantities without owning inventory (Golden Rule 4).

### 4.1 Evening order cutoff (18:00 D-1)
Buyer orders in from app/web/WhatsApp. WhatsApp orders entered into the dashboard by field sales before 18:30 (order source recorded as `whatsapp`).

### 4.2 Evening supply check & provisional confirm (→19:30 D-1)
1. **Supply match, not allocation:** open listings (kg by crop) vs ordered kg. Because listings are still ungraded, this is a *provisional* match — **no `order_allocations` are created yet**.
2. Shortfall ⇒ provisional priority: (1) buyers by longest standing weekly volume, (2) partial-fill everyone above 80% rather than zero-fill anyone.
3. Orders move `placed → confirmed` **with provisional quantities**; buyers get confirmation with delivery window, and affected buyers get a WhatsApp by 19:30 with the **provisional** quantities — surprise at the door is forbidden, and these numbers are stated as "subject to morning grading."

### 4.3 Harvest instructions (19:00 D-1)
Per-crop kg to harvest, sent to the FPO WhatsApp group: harvest **tomorrow morning** (harvest-at-dawn is what makes <36h median achievable — produce harvested the previous afternoon starts the clock 12h in the hole). Quantities are **capped at confirmed demand + a 15% buffer** per crop (over-supply protocol, §4.6).

### 4.4 Morning binding allocation run (06:30–07:15 D0)
1. Once produce is graded (§2.3), the **binding allocation run** (dashboard, [09-PRD-OPS-DASHBOARD.md](09-PRD-OPS-DASHBOARD.md)) matches **graded** listings → `order_items` by crop + grade + kg and creates `order_allocations` (the traceability spine). Priority mirrors §4.2 (standing weekly volume, then partial-fill >80% over zero-fill anyone).
2. Manual override always available.
3. Any buyer whose confirmed provisional quantity changes after grading is messaged **before dispatch** — no silent short-ships.

### 4.5 Morning staging: crate → allocation link + route sheet
Each graded crate (labelled at Station 4 with `listing_id` + crate serial, §2.4) is linked to its `order_allocation` at staging — this is the moment the physical crate joins the traceability spine. The dashboard then emits the **route sheet** (crates by delivery route, reverse-drop order) used at dispatch (§2.5).

### 4.6 Over-supply protocol (inventory-risk guardrail — Golden Rule 4)
Farmers are paid at grading for **all** graded kg (§7), allocated or not, and walk-ins are accepted (§2.1) — so an over-supply morning would leave KisanSetu holding paid-for perishables with no buyer, exactly the inventory risk Golden Rule 4 forbids. Guardrails, in order:
1. **Cap the harvest ask.** Harvest instructions (§4.3) never exceed **confirmed demand + a 15% buffer** per crop. The buffer covers reject/shrinkage and small buyer top-ups; it is a firm cap, not a suggestion. The FPO is told plainly that harvesting beyond the cap is at the farmer's own risk and is **not** an implied purchase commitment.
2. **Grade the allocated kg first.** Intake and grading (§2.2–2.3) prioritise listings tied to confirmed demand; buffer and walk-in produce is graded after, so a long morning still covers committed orders first.
3. **Clear surplus same-day at cost.** Graded kg above allocation is pushed the same morning through the discounted **surplus channel** — standing caterer / quick-commerce buyers and FPO buy-back — at (or near) the **farmer price**, i.e. at cost, to move it before the freshness clock runs down. This is the §8.4 B-grade rescue channel widened to over-supply.
4. **Book the residual.** Any graded kg still unsold at end of day is written off to the **`surplus_loss`** ledger line (distinct from `quality_loss`, §8.4). Weekly `surplus_loss` **> 2% of the week's GMV** triggers a supply-planning review in the Monday ritual (§10) — the fix is a tighter demand forecast and buffer, not owning more inventory.

In practice this keeps us **asset-light**: the cap + grade-priority + same-day clearance mean the platform almost never carries produce overnight, and when it does the loss is small, **named**, and actively reviewed rather than buried in shrinkage.

---

## 5. Daily price-setting SOP

**Who:** Founder sets, Ops Lead is trained backup (must be able to run it solo by week 4 — bus-factor rule, [16-RISKS-MITIGATIONS.md](16-RISKS-MITIGATIONS.md) R9).
**When:** 18:00 daily for the next day. One price per crop per grade per day; intraday changes only under §5.4.

**Reference data (in order of precedence):**
1. Surat APMC mandi modal prices for the day — from AGMARKNET/state market-board feed, cross-checked weekly against the physical mandi (the mandi-shadow relationship from T0.4 is maintained: one hub person walks the mandi Tue + Fri mornings and phones in real spreads).
2. Yesterday's realized buyer acceptance (did anyone push back on price? fill rate at price).
3. FPO field signal on harvest volumes coming (glut/scarcity 48h ahead).

**Setting the price** — **five** columns are entered into `price_feed` per produce per day ([06-PRD-BACKEND.md](06-PRD-BACKEND.md)): `reference_market_price`, `platform_price_a`, `platform_price_b`, `farmer_price_a`, `farmer_price_b`. Ops enters **all five** (09's Prices screen surfaces every column) — the `platform_price_*` values are what buyers pay, the `farmer_price_*` values are what farmers are paid at grading (§7). Rules:
- `reference_market_price` = today's mandi modal price for the standard grade.
- `platform_price_a` = reference × **1.10–1.20** (graded, traceable, delivered A-produce carries a premium — validated in Phase 0 interviews).
- `platform_price_b` = reference × **0.90–1.00**.
- `farmer_price_a` = `platform_price_a` × **0.65** (the 65% farmer-share target).
- `farmer_price_b` = `platform_price_b` × **0.65**.
- **Rounding:** round each farmer price to the nearest **₹0.25/kg** (clean numbers on the payout slip and price board). Never round *down* through the floor — see the next line.
- **Farmer-share check (hard constraint):** each `farmer_price_x` ÷ `platform_price_x` **targets 65%** and must **never fall below the 60% floor** after rounding. If rounding or the multiplier math would break 60%, lift the farmer price — fix the price, not the promise.
- **Take-rate check:** with farmers on ~65% of platform price the gross spread is ~35%; after 3PL, crates and shrinkage the implied **blended net take must land in the 8–12% band** ([03-BUSINESS-MODEL.md](03-BUSINESS-MODEL.md)). Below 6% = we are buying revenue; above 12% = we are squeezing one side — flag either in the weekly ritual.
- Movement damper: platform price moves ≤10% day-over-day unless reference moved more — HoReCa buyers value stability over squeezing the last rupee (interview finding to re-verify in Phase 0).

**Worked example (tomato):**

| `price_feed` column | Rule | Value |
|---|---|---|
| `reference_market_price` | mandi modal | ₹20.00/kg |
| `platform_price_a` | ref × 1.15 | ₹23.00/kg |
| `platform_price_b` | ref × 1.00 | ₹20.00/kg |
| `farmer_price_a` | 23.00 × 0.65 = 14.95 → nearest ₹0.25 | **₹14.95 → ₹15.00/kg** |
| `farmer_price_b` | 20.00 × 0.65 = 13.00 (already on grid) | **₹13.00/kg** |

Farmer share here is 15.00 ÷ 23.00 = **65%** on A and 13.00 ÷ 20.00 = **65%** on B — both on target, clear of the 60% floor.

**Publish:** price feed saved in dashboard → app price cards update → 19:00 WhatsApp broadcasts (farmer list: tomorrow's KisanSetu price vs mandi price, in HI/GU; buyer list: tomorrow's catalog prices A/B).

**5.4 Intraday correction:** only if the reference market moves >10% by the 06:30 check (e.g. supply shock). Buyer prices already confirmed for today's orders are **never** repriced after confirmation; the correction applies to the next cycle. Farmer payout uses the price at listing confirmation.

---

## 6. 3PL cold logistics

### 6.1 Selection (T6.2)
Get 3 written quotes. Score each vendor 1–5 on: per-trip cost for the standard route (35%), on-time reliability references (25%), vehicle hygiene/insulation quality (20%), driver communication + willingness to scan QRs and collect crates (10%), morning-window availability 6 days/week (10%). Minimum bar: food-carrying experience, insulated or reefer vehicle option, GST-registered, per-trip billing.

### 6.2 Two trial routes (before any contract)
- Trial #1: hub → 3-drop city route, normal load. Measure: pickup punctuality, transit time per drop, produce temperature at last drop, driver handling.
- Trial #2: stress case — 6 drops, one simulated address problem, one crate-return collection. Measure recovery behavior and communication.
- Pass = both trials within delivery window, no produce damage, driver follows the scan-and-sign protocol. Then sign **per-trip contract**: rate card per route band (km + drops), no monthly retainer, no minimum commitment (asset-light — Golden Rule 4), 24h cancellation, insurance responsibility for in-transit loss stated, payment weekly against trip log.
- Keep vendor #2 warm with one paid trip/month — single-vendor dependency is a fragility we pay a small premium to avoid.

### 6.3 Cold discipline at pilot scale
Pre-dawn ops + insulated vehicle + ice-pack boxes for leafy/delicate crops is the pilot standard; dedicated reefer per-trip only when a route's load justifies it (revisit at >300 kg/day/route). We sell **hours-from-harvest**, verified by timestamps — not a cold-chain fiction we can't yet afford. If a crop's quality demonstrably fails without reefer (M7 disputes cluster on it), that crop waits (see [16-RISKS-MITIGATIONS.md](16-RISKS-MITIGATIONS.md) R3).

### 6.4 Escalations
Driver late >20 min at hub ⇒ Ops Lead calls vendor dispatcher; >45 min ⇒ activate backup (vendor #2 / founder's arranged standby tempo); buyer informed proactively with new ETA. Every escalation logged.

---

## 7. Farmer payout SOP (the moment that builds or breaks trust)
1. **Trigger:** grading complete at intake (§2.3) — payout is initiated **before the truck leaves**, same visit. Farmer payout = Σ(graded kg × the day's farmer price for that grade). The payout rates are the `farmer_price_a` / `farmer_price_b` set in the day's `price_feed` (§5) — no per-hub recomputation; shown on the app home screen and the hub price board.
2. **Rail:** instant UPI to the farmer's registered VPA via Razorpay payout API ([06-PRD-BACKEND.md](06-PRD-BACKEND.md) `payments`, type `farmer_payout`); UTR recorded.
3. **THE PRINTED SLIP** (thermal, 2 copies — farmer + hub file; this slip is also the marketing asset per [04-GTM-SALES-MARKETING.md](04-GTM-SALES-MARKETING.md)). Exact fields:
   - KisanSetu header + hub code + date/time
   - Farmer name, village, listing ID
   - Per crop: kg Grade A @ rate, kg Grade B @ rate, rejected kg
   - **Reference market price today (mandi): ₹X/kg — struck through**
   - **KisanSetu price paid to you: ₹Y/kg** (leaf-green if printer supports; else bold)
   - "आपका हिस्सा / Your share of buyer price: **Z%**"
   - Total ₹, UPI UTR, ops signature line
4. **Failure handling:** UPI failure → one retry after re-verifying VPA → if rail down, NEFT same day from ops device; farmer told exactly when money lands before leaving the hub. **Never** "we'll sort it tomorrow." Cash is not used (no cash float at the hub — audit + safety).
5. Payout slips filed daily; the weekly farmer-share metric (M3) is computed from these slips — this is what makes the farmer-share promise (**65% target, 60% floor**) a documented number rather than a claim.

---

## 8. Buyer delivery & dispute SOP
1. **At the door:** driver hands over crates; buyer (or receiver) counts crates vs delivery note, spot-checks quality against the printed grade reference card (mini version of the hub chart, left with every buyer at onboarding). QR scan or driver app tap writes `delivered_ts`; receiver name recorded.
2. **Acceptance window:** quality claims accepted at the door or by WhatsApp photo within **2 hours** of delivery. Later claims logged but not credited (perishables — stated in the buyer agreement, [18-LEGAL-COMPLIANCE.md](18-LEGAL-COMPLIANCE.md) §7).
3. **Dispute protocol:** buyer sends photo → Ops compares against the Station-3 grading photo of that allocation (this is why §2.3.4 exists) → resolution same day: credit note on next invoice for accepted claims, replacement next delivery if wanted. Disputed crate rate is metric M7. Systematic pattern by crop/farmer/route feeds the Friday quality review.
4. **Rejection at door** (rare, drill-tested in dry run): driver returns crates to hub; produce re-graded; B-grade rescue channel (discounted same-day sale to caterers) or FPO compost. Loss booked to the `quality_loss` ledger line — we eat unclear cases in the pilot; farmer clawbacks only for proven at-intake misrepresentation, decided by Ops Lead + founder, never by the grader alone.
5. **Invoice:** one consolidated invoice per buyer per delivery (produce as exempt-goods bill of supply + taxable delivery/handling line, per [18-LEGAL-COMPLIANCE.md](18-LEGAL-COMPLIANCE.md) §4); weekly statement for credit-terms buyers (net-7 max during pilot; new buyers prepay/pay-on-delivery first 2 weeks).

---

## 9. Manual fallback kit (printed, in the hub cupboard, restocked Fridays)
- #F1 Intake/weigh/grade form (carbon duplicate) — mirrors listing fields incl. all timestamps
- #F2 Crate card (tie-on) — mirrors QR label fields
- #F3 Route/delivery sheet with per-drop signature + time column
- #F4 Payout slip pad (pre-printed layout of §7.3) + ops phone for manual UPI
- #F5 Order log (WhatsApp order transcription form)
- Laminated grade charts (the wall copies are the fallback — no dependency on the app for grading, ever)
- Fallback triggers and the backfill-within-24h rule per [13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md) §4.2; every use logged in the fallback register (#F6).

---

## 10. Weekly metrics ritual & dashboard
**Monday 18:00, 60 min, founder + Ops Lead; FPO CEO joins the first Monday of the month.** Fixed agenda:
1. (10 min) The two metrics first: freshness M4 (median + p90 by crop and route) and farmer share M3 (from payout slips). Read aloud, versus target, no commentary yet.
2. (15 min) The other six: M1 repeat, M2 fulfilment, M5 GMV/AOV, M6 take rate, M7 disputes, M8 supply reliability ([13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md) §5 definitions).
3. (15 min) Exceptions review: fallback register, escalations, disputes — each gets cause + one action + owner.
4. (10 min) Next week: price posture, supply plan from FPO, buyer pipeline (new/at-risk).
5. (10 min) One improvement to the playbook itself — this document is versioned and edited after every ritual (change log at bottom).

**Dashboard:** the SLA/metrics report views specified in [09-PRD-OPS-DASHBOARD.md](09-PRD-OPS-DASHBOARD.md) render M1–M8 for any week; until that ships, the ritual runs off the standing spreadsheet (`ops/metrics-weekly.xlsx`, template created at T6.8) fed from payout slips and route sheets. The ritual never waits for software.

**Monthly extension (first Monday, +30 min):** risk register review per [16-RISKS-MITIGATIONS.md](16-RISKS-MITIGATIONS.md); crate serial audit; scale verification sticker check; FPO relationship review.

---

## Decisions taken in this playbook (flagged for founder review)
1. Payout on grading at intake (before dispatch), not on delivery — trust-building outweighs the shrinkage risk at pilot scale; revisit if buyer-rejection losses exceed 2% of GMV.
2. Hub-scale weight is final for payment; farmer-facing scale display is the fairness mechanism.
3. 2-hour buyer claim window; platform absorbs unclear dispute losses during pilot.
4. Insulated-vehicle + ice-pack standard (not reefer) at pilot volumes, with a stated trigger to upgrade.
5. Crate deposit ₹350 enforced only after 3 deliveries' grace.
6. Two-phase allocation: the evening confirms **provisional** quantities against ungraded listings; the **binding** allocation runs next morning against graded listings (§4). Farmer share set at **65% target / 60% floor** (§5).
7. Over-supply guardrail: harvest capped at demand **+15%**, surplus cleared same-day at cost, residual booked to **`surplus_loss`** and reviewed weekly (§4.6) — we never carry inventory overnight (Golden Rule 4).

Open question (owner: Ops Lead hire + founder, by end of pilot week 2): whether the FPO's own staff or a KisanSetu-paid grader does Station 3 long-term — MoU allows either; the grading-consistency data from weeks 1–2 decides.

_Change log: v1.0 (July 2026) — initial playbook, pre-pilot. v1.1 (July 2026) — two-phase allocation aligned across §1/§4 (evening provisional match → morning binding allocation against graded listings); §2.4 crate QR encodes `listing_id` + crate serial, allocation linked at morning staging (§4.5); §2.3.4 grading-photo SOP confirmed (grade endpoint + dated-folder fallback); new §2.6 self-signup farmer linking; new §4.6 over-supply protocol (`surplus_loss`); §5 farmer-price rules + worked example; farmer share set to 65% target / 60% floor throughout._
