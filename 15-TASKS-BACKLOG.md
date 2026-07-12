# 15 — TASKS BACKLOG

_KisanSetu Brain · July 2026 · Owner: Alpesh (founder; owner defaults to founder until hires land). Siblings: [12-DEVELOPMENT-PLAN.md](12-DEVELOPMENT-PLAN.md) (sequencing), [13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md) (go-live), [14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md) (ops SOPs), PRDs 06–09._

## Legend & the gate

- Status: `[x]` done · `[~]` in progress · `[ ]` open · `[G]` **GATED — blocked until "PRD approval by founder"**
- Size: **S** ≤ 1 day · **M** 1–3 days · **L** 1–2 weeks
- Every task has acceptance criteria (AC). A task is done when its AC are demonstrably true, not when the work "feels finished".

> **THE GATE (Golden Rule 3 — PLAN BEFORE CODE):** every task in workstreams **T2 (backend), T3 (website + ops dashboard), T4 (Android), T5 (iOS)** is `[G]` — blocked until the founder has read and formally approved [06-PRD-BACKEND.md](06-PRD-BACKEND.md), [07-PRD-MOBILE-APPS.md](07-PRD-MOBILE-APPS.md), [08-PRD-WEBSITE.md](08-PRD-WEBSITE.md), [09-PRD-OPS-DASHBOARD.md](09-PRD-OPS-DASHBOARD.md) and [10-DESIGN-SYSTEM.md](10-DESIGN-SYSTEM.md) in writing (a dated "APPROVED" note per PRD, filed in this repo). Code scaffolds already exist at `~/Desktop/KisanSetu-Backend/-Website/-Android/-iOS` — they are **inputs, not progress**: on approval, each scaffold is re-reviewed against its PRD and either adopted, refactored, or discarded. No scaffold task is marked done in this backlog.

---

## T0 · Validation & setup (Phase 0 — BLOCKS EVERYTHING DOWNSTREAM)

| ID | St | Task | Acceptance criteria | Size |
|---|---|---|---|---|
| T0.1 | [ ] | List 30 Surat HoReCa targets | Sheet with name, area, cuisine, est. daily veg spend, contact, source (Zomato/Swiggy/associations); ≥30 rows, ≥5 per buyer sub-type | S |
| T0.2 | [ ] | Run 10 buyer interviews (5 restaurants, 2 hotels, 2 caterers, 1 cloud kitchen) | Filled interview sheet per buyer: current supplier, weekly veg spend, AOV, pain ranking, willingness-to-switch (committed Y/N), acceptable price position; go/pivot verdict computed (≥5 switch = strong, <3 = pivot per [16-RISKS-MITIGATIONS.md](16-RISKS-MITIGATIONS.md) R1) | M |
| T0.3 | [ ] | Identify Surat-district FPOs; meet 2; sign 1 pilot MoU | FPO shortlist from SFAC/NABARD directories; 2 meeting notes; 1 MoU signed per [18-LEGAL-COMPLIANCE.md](18-LEGAL-COMPLIANCE.md) §6 outline | M |
| T0.4 | [ ] | Mandi morning shadow + one restaurant procurement cycle | Documented price spreads, hop count, timings for 8 target vegetables; farmer-share baseline computed from observed spreads | S |
| T0.5 | [ ] | Trademark search "KisanSetu" + domain + store-name availability | IP India search report (classes 9, 31, 35, 39, 42); decision memo keep/rename; domain + Play/App Store names + social handles reserved | S |
| T0.6 | [ ] | Freeze 2 beachhead crops | Decision memo naming 2 crops with interview-demand evidence; grading-feasibility note per crop (can A/B be judged by photo chart?) | S |
| T0.7 | [ ] | Decision memo: marketplace-first vs FPO-SaaS-first | 1-page memo with the T0.2 data as input; decision recorded; MVP scope in [12-DEVELOPMENT-PLAN.md](12-DEVELOPMENT-PLAN.md) matches it | S |
| T0.8 | [ ] | Incorporate Pvt Ltd; apply FSSAI + GST; Gujarat APMC review | CIN issued; FSSAI + GST ARNs; CA/advocate-signed memo on direct-procurement + mandi-fee position per [18-LEGAL-COMPLIANCE.md](18-LEGAL-COMPLIANCE.md) §2–5 | M |
| T0.9 | [ ] | Real unit economics | [03-BUSINESS-MODEL.md](03-BUSINESS-MODEL.md) placeholders (₹2,000 AOV, 10% take) replaced by interview + trial-route numbers; per-order contribution model in a sheet | S |
| T0.10 | [ ] | DPDP-ready consent texts (farmer + buyer + leads) | HI/GU/EN consent + privacy notice drafted per [18-LEGAL-COMPLIANCE.md](18-LEGAL-COMPLIANCE.md) §10; reviewed by CA/advocate | S |

## T1 · Planning & design (the Brain) — planning corpus DONE

| ID | St | Task | Acceptance criteria | Size |
|---|---|---|---|---|
| T1.1 | [x] | Brain corpus written (README, 00–18) | All 20 files exist in `/KisanSetu-Brain/`, mutually consistent with the shared brief; PRDs contain purpose/scope/non-goals/spec/edge-cases/AC/rollout/metrics | L |
| T1.2 | [x] | Design system defined | [10-DESIGN-SYSTEM.md](10-DESIGN-SYSTEM.md) + `design/design-board.html`: palette, type, spacing, components, grade-chip rules | M |
| T1.3 | [x] | Farmer app UX flow (Hindi-first, low-literacy) | Screen list + flows in [07-PRD-MOBILE-APPS.md](07-PRD-MOBILE-APPS.md); max 2 actions/screen, ≥48dp targets honored | M |
| T1.4 | [x] | Buyer app UX flow (professional B2B) | Screen list + flows incl. traceability view in [07-PRD-MOBILE-APPS.md](07-PRD-MOBILE-APPS.md) | M |
| T1.5 | [x] | Website design (single page, EN/HI) | Spec in [08-PRD-WEBSITE.md](08-PRD-WEBSITE.md) | S |
| T1.6 | [x] | Ops dashboard flows | Grading queue, allocation board, delivery statuses, payout, SLA reports specified in [09-PRD-OPS-DASHBOARD.md](09-PRD-OPS-DASHBOARD.md) | M |
| T1.7 | [ ] | **Founder PRD approval — THE GATE** | Dated written "APPROVED" (or change requests) on 06, 07, 08, 09, 10; open questions in each PRD answered or ownered | S |
| T1.8 | [ ] | App icons + wordmark finals (contract designer) | Vector wordmark, adaptive Android icon, iOS icon set, favicon; passes trademark decision from T0.5 | M |
| T1.9 | [ ] | Printed matter designed | Payout slip (fields per [14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md) §7.3), buyer invoice, A2 grade-chart template, buyer mini grade card, fallback forms #F1–#F6 | M |

## T2 · Backend (Node.js 20 + Express + PostgreSQL 16) — ALL GATED ON T1.7

| ID | St | Task | Acceptance criteria | Size |
|---|---|---|---|---|
| T2.1 | [G] | Scaffold review vs approved PRD | Existing `KisanSetu-Backend` audited against [06-PRD-BACKEND.md](06-PRD-BACKEND.md); adopt/refactor/discard memo; repo baseline tagged | M |
| T2.2 | [G] | Schema v1 migration set | All PRD tables (users, fpos, farmer_profiles, buyer_profiles, produce, price_feed, listings, orders, order_items, order_allocations, payments, leads) with region→hub→farm hierarchy, currency field, E.164 phones, timezone-aware timestamps; migrations run clean on Postgres 16 (docker :5433) | M |
| T2.3 | [G] | Auth: phone OTP + JWT + role guards | OTP request/verify endpoints; dev OTP 123456; JWT with role claims; farmer/buyer/ops guards tested | M |
| T2.4 | [G] | Produce catalog + daily price feed API | Multilingual produce names; `price_feed` upsert per produce/day with `reference_market_price`, `platform_price_a/b`; validation per [14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md) §5 constraints | M |
| T2.5 | [G] | Listings lifecycle API | posted→graded→allocated→closed transitions with role rules; graded_qty per grade; grading photo attachment | M |
| T2.6 | [G] | Orders + allocation + traceability API | Order totals computed server-side; allocation creates order_allocations; trace endpoint returns farm/farmer/village/harvest + timestamps for an order_item | L |
| T2.7 | [G] | Freshness SLA fields + report | `harvest_ts, hub_in_ts, dispatch_ts, delivered_ts` on allocations; SLA report query (median/p90 by crop, route, week) matches metric M4 definition | M |
| T2.8 | [G] | Payments: payout + invoice records | farmer_payout & buyer_invoice records with UTR; payout slip data endpoint (all §7.3 slip fields) | M |
| T2.9 | [G] | Leads endpoint | POST /leads validated + stored; feeds website form | S |
| T2.10 | [G] | Real OTP SMS gateway (MSG91/Twilio) | OTP delivered to real numbers; rate limiting; dev stub retained behind env flag | M |
| T2.11 | [G] | Razorpay UPI payout integration | Payout provider interface (abstracted per global-first rule) with Razorpay instance; sandbox payout with UTR captured; failure→retry→NEFT-flag flow per SOP | L |
| T2.12 | [G] | WhatsApp Business API integration | Order confirmations (buyer), daily price broadcast (farmer + buyer lists), opt-in recorded per [18-LEGAL-COMPLIANCE.md](18-LEGAL-COMPLIANCE.md) §9 | L |
| T2.13 | [G] | Ops endpoints | Grading queue views; allocation suggestions (match by crop/grade/qty with priority rules of [14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md) §4.2) | M |
| T2.14 | [G] | Invoice PDF generation | One consolidated invoice per delivery: exempt bill-of-supply lines + taxable delivery/handling line per [18-LEGAL-COMPLIANCE.md](18-LEGAL-COMPLIANCE.md) §4 | M |
| T2.15 | [G] | Automated tests + CI | supertest on 5 core flows (auth, listing lifecycle, order+allocation, payout, SLA report); GitHub Actions green | M |
| T2.16 | [G] | Production deploy plan (execute only when pilot needs it) | Single VM/managed Node + managed Postgres + daily backups + restore drill documented; per [11-ARCHITECTURE.md](11-ARCHITECTURE.md); no cloud spend before buyers exist | M |

## T3 · Website + ops dashboard (vanilla JS) — ALL GATED ON T1.7

| ID | St | Task | Acceptance criteria | Size |
|---|---|---|---|---|
| T3.1 | [G] | Marketing site build/review vs PRD | Existing scaffold audited vs [08-PRD-WEBSITE.md](08-PRD-WEBSITE.md): hero, broken-chain visual, 5 steps, benefits, impact stats, EN/HI toggle, lead form → POST /leads | M |
| T3.2 | [G] | Lead form wired to production + WhatsApp click-to-chat | Live submission lands in leads table; WhatsApp CTA opens business number | S |
| T3.3 | [G] | "Hours from harvest" live counter (post-pilot only) | Renders real SLA median from API; ships only after 4 weeks of live data ([13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md) §8) | M |
| T3.4 | [G] | Deploy: static hosting + domain + analytics | HTTPS on final domain; Plausible/GA4 events on lead submit | S |
| T3.5 | [G] | Ops dashboard v1 | All [09-PRD-OPS-DASHBOARD.md](09-PRD-OPS-DASHBOARD.md) MVP views: price setting, grading queue, allocation board, delivery status, payout trigger, SLA/metrics reports (M1–M8) | L |

## T4 · Android app (Kotlin + Jetpack Compose + Material3) — ALL GATED ON T1.7

| ID | St | Task | Acceptance criteria | Size |
|---|---|---|---|---|
| T4.1 | [G] | Scaffold review vs approved PRD + parity audit | Existing `KisanSetu-Android` audited vs [07-PRD-MOBILE-APPS.md](07-PRD-MOBILE-APPS.md); screen-for-screen parity checklist against iOS started (Golden Rule 1) | M |
| T4.2 | [G] | Farmer flow screens | OTP login → role pick → Home (price cards: reference struck through, KisanSetu price green, "फ़सल बेचें" button) → New Listing (picture grid, kg stepper, harvest date default tomorrow, estimated payout) → My Listings (status chips) → Payments (UTR history); builds & runs vs local API (emulator 10.0.2.2:4000, dev OTP 123456) | L |
| T4.3 | [G] | Buyer flow screens | Catalog (A/B prices vs struck market price, freshness badge) → Cart (grade chips, delivery date ≥ tomorrow) → Orders timeline → Order Detail with traceability trace-line | L |
| T4.4 | [G] | i18n: EN/HI/GU string tables | All UI from string tables; language picker; no hardcoded strings (lint rule) | M |
| T4.5 | [G] | Offline tolerance | Prices + listings cached (Room); listing post queued offline and synced; airplane-mode demo passes | M |
| T4.6 | [G] | FCM push | "आपकी फ़सल ग्रेड हो गई", "पैसा भेज दिया गया", buyer order-status pushes; deep-link to relevant screen | M |
| T4.7 | [G] | Play internal testing track | Signed build on Play Console internal track; FPO field staff installed | S |
| T4.8 | [ ] | Field usability test with 3 real farmers | Watched sessions at hub; each farmer completes listing unaided by attempt 2; confusion list filed as issues (not gated — test can run on whatever build exists when farmers are available, incl. dry-run week) | S |

## T5 · iOS app (Swift 5 + SwiftUI, XcodeGen) — ALL GATED ON T1.7

| ID | St | Task | Acceptance criteria | Size |
|---|---|---|---|---|
| T5.1 | [G] | Scaffold review vs approved PRD + parity audit | Existing `KisanSetu-iOS` audited vs [07-PRD-MOBILE-APPS.md](07-PRD-MOBILE-APPS.md); parity checklist vs Android maintained per feature | M |
| T5.2 | [G] | Farmer flow screens (parity with T4.2) | Identical screen list, copy, and behavior; SF Pro + Dynamic Type per [10-DESIGN-SYSTEM.md](10-DESIGN-SYSTEM.md) | L |
| T5.3 | [G] | Buyer flow screens (parity with T4.3) | Identical incl. traceability trace-line; builds & runs vs local API | L |
| T5.4 | [G] | i18n EN/HI/GU (parity with T4.4) | Same string keys as Android where feasible; language picker | M |
| T5.5 | [G] | APNs push (parity with T4.6) | Same notification set as Android | M |
| T5.6 | [G] | TestFlight with 5 pilot buyers | Build distributed; 5 buyers ordering on it | S |
| T5.7 | [G] | Buyer web ordering portal — decision + build | Decision memo (needed for pilot? many HoReCa managers prefer web/Android); if yes: catalog→cart→orders on same API | L |

## T6 · Pilot ops (activates with [13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md); not code-gated — WhatsApp/paper fallback is a valid backend)

| ID | St | Task | Acceptance criteria | Size |
|---|---|---|---|---|
| T6.1 | [ ] | Hub SOP finalized + grade charts produced | [14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md) §2 rehearsed once with FPO staff; A2 photo charts printed for both frozen crops | M |
| T6.2 | [ ] | 3PL selection: 3 quotes, 2 trial routes, per-trip contract | Scorecard per §6.1; both trials passed per §6.2; contract signed; vendor #2 warm | M |
| T6.3 | [ ] | Crates + QR sourcing | 60 serialized crates, label holders, thermal printer + labels on site; rotation + deposit rule in buyer agreement | S |
| T6.4 | [ ] | Onboard 25–50 farmers (FPO demo day) | Attendance sheet; VPAs verified with ₹1 test payout; consent forms (T0.10) signed | M |
| T6.5 | [ ] | Onboard 15–25 buyers | Signed one-page buyer agreements; standing delivery-day preferences recorded; sample-crate demo done per buyer | L |
| T6.6 | [ ] | Daily price-setting SOP live | 5 consecutive rehearsal evenings done per §5; both founder and Ops Lead have run it solo | S |
| T6.7 | [ ] | Dry-run week executed | All 5 drill days passed per [13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md) §3 pass conditions; fallback kit #F1–#F6 printed and used in the D3 drill | M |
| T6.8 | [ ] | Weekly metrics ritual + tracking sheet | `ops/metrics-weekly.xlsx` template with M1–M8 formulas; first ritual run on dry-run data | S |
| T6.9 | [ ] | Ops Lead hired (month 2) | Hire can run §2–§8 solo after one supervised week; payout + price SOPs cross-trained | M |

## T7 · Marketing & sales (from launch; per [04-GTM-SALES-MARKETING.md](04-GTM-SALES-MARKETING.md))

| ID | St | Task | Acceptance criteria | Size |
|---|---|---|---|---|
| T7.1 | [ ] | WhatsApp Business setup | Catalog live; farmer + buyer broadcast lists with recorded opt-ins; quick replies configured | S |
| T7.2 | [ ] | Google Business Profile + local SEO | Profile verified; 10 target keywords mapped to site copy | S |
| T7.3 | [ ] | Sales kit | 1-page leave-behind (HI+EN), sample-crate checklist, price-comparison card printed | S |
| T7.4 | [ ] | Meta lead-gen campaign | Surat geo-fence live at ₹15–20k/month; leads land in /leads; CPL tracked weekly | M |
| T7.5 | [ ] | Referral program | "Free delivery month per successful referral" terms published; tracked in buyer notes | S |
| T7.6 | [ ] | 2 farmer payout stories filmed | Signed consent per [18-LEGAL-COMPLIANCE.md](18-LEGAL-COMPLIANCE.md) §8; reels published | S |
| T7.7 | [ ] | Trade association intros | ≥2 Surat hotelier/restaurant association meetings held; member list access | S |
| T7.8 | [ ] | Press prep (post-traction only) | "Asset-light survivor" narrative doc + real SLA/farmer-share data pack; pitched only after G1 passed | S |

## T8 · Fundraise (month 5–6, only after retention data; per [17-FUNDRAISE-FINANCE.md](17-FUNDRAISE-FINANCE.md))

| ID | St | Task | Acceptance criteria | Size |
|---|---|---|---|---|
| T8.1 | [ ] | Deck updated with real traction | GMV/retention slides use weeks 1–8 actuals, not illustrative ramps | S |
| T8.2 | [ ] | Data room | Unit economics sheet, SLA logs, payout-slip farmer-share archive, MoU + buyer agreements, cap table, compliance file ([18-LEGAL-COMPLIANCE.md](18-LEGAL-COMPLIANCE.md) §12) | M |
| T8.3 | [ ] | Target investor list + sequencing | ≥15 agritech-active seed funds (Omnivore, Ankur, Rockstud et al.) + Gujarat angels; warm-intro map; outreach only after G1 review | S |

---

## Reading order for a new contributor
[00-GOLDEN-RULES.md](00-GOLDEN-RULES.md) → [05-PRODUCT-OVERVIEW.md](05-PRODUCT-OVERVIEW.md) → your workstream's PRD → [12-DEVELOPMENT-PLAN.md](12-DEVELOPMENT-PLAN.md) → this backlog. Nothing in T2–T5 starts before T1.7 exists in writing.
