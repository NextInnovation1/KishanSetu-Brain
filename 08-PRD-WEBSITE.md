# 08 — PRD: Marketing Website

**Product:** KisanSetu marketing site (lead capture + credibility)
**Owner:** Alpesh (founder) · **Status:** Draft for founder approval · **Depends on:** `06-PRD-BACKEND.md` (`POST /leads`), `10-DESIGN-SYSTEM.md` (tokens, components), `04-GTM-SALES-MARKETING.md` (channels the site feeds)
**Golden rules that bind this doc:** #2 Global-first/Surat-first, #3 Plan before code, #5 The two metrics (see `00-GOLDEN-RULES.md`).

---

## 1. Purpose

The website is a **sales tool, not a product**. It exists to do three jobs:

1. **Credibility in the room.** When the founder walks into a Surat restaurant with a sample crate, the owner will Google us that evening. The site must make a two-person pilot look like a serious operation — real numbers, real mechanics, no startup gloss.
2. **Lead capture.** Convert Meta lead-gen clicks, local-SEO searches and word-of-mouth into `POST /leads` records with a phone number we can call back within 24 hours.
3. **Publish the two metrics.** The freshness SLA (<48h promise) and the farmer-share claim (60–70%) live here publicly — the site is where "we publish it" happens (Golden Rule #5).

It is explicitly **not** an ordering system, not a farmer app substitute, and not a content-marketing machine.

## 2. Users & jobs-to-be-done

| User | Arrives via | Job-to-be-done | Success |
|---|---|---|---|
| HoReCa owner/manager (Surat) | Meta ad, Google search, referral, founder's visiting card | "Is this outfit real? Will they actually deliver fresher than my mandi guy?" | Submits lead form or taps WhatsApp within 2 minutes |
| Farmer / FPO field staff | FPO demo day, payout-slip QR, relative's phone | "क्या सच में ज़्यादा पैसा मिलेगा? कैसे जुड़ें?" | Reads Hindi version, submits form or taps WhatsApp |
| FPO director / partner prospect | Founder outreach | "Are these people competent enough to bring my farmers demand?" | Contacts via form (role: FPO) |
| Journalist / investor | PR outreach, Inc42 angle | "What's the thesis? Are the numbers real?" | Finds cited stats, understands the asset-light wedge |

The primary persona is the **HoReCa buyer on a mid-range Android phone, on 4G, at 11 pm after closing**. Everything is optimized for that.

## 3. Scope (in)

- Single-page static site, **vanilla JS, no framework, no build step** (founder-fixed stack; `11-ARCHITECTURE.md`).
- Sections: nav, hero, problem (broken-chain visual + ₹100 receipt), how-it-works (5 steps), for-farmers, for-buyers, impact strip, contact (lead form), footer.
- Full EN/HI i18n toggle (string table; GU pluggable later).
- Lead form → `POST /leads` with WhatsApp click-to-chat fallback.
- Local SEO (Surat instance) + global-ready SEO structure.
- Privacy-light analytics (Plausible) with a defined event taxonomy.
- Deployed to static hosting on the production domain, HTTPS.

## 4. NON-GOALS (will not build, and why)

| Non-goal | Why not |
|---|---|
| **Buyer web-ordering portal** | Separate future PRD (planned as `08b-PRD-BUYER-PORTAL.md`, written only after pilot buyer interviews confirm web > app preference — see `15-TASKS-BACKLOG.md` T5.6). Mixing ordering into the marketing site would drag auth, cart and session complexity into a static page. |
| Live freshness-SLA counter (v1) | Needs real pilot data from `order_allocations` timestamps; shipping it with fake numbers violates the brand. Spec'd below as **Phase v1.1**, built only post-pilot. |
| CMS / blog | No one is writing weekly content in months 1–6. A blog with 2 posts is worse than no blog. Revisit for SEO after pilot. |
| React/Vue/site builders, npm build pipeline | Founder-fixed: vanilla JS. A marketing page does not need hydration. Also keeps the <100KB JS budget trivial to hit. |
| Cookie-consent banner | We use Plausible (no cookies, no PII) precisely to avoid one. If GA4 is ever added, this decision reopens. |
| Farmer app download links (v1) | Apps aren't live at site launch. Placeholder buttons that go nowhere destroy trust. Added in v1.2 when Play/TestFlight links exist. |
| Testimonials/logos section (v1) | We have zero customers. Fake logos are a lie. Added post-pilot with real kitchens' consent. |
| Dark mode | Marketing page, one look, cream/forest. Cuts CSS surface in half. |

## 5. Site structure & exact copy

One page, anchored sections. Nav: `Problem · How it works · For Farmers · For Buyers · Contact` + language switch (`EN | हिं`). Mobile: hamburger-free — nav collapses to a single **Contact** button + language switch; section links move to footer.

All copy lives in `js/strings.js` as `STRINGS.en` / `STRINGS.hi` keyed exactly as below. Region-specific values (city, example village, radius, currency) come from `js/region.js` (`REGION = { city, cityLocal, exampleVillage, radiusKm: 50, currency: "INR", waNumber }`) and are interpolated — **no hardcoded "Surat" outside region.js** (Golden Rule #2).

### 5.1 Hero

| Key | EN | HI |
|---|---|---|
| `hero_eyebrow` | खेत से सीधा किचन तक *(Hindi kept on EN page — deliberate brand signal)* | खेत से सीधा किचन तक |
| `hero_title` | From Farm to Business — Direct, Fresh & Fair. | खेत से धंधे तक — सीधा, ताज़ा और सही दाम। |
| `hero_sub` | KisanSetu links farmer collectives around {city} straight to hotels, restaurants, caterers and cloud kitchens. Graded produce at your door within 24–48 hours of harvest — and the farmer keeps 60–70% of what you pay, not 30–40%. | किसानसेतु {cityLocal} के आसपास के किसान संगठनों को सीधे होटल, रेस्टोरेंट, कैटरर और क्लाउड किचन से जोड़ता है। कटाई के 24–48 घंटे के अंदर ग्रेड की हुई उपज आपके दरवाज़े पर — और आपके दिए दाम का 60–70% किसान के पास जाता है, 30–40% नहीं। |
| `hero_cta_farmer` | मैं किसान हूँ *(Hindi on both langs)* | मैं किसान हूँ |
| `hero_cta_farmer_sub` | I'm a farmer — sell direct | सीधा बेचें, आज पैसा पाएँ |
| `hero_cta_buyer` | I run a food business | मैं फ़ूड बिज़नेस चलाता हूँ |
| `hero_cta_buyer_sub` | Order graded, traceable produce | ग्रेडेड, ट्रेस होने वाली उपज मंगाएँ |
| `hero_trust_1` | {city}, Gujarat | {cityLocal}, गुजरात |
| `hero_trust_2` | 24–48 hr farm-to-door | खेत से दरवाज़े तक 24–48 घंटे |
| `hero_trust_3` | FPO partnered | FPO के साथ साझेदारी |

Both hero CTAs scroll to `#contact`; the farmer CTA carries `data-prefill-role="farmer"` (pre-selects the role field and fires `cta_click`).

### 5.2 Problem — the broken chain (centerpiece)

| Key | EN | HI |
|---|---|---|
| `problem_eyebrow` | The problem | समस्या |
| `problem_title` | Five hands touch your vegetables. Only one grew them. | आपकी सब्ज़ी को पाँच हाथ छूते हैं। उगाने वाला सिर्फ़ एक है। |
| `problem_lead` | By the time produce travels from a farm near {city} to a kitchen inside the city, most of its value has been eaten along the way — and a good share of the produce itself has rotted in transit. | खेत से शहर की रसोई तक पहुँचते-पहुँचते उपज की ज़्यादातर कीमत रास्ते में बँट जाती है — और अच्छी-खासी उपज रास्ते में ही सड़ जाती है। |
| `chain_old_label` | Today — the mandi chain | आज — मंडी की कड़ी |
| `chain_new_label` | With KisanSetu — one bridge | किसानसेतु के साथ — एक पुल |
| `chain_farmer` / `chain_farmer_old_cut` | Farmer / keeps ₹30–40 | किसान / ₹30–40 रखता है |
| `chain_agent` / `chain_agent_cut` | Commission agent / + commission | आढ़तिया / + कमीशन |
| `chain_wholesaler_cut` | Wholesaler / + margin | थोक व्यापारी / + मार्जिन |
| `chain_retailer_cut` | Retailer / + margin | रिटेलर / + मार्जिन |
| `chain_consumer` / `chain_consumer_cut` | Your kitchen / pays ₹100 | आपकी रसोई / ₹100 देती है |
| `chain_farmer_new_cut` | keeps ₹60–70 | ₹60–70 रखता है |
| `chain_setu_cut` | grades · moves · pays | ग्रेड · डिलीवरी · तुरंत पैसा |
| `chain_kitchen_new_cut` | same price, fresher stock | वही दाम, ज़्यादा ताज़ा माल |
| `receipt_title` | Where your ₹100 goes | आपके ₹100 कहाँ जाते हैं |
| `stat1_txt` | is all the farmer keeps of what you pay at the end of the chain. | — बस इतना ही किसान के हाथ आता है आपके दिए दाम में से। |
| `stat2_num` / `stat2_txt` | ₹1.5 lakh crore / of produce is lost in India every year after harvest — before anyone eats it. | ₹1.5 लाख करोड़ / की उपज भारत में हर साल कटाई के बाद बर्बाद हो जाती है — किसी की थाली तक पहुँचने से पहले। |

**Broken-chain visual spec** (this is the page's hero argument — treat as a component, not an illustration):

- **Layout:** two horizontal node-chains stacked vertically. Chain 1 ("today"): 5 nodes — Farmer → Commission agent → Wholesaler → Retailer → Your kitchen — connected by arrow links in `stone-400`. Chain 2 ("KisanSetu"): 3 nodes — Farmer → KisanSetu bridge mark → Your kitchen — arrows in `leaf`.
- **Node anatomy:** card 12px radius, 1px `stone-200` border on `#FFFFFF`; name (16/600), role note (13, `stone-500`), money cut line. Farmer node money line: chain 1 in `error #DC2626` ("keeps ₹30–40"), chain 2 in `leaf-deep #15803D` ("keeps ₹60–70"). The KisanSetu node uses the bridge SVG mark (48×32 viewBox from the brand set) + `forest` tinted background at 8%.
- **The receipt bar** sits directly below: two stacked 100%-width horizontal bars, 40px tall, segments with `flex-basis` = rupee share. Row "Mandi chain": farmer ₹35 (`forest`), agent ₹8 (`stone-400`), wholesaler ₹18 (`stone-500`), retailer ₹27 (`stone-600`), wastage ₹12 (diagonal-hatch pattern on `error` 20% tint). Row "KisanSetu": farmer ₹65 (`forest`), KisanSetu ops ₹35 (`leaf`). Each segment ≥6% gets an inline ₹ label; smaller segments label only in the legend. The whole bar carries `role="img"` with a full-sentence `aria-label` (localized keys `receipt_old_aria`, `receipt_new_aria`).
- **Numbers are illustrative and must say so:** a 12px footnote under the receipt — EN: "Illustrative split for a typical vegetable crate; pilot data will replace this." / HI: "उदाहरण के लिए; पायलट के असली आँकड़े यहाँ आएँगे।" Post-pilot, these numbers are replaced by measured farmer-share (Golden Rule #5) — same DOM, new constants.
- **Motion:** on first scroll into viewport (IntersectionObserver, threshold 0.35), receipt segments grow from 0 to width over 600ms ease-out, staggered 80ms; chain nodes fade-slide up 12px. Respect `prefers-reduced-motion: reduce` → no animation, final state rendered. No motion library — CSS transitions toggled by one `is-visible` class.
- **Responsive:** ≥768px chains render horizontally; <768px each chain becomes a vertical list with downward arrows, receipt bars stay horizontal (they read fine at 320px). No horizontal page scroll ever.

### 5.3 How it works — 5 steps

Numbered list `01–05`, one line each. Keys `step{n}_t` / `step{n}_d`:

| # | EN title — desc | HI title — desc |
|---|---|---|
| 01 | **List** — The farmer lists tomorrow's harvest — crop and quantity — on the app or through FPO staff. | **लिस्ट करें** — किसान कल की फ़सल ऐप से या FPO स्टाफ़ के ज़रिए लिस्ट करता है — कौन-सी फ़सल, कितनी। |
| 02 | **Aggregate & grade** — The nearest FPO hub collects and weighs, sorts into A/B tiers and tags every crate to its farm. | **इकट्ठा और ग्रेड** — नज़दीकी FPO हब पर तौल होती है, A/B ग्रेड बनते हैं, और हर क्रेट पर उसके खेत का टैग लगता है। |
| 03 | **Order** — Kitchens order by 8 pm for next-day delivery, at prices locked for the week. | **ऑर्डर करें** — किचन रात 8 बजे तक ऑर्डर करते हैं, अगले दिन डिलीवरी — हफ़्ते भर के पक्के दाम पर। |
| 04 | **Deliver fresh** — Cold-chain vans move crates hub-to-door within 24–48 hours of harvest. | **ताज़ा डिलीवरी** — कोल्ड-चेन गाड़ियाँ कटाई के 24–48 घंटे के अंदर क्रेट पहुँचा देती हैं। |
| 05 | **Instant pay** — The farmer is paid by UPI at pickup — not 30 days later. | **तुरंत पैसा** — पिकअप पर ही किसान को UPI से पैसा — 30 दिन बाद नहीं। |

### 5.4 For farmers / For buyers

Benefit lists (check icon + title + one-liner). Farmer section headline: EN "Your harvest. Your price. Your money — today." / HI "आपकी फ़सल। आपका दाम। आपका पैसा — आज ही।" Benefits (keys `farmers_b1..b4`): 60–70% of the final price; UPI payment at pickup; Live fair prices (see what kitchens pay today); Confirmed pre-orders (harvest what's already sold). CTA `farmers_cta`: "Join as a farmer" / "किसान बनकर जुड़ें" → `#contact` with `data-prefill-role="farmer"`.

Buyer section headline: EN "One vendor. Graded, traceable, every morning." / HI "एक ही वेंडर। ग्रेडेड और ट्रेस होने वाली उपज, हर सुबह।" Benefits (keys `buyers_b1..b5`): A/B quality tiers; Farm-to-fork traceability (every crate carries its farm, farmer, harvest date); One vendor, one invoice (single tax invoice weekly); Stable pricing (weekly locked rates vs daily swings); 24–48 hour freshness ("Cut today near {exampleVillage}, in your prep line tomorrow"). CTA `buyers_cta`: "Order for your kitchen" / "अपने किचन के लिए मंगाएँ".

### 5.5 Impact strip

Four **stat tiles** (component per `10-DESIGN-SYSTEM.md` §6.4) on `forest` background, `cream` text: `~150M` farmers in India · `86%` of them small & marginal · `₹1.5 L cr` lost after harvest yearly · `<48 h` our farm-to-door promise (this tile's numeral in `leaf`). Sources cited in a 12px footnote (Agriculture Census 2015–16; NABCONS 2022) — citing sources on a marketing page is the brand.

### 5.6 Contact + lead form

Headline: EN "Join the {city} pilot." / HI "{cityLocal} पायलट से जुड़िए।" Lead: EN "We're onboarding our first farmer collectives and kitchens right now. Leave your number — we call back within a day." / HI "हम अभी अपने पहले किसान संगठनों और किचन को जोड़ रहे हैं। अपना नंबर छोड़िए — एक दिन के अंदर कॉल आएगा।" Pilot note: farmers within ~{radiusKm} km and kitchens inside the city get priority.

Footer: brand mark, tagline EN "The bridge between the farm and your kitchen." / HI "खेत और आपकी रसोई के बीच का पुल।", © year, "Made in {city} 🇮🇳" (flag emoji comes from region.js too — global-first), privacy note link (one static paragraph: what we store from the form, that we call back, no resale of data).

## 6. Lead form — fields, validation, contract

### 6.1 Fields

| Field | `name` | Type | Required | Validation (client) | Notes |
|---|---|---|---|---|---|
| Name | `name` | text | yes | 2–80 chars after trim | `autocomplete="name"` |
| Phone | `phone` | tel | yes | Strip spaces/`-`/`(`/`)`; accept `+` + 8–15 digits (E.164) or bare 10 digits → prefix `REGION.dialCode` (+91) | `inputmode="tel"`; shown error under field |
| I am a | `role` | select | yes | one of `farmer · restaurant · hotel · caterer · cloud_kitchen · fpo · other` | values are the API enum, labels localized |
| Message | `message` | textarea | no | ≤500 chars | placeholder: "e.g. I grow okra and tomatoes near {exampleVillage}" |
| Website | `company` | text | — | **honeypot**: visually hidden (`position:absolute;left:-9999px`), `tabindex="-1"`, `autocomplete="off"`; if non-empty → silently fake-succeed | spam control #1 |
| — | (timing) | — | — | reject submit if <3s since page interaction started → show generic error | spam control #2 |

Inline errors, localized, shown on blur and on submit; error text + red border + `aria-invalid="true"` + `aria-describedby` pointing at the error node. Submit button disables during flight and shows "Sending… / भेज रहे हैं…".

### 6.2 API contract — `POST /leads`

Public endpoint, no auth (full definition owned by `06-PRD-BACKEND.md`; this is the client-side contract):

```
POST {API_BASE}/leads
Content-Type: application/json

{
  "name": "Ramesh Patel",
  "phone": "+919876543210",          // E.164, normalized client-side
  "role": "restaurant",              // enum above
  "message": "Need daily tomatoes",  // optional, ≤500
  "language": "hi",                  // active site language
  "source": "website",
  "utm_source": "meta", "utm_medium": "cpc", "utm_campaign": "surat-pilot-1",  // optional, read from URL and persisted in sessionStorage
  "page_path": "/",
  "consent": true                    // implicit via submit; checkbox not shown (callback request = consent), stated in privacy note
}

201 → { "id": "…", "status": "received" }        → success state
400 → { "error": "invalid_phone" | "invalid_role" | … }  → map to localized field error; unknown code → generic error
429 → { "error": "rate_limited" }                 → show WhatsApp fallback immediately
5xx / network error / 8s timeout                  → show WhatsApp fallback
```

Success state replaces the form (not a redirect): check icon + EN "Got it. We'll call you within a day. Faster: message us on WhatsApp →" / HI "मिल गया। एक दिन के अंदर कॉल आएगा। और जल्दी चाहिए? WhatsApp पर लिखिए →" with the WhatsApp link.

### 6.3 WhatsApp fallback

- Constant `REGION.waNumber` (E.164, no `+` for wa.me). **Value = founder's WhatsApp Business number — open question Q3 below.**
- Fallback link: `https://wa.me/{waNumber}?text={urlencode(template)}` where template (localized) = EN `"Hi KisanSetu — I'm {name}, a {role}. Please call me back."` / HI `"नमस्ते किसानसेतु — मैं {name} हूँ ({role})। कृपया कॉल करें।"` using whatever fields were filled.
- Shown: (a) on any submit failure, replacing the submit error area — EN "Couldn't send just now. Message us on WhatsApp instead — it goes straight to the founder's phone." / HI "अभी भेजा नहीं जा सका। WhatsApp पर लिखिए — सीधे हमारे फ़ोन पर पहुँचेगा।"; (b) as a persistent floating WhatsApp button, 56×56px, bottom-right, `leaf` background, appears after the user scrolls past the hero, `aria-label="Chat on WhatsApp"`.
- If JS is disabled entirely: the form is a real `<form method="post">`-less block, so include a `<noscript>` panel with the phone number and wa.me link as plain anchors. Zero-JS users can still convert.

## 7. i18n toggle mechanics

- **String table:** `js/strings.js` — `const STRINGS = { en: {...}, hi: {...} }`. Adding Gujarati = add `gu` object + one button; no other code changes (acceptance-tested).
- **Markup binding:** `data-i18n="key"` (textContent), `data-i18n-placeholder`, `data-i18n-aria` (aria-label), `data-i18n-html` deliberately **not supported** (no HTML in strings → no injection surface). Interpolation: `{city}`-style tokens resolved against `REGION` at apply time.
- **`setLang(lang)`** does, in order: validate against `Object.keys(STRINGS)`; set `document.documentElement.lang`; walk all `data-i18n*` nodes and apply; toggle `aria-pressed` on the two lang buttons; persist `localStorage["ks_lang"]`; `history.replaceState` to set/remove `?lang=`; update `<title>` + `<meta name="description">` from keys `meta_title`/`meta_desc`; fire analytics `lang_toggle {to}`. Total function <60 lines.
- **Detection order on load:** `?lang=` param → `localStorage.ks_lang` → `navigator.language.startsWith("hi")` → default `en`. Applied before first paint by an inline 10-line head script that only sets a `lang-hi` class + defers full application to `main.js` (avoids FOUC of wrong-language hero for the 60%+ of ad traffic that is Hindi-first).
- **The default HTML ships English text in markup** (crawlable without JS); Hindi is applied by JS. Hindi SEO handled via §8.
- Hindi typography: `:lang(hi)` CSS bumps `line-height` +12% and disables the serif display font for Devanagari (system font renders it better) per `10-DESIGN-SYSTEM.md` §3.

## 8. SEO plan

- **Meta (per language):** EN title `KisanSetu — Farm-fresh produce for restaurants & hotels in {city} | 24–48h from harvest`; HI title `किसानसेतु — {cityLocal} के रेस्टोरेंट-होटल के लिए खेत से ताज़ा सब्ज़ी, 24–48 घंटे में`. Description = hero_sub trimmed to 155 chars.
- **URLs & hreflang:** canonical `https://{domain}/`; `<link rel="alternate" hreflang="hi" href="https://{domain}/?lang=hi">`, `hreflang="en"` → `/`, `x-default` → `/`. v1.1: a 20-line Node script (run manually, not a build step for the site) pre-renders `/hi/index.html` with Hindi strings baked in for proper Hindi indexing.
- **Structured data (JSON-LD, inline):** `Organization` (name, logo, sameAs → WhatsApp/GBP/Instagram), `LocalBusiness` (Surat instance: address locality from region.js, `areaServed`), `FAQPage` with 4 Q&As (Do you deliver daily? What does grading A/B mean? How fast is delivery? How do farmers get paid?) — the FAQ section renders visibly above the footer as `<details>` elements, so the markup is honest.
- **Local keyword set (10, from `15-TASKS-BACKLOG.md` T7.2)** worked naturally into headings/body, not stuffed: fresh vegetable supplier Surat · restaurant vegetable wholesale Surat · hotel vegetable supplier Surat · bulk vegetables for restaurants Surat · B2B farm fresh vegetables Surat · vegetable supplier for cloud kitchen · FPO produce supplier Gujarat · किसान से सीधे सब्ज़ी · सूरत सब्ज़ी होलसेल रेट · होटल के लिए सब्ज़ी सप्लायर सूरत. These live in copy/FAQ answers and the GBP profile description (see `04-GTM-SALES-MARKETING.md`). New launch city = new keyword set in region.js docs, same template.
- **OG/Twitter:** `og:title`, `og:description`, `og:image` — one 1200×630 static PNG (the broken-chain graphic, exported once; the only raster asset on the site).
- `sitemap.xml` (one URL + hi alternate), `robots.txt` (allow all), Google Search Console + Bing verified at launch; GBP website link points here.

## 9. Analytics events

Plausible (script ≤1.5KB, cookieless, EU-hostable — global-first). Custom events, exact names:

| Event | Props | Fires when |
|---|---|---|
| `pageview` | (auto: path, referrer, utm_*) | auto |
| `lang_toggle` | `to: en\|hi` | language switched |
| `cta_click` | `id: hero_farmer\|hero_buyer\|farmers_join\|buyers_order\|nav_contact` | any CTA pressed |
| `section_view` | `section: problem\|how\|farmers\|buyers\|impact\|contact` | 50% of section visible ≥1s (IntersectionObserver, once per session) |
| `lead_submit_attempt` | `role` | submit pressed, client validation passed |
| `lead_submit_success` | `role`, `lang` | 201 received |
| `lead_submit_error` | `reason: validation\|rate_limited\|server\|network\|timeout` | any failure |
| `whatsapp_click` | `location: float\|fallback\|success\|noscript-n/a` | wa.me link opened |

North-star funnel: `pageview → section_view:contact → lead_submit_success` — reviewed weekly alongside GTM metrics. Meta ads' `utm_campaign` must match the campaign naming in `04-GTM-SALES-MARKETING.md` so cost-per-lead is computable.

## 10. Performance budget (hard gates, checked at every deploy)

| Budget | Limit | Expected v1 |
|---|---|---|
| Total JS (all files + analytics, uncompressed) | **<100KB** (golden constraint) | ~25KB (`main.js` ~8KB, `strings.js` ~12KB, Plausible ~1.5KB) |
| CSS | ≤50KB uncompressed | ~30KB |
| Total transfer, first load (gzip) | ≤300KB | ~90KB (no webfonts, no raster images in-page, all SVG inline) |
| LCP (Moto G4-class, 4G throttle) | ≤2.5s | hero is text → ~1.2s |
| CLS | <0.1 | 0 (no late-loading media; SVGs have viewBox dimensions) |
| Lighthouse (mobile) | ≥95 Performance, ≥95 Accessibility, ≥95 SEO | — |

Techniques: system font stacks only (serif display = Iowan Old Style/Palatino/Georgia — zero font bytes); one CSS file, `<link>` non-blocking is unnecessary at this size; `main.js` + `strings.js` with `defer`; no third-party embeds (no maps, no chat widgets — WhatsApp is a plain link). CI check: a 10-line script fails deploy if `du -b js/*.js | total > 100000`.

## 11. Edge cases & error states

| Case | Behavior |
|---|---|
| JS disabled | Full content readable (EN), `<noscript>` contact panel with tel: + wa.me links; form hidden (it needs fetch) |
| API down / CORS / timeout 8s | WhatsApp fallback panel (§6.3); error logged to Plausible `lead_submit_error` |
| Double submit | Button disabled in-flight; backend also dedupes same phone+role within 24h (409 → treated as success client-side: "You're already on the list — we'll call.") |
| Offline (restaurant basement) | `navigator.onLine === false` at submit → skip fetch, show WhatsApp fallback + "No connection" note |
| Hindi text overflow | All buttons/cards tested with HI strings (10–30% longer); `min-height` not fixed heights |
| 320px viewport | Chains go vertical; receipt bar labels drop to legend-only below 360px |
| Old browsers (Android WebView 80+) | No optional chaining in shipped JS beyond ES2018; feature-detect IntersectionObserver → static fallback (everything visible) |
| Spam flood | Honeypot + 3s time-gate client-side; rate limit server-side (429 path handled) |
| User pastes +91 with spaces / leading 0 | Normalizer handles `0XXXXXXXXXX`, `+91 XXXXX XXXXX`, `91XXXXXXXXXX` → E.164 |

## 12. Acceptance criteria

1. **Copy & i18n:** every visible string comes from `strings.js`; toggling EN↔हिं swaps 100% of copy incl. `<title>`, meta description, aria-labels, placeholders; choice persists across reload; `?lang=hi` deep-link works; adding a stub `gu` object makes a third language appear with no other code edits.
2. **Broken chain:** renders correctly at 320/768/1280px; animates once on scroll; `prefers-reduced-motion` shows final state; receipt bars have localized `aria-label` sentences; illustrative-numbers footnote present.
3. **Lead form:** valid submit hits `POST /leads` with the exact §6.2 payload and shows success state; each failure class (400/429/5xx/network/timeout/offline) shows its specified state; honeypot fill → fake success, no request; phone normalizer passes the 4 formats in §11; all errors localized.
4. **WhatsApp:** floating button appears post-hero scroll with 56px target; fallback link opens wa.me with correctly URL-encoded localized template.
5. **SEO:** JSON-LD validates in Rich Results test; hreflang pair present; sitemap + robots served; OG image renders in WhatsApp link preview (this is how the founder's card gets shared — test it specifically).
6. **Analytics:** all 8 events fire with exact names/props above, verified in Plausible dashboard.
7. **Performance:** all §10 gates pass on a throttled Moto G4 profile in Lighthouse CI before deploy.
8. **Golden-rule check:** no "Surat", "₹", "+91", or "mandi" hardcoded outside `region.js`/`strings.js` (grep test in CI); farmer-share and freshness numbers appear and are cited.

## 13. Phased rollout

| Phase | Contents | Gate |
|---|---|---|
| **v1 (MVP, week 1 of build — needed for sales meetings)** | Everything in §5–§11; deployed to production domain on static hosting (Cloudflare Pages or equivalent; any static host works — no lock-in) | Founder approves copy; 3 test leads round-trip to backend + founder's callback sheet |
| **v1.1 (post-pilot week 2)** | **Live freshness-SLA counter:** section under impact strip — "Median farm-to-door this week: **{n} hours**, across {m} deliveries", fed by public read-only `GET /stats/freshness` (7-day rolling median from `order_allocations` timestamps, cached 1h; endpoint spec'd in `06-PRD-BACKEND.md`). Shows only when `m ≥ 20` deliveries/week — below that, hide the section (small-n numbers are noise, not proof). Also: pre-rendered `/hi/` page; real farmer-share % replaces illustrative receipt numbers. | 2 weeks of clean SLA timestamps |
| **v1.2** | App download buttons (Play + TestFlight/App Store links), testimonial section with ≥2 real named kitchens (consented), payout-slip photo gallery | Apps live; consents signed |
| **v2 (separate PRD `08b-PRD-BUYER-PORTAL.md`)** | Buyer web-ordering portal on `order.{domain}` — auth, catalog, cart against the same API. **Not part of this site**; written only after pilot proves buyer preference for web ordering. | Pilot interview data |

## 14. Metrics

Weekly: unique visitors, visitor→lead conversion (target ≥3% from paid, ≥8% from referral traffic), leads by role, cost per lead by utm_campaign (target ≤₹150 buyer-side per `04-GTM-SALES-MARKETING.md`), WhatsApp clicks, lang split (expect ≥40% HI), LCP p75 from CrUX once traffic exists.

## 15. Open questions (decided by author where unblocking; flagged for founder)

| # | Question | Decision taken here / owner | Deadline |
|---|---|---|---|
| Q1 | Domain — kisansetu.in vs .com vs both | **Assumed `kisansetu.in` + .com redirect**; blocked on trademark check (T0.5). Owner: Alpesh | Before deploy (build week 1) |
| Q2 | Analytics vendor | **Decided: Plausible** (cookieless → no consent banner, <2KB, fits budget). GA4 rejected for v1. Owner: Alpesh may veto | Build week 1 |
| Q3 | WhatsApp number for wa.me + floating button | Needs founder's WhatsApp Business number. Owner: Alpesh | Before deploy |
| Q4 | Receipt-bar illustrative split (35/8/18/27/12) | **Kept from existing scaffold**; must be replaced by mandi-shadow data from Phase 0 (T0.4). Owner: Alpesh | End of Phase 0 |
| Q5 | Static host | **Decided: Cloudflare Pages** (free, fast in India, trivial rollback). Owner: Alpesh may veto | Build week 1 |
