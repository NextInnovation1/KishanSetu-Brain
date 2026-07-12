# KisanSetu — Master Execution Plan

_Prepared from the founder pitch deck (v2) and context notes, July 2026. Founder: Alpesh, Surat._

**One line:** Asset-light B2B supply chain connecting Indian farmers (via FPO collection hubs) directly to HoReCa buyers — farmers keep 60–70% instead of 30–40%, buyers get graded, traceable produce within 24–48h of harvest.

**The wedge (from verified July-2026 landscape):** Ninjacart is broadening away from focused fresh-to-HoReCa; DeHaat/AgroStar are input/advisory-led; every asset-heavy B2B2C fresh player (Otipy, Fraazo, ReshaMandi) died in 2024–25 — proving the asset-light thesis. **No one owns focused, asset-light, HoReCa-first fresh produce in tier-2 India.** Surat is the beachhead.

---

## Phase 0 — Validate before building heavy (Weeks 1–2)

The context notes flag this and it stays the #1 rule: **software follows validated demand, not the other way around.**

| # | Action | Success looks like |
|---|--------|--------------------|
| 0.1 | Interview 10 Surat HoReCa buyers (5 restaurants, 2 hotels, 2 caterers, 1 cloud kitchen) | ≥5 say "yes, I'd switch for graded produce at stable prices"; real AOV data replaces the illustrative ₹2,000 |
| 0.2 | Meet 2 FPOs in Surat district | 1 signs a pilot MoU (they aggregate + grade, we bring demand + software) |
| 0.3 | Shadow one mandi morning + one restaurant procurement cycle | Documented current prices, hops, timing — our real baseline |
| 0.4 | Trademark/name check for "KisanSetu"; register domain | Name cleared or replaced; domain + social handles booked |
| 0.5 | Pick 2 beachhead crops from actual buyer demand (likely tomato + onion or leafy greens) | Crop list frozen for MVP |
| 0.6 | Decide model emphasis: marketplace-first vs FPO-SaaS-first | Decision recorded; MVP scope matches it |

**Kill/pivot criteria:** if fewer than 3 of 10 buyers show real willingness to switch, pivot to the FPO procurement SaaS wedge (sell grading/procurement software to those already doing physical work).

## Phase 1 — Design (Weeks 2–4, overlaps Phase 0)

- Brand identity: forest-green agri theme (already established in the deck), wordmark, app icons.
- Design system: colors (#14532D forest, #22C55E leaf, #FAF7F0 cream, #F59E0B amber), type scale, spacing, components. One system across website, Android, iOS.
- UX flows, farmer side (Android, Hindi/Gujarati-first, low digital literacy): OTP login → today's prices → list produce → grading result → payout. Max 2 actions per screen, 48dp+ targets, icons+text.
- UX flows, buyer side (iOS + web, professional B2B): OTP login → graded catalog → cart → order → status timeline → traceability view.
- Ops dashboard flows (internal, web): grade listings, allocate to orders, mark delivery, trigger payouts.
- Marketing website design: single-page, EN/HI toggle, broken-chain visual as centerpiece, lead form.

## Phase 2 — Build MVP (Weeks 4–12)

Stack (fixed): **Node.js + Express backend · PostgreSQL · Website in vanilla JS · Android in Kotlin/Jetpack Compose · iOS in Swift/SwiftUI.**

Build order (dependency-driven):
1. **Backend + DB** (weeks 4–6): auth (phone OTP + JWT), produce catalog + daily price feed, listings + grading, orders + allocation (traceability), payouts (stubbed UPI), leads. _Scaffolded — see `KisanSetu-Backend/`._
2. **Ops dashboard** (weeks 6–8): the business runs on this before anything else — grading queue, order allocation, delivery status, payout trigger. (Simple JS web app hitting the same API.)
3. **Mobile apps — Android + iOS, identical (golden rule)** (weeks 6–10): dual-role apps — farmer flow (login, prices, list produce, my listings, payments) + buyer flow (catalog by grade, cart, orders, traceability). _Scaffolded — see `KisanSetu-Android/` and `KisanSetu-iOS/`._
4. **Buyer web ordering portal** (weeks 8–11): reuse the same API; decide plain JS vs React at T1.5.
5. **Marketing website** (week 4, early — needed for credibility in sales meetings). _Built — see `KisanSetu-Website/`._
6. Integrations (weeks 10–12): Razorpay/UPI payouts, WhatsApp Business API for order confirmations (buyers) and price alerts (farmers), FCM push.

**MVP definition (smallest thing that proves the model):** one FPO hub can grade produce logged by 10 farmers, 10 buyers can order by grade and receive within 24–48h, farmer sees payout same day, every crate traceable. Everything else waits.

**Ops reality check:** in weeks 1–4 of the pilot, WhatsApp + phone calls are an acceptable backend. The software must never block a delivery.

## Phase 3 — Pilot launch (Months 3–4)

- 1 FPO hub, 2 crops, 15–25 HoReCa buyers, ~25–50 farmers via the FPO.
- Freshness SLA instrumented from day 1: harvest timestamp → delivery timestamp on every crate.
- Founder does sales and morning hub visits personally (founder-led sales is the strategy, not a stopgap).
- Weekly metrics review: fulfilment rate, freshness hours, farmer payout %, buyer repeat rate, take rate realized.

**Go/no-go for scale-up:** ≥70% weekly buyer repeat rate, ≥95% fulfilment, farmer share ≥60% documented.

## Phase 4 — Live & grow (Months 4–6)

- Add modern retail + food processors as buyer types; widen crop range.
- Pilot FPO procurement SaaS with partner FPO (the data/tech moat).
- Instrument unit economics per order; target contribution-margin positive per order by month 6.
- Prepare seed-raise narrative on retention + GMV ramp (deck already structured for this).

## Phase 5 — Marketing & advertisement (starts at launch, scales after)

**This is a B2B, sales-led business. Ads support sales; they don't replace it.**

Buyer side (HoReCa):
1. Founder-led direct sales — walk-ins to restaurant owners with a sample crate of graded produce (the demo IS the pitch). Target: 5 visits/day during pilot.
2. WhatsApp Business — catalog + daily price list broadcast; this is where Surat HoReCa already lives.
3. Google Business Profile + local SEO ("fresh vegetable supplier Surat", "restaurant vegetable wholesale").
4. Meta ads, geo-fenced Surat, targeting restaurant owners/managers — small budget (₹15–20k/month), lead-gen forms into the website's `/leads` endpoint.
5. Referral: one month of free delivery for a successful buyer referral.
6. Trade bodies: Surat hotelier & restaurant associations, food-expo stalls.

Farmer side (supply):
1. Through the FPO — their trust is the channel; village demo days at the hub.
2. The payout receipt is the ad: instant UPI + printed slip showing "mandi price vs what you got".
3. Local-language reels/shorts of real farmers getting paid (with consent) — organic, not paid.

Brand/PR: agritech press (Inc42, YourStory) angle — "the asset-light survivor thesis"; document the freshness SLA publicly (a live "hours from harvest" counter on the website is a credibility weapon).

## Team & money (first 6 months, lean)

- Founder (CEO/CTO): product, tech, sales.
- Hire 1: Ops lead at the hub (grading + logistics coordination) — month 2.
- Hire 2: Field sales/onboarding (HoReCa + farmer-facing) — month 3.
- Contract: part-time designer (month 1), CA for compliance.
- Compliance checklist: company incorporation (Pvt Ltd), FSSAI license, GST registration, APMC rules review for direct procurement in Gujarat, mandi-fee implications, agreement templates for FPO + buyers.
- Indicative burn: ₹2.5–4L/month (2 hires + hub costs + logistics per-trip + small ad budget). Bootstrap/friends-family to pilot proof, then seed.

## Deployment posture

Local/dev now; deploy publicly only when the pilot needs it (cheap single VM or managed Node host + managed Postgres; website on static hosting). Do not pay for cloud before buyers exist.

## The two numbers that matter all year

1. **Freshness:** median hours harvest→door (target <36h, promise <48h).
2. **Farmer share:** % of buyer price reaching the farmer (target ≥60%, publish it).

Everything else is instrumentation around these two.
