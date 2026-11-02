# Veltor site — real screenshot capture checklist

The landing page (`index.html`) currently renders **CSS/SVG recreations** of the
Veltor UI, built to the real palette (`ClusterPalette.veltor`), the real Space
Grotesk type, and the real feature set. They are placeholders. This page is the
list of real captures to take, and where each one drops in.

## How to capture

- **Device:** iPhone 15 Pro (or 15 Pro Max) simulator, and one modern Android
  (Pixel 8). Capture at native resolution, no device frame — the page draws its
  own frame.
- **Build:** a release build, demo drive running so the numbers are populated
  and consistent (`Settings → Demo drive`). Set the demo slider so speed ≈ **72
  mph**, battery ≈ **68–85 %**, gear **D** — that matches the copy on the page.
- **Chrome:** hide the OS status bar if you can, or crop it. No personal VIN or
  location visible anywhere.
- **Format:** PNG, sRGB. Keep the aspect ratio in the table — the page slots are
  built to it. Name them exactly as the "file" column and drop into
  `assets/screens/`.

## Landscape — dashboard packs (aspect ≈ 19.5:9, phone rotated)

| # | file | screen to capture | replaces on page |
|---|------|-------------------|------------------|
| 1 | `dash-veltor.png` | Dashboard, **Veltor** pack, RPM ring, full deck visible | hero device + Dashboards → "Veltor" panel |
| 2 | `dash-modelsx.png` | Dashboard, **Inspired by Tesla Model S/X** pack | Dashboards → "Model S/X" panel |
| 3 | `dash-mercedes.png` | Dashboard, **Inspired by Mercedes** pack (twin dials) | Dashboards → "Mercedes" panel |
| 4 | `dash-audi.png` | Dashboard, **Inspired by Audi** pack (energy rail + speed) | Dashboards → "Audi" panel |
| 5 | `dash-veltor-power.png` *(optional)* | Veltor pack with the ring swapped to **Power** | future ring-swap callout |

## Landscape — other

| # | file | screen | replaces |
|---|------|--------|----------|
| 6 | `hud-night.png` | **HUD mode**, green theme, driving (speed + gear + efficiency), ideally shot reflected in a real windscreen at night; a clean in-app HUD capture is fine as fallback | HUD section scene |
| 7 | `launch-run.png` | **Launch** screen mid/just-after a run — big 0–60 time, split ladder (60 ft, 0–60, ¼ mile) | 0–60 section (currently pure type) |
| 8 | `launch-warp.png` *(optional)* | Launch screen with the Plaid-style warp animation at speed | 0–60 background |

## Portrait (aspect ≈ 9:19.5)

| # | file | screen | replaces |
|---|------|--------|----------|
| 9 | `stats-overview.png` | **Driving stats**, Week period — miles bar chart, real efficiency, phantom drain, best 0–60 tiles | Stats section grid |
| 10 | `stats-drive.png` *(optional)* | A single **drive detail / drive history** row expanded | Stats section secondary |
| 11 | `sound-picker.png` | **Sound** pack picker showing the 18 packs + the low-speed response setting | Engine-sound section (right column) |
| 12 | `hud-settings.png` *(optional)* | HUD settings — colour themes, angle, Glass/Glow, mirror | HUD section chips |
| 13 | `paywall.png` *(optional)* | **Veltor Pro** paywall (the real one — one lifetime tier + free ladder) | Pricing section |
| 14 | `connect.png` *(optional)* | The Tesla **connect / add-key** screen (no real VIN) | Connected section |
| 15 | `final-hero.png` | Any single clean dashboard portrait shot for the closing CTA device | Final CTA device |

## Wiring them in

Each recreation on the page is inside a `.device__screen`. To swap:

1. Drop the PNG in `assets/screens/`.
2. Replace the `<div class="cluster …">…</div>` (or `.hud__scene`, the stats
   grid, etc.) with `<img src="assets/screens/NAME.png" alt="…" class="shot">`.
3. Add once to the stylesheet: `.shot{width:100%;height:100%;object-fit:cover;display:block}`.

The device frame, glow, reflection, section layout, copy and animations all
stay. Nothing else needs to change.

## Also still to do (not screenshots)

- **Store links:** `index.html` shows App Store / Google Play badges marked
  *Soon* and non-clickable. When live, replace the two `<span class="store">`
  blocks with `<a class="store" href="…">` and drop the `.soon` chip.
- **Notify form:** currently composes a `mailto:` to `veltorapp@gmail.com`. To
  collect addresses properly, point the form at a real endpoint (Formspree,
  Buttondown, etc.) — see the comment in the `#notifyForm` handler.
- **Legal pages:** `privacy/`, `support/`, `links/` still use the old Orbitron
  font. Optional: swap to Space Grotesk to match.
