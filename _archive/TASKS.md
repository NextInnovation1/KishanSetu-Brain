# KisanSetu — Task Backlog

Status legend: `[ ]` todo · `[~]` in progress · `[x]` done. Owner defaults to founder until hires land.

## T0 · Validation & setup (Weeks 1–2) — BLOCKS EVERYTHING

- [ ] T0.1 List 30 Surat HoReCa targets (name, area, cuisine, est. daily veg spend) — sources: Zomato/Swiggy listings, hotel associations
- [ ] T0.2 Run 10 buyer interviews; capture: current supplier, AOV, pain ranking, willingness-to-switch, acceptable price premium/discount
- [ ] T0.3 Identify Surat-district FPOs (SFAC/NABARD directories); meet 2; sign 1 pilot MoU
- [ ] T0.4 One mandi morning shadow: record price spreads, hop count, timing for 8 target vegetables
- [ ] T0.5 Trademark search "KisanSetu" (IP India) + domain + Play/App Store name availability
- [ ] T0.6 Freeze 2 beachhead crops from interview data
- [ ] T0.7 Decision memo: marketplace-first vs FPO-SaaS-first
- [ ] T0.8 Incorporate Pvt Ltd; apply FSSAI + GST; review Gujarat APMC rules for direct procurement
- [ ] T0.9 Set real unit economics from interview data (replace ₹2,000 AOV / 10% take-rate placeholders)

## T1 · Design (Weeks 2–4)

- [x] T1.1 Design system: palette, type, spacing, components (see design artifact + `docs/DESIGN.md`)
- [x] T1.2 Farmer app UX flow (Hindi-first, low-literacy patterns)
- [x] T1.3 Buyer app UX flow (B2B professional)
- [x] T1.4 Website design (single-page, EN/HI)
- [ ] T1.5 Ops dashboard wireframes: grading queue, allocation board, delivery statuses, payout screen
- [ ] T1.6 App icons + wordmark final files (contract designer)
- [ ] T1.7 Printed matter: farmer payout slip template, buyer invoice template, hub grading chart poster

## T2 · Backend (Node.js + Express + Postgres)

- [x] T2.1 Repo scaffold: Express app, pg pool, docker-compose Postgres, migrations runner
- [x] T2.2 Schema v1: users/roles, FPOs, farmer & buyer profiles, produce catalog (EN/HI/GU), price feed, listings+grading, orders+items, allocations (traceability), payments, leads
- [x] T2.3 Phone-OTP auth (dev stub) + JWT + role guards
- [x] T2.4 Listings API (post → grade → allocate lifecycle)
- [x] T2.5 Orders API with price computation + traceability
- [x] T2.6 Payout stub API + leads endpoint (website form)
- [x] T2.7 Seed data + smoke tests green end-to-end
- [ ] T2.8 Real OTP via SMS gateway (MSG91/Twilio)
- [ ] T2.9 Razorpay/UPI payout integration (replace stub)
- [ ] T2.10 WhatsApp Business API: order confirmations + daily price broadcast
- [ ] T2.11 Ops endpoints: grading queue views, allocation suggestions (match listings→orders by grade/qty)
- [ ] T2.12 Freshness SLA fields: harvest_ts, hub_in_ts, dispatch_ts, delivered_ts on allocations; SLA report query
- [ ] T2.13 Invoice PDF generation
- [ ] T2.14 Automated tests (supertest) on the 5 core flows; CI via GitHub Actions
- [ ] T2.15 Production deploy plan (single VM/managed host + managed Postgres + backups) — only when pilot needs it

## T3 · Website (vanilla JS)

- [x] T3.1 Single-page marketing site: hero, broken-chain visual, 5 steps, farmer/buyer benefits, impact stats, lead form, EN/HI toggle
- [ ] T3.2 Hook lead form to production backend + WhatsApp click-to-chat number
- [ ] T3.3 Live "hours from harvest" SLA counter section (post-pilot, from real data)
- [ ] T3.4 Deploy to static hosting + domain + analytics (Plausible/GA4)
- [ ] T3.5 Ops dashboard web app (can live at ops.kisansetu.in; plain JS or React — decide at T1.5)

## T4 · Android farmer app (Kotlin + Jetpack Compose)

- [x] T4.1 Project scaffold: Compose + Material3 green theme, nav, Retrofit client matching API
- [x] T4.2 Screens: OTP login, home (prices + greeting), new listing (produce grid, qty, date), my listings (status chips), payments
- [ ] T4.3 Build in Android Studio; fix compile issues; run against local backend (emulator → 10.0.2.2:4000)
- [ ] T4.4 Gujarati strings (values-gu); voice-note field on listing (many farmers prefer speaking)
- [ ] T4.5 Offline tolerance: cache prices + listings (Room); queue listing posts offline
- [ ] T4.6 FCM push: "आपकी फ़सल ग्रेड हो गई", "पैसा भेज दिया गया"
- [ ] T4.7 Play Console account; internal testing track with FPO field staff
- [ ] T4.8 Onboarding flow tested with 3 real farmers at the hub (watch them use it; fix what confuses them)

## T5 · iOS buyer app (Swift + SwiftUI)

- [x] T5.1 Project scaffold (XcodeGen): theme, API client, models
- [x] T5.2 Screens: OTP login, catalog by grade, cart, orders timeline, order detail with farm traceability
- [ ] T5.3 Generate project in Xcode; build + run against local backend
- [ ] T5.4 Push notifications for order status
- [ ] T5.5 TestFlight with 5 pilot buyers
- [ ] T5.6 Buyer web ordering portal (many HoReCa managers will prefer web/Android — reuse API; decide post-interview)

## T6 · Pilot ops (Months 3–4)

- [ ] T6.1 Hub SOP doc: intake → weigh → grade (A/B chart with photos) → crate tag (QR) → dispatch
- [ ] T6.2 3PL cold logistics: get 3 quotes, run 2 trial routes, sign per-trip contract (no owned vehicles)
- [ ] T6.3 Crate + QR label sourcing; harvest-date tagging process
- [ ] T6.4 Onboard 25–50 farmers via FPO (demo day at hub)
- [ ] T6.5 Onboard 15–25 buyers (founder sales — sample crate demos)
- [ ] T6.6 Daily price-setting SOP (mandi reference → platform price A/B) — who sets it, when, on what data
- [ ] T6.7 Week-1 dry run: 5 orders end-to-end with full manual fallback ready
- [ ] T6.8 Weekly metrics ritual: fulfilment %, freshness hours, farmer share %, repeat rate, GMV

## T7 · Marketing & sales (from launch)

- [ ] T7.1 WhatsApp Business setup: catalog, daily price broadcast list, quick replies
- [ ] T7.2 Google Business Profile + 10 local-SEO landing keywords
- [ ] T7.3 Sales kit: 1-page leave-behind (HI+EN), sample-crate checklist, price comparison card
- [ ] T7.4 Meta lead-gen campaign, Surat geo-fence, ₹15–20k/month, wired to /leads
- [ ] T7.5 Referral program: free delivery month per successful referral
- [ ] T7.6 2 farmer payout stories filmed (consent) for organic reels
- [ ] T7.7 Surat restaurant-association intro meetings
- [ ] T7.8 Press angle prep: "asset-light survivor" narrative for Inc42/YourStory (post-traction)

## T8 · Fundraise (Month 5–6, only after retention data)

- [ ] T8.1 Update deck slide 14 with real GMV/retention (replace illustrative ramp)
- [ ] T8.2 Data room: unit economics, SLA logs, farmer-share receipts, MoUs
- [ ] T8.3 Target list: agritech-active seed funds (post-winter selective revival: Omnivore, Ankur, Rockstud et al.) + Surat/Gujarat angels
