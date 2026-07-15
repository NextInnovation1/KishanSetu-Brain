# 05 — Product Overview

_The product suite of KisanSetu: what we build, for whom, and how the golden rules shape every surface._
_Siblings: [00-GOLDEN-RULES.md](00-GOLDEN-RULES.md) · [03-BUSINESS-MODEL.md](03-BUSINESS-MODEL.md) · [06-PRD-BACKEND.md](06-PRD-BACKEND.md) · [07-PRD-MOBILE-APPS.md](07-PRD-MOBILE-APPS.md) · [08-PRD-WEBSITE.md](08-PRD-WEBSITE.md) · [09-PRD-OPS-DASHBOARD.md](09-PRD-OPS-DASHBOARD.md) · [10-DESIGN-SYSTEM.md](10-DESIGN-SYSTEM.md)_

---

## 1. What KisanSetu is

KisanSetu ("setu" = bridge; working title, trademark check pending — T0.5 in [15-TASKS-BACKLOG.md](15-TASKS-BACKLOG.md)) is an **asset-light B2B fresh-produce supply chain**. Farmers list produce; partner FPO collection hubs aggregate, weigh and grade it into quality tiers (A/B); third-party cold logistics deliver to food businesses within 24–48 hours of harvest; farmers are paid instantly to UPI on grading at intake (before the truck leaves); buyers get one invoice and farm-to-fork traceability.

The product exists to move exactly two numbers ([00-GOLDEN-RULES.md](00-GOLDEN-RULES.md), rule 5):

| Metric | Target | Promise |
|---|---|---|
| Median hours harvest → buyer's door | **< 36h** | < 48h, published |
| Farmer's share of the final price | **≥ 60%** | Published publicly |

Every screen, endpoint and feature described in this corpus must serve at least one of these two. If it serves neither, it does not get built.

**Global-first, Surat-first.** The platform is designed for any geography where smallholder farmers and food businesses coexist (tier-2 India next, then Southeast Asia, Africa, LATAM). Surat, Gujarat is only the launch beachhead. Concretely: the data model speaks `region → hub → farm`, prices carry a `currency` field, phones are E.164, timestamps are timezone-aware, and the schema says `reference_market_price` — the word "mandi" appears only in Indian UI copy.

---

## 2. The product suite

Five surfaces, one backend, one design system.

| # | Surface | Tech | Audience | Status |
|---|---|---|---|---|
| 1 | **Android app** (dual-role) | Kotlin + Jetpack Compose + Material3 | Farmers + buyers | Spec: [07-PRD-MOBILE-APPS.md](07-PRD-MOBILE-APPS.md) |
| 2 | **iOS app** (dual-role) | Swift 5 + SwiftUI (XcodeGen) | Farmers + buyers | Same single PRD — parity is a contract |
| 3 | **Marketing website** | Vanilla JS, no build step | Prospective buyers, farmers, press, investors | Spec: [08-PRD-WEBSITE.md](08-PRD-WEBSITE.md) |
| 4 | **Ops dashboard** (internal web) | Web app on the same REST API | Ops manager, hub operators | Spec: [09-PRD-OPS-DASHBOARD.md](09-PRD-OPS-DASHBOARD.md) |
| 5 | **FPO procurement SaaS** (future) | Web, licensed per FPO | FPO managers & hub staff | Post-pilot; the data/tech moat (see [03-BUSINESS-MODEL.md](03-BUSINESS-MODEL.md)) |

Notes on the suite:

- **The two mobile apps are one product.** A single PRD ([07-PRD-MOBILE-APPS.md](07-PRD-MOBILE-APPS.md)) governs both; they are screen-for-screen, feature-for-feature, design-identical. There is no "Android-first feature" or "iOS-only polish". One Play Store listing, one App Store listing, same app on both — role (farmer or buyer) is chosen at signup, so we ship **one app per platform**, not four apps.
- **The ops dashboard is the real MVP.** The business runs on it before anything else: daily price setting, grading queue, listing→order allocation, delivery status, payout trigger, SLA reports. In pilot week 1, WhatsApp + phone calls are an acceptable fallback — software must never block a delivery ([14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md)).
- **The website sells the story**, not the produce: broken-chain visual (farmer keeps 30–40% today → 60–70% with us), the 5-step how-it-works, impact stats, EN/HI toggle, and a lead form posting to `POST /leads`.
- **The FPO SaaS is the endgame wedge.** Grading + procurement software licensed to FPOs turns our operating tooling into a product, and is the fallback pivot if buyer validation fails (Phase 0 kill criterion in [13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md)).

Everything is served by one backend: Node.js 20 + Express in TypeScript with Drizzle ORM ([20-CODE-ARCHITECTURE.md](20-CODE-ARCHITECTURE.md) §1), PostgreSQL 16, REST JSON snake_case, JWT + phone-OTP auth ([06-PRD-BACKEND.md](06-PRD-BACKEND.md), [11-ARCHITECTURE.md](11-ARCHITECTURE.md)).

---

## 3. Personas

Four personas. Names are illustrative; the profiles are drawn from the Phase 0 field work targets ([13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md)) and must be re-validated against real interviews.

### 3.1 Ramesh — smallholder farmer (supply side)

| | |
|---|---|
| Profile | 42, farms ~1.2 ha near Olpad, Surat district; grows tomato, okra, leafy greens. Member of the partner FPO. |
| Device | ₹8–12k Android phone, patchy 4G, WhatsApp daily, UPI via PhonePe. Reads Gujarati comfortably, Hindi passably, English barely. |
| Digital literacy | Low. Uncomfortable with forms, dropdowns, English error dialogs. Will hand the phone to his son if a screen confuses him. |
| Today | Sells at the mandi or to a village aggregator: 2–4 middlemen hops, price known only after the sale, payment in days, 30–40% of the final price reaches him. |
| Jobs-to-be-done | (1) "Know tonight what tomorrow's price is, before I harvest." (2) "Sell without a day lost at the mandi." (3) "Get paid the moment I hand over the crop, with proof." |
| Fears | Being cheated on the weighing scale; opaque grading; another app that wastes his time. |
| Success looks like | A payout notification with a UTR number the same morning he delivered, for visibly more money than the mandi rate. |

Product consequences: Hindi-first UI with English subtitles (Gujarati in v1.1 — human-reviewed before pilot week 1, Q3 in [12-DEVELOPMENT-PLAN.md](12-DEVELOPMENT-PLAN.md)), max 2 actions per screen, ≥48dp touch targets, picture grids instead of dropdowns, every money figure shown as *reference market price struck through vs KisanSetu price in green* — the trust arithmetic is the interface.

### 3.2 Kiranben — FPO hub operator (aggregation)

| | |
|---|---|
| Profile | 35, FPO field staff running the collection hub near Olpad; manages intake mornings 05:00–07:30. |
| Device | Shared Android tablet or her own phone; comfortable with WhatsApp, basic apps. |
| Today | Paper registers: who brought what, rough weights, no grading standard, disputes settled by argument. |
| Jobs-to-be-done | (1) "Process 40 farmers' intake in 2.5 hours without a queue riot." (2) "Grade consistently so neither farmer nor buyer disputes it." (3) "Know exactly which crates go on which truck." |
| Fears | Being blamed by farmers for a low grade; software that is slower than her register. |
| Success looks like | Intake → weigh → grade → crate-tag (QR) → dispatch, each in under 90 seconds per listing, with the grading chart poster on the wall as the referee. |

Product consequences: the grading queue in the ops dashboard is optimized for speed and one-handed tablet use; grading criteria are photographic (A/B chart poster, [14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md)); at MVP she uses the **ops dashboard**, not the mobile app — the future FPO SaaS is her product.

### 3.3 Arjun — restaurant owner / chef (demand side)

| | |
|---|---|
| Profile | 38, owns a 60-cover multi-cuisine restaurant in Adajan, Surat; ~₹8–15k/day vegetable spend across 2 suppliers + own mandi runs. |
| Device | iPhone or mid-range Android; WhatsApp for all supplier chatter; expects professional English UI with local produce names alongside. |
| Today | Quality roulette: what arrives is not what was promised; no invoices half the time; price changes daily without notice; his chef rejects 10–15% of deliveries. |
| Jobs-to-be-done | (1) "Order tonight, receive tomorrow morning, consistent grade." (2) "One invoice my accountant can file." (3) "Know where this crate came from when a customer or auditor asks." |
| Fears | Switching suppliers and getting burned again; hidden charges; delivery slots that miss prep time. |
| Success looks like | Grade A tomatoes at a stable published price, delivered in the 06:00–10:00 window, with a trace-line showing the farm, village and harvest time. |

Product consequences: the buyer flow is restrained and professional — grade chips everywhere, prices per kg with the market comparison struck through, order status as a timeline, and traceability as the premium moment (vertical trace line with timestamps). Payment is on invoice (B2B terms), **not** in-app at MVP.

### 3.4 Meera — ops manager (internal)

| | |
|---|---|
| Profile | KisanSetu hire #1; runs daily operations across price setting, grading oversight, allocation, dispatch and payouts. Early on, this is the founder. |
| Device | Laptop + phone; lives in the ops dashboard and WhatsApp Business. |
| Jobs-to-be-done | (1) "Set tomorrow's A/B prices from the reference market price by 6 PM." (2) "Match graded supply to placed orders with zero unfulfilled surprises." (3) "Fire farmer payouts on grading at intake and keep the SLA clock honest." |
| Fears | A delivery promised that supply can't cover; a payout double-fired; SLA breaches discovered by the buyer before her. |
| Success looks like | ≥95% fulfilment, median harvest→door < 36h on the weekly report, zero payout disputes. |

Product consequences: the ops dashboard gets allocation suggestions, SLA timers per allocation (`harvest_ts → hub_in_ts → dispatch_ts → delivered_ts`), and idempotent payout triggers ([09-PRD-OPS-DASHBOARD.md](09-PRD-OPS-DASHBOARD.md), [06-PRD-BACKEND.md](06-PRD-BACKEND.md)).

---

## 4. End-to-end journeys

### 4.1 Farmer journey (Ramesh) — "evening list, morning money"

1. **Onboarding (once):** FPO demo day at the hub. Field staff help him install the app, log in with phone OTP, pick "किसान / Farmer". His profile is linked to the FPO and village by ops.
2. **Evening (D-1):** Opens the app → Home shows today's price cards: reference market price struck through, KisanSetu price in leaf green. Taps **फ़सल बेचें / Sell produce** → picks tomato from a picture grid → sets 80 kg on a stepper → harvest date defaults to tomorrow → sees estimated payout → confirms. Two taps of decision, no typing.
3. **Morning (D0):** Brings crates to the hub. Kiranben weighs and grades: 62 kg Grade A, 15 kg Grade B (3 kg rejected, explained against the wall chart). His listing flips to **ग्रेड A·B / Graded** in My Listings.
4. **Mid-morning (D0):** 3PL truck picks up. Allocation to buyer orders happened overnight/against the morning intake. Payout already fired at grading (before dispatch — [14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md) §7): UPI credit + notification **पैसा भेज दिया गया ✓** with amount and UTR. Printed payout slip shows mandi rate vs what he got — the slip is the advertisement.
5. **Repeat:** He checks tomorrow's price each evening. The Payments tab is his ledger; every entry has an amount, date and UTR he can show the family.

Time in app per day: **under 2 minutes.** That is a design goal, not a limitation.

### 4.2 Buyer journey (Arjun) — "order tonight, traceable by lunch"

1. **Sales visit (once):** Founder walks in with a sample crate of graded produce — the demo is the pitch. Arjun installs the app, OTP login, picks "Buyer", enters business name and delivery address. Ops activates his account (B2B accounts are vetted, not self-serve).
2. **Evening (D-1):** Opens Catalog → sees tomato Grade A ₹24/kg (market ₹28 struck through) and Grade B ₹19/kg, freshness badge "delivered within 48h of harvest". Adds 25 kg A + 10 kg B to cart → delivery date tomorrow → confirms address → places order. No payment step — invoice on delivery, B2B terms.
3. **Night (D-1) → morning (D0):** Order status timeline moves: **placed → confirmed** (ops matched supply) → **out_for_delivery** (truck dispatched) → **delivered** within the 06:00–10:00 window.
4. **On delivery:** One invoice (number on the order detail). He opens **Order Detail → trace line**: Farm (Ramesh, Olpad village) → harvested 6:10 AM → hub in 7:05 AM → dispatched 7:30 AM → delivered 8:48 AM. He shows a customer where lunch came from.
5. **Repeat:** WhatsApp price broadcast each evening links him back to the catalog. Reorders take under a minute.

### 4.3 Hub operator journey (Kiranben) — one intake morning

1. **5:00 AM:** Opens the grading queue on the ops dashboard — sees all listings posted for today, sorted by farmer arrival.
2. **Per farmer (~90 s):** Weigh → grade against the A/B wall chart → enter graded qty per grade → print/stick QR crate tag (listing id + harvest date). Listing status: posted → graded.
3. **6:30–7:15 AM:** Allocation board matches graded listings to order items (FIFO suggestion, ops confirms). Crates staged per buyer route.
4. **7:30 AM:** 3PL truck loads; she marks dispatch (sets `dispatch_ts`) — farmer payouts already fired at grading.
5. **Anytime:** A farmer disputes a grade → she points at the wall chart photos and the weighing slip. The standard, not the person, decides.

### 4.4 Ops manager journey (Meera) — one day

1. **5:30 PM (D-1):** Sets tomorrow's prices per produce: `reference_market_price` in, platform price A/B out (SOP in [14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md)). Prices go live in apps + WhatsApp broadcast.
2. **Evening:** Watches orders arrive; checks projected supply (posted listings) vs demand; nudges the FPO if short.
3. **Morning:** Monitors grading queue and allocation; resolves gaps (partial fulfilment calls to buyers before they discover it).
4. **Midday:** Confirms deliveries; verifies morning payouts (fired at grading); chases any stuck `out_for_delivery`.
5. **Friday:** Weekly metrics ritual — fulfilment %, median freshness hours, farmer share %, buyer repeat rate, GMV ([13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md)).

---

## 5. How the golden rules shape the product

| Golden rule ([00-GOLDEN-RULES.md](00-GOLDEN-RULES.md)) | What it forces in the product |
|---|---|
| **1. Android ↔ iOS parity** | One PRD ([07-PRD-MOBILE-APPS.md](07-PRD-MOBILE-APPS.md)) with a parity contract; shared screen IDs and string tables; a release is incomplete until both platforms ship it; QA runs one checklist against both builds. |
| **2. Global-first, Surat-first** | i18n string tables from day 1 (EN/HI at launch, GU in v1.1, pluggable); `currency` on every money field; E.164 phones; TZ-aware timestamps; `region → hub → farm` hierarchy; `reference_market_price` in the schema ("mandi" only in Indian UI copy); per-region tax abstraction (GST is an instance). |
| **3. Plan before code** | This corpus precedes all development; the existing scaffolds (KisanSetu-Backend/-Website/-Android/-iOS) are frozen until the founder approves the PRDs, then re-reviewed against them. |
| **4. Asset-light** | No route-planning, fleet, or warehouse software — we track statuses, 3PL owns routes; the hub tooling is designed to be licensable to FPOs (the SaaS), because the FPO does the physical work. |
| **5. The two metrics** | SLA timestamps (`harvest_ts → hub_in_ts → dispatch_ts → delivered_ts`) on every allocation; farmer-share arithmetic visible on every price card and payout slip; both numbers published on the website. Feature gate: serve a metric or don't ship. |
| **6. B2B, sales-led** | No B2C surface anywhere; buyer accounts are ops-activated (vetted), not self-serve growth loops; no social features, no gamification; the website is a lead-capture tool for founder-led sales, not a storefront. |

---

## 6. Product principles (suite-wide)

1. **Show the arithmetic, skip the adjectives.** Reference market price struck through, our price in leaf green, freshness in hours. Never "empowering farmers" copy — show the payout slip.
2. **Two actions max per farmer screen.** One obvious primary button, ≥48dp targets, body ≥18sp, icon + text always together, picture grids over dropdowns, bottom nav ≤3 tabs, no hamburger.
3. **The server owns money and truth.** All totals, prices and payout amounts are computed server-side; clients render, never calculate.
4. **Traceability is the premium moment.** The trace line (farm → hub → dispatch → door, with timestamps) is the buyer-side signature feature and gets design priority.
5. **Software never blocks a delivery.** Every ops flow has a manual fallback; the pilot can run a morning on paper and reconcile later.
6. **One design system.** Tokens in [10-DESIGN-SYSTEM.md](10-DESIGN-SYSTEM.md): forest #14532D, leaf #22C55E, cream #FAF7F0, ink #1C1917, amber #F59E0B; Grade A = forest chip, Grade B = amber chip. All five surfaces draw from it.

---

## 7. Surface × persona matrix

| Persona | Android/iOS app | Website | Ops dashboard | FPO SaaS (future) |
|---|---|---|---|---|
| Farmer (Ramesh) | ✅ farmer flow | reads impact page (rarely) | — | indirectly (his data) |
| Hub operator (Kiranben) | — (MVP) | — | ✅ grading queue | ✅ primary user |
| Buyer (Arjun) | ✅ buyer flow | lands here from ads → lead form | — | — |
| Ops manager (Meera) | spot-checks both flows | — | ✅ primary user | admin |
| Prospect / press / investor | — | ✅ | — | — |

---

## 8. What the suite deliberately does NOT include

Suite-level non-goals (each PRD adds its own):

- **No chat or messaging in-app** — WhatsApp Business is where Surat HoReCa and farmers already live; building chat means moderation, notification plumbing and support load that serve neither metric.
- **No farmer-to-farmer social network** — network-effects cold-start trap, explicitly rejected in the idea evolution; we are B2B infrastructure.
- **No B2C storefront** — golden rule 6; households are a different logistics and margin game (the graveyard: Otipy, Fraazo, ReshaMandi).
- **No in-app payments at MVP** — farmer payouts are ops-triggered server-side (UPI via abstracted provider; the India instance is **FIXED: Razorpay** — founder, 14 Jul 2026; no other PSP will be evaluated or integrated); buyers pay on invoice, and if/when online buyer collection starts it also runs on Razorpay ([06-PRD-BACKEND.md](06-PRD-BACKEND.md) §6.8, [21-AI-EXECUTION-PLAYBOOK.md](21-AI-EXECUTION-PLAYBOOK.md) §10). In-app collection adds PSP + app-store complexity before we've validated the trade itself.
- **No owned-logistics software** — no route optimization, driver apps or fleet telemetry; 3PL partners own routes, we own status timestamps.
- **No advisory/agronomy content at MVP** — DeHaat/AgroStar's lane; revisit at v2 only if it serves supply reliability.

---

## 9. Rollout of the suite

| Phase | Surfaces live | Gate |
|---|---|---|
| **Phase 0 — validation** | None (docs + sales deck) | ≥5/10 buyers willing to switch (kill criterion <3/10 → pivot to FPO-SaaS-first), 1 FPO MoU, crops frozen ([13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md)) |
| **MVP (pilot)** | Website → ops dashboard → mobile apps (both platforms) | 1 hub, 2 crops, 25–50 farmers, 15–25 buyers |
| **v1.1** | + offline queue, push (apps); SLA counter (website) | Pilot go/no-go: ≥70% buyer weekly repeat, ≥95% fulfilment, farmer share ≥60% documented |
| **v2** | + voice input, advisory-lite (apps); buyer analytics | Post-pilot traction |
| **FPO SaaS** | Hub tooling licensed to partner FPOs | Pilot FPO success + second FPO signed |

Full sequencing in [12-DEVELOPMENT-PLAN.md](12-DEVELOPMENT-PLAN.md) and [13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md).

---

## 10. Open questions

| # | Question | Current decision (until overridden) | Owner / deadline |
|---|---|---|---|
| 1 | Buyer web ordering portal — many HoReCa managers may prefer web to app | Not in MVP; decide from Phase 0 interview data (reuses the same API if built) | Founder, end of Phase 0 |
| 2 | Does the hub operator get a mobile grading flow instead of the web dashboard? | Web dashboard on tablet at MVP; mobile grading is an FPO-SaaS feature | Founder, before FPO SaaS spec |
| 3 | Marketplace-first vs FPO-SaaS-first emphasis | Marketplace-first, SaaS as moat + pivot path | Founder, T0.7 decision memo |
| 4 | Brand name clearance | "KisanSetu" pending trademark check (T0.5); all surfaces must treat the name as a replaceable string | Founder, end of Phase 0 |
