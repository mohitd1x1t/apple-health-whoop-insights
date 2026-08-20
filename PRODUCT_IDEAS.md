# VibePulse — Product Enhancement Ideas

> Captured 2026-08-20. To revisit next day.

## Current State (as of last deploy `d833ac8`)
- Single-file PWA (`index.html`) parsing Apple Health `export.zip` locally (no server).
- Tabbed UI: Overview, Recovery, Strain, Sleep, Vitals.
- Daily / Weekly / Monthly range toggle.
- Dependency-free inline-SVG charts (line + stacked-bar for sleep stages).
- Real data parsing: HRV (SDNN), Resting HR, RHR, SpO2, Respiratory Rate, Steps, Active Energy, Sleep stages.
- HRV 30-day baseline shown as yellow reference line.
- Recovery algorithm: 65% HRV + 35% RHR, green/yellow/red zones.
- Strain from steps + active calories proxy.
- Service worker v2 (purges old caches on activate) + local `icon.svg`.

---

## Enhancement Backlog

### Data & Algorithms
- **HRV rolling baseline** — 7-day / 14-day / 30-day moving averages with deviation bands (±1 SD).
- **Recovery readiness model** — blend HRV, RHR, sleep quality, prior-day strain, temperature (if in export), and menstrual cycle phase.
- **Strain model upgrade** — use `HKQuantityTypeIdentifierActiveEnergyBurned` + heart-rate-in-zones if available (Apple Watch workout HR) instead of step proxy.
- **Sleep regularity index** — bedtime/wake-time consistency score (0–100).
- **VO₂ max estimation** — from outdoor walk/run HR + pace if GPS workouts exist.
- **Temperature deviation** — wrist temp (Apple Watch Series 8/Ultra) baseline vs current.

### Charts & UX
- **Interactive hover tooltips** on all SVG charts (show exact value + date).
- **Comparative overlay** — "This week vs last week" or "This month vs 3-month avg" toggle.
- **Weekly summary cards** — auto-generated Monday-morning digest (shareable image).
- **Export PNG/CSV** — download charts or raw aggregated data.
- **Dark/light theme toggle** — respect `prefers-color-scheme` + manual override.

### Privacy & Platform
- **IndexedDB persistence** — cache parsed data locally so re-upload isn't needed.
- **Shareable report link** — encode data in URL fragment (no server) for coach/trainer.
- **iOS shortcut / share-extension** — "Open in VibePulse" from Files app.

### Engagement
- **Streaks & badges** — 7-day HRV baseline streak, sleep consistency streak.
- **Weekly email (opt-in)** — static site + Netlify Functions or GitHub Actions to email digest.
- **AI coach chat** — local Ollama / cloud LLM (opt-in) for "Why is my recovery low?"

### Integrations
- **Whoop / Oura / Garmin CSV import** — unify multiple wearables.
- **HealthKit live sync (native iOS app)** — skip manual export.

---

## Suggested Next Steps (priority order)
1. IndexedDB persistence + chart hover tooltips (best UX-per-effort).
2. Weekly summary card generator (shareable PNG).
3. Recovery readiness model (richer algorithm).
4. Comparative overlay charts.

## Notes
- Deploy URL: https://mohitd1x1t.github.io/apple-health-whoop-insights/
- If old layout appears: hard refresh (Cmd+Shift+R) or unregister SW in DevTools → Application.
