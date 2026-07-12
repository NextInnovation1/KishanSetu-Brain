# 12 — Development Plan

**What this is:** how development WILL run once un-paused. Development is currently **PAUSED** (Golden Rule #3, `00-GOLDEN-RULES.md`): no code until the founder approves the PRD corpus. This document is the operating manual for the build phase that follows approval.
**Owner:** Alpesh (founder = solo dev + PM for the MVP) · **Status:** Draft for founder approval
**Reads with:** `06-PRD-BACKEND.md`, `07-PRD-MOBILE-APPS.md`, `08-PRD-WEBSITE.md`, `09-PRD-OPS-DASHBOARD.md`, `10-DESIGN-SYSTEM.md`, `11-ARCHITECTURE.md`, `13-LAUNCH-PLAN.md`, `15-TASKS-BACKLOG.md`, `19-PRD-CMS-ANALYTICS.md`.

---

## 1. Un-pause preconditions (all must be true before week 1)

1. Founder has approved PRDs 05–11 + `19-PRD-CMS-ANALYTICS.md` and this plan (sign-off recorded in `README.md` change log).
2. Phase 0 gates met (`13-LAUNCH-PLAN.md`): ≥3/10 buyer interviews positive (else FPO-SaaS pivot re-plans everything), 1 FPO pilot MoU signed, 2 beachhead crops frozen, name/trademark check done.
3. **Existing scaffolds are reviewed, not trusted:** code in `KisanSetu-Backend/`, `-Website/`, `-Android/`, `-iOS/` predates these PRDs. Week 1 includes a gap review of each scaffold against its PRD; keep what conforms, rewrite what doesn't. The PRDs win every conflict.
4. Real unit economics from Phase 0 replace illustrative numbers anywhere they appear in UI copy (website receipt bars, price examples).

## 2. Build order & rationale

Dependency-driven, matching `11-ARCHITECTURE.md` (one backend, one database, four thin clients):

```
W1–3   BACKEND        auth/OTP → catalog+price_feed → listings → orders+allocations → payments stub → leads → events storage (app_events) → seed+smoke
W1     WEBSITE v1     deploy early — credibility for founder's parallel sales meetings (08-PRD-WEBSITE.md §13)
W3–4   OPS DASHBOARD  the business runs on this before any consumer UI exists
W4–5   CONSOLE CMS    analytics module (19-PRD-CMS-ANALYTICS.md) in the same app, right after the ops module
W3–8   MOBILE         Android+iOS in lock-step cross-platform units (never sequential platforms)
W8–9   INTEGRATIONS   Razorpay payouts, WhatsApp Business, FCM/APNs, real SMS OTP; website→prod leads + analytics
W9–10  HARDENING      field testing, bug burn-down, release candidates, pilot dry run
```

Why this order: (1) every client consumes the same REST API, so the API contract freezes first; (2) the **ops dashboard precedes mobile** because during pilot week 1 the hub can run entirely on ops + WhatsApp — the farmer/buyer apps improve the experience, ops *is* the business; (3) the website ships first because it has no backend dependency except `POST /leads` (stub acceptable for 2 weeks) and the founder is selling from week 1; (4) integrations go last because every external approval (Razorpay KYC, WhatsApp Business verification, SMS DLT) runs on third-party clocks — **applications for all of them are filed in week 1** even though integration code lands in week 8 (see risks §10); (5) the Console's CMS/Analytics module (`19-PRD-CMS-ANALYTICS.md`) shares the ops dashboard's login, codebase and API, so it ships right after it (W4–5) — but its C-4 (app & users) screen can only show history that was *stored*, so the `POST /events` → `app_events` storage amendment lands with the backend milestone (W1–3), before the first mobile build reaches a device in W5: DAU history starts on day 1 of app release.

## 3. Parity workflow (Golden Rule #1 made operational)

**The unit of mobile work is the cross-platform unit, not the platform ticket.** One unit = one feature slice spanning: API change (if any) + Android implementation + iOS implementation + string keys (EN/HI/GU stubs) + design-token/component usage + analytics events. Examples: "Farmer Home + price cards", "OTP login", "Buyer order detail + trace line".

**Parity scope note:** Golden Rule #1 applies to the two mobile apps, **not** to the KisanSetu Console (internal, web-only — `09-PRD-OPS-DASHBOARD.md` + `19-PRD-CMS-ANALYTICS.md`). But the analytics events that feed the Console's CMS module **are** under the parity rule via `07-PRD-MOBILE-APPS.md`: identical event names, properties, and firing points on Android and iOS — that is what the checklist line "Same analytics event names + properties fire on both" protects, and what keeps C-4's DAU/retention numbers trustworthy.

Rules:

1. A unit is **one entry** in the tracker; it cannot be closed with only one platform done. There is no such thing as "Android shipped, iOS next sprint."
2. Same branch name in both app repos: `feat/<unit-slug>` in `KisanSetu-Android` and `KisanSetu-iOS`; PRs cross-link.
3. Build the *harder* platform's screen first when uncertain (usually the one the founder knows less well) — porting down is easier than discovering a constraint late.
4. Strings are added to the shared string table spec (`07-PRD-MOBILE-APPS.md` appendix) first, then to `strings.xml` / `Localizable.strings` with **identical keys**.
5. Every unit's PRs attach a **side-by-side screenshot grid**: (Android Pixel 6a, iPhone SE 3) × (light, dark) × each screen state. Generated from emulator/simulator, pasted in the PR description.

**Parity checklist** (pasted into both PRs; every box checked or the unit isn't done):

- [ ] Same screens, same navigation position on both platforms
- [ ] Same states implemented: default · loading · empty · error · offline
- [ ] Same string keys, all three languages present (GU may be machine-draft flagged for review)
- [ ] Zero hardcoded colors/type/spacing — tokens only (`10-DESIGN-SYSTEM.md` §8)
- [ ] Core components reused (price card / chips / trace line / stat tile), not re-implemented
- [ ] Same analytics event names + properties fire on both
- [ ] Touch targets ≥48dp/44pt (≥56 farmer primaries); farmer laws F1–F10 / buyer laws B1–B7 pass
- [ ] fontScale 2.0 (Android) and `.accessibility3` (iOS) screenshot included — no truncated money digits
- [ ] TalkBack + VoiceOver pass on the screen's happy path
- [ ] Side-by-side screenshot grid attached and visually matched
- [ ] Golden-rule check (§4) passes

## 4. Definition of Done (every unit, every repo)

1. **Golden-rule check:** (a) names which of the two metrics it serves — hours-to-door or farmer-share — or documents why it's neutral infrastructure (Golden Rule #5); (b) region-agnostic: no hardcoded city/currency/`+91`/"mandi" outside region config and localized UI strings — enforced by a CI grep (Golden Rule #2); (c) parity checklist passed for mobile units (Golden Rule #1); (d) adds no owned-asset assumptions (Golden Rule #4).
2. Acceptance criteria of the owning PRD section pass, demonstrated (screen recording or curl transcript in the PR).
3. Tests: backend units ship with supertest coverage of new endpoints; the smoke suite (§7.1) stays green in CI.
4. Error and empty states implemented — not just the happy path.
5. Server-side validation for anything money-related (totals computed server-side, never trusted from clients — `06-PRD-BACKEND.md`).
6. No secrets, keys or `.env` values in the diff.
7. Docs touched if behavior diverged from the PRD: PRD updated in `KisanSetu-Brain` **in the same week**, with the change note. The Brain never goes stale — it is the source of truth, so it must remain true.

## 5. Git & repo conventions (the 5 repos)

| Repo | Contents | Deploy target |
|---|---|---|
| `KisanSetu-Brain` | This corpus — PRDs, design board, decisions. | — (docs) |
| `KisanSetu-Backend` | Node 20 + Express API, SQL migrations, seed, smoke tests, **and the KisanSetu Console** (ops + CMS/analytics modules — one static vanilla-JS app served by the same Express instance at `/ops`) | API host |
| `KisanSetu-Website` | Marketing site, vanilla JS, no build step | Static hosting |
| `KisanSetu-Android` | Kotlin + Jetpack Compose + Material3 | Play Console |
| `KisanSetu-iOS` | Swift 5 + SwiftUI, XcodeGen (`project.yml` in repo, `.xcodeproj` gitignored) | App Store Connect |

*Decision:* the Console (both modules) lives inside `KisanSetu-Backend` — same origin (no CORS), one deploy, and it versions in lock-step with the API it administers. A sixth repo would add ceremony for a one-person team.

Conventions (all repos):

- **Trunk-based:** `main` is protected and always deployable/buildable; short-lived branches `feat/<slug>`, `fix/<slug>`, `chore/<slug>`; merge via PR even solo (the PR is where checklists, screenshots and DoD live — it's the audit trail, not bureaucracy). Squash-merge; branch deleted after.
- **Conventional commits:** `feat: farmer home price cards`, `fix(orders): compute totals from price_feed date of delivery`. Scope names match PRD section slugs where possible.
- **Versioning:** apps use marketing version `0.MINOR.PATCH` — **Android `versionName` and iOS `MARKETING_VERSION` always identical** (parity extends to version numbers); build numbers auto-increment. Backend tags `api-v0.x.y` on deploy; migrations are append-only numbered SQL (`migrations/00N_*.sql`), never edited after merge.
- **CI (GitHub Actions), minimum viable:** Backend — lint + migrations on fresh Postgres + smoke suite. Android — `assembleDebug` + unit tests. iOS — `xcodegen && xcodebuild build` on macOS runner. Website — HTML validation, i18n key-completeness check (every `data-i18n` key exists in every language), JS-size gate (<100KB), hardcoded-region grep.
- **Secrets:** `.env` gitignored, `.env.example` committed and current; production secrets live only on the host. Dev OTP `123456` via `DEV_OTP` env, absent in staging/prod builds (enforced: server refuses to boot with `DEV_OTP` set when `NODE_ENV=production`).

## 6. Environments

| | dev (laptop) | staging | prod |
|---|---|---|---|
| API | `http://localhost:4000` (Android emulator: `10.0.2.2:4000`) | `https://staging-api.{domain}` | `https://api.{domain}` |
| Postgres | Docker, port **5433** | managed/containerized on same VM, nightly dump | managed Postgres or VM + verified daily backups |
| OTP | `DEV_OTP=123456` stub | real SMS gateway, test sender | real SMS (MSG91/Twilio per region) |
| Payments | stubbed payout records | Razorpay **test** keys | Razorpay live (after KYC) |
| WhatsApp | logged to console | Meta sandbox number | approved WABA number |
| Data | seed script (2 crops, 5 farmers, 3 buyers, 7 days of price_feed) | seed + QA data, reset weekly | real; no test users |
| Website | `python3 -m http.server` / any static server | preview deploys per PR (static host feature) | production domain |

Staging = **one cheap VM (≤₹1,000/mo) running docker-compose (api + postgres + caddy)** — exists from week 3 (first moment two clients need a shared API). Production infrastructure is **not** provisioned until the pilot needs real buyers on it (deployment posture: don't pay for cloud before customers — `11-ARCHITECTURE.md`); the staging VM is promoted or cloned in week 9. App builds are pointed at environments via build flavors (Android `dev/staging/prod` product flavors; iOS xcconfig per configuration) — never a hand-edited base URL.

## 7. QA strategy

### 7.1 Automated smoke suite (backend, runs in CI on every PR)

Supertest, five core flows end-to-end against a fresh migrated DB — these mirror the pilot's real day:

1. **Auth:** request OTP → verify → JWT → role-guarded route accepts/denies correctly.
2. **Listing lifecycle:** farmer posts → ops grades (A, weighed qty) → status transitions legal-only (`posted→graded→allocated→closed`; illegal jumps 409).
3. **Order pricing:** buyer places order → totals computed server-side from that day's `price_feed` — client-sent prices ignored.
4. **Allocation & traceability:** ops allocates listing→order_item → buyer's order detail returns farm, village, harvest_ts chain.
5. **Payout + leads:** pickup triggers farmer payout record (stub) with UTR field; `POST /leads` validates, dedupes, rate-limits.

Target runtime <60s so it's never skipped.

### 7.2 Manual release scripts (executed on both platforms before every release tag)

- **Farmer script (12 steps, run in Hindi UI):** fresh install → OTP login → role pick → see today's prices with struck-through comparison → create listing via picture grid + stepper → airplane-mode mid-submit (queues, banner shown) → reconnect (auto-sends) → listing appears `posted` → ops grades it on dashboard → chip flips to `ग्रेड A` with push received → payment record visible with UTR → logout/login state intact.
- **Buyer script (10 steps, EN UI):** login → catalog shows A/B prices vs market → add both grades to cart → delivery date defaults tomorrow, can't pick today → place order → totals match price_feed math by hand-check → ops advances statuses → timeline updates (push each step) → order detail shows full trace line with real farmer name → invoice number present.
- Run on the device matrix: **Android** — one low-RAM device (2GB, Android 10–11, Redmi/realme class; this is the actual farmer device) + Pixel 6a (Android 14); **iOS** — iPhone SE (smallest screen) + current iPhone. Results logged in a per-release checklist file in the repo.

### 7.3 Field testing (the real QA)

- **Farmers: 3 real farmers at the FPO hub** (T4.8 protocol), weeks 7 and 9: hand them the app on *their* phone, in sunlight, and watch silently for 3 minutes per task — no helping, no hovering. Record: where they hesitate >5s, what they tap wrongly, which words they read aloud with confusion. Fix the top 3 confusions before the next build; re-test with at least 1 of the same farmers. Success bar: a first-time farmer posts a listing unassisted in <90 seconds.
- **Buyers: 2 pilot kitchens**, week 8: the manager places a real trial order on their own phone during evening prep chaos. Success bar: order placed in <3 minutes without a phone call to us.
- Field findings are filed as issues labeled `field`; `field` + farmer-flow issues are automatically P1 (§7.4).

### 7.4 Bug triage

**P0** — money wrong, data loss, crash on core path, security: fix before anything else ships, same day. **P1** — core flow degraded, parity break, field-test confusion: fix within the week. **P2** — everything else: backlog, reviewed weekly. Crash reporting (Crashlytics or Sentry — pick one in W5, same product on both apps) from the first internal build; bar: crash-free sessions ≥99.5% before pilot.

## 8. Release process

- **Cadence:** weekly release train from week 5 — whatever units are done ship to internal tracks every Friday; nothing waits for a "big" release.
- **Android:** Play Console **internal testing track** (founder + FPO field staff + ops hire, ≤15 testers) from W5 → **closed track** for pilot farmers at W9. Play Console account created **W1** (review/verification lead time).
- **iOS:** **TestFlight internal** (founder + team) from W5 → **external group** "Pilot kitchens" (~5 buyers) at W8 — external TestFlight needs Beta App Review, so the W8 build is submitted with 3 days' slack. App Store Connect account + agreements done W1.
- **Release checklist (both apps, same version number):** CI green · smoke suite green · manual scripts §7.2 passed on device matrix · parity screenshot grids for units in the release · version bumped identically in both repos · release notes written EN+HI (farmers get HI notes) · crash rate of previous build reviewed · tag `v0.x.y` pushed in both repos.
- **Backend:** deploy = git pull + migrate + restart behind Caddy on staging; promote to prod via the same tagged commit. Migrations run before app deploy; every migration reversible or explicitly marked destructive with a backup taken first.
- **Website:** deploy on merge to `main` (static host auto-deploy); rollback = redeploy previous commit.
- **Public store launch is NOT in the MVP:** the pilot runs entirely on internal/closed tracks + TestFlight. Production store listing is a `13-LAUNCH-PLAN.md` event after the pilot proves retention.

## 9. MVP timeline — 8–10 weeks, weekly milestones

Assumes: founder full-time (sales in mornings, code after), contract designer part-time W1–4, ops hire joining ~W6 (month 2 per `14-OPS-PLAYBOOK.md`). Weeks 9–10 are the buffer that makes 8 honest.

| Week | Milestone (exit criteria) |
|---|---|
| **W1** | Scaffold gap-review vs PRDs done; repos + CI skeletons up; **all third-party applications filed** (Play Console, Apple Developer, Razorpay KYC, WhatsApp Business verification, SMS DLT registration); **website v1 live on domain** with `/leads` stub captured to DB; backend: auth + users + migrations green |
| **W2** | Backend: produce, price_feed, listings lifecycle done with smoke tests 1–2 green; ops dashboard wireframes (T1.5) approved |
| **W3** | Backend: orders + allocations + payments stub + leads + `app_events` storage (S4 amendment — events stored, not just logged, so C-4 has history from the first app build; `19-PRD-CMS-ANALYTICS.md`); **API contract frozen** (breaking changes now require PRD amendment); smoke suite 1–5 green; staging VM live |
| **W4** | **Ops dashboard usable end-to-end:** daily price setting, grading queue, allocation, delivery status, payout trigger — a full fake order-day run-through by founder on staging. *Gate A: the business could run on ops + WhatsApp alone from this point* |
| **W5** | Mobile units: app shells, design-system components (4 core components both platforms), OTP login + role pick shipped to internal tracks (first Play internal + TestFlight builds); crash reporting live; Console CMS/Analytics module v1 usable on staging (C-1 overview, C-2 farmers, C-3 buyers, C-4 basics per `19-PRD-CMS-ANALYTICS.md`) |
| **W6** | Farmer flow: Home (price cards) + New Listing (picture grid, stepper, offline queue) both platforms, parity-checked |
| **W7** | Farmer flow complete: My Listings (chips) + Payments; **field test #1 with 3 farmers at hub** → top-3 fixes; buyer Catalog + Cart underway |
| **W8** | Buyer flow complete: Catalog, Cart, Orders timeline, Order Detail with trace line; buyer field test with 2 kitchens; TestFlight external submitted; **code freeze for integrations** |
| **W9** | Integrations: Razorpay payout (test→live pending KYC), real SMS OTP, FCM/APNs pushes, WhatsApp order confirmations + price broadcast; website `/leads` → prod API + analytics verified; prod environment promoted; farmer field test #2 |
| **W10** | Hardening + **pilot dry run (T6.7): 5 orders end-to-end** — real crates, real hub, staged buyers, full manual fallback rehearsed; P0/P1 zero; release candidates tagged. *Gate B: go/no-go for pilot week 1 (`13-LAUNCH-PLAN.md`)* |

If W8 slips >1 week: cut scope, not parity — drop WhatsApp broadcast and invoice PDFs from MVP before ever shipping one platform ahead of the other.

## 10. Risks & dependencies (development-specific; business risks in `16-RISKS-MITIGATIONS.md`)

| Risk | Mitigation |
|---|---|
| **Third-party approval clocks:** India SMS DLT registration (2–4 wks), WhatsApp Business verification (1–3 wks), Razorpay KYC, Apple review | All filed W1 (§9); every integration has a manual fallback (dev OTP on staging, ops-triggered UPI from founder's phone, plain WhatsApp from handset) |
| Solo-dev bus factor & burnout (founder is also doing 5 sales visits/day) | Weekly release train keeps WIP small; W9–10 buffer; ops hire absorbs hub coordination from W6; scope cuts pre-agreed (§9) |
| iOS toolchain friction (XcodeGen, signing, macOS CI) | `project.yml` committed, signing documented in repo README W5, not discovered at W8 |
| Parity drift under time pressure | Parity checklist is structural (one unit, two PRs, cross-linked) — there is no tracker state that represents "half-shipped" |
| Scaffold code quality unknown | W1 gap review with keep/rewrite verdict per module, logged in `15-TASKS-BACKLOG.md` |
| Low-end Android performance (2GB devices) | Device matrix includes one from W5, not at the end; Compose baseline profiles if needed |

## 11. Development metrics (reviewed Fridays with the release train)

Units completed vs planned · parity defects caught in review (target: caught in review, zero in field) · smoke-suite status streak · crash-free sessions per build · field-test task success (farmer <90s listing, buyer <3min order) · P0/P1 open count (must be 0 at each gate).

## 12. Open questions

| # | Question | Decision taken here / owner | Deadline |
|---|---|---|---|
| Q1 | Ops dashboard: plain JS vs React (T3.5 left it open) | **Decided: plain vanilla JS** — same skills as website, no build step, ~10 screens of tables and forms doesn't justify a framework; revisit only if it exceeds ~3k lines | — |
| Q2 | Crash reporting: Crashlytics vs Sentry | Pick W5; leaning Sentry (works identically on both platforms + backend, self-hostable later). Owner: Alpesh | W5 |
| Q3 | GU strings at MVP or post-pilot | **Decided: keys + machine-draft GU shipped behind review flag; human-reviewed GU before pilot week 1** (Surat farmers are Gujarati-first). Owner: Alpesh + FPO staff reviewer | W9 |
| Q4 | macOS CI runner cost for iOS | GitHub-hosted macOS minutes for MVP; local `xcodebuild` acceptable fallback for a solo dev. Owner: Alpesh | W1 |
| Q5 | Ops hire timing vs W6 assumption | If hire slips, W7 field tests move to W8 and pilot dry-run risk rises — flag in weekly review. Owner: Alpesh | W5 |
