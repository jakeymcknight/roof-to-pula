# Roof to Pula — Botswana household solar estimator

A single-page, phone-first tool for estimating what rooftop solar saves a Botswana
household on BPC grid electricity, and whether a small battery (2, 5 or 10 kWh) pays
for itself. A household-scale companion to the Solar SHIELD hospital tool.

Everything is in `index.html`: no build step, no dependencies, no API keys, no
tracking. Drop it on GitHub Pages and it works.

## Deploying to GitHub Pages

1. Create a new repository (e.g. `botswana-solar`).
2. Copy `index.html` into the root of the repo.
3. Settings → Pages → Build and deployment → Source: *Deploy from a branch*, branch
   `main`, folder `/ (root)`.
4. The tool is live at `https://<user>.github.io/<repo>/` within a minute or two.

The satellite trace needs internet and loads Esri World Imagery tiles directly. That
works on GitHub Pages; it does not work where a host blocks outside images (the
published Claude artifact version, for instance), and the tool falls back to manual
measurement with a visible notice when tiles fail to load. If you publish this
somewhere public and expect real traffic, check Esri's terms for the basemap or swap
the `TILE` constant near the bottom of the script for your own imagery source.

## What the tool does

**Scope: savings against BPC grid electricity only.** No export credit, no net
metering, no generator comparison, no financing, no carbon valuation. Surplus solar
the household cannot use and the battery cannot store earns nothing — which is what
makes the battery question interesting, and usually negative.

1. **Roof capture** — measure one roof face (length × width, in metres, pace-able),
   or trace it on a satellite view. Panels are laid out for real, trying both portrait
   and landscape with a 0.30 m edge setback, and the better fit wins.
2. **Tariff** — BPC domestic energy charges as cumulative monthly blocks. Solar
   displaces consumption from the *top* block downwards, so the tool values a saved
   kWh at its marginal rate, never the average. A monthly spend in Pula is inverted
   back to units through the same blocks.
3. **Hourly simulation** — 12 months × 24 hours, using a residential load shape
   rescaled to the chosen daytime-occupancy preset and a Botswana PV generation shape.
   This produces direct self-consumption, battery throughput and wasted surplus,
   rather than a hand-waved self-consumption percentage.
4. **Money** — installed cost built from unit rates, then 20-year appraisal with real
   tariff escalation, panel degradation, maintenance, one inverter replacement and, if
   fitted, a battery replacement at end of warranty life. Reports simple payback and
   discounted net benefit.
5. **Right-sizing** — sweeps every panel count the roof allows and marks the one with
   the best net return. On a small roof with no export payment, filling the roof is
   almost never the best value, and the tool says so plainly.

## Assumptions

Every figure is editable in the tool's *Assumptions* panel, and the full set — with
sources, confidence levels and a worked example to check the tool against — is in the
companion workbook `Botswana_Home_Solar_Key_Figures.xlsx`.

Headline sources:

- **Tariff blocks** — BERA, *2026-27 BPC Electricity Tariff Application Consultation
  Paper*, Table 4. The 2026/27 rates were proposed effective 1 April 2026; confirm
  against a current bill. The 2025/26 column is included and selectable.
- **Solar resource** — World Bank / ESMAP Global Solar Atlas. Botswana's practical
  potential is about 5.1–5.3 kWh/kWp/day; town values are interpolated and rounded.
- **Equipment costs** — benchmarked from 2026 South African retail pricing converted
  at 1 BWP = 1.181 ZAR with a 10% Botswana landed-cost uplift. **Replace these with
  local quotes before the output is used for advice.**

The weakest assumption is the household load shape: it is a plausible Botswana
profile, not metered data, and it is the single input that most changes the battery
answer. It is exposed as an occupancy preset for that reason.

## Editing the model

The script is one IIFE at the bottom of `index.html`, in plain ES2017 with no
framework. The parts worth knowing:

| What | Where |
|---|---|
| All default assumptions | `const DEF = { … }` |
| Towns, orientation, shading, occupancy tables | `TOWNS`, `DIRS`, `SHADES`, `OCCS` |
| Hourly load and PV shapes | `LOAD_H`, `PV_H` |
| Tariff maths | `bill()`, `kwhFromSpend()` |
| Panel layout | `panelFit()` |
| Energy balance | `simulate()` |
| Costs and appraisal | `capex()`, `finance()` |
| Satellite tiles | `TILE`, `drawTiles()` |

Inputs are kept in `localStorage` under `roof2pula` so a returning user's roof and
bill survive a reload; *Reset to defaults* clears it.

## Regression check

With the default assumptions, Gaborone, a north-facing medium-pitch roof with light
shading, someone home during the day, and a P600/month bill (≈ 501 kWh):

| | 2 panels (0.90 kWp) | 6 panels (2.70 kWp) |
|---|---|---|
| Generated | 1,607 kWh/yr | 4,822 kWh/yr |
| Used directly | 1,434 kWh/yr | 1,936 kWh/yr |
| Wasted | 174 kWh/yr | 2,886 kWh/yr |
| Installed cost | P22,085 | P39,611 |
| Saving | P2,235/yr | P2,901/yr |
| 5 kWh battery catches | 156 kWh/yr | 1,478 kWh/yr |

If a change to the model moves these, it was not a cosmetic change.

## Licence and attribution

Built for the Health Systems Collaborative, Centre for Tropical Medicine and Global
Health, University of Oxford. Tariff and cost figures are third-party data cited
above; the tool itself is yours to license as you see fit.
