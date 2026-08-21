# VibePulse Health

**Whoop-Style Insights from Apple Health Export Data** — 100% client-side, zero server uploads, total privacy.

[![Live Demo](https://img.shields.io/badge/Live-Demo-00f2fe?style=for-the-badge)](https://mohitd1x1t.github.io/apple-health-whoop-insights/)

---

## 🌟 Features

- **Recovery Score** — HRV + RHR derived gauge (green/yellow/red zones)
- **Day Strain** — Steps → 0–21 scale with active calorie & step breakdown
- **Sleep Performance** — Duration vs 8h 15m optimal, sleep-stage bar (Deep/REM/Core/Awake)
- **Health Monitor Vitals** — Latest RHR, HRV, SpO₂, Respiratory Rate with status indicators
- **Interactive Charts** — HRV, RHR, Steps, Energy, Sleep trends with glassmorphic tooltips
- **Weekly Digest** — One-click copy of 7-day averages (Recovery, Strain, Sleep, HRV, RHR, Steps, Active kcal)
- **Tabbed Dashboard** — Overview / Recovery / Strain / Sleep / Vitals
- **Time Granularity** — Daily / Weekly / Monthly aggregation
- **Sample Data** — One-click 90-day realistic dataset for instant exploration
- **Local Persistence** — Data auto-saves to IndexedDB; reloads instantly on revisit
- **Clear Data** — One-click wipe of all local data
- **PWA Ready** — Installable, offline-capable, neon pulse icon

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
├── sw.js             # Service worker (caches app + icon)
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
VibePulse: wk avg · Recovery: 72% · Strain: 13.2 · Sleep: 7.4h · HRV: 58.3ms · RHR: 51.2bpm · Steps: 8421 · Active: 412kcal
```

Paste into Notion, Obsidian, Messages, or your training log.

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
