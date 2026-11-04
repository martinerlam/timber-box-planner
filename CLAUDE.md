# Timber Box Planner

Single self-contained `index.html` (HTML, CSS, JS). No build step, no package manager.

- [HANDOFF.md](HANDOFF.md) is the spec and the source of truth for geometry and cut list formulas. Keep it updated when behaviour changes.
- Mitre joints only. Do not reintroduce a joint selector.
- Units are mm throughout.
- Libraries: three.js r128 and its OrbitControls from CDN. Keep versions pinned.
- Settings persist in `localStorage` under `boxplanner2`. Changing the shape of saved state needs a migration or a new key.
- Hosted on GitHub Pages, so `index.html` must stay at the repo root.
