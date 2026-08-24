# VibePulse — Product Enhancement Ideas

> Captured 2026-08-20. Last updated 2026-08-22.

## Current State (as of last commit `9cc5a68` + the VO₂ max / re-skin stage)
- Single-file PWA (`index.html`) parsing Apple Health `export.zip` locally (no server).
- Tabbed UI: Overview, Recovery, Strain, Sleep, **VO₂ Max**, Vitals.
- Daily / Weekly / Monthly range toggle.
- Dependency-free inline-SVG charts (line + stacked-bar for sleep stages).
- Real data parsing: HRV (SDNN), Resting HR, SpO2, Respiratory Rate, Steps, Active Energy, Sleep
  stages, plus **daily HR aggregates, `<Workout>` elements, `<Me>` profile, body mass, height,
  walking/running distance, exercise minutes, 6-minute walk test, 1-minute HR recovery**, and
  Apple's own `VO2Max` records (overlay only, never fed to the model).
- HRV 30-day baseline shown as yellow reference line, with a shaded ±1 SD band.
- Recovery algorithm: baseline-relative readiness model (HRV, RHR, sleep quality, prior-day strain
  scored as deviations from the user's own trailing 30-day baseline). Falls back to the old absolute
  65% HRV + 35% RHR formula until 14 days of history exist.
- **VO₂ max engine**: three-tier model (ACSM submaximal HR/speed → Uth–Sørensen HR ratio → Jackson
  non-exercise), robust observed HRmax, fitness age, age/sex percentile rail.
- Strain from steps + active calories proxy.
- Comparative overlay toggle ("⇄ Compare previous") on every line chart.
- **Validator-checked palette** and motion (ring sweep, count-up, chart draw-in), all suppressed
  under `prefers-reduced-motion`.
- Service worker v4 — network-first for HTML so app updates land without a hard refresh.

---

## Enhancement Backlog

### Data & Algorithms
- ~~**HRV rolling baseline** — deviation bands (±1 SD).~~ **Done.** True trailing 30-day mean + SD,
  drawn as a shaded band behind the HRV charts. Multi-window (7/14/30-day) selection is still open.
- ~~**Recovery readiness model** — blend HRV, RHR, sleep quality, prior-day strain.~~ **Done**
  (baseline-relative z-score model; weights renormalise over whatever inputs are present, so missing
  data is treated as unknown rather than zero). Temperature and menstrual-cycle phase are still open,
  and both depend on those types being present in the export.
- ~~**VO₂ max estimation** — from outdoor walk/run HR + pace if GPS workouts exist.~~ **Done**, and
  broader than originally scoped: three tiers so users *without* GPS workouts still get a number,
  with the tier always disclosed. Only one tier is ever plotted, because mixing them draws a
  switchover cliff that reads as a fitness change. Still open: the `VO2_NORMS` percentile table is
  unverified against a primary source, and `SixMinuteWalkTestDistance` / `HeartRateRecoveryOneMinute`
  are parsed but not yet used as independent tiers.
- **Strain model upgrade** — use `ActiveEnergyBurned` + heart-rate-in-zones instead of the step
  proxy. **Now much cheaper than it was**: `DATA.hrDay` gives daily HR aggregates and `DATA.workouts`
  gives per-workout average/max HR, so the inputs exist. Partly addressed by the ACWR work planned
  for stage 3.
- **Sleep regularity index** — bedtime/wake-time consistency score (0–100). *Planned for stage 3.*
- **Temperature deviation** — wrist temp (Apple Watch Series 8/Ultra) baseline vs current.
- **Recovery trend chart** — `RECOVERY` is now a full date-keyed index, so plotting the score over
  time is mostly wiring. *Stage 3 takes this further as a year heatmap.*

### Charts & UX
- ~~**Interactive hover tooltips** on all SVG charts (show exact value + date).~~ **Done** — 11
  charts, all verified firing with correct units.
- ~~**Comparative overlay** — "This week vs last week" toggle.~~ **Done.** Global toggle; daily
  compares last 7 vs previous 7, weekly 4 vs 4, monthly 3 vs 3. Sleep-stage stacked bars are
  deliberately excluded (two stacked series are unreadable overlaid).
- ~~**More interactive, less templated UI**~~ **Done** (re-skin + motion): three-step surface
  hierarchy, validated palette, recovery-band-driven accent, ring sweep, count-up, chart draw-in,
  KPI sparklines. The single shared `linear-gradient(135deg, #00e676, #00b0ff)` that was doing duty
  for the logo, the active tab *and* the primary button is gone — one gradient on three unrelated
  things was most of why the app read as templated.
- **Weekly summary cards** — auto-generated Monday-morning digest (shareable image). The text digest
  ships (now including VO₂ max); the shareable PNG does not. *Narrative cards planned for stage 3.*
- **Export PNG/CSV** — download charts or raw aggregated data.
- **Dark/light theme toggle** — respect `prefers-color-scheme` + manual override. Note: the palette
  is currently validated for the **dark** surface only; a light mode needs its own validated steps,
  not an automatic flip.

### Privacy & Platform
- ~~**IndexedDB persistence** — cache parsed data locally so re-upload isn't needed.~~ **Done.**
  Now two keys: `historicalData` and `profile`, the latter surviving Clear Data.
- **Shareable report link** — encode data in URL fragment (no server) for coach/trainer.
- **iOS shortcut / share-extension** — "Open in VibePulse" from Files app.

### Engagement
- **Streaks & badges** — 7-day HRV baseline streak, sleep consistency streak. *Planned for stage 3.*
- **Weekly email (opt-in)** — static site + Netlify Functions or GitHub Actions to email digest.
- **AI coach chat** — local Ollama / cloud LLM (opt-in) for "Why is my recovery low?"
  *Planned for stage 2: Gemini, user-supplied key, aggregates-only context, explicit opt-in, and an
  inspectable "show exactly what will be sent" disclosure. Requires correcting the README's
  "zero network requests" claim in the same commit.*

### Integrations
- **Whoop / Oura / Garmin CSV import** — unify multiple wearables.
- **HealthKit live sync (native iOS app)** — skip manual export. *Investigated 2026-08-22: not
  possible from a web app. HealthKit has no web API and no browser permission prompt; only a native
  iOS app (or a Shortcuts automation writing a file) can read it. Manual export stays the path.*

---

## Suggested Next Steps (priority order)
1. ~~IndexedDB persistence + chart hover tooltips (best UX-per-effort).~~ **Done** (`5babc98`).
2. ~~Recovery readiness model (richer algorithm).~~ **Done.**
3. ~~Comparative overlay charts.~~ **Done.**
4. ~~VO₂ max engine + re-skin/motion.~~ **Done** (stage 1).
5. **Gemini assistant** (stage 2) — with the privacy-documentation correction in the same commit.
6. **Engagement features** (stage 3) — recovery calendar/streaks/PRs; training load ACWR + personal
   strain ceiling; illness early-warning + sleep debt; weekly story cards + sleep regularity.
7. Strain model upgrade (heart-rate-in-zones instead of the step proxy) — the inputs now exist in
   `DATA.hrDay` / `DATA.workouts`.
8. Weekly summary card generator (shareable PNG) — the *text* digest ships; the image does not.

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
- **`HeartRate` cannot share the record cap.** It is orders of magnitude higher-volume than any
  other type, so counting its samples against the shared `RECORD_CAP` would silently truncate every
  later record of every *other* type in a multi-year export. It has its own uncapped counter and is
  aggregated to `{min,max,sum,n}` per day in the parse loop — never stored raw.
- **Ordinal ramps can't be background washes.** The VO₂ percentile ramp passes its validator checks
  at full opacity (adjacent ΔL ≥ 0.06, light end 2.23:1) but fails at the 0.22 alpha a background
  needs — the five steps composite to `#132339`…`#3b434f`, adjacent ΔL 0.02–0.04 and 1.14:1
  contrast. It ships as a full-opacity rail in the right gutter instead. Validate the color as it
  *composites*, not as authored.
- **`requestAnimationFrame` is starved when the page isn't painting** (backgrounded tab, prerender).
  Both the gauge sweep and the count-up needed non-rAF backstops — a synchronous
  `getBoundingClientRect()` style flush and a `setTimeout`, respectively — or they left the UI stuck
  at its start value.
