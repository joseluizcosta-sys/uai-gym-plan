# UAI Dashboard — Conventions

Single-page training/health dashboard. `index.html` is large (~14 MB) because
assets (fonts/images) are embedded as base64 — **do not try to read the whole
file**; grep for the specific anchor you need (`function show`, `id="p-garmin"`,
etc.) and edit in place. Lines can be very long (single-line minified blocks) —
when slicing with `sed`/`grep`, pipe through `cut -c1-N` or use Python
`open(...).read()[a:b]` to avoid dumping huge single lines into context.

**Body-weight tracking was removed (Jul/2026)** — there is no more `weightChart`,
`weight_data.json`, or GitHub-Contents-API weight-sync form in `index.html`.

Repo: `joseluizcosta-sys/uai-gym-plan` · published via GitHub Pages (`main`, `.nojekyll`).

## Multi-race context (since Jul/2026)

José now trains toward **5 independent races**, each with its own periodization
(not one shared block toward a single goal):

| Race | Date | Distance | D+ |
|---|---|---|---|
| UAI 100K | 25/07/2026 | 100 km | +1.733 m |
| A Muralha Up and Down | 16/08/2026 | 42,195 km | +1.324 m |
| Paraty Brazil by UTMB — PTR58 | 19/09/2026 | 58 km | +3.400 m |
| Brazil135 Ultramarathon (solo) | 07/01/2027 | 239,4 km (GPX 2026)* | ~5.000–8.400 m** |
| Comrades Marathon 2027 (100ª edição, Down Run) | 13/06/2027 | ~87 km | alto impacto excêntrico |

*Brazil135 distance/elevation come directly from the official 2026 GPX (uploaded by
José) — measured at 239.4 km, notably longer than the historic "135mi/217km"
branding. **D+ is method-sensitive (raw GPS elevation noise); range reflects
different smoothing levels, not a single authoritative figure.

Comrades 2027 is a **strength-training target only** for now — no Provas-tab
altimetry/pacing yet (added 13/06/2027 date + Down Run format from official
comrades.com; full race detail pending closer to registration). Its "Branca
Esportes" strength protocol (Aug/2026 PDF) drives two new Força exercises
(Avanço Reverso on Wednesday, Extensora Unilateral Excêntrica on Thursday),
both gated to start **18/08/2026** (after A Muralha) to avoid adding novel
eccentric loading during the UAI taper/race or the tight A Muralha turnaround.
When adding new exercises sourced from an external coach protocol like this,
always check the current phase against the race calendar before making the
change effective immediately — flag the conflict and propose a start date
instead of applying blindly.

The **Plano Branca (`#p-garmin`) tab always reflects whichever race is nearest** —
it is a rolling single plan, not 4 parallel plans. When a race passes, rewrite the
`#p-garmin` weekly-plan content (and the "🎯 Prova-alvo atual" banner near the top
of that tab) to target the next race on the list. This is a manual editorial step,
not automatic logic.

## Data files (source of truth)

- `garmin_metrics.json` — `{ "last_updated": "YYYY-MM-DD", "entries": [ … ] }`.
  Each entry: `date, hrv, rhr, sleep_score, sleep_hours, body_battery_peak,
  body_battery_end, steps, weight, body_fat, activities[]`.
  Each activity: `type, name, distance_km, duration_min, pace_min_per_km,
  hr_avg, elevation_gain_m, training_load, hr_zones {z1_sec…z5_sec}`.
  **Entries are newest-first** (prepend new dates to the top of the array).

## Data Entry Rules

- **ALWAYS add a NEW dated entry; never edit/overwrite an existing dated row.**
  If a row for the same date already exists, ask before changing it — do not
  silently update it.
- Confirm the date is **today's actual date** before writing. State the date you
  are using out loud.
- Never claim "no changes needed" and stop — if asked to update, add the row.
- After adding to `garmin_metrics.json`, bump `last_updated` to the new date.

## Tab / HTML structure (the thing that keeps breaking)

- Top-level tabs are `<div class="page" id="p-<id>">`; `show(id, btn)` toggles
  class `a`. CSS: `.page{display:none}` / `.page.a{display:block}`.
- **Current tab ids (4 total, since Jul/2026 reformulation):**
  - `garmin` — **"Plano Branca"**, the default/active tab on load
    (`<div class="page a" id="p-garmin">`). Weekly plan + Garmin recovery
    analysis (HRV/RHR/BB charts). Has a "🎯 Prova-alvo atual" banner near the
    top — update it manually when the target race changes.
  - `forca` — **"Força"**. A single tab holding 4 internal day sub-panels
    (`<div class="forca-day" id="fd-monday|tuesday|wednesday|thursday">`),
    toggled by a `<select id="forca-day-select" onchange="showForcaDay(this.value)">`
    — NOT by the top-level `show()`/nav mechanism. CSS: `.forca-day{display:none}`
    / `.forca-day.a{display:block}`. To edit a specific day's exercises, find
    `id="fd-<day>"` and edit inside that sub-panel; don't touch the `<select>`
    unless adding/removing a day.
  - `nutricao` — **"Nutrição"**. Static content, pre-race + cotidiana +
    suplementos. Rarely changes.
  - `provas` — **"Provas"**. A list view (`#provas-list-view`, grid of
    `.prova-card`, each `onclick="showProva('<raceid>')"`) plus 5 detail
    sub-panels (`<div class="prova-detail" id="pd-uai|amuralha|utmbparaty|br135|comrades">`),
    toggled by `showProva(id)` / `showProvasList()` — again NOT the top-level
    `show()` mechanism. CSS: `.prova-detail{display:none}` / `.prova-detail.a{display:block}`.
    Each `pd-*` panel starts with a `<button class="prova-back" onclick="showProvasList()">`.
- **There are now THREE independent show/hide mechanisms in this file** — don't
  mix them up: `show(id,btn)` for top-level nav tabs, `showForcaDay(day)` for
  the day picker inside Força, `showProva(id)`/`showProvasList()` for the race
  picker inside Provas.
- **Any element placed OUTSIDE a `.page` div shows on EVERY tab.** This is the
  recurring bug (e.g. the VFC & RHR chart leaking into all tabs because it sat
  outside `#p-garmin`). Charts/analysis for Plano Branca must live **inside
  `#p-garmin`**, within its grid container. Same rule applies one level down:
  anything meant for one day must live inside its own `#fd-<day>`, and anything
  meant for one race must live inside its own `#pd-<raceid>`.
- A single unclosed `</div>` nests later pages inside the wrong parent and can
  erase a whole tab. **Before committing, verify div balance** around the region
  you edited (see checklist below).
- **Removed in Jul/2026:** the old `periodization`, `race`, `strategy`, `monday`,
  `tuesday`, `wednesday`, `thursday` top-level tab ids no longer exist — don't
  reference them. Periodization phase tables were dropped (redundant with the
  weekly plan); UAI's race detail + strategy content now lives inside
  `#pd-uai`; the 4 day-tabs were consolidated into `#p-forca`.

## Adding a new race to the Provas tab

1. Research distance/D+/cutoff/date from the **official source only** (see
   Research/Sources below).
2. Add a `.prova-card` to `#provas-list-view` with `onclick="showProva('<id>')"`.
3. Add a matching `<div class="prova-detail" id="pd-<id>">` panel (copy the
   structure of `#pd-amuralha` or `#pd-utmbparaty` as a template — `.profile-grid`
   of `.info-box` cards for facts, `.sci-grid` of `.sci-card` for rules/notes).
   Every panel needs the `<button class="prova-back" onclick="showProvasList()">`
   as its first child.
4. If GPX/altimetry/pace data isn't available yet, say so in the panel (see the
   "em breve" notes in the existing non-UAI panels) rather than fabricating it.

## Per-card conventions (don't skip these)

- **HR-zone mini-bar on EVERY day card — including Tuesday.** Don't omit a card
  just because the data looks empty. (This applies to `.gday` cards inside
  `#p-garmin`'s weekly plan — unrelated to the Força day sub-panels.)
- Apply the **check-icon convention** for completed/done days consistently.
- Keep new chart canvases inside their tab's grid; match the surrounding card
  markup (`.garmin-week`, `.gday`, etc.).

## Pre-commit checklist

1. The edited region's `<div>`/`</div>` count is balanced and content stays in
   its own `#p-<id>` page (or, for Força/Provas, its own `#fd-<day>`/`#pd-<raceid>`
   sub-panel).
2. Every day card (incl. Tuesday) has its HR-zone mini-bar; check-icons applied.
3. New entries are NEW dated rows with today's correct date; `last_updated` bumped.
4. Open each affected tab mentally: nothing leaks across tabs, and nothing from
   one Força day or one Provas detail leaks into another.

## GitHub / Deployment

- Commit + push to `main`; GitHub Pages serves it.
- There is no more in-page GitHub-Contents-API sync form (that was the weight
  form, now removed). If you reintroduce any client-side GitHub write, follow
  the old pattern's precautions: `cache:'no-store'` + `?t=<timestamp>` buster,
  never `raw.githubusercontent.com` (≈5-min CDN cache breaks cross-device sync).

## Research / Sources

- When researching races, supplements, or training data, use **ONLY official
  sources** (official event site, federation). No blogs/aggregators unless
  explicitly asked. If official sources don't have it, say so — don't guess.

## Updating this dashboard

Prefer the `/update-dashboard` skill, which encodes the fetch → add-dated-row →
mini-bars → div-integrity → deltas → commit → push routine.
