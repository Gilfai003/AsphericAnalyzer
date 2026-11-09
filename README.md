# Aspheric Surface Manufacturability Analyzer

Design screen for optical aspheric surfaces. Given a prescription, the page computes sag and local geometry, then compares that geometry with sourced notes for four fabrication routes. The result is a screening checklist for a shop conversation. It is not a manufacturing certification, a tool-path generator, or a tolerance analysis.

The interactive page was built as a Grok Bot Page:

<https://cursor.com/canvas/store-b4b56f75-ded9-41db-84fc-e82381986140/8c25ba20-0ad7-452d-a043-5536a295a04a>

## What the analyzer does

`src/AsphericAnalyzer.tsx` evaluates one surface at a time and reports:

- sag, departure from a best-fit reference sphere, local slope, curvature, and (outside the rotational mode) departure from rotational symmetry
- a per-process checklist covering material family, aperture, and the geometry checks that have a sourced guide
- how many checks were skipped because no published limit was found

**Skipped checks are not passes.** A row marked `SKIPPED` means that criterion was left unevaluated. It does not mean the surface met the criterion. A process status of `PASS` means only that every check the page actually applied came back clear. `WARN` and `FAIL` mark applied checks that need review or that fall outside a sourced screen. A shop still has to qualify the full prescription, tolerances, material, tool path, and metrology.

## Three surface modes

1. **Rotationally symmetric asphere.** Even asphere on a conic base:

   `z(r) = c r² / [1 + √(1 − (1+k) c² r²)] + Σ A₂ᵢ r²ᵢ`, with `i = 2…8` and `c = 1/R`.

   Radius, sag, and coordinates are in millimeters. Analysis is radial (401 samples) and keeps both a vertex-edge best-fit sphere and a least-squares best-fit sphere.

2. **Off-axis segment.** A circular or rectangular piece of a parent rotational asphere, decentered along the parent y axis. Local sag is the parent sag at `√(x² + (y + d)²)`, with axes left parallel to the parent frame. The aperture is the projected local XY footprint. Piston and tilt are removed only for departure and asymmetry metrics; surface slope keeps the parent-frame inclination.

3. **Freeform.** A base conic (`R`, `k`, no even-asphere terms) plus one expansion. Coefficients are surface sag in micrometers, not optical wavefront:

   - **XY polynomials** through total order 6, including piston and tilt: `Σ aₚq (x/N)ᵖ (y/N)ᑫ` with `p + q ≤ 6`.
   - **Standard OSA/ANSI Zernike** terms `j = 0…20` (radial orders 0 through 5). Normalization is unit-RMS on the full normalization disk. Negative `m` is sine and non-negative `m` is cosine. The normalization radius must cover the aperture; coordinates are not clamped.

## Four processes

| Process | What the screen is for |
| --- | --- |
| CNC grind + polish | Conventional generation and polish of glass aspheres, using inherited rotational capability notes. |
| MRF (magnetorheological finishing) | Sub-aperture finishing, including platform-specific raster and head notes where those were sourced. |
| Single-point diamond turning | Diamond turning of compatible materials, including servo-path and parent-blank reviews for non-rotational surfaces. |
| Precision glass molding | Molding of moldable glass (and, in the material screen, PMMA). Polymer molding is a different process from precision glass molding and still needs shop-specific qualification. |

The material families in the inherited screen are N-BK7, fused silica, L-BAL42, and PMMA. Compatibility is a family-level flag. Supplier chemistry and coatings still need review.

## Published figures are not universal process limits

Numbers in the checklist are **machine-specific demonstrations or specifications**, or application heuristics called out as such in the source. They are not universal process limits. Examples include a QED Q-flex 300 raster envelope, QED MRF head concave-radius and field-angle figures, Precitech FTS and STS travel examples, Moore Nanotech NFTS travel specifications, an asphericon capability chart, Optimax asphericity guidance, a Kumler 2005 MRF demonstration, and a Rochester Precision Optics diameter range.

Where the page could not source a general ceiling — freeform departure, freeform slope, coupled head clearance, servo dynamics, molding stroke and release, metrology accuracy — that check is **skipped**. Rotational departure limits are not copied onto freeform surfaces. Warning bands such as half of a published maximum, or 1.5× a published minimum radius, are engineering heuristics where the page labels them that way.

Citations, access notes, and the exact use of each figure live in the Sources section of `src/AsphericAnalyzer.tsx`. This repository does not add process limits or citations beyond that file.

## Running the UI

The module imports its UI primitives from `grok/canvas`:

```ts
import { Page, Section, Text, Callout, Grid, Table, /* ... */ } from "grok/canvas";
```

**This repository is not a standalone npm app.** There is no `package.json`, bundler, or public `grok/canvas` package here. The default export `App` runs inside the Grok Bot Page host that provides that import. Opening the `.tsx` file in a normal React or Next.js project will not resolve `grok/canvas`. The UI is intentionally left in that form.

## Repository layout

- `src/AsphericAnalyzer.tsx` — analyzer source
- `LICENSE` — MIT License
- `README.md` — this file

## License

MIT. See [LICENSE](LICENSE).
