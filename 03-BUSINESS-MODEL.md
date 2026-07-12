# 03 — BUSINESS MODEL

_Doc owner: Alpesh (founder). Status: Draft v1.0 for founder approval. Last updated: 2026-07-12._
_Reads with: 02-MARKET-RESEARCH.md (market sizing & competitive wedge), 04-GTM-SALES-MARKETING.md (how we acquire the buyers this model depends on), 13-LAUNCH-PLAN.md (pilot gates), 14-OPS-PLAYBOOK.md (the physical flow these economics price), 16-RISKS-MITIGATIONS.md, 17-FUNDRAISE-FINANCE.md (burn & raise built on these numbers), 18-LEGAL-COMPLIANCE.md (APMC/GST/FSSAI constraints on pricing and invoicing)._

---

## 1. Purpose

Define exactly how KisanSetu makes money: the five revenue streams and when each switches on, the per-order contribution-margin math, the path to profitability with tens of business buyers (Golden Rule 6), and the pre-committed pivot option (FPO-SaaS-first) if Phase-0 validation fails. Every number in §4 is **illustrative until Phase 0 replaces it** — §5 is the plan to replace them.

**Non-negotiable constraints from 00-GOLDEN-RULES.md that shape this model:**

- **Asset-light (Rule 4):** no owned farms, trucks, warehouses. Our COGS is partner fees (FPO hub fee, 3PL per-drop fee), never depreciation.
- **The two metrics (Rule 5):** farmer share of final price **≥60% (published)** is a hard pricing floor, not an aspiration. Any pricing decision that pushes farmer share below 60% is rejected even if it improves margin.
- **B2B, sales-led (Rule 6):** the model must reach city-level operating break-even with tens of buyers, not thousands.
- **Global-first (Rule 2):** the model is expressed in abstractions — take rate, per-drop logistics cost, hub fee, instant-payment rail — so the same P&L template prices Surat, city #2, and country #2 (see §9).

## 2. Model in one picture

```
FARMER ──produce──▶ FPO HUB ──graded crates──▶ 3PL COLD VAN ──▶ BUYER (HoReCa)
   ▲                  (partner,                  (partner,          │
   │                   per-kg fee)                per-drop fee)     │ pays one invoice
   └────────── instant UPI payout ◀── KISANSETU (platform) ◀────────┘
                (≥60% of buyer price)      earns the spread + fees
```

KisanSetu transacts as a **principal-lite reseller**: we buy from the farmer at the published platform price (paid instantly to UPI on pickup) and sell to the buyer on a single tax invoice. This gives the buyer the one-invoice, one-throat-to-choke experience HoReCa demands, while our balance sheet stays asset-light because the goods are in our custody for hours, not days, and physical work is done by partners. (Legal structure of this — invoicing entity, GST treatment, APMC direct-procurement rules in Gujarat — is specified in 18-LEGAL-COMPLIANCE.md.)

**GMV** = buyer produce invoice value. **Net revenue (take)** = everything left after farmer payout and direct chain costs. Target blended take rate: **8–12% of GMV**.

## 3. The five revenue streams — and when each activates

| # | Stream | Who pays | Pricing (launch defaults) | Activates | Activation trigger |
|---|--------|----------|---------------------------|-----------|--------------------|
| 1 | **Transaction take rate** (core) | Buyer (embedded in produce price) | 8–12% of GMV, blended target 10% | **Month 0** (pilot day 1) | First order |
| 2 | **Handling & delivery fee** | Buyer (separate invoice line) | ₹50/order flat; waived on orders ≥₹4,000 | **Month 0** | First order |
| 3 | **Buyer premium subscription** ("KisanSetu Priority") | Buyer | ₹1,499/month (pilot price) | **Month 5–6** | ≥70% weekly repeat rate AND ≥95% fulfilment for 4 consecutive weeks |
| 4 | **FPO procurement SaaS** | FPO | Free for pilot FPO through pilot; then ₹5,000/hub/month intro | **Month 6–9** | Ops dashboard hardened + 2nd FPO signed |
| 5 | **Financial services** (input credit referral, buyer invoice financing) | Farmer/buyer via NBFC partner | 1–1.5% origination/referral fee; we never lend from our books | **Month 12+ / Year 2** | ≥6 months of payment-history data + NBFC partnership signed |

### Stream 1 — Transaction take rate (the business)

- Mechanics: ops sets `platform_price_a` / `platform_price_b` per produce per day in `price_feed` (see 06-PRD-BACKEND.md), anchored to `reference_market_price`. The farmer payout price and buyer price are both published; the spread net of chain costs is our take.
- Pricing policy: buyer price for Grade A targets **parity to −5% vs their current vendor's landed price** (we win on grading, freshness, reliability, traceability — not on being cheapest; see 04-GTM-SALES-MARKETING.md §3). Grade B priced 12–18% below Grade A — sized for caterers/cloud kitchens where cosmetic perfection doesn't matter.
- The farmer-share floor binds here: `farmer_payout / buyer_produce_price ≥ 0.60` is enforced as a check in the ops price-setting screen (09-PRD-OPS-DASHBOARD.md) and reported weekly.

### Stream 2 — Handling & delivery fee

- ₹50/order flat at launch. Deliberately below our true per-drop logistics cost (~₹280, §4) — it is a price signal that delivery is a paid service, not free forever, and it disciplines tiny orders. Minimum order value: **₹1,000** for the same reason.
- Waived: on first trial order (sales tool, see 04 §4), on orders ≥₹4,000, and during a won referral's free-delivery month.
- Reviewed monthly against route density; may become slab-based (₹0/₹50/₹100 by order value) from Month 4.

### Stream 3 — KisanSetu Priority (buyer subscription)

- What ₹1,499/month buys: (a) **priority allocation** when supply is short (priority flag consumed by the allocation queue, 09-PRD-OPS-DASHBOARD.md), (b) 7-day price lock on their top-5 SKUs, (c) monthly purchase analytics + GST-ready statement export, (d) first access to new crops, (e) delivery-fee waiver.
- Why it will sell: HoReCa's #1 pain after price is **supply reliability during festivals/weddings/season transitions** (validate in Phase-0 interviews, §5). Priority during shortage is worth more than ₹1,499 to a 100-cover restaurant.
- Guardrail: never sell more Priority seats than ~40% of active buyers per city, or priority becomes meaningless. Cap enforced in ops.

### Stream 4 — FPO procurement SaaS (the moat)

- Product: the grading + procurement module of our own ops stack (grading queue, price feed, farmer payout rail, traceability labels) licensed to FPOs for **their own non-KisanSetu procurement** (their existing government/mandi/export channels).
- Pricing: free for the pilot FPO during the pilot (their payment is partnership + data); ₹5,000/hub/month intro from FPO #2; later tiered by volume (₹5k/10k/20k per hub/month at <50t / <200t / 200t+ per month).
- Strategic value exceeds revenue: every FPO on the SaaS becomes (a) a pre-integrated supply hub for our marketplace when we enter their region, and (b) a data feed of regional supply/price — the data moat. This is also the **pivot destination** (§8).

### Stream 5 — Financial services (Year 2, partner-led)

- Input credit: refer farmers with ≥3 months payout history to a partner NBFC/agri-fintech for input loans; we earn a referral fee (1–1.5% of disbursal) and repayment can be deducted from payouts **only with explicit farmer consent** (consent flow specified in 18-LEGAL-COMPLIANCE.md).
- Invoice financing: buyers who want 15–30 day credit terms get them from a partner financier against our invoice; we earn an origination fee and keep our own receivables ≤7 days (§6, working capital).
- Hard rule: **we are not an NBFC and will not lend from our own balance sheet** — capital-heavy lending is how WayCool-adjacent models got hurt (02-MARKET-RESEARCH.md).

## 4. Illustrative unit economics (to be replaced by Phase-0 data — see §5)

**Base-case order (Surat pilot assumptions):** avg order = 50 kg mixed produce @ ₹40/kg blended → **AOV ₹2,000**. One cold-van route serves 10 drops.

### 4.1 Per-order waterfall

| Line | ₹ | % of GMV | Notes |
|---|---:|---:|---|
| Buyer produce invoice (GMV) | 2,000 | 100.0% | 50 kg @ ₹40/kg blended A/B |
| Handling & delivery fee | +50 | — | Stream 2 |
| **Total billed to buyer** | **2,050** | | one invoice |
| Farmer payout (instant UPI) | −1,300 | 65.0% | ≥60% floor; base case 65% |
| FPO hub fee (aggregate, weigh, grade) | −120 | 6.0% | ₹2.4/kg partner fee |
| 3PL cold logistics (per-drop share) | −280 | 14.0% | ₹2,800 route / 10 drops |
| Crates, QR labels, consumables | −25 | 1.3% | amortized crate + label + tag |
| Wastage / QA-credit provision | −50 | 2.5% | grading rejects, buyer credits |
| Payments + messaging | −15 | 0.8% | UPI payout cost, SMS/WhatsApp |
| **Contribution per order** | **≈ 260** | **≈ 13% of billing** | before fixed costs |

Conservative planning case (the number used everywhere else in the Brain): **₹200 contribution/order (~10% of GMV)** — i.e., the base case with a thinner route (7–8 drops, 3PL ≈ ₹340/drop). The headline "₹2,000 AOV, ~10% blended take, ~₹200/order" from the shared brief is this row.

### 4.2 What moves the number (sensitivity)

| Lever | Bear | Base | Bull | Contribution/order |
|---|---|---|---|---|
| Drops per route (density) | 5 | 10 | 15 | ₹90 / ₹260 / ₹320 |
| AOV (buyer mix) | ₹1,200 (small resto) | ₹2,000 | ₹4,500 (hotel/caterer) | ₹60 / ₹260 / ₹700+ |
| Wastage | 5% | 2.5% | 1.5% | −₹50 / base / +₹20 |
| Farmer share | 60% | 65% | 68% | +₹100 / base / −₹60 |

Three rules fall out of this table and are binding on GTM and ops:

1. **Route density is the whole game.** Below ~6 drops/route, orders are contribution-negative. GTM must sell in **geographic clusters** (04-GTM-SALES-MARKETING.md §4 targets one food-street/market area at a time), and ops must batch delivery windows (14-OPS-PLAYBOOK.md).
2. **AOV mix matters more than take rate.** One hotel at ₹4,500 AOV ≈ 3 small restaurants in contribution but 1 drop instead of 3. Sales tiering in 04 reflects this.
3. **Never buy margin from the farmer.** Improving contribution by cutting farmer share below 60% is prohibited (Rule 5); improve density and wastage instead.

### 4.3 Per-crop view (why we freeze 2 beachhead crops)

Illustrative for the likely pair (final pair frozen in Phase 0, task T0.6):

| Crop (illustrative) | Farmer price ₹/kg | Buyer price A ₹/kg | Shelf life | Why it works for us |
|---|---:|---:|---|---|
| Tomato | 26 | 40 | 4–6 days chilled | daily HoReCa staple, grading spread A/B is visible and valued |
| Leafy greens (methi/palak/coriander) | 30 | 46 | 24–48h | freshness SLA is the entire pitch; incumbents are worst here |

High-perishability crops are where "<36h harvest→door" converts into pricing power — the SLA (Rule 5 metric #1) is a revenue lever, not just an ops KPI.

## 5. Validation plan — replacing illustrative numbers with real ones (Phase 0)

Every cell below gets filled during Phase-0 (13-LAUNCH-PLAN.md; interview scripts in 04-GTM-SALES-MARKETING.md §2). **No fundraising material may quote §4 numbers after Phase 0 completes** — real data supersedes.

| Illustrative number | Replaced by | Source | Owner / deadline |
|---|---|---|---|
| AOV ₹2,000 | Real avg daily veg spend per buyer segment | 10 buyer interviews (Q4–Q6 of script) | Alpesh / Phase-0 week 2 |
| Order frequency 15–20/mo | Stated ordering cadence + split across vendors | Buyer interviews Q5 | Alpesh / week 2 |
| Buyer price parity assumption | Actual vendor rate cards for 8 vegetables | Mandi shadow + buyer invoices collected in interviews | Alpesh / week 2 |
| Farmer payout 65% | Documented mandi-chain farmgate price for the 2 crops | Mandi morning shadow (T0.4) + FPO records | Alpesh / week 2 |
| FPO hub fee ₹2.4/kg | Negotiated MoU rate | FPO pilot MoU (T0.3) | Alpesh / week 2 |
| 3PL ₹2,800/route | 3 written quotes + 2 trial routes | 3PL quotes (T6.2 pulled forward for pricing) | Alpesh / week 3 |
| Wastage 2.5% | Measured on first 50 pilot orders | pilot instrumentation | Ops lead / pilot week 4 |
| Priority sub ₹1,499 willingness | Van Westendorp-style price question in interviews | Buyer interviews Q10 | Alpesh / week 2 |
| Credit-terms demand | % of buyers demanding >7-day credit | Buyer interviews Q8 | Alpesh / week 2 |

**Kill/pivot criterion (pre-committed):** if **<3 of 10** interviewed buyers show real willingness to switch (rubric in 04 §2.4), we do not "try harder" — we execute the FPO-SaaS pivot in §8.

## 6. Working capital — the honest cost of "instant farmer payout"

Instant UPI payout to farmers + any credit to buyers = a float we finance.

- Policy at launch: buyers pay **on delivery (UPI/link)** for their first 4 orders; vetted buyers then get **max 7-day credit**, per-buyer exposure cap **₹25,000**, auto-suspend at 10 days overdue (enforced in backend, 06-PRD-BACKEND.md).
- Float math at city break-even scale (§7): ₹30L GMV/month with ~50% of GMV on 7-day terms ≈ **₹3.5–5L standing receivables float**, financed from bootstrap capital (17-FUNDRAISE-FINANCE.md §5 reserves this) and later from seed + Stream-5 invoice financing.
- HoReCa habitually demands 15–30 day credit. We do **not** match it from our books; we hold the line at 7 days and route longer terms to the financing partner (Stream 5). Losing a buyer over credit terms is acceptable; becoming an unlicensed lender is not.

## 7. Path to profitability with tens of buyers

City-level fixed opex at pilot scale (mirrors the burn build-up in 17-FUNDRAISE-FINANCE.md §5): **≈ ₹3.0L/month** (founder stipend, ops lead, field sales, hub cost share, software/messaging, marketing ₹15–20k, compliance, travel).

**Break-even orders/month = ₹3,00,000 ÷ ₹200 contribution ≈ 1,500 orders/month (~58/day over 26 delivery days).**

| Buyer mix scenario | Orders/buyer/month | Active buyers to break even |
|---|---:|---:|
| Small restaurants only (AOV ₹2,000, ₹200 contribution) | 15 | 100 |
| Base mix, denser routes (₹260 contribution) | 18 | 64 |
| Hotel/caterer-weighted (blended AOV ₹3,200, ₹420 contribution) | 18 | **~40** |

So: **city-level operating break-even at roughly 40–100 active buyers** depending on mix and density — squarely "tens of buyers" (Golden Rule 6). GMV at break-even ≈ ₹30L/month (~$36k/mo).

### Milestone ramp (indicative, gates in 13-LAUNCH-PLAN.md)

| Month | Active buyers | Orders/day | GMV ₹L/mo | Contribution ₹L/mo | Fixed opex ₹L/mo | City EBITDA ₹L/mo |
|---|---:|---:|---:|---:|---:|---:|
| M3 (pilot start) | 15 | 8 | 4.2 | 0.5 (thin routes) | 2.6 | −2.1 |
| M4 | 25 | 15 | 7.8 | 1.2 | 3.0 | −1.8 |
| M6 | 45 | 28 | 14.6 | 2.6 | 3.0 | −0.4 |
| M8 | 65 | 42 | 21.8 | 4.4 (+Priority subs) | 3.2 | **+1.2** |
| M12 | 100 | 62 | 32.0 | 7.0 (+SaaS FPO #2) | 3.6 | +3.4 |

Every order must clear a **contribution-positive test by M6** (per-order instrumentation in 06-PRD-BACKEND.md; weekly review ritual in 14-OPS-PLAYBOOK.md). Company-level profitability then follows from replicating break-even cities (04 §8) faster than HQ costs grow — HQ stays a founder + small eng team until seed (17-FUNDRAISE-FINANCE.md).

## 8. The pre-committed pivot: FPO-SaaS-first

If the Phase-0 kill criterion fires (<3/10 buyers willing to switch), we pivot the **sequence**, not the vision: sell software to the people already doing the physical work, and earn the right to run the marketplace later.

| Dimension | Marketplace-first (default) | FPO-SaaS-first (pivot) |
|---|---|---|
| First customer | Surat HoReCa buyer | FPO (Gujarat has hundreds; SFAC/NABARD directories) |
| Product v1 | Full flow (apps + ops + logistics) | Grading queue + procurement + payout module of the ops dashboard, sold standalone |
| Revenue | Take rate + fees | ₹5,000/hub/month + onboarding fee ₹10,000; later % of incremental sales facilitated |
| Ops burden | High (daily physical SLA) | Near-zero (software + field onboarding only) |
| Path back to marketplace | — | Each SaaS FPO = pre-integrated supply hub + regional price/supply data; relaunch buyer side when ≥5 hubs live in one city's catchment |
| Break-even | 40–100 buyers (§7) | ~50–60 paying hubs ≈ ₹3L MRR — slower revenue, ~10x lower burn |

What carries over unchanged: the entire backend domain model (grading, price feed, payments, traceability), the ops dashboard, the design system, the farmer-side app patterns. What is shelved: buyer apps, 3PL contracts, buyer-side GTM. The decision memo (task T0.7) records the choice with reasons; 16-RISKS-MITIGATIONS.md carries the risk framing.

## 9. Global replication of the model (Rule 2)

The P&L template of §4 is parameterized, not Surat-shaped. To price any new region, fill one config sheet:

| Parameter | Surat instance | Abstraction in product |
|---|---|---|
| Currency | INR | `currency` field on all money columns |
| Instant payout rail | UPI (Razorpay) | payment-provider interface (06-PRD-BACKEND.md) |
| Reference price source | mandi (APMC) daily prices | `reference_market_price` in `price_feed` — never named "mandi" in schema |
| Aggregator partner | FPO | `fpos` table = generic producer-organization entity (co-ops in Kenya/Vietnam, ejidos-adjacent structures in LATAM) |
| Tax regime | GST | per-region tax abstraction |
| Per-drop logistics cost | ₹280 | region config; drives min-order & density rules |
| Farmer-share floor | ≥60% | global constant — it is the brand, everywhere |

City #2 and country #2 GTM replication is specified in 04-GTM-SALES-MARKETING.md §8; the finance implications in 17-FUNDRAISE-FINANCE.md §6.

## 10. Metrics & instrumentation for this doc

Weekly (from pilot day 1, dashboard spec in 09-PRD-OPS-DASHBOARD.md):
`contribution_per_order` (waterfall per §4.1, computed per order), `blended_take_rate`, `farmer_share_pct` (published), `drops_per_route`, `wastage_pct`, `aov`, `orders_per_buyer_per_month`, `receivables_days`, `priority_subscriber_count`, `saas_mrr`. Monthly: city EBITDA vs §7 ramp.

## 11. Open questions (owner + deadline; no orphan TBDs)

1. **Final take-rate split per crop/grade** — Alpesh, end of Phase-0 week 3, from real vendor rate cards.
2. **Handling-fee slab design from M4** (flat ₹50 vs ₹0/50/100 slabs) — Alpesh with ops lead, end of pilot month 1, from density data.
3. **Priority subscription price point** (₹999 vs ₹1,499 vs ₹1,999) — Alpesh, M5, from interview Q10 + first 8 upsell conversations.
4. **FPO SaaS pricing: per-hub flat vs per-tonne** — Alpesh, before FPO #2 signing (target M6).
5. **Whether Grade B demand is deep enough for 12–18% discount or needs 20%+** — ops lead, pilot week 6, from Grade-B sell-through rate.
6. **NBFC partner shortlist for Stream 5** — Alpesh, M9 (not urgent; Year-2 stream).

**Decisions taken in this doc that the founder should ratify:** principal-lite invoicing model (§2); 65% base farmer share; ₹50 flat handling fee + ₹1,000 minimum order; 7-day/₹25k buyer-credit policy; Priority capped at 40% of buyers; "never lend from our books" rule.
