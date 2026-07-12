# 01 — Vision & Mission

_Reads with: [00-GOLDEN-RULES.md](00-GOLDEN-RULES.md) (the constraints), [02-MARKET-RESEARCH.md](02-MARKET-RESEARCH.md) (the evidence)._

---

## Vision

**A world where every smallholder farmer can sell directly to the businesses that feed cities — and keep most of the price.**

The gap KisanSetu closes is universal: wherever smallholder farming meets urban food service, a chain of 4–6 intermediaries takes 60–70% of the final price while produce loses days of freshness in transit. India is where we start; the problem exists in Jakarta, Nairobi, Ho Chi Minh City and Bogotá in nearly identical form (see [02-MARKET-RESEARCH.md](02-MARKET-RESEARCH.md), Global section).

## Mission

**Bridge smallholder farmers to food businesses through an asset-light, tech-led supply chain — produce delivered within 48 hours of harvest, farmers paid instantly and keeping ≥60% of the final price, buyers getting graded, traceable supply on one invoice.**

Every word is load-bearing:
- **Bridge** — "setu" means bridge. We connect; we don't own the ends (rule 4, ASSET-LIGHT).
- **Smallholder farmers** — ~86% of India's ~146–150M holdings are under 2 hectares; they are the supply side, reached through FPO partners, never bypassed.
- **Food businesses** — B2B, sales-led (rule 6). HoReCa first, then modern retail and processors.
- **Asset-light, tech-led** — partner FPO hubs + 3PL cold chain; our contribution is demand, software, grading standards, and data.
- **48 hours / instantly / ≥60% / traceable** — the measurable promises. The first and third are THE TWO METRICS (rule 5); they are published, not just tracked.

## The one-lever insight

The entire company rests on a single piece of arithmetic:

> **A shorter chain fixes both sides with the same move.** Farmer → FPO hub → cold 3PL → buyer replaces farmer → aggregator → mandi trader → wholesaler → sub-wholesaler → buyer. Removing the middle roughly **doubles the farmer's share (~30–40% → ~60–70%)** and lands produce **days fresher** — without subsidy, without discounting, without owning assets. The margin we earn (8–12% take rate; [03-BUSINESS-MODEL.md](03-BUSINESS-MODEL.md)) comes out of eliminated intermediation, not out of the farmer or the buyer.

If this arithmetic survives Phase 0 contact with real Surat orders, everything else is execution.

---

## Global ambition, Surat beachhead

KisanSetu is a **global platform launched hyper-locally** (rule 2). The expansion ladder:

| Stage | Geography | What must be true to advance |
|---|---|---|
| 0. Beachhead | **Surat, Gujarat** — 1 FPO hub, 2 crops, 15–25 HoReCa buyers | Phase 0 validation passed; pilot gates: ≥70% weekly buyer repeat, ≥95% fulfilment, farmer share ≥60% documented ([13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md)) |
| 1. Region | Gujarat (Ahmedabad, Vadodara, Rajkot) | Surat contribution-margin positive; ops playbook repeatable without founder present daily |
| 2. Tier-2 India | Nashik, Indore, Coimbatore, Jaipur-class cities — the uncontested lane | Multi-hub ops proven; FPO SaaS deployed at partner hubs |
| 3. Indian metros | Selective — enter where incumbents have vacated focused fresh | Brand + balance sheet to fight for supply |
| 4. International | Southeast Asia → Africa → LATAM (same smallholder→HoReCa gap) | Platform fully region-agnostic in production (it is by design from day 1); local partner model validated remotely first |

Why Surat first: founder lives there (founder-led sales is the GTM); dense HoReCa market with tier-2 economics; Gujarat's strong FPO ecosystem; and no incumbent focus — the big players fight over metros while tier-2 goes unserved.

Why global-first engineering anyway: retrofitting i18n/currency/tax abstraction is a rewrite; building it as configuration from day 1 is nearly free. The schema speaks generic supply-chain language (`reference_market_price`, region → hub → farm); India is an instance, not the architecture. See rule 2 for the full checklist.

---

## What KisanSetu IS / IS NOT

| KisanSetu IS | KisanSetu IS NOT |
|---|---|
| A B2B supply-chain platform: farmers → FPO hubs → 3PL → food businesses | A consumer grocery/delivery app (that model's graveyard is documented in [02-MARKET-RESEARCH.md](02-MARKET-RESEARCH.md)) |
| Asset-light: demand, software, standards, data | An owner of farms, trucks, warehouses, or dark stores |
| Sales-led: tens of business customers, founder with sample crates | Growth-hacked: downloads, DAU, virality, discount wars |
| A freshness & farmer-share company — both metrics published | A "super app" for farmers (no lending-first, no social feed, no agri-content play) |
| An FPO partner and, later, their software vendor (procurement SaaS = the moat) | An FPO competitor or replacement |
| Focused: 1 city, 2 crops, 1 buyer type at launch | Everything-everywhere: staples, q-commerce, exports, inputs (that's the incumbents' drift, and our opening) |
| A traceability layer: every crate maps farm → order (`order_allocations`) | A commodity trader taking inventory risk on unsold produce |
| Global by design (i18n, multi-currency, tax abstraction from day 1) | India-hardcoded software with "international" on a slide |

Adjacent revenue (input credit, invoice financing) enters only after the core loop is profitable, and as partnerships first — see [03-BUSINESS-MODEL.md](03-BUSINESS-MODEL.md).

---

## The 10-year view

**Years 0–1 — Prove (Surat).** Phase 0 validation, then pilot: 1 FPO hub, 2 crops, 15–25 buyers, 25–50 farmers. Freshness SLA instrumented from crate one. Exit criteria: the two metrics hit and published, contribution-margin-positive orders. If <3 of 10 buyers bite in Phase 0 → pivot to FPO-SaaS-first (the fallback is a feature of the plan, not an embarrassment).

**Years 1–3 — Repeat (Gujarat → tier-2 India).** 3–5 cities, multi-hub ops, ops playbook executed by hired city leads instead of the founder. FPO procurement SaaS licensed to partner hubs — the software becomes the switching cost and the data moat (grading records, yield patterns, demand curves). Seed → Series A on retention and unit economics, not GMV theater. Directional target: capture a meaningful slice of the ~$300M beachhead SOM.

**Years 3–5 — Compound (India scale + first international).** 15–25 Indian cities; buyer types widen to modern retail and processors; crop range widens with grading capability. Data products mature: demand forecasting for FPOs, dynamic daily pricing, credit-scoring signals from transaction history (partner-delivered financing). First international pilot in one Southeast Asian city (likely Indonesia or Vietnam) run with a local partner on the same codebase — the day-1 region abstraction pays out here.

**Years 5–10 — The standard.** KisanSetu is the default procurement rail for fresh produce in tier-2 India and 2–3 international markets: thousands of FPO/co-op hubs on our SaaS, tens of thousands of food businesses buying on published freshness SLAs, millions of farmers paid instantly at ≥60% share. The long-run identity is **infrastructure**: the grading standard, the traceability record, and the price signal for smallholder fresh produce — the Visa-network of farm-to-business, owning the protocol and the data while partners own every physical asset.

**What stays true the whole decade:** the six golden rules; the two published metrics; asset-light discipline at every scale ("add partners, not assets"); and the farmer's payout slip as the ultimate proof — freshness in hours, share in percent, no adjectives.

---

## Founder's operating principles (voice of the company)

- Show the payout slip, don't say "empowering farmers." The slip with the struck-through reference market price IS the message.
- Freshness in hours, not adjectives. "Harvested 22 hours ago" beats "farm fresh" every time.
- Software must never block a delivery — in early pilot weeks, WhatsApp and phone calls are an acceptable backend ([14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md)).
- When in doubt, do the thing that moves one of the two metrics.
