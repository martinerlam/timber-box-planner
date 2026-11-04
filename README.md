# Timber Box Planner

A single-page browser tool for designing a mitred timber box with a lid. Set the sizes and see a live 3D model, drawing-style dimensions and a cut list in millimetres.

## Features

- Length, width, overall height (a) and lid height (b), as outside or inside sizes
- Selectable timber, bottom panel and top panel thickness
- Bottom in a groove or glued on underneath
- Four lid types: sawn off the box, lift-off with lip, hinged, sliding in grooves
- 3D view with orbit, lid opening, exploded view and section cut
- Outside dimensions on the model, inside dimensions in section view
- Cut list with highlight-in-3D, totals, trimming allowance and copy to spreadsheet
- Warnings for impossible or risky settings
- Light and dark themes

## Running it

Open `index.html` in a browser. There is no build step. three.js r128 and the fonts load from a CDN, so it needs an internet connection the first time.

To serve it locally:

```bash
python3 -m http.server 8000
```

Then open http://localhost:8000.

## Hosting

Published with GitHub Pages from the `main` branch root.

## Design notes

See [HANDOFF.md](HANDOFF.md) for the spec, geometry rules, cut list formulas and code layout.
