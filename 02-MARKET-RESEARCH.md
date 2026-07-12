# 02 — Market Research

_Figures verified against live sources in July 2026 unless labeled otherwise. Founder estimates and directional numbers are explicitly marked — do not present them externally without the label. Reads with: [01-VISION-MISSION.md](01-VISION-MISSION.md) (what we do with this evidence) and [03-BUSINESS-MODEL.md](03-BUSINESS-MODEL.md) (how it converts to revenue)._

---

## 1. Executive summary

- India's fruit & vegetable market is **~$47B (2025)** growing 6–7% a year; the B2B slice we serve (HoReCa + modern retail + institutions) is **~$10B** (founder estimate); our 3–5-year beachhead opportunity is **~$300M** (directional).
- The dysfunction is quantified: **₹1.5 lakh crore/year post-harvest loss** (NABCONS 2022) and farmers keeping only **~30–40%** of the final price across 4–6 intermediation hops.
- The competitive field has **bifurcated and vacated our exact lane**: the horizontal leader is pivoting away from focused fresh, the full-stack players are inputs-led, and **every asset-heavy B2B2C fresh player died in 2024–25** — which proves the asset-light thesis rather than threatening it.
- The wedge: **focused, asset-light, HoReCa-first fresh produce in tier-2 India.** Nobody owns it.
- The same smallholder→HoReCa gap exists at comparable scale in **Southeast Asia, Africa and LATAM** (Section 9) — which is why the platform is built region-agnostic from day 1 (golden rule 2).

---

## 2. Market size: TAM / SAM / SOM

| Layer | Definition | Figure | Confidence & source |
|---|---|---|---|
| **TAM** | India fruits & vegetables market, all channels | **~$47B (2025)**, ~6–7% CAGR | **Verified.** Grand View Research ($46.9B, 2025); TechSci Research ($48.8B, 2025). Two independent estimates within 4% of each other. |
| **SAM** | B2B fresh procurement: HoReCa + modern retail + institutional kitchens | **~$10B** (~20% of TAM) | **Founder estimate.** Derived from channel-share logic; pressure-test against real HoReCa procurement data in Phase 0. |
| **SOM** | Beachhead serviceable share in target tier-2 metros over 3–5 years | **~$300M** | **Directional founder estimate.** Sanity check: ~₹2,500 Cr ≈ fresh-produce procurement of a handful of tier-2 cities' organized HoReCa. |

**Honesty note (keep this discipline):** the original pitch deck overstated TAM at "$150B+" (that figure describes broader food/agri, not F&V) and SAM at ~$35B. Corrected July 2026. Never quote the old numbers.

**Adjacent growth context (verified):** Indian agritech overall is compounding at **~25% CAGR toward ~$28B by 2030**, and farm-to-market **supply-chain/market-linkage models alone are projected to $12B+ by 2030** (Inc42 Agritech Landscape 2025). We sit squarely in the second figure.

**Bottom-up cross-check (validate in Phase 0, [03-BUSINESS-MODEL.md](03-BUSINESS-MODEL.md)):** a mid-size Surat restaurant plausibly buys ₹1,500–3,000 of fresh produce per day (illustrative ₹2,000 AOV). 25 buyers ordering ~25 days/month ≈ **₹12.5L GMV/month ≈ ₹1.5 Cr/year** from a single hub's demand side — enough to prove unit economics with one FPO partner. That's the pilot math; the SOM is that, repeated across hundreds of buyer clusters.

---

## 3. The problem, quantified

### 3.1 Post-harvest loss
- **₹1.5 lakh crore (~$18B) lost per year across all crops** — NABCONS 2022 study for MoFPI; still the standard citation in 2026. Horticulture (fruits & vegetables) is the worst-hit segment, on the order of ~50M tonnes/year.
- Loss caption discipline: this is **total post-harvest crop loss**, not "loss caused by missing cold chain" alone. Cold chain gaps are one driver among handling, multi-hop transit, and time-to-market. (The old deck mis-captioned this; corrected.)
- Mechanism relevant to us: each intermediation hop adds loading/unloading damage and 6–24 hours of ambient-temperature time. Compressing farmer→buyer to one hub stop and one cold-chain leg attacks the loss at its structural cause — this is why freshness (metric 1) and loss reduction are the same lever.

### 3.2 Farmer economics
- Farmers retain **~30–40% of the consumer/buyer price** for fresh produce after 4–6 intermediaries (aggregator → mandi trader → wholesaler → sub-wholesaler → retailer/kitchen supplier).
- KisanSetu's direct chain targets **≥60% farmer share** (metric 2) with the take rate coming out of eliminated intermediation, not out of either end. [01-VISION-MISSION.md](01-VISION-MISSION.md) calls this the one-lever insight.
- Payment terms are the hidden second problem: mandi settlements commonly take days and involve deductions; **instant UPI payout on pickup** is by itself a switching reason for farmers.

### 3.3 Buyer (HoReCa) pain — to verify in Phase 0 interviews
- Quality inconsistency: no grading; the same crate mixes A and C produce; chefs re-sort by hand.
- Price opacity and volatility: daily haggling, no invoice trail, GST-compliant billing hard to get (matters for organized HoReCa input credit).
- Supply unreliability: procurement staff spend 2–4 early-morning hours/day at the mandi or managing multiple informal vendors.
- Zero traceability: no answer to "where was this grown and when was it picked" — increasingly a requirement for hotel chains and QSR audits.

---

## 4. Supply side: farmer demographics

| Fact | Figure | Source |
|---|---|---|
| Operational farm holdings in India | **~146–150M** | Agriculture Census 2015–16 |
| Small & marginal holdings (<2 ha) | **~86%** (86.1%) | Agriculture Census 2015–16; NSO 77th round puts it at 89.4% |
| Implication | The median supplier is a **smallholder who cannot individually fill a truck or negotiate with a buyer** — aggregation is structurally necessary | — |

This is why the FPO (Farmer Producer Organisation) is the load-bearing partner in our model:
- **~10,000+ FPOs** exist under the central government's "10,000 FPOs" formation scheme (launched 2020, ₹6,865 Cr budget) plus older NABARD/SFAC-promoted ones. Gujarat has an active FPO ecosystem including in Surat district. **Phase 0 task: map and meet the 5–10 FPOs within 50 km of Surat; sign 1 pilot MoU.**
- FPOs already do the physical work (aggregation, sometimes grading) but chronically lack **demand linkage and software** — exactly the two things we bring. This is also the fallback business ([03-BUSINESS-MODEL.md](03-BUSINESS-MODEL.md): FPO procurement SaaS) if marketplace validation fails.
- Smartphone penetration among farm households is high enough for an app-based flow (UPI usage in rural India has exploded; see §7), but the UX must assume low digital literacy — hence the farmer-flow design constraints in [07-PRD-MOBILE-APPS.md](07-PRD-MOBILE-APPS.md) (max 2 actions/screen, picture grids, Hindi-first).

## 5. Demand side: the HoReCa beachhead

- Surat: ~7M+ metro population, one of India's fastest-growing cities, with a dense restaurant/caterer/cloud-kitchen economy (diamond & textile industry canteens included) and tier-2 cost structure.
- Target buyer profile at launch: mid-size restaurants, caterers, cloud kitchens, small hotel kitchens buying **₹1,500–5,000/day** of fresh produce — big enough to matter, small enough to switch without a procurement committee.
- Later buyer types (post-pilot gates only): modern retail (local supermarket chains), food processors, institutional kitchens.
- **Phase 0 instrument:** 10 structured interviews (5 restaurants, 2 hotels, 2 caterers, 1 cloud kitchen). Kill/pivot criterion: **<3 of 10 willing to switch → pivot to FPO-SaaS-first wedge.** Interview guide lives in [13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md).

---

## 6. Competitive landscape (verified July 2026)

### 6.1 Active players — and why each leaves the lane open

| Player | Funding / scale | Position (July 2026) | Why they don't own our lane |
|---|---|---|---|
| **Ninjacart** | Walmart/Flipkart-backed; ~$370–500M raised; ~₹1,600–2,000 Cr revenue | **Pivoting AWAY from focused fresh** into staples, quick-commerce enablement, exports (Ninja Global), retailer SaaS. FY25 operating scale reportedly down ~19% YoY | The one player built for fresh-to-business is deliberately broadening out of it; metro-focused besides |
| **DeHaat** | ~$270–325M raised (Temasek, Peak XV); **₹3,010 Cr FY25 revenue** — agritech's highest; EBITDA breakeven Q1 FY26 | Full-stack farmer platform: inputs + advisory + linkage; acquired AgriCentral, NEERX | **Inputs/advisory-led**; light on buyer-side fresh delivery and HoReCa relationships |
| **AgroStar** | ~$186M raised; $30M Series E (2025) at flat ~$250M valuation | Inputs + advisory + some market linkage | Same shape as DeHaat: farmer-side revenue, not buyer-side fresh logistics |
| **WayCool** | ~$307M raised — was the only agritech unicorn | **Distressed**: layoffs, governance-lapse scrutiny, struggling to raise beyond small debt infusions | Capital-heavy full-stack model straining — a live warning, not a threat |
| **Arya.ag** | $80M raised (late 2025) | Grain-centric warehousing/finance; **profitable** | Grains & storage finance, not fresh perishables to HoReCa |
| Local incumbents | — | Mandi traders & informal HoReCa suppliers in every city | The real day-to-day competitor: relationships + credit. Beaten on grading, reliability, invoice, traceability — not on price alone |

### 6.2 The graveyard — dead players as thesis proof

All of these were **asset-heavy B2B2C fresh** models. Their common cause of death — capex + inventory risk + consumer discounts — is precisely what golden rules 4 (ASSET-LIGHT) and 6 (B2B) prohibit.

| Player | Model | Outcome |
|---|---|---|
| **Otipy (Crofarm)** | B2B2C fresh, own micro-warehousing, reseller network | **Shut down May 2025** after failing to raise ~$10M |
| **Fraazo** | D2C fresh, dark stores | Dead (2024–25) |
| **ReshaMandi** | Full-stack silk/agri supply chain, heavy working capital | Collapsed (2024) |
| **Greenikk** | Banana supply chain + financing | Shut (2024) |
| **Deep Rooted** | Managed farms + D2C/B2C fresh | Shut (2024–25) |

**Read the graveyard correctly:** these failures are not evidence that farm-to-business is a bad market; they are evidence that **owning the physical layer while chasing consumers** is a bad model in this market. The demand-side pain and supply-side pain both remain unserved — by the incumbents' own pivots.

### 6.3 Funding climate
- 2025 was an agritech funding winter: **~$160M** total (vs ~$360M in 2024).
- Late 2025–early 2026: **selective revival + consolidation** — Arya.ag $80M, AgroStar $30M, Unnati+Gramophone merger. Capital now flows to **capital-efficient, profitability-visible** models — which is a tailwind for an asset-light, B2B, unit-economics-first pitch, and a headwind for anyone trying to out-spend their way in ([17-FUNDRAISE-FINANCE.md](17-FUNDRAISE-FINANCE.md) builds the raise narrative on this).

### 6.4 The wedge

> **Nobody owns focused, asset-light, HoReCa-first fresh produce in tier-2 India.**

Three structural facts create it:
1. The horizontal leader (Ninjacart) is **broadening away** from focused fresh-to-HoReCa.
2. The well-funded full-stack players (DeHaat, AgroStar) are **inputs/advisory-led** — their revenue engine points at the farmer's wallet, not the buyer's kitchen.
3. Every asset-heavy fresh player **died**, clearing the field and proving which cost structure survives.

And a geographic fact: what fresh-focused capacity remains is **metro-concentrated**, while tier-2 cities (Surat-class) have the same organized-HoReCa growth with no organized supplier. Defensibility over time comes from the FPO SaaS + traceability data layer ([03-BUSINESS-MODEL.md](03-BUSINESS-MODEL.md)), not from being first.

---

## 7. Why now

Four independent shifts converge in 2025–26; any one alone would be insufficient.

1. **Digital rails are done.** UPI processes 18B+ transactions/month (2025) and works in villages — instant farmer payout requires zero user education. WhatsApp is India's de-facto business channel (500M+ users), so buyer ordering/price broadcast needs no app-install barrier on day 1. Aadhaar/eKYC makes onboarding cheap. None of this existed at scale when Ninjacart's cohort designed their models (2015–18) — they had to build trust with cash and feet; we inherit the rails.
2. **The FPO push matured.** The 10,000-FPO scheme (2020) has moved from formation to the "now what?" phase: thousands of FPOs exist with aggregation capability but **no demand linkage and no software**. They need exactly what we bring, making partner acquisition cheap — and making the FPO-SaaS fallback a real business.
3. **Cold-chain capacity is investable and rentable.** Sustained public investment (PM Kisan SAMPADA-family schemes, ~35%-subsidy cold-chain grants) plus private reefer fleet growth means **3PL cold logistics can be rented per-trip in tier-2 corridors** — the enabling condition for asset-light. In 2016 you had to buy the truck; in 2026 you book it.
4. **B2B procurement is formalizing.** GST discipline pushes organized HoReCa toward invoice-clean suppliers; the post-2020 boom in cloud kitchens and food-service chains professionalized purchasing (a procurement manager with targets, not a cook with cash); traceability/food-safety expectations (FSSAI tightening, chain audits) reward exactly what we sell. Simultaneously, the 2024–25 shakeout removed the discount-subsidized alternatives that distorted buyer price expectations.

**Why-now risk check:** the same funding winter that cleared the field also means we must reach proof on very little capital — which the B2B sales-led model (tens of customers) is designed for. See [16-RISKS-MITIGATIONS.md](16-RISKS-MITIGATIONS.md).

---

## 8. What we must still validate (Phase 0 — owners & deadlines in [13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md))

| Open item | Method | Replaces |
|---|---|---|
| Real HoReCa AOV, order frequency, switching willingness | 10 structured buyer interviews | Illustrative ₹2,000 AOV, ~20% SAM share assumption |
| FPO capacity, grading capability, MoU terms | Meet 2+ Surat-district FPOs, sign 1 pilot MoU | Assumed hub throughput |
| True current-chain baseline (prices, hops, hours) | Shadow one mandi morning + one restaurant procurement cycle | Literature-derived 30–40% farmer share |
| 3PL cold logistics per-trip cost, Surat corridors | 3 quotes | Assumed logistics cost line |
| Beachhead crops | Freeze 2 from actual buyer demand (working hypothesis: tomato + onion or leafy greens) | Crop assumptions in PRDs |
| Name | Trademark/availability check for "KisanSetu" | Working title |

---

## 9. GLOBAL: the same gap elsewhere

**Why this section exists:** golden rule 2. The platform is built region-agnostic because the smallholder→HoReCa structure recurs across the global South. This section identifies where, and gives the sizing logic — all figures here are **directional desk research** for architecture and ambition-setting, to be re-verified before any entry decision.

### 9.1 The pattern to look for
A market fits KisanSetu when all four hold:
1. **Fragmented smallholder supply** — farms too small to fill a truck (globally, ~500M+ smallholder farms produce roughly a third of the world's food — FAO).
2. **Multi-hop intermediation** — farmer share materially below 50%, post-harvest losses high (FAO: ~14% of food lost between harvest and retail globally, worst in fruits & vegetables in developing regions).
3. **Urbanizing food-service demand** — restaurants/hotels/caterers formalizing procurement.
4. **Digital rails present** — real-time payments + ubiquitous messaging (the UPI/WhatsApp equivalents).

### 9.2 Candidate markets

| Market | Rails (payments/messaging) | Smallholder structure | F&V / food-service signal (directional) | Notes & local comparables |
|---|---|---|---|---|
| **Indonesia** | QRIS real-time payments; WhatsApp-dominant | ~90%+ of ~30M+ farms are smallholders | F&V market tens-of-$B; huge warung/HoReCa base across Java's dense cities | TaniHub (farmer marketplace) collapsed 2022 — asset-heavy again; Eden Farm/Sayurbox retrenched. Lane open. **First international candidate.** |
| **Vietnam** | Instant transfers (NAPAS), Zalo messaging | ~9M+ smallholder households | Strong horticulture base; booming urban food service in HCMC/Hanoi | Cooperative (HTX) structure is the FPO analog |
| **Philippines** | GCash/Maya wallets ubiquitous | Highly fragmented island supply | High food-service spend share; Manila import-substitution angle | Inter-island logistics = harder cold chain; later |
| **Kenya** | M-Pesa — the world's proof of mobile money | ~7.5M smallholder farms dominate supply | Nairobi HoReCa + strong horticulture sector (export-grade practices exist) | Twiga Foods raised ~$150M+, hit distress/layoffs 2023–24 under an asset-heavy build-out — same graveyard lesson. Lane re-opening |
| **Nigeria** | Fast-growing instant payments (NIP), WhatsApp | Tens of millions of smallholders | Lagos food-service scale; severe post-harvest loss (>30% in perishables cited) | Infrastructure/cold-chain gaps worst-in-class; partner-density problem; later |
| **Egypt** | InstaPay growth | Nile-delta smallholders | Cairo HoReCa + processing demand | MaxAB (B2B commerce, merged with Wasoko) shows B2B rails appetite, but FMCG-first not fresh |
| **Mexico** | SPEI transfers, WhatsApp-heavy | Ejido/smallholder mix | Large restaurant industry; proximity to US food-service standards | Frubana (Colombia-origin, B2B restaurant supply, ~$200M raised) operates here — the closest living analog to our model, validating B2B-restaurant-supply economics in LATAM |
| **Colombia / Peru** | PSE/Yape-Plin rails | Andean smallholders | Bogotá/Lima food-service density | Frubana's home turf (Colombia); watch their asset posture vs ours |

### 9.3 Rough sizing logic (method, not gospel)
For any candidate market: `Opportunity ≈ national F&V consumption value × urban food-service share (10–25%) × addressable-city fraction`. Applied directionally: Indonesia and Vietnam each plausibly hold a **$3–8B B2B fresh procurement pool** (F&V markets in the tens of billions × ~20% food-service share), Kenya/Nigeria smaller pools ($0.5–2B) but with the highest farmer-share uplift potential, LATAM candidates in between — i.e., **each top candidate is a meaningful fraction of the Indian SAM**, and collectively they exceed it. That is enough to justify day-1 region abstraction (cheap) without justifying any near-term entry spend (expensive). **Rule: re-verify all of §9 with in-country data before any entry decision; owner: founder; trigger: Stage 4 of the expansion ladder in [01-VISION-MISSION.md](01-VISION-MISSION.md).**

### 9.4 What global entry will look like (constraint, not plan)
Same model, new instances: local farmer-org partner (co-op/HTX/FPO-analog) + local 3PL cold chain + local payment provider behind the abstract payout interface + local language pack. No owned assets, no B2C, same two metrics. If a market can't support the two metrics, we don't enter it.

---

## 10. Source register

| Claim | Source | Year |
|---|---|---|
| India F&V TAM ~$47B | Grand View Research ($46.9B); TechSci ($48.8B) | 2025 |
| Agritech ~25% CAGR → ~$28B by 2030; market-linkage → $12B+ | Inc42 Agritech Landscape 2025 | 2025 |
| Post-harvest loss ₹1.5 lakh crore/yr | NABCONS study for MoFPI | 2022 |
| ~146–150M holdings; 86% small & marginal | Agriculture Census 2015–16; NSO 77th round (89.4%) | 2016/2021 |
| Competitor statuses (Ninjacart pivot, DeHaat ₹3,010 Cr FY25, Otipy shutdown, WayCool distress, graveyard list, funding winter ~$160M) | Live web verification for pitch deck v2 | July 2026 |
| SAM ~$10B; SOM ~$300M | Founder estimates | 2026 — validate Phase 0 |
| §9 global figures (FAO smallholder/loss stats, market comparables) | Desk research, directional | 2026 — re-verify before entry |

**Maintenance:** re-verify §6 (competitors) quarterly and before any investor conversation; re-verify §2 annually. Owner: founder.
