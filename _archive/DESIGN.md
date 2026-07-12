# KisanSetu — Brand & Design System

One system across website, Android (Compose/Material3), iOS (SwiftUI), and print (payout slips, grading charts).

## Brand idea

**"Setu" = bridge.** The brand's job is trust on both ends of the bridge: a farmer who has been underpaid for generations, and a chef who has been burned by rotten deliveries. Everything is honest, legible, unfussy: real numbers, real farm names, no gloss.

## Color

| Token | Hex | Use |
|---|---|---|
| `forest` | `#14532D` | Primary — headers, primary buttons, brand surfaces |
| `leaf` | `#22C55E` | Accent — prices in farmer's favor, success, "fresh" markers |
| `cream` | `#FAF7F0` | Light background |
| `ink` | `#1C1917` | Text on light |
| `amber` | `#F59E0B` | Sparingly — grades, alerts, harvest highlights |
| `grade-a` | forest chip | Grade A (Premium) |
| `grade-b` | amber chip | Grade B (Standard) |

Semantic: success=leaf, warning=amber, error=`#DC2626`. Dark surfaces (iOS dark mode, dashboards): green-black `#0C1A12`, text cream.

## Typography

- **Website/print display:** serif with warmth (system stack: Iowan Old Style / Palatino / Georgia). Body: system sans.
- **Android:** Material3 default (Roboto) with an enlarged type scale — farmer app body ≥18sp, titles ≥24sp.
- **iOS:** SF Pro, Dynamic Type respected.
- **Devanagari/Gujarati:** system fonts (Noto/Mukta family on Android); always test line-height with Hindi strings — they need +10–15% over Latin.

## Voice

- Farmer-facing: Hindi first, English subtitle. Short, spoken-style: "फ़सल बेचें", "पैसा आ गया ✓". Numbers big.
- Buyer-facing: professional, specific: "Harvested yesterday, 6:10 AM · Olpad village". Freshness always stated in hours, not adjectives.
- Never "empowering farmers" corporate-speak. Show the payout slip instead.

## Farmer flow rules (both platforms — golden rule)

1. Max 2 actions per screen; one obvious primary button.
2. Touch targets ≥48dp; body ≥18sp.
3. Icon + text always together; produce chosen from a picture grid, not a dropdown.
4. No hamburger menu — bottom nav with 3 tabs max.
5. Every money number shows the comparison: mandi price struck through, KisanSetu price in leaf green.
6. Success screens celebrate plainly: amount, where it went (UPI), when.

## Buyer flow rules (both platforms — golden rule)

1. Restrained: cream/white surfaces, forest accents, no gimmicks.
2. Grade chips everywhere produce appears (A = forest, B = amber).
3. Traceability is the premium moment — vertical trace line: farm → hub → dispatch → door, with timestamps.
4. Order status as a timeline, not text.
5. Prices per kg with mandi comparison — we sell on trust arithmetic.

## Components (shared vocabulary)

- **Price card:** produce name (local language first), mandi price struck, our price large in leaf.
- **Status chip:** listing (नई/ग्रेड A·B/बिक गई) and order (placed→delivered) states, tinted backgrounds.
- **Trace line:** vertical stepper with dots — farm, hub, dispatch, delivered + timestamps.
- **Stat tile:** big tabular-nums figure + one-line caption (used on website + dashboards).

## Artifacts

- Visual design board (tokens + app mockups + website concept): `design-board.html` — published as a Claude artifact; open in browser.
