# 04 — GO-TO-MARKET, SALES & MARKETING

_Doc owner: Alpesh (founder). Status: Draft v1.0 for founder approval. Last updated: 2026-07-16._
_Reads with: 02-MARKET-RESEARCH.md (who the buyers are and why the lane is open), 03-BUSINESS-MODEL.md (pricing, contribution math that GTM must respect — especially route density), 08-PRD-WEBSITE.md (lead capture this plan feeds), 13-LAUNCH-PLAN.md (phase gates), 14-OPS-PLAYBOOK.md (hub and payout SOPs), 17-FUNDRAISE-FINANCE.md (marketing budget inside the burn)._

**Operating principle (Golden Rule 6): this is a B2B, sales-led business. Ads support sales; they never replace it.** The founder selling with a sample crate is the strategy, not a stopgap until "real marketing" starts.

---

## 1. Positioning & messaging

**Category:** graded, traceable, farm-direct fresh produce supply for food businesses.

**The two public numbers are the brand (Rule 5):** median hours harvest→door (<36h target, <48h promise) and farmer share of final price (≥60%, published). Every asset — website, price card, payout slip, press pitch — carries at least one of them.

Messages per audience (exact copy; localize per region, never corporate-ize):

| Audience | Message | Proof shown |
|---|---|---|
| Restaurant owner | "Graded vegetables, harvested yesterday, one invoice, at your current price or better." | sample crate + price comparison card |
| Hotel/caterer purchase manager | "≥95% fulfilment and priority supply in shortage weeks — with farm-level traceability for your audits." | trace-line demo in the buyer app + SLA report |
| Farmer (via FPO) | "आज तोलो, आज पैसा। मंडी से ज़्यादा।" (weigh today, paid today, more than the market chain) | live UPI payout + printed slip showing reference price vs KisanSetu price |
| FPO leadership | "We bring you urban demand and free software; you do what you already do — aggregate and grade." | ops dashboard demo + MoU draft |
| Press/investors | "Every asset-heavy fresh player died in 2024–25. We're the asset-light survivor design — and we publish our farmer-share number." | 02-MARKET-RESEARCH.md data |

**Voice (from 10-DESIGN-SYSTEM.md):** no "empowering farmers" speak. Show the payout slip. Freshness in hours, not adjectives.

## 2. Phase-0 validation playbook (before any build — Rule 3)

Goal: replace assumptions with data in 2 weeks and produce a go / pivot decision. Tasks mirror 15-TASKS-BACKLOG.md T0.x; unit-economics targets from 03-BUSINESS-MODEL.md §5.

### 2.1 Build the target list (T0.1)

30 Surat HoReCa targets in a sheet: name, area, cuisine, seats/covers, est. daily veg spend, current supplier type, decision-maker name, phone. Sources: Zomato/Swiggy listings by locality, hotel & restaurant association member lists, walking the food streets (Ghod Dod Road, Vesu, Adajan, Piplod, City Light, textile-market canteens). Interview 10: 5 restaurants, 2 hotels, 2 caterers, 1 cloud kitchen.

### 2.2 Buyer interview script (30–40 min, in person, at their slow hour 3:30–5:30 pm)

Rules: ask, then shut up. No pitching until Q12. Record numbers verbatim. Ask to see a real purchase bill (photo with permission).

1. "Walk me through how vegetables got into your kitchen this morning — who, when, how ordered?"
2. "Who are your current suppliers and how long have you used them?" (loyalty depth)
3. "What do you check when the produce arrives? What happens when it's bad?" (rejection/credit process — our QA-credit design must beat this)
4. "Roughly how much do you spend on vegetables daily? Can you show me yesterday's bill?" (**AOV — replaces ₹2,000 placeholder**)
5. "How many days a week do you order? Split across how many vendors?" (**frequency + share-of-wallet**)
6. "Which 8–10 items are your daily staples? Which hurt most on quality?" (**beachhead-crop selection, T0.6**)
7. "Rank your top 3 frustrations: price volatility, quality inconsistency, short supply in season spikes, delivery timing, billing/GST mess, credit terms." (pain ranking)
8. "What payment terms do you get today? What would you need from a new supplier?" — then, explicitly: **"Our terms are pay-on-delivery for the first 4 orders, then maximum 7-day credit with a ₹25,000 cap. Would you still switch on those terms?"** (**credit-terms demand at OUR terms, not in the abstract — feeds 03 §6; a switch conditional on 15–30-day credit scores as NO in the §2.4 rubric**)
9. "If produce arrived graded A/B, within 36 hours of harvest, with the farm name on the crate — what's that worth? Same price? 5% more? Only if cheaper?" (willingness-to-pay)
10. "Would a paid priority tier — guaranteed supply during shortage weeks, price locked 7 days — interest you? What monthly fee feels fair / expensive / absurd?" (**Priority pricing — feeds 03 Stream 3**)
11. "What would make you actually switch a first order this month? What would make you refuse forever?" (conversion trigger + deal-breaker)
12. Only now, 60-second pitch, then: "Would you take a free trial crate next week?" (**the real answer**)

### 2.3 FPO interview script (with 2 FPOs, target 1 pilot MoU — T0.3)

1. "How many member farmers, which villages, which crops, what monthly volume through the hub?"
2. "Where does hub produce go today — mandi, government procurement, traders? At what realization?"
3. "How do you grade today, if at all? Who decides, on what standard?"
4. "What does the hub charge or retain per kg? What are its costs?" (**FPO fee — replaces ₹2.4/kg placeholder**)
5. "How do members get paid today, and after how many days?" (instant-UPI contrast)
6. "Have agritech companies approached you before? What happened?" (scar tissue — expect Ninjacart-era stories)
7. "If we bring confirmed urban orders and free grading/procurement software, and pay members instantly on pickup — what would you need from us to run a 2-crop pilot?"
8. "Who signs, and what does your board need to see?" (MoU path)
9. "Can we spend one morning at your hub this week?" (the real answer)
10. "What broke the last partnership you tried?" (design the MoU around this answer)

### 2.4 Willingness-to-switch rubric (pre-committed, no post-hoc rationalizing)

Score each buyer 0–3: **0** = happy, won't try. **1** = polite interest, no commitment. **2** = named a concrete condition and accepted a trial crate. **3** = accepted trial AND stated a start condition we can meet within 30 days.

**Scoring override (added 16 Jul 2026):** a "2" requires the named condition to be one we can actually meet. A switch conditional on **>7-day credit terms or exposure above the ₹25k cap** (Q8) scores at most **1** — the condition violates the 03 §6 net-7 line, so it is not a switch we can win. No post-hoc exceptions.

**Go:** ≥5 buyers scoring 2+. **Kill/pivot:** <3 scoring 2+ → execute FPO-SaaS-first pivot (03-BUSINESS-MODEL.md §8). Between 3 and 4: extend to 10 more interviews, one time only.

### 2.5 Mandi morning shadow (T0.4) — data checklist

One 4 am–9 am visit: for 8 target vegetables record farmgate price paid to arriving farmers, each hop's margin (commission agent → wholesaler → sub-wholesaler → retailer/vendor), waiting time, weighing/deduction practices ("kaanta" cuts), wastage piles, time from farm arrival to buyer dispatch. Output: the **broken-chain diagram with real Surat numbers** — used on the website (08-PRD-WEBSITE.md), the sales price card, and the pitch deck.

**The existential measurement (added 16 Jul 2026 — do NOT skip):** for the same crops on the same day, also record **what a restaurant actually pays its current vendor** (landed price — from the §2.2 interview bill photos and the HoReCa-supplier hop of the chain). Compute **farmer share of the HoReCa landed price** per crop. This — not the farmer-share-of-retail number — is the denominator our ≥60% promise and entire take-rate live inside. Thresholds (pre-committed in [03-BUSINESS-MODEL.md](03-BUSINESS-MODEL.md) §5): **≥55% → structural-spread alarm, marketplace thesis fails on arithmetic; 45–55% → re-run the 03 §4 model with measured numbers before any build decision; <45% → the spread exists, proceed to the interview verdict.**

## 3. ICP & buyer tiering (who we sell to, in what order)

| Tier | Segment | AOV (est.) | Why this order | Sales motion |
|---|---|---:|---|---|
| 1 | Standalone restaurants (20–100 covers), cloud kitchens | ₹1,500–3,000 | fast decision (owner on site), daily orders, feedback loop | founder walk-in + sample crate |
| 2 | Caterers & banquet kitchens | ₹4,000–15,000 (spiky) | huge AOV, planned menus; Grade B fits them | referral + association intro |
| 3 | Hotels (3–4 star) | ₹3,000–8,000 | procurement process, audits — needs traceability story matured | meetings, trial via F&B manager |
| Later | Modern retail, processors | — | different quality specs and payment terms; not before city #2 | not in pilot — explicitly out of scope |

**Cluster rule (from 03 §4.2): sell one locality at a time.** Route density below ~6 drops is contribution-negative, so the sales territory for month 1 is ONE food-dense zone (pick from Vesu / Adajan / City Light after the target-list mapping), and we saturate it before opening the next.

## 4. Founder-led sales playbook

Cadence: **5 visits/day**, 11:00–13:00 and 15:30–18:30 (avoid service rush), 6 days/week during pilot. CRM = one Google Sheet (columns: name, area, tier, stage: lead→visited→demo→trial→active→dormant, next action, owner) + WhatsApp labels mirroring stages. Every visit logged same day.

### 4.1 The sample-crate demo (the demo IS the pitch)

Bring: one crate with both grades of the 2 beachhead crops, QR tag on the crate, printed price comparison card (their staple items: current vendor rate vs our rate), one farmer payout slip (real, consented), the 1-page leave-behind (HI+EN).

Script (~4 minutes, then listen):

> "Namaste, main Alpesh — KisanSetu se. Two minutes? *(open the crate)* This was harvested yesterday morning at [village], 40 km from here — scan this QR, you'll see the farm, the farmer, and the harvest time. We grade at the collection hub: this is Grade A, this is Grade B — B is 15% cheaper and perfect for gravies and staff meals. Here's the price card against what you're paying now — same or better on your staples. One invoice, GST-clean, delivered before 7 am. And one thing nobody else will show you: this is the slip of what the farmer got paid — more than sixty percent of what you pay. Fresher for you, fairer for him, same money. **First crate is free — what are tomorrow's five items?**"

Close on a **specific first order**, never on "think about it." If no order: get the top-5 staples list and permission to WhatsApp the daily price list — that's a lead, not a loss.

### 4.2 Objection handling (exact responses)

| Objection | Response |
|---|---|
| "My vendor gives 30-day credit." | "We do 7 days after your first four orders — and because you're not funding three middlemen, the price card usually saves you more than the credit is worth. Compare one week's bills with me." (Never match 30 days — policy in 03 §6.) |
| "My guy has been with me 10 years." | "Keep him. Give us your 5 most quality-sensitive items — leafy greens and tomatoes — and judge us on those for two weeks." (Wedge share-of-wallet, don't demand switching.) |
| "Price will creep up after the trial." | "Prices are published in the app every morning against the market reference price — you'll see both numbers. And Priority members get a 7-day lock." |
| "Startups like you shut down." (they'll remember Otipy) | "The ones that died owned trucks and warehouses. We own neither — the FPO grades, a cold-chain partner delivers, and we're profitable per order. That's exactly why we're still here." |
| "Quality claim, but what if a crate is bad?" | "Reject it at the door, photo in WhatsApp, credit on the same day's invoice. It's in the terms." |

### 4.3 Trial & funnel targets

Trial terms: first delivery fee waived, no minimum on order #1, no lock-in, reject-at-door right. Funnel targets (reviewed weekly): visit→demo 60%, demo→trial 40%, trial→active (≥4 orders in 2 weeks) 50%. That's ~3 new active buyers per 25 visits — one week of walking. At those rates founder-led sales alone reaches the M4 target of 25 active buyers (03 §7) with ads contributing on top.

## 5. Post-launch advertisement plan (supports sales, per-channel budgets)

Total paid + tools budget inside the ₹2.5–4L/mo burn (17-FUNDRAISE-FINANCE.md §5): **₹25–45k/month.**

### 5.1 WhatsApp Business (the primary buyer channel — ₹0–3k/mo)

- Business profile + verified catalog of the live SKUs with grade tiers.
- **Daily price broadcast, 06:45**, to opted-in buyers and leads. Format (exact template):
  > *KisanSetu — आज के दाम / Today's prices (Sun 12 Jul)*
  > Tomato A ₹40 · B ₹34 (market ref ₹46)
  > Palak A ₹46 · B ₹38 (market ref ₹52)
  > Order by 4 pm → delivered before 7 am. Reply 1 to order, 2 for a callback.
- Quick replies for: today's prices, place order, delivery status, QA complaint (photo → same-day credit).
- Click-to-chat number on the website, price card, invoice, and van signage.
- Post-MVP this runs on the WhatsApp Business API (15-TASKS-BACKLOG.md T2.12); during pilot it's the founder's phone + WhatsApp Business app — acceptable manual backend (14-OPS-PLAYBOOK.md).

### 5.2 Google Business Profile + local SEO (₹0, founder time)

- GBP listing: category "Produce wholesaler", service area Surat, photos of graded crates and the hub, weekly posts (price highlights, new crops), review requests to every active buyer after order #10.
- Website landing sections (08-PRD-WEBSITE.md) target 10 keywords: "vegetable supplier for restaurants Surat", "wholesale vegetables Surat", "hotel vegetable supplier Surat", "farm fresh vegetables B2B Surat", "restaurant vegetable wholesale Surat", "bulk vegetables for caterers Surat", "cloud kitchen vegetable supply Surat", "fresh produce supplier Surat", "onion tomato wholesale price Surat", "traceable FSSAI vegetable supplier Surat".
- Every lead → `POST /leads` with `source` field (06-PRD-BACKEND.md) so channel CAC is measurable from day 1.

### 5.3 Geo-fenced Meta ads (₹15–20k/mo pilot)

- Structure: 1 campaign, 2 ad sets. **Ad set A (70%):** lead-gen forms, Surat 15 km radius, interests: restaurant management, HoReCa, food service, catering; age 24–55. **Ad set B (30%):** retargeting website visitors + WhatsApp-click custom audience.
- Creative: 15-sec reel of crate packing at the hub with on-screen harvest timestamp; carousel "market ref price vs our price"; the payout-slip photo ad ("this is what the farmer got"). All creatives carry one of the two public numbers.
- Lead form fields: name, business name, area, phone (auto), top 3 items you buy daily.
- Targets: CPL ₹150–250 → 60–130 leads/mo → (25% lead→demo assumed, then demo→trial 40% and trial→active 50% per §4.3) → **3–7 new active buyers/mo from paid**. Paid CAC ≈ ₹3,000–5,000; a base buyer contributes ~₹3,000/mo (15 orders × ₹200, 03 §4) → **payback ≈ 1–1.7 months**. Kill rule: pause any ad set with CPL >₹400 for 2 consecutive weeks.

### 5.4 Referral program (₹1,500 cost/referral, capped)

- Offer: an active buyer whose referral completes 4 orders gets **one month of free delivery** (≈ 25 orders × ₹50 ≈ ₹1,250 value, within the ₹1,500 cap). Referred buyer gets first-delivery-free trial terms.
- Cap: 2 rewarded referrals/buyer/quarter. Tracked during pilot via lead `source='referral'` (06-PRD-BACKEND.md `leads` table) plus referrer name in the CRM sheet. Restaurant owners talk to each other — this should become the #1 channel by M6; target ≥30% of new buyers via referral by then.

### 5.5 Trade associations & events (₹2–5k/mo)

Surat hotel & restaurant association meetings (ask for 10 minutes + sample crates at the door), food-expo stall once in the first 6 months, textile-market canteen operators as a caterer-adjacent niche.

## 6. Farmer-side channel (supply GTM)

Farmers are recruited **through the FPO's trust, never around it.** We do not poach non-member farmers in pilot; the FPO relationship is the channel and the moat.

1. **Village demo days at the hub** (run-of-show in 13-LAUNCH-PLAN.md): FPO convenes 25–50 members → live weigh + grade of one farmer's lot → **live UPI payout on the projector/phone, the money arriving is the demo** → printed slip handed over → app walkthrough on 3 shared phones (farmer flow, 07-PRD-MOBILE-APPS.md) → sign-up desk with FPO staff. Target: 25–50 farmers onboarded in 2 demo days (T6.4).
2. **The payout slip is the advertisement.** Printed slip per pickup: farmer name, crop, kg, grade, reference market price (struck through), KisanSetu price, amount, UTR, time. Slips travel through villages by hand — deliberately designed as marketing collateral (template task T1.9).
3. **Consent-based payout reels**: 2/month, 30–60s, Gujarati/Hindi — real farmer, real slip, real UPI notification sound, no music-video gloss. Written consent form (18-LEGAL-COMPLIANCE.md), ₹500 courtesy payment to the featured farmer. Organic distribution: FPO WhatsApp groups, village groups, Instagram/YouTube Shorts. Never paid promotion on farmer-side content — trust content, not ads.
4. **Price broadcast to farmers**: daily WhatsApp/SMS of tomorrow's platform prices for the 2 crops, so listing on KisanSetu becomes a morning habit (mirrors the app's Home price cards).

## 7. PR / press plan (gated: nothing before pilot metrics exist, M5+)

- **Angle 1 (lead): "the asset-light survivor thesis."** Otipy, Fraazo, ReshaMandi, Greenikk, Deep Rooted all died asset-heavy in 2024–25; Ninjacart pivoted away. We're the model built to survive — and here are our live numbers. (Data from 02-MARKET-RESEARCH.md.)
- **Angle 2: radical transparency** — "the produce company that publishes its farmer-share percentage and its harvest-to-door hours" (live SLA counter on the website, T3.3).
- **Angle 3 (local): Surat-founder story** for Gujarati press.
- Targets: Inc42, YourStory, ET tech/agri desks, AgFunder News (global, sets up country #2 credibility); local: Divya Bhaskar, Gujarat Samachar business pages, TV9 Gujarati.
- Press kit: 1-page fact sheet (verified market numbers only, cited), founder bio + photo, hub/crate photo set, one consented farmer story with payout slip, SLA dashboard link.
- **Gate: no press outreach until go/no-go metrics are green** (≥70% weekly repeat, ≥95% fulfilment, farmer share ≥60% documented — 13-LAUNCH-PLAN.md). Press on top of weak ops burns the one story we get.

## 8. Replication templates

### 8.1 City #2 (Gujarat first: shortlist Vadodara / Ahmedabad / Rajkot)

Entry criteria (all must hold): ≥500 organized HoReCa outlets; ≥1 willing FPO within 50 km; 3PL cold route available per-trip; ≤4 h travel from Surat for founder oversight; Surat at operating break-even first (03 §7).

T-minus playbook (6 weeks, everything is a template reused from Surat):

| Week | Action | Template reused |
|---|---|---|
| T-6 | FPO scouting via SFAC/NABARD directories; 2 meetings | FPO interview script §2.3, MoU draft |
| T-5 | 30-target buyer list for ONE food-dense zone | target-list sheet §2.1 |
| T-4 | 5 compressed buyer interviews (pricing/credit check, not full validation) | script §2.2, rubric §2.4 |
| T-3 | Mandi shadow + price feed configured (new region record, reference price source) | checklist §2.5; region config 03 §9 |
| T-2 | 3PL contract (3 quotes, 2 trial routes) + hub SOP transplant + city ops hire | 14-OPS-PLAYBOOK.md SOPs |
| T-1 | Founder 2-week sales blitz with sample crates; GBP listing + Meta geo-fence cloned | §4 + §5 verbatim |
| T-0 | Launch with 2 crops, 10–15 trial buyers; weekly metrics ritual from day 1 | 13-LAUNCH-PLAN.md gates |

In the product, city #2 is **configuration, not code**: new region + hub records, price feed rows, same apps (region → hub → farm hierarchy per the shared data model).

### 8.2 Country #2 (evaluate at Series-A horizon; candidates: Vietnam, Indonesia, Kenya)

Selection criteria: smallholder share >70% of holdings; dense urban HoReCa; an instant-payment rail (VietQR, QRIS, M-Pesa — swapped behind the payment-provider interface); a producer-organization structure analogous to FPOs (agricultural cooperatives); import-independent local F&V supply.

What stays constant (the global playbook): sample-crate founder-led sales, the two public numbers, payout-slip marketing, cluster-based route density rule, demo days via the producer organization, validation-before-build with the same interview scripts (translated).
What swaps (all pre-abstracted per 03 §9 and the PRDs): currency, language packs (i18n string tables), payment provider (interface per Rule 2 — the India instance is FIXED: Razorpay, for farmer UPI payouts and buyer-side collection when online collection starts; founder decision 2026-07-14, spec in 06-PRD-BACKEND.md §6.8), tax module, reference-price source, messaging channel (WhatsApp → Zalo in Vietnam / WhatsApp in Indonesia & Kenya), association equivalents. **No GTM asset may hardcode India**: templates keep `{{region}}`, `{{currency}}`, `{{reference_price_source}}` slots.

## 9. Metrics & instrumentation

Weekly GTM scorecard (sheet during pilot; ops dashboard later, 09-PRD-OPS-DASHBOARD.md): visits, demos, trials, new active buyers, funnel conversion vs §4.3 targets, CAC by channel (`source` on every lead), % new buyers via referral, WhatsApp broadcast opt-ins/outs, GBP calls, Meta CPL, active-buyer repeat rate (the north-star retention gate), farmer sign-ups per demo day. Monthly: blended CAC vs 1-month payback rule; channel kill/scale decisions in writing.

## 10. Open questions (owner + deadline)

1. **Which food-dense zone is territory #1** (Vesu vs Adajan vs City Light) — Alpesh, end of Phase-0 week 1, from target-list mapping density.
2. **Beachhead crop pair final** — Alpesh, Phase-0 week 2 (T0.6), from interview Q6.
3. **Delivery window promise** (pre-7 am only vs 2 windows) — Alpesh + ops lead, pilot week 2 — density (03 §4.2) says one window; buyer interviews may force two.
4. **Association access** (which Surat association responds first) — Alpesh, M1.
5. **Reels production** (founder phone vs ₹3–5k/mo freelancer) — Alpesh, M4, based on organic traction of first 4 reels.
6. **City #2 pick** — Alpesh, gated on Surat break-even; scorecard per §8.1 criteria.

**Decisions taken in this doc that the founder should ratify:** cluster-saturation territory rule; 7-day-credit line held in sales (no 30-day matching); trial terms (first delivery free, reject-at-door); funnel targets §4.3; Meta kill rule CPL >₹400; referral cap 2/quarter; press gated on green metrics.
