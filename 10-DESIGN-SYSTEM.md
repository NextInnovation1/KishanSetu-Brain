# 10 — Design System

**Scope:** one visual + interaction system across Website (vanilla JS), Android (Jetpack Compose + Material3), iOS (SwiftUI), Ops Dashboard (web), and print (payout slips, grading charts, sales leave-behinds).
**Owner:** Alpesh (founder) + contract designer · **Status:** Draft for founder approval
**Binds:** `07-PRD-MOBILE-APPS.md`, `08-PRD-WEBSITE.md`, `09-PRD-OPS-DASHBOARD.md`. Governed by Golden Rule #1 (Android↔iOS parity) — one system, two renderers.
**Visual reference:** `design/design-board.html` (this repo; also published as a Claude artifact) — sections: *Trust, in green and cream* (tokens), *Two taps to sell* (farmer app), *Order by grade, verify by farm* (buyer app), *The broken chain is the centerpiece* (website), *Four components carry the whole product*. Where that board and this doc disagree, **this doc wins**; update the board in the same change.

---

## 1. Brand idea

**"Setu" = bridge.** The brand's job is trust on both ends: a farmer who has been underpaid for generations, and a chef who has been burned by rotten deliveries. Everything is honest, legible, unfussy — real numbers, real farm names, no gloss. The design system's single test: *would this element survive being printed on a payout slip and handed to a farmer?* If it's decorative, cut it.

## 2. Color tokens

Source of truth is this table. Each platform mirrors it verbatim (naming per §8). Never hardcode a hex outside the token file.

### 2.1 Core palette

| Token | Hex | Usage | Never use for |
|---|---|---|---|
| `forest` | `#14532D` | Primary. Headers, primary buttons, brand surfaces, Grade A chip fill, links on cream | Large text blocks (use `ink`) |
| `leaf` | `#22C55E` | Money in the user's favor, success fills, "fresh" markers, icons, large display numerals (≥24sp bold) | **Body-size text on light** (2.1:1 — fails contrast; use `leaf-deep`) |
| `leaf-deep` | `#15803D` | Text-sized money/success copy on light backgrounds (4.7:1 on cream) | Fills where `leaf` is expected |
| `cream` | `#FAF7F0` | Ground. Light-mode background, text on forest/night | — |
| `ink` | `#1C1917` | Text on light, dark neutral | Backgrounds except footer |
| `amber` | `#F59E0B` | Grade B chip fill, harvest highlights, stale-data warnings. **Sparingly** — if a screen has >2 amber elements, redesign it | Success or money |
| `error` | `#DC2626` | Errors, destructive actions, "money against the user" (old-chain farmer share) | Emphasis/decoration |

### 2.2 Neutrals & dark surfaces

| Token | Hex | Usage |
|---|---|---|
| `white` | `#FFFFFF` | Cards on cream |
| `stone-200` | `#E7E5E4` | Borders, dividers, disabled fills |
| `stone-400` | `#A8A29E` | Struck-through reference prices, placeholders |
| `stone-500` | `#78716C` | Secondary text on light (4.5:1 on cream — smallest text allowed) |
| `night` | `#0C1A12` | Dark-mode background (green-black), footer, dashboards dark |
| `night-raised` | `#132A1C` | Dark-mode cards/raised surfaces |

### 2.3 Semantic mapping

`success = leaf` (fills) / `leaf-deep` (text) · `warning = amber` · `danger = error` · `bg = cream` (light) / `night` (dark) · `surface = white` / `night-raised` · `text = ink` / `cream` · `text-secondary = stone-500` / `#9CB8A6` (muted green, 8.4:1 on `night`) · `border = stone-200` / `#24402F`.

Dark mode ships on **iOS + Android + ops dashboard** (system-following). Website is light-only (`08-PRD-WEBSITE.md` non-goal).

### 2.4 Grade colors (fixed, product-wide)

**Grade A = `forest` chip, `cream` text. Grade B = `amber` chip, `ink` text.** The letter A/B is always rendered inside the chip — grade is never conveyed by color alone (a11y §9).

## 3. Typography

### 3.1 Families

| Platform | Display | Body | Numerals |
|---|---|---|---|
| Web / print | Serif stack: `Iowan Old Style, "Palatino Linotype", Palatino, Georgia, serif` | System sans: `system-ui, -apple-system, "Segoe UI", Roboto, sans-serif` | `font-variant-numeric: tabular-nums` on all money/stat figures |
| Android | Roboto (Material3 default) — no custom font download | Roboto | `FontFeature "tnum"` on money styles |
| iOS | SF Pro (system) | SF Pro, Dynamic Type respected | `.monospacedDigit()` on money styles |
| Devanagari/Gujarati | System (Noto/Mukta family) — **never the serif display**; `:lang(hi)`/locale check swaps to system sans | same | same |

Hindi/Gujarati strings need **+10–15% line-height** over Latin — bake into the HI text styles, don't patch per screen. Zero webfont downloads anywhere (performance + rural bandwidth).

### 3.2 Type scale — Android (Material3 roles, enlarged for farmer flow)

| Role (M3 token) | Farmer flow | Buyer flow | Use |
|---|---|---|---|
| `displaySmall` | 40sp/48 bold | 36sp/44 bold | Money hero figures (payout amount, today's price) |
| `headlineMedium` | 28sp/36 semibold | 28sp/36 | Screen titles |
| `titleLarge` | 24sp/32 semibold | 22sp/28 | Card titles, produce names |
| `bodyLarge` | **18sp/28** (farmer minimum body) | 16sp/24 | Body copy |
| `labelLarge` | 18sp/24 medium | 14sp/20 | Buttons, chips |
| `bodySmall` | 14sp/20 (only for legal/meta, never instructions) | 12sp/16 | Timestamps, UTR numbers |

### 3.3 Type scale — iOS (SwiftUI text styles)

| SwiftUI style | Farmer flow | Buyer flow | Maps to Android |
|---|---|---|---|
| `.system(size: 40, weight: .bold).monospacedDigit()` | money hero | 36pt | `displaySmall` |
| `.title` (28) | screen titles | `.title` | `headlineMedium` |
| `.title2` (22) / `.title3` (20) | card titles | `.title2` | `titleLarge` |
| `.title3` (20) as body | **farmer body ≥20pt** | `.body` (17) | `bodyLarge` |
| `.callout` (16) | buttons/chips | `.subheadline` (15) | `labelLarge` |
| `.footnote` (13) | meta only | `.caption` (12) | `bodySmall` |

Dynamic Type: all styles use relative text styles or `@ScaledMetric`; layouts must survive up to `.accessibility3` without truncating any money digit (see §9).

### 3.4 Web scale

Fluid: display `clamp(34px, 6vw, 48px)` serif; h2 `clamp(26px, 4vw, 34px)`; h3 22/30; body 17/28; small 14/20. Stat numerals 40–44px tabular.

## 4. Spacing, radius, elevation

- **Spacing scale (4dp/pt/px base):** `s1=4 · s2=8 · s3=12 · s4=16 · s6=24 · s8=32 · s12=48 · s16=64`. Screen gutters: 16 (phones), 24 (≥400dp width). Card padding: 16. Between stacked cards: 12.
- **Radius:** `r-sm=8` (inputs), `r-md=12` (cards, chips container), `r-lg=16` (hero cards, sheets), `r-pill=999` (chips, buttons on farmer flow).
- **Elevation:** light mode = flat white cards with 1px `stone-200` border + shadow `0 1px 2px rgba(28,25,23,0.06)` (Compose: `elevation 1.dp` + border; SwiftUI: `.shadow(radius:1, y:1)` + overlay stroke). Dark mode = no shadows; raise with `night-raised`. Max two elevation levels anywhere — this is a flat, printed-matter brand.
- **Icons:** Material Symbols Rounded (Android/web) ↔ SF Symbols (iOS), 24 default / 28 farmer flow, stroke-style matched by choosing filled variants on both. Icon **always** paired with a text label in farmer flow.

## 5. Buttons & inputs (base components)

- **Primary button:** `forest` fill, `cream` text, height 56 (farmer) / 48 (buyer), `r-pill` farmer / `r-md` buyer, full-width in farmer flow. Pressed: 8% ink overlay. Disabled: `stone-200` fill, `stone-500` text — never a spinner *inside* a disabled button without a label change ("भेज रहे हैं…").
- **Secondary:** 1.5px `forest` outline, `forest` text, transparent fill.
- **Destructive:** `error` fill, only in confirmation dialogs.
- **Inputs:** 56 height, `r-sm`, 1px `stone-400` border → 2px `forest` on focus; error = 2px `error` + message below with icon. Compose `OutlinedTextField` ↔ SwiftUI `TextField` + `.textFieldStyle` custom.
- **Steppers (kg quantity):** 56×56 `−`/`+` buttons flanking a 40sp tabular figure — the whole flow must work with steppers alone; direct numeric entry is available but never required (`07-PRD-MOBILE-APPS.md`).

## 6. The four core components

These four carry the whole product (per the design board). Each is built once per platform, in the shared UI module, and reused — screens compose these, they don't reinvent them.

### 6.1 Price card

*Where:* farmer Home (today's prices), buyer Catalog, website hero mock, ops price-setting review.

**Anatomy** (top→bottom, left-aligned):
1. Produce identity: 48×48 produce image/emoji tile + name — **local language first, big** (`titleLarge`), English/botanical subtitle in `bodySmall stone-500` beneath (buyer flow reverses: EN first, local subtitle).
2. Reference market price: `stone-400`, struck through (`14–16sp`, prefixed localized "मंडी भाव / market"). Never omitted — the comparison IS the sell (farmer-flow law F5).
3. Platform price: `displaySmall` tabular in `leaf-deep` (light) / `leaf` (dark), unit suffix ` /kg` in `bodySmall`. Buyer variant shows two price rows: `A` chip + price, `B` chip + price.
4. Optional footer: freshness/harvest badge or "estimated payout" line.

**Behavior:** farmer tap → New Listing prefilled with that produce; buyer tap → grade/qty selector. **States:** default · loading (skeleton blocks, no spinners) · stale (`amber` dot + "कल का भाव / yesterday's price" when `price_feed` date < today) · no-price (card grayed, "आज भाव नहीं / no price today", not tappable).

**Platform mapping:** Compose `Card(colors = surface)` with `Modifier.clickable`, price via `Text(fontFeatureSettings="tnum")` ↔ SwiftUI `VStack` in `RoundedRectangle(16)` fill `surface`, `.monospacedDigit()`, `Button`/`onTapGesture`. Web: `article.price-card`.

### 6.2 Status chip set

*Where:* farmer My Listings, buyer Orders/Order Detail, ops queues.

**Anatomy:** pill (`r-pill`), height 36 farmer / 28 buyer / 24 ops-dense, 8px status dot + label (`labelLarge`), horizontal padding `s3`. Chip label text is always present (never dot-only).

**Fixed vocabulary — listing states** (`listings.status`, strings from the shared table):

| State | Label (HI · EN) | Fill / text |
|---|---|---|
| `posted` | नई · New | `white` + 1px `stone-400` outline / `ink` |
| `graded` (A) | ग्रेड A · Grade A | `forest` / `cream` |
| `graded` (B) | ग्रेड B · Grade B | `amber` / `ink` |
| `allocated` | बिक गई · Sold | `leaf` 15% tint / `leaf-deep` |
| `closed` | बंद · Closed | `stone-200` / `stone-500` |
| `cancelled` | रद्द · Cancelled | `error` 10% tint / `error` |

**Order states:** `placed` (outline) → `confirmed` (forest 10% tint/forest) → `out_for_delivery` (amber 15% tint/ink) → `delivered` (leaf 15% tint/leaf-deep) · `cancelled` (error 10% tint/error).

**Behavior:** chips are status displays, not buttons — no tap target pretense. Unknown state from API → outline chip with raw status string (never crash on a new enum value).

**Platform mapping:** Compose `SuggestionChip`/custom `Surface(shape=CircleShape)` ↔ SwiftUI `Capsule().fill(...)` + `Label`. Web: `span.chip.chip--graded-a`.

### 6.3 Trace line

*Where:* buyer Order Detail (the premium moment), ops allocation detail, print (invoice annex). This component justifies the take rate — build it beautifully.

**Anatomy:** vertical stepper, 4 fixed nodes mapping to allocation timestamps (`06-PRD-BACKEND.md`): **Farm** (`harvest_ts`) → **Hub** (`hub_in_ts`) → **Dispatch** (`dispatch_ts`) → **Door** (`delivered_ts`). Per node: 12px dot (`forest` filled when timestamp exists, `stone-200` hollow when pending) on a 2px vertical connector (colored to last completed node); left column = node title + detail lines; right-aligned timestamp (`bodySmall`, localized `d MMM · h:mm a`, timezone-aware). Farm node detail: **farmer name, village, produce + grade** — real names, this is the product. Terminal badge under Door: total elapsed `"{n} h farm-to-door"` — `leaf-deep` if ≤48h, `amber` if >48h (we show the number even when it embarrasses us; that's the brand).

**Behavior:** renders from whatever timestamps exist — missing = pending, never fabricated. Multiple farms in one order item → stacked trace lines grouped by farm, farm node first. Tap farm node (buyer) → sheet with farm/FPO details.

**Platform mapping:** Compose custom `Column` + `Canvas` connector ↔ SwiftUI `VStack` + `Path`/`Rectangle` connector, `alignmentGuide` for dots. Web/print: CSS `::before` dots on `li`, prints clean in one color.

### 6.4 Stat tile

*Where:* website impact strip, ops dashboard SLA reports, buyer analytics (later), pitch deck.

**Anatomy:** big numeral (`displaySmall`–44, tabular) + one-line caption (`bodySmall`, `stone-500` on light) + optional delta arrow (`leaf-deep` up-good / `error` down-bad, with explicit `+`/`−` sign, never color-only). Left-aligned; number first, caption under.

**States:** loaded · loading (shimmer on the numeral block) · n/a (em-dash "—" with caption intact — never `0` when data is missing; 0 and unknown are different facts).

**Platform mapping:** Compose `Column` in plain surface ↔ SwiftUI `VStack(alignment:.leading)`. Web: `div.stat-tile`.

## 7. Flow laws

### 7.1 Farmer-flow UX laws (both apps — Golden Rule #1; violations block release)

1. **Max 2 actions per screen**; exactly one obvious primary button, full-width, bottom.
2. **Touch targets ≥48dp/44pt**, primary actions 56; text body ≥18sp/20pt.
3. **Icon + text always together.** Produce is chosen from a **picture grid**, never a dropdown or search field.
4. **No hamburger menu.** Bottom nav, 3 tabs max (घर Home · मेरी फ़सल My produce · पैसा Money).
5. **Every money number shows the comparison:** reference market price struck through in `stone-400`, KisanSetu price in green. No naked prices.
6. **Success screens celebrate plainly:** amount huge, where it went (UPI), when. "पैसा आ गया ✓ · ₹2,208 · 9:14 AM".
7. **Never block on network:** listing posts queue offline and auto-send; the UI says so in one sentence.
8. **Hindi first, English subtitle** on every label; numerals are digits (₹2,208), never spelled out.
9. **No jargon:** "ग्रेड" is allowed (hub staff use it); "allocation", "SKU", "catalog" are not. Vocabulary list lives with the string table.
10. **Confirmation over configuration:** defaults pre-filled (harvest date = tomorrow, last-used produce first in grid); settings screens don't exist in the farmer flow.

### 7.2 Buyer-flow UX laws

1. **Restrained surfaces:** cream/white, forest accents, zero gimmicks — this is a professional procurement tool, not a shopping app. No sale banners, no confetti.
2. **Grade chips everywhere produce appears** — list, cart, order lines, invoice.
3. **Traceability is the premium moment:** trace line gets full-width, generous spacing, real names. It is never collapsed behind a "details" link.
4. **Order status is a timeline, not text** — same trace-line component grammar.
5. **Prices always per kg with the reference market comparison** — we sell on trust arithmetic, both sides of the bridge see the same math.
6. **Freshness in hours, not adjectives:** "Harvested 21 h ago", never "farm-fresh".
7. **Invoice-grade correctness:** totals, tax lines and units formatted to accounting standards (`Intl.NumberFormat`/locale); buyer-facing money uses tabular numerals everywhere.

## 8. Token naming & platform mirrors

| Concept | Web (`css/tokens.css`) | Android (`ui/theme/`) | iOS (`Sources/DesignSystem/`) |
|---|---|---|---|
| Color | `--ks-forest` etc. | `KsColor.Forest` in `Color.kt`; mapped into `lightColorScheme(primary=Forest, …)` | `Color.ksForest` in `Theme.swift` asset catalog + extension |
| Type | `--ks-display`, classes | `KsType` → `Typography(displaySmall=…)` | `Font.ksMoneyHero` etc. |
| Spacing | `--ks-s4` | `KsSpace.s4 = 16.dp` | `KsSpace.s4: CGFloat = 16` |
| Components | `.price-card`, `.chip`, `.trace`, `.stat-tile` | `PriceCard.kt`, `StatusChip.kt`, `TraceLine.kt`, `StatTile.kt` | `PriceCard.swift`, `StatusChip.swift`, `TraceLine.swift`, `StatTile.swift` |

Material3 `colorScheme` mapping (Android): `primary=forest, onPrimary=cream, secondary=leaf-deep, tertiary=amber, error=error, background=cream, surface=white, onSurface=ink, outline=stone-400`; dark: `background=night, surface=night-raised, onSurface=cream, primary=leaf`. iOS mirrors via semantic Color extensions with light/dark variants in the asset catalog.

**Governance:** this file is the single source of truth. Token change protocol: (1) edit this doc, (2) update `design/design-board.html`, (3) update all four mirrors (website CSS, Console CSS, Android, iOS) **in the same cross-platform unit** (parity workflow, `12-DEVELOPMENT-PLAN.md` §3). A PR that changes a token in one platform only must be rejected in review.

## 9. Accessibility (non-negotiable, in Definition of Done)

- **Touch targets:** ≥48dp Android / ≥44pt iOS everywhere; ≥56 for farmer-flow primaries and stepper buttons; website tap targets ≥44px (incl. the 56px WhatsApp button).
- **Contrast (measured, WCAG 2.1):**

| Pair | Ratio | Verdict |
|---|---|---|
| `ink` on `cream` | 16.4:1 | AAA — default body |
| `forest` on `cream` | 8.5:1 | AAA — headings, buttons-as-text |
| `cream` on `forest` | 8.5:1 | AAA — primary buttons |
| `leaf-deep` on `cream` | 4.7:1 | AA — money text ✔ |
| `leaf` on `cream` | 2.1:1 | **FAIL for text** → `leaf` only for ≥24sp-bold numerals (AA large = 3:1 still fails: pair large `leaf` numerals with `leaf-deep` at body sizes) and non-text fills |
| `ink` on `amber` (Grade B chip) | 8.1:1 | AAA |
| `stone-500` on `cream` | 4.5:1 | AA — smallest allowed secondary text |
| `error` on `cream` | 4.5:1 | AA |
| `cream` on `night` | 16.8:1 | AAA — dark mode body |
| `leaf` on `night` | 7.9:1 | AAA — dark-mode money |

- **Dynamic type / font scale:** layouts survive Android `fontScale 2.0` and iOS `.accessibility3` — money digits may wrap but never truncate or ellipsize; test cases in the parity checklist.
- **Never color-alone:** grade shows the letter, deltas show the sign, errors show icon + text, chips carry labels.
- **Screen readers:** every chip announces "स्थिति: ग्रेड A / Status: Grade A"; trace line announces as an ordered list with timestamps; struck-through prices announce "market price {x}, KisanSetu price {y}" (strikethrough alone is invisible to TalkBack/VoiceOver).
- **Localization stress:** every component reviewed with HI strings (longer, taller script) and `lang`-correct fonts; +10–15% line-height rule (§3.1).

## 10. Voice & tone (with good/bad copy)

Principles: numbers over adjectives · Hindi that sounds spoken, not translated · never corporate-NGO "empowerment" speak — show the payout slip instead · freshness always in hours · errors say what happens next, not who to blame.

| Context | ❌ Bad | ✔ Good |
|---|---|---|
| Farmer payout push | "Congratulations! Your payment has been processed successfully! 🎉" | "पैसा आ गया ✓ ₹2,208 आपके UPI में · 9:14 AM" |
| Buyer freshness | "Farm-fresh, hand-picked premium quality!" | "Harvested yesterday 6:10 AM · Olpad village · 21 h ago" |
| Farmer offline error | "Oops! Something went wrong 😅 Please try again." | "इंटरनेट नहीं मिला। लिस्टिंग सेव है — नेट आते ही अपने-आप चली जाएगी।" |
| Grading result | "Your produce has been evaluated by our quality team." | "ग्रेड A · 92 किलो तौला गया · अनुमानित ₹2,208" |
| Website hero | "Empowering farmers through technology-driven supply chains" | "The farmer keeps 60–70% of what you pay, not 30–40." |
| Buyer delay notice | "We apologize for any inconvenience caused." | "Your order is running ~40 min late (traffic at Sahara Darwaja). New ETA 7:40 AM." |
| Marketing stat | "Massive post-harvest losses plague Indian agriculture" | "₹1.5 lakh crore of produce is lost every year after harvest (NABCONS 2022)." |

Rules of thumb: farmer sentences ≤8 words where possible; one emoji max, only ✓ in success contexts; buyer copy never uses exclamation marks; every claim on the website carries a source or the word "illustrative".

## 11. Print

Payout slip (thermal 80mm + A5): forest header bar, bridge mark, farmer name/village, produce+grade+kg, **reference price struck, paid price large**, UTR, date-time, QR to the website. Grading chart poster (A2, at hub): photo grid of A vs B per crop, amber/forest headers, Gujarati + Hindi. Both use the same tokens; print is monochrome-safe (information never in color alone).

## 12. Acceptance criteria

1. All four platforms consume tokens only from their mirror files; grep for raw brand hexes in feature code returns zero.
2. The four core components exist as reusable units in each codebase with all listed states; a screenshot grid (component × state × platform × light/dark) is attached to the design-system PR and matches `design/design-board.html`.
3. Every §9 contrast pair verified in implementation (automated check on token file where tooling allows).
4. Farmer screens pass laws F1–F10 in review checklist form; buyer screens pass B1–B7.
5. HI string rendering verified on both apps for the 5 core screens (no clipped Devanagari, line-height correct).
6. Voice table (§10) is applied: release review samples 10 random strings against the bad/good patterns.

## 13. Open questions

| # | Question | Decision taken here / owner | Deadline |
|---|---|---|---|
| Q1 | Wordmark + app icon final art | Contract designer (T1.8); interim = bridge SVG from website scaffold. Owner: Alpesh | Build week 3 |
| Q2 | Produce imagery: photos vs illustrations in the picture grid | **Decided: real photos** (trust brand; illustrations read as toys to farmers). Shot at hub during Phase 0. Owner: Alpesh | Build week 4 |
| Q3 | Gujarati type scale verification (Mukta metrics differ from Devanagari) | Test at GU-string integration. Owner: Alpesh | Before GU ships |
| Q4 | Ops dashboard density tokens (24px chips, 13px body) | Sketched here; finalize in `09-PRD-OPS-DASHBOARD.md` wireframes (T1.6). Owner: Alpesh | Build week 2 |
