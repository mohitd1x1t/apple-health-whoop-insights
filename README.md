# VibePulse Health

**Whoop-Style Insights from Apple Health Export Data** — 100% client-side, zero server uploads, total privacy.

[![Live Demo](https://img.shields.io/badge/Live-Demo-00f2fe?style=for-the-badge)](https://mohitd1x1t.github.io/apple-health-whoop-insights/)

---

## 🌟 Features

- **Recovery Score** — readiness scored against *your own* rolling baseline, not a fixed scale (see below)
- **Day Strain** — Steps → 0–21 scale with active calorie & step breakdown
- **Sleep Performance** — Duration vs 8h 15m optimal, sleep-stage bar (Deep/REM/Core/Awake)
- **Health Monitor Vitals** — Latest RHR, HRV, SpO₂, Respiratory Rate with status indicators
- **Interactive Charts** — HRV, RHR, Steps, Energy, Sleep trends with glassmorphic tooltips
- **HRV Baseline Band** — trailing 30-day mean plus a shaded ±1 SD deviation band
- **Compare Previous** — one toggle overlays the preceding window of equal length on every line chart
- **Weekly Digest** — One-click copy of 7-day averages (Recovery, Strain, Sleep, HRV, RHR, Steps, Active kcal)
- **Tabbed Dashboard** — Overview / Recovery / Strain / Sleep / Vitals
- **Time Granularity** — Daily / Weekly / Monthly aggregation
- **Sample Data** — One-click 90-day realistic dataset for instant exploration
- **Local Persistence** — Data auto-saves to IndexedDB; reloads instantly on revisit
- **Clear Data** — One-click wipe of all local data
- **PWA Ready** — Installable, offline-capable, neon pulse icon

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
| **Storage** | Native IndexedDB (no external deps) |
| **PWA** | Service Worker (`sw.js`) + Web App Manifest |
| **Icons** | Hand-crafted animated SVG (neon `#00f2fe` pulse) |
| **Zip** | [JSZip](https://stuk.github.io/jszip/) via CDN (only external script) |

---

## 📂 Project Structure

```
apple-health-whoop-insights/
├── index.html        # Single-file app (HTML + CSS + JS)
├── manifest.json     # PWA manifest (local SVG icon)
├── sw.js             # Service worker (network-first HTML, cache-first assets)
├── icon.svg          # Animated neon pulse icon (maskable)
└── README.md         # This file
```

---

## 📊 Supported HealthKit Types

| Category | Identifiers |
|----------|-------------|
| **Recovery** | `HeartRateVariabilitySDNN`, `RestingHeartRate` |
| **Strain** | `StepCount`, `ActiveEnergyBurned` |
| **Sleep** | `SleepAnalysis` (Deep, REM, Core, Awake, Unspecified) |
| **Vitals** | `OxygenSaturation`, `RespiratoryRate` |

> `OxygenSaturation` is normalised whether your export stores it as a fraction (`0.97`) or a
> percentage (`97`).

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

---

## 📈 Weekly Digest Format

Click **“📋 Copy Weekly Digest”** → clipboard receives:

```
VibePulse: wk avg · Recovery: 54% · Strain: 13.2 · Sleep: 7.4h · HRV: 58.3ms · RHR: 51.2bpm · Steps: 8421 · Active: 412kcal
```

Paste into Notion, Obsidian, Messages, or your training log. Recovery here is the average of the
same baseline-relative daily scores the dashboard shows, so ~50% is a normal week, not a bad one.

---

## 🧹 Clear Data

Click **“🗑 Clear Data”** → confirms → wipes IndexedDB + resets UI. No trace left.

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
- **Color palette**: Neon cyan `#00f2fe` + recovery green `#00e676`
- **Chart principles**: [dataviz skill](https://github.com/anthropics/claude-code-skills) — form-first, color-last, accessible by default
- **Icons**: Custom SVG — no external icon fonts

---

**Built with 💚 for quantified-self nerds who value privacy.**  
[Open an issue](https://github.com/mohitd1x1t/apple-health-whoop-insights/issues) • [Star the repo](https://github.com/mohitd1x1t/apple-health-whoop-insights) • [Live demo](https://mohitd1x1t.github.io/apple-health-whoop-insights/)
