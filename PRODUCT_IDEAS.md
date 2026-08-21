# VibePulse — Product Enhancement Ideas

> Captured 2026-08-20. Last updated 2026-08-21.

## Current State (as of last deploy `d833ac8`)
- Single-file PWA (`index.html`) parsing Apple Health `export.zip` locally (no server).
- Tabbed UI: Overview, Recovery, Strain, Sleep, Vitals.
- Daily / Weekly / Monthly range toggle.
- Dependency-free inline-SVG charts (line + stacked-bar for sleep stages).
- Real data parsing: HRV (SDNN), Resting HR, RHR, SpO2, Respiratory Rate, Steps, Active Energy, Sleep stages.
- HRV 30-day baseline shown as yellow reference line, with a shaded ±1 SD band.
- Recovery algorithm: baseline-relative readiness model (HRV, RHR, sleep quality, prior-day strain
  scored as deviations from the user's own trailing 30-day baseline). Falls back to the old absolute
  65% HRV + 35% RHR formula until 14 days of history exist.
- Strain from steps + active calories proxy.
- Comparative overlay toggle ("⇄ Compare previous") on every line chart.
- Service worker v3 — network-first for HTML so app updates land without a hard refresh.

---

## Enhancement Backlog

### Data & Algorithms
- ~~**HRV rolling baseline** — deviation bands (±1 SD).~~ **Done.** True trailing 30-day mean + SD,
  drawn as a shaded band behind the HRV charts. Multi-window (7/14/30-day) selection is still open.
- ~~**Recovery readiness model** — blend HRV, RHR, sleep quality, prior-day strain.~~ **Done**
  (baseline-relative z-score model; weights renormalise over whatever inputs are present, so missing
  data is treated as unknown rather than zero). Temperature and menstrual-cycle phase are still open,
  and both depend on those types being present in the export.
- **Strain model upgrade** — use `HKQuantityTypeIdentifierActiveEnergyBurned` + heart-rate-in-zones if available (Apple Watch workout HR) instead of step proxy.
- **Sleep regularity index** — bedtime/wake-time consistency score (0–100).
- **VO₂ max estimation** — from outdoor walk/run HR + pace if GPS workouts exist.
- **Temperature deviation** — wrist temp (Apple Watch Series 8/Ultra) baseline vs current.
- **Recovery trend chart** — `RECOVERY` is now a full date-keyed index, so plotting the score over
  time is mostly wiring.

### Charts & UX
- ~~**Interactive hover tooltips** on all SVG charts (show exact value + date).~~ **Done.**
- ~~**Comparative overlay** — "This week vs last week" toggle.~~ **Done.** Global toggle; daily
  compares last 7 vs previous 7, weekly 4 vs 4, monthly 3 vs 3. Sleep-stage stacked bars are
  deliberately excluded (two stacked series are unreadable overlaid).
- **Weekly summary cards** — auto-generated Monday-morning digest (shareable image). The text digest
  ships; the shareable PNG does not.
- **Export PNG/CSV** — download charts or raw aggregated data.
- **Dark/light theme toggle** — respect `prefers-color-scheme` + manual override.

### Privacy & Platform
- ~~**IndexedDB persistence** — cache parsed data locally so re-upload isn't needed.~~ **Done.**
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
1. ~~IndexedDB persistence + chart hover tooltips (best UX-per-effort).~~ **Done** (`5babc98`).
2. Weekly summary card generator (shareable PNG) — the *text* digest ships; the image does not.
3. ~~Recovery readiness model (richer algorithm).~~ **Done.**
4. ~~Comparative overlay charts.~~ **Done.**
5. Strain model upgrade (heart-rate-in-zones instead of the step proxy) — now the biggest remaining
   gap in the algorithms, since recovery already accounts for prior-day strain and inherits the
   step-proxy's inaccuracy.
6. Recovery trend chart over time (cheap, given the new `RECOVERY` index).

## Notes
- Deploy URL: https://mohitd1x1t.github.io/apple-health-whoop-insights/
- The old "if old layout appears, hard refresh" workaround is no longer needed: it was a symptom of
  the service worker serving *every* request cache-first, so `index.html` updates never reached
  users. SW v3 is network-first for navigations/HTML (with a cache fallback for offline) and
  cache-first only for static assets.
- **Bucket keys must be built from local date parts, not `toISOString()`.** Fixed 2026-08-21: the
  aggregation helpers stringified a local-midnight `Date` via `toISOString()`, which shifts the
  calendar day in any non-UTC zone. In UTC+5:30 that silently dropped the most recent
  day/week/month from every chart (so "latest" tiles showed yesterday) and put the monthly view a
  whole month off. Use the `ymd()` helper.
