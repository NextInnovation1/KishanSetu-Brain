# 19 — PRD: CMS / Analytics module (KisanSetu Console)

**Doc owner:** Founder (Alpesh) · **Status:** Draft for founder approval · **Last updated:** 2026-07-12
**Siblings:** `09-PRD-OPS-DASHBOARD.md` (the other module of the same console — one login, one codebase, one API) · `06-PRD-BACKEND.md` (every screen here maps to endpoints defined there — this doc continues its R-series with R7–R14 + S5 and amends S4) · `10-DESIGN-SYSTEM.md` (stat tile §6.4, tokens) · `18-LEGAL-COMPLIANCE.md` (§10 data protection) · `14-OPS-PLAYBOOK.md` (weekly metrics ritual).
**Gate:** every build task from this PRD is `[G]` gated on T1.7 founder PRD approval (Golden Rule 3) — same as everything else in the Brain.

---

## 1. Purpose

The ops module (`09-PRD-OPS-DASHBOARD.md`) runs **today**: grade, allocate, deliver, pay. This module answers **what happened and where is it going**: who sold what, who bought what, how many people downloaded and use the apps, and whether the two golden metrics (freshness, farmer share — Golden Rule 5) are trending the right way. It is the founder's Friday ritual, the investor-update generator, and the early-warning system for buyer churn (Golden Rule 6: buyer repeat-rate is the retention metric that matters).

**Product decision (canon):** there is ONE internal web app — the **KisanSetu Console** — with two modules sharing one OTP login, one vanilla-JS codebase, and one API. This doc specifies the CMS/Analytics module: five screens, C-1 through C-5. Any `role='ops'` user sees both modules at MVP (pilot team = founder + Ops Lead); per-screen permissions are a v2 non-goal. English-only at MVP, like the ops module. Desktop-first, fully usable at 768 px. No native app.

Same reality constraint as the sibling: analytics must never block operations. Graphs read a nightly rollup, never the hot tables; if the rollup fails, ops screens are untouched and this module shows a dated banner, not wrong numbers.

## 2. Users & jobs-to-be-done

| User | Context | Jobs |
|---|---|---|
| Founder | Laptop, Friday ritual + before investor calls | C-1 trends, C-3 buyer health, C-5 export for updates |
| Ops lead | Laptop at hub, weekly | C-2 farmer follow-ups (inactive farmers, data-quality flags), weekly downloads entry on C-4 |
| Field sales (read) | Phone browser | C-3 buyer list before a visit (repeat-rate, outstanding) |

## 3. Scope (in) / NON-GOALS

**In (MVP):** C-1 Overview, C-2 Farmers, C-3 Buyers, C-4 App & Users basics (DAU graph + manual downloads entry); stored app events + nightly rollup; endpoints R7–R13 + S5. **v1.1:** C-5 Export (R14), retention cohorts, WAU/MAU graph polish.

**NON-GOALS (explicit):**

| Not building | Why |
|---|---|
| Store-API auto-sync (Play/App Store Connect) | Manual weekly entry into `store_metrics` is 10 minutes of work at pilot scale; the two store APIs are a v2 integration. |
| Per-screen permissions | Pilot team is 2–3 trusted staff; all `role='ops'` see both modules. Role editor is a v2 item, same as in `09-PRD-OPS-DASHBOARD.md` §3. |
| Chart library / CDN anything | Site performance budget + CSP posture (no third-party origins). One hand-rolled SVG chart helper (§8) — line + bar, ~150 lines, design-system tokens. |
| Realtime / websockets | Nightly rollup + a visible Refresh button. Analytics that is 24 h old is fine; analytics that breaks is not. |
| Farmer/buyer-facing analytics | This console is internal staff only. Buyer-facing analytics is a Phase 3 idea in `07-PRD-MOBILE-APPS.md` territory, not here. |
| BI suite / SQL explorer | Five fixed screens + CSV export. Real BI waits for real volume. |

## 4. Information architecture & the week in the life

The console header gains a module switcher: **Ops | Analytics**. Analytics left nav: **Overview · Farmers · Buyers · App & Users · Export**. Header keeps region name + business date (region tz), user, Refresh with "updated n s ago". All money uses the region currency from `GET /me` — no hardcoded ₹ (Golden Rule 2); every query is region-scoped by `region_id`.

| When | Who | Screen | Action |
|---|---|---|---|
| Friday AM | Founder | **C-1** | Metrics ritual: GMV/orders/DAU trend, farmer-share % vs 60% target; paste numbers into weekly update. |
| Friday AM | Founder | **C-3** | Sort by last order desc → who hasn't reordered in 14 days → call list for field sales. |
| Monday | Ops lead | **C-4** | Enter last week's downloads from Play Console + App Store Connect (2 numbers each). |
| Monthly | Ops lead | **C-2** | Data-quality sweep: farmers with no-shows or dispute-traced crates → FPO conversation. |
| Before investor call | Founder | **C-5** | Export metrics CSV for the date range; the export itself is audited. |

## 5. Metric definitions (stated once — every screen, endpoint and export uses exactly these)

| Metric | Definition | Source |
|---|---|---|
| **DAU** | Distinct `user_id` with ≥1 `app_events` row that calendar day (region tz) | `app_events` → `metrics_daily.dau` |
| **WAU / MAU** | 7-day / 30-day rolling distinct `user_id`, same event basis | `app_events` (computed in R13) |
| **Active farmer** (business sense) | ≥1 **graded** listing in the period | `listings` |
| **Active buyer** (business sense) | ≥1 **delivered** order in the period | `orders` |
| **Downloads** | Store-reported installs, entered weekly per platform | `store_metrics` |
| **Signups** | Rows in `users` by role and created date — **never conflated with downloads**; C-4 shows both, labelled | `users` |
| **GMV** | Σ delivered order totals, gross (credit notes shown separately, never netted silently) | `orders` → `metrics_daily.gmv` |
| **AOV** | GMV ÷ delivered orders in period | derived |
| **Repeat-rate** | Buyers with ≥2 delivered orders in window ÷ active buyers (canonical definition: `06-PRD-BACKEND.md` §6.10, served by R11) | `orders` |
| **Farmer-share %** | Σ farmer payouts ÷ Σ delivered produce subtotal (identical to R3 — one formula, one place) | `payments`/`orders` |

DAU/MAU measure **app engagement**; active farmer/buyer measure **business activity**. A farmer who opens the app daily but never sells is DAU, not an active farmer. Screens label which one they show.

## 6. Functional spec (per screen)

### 6.1 C-1 Overview — the pulse

Data: R7 `GET /reports/overview` (today's tiles, computed live) + R8 `GET /reports/metrics/daily` (graph series from `metrics_daily`).

Layout sketch (desktop; tiles wrap 2-per-row at 768 px, graphs stack):

```
Overview — Surat · Wed 2026-11-11            [7d] [30d] [90d]        [Refresh]
┌─────────┬─────────┬─────────┬──────────┬─────────┐
│ ₹41,200 │ 18      │ 640 kg  │ 23 / 11  │ 57      │   ← stat tiles (10-DESIGN-
│ GMV     │ orders  │ moved   │ actv F/B │ DAU     │      SYSTEM.md §6.4)
├─────────┼─────────┼─────────┼──────────┼─────────┤
│ 4       │ 2       │ 6       │ ₹18,300  │         │
│ signups │ open    │ pending │ payout   │         │
│ today   │ disputes│ payouts │ amount   │         │
└─────────┴─────────┴─────────┴──────────┴─────────┘
┌─ GMV / day (line) ────────┐  ┌─ Orders / day (bar) ──────┐
│        ╱╲    ╱─╲          │  │  ▂ ▄ ▆ ▅ ▇ ▆ █            │
└───────────────────────────┘  └───────────────────────────┘
┌─ DAU / day (line) ────────┐  ┌─ Farmer-share % / wk ─────┐
│   ╱─╲╱╲╱                  │  │ ──────60% target─────     │
└───────────────────────────┘  └───────────────────────────┘
Graphs show data through yesterday (nightly rollup); tiles are live.
```

- Tiles (today, live from R7): GMV, orders, kg moved, active farmers/buyers today, DAU, new signups, open disputes, pending payouts (count + amount). Disputes and payouts tiles deep-link into the **ops module** screens — the two modules are one console.
- Four fixed graphs: **GMV/day** (line), **orders/day** (bar), **DAU/day** (line), **farmer-share %/week** (line with a rendered 60% target rule — Golden Rule 5 gets a first-class graph, as does freshness via the median-freshness series available in R8). One shared 7/30/90-day range toggle drives all four (one R8 call, `metrics=gmv,orders,dau,farmer_share`).
- **States:** loading = shimmer tiles + empty chart frames with axis only. No data yet (pre-pilot) = tiles show "—" per §6.4 stat-tile law (0 and unknown are different facts) + "Metrics begin with the first delivered order." Error = red banner + Retry; tiles keep last-good values with a "stale" amber dot. Rollup missed = amber banner "Graphs current through {last rollup date}".
- **Acceptance:** (1) founder reads GMV, DAU and farmer-share trend in one glance, no scrolling on a 13" laptop; (2) range toggle re-renders all four graphs in <1 s at pilot volumes with a single R8 call; (3) farmer-share graph always renders the 60% rule and colors the series point green/amber against it; (4) tile numbers match R7 JSON exactly — no client-side arithmetic on money.

### 6.2 C-2 Farmers — who sold what

Data: R9 `GET /reports/farmers?sort=&period=&limit=&offset=` (list) + R10 `GET /reports/farmers/:id` (drill-down).

**List:** columns **name · village · hub · listings · kg graded A / B · payouts total · farmer-share % · last active**. Default sort: payouts total desc; every numeric column sortable (server-side, `sort=` param); period selector (30 d default / 90 d / all); search by name or phone; 50 rows per page, limit/offset. Row click → drill-down.

**Drill-down** layout sketch:

```
← Farmers   Ramesh Patel · Kathor · Hub: Olpad FPO · +91 98765 xxxxx [call]
┌ Lifetime ─────────────────────────────────────────────────────────┐
│ ₹1,84,300 paid · 6,120 kg graded (A 72%) · share 64% · since Nov  │
└───────────────────────────────────────────────────────────────────┘
┌ Monthly kg + earnings (12 mo, bar + line) ────────────────────────┐
│ ▄ ▆ █ ▅ ▇ ...     ── earnings line ──                             │
└───────────────────────────────────────────────────────────────────┘
┌ Data-quality flags ───────────────────────────────────────────────┐
│ ⚠ 2 no-shows (expired listings) · 1 dispute traced to his crates  │
└───────────────────────────────────────────────────────────────────┘
Sell history (every listing)
│ date · crop · posted kg · graded kg · A/B split · payout ₹ + UTR │
```

- Sell history is complete: one row per listing — date, crop, posted/graded kg, grade split, status, payout amount + UTR (link to the payment row). Paginated at 50.
- Monthly trend graph: kg graded (bar) + earnings (line), 12 months, SVG helper.
- Data-quality flags: no-shows (listings closed `expired_no_show`, from the `06-PRD-BACKEND.md` §6.6 sweep), disputes traced to this farmer's crates (via `order_allocations` → `disputes`), grading rejects. Flags are facts with counts and links — never a "score".
- **States:** loading skeleton rows; empty list (period has no activity) = "No graded listings in this period" with period widening hint; farmer with zero payouts renders "—" not ₹0; error banner + retry keeps the filter state.
- **Acceptance:** (1) "top 10 farmers by payout this month" answered in ≤2 interactions (default view + period); (2) drill-down payout total equals Σ of the visible sell-history rows (same query, by construction); (3) UTR on every paid row matches the ops Payouts screen for the same payment; (4) a flagged no-show links to the exact expired listing.

### 6.3 C-3 Buyers — who bought what

Data: R11 `GET /reports/buyers?sort=&period=&limit=&offset=` + R12 `GET /reports/buyers/:id`.

**List:** columns **business · type · orders · GMV · AOV · repeat-rate · credit status / outstanding · disputes**. Default sort: GMV desc; period selector as C-2. Credit column shows outstanding amount + `credit_hold` red chip when true (mirrors R5 verdict — one source of truth). A "no order in 14 d" amber dot flags churn risk (the metric that matters, Golden Rule 6).

**Drill-down:** identity header (business, type, contact, credit terms) · purchase history table (every order: date, items by crop+grade with grade chips, value, status, invoice no) · monthly GMV trend graph (12 mo) · credit/receivables panel (outstanding, exposure vs cap, aging buckets, credit notes — same numbers as R5) · dispute history (date, crop, resolution, credit-note value, link to the dispute).

- **States:** loading/error as C-2; pending buyer (never ordered) shows "no delivered orders yet" not zeros; credit panel for a cash buyer collapses to "no credit extended".
- **Acceptance:** (1) repeat-rate in the list equals the canonical `06-PRD-BACKEND.md` §6.10 formula for the same window (shared definition §5); (2) drill-down GMV = Σ visible delivered orders, gross, with credit notes itemized separately; (3) `credit_hold` chip here always agrees with the order-blocking guard in `06-PRD-BACKEND.md` §6.7; (4) invoice numbers match the ops Deliveries screen.

### 6.4 C-4 App & Users — downloads, DAU, versions

Data: R13 `GET /reports/app-usage?from=&to=` (computed from `app_events` + `store_metrics` + `users`) + S5 `PUT /store-metrics/:date` (manual downloads entry write).

- **Downloads panel:** two labelled tiles per platform — "Play Store installs" / "App Store installs" — plus a **weekly entry form** (date-week picker, downloads_total, downloads_new per platform → S5 `PUT /store-metrics/:date`, `source='manual'`, `entered_by` stamped). Beside them, "Signups (farmer/buyer)" tiles from `users`. Downloads and signups sit side-by-side, both labelled with their §5 definitions — the screen never conflates them, and shows the ratio as "install → signup" for the funnel.
- **DAU/WAU/MAU:** DAU/day graph (MVP); WAU + MAU lines added v1.1. Uses stored `app_events` (S4 amendment, §7) — without stored events this screen is impossible, which is why the amendment ships in the MVP cut.
- **Signups by role over time:** stacked bar, farmer vs buyer, weekly buckets.
- **Version adoption:** table per platform — app_version, users on it (distinct from events, last 7 d), % — with the row < `region_settings.min_supported_version_{android|ios}` for the row's platform highlighted red ("these users must update").
- **Platform split:** Android/iOS share of DAU (single bar). Parity check: a persistent gap here is a Golden Rule 1 smell worth investigating, and the caption says so.
- **Retention cohorts (v1.1):** W1/W4 signup cohorts, farmer and buyer separately.
- **States:** no store_metrics rows yet = downloads tiles "—" + "enter this week's numbers" prompt; no events yet = DAU graph empty-state explaining events start at first app release; error as C-1.
- **Acceptance:** (1) weekly downloads entry takes <2 minutes and appears in the tiles immediately; (2) DAU for any past day matches a hand-run distinct count on `app_events` for that region-tz day; (3) version-adoption red rows agree with the `min_supported_version` the apps enforce; (4) downloads and signups are never summed, ratioed silently, or shown in one tile.

### 6.5 C-5 Export — audited CSV (v1.1)

Data: R14 `GET /reports/export?entity=farmers|buyers|metrics&format=csv&from=&to=`.

- Entity picker (Farmers list / Buyers list / Metrics time series) + date-range picker (presets: last 7/30/90 d, custom). Columns are exactly the on-screen columns of C-2/C-3 (plus ids) or the R8 series for metrics — no bespoke export shapes.
- Download streams the CSV (server-side, no client assembly). **Every export writes an `audit_events` row `report_exported`** with entity, range, row count and user — shown back to the user in a small "recent exports" list on this screen (transparency breeds discipline).
- A confirm note above the button: "This file contains personal data of farmers/buyers. Handle per `18-LEGAL-COMPLIANCE.md` §10 — no forwarding outside the team."
- **States:** empty range = disabled button + explanation; oversized range streams anyway (server streams; no timeout at pilot volumes); failure mid-stream = browser-level, retry regenerates.
- **Acceptance:** (1) exported farmer CSV row count equals the C-2 list total for the same filters; (2) every download produces exactly one `report_exported` audit row, verified in tests; (3) CSV opens clean in Excel/Sheets (UTF-8 BOM, ISO dates, unformatted numbers).

## 7. Backend contract (amendments + new surface for `06-PRD-BACKEND.md`)

**S4 amendment — events are now STORED, not just logged.** `POST /events` keeps its transport semantics (batch, fire-and-forget, `202`) but writes thin rows to a new table `app_events (id, user_id nullable, event_name, platform android|ios, app_version, region_id, ts timestamptz, props jsonb)`, indexed on `(ts)`, `(user_id, ts)`, `(event_name, ts)`. Retention 180 days; a nightly job then deletes raw rows older than that, keeping aggregates only (they are already in `metrics_daily`). This powers DAU/MAU; without stored events C-4 is impossible.

**New tables:** `store_metrics (region_id, date, platform, downloads_total, downloads_new, source manual|api, entered_by)` — manual weekly entry at MVP (store APIs are v2, §3); `metrics_daily (date, region_id, gmv, orders, kg_moved, dau, new_farmers, new_buyers, farmer_share_pct, median_freshness_h)` — filled by a **nightly rollup job at 00:30 region tz**. Graphs read `metrics_daily`, never raw tables; a rerun of the rollup for a date is idempotent (upsert).

**New endpoints** (all `ops` role; R-series IDs continue `06-PRD-BACKEND.md` §6.10; S5 lives in its §6.11 S-series):

| # | Method & path | Purpose | Feeds |
|---|---|---|---|
| R7 | `GET /reports/overview` | Today snapshot, computed live | C-1 tiles |
| R8 | `GET /reports/metrics/daily?metrics=gmv,orders,dau,farmer_share&from=&to=` | Time series from `metrics_daily` | C-1 graphs, C-5 metrics export |
| R9 | `GET /reports/farmers?sort=&period=&limit=&offset=` | Farmer list with §6.2 columns | C-2 list |
| R10 | `GET /reports/farmers/:id` | Sell history, monthly trend, flags | C-2 drill-down |
| R11 | `GET /reports/buyers?sort=&period=&limit=&offset=` | Buyer list with §6.3 columns | C-3 list |
| R12 | `GET /reports/buyers/:id` | Purchase history, GMV trend, credit panel, disputes | C-3 drill-down |
| R13 | `GET /reports/app-usage?from=&to=` | DAU/WAU/MAU, version adoption, platform split | C-4 |
| R14 | `GET /reports/export?entity=farmers\|buyers\|metrics&format=csv&from=&to=` | Streams CSV; writes `audit_events` `report_exported` | C-5 (v1.1) |
| S5 | `PUT /store-metrics/:date` | Manual weekly downloads entry (upsert on (region_id, date, platform)) | C-4 entry form |

## 8. Technical notes (binding for the build)

- **One codebase:** same static app as the ops module (`11-ARCHITECTURE.md` §7) — the module switcher is client routing, not a second deploy. Shared `api.js`, shared tokens, shared chip/tile CSS.
- **Charts:** one hand-rolled SVG helper (`charts.js`, ~150 lines): line + bar only, axis + gridlines, target-rule support (the 60% line), tabular-numeral labels, colors from `10-DESIGN-SYSTEM.md` tokens (`forest` series, `leaf-deep` positive, `amber` warning, `stone-200` grid), dark-mode aware via the semantic tokens. No chart library, no CDN, ever (§3).
- **Stat tiles** are the design system's §6.4 component verbatim, including the "— when unknown, 0 only when truly zero" law.
- **Polling:** none by default — analytics refreshes on navigation and on the Refresh button; C-1 tiles may poll at 60 s when visible. Nightly-rollup data cannot change intra-day, and the button says so ("graphs update nightly").
- **Region scoping:** every endpoint takes/derives `region_id`; a second region appearing must require zero schema or screen changes (Golden Rule 2).

## 9. Edge cases & error states

| Case | Behavior |
|---|---|
| Nightly rollup failed | Amber banner on C-1/C-4 with last successful rollup date; tiles (live R7) unaffected; ops module unaffected by construction. Rerun is idempotent. |
| Events flood / bad client loop | S4 keeps its per-user rate limit; dropped rows counted in logs; DAU unaffected (distinct user, not volume). |
| Store numbers entered twice | S5 upserts on (region_id, date, platform); the form shows the existing value before overwrite. |
| Farmer/buyer deleted or blocked | Rows remain in history (facts don't vanish); name renders with a "blocked" chip. |
| Timezone edges | All day-bucketing in region tz, everywhere, including DAU (§5) — never UTC midnight. |
| Export of a huge range | Server streams; client shows browser download progress; no in-memory assembly on either side. |

## 10. Privacy & data protection (DPDP)

These screens expose farmer and buyer operational data to internal staff — covered by the DPDP service purpose documented in `18-LEGAL-COMPLIANCE.md` §10. Binding rules: (1) console access is authenticated ops staff only; (2) **no per-farmer/per-buyer data leaves the console except via the audited CSV export** (R14 → `report_exported`); (3) raw `app_events` retention is 180 days, aggregates only thereafter; (4) `app_events.props` never contains free-text personal data — event names and enum props only, enforced by the S4 allowlist; (5) the C-5 screen carries the handling note verbatim (§6.5).

## 11. Phased rollout

| Phase | Contents |
|---|---|
| **MVP** | C-1, C-2, C-3 complete + R7–R12 (thin queries over existing tables); S4 stored-events amendment + `app_events`/`store_metrics`/`metrics_daily` + nightly rollup; C-4 basics (DAU graph, manual downloads entry via S5/R13). |
| **v1.1** | C-5 export (R14 + audit), W1/W4 retention cohorts, WAU/MAU graphs polish. |
| **v2** | Store-API auto-sync (`store_metrics.source='api'`), per-screen permissions, threshold alerting (e.g. farmer-share week < 60% pings the founder). |

## 12. Instrumentation of the console itself

**Decision:** the web console does **not** emit S4 events — S4 stays mobile-only (`platform ∈ android|ios`), and the §5 DAU definition stays unfiltered and uninflated. Why: keeping S4 mobile-only keeps the C-1/C-4 DAU graphs honest without a role-filter join. Console usage is derived from what already exists instead: `audit_events` (`report_exported` via R14, `store_metrics_entered` via S5, plus the other audited ops actions) and server request logs (`request_id`, `user_id`, `role` — `06-PRD-BACKEND.md` §11). Reviewed monthly: which CMS screens are actually opened, per the request logs (unused screens get cut, not polished), export frequency, and rollup-failure count (>1/month → rollup gets a retry + alert before v2's alerting arrives generally).

## 13. Open questions (owner + deadline)

| # | Question | Provisional decision | Owner | Deadline |
|---|---|---|---|---|
| Q1 | Who enters weekly store downloads, and which day? | Ops lead, Mondays, from Play Console + App Store Connect; founder covers until hire | Founder | Pilot week 1 |
| Q2 | Retention cohorts at MVP or v1.1? | v1.1 (canon: "v1.1 if heavy" — it is heavy: cohort SQL + a matrix component) | Founder | v1.1 planning |
| Q3 | GMV gross vs net of credit notes | Gross, credit notes itemized separately (§5) — silent netting hides dispute cost | Founder | T1.7 approval |
| Q4 | Store-metrics write path + S4 amendment adoption into `06-PRD-BACKEND.md` | Resolved — adopted as S5 `PUT /store-metrics/:date` in 06 (rev 2026-07-12) | Founder | Done |
