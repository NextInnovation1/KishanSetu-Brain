# 18 — LEGAL & COMPLIANCE

_KisanSetu Brain · July 2026 · Owner: Alpesh (founder), executed with a retained CA (compliance) and a Gujarat advocate (APMC + contracts) — both engaged in Phase 0 (T0.8, [15-TASKS-BACKLOG.md](15-TASKS-BACKLOG.md)). Siblings: [03-BUSINESS-MODEL.md](03-BUSINESS-MODEL.md) (what is being monetized, hence taxed), [13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md) (launch preconditions P5), [14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md) (SOPs that must stay legal), [16-RISKS-MITIGATIONS.md](16-RISKS-MITIGATIONS.md) (R7 regulatory risk)._

**Posture:** compliance is a launch precondition, not an afterthought — but proportionate to pilot scale. We do the mandatory items before the first commercial delivery, keep a single compliance file (physical + `legal/` folder) that could be handed to an investor or inspector on any day, and never let a *recoverable* fee dispute stop a delivery ([16-RISKS-MITIGATIONS.md](16-RISKS-MITIGATIONS.md) R7). Where a point of law is genuinely uncertain, this document says so and assigns an owner + deadline instead of pretending certainty.

---

# PART A — INDIA LAUNCH (Gujarat / Surat)

## 1. Entity: Private Limited incorporation
- **Form:** Private Limited Company under Companies Act 2013 via SPICe+ (MCA). Name: "KisanSetu" pending T0.5 trademark clearance — reserve two backups in the same SPICe+ Part A cycle. Registered office: Surat.
- **Structure at incorporation:** founder ~99%+, nominee shareholder for the second-member requirement; authorized capital ₹1,00,000 (raise later as needed); founder as sole director initially (add a second director within statutory limits — a Pvt Ltd needs **two directors**; use a family member with DIN as non-executive until a co-founder/senior hire exists).
- **Deliverables bundle (CA-executed, ~₹15–25k all-in, 10–15 working days):** DSC, DIN, CIN, PAN, TAN, bank current account, share certificates, statutory registers, auditor appointment (within 30 days of incorporation).
- **Object clause:** trading and marketing of agricultural produce; supply-chain, logistics-arrangement and technology services; software licensing (covers the future FPO-SaaS line — write it in now, avoid amending later).
- **Also at incorporation:** Udyam (MSME) registration (free, unlocks delayed-payment protections against buyers); Gujarat Shops & Establishments registration for the office; Gujarat professional tax enrollment (company + employees).
- **Why not LLP/proprietorship:** the seed raise ([17-FUNDRAISE-FINANCE.md](17-FUNDRAISE-FINANCE.md)) requires equity instruments; converting later costs more than incorporating right.

## 2. FSSAI (food business license)
- KisanSetu trades in food (fresh produce) → it is a Food Business Operator. Farmers themselves are exempt; **we are not**.
- **Path:** start with **FSSAI Basic Registration** (projected first-year turnover < ₹12 lakh) filed at incorporation; **upgrade to State License** (category: trade/distribution) the quarter turnover run-rate crosses ₹12 lakh — set a standing check in the compliance calendar (§12). Fees are nominal (₹100/yr registration; ₹2,000–5,000/yr state license).
- FSSAI number printed on buyer invoices and the website footer. Hub hygiene SOP ([14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md) §2) doubles as our Schedule 4 hygiene evidence.
- The **FPO** should hold its own FSSAI registration for hub aggregation activity — made a warranty in the MoU (§6).

## 3. GST registration & structure
- **Register from day 1** (voluntary if below threshold): the platform's service revenue (delivery/handling fee, later SaaS and subscriptions) is taxable, and buyers will demand GST invoices. GSTIN in Gujarat; add places of business as hubs/cities are added (E1/E2 in [13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md)).
- **Tax treatment (to be confirmed in writing by the CA at T0.8 — this is the working position):**
  - **Fresh, unprocessed vegetables & fruits are GST-exempt (0%/nil-rated)** — the produce line on the buyer document is a **bill of supply**, not a tax invoice.
  - **Delivery/handling fee to the buyer: taxable at 18%** (support/logistics-arrangement service) — separate tax-invoice line. This is why [14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md) §8.5 and T2.14 specify a combined document: exempt goods lines + taxable service line.
  - **Buyer premium subscription and FPO SaaS (later): 18%.**
  - Purchases from farmers (agriculturists supplying their own exempt produce): no GST, no reverse charge on the produce itself.
- **Legal form of the transaction (decisive choice, CA to confirm):** at pilot, KisanSetu operates as **principal (buy–sell trader)** — we buy from the farmer at the payout rate and sell to the buyer at the platform price. Economically it is a take rate ([03-BUSINESS-MODEL.md](03-BUSINESS-MODEL.md)); legally buy–sell is simpler: one invoice to the buyer (the product promise), no agent/commission GST complexity, no e-commerce-operator TCS question on exempt goods. Revisit the marketplace/agent structure only if inventory-risk or regulatory reasons force it. **Owner: CA memo by end of Phase 0.**
- Compliance rhythm: monthly GSTR-1/3B (CA-run), ITC only on taxable-supply-linked inputs (mixed-use apportionment — CA handles), e-invoicing/e-way-bill thresholds monitored (not applicable at pilot turnover; exempt goods are outside e-way-bill anyway).
- Income-tax notes: TDS on salaries and contractor payments (194C for 3PL, 194J for professionals) from day 1; 194Q (goods purchases > ₹50L/seller/yr) is a scale-up item, flagged in the calendar.

## 4. Invoices & records (what the software must produce)
Per delivery, one consolidated buyer document ([06-PRD-BACKEND.md](06-PRD-BACKEND.md) `payments`, T2.14): supplier details + GSTIN + FSSAI no., buyer details + GSTIN, bill-of-supply lines for produce (crop, grade, kg, rate — exempt), tax-invoice line for delivery/handling (18% CGST+SGST), invoice number series per FY, and the traceability reference (order ID → allocations). Farmer side: the payout slip ([14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md) §7.3) is our purchase record — slip archive + bank UTR trail is the audit spine. Retention: 8 years (GST) — the `legal/` folder and payout-slip files follow this.

## 5. APMC / mandi-fee rules — direct procurement in Gujarat
**The question:** can KisanSetu buy produce directly from farmers at an FPO hub, outside the APMC market yard, without an APMC license or market cess?

**Working position (to be confirmed by a Gujarat advocate's signed memo at T0.8 — launch precondition P5):**
- The Gujarat Agricultural Produce Markets Act, 1963, **as amended in 2020**, restricted APMC regulation and fee jurisdiction to transactions **inside the market yard**, and created a framework for direct/private purchase and unified licensing. The central 2020 farm laws were repealed in 2021, but Gujarat's own state amendments are what govern us — the memo must confirm their current operative text and any 2021–2026 changes for our specific commodities.
- **Structural belt-and-braces regardless of the memo:** procurement happens at the **FPO's hub**, from the FPO's own member-farmers, with the FPO as aggregator — the most defensible configuration under any reading; if a unified/trader license is cheap and available, **obtain it even if arguably unnecessary** (a few thousand rupees buys immunity from harassment); if cess (~0.5–1%) is demanded despite the position memo, pay under protest and recover — never halt deliveries (R7 playbook).
- The memo must answer in writing: (a) is a license required for hub-gate purchase from FPO members — if yes, which and how obtained; (b) does market cess apply and at what rate; (c) does the answer change if the **FPO is the seller-of-record** to KisanSetu; (d) any commodity-specific rules for our two frozen crops. **Owner: founder + advocate, deadline: before dry-run week.**
- New-state expansions repeat this memo per state ([13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md) E2 checklist) — APMC law is state law; nothing transfers automatically.

## 6. FPO MoU — template outline (pilot, 12 months, renewable)
One page of recitals + these clauses (advocate drafts from this outline; Gujarati translation attached, English prevails):
1. **Parties & purpose:** FPO provides aggregation hub, farmer mobilization, grading labor; KisanSetu provides demand, software, grading training/charts, crates, price transparency.
2. **Hub access:** space for the 5 stations of [14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md) §2, 6 days/week, morning window; utilities; KisanSetu equipment (scale, printer, crates) remains KisanSetu property.
3. **Service expectations:** grading per KisanSetu charts by trained staff; supply-forecast signal 48h ahead; target supply reliability (M8 ≥90%) as a *shared goal*, with a review-not-penalty mechanism at pilot.
4. **Farmer payments:** paid by KisanSetu directly to farmers (instant UPI) — the FPO never handles KisanSetu payout money (clean audit line, protects both parties).
5. **FPO compensation:** per-kg hub facilitation fee (rate in schedule, from T0.9 economics) — not a margin on farmer price (farmer-share integrity, Golden Rule 5).
6. **Warranties:** FPO holds its FSSAI registration; produce is member-farmers' own; FPO board approval of the MoU.
7. **Data & IP:** grading data, timestamps and software remain KisanSetu's; FPO gets its own hub reports; groundwork clause for the future FPO-SaaS license (right-of-first-offer, no obligation).
8. **Exclusivity:** none demanded (FPO sells to whomever it likes) — but KisanSetu-listed produce is exclusively for KisanSetu fulfilment once graded (anti-double-selling clause, R5).
9. **Term & exit:** 12 months; either party may exit with 30 days' notice; immediate exit for documented grading manipulation or payment default (R2 kill criterion, [16-RISKS-MITIGATIONS.md](16-RISKS-MITIGATIONS.md)); equipment return protocol.
10. **Disputes:** escalation founder↔FPO CEO → mediation; jurisdiction Surat.

## 7. Buyer agreement — outline (one page + rate schedule)
1. Parties, buyer GSTIN/FSSAI (if held), delivery address(es), standing delivery window.
2. Ordering & cutoff (18:00 D-1), confirmation and partial-fill rules ([14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md) §4.2 — buyer acknowledges proactive-shortfall protocol).
3. Pricing: daily catalog prices per grade; prices fixed at confirmation; delivery/handling fee as separate taxable line.
4. Quality & claims: grades defined by the photo chart (mini-card annexed); **claims at door or within 2 hours with photo**; resolution by credit note/replacement; no returns after window (perishables).
5. Payment terms: pay-on-delivery/prepaid first 2 weeks, then net-7 cap at pilot; late payment → supply suspension (stated plainly — it is the enforcement mechanism).
6. Crates: KisanSetu property; rotation at each delivery; ₹350/crate charged after 3 deliveries unreturned.
7. Freshness SLA: <48h harvest→door **promise** with the traceability record as evidence; SLA miss remedy = delivery-fee waiver on the affected order (money-backed but capped — do not promise produce refunds for an on-time-but-late-harvest edge case; the timestamps define truth).
8. Data & privacy: order data used for operations and aggregate analytics; DPDP-compliant notice reference (§10).
9. Term: rolling, 7-day exit either side; jurisdiction Surat.

## 8. Farmer consent for payout-story marketing
The payout-slip reels and farmer stories ([04-GTM-SALES-MARKETING.md](04-GTM-SALES-MARKETING.md), T7.6) use real people's images, voices, village names and **income information** — sensitive in a village context. Rules:
- **Written + recorded consent** (form in Gujarati/Hindi, read aloud on camera for low-literacy signers): what will be filmed, exactly where it may appear (Instagram/YouTube/website/press), that the payout amount will be visible, no salary/land data beyond the slip, term (24 months), and **revocation at any time** via the ops number → content taken down within 7 days (best-effort on re-shares, stated honestly).
- No cash payment for testimonials (bought stories poison trust); a small gift (KisanSetu crate/cap) is fine and disclosed.
- No minors as subjects; bystanders blurred or consented.
- Consent forms live in the compliance file; a consent register maps each published asset → form. This consent is *separate from* and *in addition to* the DPDP service-consent of §10 — signing up for payouts never implies consenting to marketing.

## 9. Communications compliance
- **WhatsApp Business API:** all broadcast recipients (farmer price list, buyer price list) must have **recorded opt-in** (checkbox at onboarding + first-message confirmation); template messages per Meta policy; opt-out honored immediately. Opt-in state stored on the user record ([06-PRD-BACKEND.md](06-PRD-BACKEND.md)).
- **SMS OTP:** sender registration under TRAI DLT (done via the SMS gateway — MSG91/Twilio guide this); transactional route for OTPs.
- Meta lead-gen ads (T7.4): lead-form privacy-policy link mandatory → website privacy notice (§10).

## 10. Data privacy — DPDP Act 2023 (+ DPDP Rules)
KisanSetu is a **data fiduciary** for farmers, buyers, and leads (phone numbers E.164, names, villages, VPAs/UTRs, business details, order histories). Pilot-scale obligations we implement from day 1:
1. **Notice + consent at signup** (app and web, in EN/HI/GU — DPDP explicitly contemplates Eighth-Schedule languages; ours are ready by design): what we collect, why (orders, payouts, deliveries, service messages), who sees it (3PL gets delivery name/address only; Razorpay gets payout details), retention.
2. **Purpose limitation:** payout data is not marketing data (§8 separation); no sale of data, ever — also a stated brand position.
3. **Data-principal rights:** access/correction/erasure requests via the ops number + support email; grievance officer = founder (named on the website); respond within the Rules' timelines.
4. **Security safeguards:** JWT auth with role guards, TLS everywhere, payout credentials separated, Postgres backups encrypted, access on a named-person basis; **breach protocol**: contain → assess → notify the Data Protection Board and affected users per the Rules — drafted as a one-page runbook in the compliance file.
5. **Children's data:** not applicable by design (18+ commercial users); stated in the notice.
6. **Cross-border:** data stays on India-region infrastructure at launch (also the cheapest option); revisit per-country in Part B.
7. We are far below "significant data fiduciary" scale; re-check at each expansion stage.

## 11. Miscellaneous but mandatory
- **Legal Metrology:** the hub platform scale must be a verified/stamped weight-measure, re-verified annually — the payout depends on it and the farmer-facing display is the fairness mechanism ([14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md) §2.2). Keep the verification certificate at the hub.
- **Trademark:** file "KisanSetu" (or the cleared replacement) in classes **9, 31, 35, 39, 42** after T0.5 clearance; ™ usable on filing.
- **Insurance:** workmen's compensation for hub-present staff; goods-in-transit responsibility allocated in the 3PL contract ([14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md) §6.2); revisit product-liability cover at scale-up.
- **Employment:** offer letters + basic HR policies for hires #1–2; EPF/ESI thresholds (20/10 employees) not yet triggered — calendar-flagged for the hiring plan.

## 12. The compliance file & calendar
**File (physical + `legal/`):** CIN set, GST + FSSAI certificates, APMC memo, MoU + buyer agreements, consent registers (§8, §9, §10), scale certificate, insurance, trademark filings, board resolutions. This file is also data-room section 4 (T8.2).

| Rhythm | Item |
|---|---|
| Monthly | GSTR-1/3B; professional tax; payout-slip vs bank reconciliation sign-off |
| Quarterly | TDS returns; FSSAI turnover-threshold check (§2); DPDP request log review |
| Annually | ROC filings (AOC-4, MGT-7), audit, scale re-verification, FSSAI renewal, insurance renewal, MoU review |
| Per expansion | New place of business in GST; state APMC memo; hub scale stamping; local licenses |

---

# PART B — GLOBAL EXPANSION COMPLIANCE CHECKLIST

The product is global-first ([00-GOLDEN-RULES.md](00-GOLDEN-RULES.md) Rule 2); the compliance layer is the part that **never** transfers automatically. Everything in Part A is the *India instance* of a per-country dimension below. Country #2 entry ([13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md) E3) requires this checklist completed by local counsel **before** the first pilot order — budget it as a real line item (est. US$10–25k of local legal/tax setup per country).

| # | Dimension | The question to answer per country | India instance (Part A ref) |
|---|---|---|---|
| 1 | Entity & foreign ownership | Local subsidiary vs branch; foreign-ownership limits in agri-trading (several countries restrict them); minimum local directors/capital | Pvt Ltd (§1) |
| 2 | **Food safety licensing** | Which authority licenses a fresh-produce trader/distributor; hub registration; hygiene standards; per-crop rules | FSSAI (§2) |
| 3 | **Tax** | VAT/GST treatment of fresh produce (exempt? zero-rated? standard?); service-fee taxation; invoice/e-invoice formats; withholding on farmer payments; the principal-vs-agent question re-decided under local law | GST (§3–4) |
| 4 | Agricultural market regulation | The local APMC-equivalent: is direct farm-gate/co-op procurement legal, licensed, or fee-bearing; state/province variation | APMC Gujarat (§5) |
| 5 | Payments & instant rail | PSP licensing for payouts; the UPI-equivalent (PIX, PromptPay, M-Pesa, GCash…) and its business-payout rules; KYC on farmer recipients; cash-economy fallback rules | Razorpay/UPI (§3, [14-OPS-PLAYBOOK.md](14-OPS-PLAYBOOK.md) §7) |
| 6 | **Data protection & residency** | Local privacy law (consent model, breach duties); **data-residency mandates** — whether user/payment data must be stored in-country (drives per-region infrastructure in [11-ARCHITECTURE.md](11-ARCHITECTURE.md)); cross-border transfer rules | DPDP (§10) |
| 7 | Producer-organization law | Legal form of the FPO-equivalent (co-op, association); can it aggregate & sell members' produce; MoU enforceability against it | FPO MoU (§6) |
| 8 | Contracts & commercial law | B2B agreement enforceability, late-payment law (our supply-suspension lever), local-language requirements for consumer-adjacent terms | Buyer agreement (§7) |
| 9 | Weights & measures | Scale certification regime; metric labeling rules | Legal Metrology (§11) |
| 10 | Communications & marketing | WhatsApp/SMS marketing consent law; testimonial/likeness rules for farmer stories; ad-platform local rules | §8–9 |
| 11 | Labor & field staff | Hiring hub staff or contracting the producer org; contractor-vs-employee tests; minimum insurance | §11 |
| 12 | Brand & IP | Trademark availability in local classes; local-script transliteration of the brand | §11 trademark |

**Process rule for E3:** dimensions **2, 3, 5, 6** (bolded) are gating — a red on any of them defers the country, whatever the market math says. The rest can be yellow-with-a-plan at pilot start. Each country gets its own `legal/<country>/` file mirroring §12, and its own version of the §5-style advocate memo *before* the dry-run week — the launch playbook is verbatim ([13-LAUNCH-PLAN.md](13-LAUNCH-PLAN.md) §7), including its legal preconditions.

---

## Decisions taken in this document (flagged for founder + CA/advocate confirmation)
1. **Principal (buy–sell) legal structure at pilot**, not agent/marketplace — pending CA memo (§3). This is the single biggest structural choice; everything in §4 follows from it.
2. Voluntary GST registration from day 1.
3. Obtain any cheap available APMC/unified license even if arguably unnecessary (§5).
4. FPO never handles payout money (§6.4); FPO fee is per-kg facilitation, not a price margin (§6.5).
5. SLA-miss remedy = delivery-fee waiver, not produce refund (§7.7).
6. No cash payment for farmer testimonials (§8).
7. Bolded global dimensions (food safety, tax, payments, data) are hard gates for country entry (Part B).

Open items with owners: Gujarat APMC memo (founder + advocate, before dry-run week); GST structure memo (CA, end of Phase 0); trademark search & filing decision (founder, T0.5).
