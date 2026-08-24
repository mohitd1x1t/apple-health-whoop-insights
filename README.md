# VibePulse Health

**Whoop-Style Insights from Apple Health Export Data** — 100% client-side, zero server uploads, total privacy.

[![Live Demo](https://img.shields.io/badge/Live-Demo-00f2fe?style=for-the-badge)](https://mohitd1x1t.github.io/apple-health-whoop-insights/)

---

## 🌟 Features

- **Recovery Score** — readiness scored against *your own* rolling baseline, not a fixed scale (see below)
- **VO₂ Max — computed by us, not read from Apple** — a daily estimate from your workouts, HR and profile, with fitness age and age/sex percentile (see below)
- **Day Strain** — Steps → 0–21 scale with active calorie & step breakdown
- **Sleep Performance** — Duration vs 8h 15m optimal, sleep-stage bar (Deep/REM/Core/Awake)
- **Health Monitor Vitals** — Latest RHR, HRV, SpO₂, Respiratory Rate with status indicators
- **Interactive Charts** — HRV, RHR, Steps, Energy, Sleep, VO₂ max trends with hover tooltips
- **HRV Baseline Band** — trailing 30-day mean plus a shaded ±1 SD deviation band
- **Compare Previous** — one toggle overlays the preceding window of equal length on every line chart
- **Weekly Digest** — One-click copy of 7-day averages (Recovery, Strain, Sleep, HRV, RHR, Steps, Active kcal, VO₂ max)
- **Tabbed Dashboard** — Overview / Recovery / Strain / Sleep / VO₂ Max / Vitals
- **Time Granularity** — Daily / Weekly / Monthly aggregation
- **Profile Card** — age, sex, height, weight read from your export where present, editable where not
- **Readiness-Driven Accent** — the whole UI shifts color with today's recovery band
- **Validated Palette** — every color checked with the data-viz validator, not picked by eye (see Credits)
- **Sample Data** — One-click 90-day realistic dataset for instant exploration
- **Local Persistence** — Data auto-saves to IndexedDB; reloads instantly on revisit
- **Clear Data** — One-click wipe of all local data (your profile is kept)
- **Reduced-Motion Aware** — every animation is suppressed under `prefers-reduced-motion`
- **PWA Ready** — Installable, offline-capable

---

## 🧠 How the Recovery Score Works

The score is **relative to your own baseline**, not an absolute scale. Recovery only means anything
compared to *your* normal — a 45 ms HRV baseline is not "unrecovered," it's just your number — so
each input is scored as a deviation (in standard deviations) from your own trailing 30 days:

| Input | Weight | Direction |
|-------|--------|-----------|
| HRV vs your baseline | 50% | Higher is better |
| Resting HR vs your baseline | 25% | **Lower** is better |
| Sleep quality (65% duration + 35% Deep+REM share) | 15% | Higher is better |
| Yesterday's strain vs your baseline | 10% | **Lower** is better (load is a debt) |

Each deviation is clamped to ±2 SD, combined into a weighted mean, and mapped as
`50 + 25 × z`, clamped to 1–99. So **50% means "right at your own baseline."**

Two consequences worth knowing:

- **Missing inputs are treated as unknown, not zero.** If your export has no sleep data, the sleep
  weight is redistributed across the inputs you *do* have rather than scoring you as if you hadn't
  slept.
- **Cold start.** Until 14 days of HRV history exist, there's no trustworthy baseline, so the app
  falls back to an absolute HRV/RHR formula and says so in the Recovery tab.

The Recovery tab shows which inputs drove today's number (e.g. *HRV +1.2 SD · RHR −0.4 SD*) so the
score is explainable rather than a bare percentage.

---

## 🫁 How the VO₂ Max Estimate Works

**This number is computed by VibePulse. It is not Apple's `VO2Max` record.** Apple only writes one
of those during an outdoor walk or run with the watch, so a chart of them is a handful of dots
across a year. We derive an estimate for *every* day instead, from data you already have.

The model picks the best method your data supports and always tells you which one it used, because
the three tiers are not equally trustworthy:

| Tier | Method | Needs | Used when |
|---|---|---|---|
| **1** | Submaximal HR/speed extrapolation (ACSM metabolic equations + HR-reserve) | **Running** workouts with duration, distance and average HR | ≥3 qualifying runs in the trailing 28 days |
| **2** | Uth–Sørensen HR ratio — `15.3 × HRmax / RHR` | Resting HR + an HRmax | You have RHR but don't log workouts |
| **3** | Jackson non-exercise model | Age, sex, BMI, activity volume | Neither of the above |

Tier 1, per qualifying workout:

```
speed  = distance_m / duration_min                 # m/min
VO2eff = speed >= 134 ? 0.2*speed + 3.5            # ACSM running
                      : 0.1*speed + 3.5            # ACSM walking
f      = (avgHR - RHR) / (HRmax - RHR)             # HR-reserve fraction
VO2max = 3.5 + (VO2eff - 3.5) / f                  # HR reserve ∝ VO2 reserve
```

A workout must clear every gate — duration ≥ 8 min, distance > 0, average HR present,
`0.40 ≤ f ≤ 0.95`, result within 15–85 — or it is **discarded, not clamped**. A bad GPS trace
should drop out of the series, not bias it. The day's value is the **median** of the qualifying
workouts in the trailing 28 days.

Five things worth knowing:

- **Only running feeds tier 1.** The speed→O₂ step is the ACSM *running* equation, and the
  HR-reserve extrapolation only behaves for a sustained aerobic effort. Walking has too low an
  oxygen cost to extrapolate — on a walk-heavy log a real ~42 collapses toward ~16 — and
  cycling/rowing/strength either cover distance at a different economy or carry none at all. So
  walking, cycling and strength workouts are excluded from tier 1; if you don't run, the estimate
  comes from the HR-ratio or non-exercise tier instead, which is the honest answer rather than a
  wrong one dressed up as a measurement.
- **HRmax is observed, not assumed, and it's robust.** We take the **median of the top decile** of
  your daily HR maxima over the trailing year (≥20 days required), after discarding any reading
  outside 90–220 bpm. The obvious choice — a 99th percentile — is wrong at realistic sample sizes:
  with ~60 days of history the p99 index lands on the very top of the sorted array, so one sensor
  artifact leaks straight through. Measured on synthetic data, a single 240 bpm blip moved p99
  HRmax from 170 → 194 and the resulting VO₂max from 43.6 → 51.7. The median of the top 10% has a
  5% breakdown point, so several bad days can sit above it without moving it. Falls back to Tanaka
  (`208 − 0.7 × age`) when there isn't enough history.
- **Only one tier is ever plotted.** Tiers are different measuring instruments. Drawing a tier-2 day
  beside a tier-1 day produces a step at the switchover that looks like a fitness change and isn't
  one — on sample data, a 7-point cliff. Days from other tiers are excluded from the line, and the
  chart says how many were left out. The 90-day delta is computed the same way.
- **Fitness age has a floor.** The norm table starts at 25, so a strong estimate reads **≤25**
  rather than inventing precision the table doesn't have.
- **Apple's own readings are comparison-only.** The "Show Apple's readings" checkbox (off by
  default) draws them as hollow circles with a `our estimate X · Apple Y · Δ Z` footnote. They
  **never** feed the model.

> ⚠️ The age/sex percentile table (`VO2_NORMS`) follows Cooper Institute–style norms from background
> knowledge and **has not been verified against a primary source**. It is a single clearly-labelled
> constant so it can be corrected without touching the model. The estimate itself is a modelled
> number, not a lab measurement — treat it as a trend to watch, not a diagnosis.

---

## ⇄ Comparative Overlay

The **“⇄ Compare previous”** toggle overlays the preceding window of equal length on every line
chart — dashed and faded, sharing the y-scale so the two are actually comparable:

| Granularity | Compares |
|-------------|----------|
| Daily | Last 7 days vs the 7 before |
| Weekly | Last 4 weeks vs the 4 before |
| Monthly | Last 3 months vs the 3 before |

Hovering shows both values with the previous bucket's own date. The sleep-stage stacked bars are
deliberately excluded — two overlaid stacked series are unreadable, so that chart keeps showing the
full range.

---

## 🔐 Privacy Guarantee

> **Your health data never leaves your browser.**

- **Zero network requests** for data parsing or computation
- **No analytics, no tracking, no cookies** (except the PWA service worker for offline)
- **IndexedDB storage** is local to your device/browser profile
- **Export.zip** is read entirely in-memory via `JSZip` — nothing uploaded
- **Open source** — audit every line at `github.com/mohitd1x1t/apple-health-whoop-insights`

---

## 📥 How to Export Your Apple Health Data

### On iPhone (iOS 15+)

1. Open the **Health** app
2. Tap your **profile icon** (top-right)
3. Tap **“Export All Health Data”** → confirm
4. Wait for the zip to generate (can take 30–120 seconds for large exports)
5. Share → **Save to Files** → choose **On My iPhone** or iCloud Drive

### Transfer to Computer

- **AirDrop** the `export.zip` to your Mac
- Or use **Files app** → iCloud Drive → download on desktop

---

## 🚀 Quick Start

### Option A: Use the Live Site (Recommended)

Visit **[https://mohitd1x1t.github.io/apple-health-whoop-insights/](https://mohitd1x1t.github.io/apple-health-whoop-insights/)**

1. Click **“⚡ Load Sample Data”** to explore instantly
2. Or drag & drop your `export.zip` onto the dropzone

### Option B: Run Locally

```bash
git clone https://github.com/mohitd1x1t/apple-health-whoop-insights.git
cd apple-health-whoop-insights
# Serve with any static server, e.g.:
npx serve .        # or: python3 -m http.server 8080
# Open http://localhost:8080
```

> **Must serve over HTTP(S)** — `file://` protocol blocks IndexedDB, service workers, and JSZip.

---

## 🛠 Tech Stack

| Layer | Technology |
|-------|------------|
| **Parsing** | Streaming regex over `export.xml` (no DOM parser — memory efficient) |
| **Charts** | Dependency-free SVG (`<path>`, `<rect>`, `<circle>`) |
| **Storage** | Native IndexedDB (no external deps) — `historicalData` + `profile` keys |
| **PWA** | Service Worker (`sw.js`) + Web App Manifest |
| **Icons** | Hand-crafted animated SVG (maskable) |
| **Zip** | [JSZip](https://stuk.github.io/jszip/) via CDN (only external script) |

---

## 📂 Project Structure

```
apple-health-whoop-insights/
├── index.html        # Single-file app (HTML + CSS + JS)
├── manifest.json     # PWA manifest (local SVG icon)
├── sw.js             # Service worker (network-first HTML, cache-first assets)
├── icon.svg          # Animated pulse icon (maskable)
└── README.md         # This file
```

---

## 📊 Supported HealthKit Types

| Category | Identifiers |
|----------|-------------|
| **Recovery** | `HeartRateVariabilitySDNN`, `RestingHeartRate` |
| **Strain** | `StepCount`, `ActiveEnergyBurned`, `AppleExerciseTime` |
| **Sleep** | `SleepAnalysis` (Deep, REM, Core, Awake, Unspecified) |
| **Vitals** | `OxygenSaturation`, `RespiratoryRate` |
| **VO₂ max inputs** | `HeartRate`, `DistanceWalkingRunning`, `BodyMass`, `Height`, `SixMinuteWalkTestDistance`, `HeartRateRecoveryOneMinute` |
| **Elements** | `<Workout>` (both the modern `<WorkoutStatistics>` shape and the older element-attribute shape), `<Me>` (`DateOfBirth`, `BiologicalSex`) |
| **Overlay only** | `VO2Max` — parsed for the comparison overlay, never fed to the model |

> `OxygenSaturation` is normalised whether your export stores it as a fraction (`0.97`) or a
> percentage (`97`). `BodyMass`, `Height` and `DistanceWalkingRunning` are normalised from whatever
> `unit=` the record carries (`kg`/`lb`, `cm`/`in`, `km`/`mi`) to kg / cm / metres.

> `HeartRate` is **never stored raw** — it is by far the highest-volume type (samples seconds apart
> during a workout), so it is aggregated to `{min, max, sum, n}` per day inside the parse loop.
> Memory stays O(days) and IndexedDB stays small. It also gets its own uncapped counter: counting
> millions of HR samples against the shared record cap would silently truncate every later record
> of every other type.

> Missing a type? Open an issue or PR — parsing is a single `INTEREST` set.

---

## 🧪 Sample Data

Click **“⚡ Load Sample Data”** to generate 90 days of realistic synthetic data:
- HRV: 40–95 ms (daily variation ±6)
- RHR: 44–62 bpm
- SpO₂: 96–99%
- Respiratory Rate: 13–16 rpm
- Steps: 5k–14k/day
- Sleep: ~5–6h total (Deep 1–2.2h, REM 1.2–2.2h, Core 2.5–4h, Awake 0.2–0.8h)
- Daily heart-rate aggregates, exercise minutes and walking/running distance
- A Running workout every third day (with distance, duration and average HR) — enough to exercise
  the tier-1 VO₂ max path
- A profile (34, male, 178 cm, 74 kg) marked as export-sourced
- Monthly `VO2Max` records, deliberately offset from our estimate so the comparison overlay has
  something to disagree with

---

## 📈 Weekly Digest Format

Click **“📋 Copy Weekly Digest”** → clipboard receives:

```
VibePulse: wk avg · Recovery: 54% · Strain: 13.2 · Sleep: 7.4h · HRV: 58.3ms · RHR: 51.2bpm · Steps: 8421 · Active: 412kcal · VO2max: 47.6 (workout-based)
```

Paste into Notion, Obsidian, Messages, or your training log. Recovery here is the average of the
same baseline-relative daily scores the dashboard shows, so ~50% is a normal week, not a bad one.
VO₂ max is a level rather than a weekly average, so the digest quotes the latest estimate and names
the tier that produced it — a shared number shouldn't imply more precision than the method has.

---

## 🧹 Clear Data

Click **“🗑 Clear Data”** → confirms → wipes the stored health data + resets UI. Your **profile**
(age, sex, height, weight) is deliberately kept, since re-typing it on every import would be
tedious and it isn't health history.

---

## 🌐 Deploy Your Own (GitHub Pages)

1. Fork this repo
2. Settings → Pages → **Deploy from branch** → `main` / `/root`
3. Your site lives at `https://<your-username>.github.io/apple-health-whoop-insights/`
4. Update the badge in this README if you like

---

## 📝 License

MIT — free for personal and commercial use. Attribution appreciated.

---

## 🙏 Credits

- **Design inspiration**: WHOOP strap & app
- **Color palette**: validated with the data-viz validator against this app's real card surface
  (`#12161f`) rather than picked by eye. The previous neon palette had a genuine defect: sleep-stage
  Deep (`#5e35b1`) and REM (`#8e24aa`) measured OKLab ΔE **1.1 under protanopia** and 9.6 with full
  color vision, against a hard floor of 15 — the two bars were the same color to a red-green
  colorblind reader and near-identical to everyone else. Status colors are now reserved for state,
  text always wears a text token with a colored *mark* carrying identity, and the VO₂ percentile
  rail uses an ordinal blue ramp that passes at full opacity (it fails as a background wash — any
  alpha crushes the steps to ΔL 0.02–0.04).
- **Chart principles**: [dataviz skill](https://github.com/anthropics/claude-code-skills) — form-first, color-last, accessible by default
- **VO₂ max model**: ACSM metabolic equations; Uth–Sørensen HR ratio; Jackson non-exercise model;
  Tanaka HRmax formula
- **Icons**: Custom SVG — no external icon fonts

---

**Built with 💚 for quantified-self nerds who value privacy.**  
[Open an issue](https://github.com/mohitd1x1t/apple-health-whoop-insights/issues) • [Star the repo](https://github.com/mohitd1x1t/apple-health-whoop-insights) • [Live demo](https://mohitd1x1t.github.io/apple-health-whoop-insights/)
