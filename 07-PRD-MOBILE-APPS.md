# 07 — PRD: Mobile Apps (Android + iOS, one spec)

_The single PRD governing BOTH mobile apps. There is no separate Android or iOS PRD and there never will be._
_Siblings: [00-GOLDEN-RULES.md](00-GOLDEN-RULES.md) · [05-PRODUCT-OVERVIEW.md](05-PRODUCT-OVERVIEW.md) · [06-PRD-BACKEND.md](06-PRD-BACKEND.md) · [10-DESIGN-SYSTEM.md](10-DESIGN-SYSTEM.md) · [11-ARCHITECTURE.md](11-ARCHITECTURE.md) · [12-DEVELOPMENT-PLAN.md](12-DEVELOPMENT-PLAN.md)_

---

## 1. Purpose

One dual-role mobile app, shipped on two platforms, that lets:

- a **smallholder farmer** see tomorrow's price tonight, list produce in under 2 minutes with almost no typing, track grading, and see every rupee paid to him with a UTR; and
- a **HoReCa buyer** order graded produce for next-day delivery in under 1 minute and see farm-to-fork traceability on every order.

Both flows exist to move the two metrics: **median harvest→door hours (<36h)** and **farmer share of final price (≥60%)**. The farmer flow removes listing friction (more supply, earlier, so grading and dispatch happen faster); the buyer flow removes ordering friction and makes traceability — the proof of both metrics — visible.

## 2. The parity contract (Golden Rule 1)

This section is a **contract**, not guidance. Violations are release blockers.

1. **One PRD.** This document specifies every screen once. Android (Kotlin + Jetpack Compose + Material3) and iOS (Swift 5 + SwiftUI, XcodeGen) are two renderings of the same spec.
2. **Screen-for-screen, feature-for-feature, design-identical.** Same screens, same navigation graph, same states, same copy, same design tokens ([10-DESIGN-SYSTEM.md](10-DESIGN-SYSTEM.md)).
3. **A change to one platform is incomplete until mirrored on the other.** No ticket is "done" — and no release ships — with a one-platform change. CI/release checklist includes a parity sign-off per screen ID.
4. **One string table.** All UI copy lives in a single master string table (`strings/master.csv`: `key, en, hi, gu`) checked into a shared repo; Android `strings.xml` and iOS `Localizable.strings` are **generated** from it, never hand-edited. Copy divergence is therefore structurally impossible.
5. **Same version, same train.** Both apps share the version number (e.g. `1.4.0`) and release together. If one store review blocks, the other release waits (exception: emergency crash hotfix, which must reach the other platform within 72h).
6. **Permitted divergence — platform conventions only:** system back gesture vs. navigation-bar back, system share sheets, date pickers, keyboard behaviors, Material3 ripple vs. iOS highlight, system fonts (Roboto/Noto vs. SF Pro), FCM vs. APNs plumbing. Anything user-visible beyond these requires a written exception in this PRD first.
7. **One QA pass, two builds.** The acceptance criteria in §7 are executed against both platforms from the same checklist, every release.

## 3. Users & jobs-to-be-done

Personas in full: [05-PRODUCT-OVERVIEW.md](05-PRODUCT-OVERVIEW.md) §3.

| User | Device reality | Jobs-to-be-done in this app |
|---|---|---|
| Farmer (Ramesh) | ₹8–12k Android, patchy 4G, low digital literacy, Hindi/Gujarati | Know tomorrow's price tonight · list produce fast · see grading result · see money received with proof |
| Buyer (Arjun) | iPhone or mid Android, professional English | Order graded produce for tomorrow · track order status · get invoice number · show traceability |

Design constraints that follow (farmer flow, both platforms): max 2 actions per screen; one primary button; touch targets ≥48dp; body text ≥18sp; icon + text always together; picture grids not dropdowns; bottom nav ≤3 tabs; no hamburger menu; every money figure shows the comparison (reference market price struck through, KisanSetu price in leaf green).

## 4. Scope

### 4.1 In scope (MVP)

- Phone OTP auth, role pick at signup (farmer/buyer), JWT session.
- Farmer: Home (prices + sell CTA), New Listing, My Listings, Payments.
- Buyer: Catalog, Cart, Orders, Order Detail with traceability.
- Shared: Profile (language switch, support call, logout).
- i18n EN + HI (string table architecture supports pluggable languages; GU strings land in v1.1).
- Read-path caching for offline viewing; instrumentation events.

### 4.2 NON-GOALS (explicit, with reasons)

| We will NOT build | Why not |
|---|---|
| **Chat / in-app messaging** | WhatsApp Business already carries all farmer–ops and buyer–ops conversation; in-app chat adds moderation, storage and support load and serves neither of the two metrics. |
| **Farmer-to-farmer social features** (feeds, likes, forums) | Network-effects cold-start trap explicitly rejected in the founding analysis; we are B2B infrastructure, not a community app. |
| **B2C purchasing** (households) | Golden Rule 6 — B2B sales-led. Household delivery is a different logistics/margin game; it killed Otipy, Fraazo, ReshaMandi. |
| **In-app payments at MVP** | Farmer payouts are triggered server-side by ops (UPI via abstracted provider; Razorpay = India instance). Buyers pay on invoice per B2B terms. In-app collection adds PSP integration, app-store payment review friction, and refund flows before the trade itself is validated. |
| **Price negotiation / bidding** | The fixed daily published price IS the trust product, for both sides. |
| **Route/driver tracking, maps** | Asset-light (Golden Rule 4): 3PL owns routes; we show status timestamps only. |
| **Advisory / agronomy content at MVP** | DeHaat/AgroStar's lane; revisit at v2 only if it demonstrably improves supply reliability. |
| **Buyer analytics dashboards at MVP** | Premium-subscription feature, post-pilot ([03-BUSINESS-MODEL.md](03-BUSINESS-MODEL.md)). |
| **Web checkout inside the app / webviews for core flows** | Native only; webviews break the low-end-device performance budget. |

## 5. Global conventions (apply to every screen)

### 5.1 Platform & performance floor

| Item | Value |
|---|---|
| Android minimum | Android 8.0 (API 26), 2 GB RAM devices must be usable |
| iOS minimum | iOS 16 |
| Cold start | < 2.5 s to interactive Home on a 2 GB RAM Android |
| App size | ≤ 25 MB Android APK download, ≤ 40 MB iOS |
| API base | `https://api.<domain>/` (dev: `http://localhost:4000`, Android emulator `10.0.2.2:4000`) |

### 5.2 Auth & session

- JWT bearer from `POST /auth/otp/verify`; stored in EncryptedSharedPreferences (Android) / Keychain (iOS). Token TTL 30 days.
- Any `401` → clear session → Login screen (phone number prefilled from local storage).
- Role comes from the JWT/user object and selects the entire post-login experience. **One account = one role at MVP**; changing role = support call (prevents farmers accidentally landing in a B2B ordering flow).

### 5.3 i18n, currency, time

- Every string via the string table; **no hardcoded copy**. Launch languages EN, HI; GU in v1.1; table supports unlimited languages (Golden Rule 2).
- Farmer-facing screens render Hindi first with English subtitle (two-line pattern); buyer-facing screens default to English with local produce names alongside (`name_hi`/`name_gu` from the produce catalog).
- Money: server sends `amount` + `currency` (ISO 4217); client formats per locale (₹ with Indian digit grouping for INR). Clients never compute money — totals, estimates and payouts come from the server ([06-PRD-BACKEND.md](06-PRD-BACKEND.md)).
- Timestamps: API is ISO-8601 UTC; display in device timezone. Freshness always in **hours**, never adjectives.
- Hindi/Gujarati text gets +10–15% line-height over Latin ([10-DESIGN-SYSTEM.md](10-DESIGN-SYSTEM.md)).

### 5.4 Standard states (every screen implements all five)

| State | Behavior |
|---|---|
| **Loading** | Skeleton placeholders matching final layout (no spinners on full screens; spinner allowed inside buttons). |
| **Empty** | Illustration + one-line explanation + one action (copy per screen below). |
| **Error** | Inline card: `common.error_generic` + Retry button. Errors never use raw server text or stack traces. |
| **Offline** | Persistent top banner `common.offline_banner`; read screens show last cached data with an "as of {time}" stamp; write actions disabled with explanatory toast (MVP) or queued (v1.1, farmer listing only). |
| **Stale price** | If today's price feed is absent, show the last published prices visually muted + `home.prices_stale` label. |

Shared copy:

| key | EN | HI |
|---|---|---|
| `common.retry` | Try again | फिर से कोशिश करें |
| `common.error_generic` | Something went wrong. Try again. | कुछ गड़बड़ हो गई। फिर से कोशिश करें। |
| `common.offline_banner` | You're offline. Showing saved info. | इंटरनेट नहीं है। पुरानी जानकारी दिख रही है। |
| `common.asof` | As of {time} | {time} तक की जानकारी |
| `common.cancel` | Cancel | रद्द करें |

### 5.5 Accessibility

- Touch targets ≥48dp everywhere (not only farmer flow).
- Dynamic Type (iOS) / font scale (Android) supported to 200% without clipped text.
- Every interactive element has a TalkBack/VoiceOver label from the string table.
- Contrast AA against cream `#FAF7F0` and forest `#14532D` surfaces.

### 5.6 Navigation model

- **Farmer** bottom nav (3 tabs): `घर / Home` · `मेरी फ़सल / My produce` · `पैसा / Money`. Profile via avatar icon top-right of Home.
- **Buyer** bottom nav (3 tabs): `Catalog` · `Orders` · `Profile`. Cart via badge icon in the Catalog top bar.
- No hamburger menus anywhere.

---

## 6. Screen specifications

Screen IDs are shared vocabulary across both codebases, QA and design. Flow:

```
S-01 Splash → S-02 Language → S-03 Login → S-04 OTP → (new user) S-05 Role pick → role home
                                            └→ (existing user) F-01 or B-01
```

**S-06 Update required** is not part of this linear flow — it is a guard: raised at launch (from S-01) when the build is below `min_supported_version`, or over any screen the moment an API returns `426`.

### S-01 Splash

- **Purpose:** brand beat + session check. **Max 1.5 s.**
- **Layout:** forest `#14532D` full screen, wordmark centered, leaf accent.
- **Logic:** valid token → route by role to F-01/B-01; no token → S-02 (first run) or S-03 (returning device). No network call blocks this screen.
- **Acceptance:** cold start to next screen <2.5 s on floor device; kill-and-reopen preserves session.

### S-02 Language select (onboarding)

- **Purpose:** language before anything else — the farmer must never see an English wall.
- **Layout:** title + 2 large option cards (EN, HI; GU card added v1.1), each showing its own native label; primary button.
- **Copy:**

| key | EN | HI |
|---|---|---|
| `lang.title` | Choose your language | अपनी भाषा चुनें |
| `lang.en` | English | English |
| `lang.hi` | हिन्दी | हिन्दी |
| `lang.continue` | Continue | आगे बढ़ें |

- **Data/API:** none (stored locally; synced to `PATCH /users/me {language}` after login).
- **States:** none beyond default (fully offline-capable).
- **Edge cases:** device language HI → preselect HI card.
- **Acceptance:** changing selection instantly re-renders the screen's own copy; choice persists across reinstall-free restarts.

### S-03 Login (phone)

- **Purpose:** phone number capture, E.164 normalization.
- **Layout:** title, one phone field with country-code prefix (default from device region/SIM; `+91` in India — **not hardcoded**), primary button. Number keyboard auto-focused.
- **Copy:**

| key | EN | HI |
|---|---|---|
| `login.title` | Your mobile number | अपना मोबाइल नंबर |
| `login.subtitle` | We'll send a code by SMS | हम SMS से कोड भेजेंगे |
| `login.cta` | Get code | कोड भेजें |
| `login.err_invalid_phone` | Enter a valid mobile number | सही मोबाइल नंबर डालें |

- **Data/API:** `POST /auth/otp/request {phone}` (E.164). Button shows inline spinner while pending.
- **States:** error → inline field error; offline → button disabled + banner; rate-limited (429) → `otp.err_locked`.
- **Edge cases:** leading zeros / spaces stripped; paste of full `+91…` accepted; double-tap debounced.
- **Acceptance:** invalid numbers never reach the API; happy path lands on S-04 with the phone shown.

### S-04 OTP verify

- **Purpose:** verify the 6-digit code, mint session.
- **Layout:** title, 6 individual digit boxes (auto-advance), sent-to line with "edit" link back to S-03, resend link with 30 s countdown, primary button (auto-submits on 6th digit).
- **Copy:**

| key | EN | HI |
|---|---|---|
| `otp.title` | Enter the 6-digit code | 6 अंकों का कोड डालें |
| `otp.sent_to` | Sent to {phone} | {phone} पर भेजा गया |
| `otp.cta` | Confirm | आगे बढ़ें |
| `otp.resend` | Resend code | कोड दोबारा भेजें |
| `otp.resend_wait` | Resend in {s}s | {s} सेकंड में दोबारा भेजें |
| `otp.err_wrong` | Wrong code. Try again. | ग़लत कोड। फिर से डालें। |
| `otp.err_locked` | Too many tries. Wait 1 hour. | बहुत कोशिशें हो गईं। 1 घंटे बाद कोशिश करें। |

- **Data/API:** `POST /auth/otp/verify {phone, otp}`. Existing user → `200 {token, user}` (route to role home). New (unregistered) phone → `200 {signup_required: true, signup_token}` — verification succeeded but **no account exists yet and none is created here**; the client carries `signup_token` forward to S-05 and completes registration there via `POST /auth/signup`. The client never re-calls verify with role/name. 5 wrong attempts → server locks that **phone for 1 hour** (`otp.err_locked`). Exact contract in [06-PRD-BACKEND.md](06-PRD-BACKEND.md).
- **Android extra (permitted divergence):** SMS Retriever API auto-fills the code. iOS uses the system OTP AutoFill from Messages.
- **States:** wrong code shakes boxes + error; locked state disables input with countdown.
- **Edge cases:** dev OTP `123456` in dev builds only — release builds must strip it; app backgrounded during entry keeps state 5 min.
- **Acceptance:** OTP autofill works on both platforms; existing user reaches role home in ≤2 taps from S-03.

### S-05 Role pick (+ per-role basics; new users only)

- **Purpose:** the fork that defines the account. One decision, clearly irreversible in-app.
- **Layout:** title + two large illustrated cards (farmer: field illustration; buyer: kitchen/restaurant illustration). Selecting a card reveals its basics form beneath: **farmer** → name (optional); **buyer** → business name (required), business type picker (restaurant / hotel / caterer / cloud kitchen / other), delivery address (required). Primary button.
- **Copy:**

| key | EN | HI |
|---|---|---|
| `role.title` | Who are you? | आप कौन हैं? |
| `role.farmer` | I am a farmer — I want to sell produce | मैं किसान हूँ — फ़सल बेचनी है |
| `role.buyer` | I buy for a business — restaurant, hotel, caterer | मैं व्यापार के लिए ख़रीदता हूँ — रेस्टोरेंट, होटल, कैटरर |
| `role.note` | You can't change this later. Call us if you picked wrong. | इसे बाद में बदला नहीं जा सकता। ग़लत चुना हो तो हमें फ़ोन करें। |
| `role.name_label` | Your name (optional) | आपका नाम (ज़रूरी नहीं) |
| `role.biz_name` | Business name | व्यापार का नाम |
| `role.biz_type` | Business type | व्यापार का प्रकार |
| `role.biz_address` | Delivery address | डिलीवरी का पता |
| `role.cta` | Start | शुरू करें |

- **Data/API:** registers the account via `POST /auth/signup {signup_token, role, name?}` → `{token, user}`, using the `signup_token` minted by S-04's verify (no OTP is re-sent or re-verified on this screen). For buyers, business basics are saved immediately after — with the new session — via `PATCH /buyer-profiles/me {business_name, business_type, address}` ([06-PRD-BACKEND.md](06-PRD-BACKEND.md)).
- **States:** submit failure keeps entered data; `signup_token` expired/invalid (or signup returns 4xx for it) → route back to S-04 to re-verify (re-request OTP), with the picked role and entered name preserved locally so the farmer/buyer re-enters nothing.
- **Edge cases:** buyer accounts start **pending activation** (`users.status = pending`; ops vets B2B accounts) — buyer lands on B-01 with the pending banner (see B-01); farmer accounts are active immediately, FPO/village/hub linkage is done later by ops (see [09](09-PRD-OPS-DASHBOARD.md)/[14](14-OPS-PLAYBOOK.md)) — until linked, a farmer cannot list (see F-01/F-02 "not linked to a hub").
- **Acceptance:** farmer can complete signup with zero typing (skip name); buyer cannot proceed without business name + address; role is absent from every subsequent settings screen.

---

### S-06 Update required (force update / soft nudge)

- **Purpose:** keep every live client above the server's `min_supported_version` so backend/app contract drift can never strand a user on a broken build. **Identical on Android and iOS** — same trigger, same states, same copy, same store-redirect behavior (parity contract §2; only the store target differs — Play Store vs. App Store — which is a permitted platform-convention divergence).
- **Trigger:** two entry points — (1) **at app launch** (post-splash, non-blocking on S-01) the client compares its build version to the cached `min_supported_version` for its platform; (2) **any API response `426` / `{error: "upgrade_required"}`** from any screen immediately routes here. Both resolve to the same screen.
- **Layout:** full-screen blocking view (no bottom nav, no back/dismiss) — illustration, `update.title`, `update.body`, one primary **Update now** button that deep-links to the platform store listing (`market://` / App Store URL from `GET /config`, with an https store fallback). No "later"/skip control — this state is a hard gate.
- **Soft nudge (non-blocking):** when the client is **exactly one minor version above** `min_supported_version` (i.e. one release from being cut off), show a dismissible top banner `update.soft_nudge` on Home (F-01/B-01) instead of the blocking screen; tapping it opens the store. Dismiss persists for the session only.
- **Copy:**

| key | EN | HI |
|---|---|---|
| `update.title` | Please update KisanSetu | किसानसेतु अपडेट करें |
| `update.body` | This version is no longer supported. Update to keep ordering and selling. | यह वर्ज़न अब नहीं चलेगा। ख़रीदने-बेचने के लिए अपडेट करें। |
| `update.cta` | Update now | अभी अपडेट करें |
| `update.soft_nudge` | A new version is ready — update soon. | नया वर्ज़न आ गया है — जल्द अपडेट करें। |
| `update.store_unavailable` | Couldn't open the store. Search "KisanSetu" in your app store. | स्टोर नहीं खुला। अपने ऐप स्टोर में "KisanSetu" खोजें। |

- **Data/API:** `min_supported_version` (per platform: `min_supported_version_android` / `min_supported_version_ios`) and store URLs come from `GET /config`; the last-fetched values are cached so the launch check and the blocking screen's copy work **offline** (a client already known to be below minimum shows the gate even with no network). A live `426` is authoritative and overrides the cached comparison.
- **States:** blocking (default) with store button; offline blocking → same screen, `update.body` still shown from cache, store button attempts the store deep-link and falls back to `update.store_unavailable` if it can't open; soft-nudge banner variant (non-blocking). This screen deliberately has no loading/empty state — it renders from cached config instantly.
- **Edge cases:** store deep-link failure (no store app / restricted device) → `update.store_unavailable` toast, screen stays; a `426` received mid-flow (e.g. during checkout) discards the in-progress action and routes here — the server rejected it anyway; config that hasn't been fetched yet on a first run treats the client as up-to-date (fail-open) until the first `GET /config`, after which the gate applies.
- **Acceptance:** a build below `min_supported_version` cannot reach any authenticated screen on either platform; the blocking copy renders in the selected language with no network; the soft-nudge appears at exactly one version above minimum and never blocks; store button opens the correct platform store.

---

### F-01 Farmer Home

- **Purpose:** answer "क्या भाव है?" in 3 seconds and put **Sell** one thumb away.
- **Layout (top→bottom):** greeting row (avatar → P-01, `home.greeting`), date; **optional "not linked to a hub yet" banner** (`home.not_linked` + a call-support button; shown only when the farmer has no assigned hub — see States); **price cards list** — one card per beachhead produce: produce photo, local name large + EN subtitle, reference market price struck through (`मंडी ₹28` — "mandi" appears in the HI/India copy only), **farmer price in leaf green, biggest number on the card** (`आपको मिलेगा ₹24/किलो`), A/B prices as forest/amber chips; sticky bottom **फ़सल बेचें / Sell produce** button (full-width, ≥56dp tall).
- **Copy:**

| key | EN | HI |
|---|---|---|
| `home.greeting` | Hello, {name} | नमस्ते, {name} |
| `home.prices_title` | Today's prices | आज के भाव |
| `home.price_market` | Market ₹{x} | मंडी ₹{x} |
| `home.price_you_get` | You get ₹{y}/kg | आपको मिलेगा ₹{y}/किलो |
| `home.grade_a` | Grade A ₹{a} | ग्रेड A ₹{a} |
| `home.grade_b` | Grade B ₹{b} | ग्रेड B ₹{b} |
| `home.sell_cta` | Sell produce | फ़सल बेचें |
| `home.prices_stale` | Yesterday's prices — new prices coming soon | कल के भाव — नए भाव जल्द आएँगे |
| `home.empty` | Prices will appear here soon | भाव जल्द यहाँ दिखेंगे |
| `home.not_linked` | Not linked to a hub yet — call us | हब से जुड़ना बाकी है — हमें फ़ोन करें |

- **Data/API:** `GET /prices/today` → `[{produce_id, name_en, name_hi, name_gu, reference_market_price, farmer_price_a, farmer_price_b, currency, unit, date}]`. `farmer_price_*` is what the farmer receives per kg — the take rate is already netted out server-side, so the green number is always literally true. Hub-linkage status is read from `GET /me` — a farmer with no assigned hub renders the not-linked banner (ops-side hub linking is handled in [09](09-PRD-OPS-DASHBOARD.md)/[14](14-OPS-PLAYBOOK.md)). (Backend note: the scaffold's `mandi_price_per_kg` must become `reference_market_price`, and farmer-facing prices must be added to the feed — requirement logged in [06-PRD-BACKEND.md](06-PRD-BACKEND.md).)
- **States:** loading skeleton of 3 cards; empty → `home.empty`; stale → muted cards + `home.prices_stale`; offline → cached prices + `common.asof` stamp; **not linked** (no hub assigned per `GET /me`) → persistent `home.not_linked` banner with a call-support button (`tel:` from `GET /config`) above the price list — prices still render, but starting a listing surfaces the same block at F-02 submit; Sell stays enabled offline only in v1.1 (queue), disabled with toast at MVP.
- **Edge cases:** greeting without name (name skipped) → "नमस्ते 🙏" form without `{name}`; >6 produce items scroll, Sell button stays sticky.
- **Acceptance:** price comparison (struck market price + green farmer price) visible without scrolling for the first 2 produce items on a 5" screen; Sell reachable with one thumb; TalkBack reads "Tomato. Market price 28 rupees, struck out. You get 24 rupees per kilo."

### F-02 New Listing (3-step flow, one screen sequence)

- **Purpose:** post produce for tomorrow's hub intake in <60 s with ~zero typing.
- **Layout:** progress dots (3). **Step 1 — picture grid** of produce (2 columns, photo + local name, ≥120dp cells; only produce with published prices at top). **Step 2 — quantity:** giant number readout, stepper buttons −/+ (10 kg increments, long-press accelerates; direct numeric entry available but not required) + harvest date as two large toggle chips: `आज / Today`, `कल / Tomorrow` (**default Tomorrow**). **Step 3 — confirm card:** produce photo, qty, harvest date, **estimated payout range** in leaf green, note, primary button.
- **Copy:**

| key | EN | HI |
|---|---|---|
| `listing.pick_title` | What are you selling? | क्या बेचना है? |
| `listing.qty_title` | How much? | कितना है? |
| `listing.qty_unit` | kg | किलो |
| `listing.harvest_title` | Harvest date | फ़सल कब कटेगी? |
| `listing.harvest_today` | Today | आज |
| `listing.harvest_tomorrow` | Tomorrow | कल |
| `listing.est_payout` | Estimated payout ₹{low}–₹{high} | अनुमानित पैसा ₹{low}–₹{high} |
| `listing.est_note` | Final amount after weighing and grading at the hub | हब पर तौल और ग्रेड के बाद पक्की रक़म |
| `listing.submit` | Confirm | पक्का करें |
| `listing.success_title` | Done ✓ | हो गया ✓ |
| `listing.success_body` | Bring your produce to the {hub} hub tomorrow morning | कल सुबह फ़सल {hub} हब पर ले आएँ |
| `listing.err_no_price` | Today's price isn't out yet — you'll see the final rate after grading | आज का भाव अभी नहीं आया — पक्की रक़म ग्रेड के बाद दिखेगी |
| `listing.err_no_hub` | Not linked to a hub yet — call us | हब से जुड़ना बाकी है — हमें फ़ोन करें |
| `listing.err_no_hub_cta` | Call us | हमें फ़ोन करें |

- **Data/API:** produce grid from `GET /produce` (+ cached photos bundled for beachhead crops); estimate computed **server-side** via `GET /listings/estimate?produce_id&qty_kg` → `{low, high, currency}` (range = qty × farmer_price_b … qty × farmer_price_a); submit `POST /listings {produce_id, qty_kg, harvest_date}` → 201. A farmer who self-signed up outside a demo day has no hub assigned, so the submit can return **`409 profile_incomplete`** — the account exists but isn't yet linked to a hub (ops-side linking handled in [09](09-PRD-OPS-DASHBOARD.md)/[14](14-OPS-PLAYBOOK.md)).
- **States:** estimate loading → shimmer on the number only; no price today → `listing.err_no_price` replaces the estimate, submit still allowed; **not linked to a hub** (`409 profile_incomplete` on submit) → blocking error card on step 3 with `listing.err_no_hub` + a `listing.err_no_hub_cta` call-support button (`tel:` from `GET /config`); entered produce/qty/date preserved, no retry-submit (calling ops is the only unblock); submit error (other) → stays on step 3 with retry, data intact; offline at submit → MVP blocks with toast, v1.1 queues (see §8) and shows success variant "will send when online / नेट आने पर भेज देंगे".
- **Edge cases:** qty bounds 5–2,000 kg (server-validated too); double-tap submit debounced (client idempotency key `client_ref` on the POST); back from step 3 preserves steps 1–2; success screen auto-returns to F-01 after 3 s or on tap.
- **Acceptance:** entire flow with stepper only (no keyboard) is possible; a listing created here appears in F-03 as नई/Posted within one refresh; estimated range always brackets the eventual payout for correct grading (spot-checked weekly against real payouts).

### F-03 My Listings

- **Purpose:** "what happened to my crop?" — status at a glance.
- **Layout:** list of cards, newest first: produce photo + name, qty (posted → graded qty once graded), harvest date, **status chip**. Chips: `नई / New` (neutral), `ग्रेड A / Grade A` (forest), `ग्रेड B / Grade B` (amber), `बिक गई / Sold` (leaf), `बंद / Closed` (grey). Tap card → detail bottom sheet: full timeline (posted → graded → sold), graded qty per grade, linked payout row (→ F-04) when paid.
- **Copy:**

| key | EN | HI |
|---|---|---|
| `listings.title` | My produce | मेरी फ़सल |
| `listings.status_posted` | New | नई |
| `listings.status_graded_a` | Grade A | ग्रेड A |
| `listings.status_graded_b` | Grade B | ग्रेड B |
| `listings.status_allocated` | Sold | बिक गई |
| `listings.status_closed` | Closed | बंद |
| `listings.graded_qty` | Weighed: {qty} kg | तौल: {qty} किलो |
| `listings.empty` | You haven't sold anything yet. Tap "Sell produce". | अभी तक कुछ नहीं बेचा। "फ़सल बेचें" दबाएँ। |

- **Data/API:** `GET /listings` (server scopes to own listings for the farmer role); pull-to-refresh; cached for offline read.
- **States:** skeleton cards; empty state links to F-02; offline shows cache + as-of stamp.
- **Edge cases:** listing partially rejected at grading (posted 80 kg → graded 62 kg) shows both numbers, no apology copy — the hub chart is the referee; statuses only ever move forward.
- **Acceptance:** status chip colors match [10-DESIGN-SYSTEM.md](10-DESIGN-SYSTEM.md) tokens exactly on both platforms; a grading done on the ops dashboard is visible here after pull-to-refresh.

### F-04 Payments

- **Purpose:** the farmer's ledger — every rupee, with proof. This screen is the retention feature.
- **Layout:** header tile: `इस महीने / This month` + big tabular-nums sum in leaf green. List rows: **amount (big, leaf green)**, date, `UPI • UTR {utr}`, status `पैसा आ गया ✓ / Paid ✓`. Tap row → detail sheet: linked listing (produce, qty, grade), payout date, UTR copyable.
- **Copy:**

| key | EN | HI |
|---|---|---|
| `payments.title` | My money | पैसा |
| `payments.month_total` | This month | इस महीने |
| `payments.paid` | Paid ✓ | पैसा आ गया ✓ |
| `payments.utr` | UPI • UTR {utr} | UPI • UTR {utr} |
| `payments.empty` | Your payments will show here | आपका पैसा यहाँ दिखेगा |

- **Data/API:** `GET /payments` (server scopes to payee); each row `{amount, currency, created_at, utr, status, listing_id}`. A farmer payout is tied to the **listing** that was graded and sold (not to a buyer order) — the detail sheet resolves `listing_id` to its produce/qty/grade.
- **States:** skeleton rows; empty; offline cache + as-of.
- **Edge cases:** payout pending (initiated, no UTR yet) → row shows `…` clock chip, no UTR — copy `भेज रहे हैं / Sending…`; multiple payouts same day listed separately (one per listing).
- **Acceptance:** month total equals the sum of visible rows (server-computed, client-verified in QA); UTR copy works on both platforms; VoiceOver reads amounts before dates.

---

### B-01 Buyer Catalog

- **Purpose:** tonight's order in under a minute; prices that prove themselves against the market rate.
- **Layout:** top bar: title + cart icon with count badge; **delivery-context strip**: "Ordering for delivery {date}" (see cutoff logic in B-02); optional pending-activation banner; product cards: produce photo, EN name + local name subtitle, **freshness badge** (`Within 48h of harvest`), market price struck through, per-grade rows — `Grade A` chip (forest) ₹a/kg, `Grade B` chip (amber) ₹b/kg — each with qty stepper (kg, 5 kg increments) and Add. **Availability:** a card whose `available_qty` is zero (`availability: sold_out`) renders muted with a `catalog.sold_out` label and its steppers + Add disabled; a card that is running low (`availability: low`) shows a `catalog.low_stock` chip with the stepper still enabled but capped at `available_qty`.
- **Copy:**

| key | EN | HI |
|---|---|---|
| `catalog.title` | Catalog | कैटलॉग |
| `catalog.fresh_badge` | Within 48h of harvest | कटाई के 48 घंटे के अंदर |
| `catalog.market_price` | Market ₹{x} | बाज़ार भाव ₹{x} |
| `catalog.per_kg` | /kg | /किलो |
| `catalog.add` | Add | जोड़ें |
| `catalog.sold_out` | Sold out today | आज उपलब्ध नहीं |
| `catalog.low_stock` | Only a little left | थोड़ा बचा है |
| `catalog.pending_banner` | Account under review — ordering unlocks after our call. | खाता जाँच में है — हमारे कॉल के बाद ऑर्डर चालू होगा। |
| `catalog.empty` | Today's catalog is being prepared. Check back shortly. | आज का कैटलॉग तैयार हो रहा है। थोड़ी देर में देखें। |

- **Data/API:** `GET /catalog` → the buyer catalog, one entry per available produce: `[{produce_id, name_en, name_hi, name_gu, photo, reference_market_price, platform_price_a, platform_price_b, currency, unit, available_qty, availability}]`, where `availability ∈ available | low | sold_out` is **server-derived** from `available_qty` (thresholds are server-owned). The buyer-pays prices (`platform_price_a/b`) and the market reference come pre-joined, so the client does **not** build the catalog from `/prices/today` + `/produce`. Account state (`users.status`) is read from `GET /me` to decide whether to show the pending-activation banner and disable steppers. Cart is client-local until order placement.
- **States:** skeletons; empty (`catalog.empty` — before the daily catalog is published); offline → cached catalog, Add disabled; **pending activation** (`users.status = pending`) → catalog browsable, steppers disabled, banner shown; **sold-out / low** per card (from `availability`) → sold-out cards muted with steppers disabled, low cards flagged with `catalog.low_stock` and capped at `available_qty`.
- **Edge cases:** produce the server omits from `GET /catalog` isn't shown at all; produce that is present but out of stock shows the `catalog.sold_out` card (never hidden as ₹0); quantities capped at the lesser of 500 kg/line and `available_qty` (server re-validates); buyer language switched to HI renders this screen in HI (parity of i18n across roles).
- **Acceptance:** market-price strike-through + our price render on every purchasable card; a zero-`available_qty` produce shows the sold-out state with its stepper disabled; add-to-cart updates the badge instantly; pending buyers can browse but never reach B-02 checkout.

### B-02 Cart

- **Purpose:** review, set delivery, place order — no payment step.
- **Layout:** line items (produce, grade chip, qty stepper, line total, remove); delivery date field (date picker, min = tomorrow, or day-after if past cutoff), with the delivery-window helper `cart.delivery_window` (window from `config.delivery_window`, e.g. 06:00–10:00); delivery address (prefilled from buyer profile, editable); totals block: Subtotal / Delivery fee / **Total** (all server-quoted; delivery fee shows ₹0 when the order qualifies for the waiver); invoice note; primary button.
- **Copy:**

| key | EN | HI |
|---|---|---|
| `cart.title` | Cart | कार्ट |
| `cart.delivery_date` | Delivery date | डिलीवरी की तारीख़ |
| `cart.delivery_window` | Delivery {start}–{end} | डिलीवरी {start}–{end} |
| `cart.address` | Delivery address | डिलीवरी का पता |
| `cart.subtotal` | Subtotal | जोड़ |
| `cart.delivery_fee` | Delivery fee | डिलीवरी शुल्क |
| `cart.total` | Total | कुल |
| `cart.invoice_note` | Pay on invoice. No payment needed now. | पेमेंट इनवॉइस से। अभी कुछ नहीं देना है। |
| `cart.place_order` | Place order | ऑर्डर करें |
| `cart.cutoff_note` | Orders after {time} are delivered the day after tomorrow. | {time} के बाद के ऑर्डर परसों पहुँचेंगे। |
| `cart.min_order` | Minimum order ₹{v} | कम से कम ऑर्डर ₹{v} |
| `cart.price_changed` | Prices updated since you added items. New total: ₹{t} | भाव बदल गए हैं। नया कुल: ₹{t} |
| `cart.empty` | Your cart is empty. | कार्ट खाली है। |
| `cart.success_title` | Order placed ✓ | ऑर्डर हो गया ✓ |
| `cart.success_body` | We'll confirm shortly. Track it in Orders. | जल्द पक्का करेंगे। Orders में देखें। |

- **Data/API:** totals quoted via `POST /orders/quote {items, delivery_date}` → `{subtotal, delivery_fee, total, currency}` (server-priced; client never multiplies); place via `POST /orders {items:[{produce_id, grade, qty_kg}], delivery_date, delivery_address, client_ref}` → 201 with order. **Pilot params come from `GET /config` and are never hardcoded:** `order_cutoff_local_time` (18:00 — an order placed after cutoff on day D can only take delivery on D+2, i.e. cutoff is 18:00 on D-1 for next-day delivery), `delivery_window` (06:00–10:00), `min_order_value` (₹1,000), `delivery_fee` (₹50) and `delivery_fee_waiver_threshold` (₹4,000, at/above which delivery is free). Any value shown to the buyer is rendered from these config values (example display values: cutoff 18:00, minimum ₹1,000); the server re-validates all of them on quote and place.
- **States:** quote loading → totals shimmer; quote/placement failure → retry keeping cart; offline → place disabled; price drift between add and place → `cart.price_changed` dialog with Accept / Back.
- **Edge cases:** below min order (`config.min_order_value`) → button disabled + `cart.min_order` (₹{v} filled from config, e.g. ₹1,000); past cutoff (`config.order_cutoff_local_time`, e.g. 18:00) → date picker min shifts, `cart.cutoff_note` shows with `{time}` from config; delivery fee waived at/above `config.delivery_fee_waiver_threshold` → quote returns `delivery_fee: 0`; a **pending buyer** whose stale session reaches placement is server-rejected **`403 account_pending`** → the app surfaces it gracefully as the `catalog.pending_banner` message and routes back to B-01 (no raw error, no created order); duplicate submit prevented by `client_ref` idempotency; item availability is best-effort (allocation is ops-side) — order may be confirmed partially, communicated via order status + ops call, not in cart.
- **Acceptance:** placing an order twice rapidly creates exactly one order; totals on the success screen match `GET /orders/:id` exactly; cart survives app kill (persisted locally).

### B-03 Orders

- **Purpose:** every order, its status, at a glance.
- **Layout:** list newest first: order date, invoice no, total, item summary ("Tomato A 25kg +2 more"), **status timeline chip row**: `Placed → Confirmed → Out for delivery → Delivered` with the current step highlighted (cancelled shows a red terminal chip). Tap → B-04.
- **Copy:**

| key | EN | HI |
|---|---|---|
| `orders.title` | Orders | ऑर्डर |
| `orders.status_placed` | Placed | ऑर्डर हुआ |
| `orders.status_confirmed` | Confirmed | पक्का हुआ |
| `orders.status_out` | Out for delivery | रास्ते में |
| `orders.status_delivered` | Delivered | पहुँच गया |
| `orders.status_cancelled` | Cancelled | रद्द |
| `orders.empty` | No orders yet. Browse the catalog. | अभी कोई ऑर्डर नहीं। कैटलॉग देखें। |

- **Data/API:** `GET /orders` (server scopes to the buyer). Pull-to-refresh; 60 s auto-refresh while foregrounded on this screen.
- **States:** skeletons; empty links to B-01; offline cache + as-of.
- **Edge cases:** statuses only move forward (server enforces the state machine); a cancelled order shows reason line if provided by ops.
- **Acceptance:** an ops status change appears within one refresh; timeline renders identically (order, colors, icons) on both platforms.

### B-04 Order Detail — with traceability (the signature screen)

- **Purpose:** invoice-grade detail + the **trace line** — the premium differentiator that justifies the subscription later.
- **Layout (top→bottom):** status timeline (full-width horizontal stepper); order meta: invoice no (copyable), delivery date, address; items table: produce + grade chip, qty, unit price, line total; totals block; **Traceability section** — title, then per item a **vertical trace line** (dots + connecting line, per [10-DESIGN-SYSTEM.md](10-DESIGN-SYSTEM.md)):
  1. `Harvested / कटाई` — farmer name, village, `harvest_ts`
  2. `At hub / हब पर` — hub name, `hub_in_ts`
  3. `Dispatched / रवाना` — `dispatch_ts`
  4. `Delivered / पहुँचा` — `delivered_ts`
  and a freshness stat beneath: **"{h} hours from harvest / कटाई से {h} घंटे"** (computed server-side, shown once delivered).
- **Copy:**

| key | EN | HI |
|---|---|---|
| `detail.invoice` | Invoice {no} | इनवॉइस {no} |
| `detail.items` | Items | सामान |
| `detail.trace_title` | Farm to your door | खेत से आपके दरवाज़े तक |
| `detail.trace_harvested` | Harvested | कटाई |
| `detail.trace_hub` | At hub | हब पर |
| `detail.trace_dispatched` | Dispatched | रवाना |
| `detail.trace_delivered` | Delivered | पहुँचा |
| `detail.trace_farmer` | {farmer}, {village} | {farmer}, {village} |
| `detail.fresh_hours` | {h} hours from harvest | कटाई से {h} घंटे |
| `detail.trace_pending` | Updates as your order moves | ऑर्डर बढ़ने पर अपडेट होगा |

- **Data/API:** `GET /orders/:id` → order + items + allocations, each allocation carrying `{farmer_name, village, harvest_ts, hub_in_ts, dispatch_ts, delivered_ts}` (SLA timestamps per [06-PRD-BACKEND.md](06-PRD-BACKEND.md)). Farmer surname/phone are **not** exposed — first name + village only (privacy, [18-LEGAL-COMPLIANCE.md](18-LEGAL-COMPLIANCE.md)).
- **States:** future trace steps rendered as hollow dots with `detail.trace_pending`; one item sourced from multiple farms → multiple trace lines under that item, each labeled with its share (`{qty} kg`).
- **Edge cases:** pre-confirmation orders show items + totals but the trace section shows only the pending placeholder; missing timestamp (ops lapse) shows the step without a time rather than a fake time — never fabricate freshness.
- **Acceptance:** for a delivered order, all four trace steps show real timestamps in device TZ; freshness hours = `delivered_ts − harvest_ts` rounded to the hour, matching the weekly SLA report; screenshot of this screen is legible enough to be shown to a diner (design QA gate).

---

### P-01 Profile & logout (both roles)

- **Purpose:** identity, language, support, exit. Deliberately thin.
- **Layout:** identity block (name, phone, role label, buyer: business name + address with edit); language row (opens S-02 as a modal); support row (**Call support / हमें फ़ोन करें** — `tel:` link, number from `GET /config`); app version; logout row.
- **Copy:**

| key | EN | HI |
|---|---|---|
| `profile.title` | Profile | प्रोफ़ाइल |
| `profile.language` | Language | भाषा |
| `profile.support` | Call support | हमें फ़ोन करें |
| `profile.logout` | Log out | लॉग आउट |
| `profile.logout_confirm` | Log out of KisanSetu? | किसानसेतु से लॉग आउट करें? |
| `profile.version` | Version {v} | वर्ज़न {v} |

- **Data/API:** `GET /users/me`; `PATCH /users/me {language}`; buyer address edit via `PATCH /buyer-profiles/me`.
- **States/edge cases:** logout clears token, cached data and event queue (flushes first if online), returns to S-03 with phone prefilled; language change re-renders in place, no restart.
- **Acceptance:** language change reflects on every screen without restart; after logout, back button cannot re-enter authenticated screens.

---

## 7. Global acceptance criteria (release gate, both platforms)

1. Every screen implements all five standard states (§5.4) — QA walks each with airplane mode, empty seeds and a 500-stubbed API.
2. All copy from the generated string tables; a `grep` for hardcoded user-facing strings in either codebase returns zero.
3. Farmer flow completable end-to-end (login → list → see payout) with font scale 200% and TalkBack/VoiceOver on.
4. No client-side money math anywhere (code review gate).
5. Parity checklist signed per screen ID: same states, same copy keys, same token colors, same nav position.
6. Crash-free sessions ≥99.5% in the internal-testing week before each release.

## 8. Phased rollout

| Phase | Contents |
|---|---|
| **MVP** | Everything in §6 as specified; EN+HI; read-cache offline; no push (status changes via WhatsApp from ops + pull-to-refresh); events buffered to `POST /events`. |
| **v1.1** | **Gujarati** (`gu` column already in the master table — translate + enable card in S-02). **Offline queue**: `POST /listings` queued on-device (SQLite/Core Data), auto-sent on reconnect, listing cards show `भेज रहे हैं / Sending…` chip; conflict rule: server timestamps win. **Push** (FCM/APNs): farmer `graded` ("आपकी फ़सल ग्रेड हो गई"), `payout_paid` ("पैसा भेज दिया गया ✓"), buyer order-status changes; deep-link to F-03/F-04/B-04. |
| **v2** | **Voice input** for listing qty/produce (speech-to-intent, HI/GU); **advisory-lite** (harvest-timing notes tied to listed crops) only if it provably improves supply reliability; buyer analytics preview (premium hook). |

v1.1 and v2 items ship on both platforms simultaneously per the parity contract — no "Android first because FCM is easier".

## 9. Instrumentation

Transport: events buffered on-device and batched to `POST /events` (S4, [06-PRD-BACKEND.md](06-PRD-BACKEND.md) §6.11) — batch body is exactly `{platform, app_version, events: [{name, ts, props}]}`; flush at 20 events / 30 s / app-background; offline buffer survives restarts. `user_id` and `region_id` are derived server-side from the JWT and never client-sent; `role` is not stored on `app_events`. `os_version`, `language`, `network_state` (and `device_id`, if kept) travel as `props` keys, not top-level fields. Pre-login events (`app_open`, `otp_requested`, `otp_failed`, `role_selected`, …) are buffered on-device and flushed only **after** auth, attributed to the resulting `user_id`; sessions that never authenticate are never uploaded (S4 is authed-only at MVP; pre-auth ingestion is deferred per 06). No third-party analytics SDK at MVP (size + privacy budget).

Batch-level fields: `platform`, `app_version`. Per-event fields: `name`, `ts`, `props` — with `os_version`, `language`, `network_state` carried as `props` keys on every event.

| Event | Extra properties | Fired when |
|---|---|---|
| `app_open` | `cold_start_ms` | App foregrounded |
| `language_selected` | `language` | S-02 confirm |
| `otp_requested` | — | S-03 CTA success |
| `otp_verified` | `new_user` | S-04 success |
| `otp_failed` | `reason` (wrong/locked/network) | S-04 failure |
| `role_selected` | `role` | S-05 card chosen |
| `signup_completed` | `role` | S-05 submit success |
| `farmer_home_viewed` | `price_count, prices_stale` | F-01 render |
| `sell_started` | — | F-01 sell CTA |
| `sell_produce_selected` | `produce_id` | F-02 step 1 |
| `sell_qty_set` | `qty_kg, harvest_date_offset` | F-02 step 2 → 3 |
| `sell_submitted` | `listing_id, produce_id, qty_kg, est_low, est_high` | F-02 201 |
| `sell_failed` | `error_code, step` | F-02 failure |
| `my_listings_viewed` | `count_posted, count_graded, count_allocated` | F-03 render |
| `listing_detail_viewed` | `listing_id, status` | F-03 sheet open |
| `payments_viewed` | `payout_count, month_total` | F-04 render |
| `payout_detail_viewed` | `payment_id` | F-04 sheet open |
| `catalog_viewed` | `sku_count, activated` | B-01 render |
| `catalog_item_added` | `produce_id, grade, qty_kg` | B-01 add |
| `cart_viewed` | `item_count, quoted_total` | B-02 render |
| `cart_item_removed` | `produce_id, grade` | B-02 remove |
| `order_submitted` | `order_id, item_count, total, delivery_date_offset` | B-02 201 |
| `order_failed` | `error_code` | B-02 failure |
| `orders_viewed` | `order_count` | B-03 render |
| `order_detail_viewed` | `order_id, status` | B-04 render |
| `trace_viewed` | `order_id, fresh_hours` | B-04 trace scrolled into view |
| `profile_viewed` | — | P-01 render |
| `language_changed` | `from, to` | P-01 change |
| `logged_out` | — | P-01 confirm |
| `api_error` | `endpoint, status_code` | Any non-2xx |
| `offline_banner_shown` | `screen_id` | Banner appears |
| `update_required_shown` | `min_version, current_version, blocking` | S-06 blocking screen shown, or soft-nudge banner shown (`blocking=false`) |
| `push_opened` (v1.1) | `type, target_screen` | Notification tap |

North-star product funnels: farmer **sell funnel** (`sell_started → sell_submitted`, target ≥80% completion) and buyer **order funnel** (`catalog_item_added → order_submitted`, target ≥60%); plus weekly active farmers listing and buyer repeat-order rate feeding the pilot go/no-go in [13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md).

## 10. Open questions (decisions taken in this draft — founder to ratify)

| # | Question | Decision taken here | Owner / deadline |
|---|---|---|---|
| 1 | One account = one role, or role switching? | One role per account at MVP; switch via support call | Founder, PRD approval |
| 2 | Order cutoff time & minimum order value | 18:00 (D-1) cutoff / ₹1,000 minimum / ₹50 delivery fee (waived ≥₹4,000) / 06:00–10:00 delivery window — all server-configured via `GET /config`; validate against real HoReCa behavior | Founder, end of Phase 0 (T0.2 interview data) |
| 3 | Farmer-facing price = net farmer price (take rate pre-netted) | Yes — the green number must be literally what the farmer receives; requires `farmer_price_a/b` in the price feed | Backend PRD ([06-PRD-BACKEND.md](06-PRD-BACKEND.md)), before build |
| 4 | Buyer self-serve vs ops-activated | Ops-activated (pending banner); B2B sales-led, prevents junk accounts | Founder, PRD approval |
| 5 | Third-party analytics SDK | No at MVP — own `POST /events` endpoint | Revisit at v1.1 if querying becomes painful |
| 6 | Farmer traceability privacy (name/village exposure to buyers) | First name + village only, with farmer consent captured at FPO onboarding | [18-LEGAL-COMPLIANCE.md](18-LEGAL-COMPLIANCE.md), before pilot |
| 7 | Voice-note attachment on listings (field request in old backlog) | Deferred to v2 alongside voice input — one voice feature, done properly | Founder, v2 planning |
