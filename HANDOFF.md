# Veltor website — handoff

## 2026-11-08: pricing and support brought up to date with app build 138

The live site still said "Pro is a single purchase … There's no subscription"
(flagged in the marketing plan as a must-fix before campaign traffic). Build
138 (`748a537`) sells Veltor Pro monthly / yearly (7-day free trial, new
subscribers) / lifetime, plus GPS Pro as a one-time purchase for GPS mode.

- **Homepage pricing** is now three cards: Free (Tesla, no ads, two sounds
  earned at 7 and 30 days), Veltor Pro (monthly, yearly or lifetime; includes
  GPS Pro and every sound), GPS Pro (free GPS mode = GPS speed, Veltor dash,
  4 sounds, drive log, banner ad; GPS Pro adds every GPS dashboard design,
  every sound, the HUD, no ads). Still **no prices** on the site: the store
  shows them per currency (`docs/store_copy.md` rule), and no strikethroughs.
  `.card` is now a flex column so the price rows sit at the card bottom.
- Set-up step 1 no longer promises the HUD for "any other car" (it's GPS Pro).
- Final CTA: "on its way to the App Store and Google Play" (was "in testing").
  Store badges still SOON: swap for real links once both listings are live.
- **Support page**: new "Veltor Pro and GPS Pro" section (what each is,
  Restore = Settings → Veltor Pro / GPS Pro → Restore purchases, cancel in
  the store); privacy blurb no longer says "collects nothing" (matches the
  privacy page's AdMob section); GPS mode and the demo drive's real path
  (Settings → Car & connection) added to Getting started.

Checked at 1440px and 375px: no horizontal overflow, console clean.

## 2026-11-02 (pm, 2): charging — split content from the nav bar

The real-crop fix from the prior round removed the correct amount of dead
space, but as a single image that's now shorter than the phone frame, it
left the gap at the BOTTOM instead (the nav bar floating above empty
space) — cropping top-only pixels can't both close the top gap and still
exactly fill a fixed-aspect frame without one.

Fixed properly by splitting the one `charging.webp` into two assets,
measured from the same `charging_dc_colourful.png` source:
- `charging-content.webp` — top 10% trimmed, same as before, everything
  down to the nav bar's own top edge (found by sampling the background
  colour step at the image's left edge: y=2190 of 2532, i.e. the nav bar
  is the bottom **13.51%** of the full true frame).
- `charging-nav.webp` — just that bottom 13.51% strip, nav bar only.

In CSS (`.charging-content` / `.charging-nav`), the content image is
absolutely positioned to fill the top 86.49% with `object-fit: cover`
(crops a little off the sides, never the nav bar, nothing with text in
that margin), and the nav strip is pinned `bottom: 0` at exactly 13.51%
height — its own native aspect ratio already matches that allocated box
almost exactly (3.42 vs 3.419), so it needs no cropping at all. Content
now fills edge-to-edge with no gap at either end, zero distortion, no
stray black bar under the nav bar.

Old combined `charging.webp` removed — nothing references it anymore.

## 2026-11-02 (pm): second punch list

1. **Neon · high contrast still wasn't applied to the hero dashboard.** It
   was genuinely applied in code (`gaugeStyle: GaugeStyle.contrast` was
   already set), but the difference from the default Instrument face is
   subtle — denser tick marks (5/division vs 4), brighter tick-label colour,
   a slightly different heat gradient (`palette.heatStopsContrast`, not
   `heatStops`) — easy to miss at a glance. Confirmed for real this time
   with a pixel-diff + side-by-side crop before calling it done.
2. **The "Veltor" slide in Dashboards had a nav bar; it shouldn't** — a real
   app behaviour, not a website-only crop: `app.dart`'s `hideNav` drops the
   bottom bar whenever the dashboard is in landscape (`isLandscape ||
   fullscreen`), so a landscape dashboard is NEVER shown with a nav bar in
   the real app. `theme_preview_capture_test.dart` always added one
   regardless of orientation — fixed to match the real rule
   (`bottomNavigationBar: land ? null : AppNavBar(...)`). Also folded in
   fix #1's gaugeStyle and the inside/outside temps for this render too.
3. **Sound tab now renders with Sound Mode ON** — seeded
   `engineSoundEnabled: true` in `website_sound_preview_capture_test.dart`'s
   `app_settings` (was defaulting off).
4. **Charging crop was the wrong tool**: the CSS `scale(1.16)` from a
   bottom transform-origin cropped the sides too ("Dashboard"/"Settings"
   clipped) and read as over-zoomed. Replaced with a real pixel crop at
   export time (`PIL.crop`, top 10% only, no scaling) — the image is
   simply shorter now, nothing is stretched or cut sideways, and the `.crop
   -top` CSS class is gone.
5. **The VIN wasn't the owner's** — `LRW3F7EK6NC507857` is a synthetic
   fixture value shared across ~20 other test files (confirmed via
   grep; one has an explicit "repo VIN" comment). Still swapped it for
   the one render the website uses (`app_screens_preview_capture_test.dart`
   → `pairing_paired` case) to an unambiguously different serial
   (`LRW3F7EK6PX483920`) rather than leave anything that could be mistaken
   for a real plate on a public page. The shared fixture used by the other
   20+ tests was left alone.

Also from the same session: the nav speedo and every dashboard's tacho
were bumped from 67→70 mph (`speedMs: 30` → `31.293` across the relevant
fixtures), matching the owner's ask mid-task.

Browser note: after re-exporting an asset under the SAME filename
(`paired.webp` etc.), the tab can still show the old bytes — plain
`location.reload()` isn't always enough against the browser's image
cache; `location.reload(true)` (or a hard refresh) is what actually
re-fetched it when verifying this round.

Re-verified at 1440px and 375px, console clean, no overflow.

## 2026-11-02: owner's 7-point punch list, all addressed

New Dart preview tests added to `veltor/test/` for two screens that didn't
exist as clean renders before:
- `website_final_hero_preview_capture_test.dart` → `build/website_final_hero.png`
  → `assets/app/final-hero.webp`. Colourful palette + `GaugeStyle.contrast`
  ("Neon · high contrast" in Settings → Tacho face), landscape, **no**
  `bottomNavigationBar` (the website crops the frame itself), with
  `outsideTempC`/`insideTempC` filled in so Inside/Outside don't read '—'.
- `website_sound_preview_capture_test.dart` → `website_sound.png` →
  `assets/app/sound.webp`. The real bug: the old `app_sound.png` (from
  `app_screens_preview_capture_test.dart`) rendered `SoundSettingsScreen`
  **standalone**, skipping the outer `Scaffold(bottomNavigationBar: AppNavBar)`
  that `app.dart`'s real route tree always wraps it in — so the website's
  sound screen was missing the nav bar the real app always shows there, and
  used a different device-padding setup than every other screen, hence the
  "glitch" and the inconsistent tacho position. Fixed by rendering it
  through the same `Scaffold` + `AppNavBar(selectedIndex: 1)` + `top:141,
  bottom:102` safe area as `theme_preview_capture_test.dart`.
- The Sound chapter's **background** dashboard is no longer a separate
  render (`dash-sound.webp`, from the zero-padding `sound_off_cluster_*`
  debug test) — it now reuses `dash.webp`, same image as the hero. One
  fewer inconsistent render, and the "glitch" is gone because it's
  literally the same frame.
- `theme_preview_capture_test.dart` and the new final-hero test: `_tele`
  speed bumped from `speedMs: 30` (67 mph) to `31.293` (70 mph), and the
  website's nav speedo chip now climbs to 70 too (was capped at 60) —
  `.is-60` renamed `.is-max` throughout.
- `stats_preview_capture_test.dart`: the Recent Drive card had SEVEN fields
  reading '—' (Start/End, Phantom, Phantom %, Real mi/kWh, Est. total,
  Regen total, Outside, Inside) because the fixture never set `o0`/`o1`
  (odometer), `gk`/`rk` (gross/regen kWh — NOT the same as `pkw`/`rkw`,
  which are PEAK kW), `to`/`ti` (temps) or a `pk` (ParkStart, for Phantom).
  All added. Also found and fixed: **`ParkStart.toJson` keys are `at`/`b`/
  `o`**, not `at`/`pct`/`odo` — a first attempt at the `pk` field used the
  wrong keys and silently parked-drain-is-null. Also fixed a **real,
  pre-existing flakiness**: the first drive was dated `now.subtract(Duration
  (minutes: 40))`; a render started in the first 40 minutes after midnight
  pushes that into *yesterday*, and "Recent drive" silently goes empty (hit
  this exact bug once this session, at 00:15am). Anchored it to a fixed
  `todayAnchor` (14:30 today) instead. Also seeded `perf_best_mph_60_*` +
  the `perf_bests_reset_2026_10` flag (without it, `acceleration_timer_
  provider.dart`'s one-time wipe clears any seeded best on first read), so
  "Best 0–60 mph" isn't '—' either. `stats.webp` is also now CROPPED to the
  real content height (was 6600px tall with ~1900px of dead black space at
  the bottom) — the mobile auto-scroll keyframe (`-74.4%` → `-64.4%`) was
  recomputed to match.
- Dashboard pack order is now the owner's: Veltor (landscape, reusing
  `dash-landscape`'s content as `packs/veltor_land.webp`), Luxury Glow,
  Track HUD, Needle EV, Cockpit Canvas, Chevron Blue, Flat Rings, Racing
  Red, Nordic Minimal, Pixel Rings.
- Tile lift-off (01 Live data): the dashboard image behind the floating
  tiles used to just dim 45% — the original tiles were still faintly
  visible under their own flown-off copies. Added `.tile-mask`, a solid
  black panel sized exactly to the tiles' bounding box (computed from
  `TILES` in JS, not hand-measured), that fades in as they lift and out as
  they return. The speedo above it stays fully lit throughout.
- Hero: "VELTOR" wordmark overlay added in the safe-area gap every portrait
  render reserves at the top (was dead space) — same overlay reused on the
  Sound scene since it's the same `dash.webp` now. Float chip text
  "speed · power · battery" → "Speed · HP · Regen" per the owner.
- Charging chapter: added a line about every charge teaching Veltor the
  real usable battery capacity. The image itself had ~14% dead black space
  at the top (SafeArea + the widget's own top padding) and felt cramped at
  the bottom — added `.crop-top` (`scale(1.16)` from a bottom transform-
  origin, so it crops the top gap and the sides a little, never the bottom
  nav bar).

Stale assets removed: `dash-sound.webp`, `dash-landscape.webp` (superseded
by `final-hero.webp` + `packs/veltor_land.webp`).

Re-verified at 1440px and 375px after every change, console clean, no
horizontal overflow.

## 2026-11-01 (pm, 2): calm theme + landscape dashboards

Owner feedback: didn't like "colourful" (the app's own rainbow heat-arc
theme toggle — not a generic complaint about color) and didn't like the
"Pick a look" chapter. Two changes:

1. **Hero + 01 Live data now use the Calm theme**, not Colourful. Swapped
   `assets/app/dash.webp` and `dash-landscape.webp` to be exported from
   `theme_dashboard_calm.png` / `_calm_landscape.png` instead of the
   `_colourful` renders (both exist in `veltor/build/` already, same
   harness run — `AppTheme`'s Calm/Colourful toggle, see
   [[project_veltor_landing_redesign]]). The tile lift-off crops pull from
   the same file, so they picked it up automatically.
2. **02 Dashboards is now a landscape phone**, not portrait — closer to how
   these cluster packs are actually mounted behind the wheel. The portrait
   `#phone` crossfades out (`landOn` ramp in `render()`) and a new
   `#landWrap`/`.phone--land` crossfades in, showing `packs/<id>_land.webp`
   (exported from the harness's existing `*_l390.png` gallery renders —
   `test/pack_gallery_preview_capture_test.dart` already renders landscape
   for every pack, nothing new to render). Applied the same landscape
   phone to the mobile swipe row. The 9 packs' own colors (Racing Red,
   Luxury Glow, etc.) are untouched — those are per-pack designs, not the
   Calm/Colourful toggle, and "colourful" there is the point.

Re-verified at 1440 and 375px, console clean, no overflow.

## 2026-11-01 (pm): fixed a real image-distortion bug + proportion pass

The owner was right — it looked bad. Root cause of the "out of proportion"
hero image: two `<img>` tags carried stray `width="390" height="844"` HTML
attributes left over from copy-paste, and the stylesheet only reset `width`
(`.scene>img{width:100%}`), not `height`. Browsers apply an unset HTML
`height` attribute as a literal pixel box when no CSS height is set, so the
image was forced into a 844px-tall box at a 100%-wide box — two different
scale factors, i.e. visibly squashed. Fixed by adding `height: auto` to the
base `img` rule (belt-and-braces, covers every image on the page) and
deleting the two stray attributes.

Also fixed while in there, because "looks like crap" was bigger than one bug:
- The phone was capped at 330px wide inside a ~650px column — tiny, floating
  in a lot of dead black space. Bumped `--pw` to a 420px cap / 0.64×vh (was
  280/0.45), added a second tighter "rim" glow behind the phone, and a top
  highlight on the phone bezel.
- The h1 auto-wrapped into an ugly ragged 4-line stack ("The" alone on line
  1). Gave it a manual break + `&nbsp;` between "never" and "had." so it's
  a clean 2-line shape instead of leaving `text-wrap:balance` to find its
  own break points in a column that was too narrow for an 84px font.
- Removed the fake "dynamic island" notch overlay (`.screen::after`) — it
  was positioned at a fixed 1.3%–4.9% from the top assuming a reserved
  status-bar area, but the real renders have no reserved top safe-area
  (confirmed against `assets/app/paired.webp`: "Pair with your Tesla" sits
  right at the top), so the island was clipping that header to "Pair with
  your [Tesla cut off]". A fake notch over an edge-to-edge screenshot will
  always risk this for any future screen too — not worth keeping.
- Widened the story grid (1320→1400px max, explicit column-gap) and made
  the background glow richer (3 radial layers instead of 2 faint ones).

Verified at 1920, 1440, 1024 and 375px, console clean, no horizontal scroll
at any width.

## START HERE (2026-11-01): scroll-story page at `/story/`

New page `story/index.html`, inspired by sparkle.ankatms.com. One phone stays
pinned on the right while 7 chapters scroll past, and its screen moves the way
the app does:
- **01 Live data**: the six tiles are cropped out of the dashboard render and
  fly off the screen (`TILES` in the script holds their 390×844 pt boxes).
- **02 Dashboards**: swipes through 9 gallery packs and pauses on each one.
- **03 Sound**: the Sound tab slides up over the dashboard like a sheet.
- **04 Charging** is a push transition. **05 Stats**: the long stats page scrolls
  inside the phone.
- **06 Set up**: onboarding → paired, with the 3 steps lighting up in turn.

There is also a nav "speedo" chip that climbs from 0 to 60 mph as you scroll,
plus a chapter rail at the bottom. Under 900 px each chapter gets its own phone
and the packs become a swipeable row.

**Screens are REAL app renders, not CSS mock-ups.** They come from the app's
preview harness at `fb8086b`:

```
~/flutter-3.47.6/bin/flutter test --tags preview --run-skipped test/theme_preview_capture_test.dart test/sound_off_cluster_preview_capture_test.dart test/stats_preview_capture_test.dart test/drive_week_preview_capture_test.dart test/charging_preview_capture_test.dart test/onboarding_preview_capture_test.dart test/app_screens_preview_capture_test.dart test/pack_gallery_preview_capture_test.dart
```

The output is converted to webp in `assets/app/` (under 1 MB in total).
Homebrew `flutter` (3.35) can't resolve the app's pubspec, so use
`~/flutter-3.47.6`. If the dashboard render changes, re-measure `TILES`.

- Reduce Motion keeps the scroll-linked moves, because they only move while
  the reader scrolls. Only the looping animations stop. The built-in browser
  pane reports Reduce Motion ON.
- The Free/Pro copy follows `ProFeature` in the app's `entitlements.dart`:
  real mi/kWh, cost, phantom drain and battery health are Pro. No prices,
  no ads claims, no marque names.
- **Not wired in:** `index.html` (the uncommitted rewrite) is untouched, and
  nothing is committed. Next: owner's look, then decide whether `/story/`
  becomes the homepage.

---

## Previous (2026-09-05): redesign canvas

The 2026-09 redesign is **not** in this repo yet. It lives entirely in a
published Claude Design canvas (a separate, editable multi-artboard
mockup), waiting on your review before anything gets ported into
`index.html`.

**Canvas:** https://claude.ai/code/artifact/8926be12-25e8-4366-a9e3-d524b7b376dd

Repo state right now: `main` branch, `index.html` has local uncommitted
edits from *before* this session (the earlier CSS/SVG-recreation pass —
see "Prior state" below), nothing from the canvas has been applied.
`redesign-2026-09` branch is stale (still at the old `/sound` commit) —
ignore it or delete it, the live work is the canvas link above.

## Why a canvas instead of editing index.html directly

You said the first pass ("just edited a few bits") wasn't enough and
asked for a full modern redesign referencing 5 ChatGPT-generated
mockups + opalapp.com + an eBike template site. A from-scratch visual
redesign is much faster to iterate on as a click-to-edit canvas than as
raw HTML round-trips — so that's where all of this session's actual
design work happened. Nothing here is final until you say so.

## What's on the canvas (9 artboards, top to bottom)

1. **Main** — nav + hero. Headline "An app that adds soul to your
   Tesla", three floating tilted device mockups (live cluster, HUD,
   stats), the brand's real heat-arc gradient (blue→white→amber→red,
   lifted from `assets/logo.png`) behind everything and through the
   headline/CTA gradients.
2. **Features** — bento-grid redesign of the old arrow-flow "what is
   Veltor" row.
3. **DashboardPacks** — the 4 real cluster packs (Veltor/Model
   S-X/Mercedes/Audi) as floating cards. Kept blue-only here
   deliberately — that's the actual in-app palette; adding red would
   misrepresent the product.
4. **SoundHud** — engine sound (rev dial, waveform, pack chips) split
   with the HUD device. Red-leaning (performance/energy).
5. **Launch** — the 0-60 timer, the one dedicated "redline" moment,
   big amber-to-red glowing number.
6. **Stats** — driving-stats grid.
7. **Support** — connection explainer (BLE, read-only key) + Tesla vs.
   universal-GPS vehicle-support cards.
8. **Pricing** — Free vs. Veltor Pro (lifetime-only, matches
   `project_veltor_entitlements`).
9. **FinalCTA** — closing CTA, App Store / Google Play badges marked
   **SOON** (not clickable — matches real launch status).

## Deliberate decisions (don't relitigate these without reason)

- **Blue + red, not blue-only.** Earlier in this session I defaulted to
  blue-only assuming it was locked from the logo. You corrected that —
  the actual logo (`assets/logo.png`) is a blue-to-red speedometer arc.
  Red is now used for: headline/CTA gradients, the engine-sound/0-60
  sections, Pro-tier pricing accents. Blue stays primary for
  telemetry/UI chrome and the dashboard-pack recreations.
- **No fabricated social proof.** The app hasn't launched — no star
  ratings, review counts, or "trusted by" claims anywhere, unlike the
  ChatGPT reference mockups which invented all of that. Proof line
  instead states real, verifiable facts (launching iOS/Android, real
  model compatibility).
- **Store badges = SOON, not linked.** Matches the real current launch
  status; don't make them clickable until there's an actual store
  listing.
- **HUD projection stays `#00ff88`**, not the app's status-green
  `--green: #35e08a`. That's the literal color from the real
  `index.html` HUD scene (`.hud__proj`), correct as-is even though it
  reads as a second, un-tokenized green — a background review flagged
  this and it was confirmed intentional, not a bug.
- **Dashboard-pack screens are true recreations** of the real gauge/
  tile markup from `index.html` (`cluster`, `cluster__gauge`, `tile`,
  etc.), not generic phone-store dashboards — same as the placeholder
  approach already documented in `SCREENSHOTS.md`.

## A caching gotcha you'll hit again

Republishing the canvas updates the *same* URL, but the browser tab
caches the old page. If it "looks the same" after an update: hard
reload (Cmd+Shift+R) or close/reopen the tab before concluding the
publish didn't take. Confirmed once already this session (grepped the
published HTML server-side to prove new content was live while the
open tab still showed stale content).

## Editing the canvas further

It's a live Claude Design canvas — click into any artboard to
edit text/layout directly, or describe the change here and it'll be
made the same way (edit the `.dc.html` working files under
`/private/tmp/.../scratchpad/veltor-design/` from this session, or
`--extract` fresh from the published artifact if edited elsewhere
first, then re-seed and republish to the same URL). Working files from
this session may not survive to a new session/machine — if picking
this up fresh and the local scratch files are gone, `--extract` from
the artifact URL is the way back in.

## Next action

Get your sign-off (or another round of changes) on the canvas, then
port the approved design into `index.html` for real — that hasn't
happened yet. Also still outstanding from before this session (see
`SCREENSHOTS.md`): real device screenshots eventually replacing every
CSS/SVG recreation, and the legal pages (`privacy/`, `support/`,
`links/`) still using the old Orbitron font instead of Space Grotesk.

## Prior state (before this session, still true)

- `index.html` working-tree diff (uncommitted, pre-dates this session):
  a ~2,660-line rewrite already in place — full page content-complete,
  CSS/SVG recreations throughout, reveal-on-scroll verified working via
  DOM inspection (a `.dc.html` browser-automation screenshot glitch
  earlier made it *look* broken; it wasn't — confirmed by reading
  computed styles directly).
- `redesign-2026-09` git branch: stale, do not use.
