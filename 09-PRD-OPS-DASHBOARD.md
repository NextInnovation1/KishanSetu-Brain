# 09 — PRD: Ops Dashboard (internal)

**Doc owner:** Founder (Alpesh) · **Status:** Draft for founder approval · **Last updated:** 2026-07-12
**Siblings:** `06-PRD-BACKEND.md` (every screen here maps to endpoints defined there — endpoint IDs like F2, L6, O11 refer to its §6 tables) · `14-OPS-PLAYBOOK.md` (the human SOPs this tool digitizes) · `10-DESIGN-SYSTEM.md` (palette/type) · `05-PRODUCT-OVERVIEW.md`.

---

## 1. Purpose

**This is the tool the business runs on.** Before a single farmer or buyer opens an app, the ops dashboard must let one person run a full day: set prices the evening before, grade the morning intake, match graded supply to orders, push deliveries through their statuses, pay farmers, answer leads, and read the two metrics (freshness hours, farmer share — Golden rule #5). Build order in `12-DEVELOPMENT-PLAN.md` puts this **before** the mobile apps for exactly that reason.

Reality constraint (from `PLAN.md` heritage, kept as law): *the software must never block a delivery.* Every action here has a manual fallback documented in `14-OPS-PLAYBOOK.md`; the dashboard records reality, it does not gate it.

## 2. Users & jobs-to-be-done

| User | Context | Jobs |
|---|---|---|
| Founder (pilot months 1–3) | Laptop, evening at home + tablet at hub 05:00–08:00 | Everything below |
| Ops lead (hire #1, month 2) | Tablet/laptop at the hub, standing, gloves sometimes off | Grading queue, allocation, delivery statuses, payouts |
| Field sales (hire #2, month 3) | Phone browser between visits | Lead inbox only |

Environment truths that shape the UI: used at 05:30 in a shed with patchy 4G; big tap targets, minimal typing, obvious state colors, and every list refreshable in one tap. Desktop-first layout (13" laptop), fully usable at 768 px width (tablet portrait). No native app — this is a web app.

## 3. Scope (in) / NON-GOALS

**In (MVP):** OTP login (ops role), Today digest, Daily Price Setting, Grading Queue, Allocation Board, Deliveries, Payouts, Reports (SLA + farmer share + fulfilment), Lead Inbox, Partners (hubs/FPOs/users) admin, audit visibility.

**NON-GOALS (explicit):**

| Not building | Why |
|---|---|
| React/Vue/build pipeline | Founder-fixed stack: vanilla JS, no build step (same discipline as `08-PRD-WEBSITE.md`). One `index.html` + ES modules + `fetch`. Zero deploy friction. |
| Real-time sync (websockets) | 1–2 concurrent ops users at pilot. 30 s polling + a visible "Refresh" button is sufficient and debuggable. |
| Role/permission editor | Roles are fixed; ops accounts are seeded (`06-PRD-BACKEND.md` §6.2). |
| Charts library / BI suite | Reports are tables with big numbers. Export = copy-paste or CSV later. Real BI waits for real volume. |
| Mobile native ops app | Browser on tablet covers the hub; offline-first ops tooling is Phase 3 at the earliest. |
| Route planning / 3PL integration | 3PL owns routes (Golden rule #4, asset-light). We record statuses only. |
| Printable label designer | Crate QR labels are generated from a fixed template (Phase 2); design tool never. |

## 4. Information architecture & a day in the life

Left nav (order = daily workflow order): **Today · Prices · Grading · Allocation · Deliveries · Payouts · Leads · Reports · Partners**. Header: region name + business date (region tz), logged-in user, Refresh button with "updated 12 s ago" stamp. All money rendered with the region's currency symbol from `GET /me` — no hardcoded ₹ in code (Golden rule #2).

The dashboard exists to run this loop (times are Surat pilot defaults; SOP detail in `14-OPS-PLAYBOOK.md`):

| Time (region tz) | Who | Screen | Action |
|---|---|---|---|
| 17:00–18:00 (D−1) | Founder/ops | **Prices** | Check reference market prices from the mandi shadow/contacts → copy yesterday → adjust → publish tomorrow's price feed. Farmer apps update; WhatsApp price broadcast goes out (manual at MVP). |
| Evening (D−1) | Farmers | (farmer app) | Post listings for tomorrow's harvest against published prices. |
| By 18:00 (D−1) | Buyers | (buyer app) | Place orders for tomorrow's delivery (canonical cutoff, [13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md) §2.1). |
| 05:00–07:00 (D) | Ops at hub | **Grading** | Farmers arrive; weigh, grade A/B (split where needed); farmer notified with graded qty. |
| 06:30–07:15 (D) | Ops at hub | **Allocation** | "Suggest all" FIFO match → review → fix shortfalls (call buyers / edit items). |
| 07:15–07:45 (D) | Ops at hub | **Deliveries** | Confirm orders → mark out-for-delivery as 3PL loads (stamps `dispatch_ts`). |
| 07:30–08:30 (D) | Ops | **Payouts** | Trigger farmer payouts at pickup; pay via UPI; enter UTR. The payout slip is the ad (`04-GTM-SALES-MARKETING.md`). |
| 08:00–11:00 (D) | Ops | **Deliveries** | Mark delivered as 3PL confirms (stamps `delivered_ts`, issues invoice number). |
| Midday (D) | Field sales | **Leads** | Work new leads from website/Meta ads between visits. |
| Friday | Founder | **Reports** | Weekly metrics ritual: freshness median, farmer share, fulfilment, repeat rate. |

If the dashboard is unusable at any step, the paper fallback in `14-OPS-PLAYBOOK.md` applies and data is entered afterwards — reality first, records second, but always both.

## 5. Functional spec (per screen)

### 5.1 Login
OTP flow via A1/A2 (`06-PRD-BACKEND.md`). If the verified user's role ≠ `ops` → "This console is for KisanSetu staff." No signup path.
**Acceptance:** ops user in ≤ 3 interactions; farmer/buyer token can never render any page (role checked client-side for UX and server-side for truth on every call).

### 5.2 Today (daily digest)
Data: R1 `GET /reports/summary?date=today`. Cards: orders by status (placed/confirmed/out/delivered), kg posted vs graded, unallocated order items (red if > 0 after 06:30), pending payouts count + amount, fulfilment % yesterday, median freshness h (7-day), farmer share % (7-day). Each card links to its screen. Amber banner if today's prices are unpublished by 18:00 for tomorrow.
**Acceptance:** one glance answers "what's on fire?"; every number click-throughs to the working screen; loads < 2 s on hub 4G.

### 5.3 Prices (daily price setting) — the evening ritual
The SOP it implements: `14-OPS-PLAYBOOK.md` "Daily price-setting" (heritage T6.6).

Layout sketch (desktop; columns collapse into a two-line row on tablet):

```
Prices for [ 2026-11-07 ▾ ]           [Copy yesterday]  [Save draft]  [PUBLISH]
┌────────────┬─────────────┬────────┬────────┬────────┬────────┬───────────┬──────────┐
│ Produce    │ Mandi price*│ Plat A │ Plat B │ Farm A │ Farm B │ Share A/B │ 7d range │
├────────────┼─────────────┼────────┼────────┼────────┼────────┼───────────┼──────────┤
│ टमाटर Tomato│  [ 32.00 ]  │[30.00] │[24.00] │[21.00] │[16.00] │ 70% / 67% │ 28–35    │
│ प्याज़ Onion │  [ 24.00 ]  │[22.00] │[18.00] │[15.00] │[12.00] │ 68% / 67% │ 22–26    │
└────────────┴─────────────┴────────┴────────┴────────┴────────┴───────────┴──────────┘
* column header uses region.reference_market_label — "Mandi price" only in India
```

- Date selector defaults to **tomorrow** (region tz). Table: one row per active produce.
- Columns: produce (local name + EN) · **reference market price** (labelled with `region.reference_market_label` — "Mandi price" in Surat UI) · platform A · platform B · farmer A · farmer B · **farmer-share A/B %** (computed live, green ≥ 60%, amber < 60%) · 7-day sparkline text (min–max from F4).
- "Copy yesterday" button prefills all rows (F4), then ops edits reference price + adjusts.
- Save = F2 (draft, invisible to users). **Publish** = F3 with confirm dialog showing: rows count, any share < 60% ("Golden rule #5: farmer share target is 60%. Publish anyway?"), any produce with open listings but missing row.
- After first order references a row, the row locks with a lock icon and tooltip pointing to the correction SOP (server enforces via 409).

**Acceptance:** full 24-produce day priced in < 5 minutes using copy-forward; farmer share % visible before publish, sub-60% requires explicit confirm; a locked row visibly explains why; draft never leaks to apps (verified against F1 as farmer token in tests).

### 5.4 Grading Queue — the 05:00 hub screen
Data: L3 `GET /listings?hub_id=&status=posted&harvest_date=today`.

- Filter chips: hub (default: user's last-used), harvest date (default today), produce.
- Card per listing: farmer name + village + phone (tap-to-call), produce, posted qty, listing age. Sorted oldest-first.
- Tap → grading form (L6): grade A/B big toggle, weighed qty numeric pad, optional split ("also graded: [B] [qty]"), reject-all path (L7 close with reason picker: spoiled / no-show / quality-fail).
- On submit: card flips to a green "Graded — A 70 kg · B 30 kg" state with the farmer-notification confirmation; payout estimate shown.
- Blocked state: if no published price for today (422 from L6), banner links to Prices screen.

**Acceptance:** grade-with-split completes in ≤ 5 taps + one number entry; two devices grading the same listing → second sees a clean "already graded by <name>" message (409 handled); rejected listing records reason and creates no payout; queue count in nav badge matches L3 total.

### 5.5 Allocation Board — the matching heart
Data: O11 `GET /allocation_board?delivery_date=`. Layout sketch:

```
Allocation — delivery 2026-11-08          [Suggest all]     Shortfalls: Tomato A −12 kg ⚠
┌─ DEMAND (order items) ──────────────┐   ┌─ SUPPLY (graded listings: Tomato · A) ────┐
│ ▸ Tomato · A   ordered 75 │ alloc 63│   │ Ramesh (Kathor) 70 kg · 6 h old  [assign] │
│   · Hotel Gateway   25/25 ✓         │   │ Suresh (Olpad)  40 kg · 9 h old  [assign] │
│   · Spice Villa     38/50  −12 ⚠    │   │ Meena (Kamrej)  15 kg · 4 h old  [assign] │
│ ▸ Onion · B    ordered 30 │ alloc 30│   │                                           │
└─────────────────────────────────────┘   └───────────────────────────────────────────┘
Selecting a demand row filters supply to same produce+grade; [assign] prompts qty.
```

Two panes in detail:

- **Left — demand:** order items for the delivery date, grouped by produce+grade, each showing ordered qty, allocated qty, remaining (red badge), buyer business name.
- **Right — supply:** graded unallocated listings matching the selected demand row (same produce+grade), each showing farmer, hub, graded qty, remaining qty, harvest_ts age in hours (freshness-sorted, oldest first — FIFO fights spoilage).
- Select an order item → matching listings highlight → tap a listing → qty prompt (prefilled with `min(remaining_demand, remaining_supply)`) → O8 creates the allocation. Undo chip on each allocation (O9, disabled after dispatch).
- **"Suggest all"** button: server-side greedy FIFO match per produce+grade (part of O11 response as `suggestions`), applied one-click, each still individually undoable.
- Unmatchable demand (no supply) surfaces a shortfall panel: "Tomato A short 12 kg across 2 orders" → links to the order for ops to edit down or call the buyer (SOP in `14-OPS-PLAYBOOK.md`).

**Acceptance:** a 10-order, 30-listing morning fully allocated in < 10 minutes; suggestion pass allocates everything allocatable (greedy FIFO, deterministic, unit-tested server-side); grade/produce mismatch is impossible through the UI *and* rejected by the server; every allocation immediately visible in the order's traceability (O5).

### 5.6 Deliveries
Data: O4 by status columns: **Placed → Confirmed → Out for delivery → Delivered** (kanban-ish columns, buttons not drag — drag is fiddly on tablets).

- Order card: buyer, items summary, total, allocation completeness (✓ or "2/3 allocated").
- Advance buttons call O7; "Out for delivery" is disabled until fully allocated (server enforces too) and stamps `dispatch_ts`; "Delivered" stamps `delivered_ts`, assigns invoice number, and shows it on the card.
- Cancel (with reason) available per lifecycle rules; released allocations return to the board.
- Per-allocation timestamp editor (O10) behind a "fix timestamps" link — for backfilling when the 3PL driver called instead of the app being updated live; every manual edit flagged in audit and in SLA report's `incomplete/backfilled` counts.

**Acceptance:** status can only move along the legal lifecycle (illegal button never rendered; server 409 still handled gracefully); delivering an order produces invoice_no + buyer_invoice payment row visible in Payouts screen; SLA chain complete for ≥ 95% of allocations in a normal day without using the backfill editor.

### 5.7 Payouts — the trust machine
Data: M2 worklists in two tabs: **Farmer payouts** / **Buyer invoices**.

- Farmer tab: graded listings without payout (join surfaced by backend as `payable_listings` in M2 response) → "Create payout" (M3) shows computed amount + formula (`70 kg × ₹21.00 (A, 7 Nov)`); pending payouts list → "Mark paid" (M4) with UTR field (required, min 12 chars) — MVP is manual UPI from the founder's business account per `14-OPS-PLAYBOOK.md`. Phase 2 swaps the button for "Pay via UPI" (M5) with the same list semantics.
- Buyer tab: delivered orders' invoices, mark-paid with reference when collection lands (net-7 per `06-PRD-BACKEND.md` Q2).
- Day-end strip: "Today: 14 payouts · ₹38,400 · avg 2.1 h grade→paid" (the payout-slip-as-advertisement metric — `04-GTM-SALES-MARKETING.md`).

**Acceptance:** payout amount is never typed by a human — always server-computed; UTR mandatory to settle; double-payout impossible (409 surfaced as "already paid"); farmer sees the payment in their app (M1) the moment M4 succeeds; daily payout total reconciles against bank statement line-count in the weekly ritual.

### 5.8 Leads inbox
Data: D2/D3. Table: name, E.164 phone (tap-to-call + WhatsApp deep link `wa.me/<phone>`), business type, message, source, age. Status chips: new → contacted → sampled → converted → dropped; assignee picker; free-text notes. Default filter `status=new`, sorted oldest-first. New-lead count badge in nav.
**Acceptance:** lead from website form appears within one poll cycle (≤ 30 s); every status change audited; zero leads lost (total in UI = D2 total = rows in table, checked in weekly ritual).

### 5.9 Reports
Data: R1/R2/R3.
- **SLA report:** date range picker (default last 7 days) → median + p90 harvest→door overall and per produce; breach list (> 48 h) with order links; incomplete-chain count with listing links ("fix your timestamps" queue). Big number styled green < 36 (target), amber 36–48 (promise), red > 48.
- **Farmer share:** Σ payouts ÷ Σ delivered produce subtotal, overall + per produce + weekly trend table. Green ≥ 60%.
- **Fulfilment:** ordered vs delivered kg, cancellation reasons histogram, buyer repeat rate (buyers with ≥ 2 delivered orders in window ÷ active buyers).
- Every report has "Copy as text" (tab-separated) for pasting into the weekly WhatsApp update / investor notes.

**Acceptance:** the two golden metrics render on one screen without scrolling; numbers match hand-run SQL from `reports/weekly.sql` (same queries, by construction — the endpoints run those queries); date-range change < 1 s at pilot volumes.

### 5.10 Partners (admin)
CRUD-lite for hubs, FPOs (U8/U9), user search + block (U5–U7), produce catalog + translations (P2), region settings view (read-only at MVP — new regions are seeded, `11-ARCHITECTURE.md` §7).
**Acceptance:** onboarding a new farmer at a demo day (create user via signup on their phone, then ops verifies profile + hub) takes < 2 minutes; blocking a user takes effect on their next API call.

## 6. Technical notes (binding for the build)

- **Stack:** static files served by the same Caddy as the API (`11-ARCHITECTURE.md` §7): `ops/index.html` + ES-module JS + one CSS file using `10-DESIGN-SYSTEM.md` tokens. No bundler, no framework, no npm dependencies in the client.
- **Auth storage:** JWT in `localStorage` (`ks_ops_token`); acceptable because the dashboard origin serves no third-party scripts and CORS allowlists only first-party origins (`06-PRD-BACKEND.md` §8). Logout = token delete.
- **Data layer:** one `api.js` wrapper around `fetch` — attaches bearer token, maps the uniform error shape to toasts, retries idempotent GETs once on network failure, exposes `pollEvery(screen, 30_000)` with backoff to 2 min when the tab is hidden.
- **State colors** (from `10-DESIGN-SYSTEM.md`): posted/placed = ink outline · graded/confirmed = forest `#14532D` · out_for_delivery = amber `#F59E0B` · delivered/paid = leaf `#22C55E` · cancelled/failed = red. Grade A chip forest, Grade B chip amber — identical semantics to the apps (Golden rule #1 extends to internal tools where concepts overlap).
- **Numbers:** all quantities/money right-aligned tabular numerals; currency symbol + thousand separators from `Intl.NumberFormat(user.language, {currency})`.
- **Accessibility floor:** every action reachable by keyboard; tap targets ≥ 44 px; no color-only state (chips carry text).

## 7. Edge cases & error states

| Case | Behavior |
|---|---|
| 4G drop mid-grade | Submit failure keeps the form filled + red retry banner; nothing optimistic — UI state only flips on 2xx. |
| Token expired (30 d) at 05:00 | Full-screen re-OTP overlay preserving the screen you were on. |
| Two ops both allocating | Second write on exhausted supply → 409 → board auto-refreshes with toast "Supply changed, re-check". |
| Backend down | Every screen has a red offline banner + the `14-OPS-PLAYBOOK.md` manual-fallback reference ("paper grading sheet, enter later"). |
| Price publish forgotten | 18:00 amber banner on Today; farmer app shows yesterday's prices with "estimates" note (see `06-PRD-BACKEND.md` §7). |
| Wrong grade entered | No re-grade endpoint at MVP: close the listing's allocations (if any), cancel-and-recreate via documented correction SOP; all steps audited. Rare by design (confirm dialog shows grade big and green/amber). |

## 8. Phased rollout

| Phase | Contents |
|---|---|
| **MVP** | §5.1–5.10 complete; manual payouts; polling refresh. This ships before the mobile apps (`12-DEVELOPMENT-PLAN.md`). |
| **Phase 2** | Razorpay payout button, crate QR label print (per allocation), invoice PDF links, WhatsApp template send buttons, CSV export on reports. |
| **Phase 3** | Hub-scoped accounts for FPO staff (the FPO-SaaS seed — `03-BUSINESS-MODEL.md`), multi-region switcher for ops, standing-order management. |

## 9. Metrics & instrumentation

Dashboard usage itself is instrumented via backend audit/events: time from listing posted → graded (ops responsiveness), grade → payout-paid latency, % allocations via "Suggest all" vs manual, price-publish time-of-day histogram, lead first-touch time. Reviewed monthly; if suggest-all usage > 80% with no undo, greedy matching graduates to auto-allocate-with-review (Phase 3 candidate).

## 10. Open questions (owner + deadline)

| # | Question | Provisional decision | Owner | Deadline |
|---|---|---|---|---|
| Q1 | Does FPO staff get dashboard access during pilot (vs KisanSetu ops only)? | Pilot: KisanSetu ops only; FPO staff use paper + phone, ops enters data | Founder | FPO MoU signing |
| Q2 | Tablet purchase vs founder laptop at hub | One rugged Android tablet (~₹15k) in hub budget | Founder | Pilot week −2 |
| Q3 | Hindi/Gujarati UI for ops dashboard | English-only at MVP (ops users are staff); revisit when FPO staff onboard (Phase 3) | Founder | Phase 3 |
