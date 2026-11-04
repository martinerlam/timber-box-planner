# Timber Box Planner: handoff for Claude Code

## What it is
A single-page browser tool for designing a mitred timber box with a lid. You set the sizes and see a live 3D model, drawing-style dimensions and a cut list in mm. It was prototyped in Claude chat from a paper sketch. **From here on, this repo is the only home for the code. All changes happen here.**

- File: `index.html`, self-contained (HTML, CSS and JS in one file).
- Libraries, loaded from a CDN:
  - three.js r128 (`cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js`)
  - OrbitControls (`cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js`)
- Fonts: Big Shoulders Display, IBM Plex Sans and IBM Plex Mono, from Google Fonts.
- Hosting: GitHub Pages.

## Spec from the sketch (the source of truth)
- Main dimensions:
  - **length (c)**
  - **width**
  - **overall height including lid (a)**
  - **lid height (b)**
  - **container height = a − b**, derived and not entered
- **Mitre joints only.** There is no joint selector.
- Timber thickness is selectable, and the box adjusts parametrically.
- Keep the bottom options, the lid type options and the internal dimensions.
- Produce a cut list and dimensions in mm.

## Features (current state)
- **Size inputs:**
  - L, W, a and b, each a slider plus a number box.
  - An outside/inside toggle chooses whether L, W and H are outside or inside sizes. Switching converts the numbers so the box stays the same.
- **Timber:**
  - wall thickness T, with preset buttons for 6, 9, 12, 15, 18, 20, 22 and 25 mm
  - bottom panel thickness Tb
  - top panel thickness Tl, shown only for the sawn-off lid
- **Bottom:** `groove` (panel in a groove) or `glued` (glued on underneath).
- **Lid types:**
  - `cutoff`, the default: build a closed box, then saw the lid off. b is the band height. The top panel sits in a groove. A saw kerf allowance is added to the wall stock height.
  - `lift`: flat panel b thick, with locating lip strips underneath. Has settings for lip depth and clearance each side.
  - `hinged`: flat panel b thick, hinged on the back long edge.
  - `slide`: panel b thick running in grooves in both sides and the back. The front end is cut down so the lid can slide out. Has a clearance setting.
- **3D viewer:**
  - orbit controls
  - a lid slider that lifts, hinges or slides the lid depending on type
  - an exploded view
  - a section cut at the mid-plane (z = 0), using a clipping plane
  - a reset view button
- **Dimensions toggle**, on by default:
  - outside dimensions: length (c), width, overall (a), container, lid (b) and timber thickness
  - each drawn as a 3D line with extension lines and ticks, with HTML labels projected on top
  - lid figures hide while the lid is open
  - all hidden in exploded view
- **Inside dimensions** show only in section view:
  - inside length, width and height
  - for the lift-off lid: clear depth under the lip and the lip gap
  - for the sliding lid: the groove play on each side
- **Cut list table:**
  - columns: part, quantity, length (along the grain), width, thickness, notes
  - hovering or clicking a row highlights that part in 3D
  - summary of piece count, face area (m²) and timber volume (litres)
  - "Copy cut list" copies tab-separated text that pastes into a spreadsheet as columns
  - an optional trimming allowance is added to the length and width of every part
- **Warnings** for impossible or risky settings, for example walls too thick for the box, a lid nearly as tall as the box, a tight lip, or a groove too wide for the wall.
- Light and dark themes come from CSS tokens. Settings persist in `localStorage` under the key `boxplanner2`.

## Geometry rules
Units are mm. H means overall height (a), b is the lid height and T is the wall thickness.

**Constants:**
- `GOFF = 6`: grooves sit 6 mm from the edge.
- `SLIP = 3`: the sliding-lid groove sits 3 mm below the top.
- `LIPT = 8`: maximum lip strip thickness.
- Groove depth `gd = min(T/2, 6)`, rounded to 0.5.

**Lid panel thickness:** `eff()` maps the lid panel thickness to b for the lift, hinged and slide lids. The cutoff lid uses Tl for its top panel.

**Base and top allowances, used for inside height:**
- base = Tb + 6 (groove), or Tb (glued)
- top = b (lift or hinged), 3 + b + clr (slide), or 6 + Tl (cutoff)

**Inside sizes:** Lin = L − 2T, Win = W − 2T, Hin = H − base − top.

**Container height:** H − b. For the slide lid it is H, because that lid sits inside the walls.

**Wall stock height:** top of the wall stock minus the wall base.
- Wall base = Tb if the bottom is glued on, otherwise 0.
- Top of the wall stock = H + kerf (cutoff), H − b (lift or hinged), or H (slide).

## Cut list formulas
All lengths for mitred parts are to the long point.

| Part | Qty | Length | Width | Thick |
|---|---|---|---|---|
| Sides | 2 | L | wall stock height | T |
| Ends | 2 | W | wall stock height | T |
| Bottom, in groove | 1 | L − 2T + 2gd − 2 | W − 2T + 2gd − 2 | Tb |
| Bottom, glued | 1 | L | W | Tb |
| Top panel (cutoff) | 1 | L − 2T + 2gd − 2 | W − 2T + 2gd − 2 | Tl |
| Lid panel (lift or hinged) | 1 | L | W | b |
| Lip strips, long (lift) | 2 | 2·(L/2 − T − clr) | lip depth | lt |
| Lip strips, short (lift) | 2 | 2·(W/2 − T − clr) − 2·lt | lip depth | lt |
| Sliding lid | 1 | L − T + gd − clr | W − 2T + 2gd − clr | b |
| Front end (slide) | 1 | W | (H − 3 − b − clr) − wall base | T |

- **Lip strip thickness:** lt = min(8, max(3, (W/2 − T − clr)/2)).
- **Bottom and top panels in grooves** get a 1 mm gap each side for wood movement.
- **Cutoff:** the note on the sides and ends says where to saw the lid off (b from the top).
- **Slide:** the back end is a normal end, and the front end is cut down to the underside of the lid groove.

## Code layout (inside the main script)
- `DEF`, `S`, `eff(S)`: defaults, state, and the effective state passed to geometry.
- `NUM`, `SEL`: definitions for the number and select controls. The form is built from these.
- `derive(S)`: outside sizes, inside sizes and the base and top allowances.
- `build(S)`: returns `{d, parts, rows, warn, contH, gd, ...}`.
  - `parts` are lists of extruded polygons per part, used for the 3D view.
  - `rows` is the cut list.
- `rebuild()`, `pose()`, `loop()`: three.js scene setup, lid and exploded animation, and the render loop.
- `buildDims()`, `updateDims()`: dimension lines and labels, including the inside dimensions in section view.
- `renderUI()`: readout chips, cut list table, summary and warnings.

## Known limitations and ideas not yet built
- No STEP or 3MF export. The user models in Shapr3D, so STEP with each part as its own body would be the useful export.
- Hinged lid: no hinge hardware or knuckle clearance is modelled. The page only shows a warning.
- The sliding lid is modelled as a full-thickness panel. Real ones usually have rebated edges to fit a narrower groove.
- Wood movement is handled only by the fixed 1 mm panel gaps.
- The libraries load from a CDN, so the page needs an internet connection the first time it opens.
