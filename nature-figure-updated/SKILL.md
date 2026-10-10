---
name: nature-figure
description: >-
  Submission-grade Nature/high-impact journal figure workflow for Python or R. Use whenever the user asks to create, revise, audit, or polish manuscript figures, multi-panel scientific plots, or journal-ready PNG (default) / SVG / TIFF outputs, especially for Nature-family or other high-impact journals. Before plotting, define the figure's conclusion, evidence logic, export needs, and review risks. If the user has not chosen Python or R, ask "Python or R?" and stop. Use only the selected backend for figure generation, previewing, exporting, and QA. Supports matplotlib/seaborn and ggplot2/patchwork/ComplexHeatmap. Not for dashboards or Illustrator/Figma-first infographics.
---

# Nature Figure Making Skill

A guide for producing publication-quality scientific figures as a visual argument, not
as isolated pretty plots. Every figure starts from a claim, an evidence hierarchy, and a
review-risk check before code or aesthetics.

The older Python/matplotlib rules in this skill remain valid. The skill now also supports
R, especially `ggplot2 + patchwork + ComplexHeatmap + ggrepel + ragg/svglite`.
If the user provides a private plotting template collection, use it only as an internal
adaptation source and do not reveal its path, filenames, or provenance in user-facing output.

Color policy: prefer **unified method families across all panels** over maximal hue separation.
For dense Nature Machine Intelligence-style figure pages, use the low-saturation `NMI pastel`
family described in `references/api.md` and reserve green/red mainly for gains, drops, and other directional cues.

## First move: figure contract before plotting

Before generating or editing code, establish the contract below.

**Backend selection is a blocking gate.** If the user has not explicitly chosen Python
or R in the current request or provided a clearly language-specific input file/workflow,
ask one concise question: **Python or R?** Then stop and wait for the user's answer.
Do not generate mock data, write scripts, create figures, or choose Python/R by default.
This overrides general autonomy/default-execution behavior for figure tasks.

**The selected backend is exclusive for all figure generation.** Once Python or R is
selected, every plotting script, preview image, PNG/SVG/TIFF export, QA render,
and visual workaround must be produced by that same backend. Do not use Python to
draw a preview for an R figure, and do not use R to draw a preview for a Python figure,
even if the selected runtime or packages are missing locally. The non-selected language
may only be used for non-visual file inspection or data conversion when it does not
open a graphics device, import plotting libraries, create image/vector files, or
change the final visual appearance.

**Missing runtime/package rule.** After the backend is selected, check the selected
runtime early (`Rscript`/R for R; Python and required plotting packages for Python).
If the selected runtime or required packages are unavailable, stop before rendering
and report the exact blocker. You may provide a selected-backend script and installation
commands, or ask permission to install dependencies, but you must not fall back to the
other language to make a substitute figure.

Only recommend a backend when the user explicitly asks you to choose or recommend one.
In that case, use `references/backend-selection.md`, state the reason, and then proceed
with the recommended backend.

1. Core conclusion: write the one-sentence claim the figure must defend.
2. Evidence chain: map each planned panel to the claim, and drop panels that do not carry
   a unique piece of evidence.
3. Archetype: classify the figure as `quantitative grid`, `schematic-led composite`,
   `image plate + quant`, or `asymmetric mixed-modality figure`.
4. Backend: use the selected Python or R track exclusively for all figure drawing,
   previewing, exporting, and visual QA. Do not cross-render with the other language.
5. Journal/export contract: set final dimensions, editable text, source data, statistics,
   image-integrity notes, and export formats before styling.

The highest-priority rule is: **the chart serves the scientific logic**. Aesthetic polish,
template matching, and complex layout are subordinate to making the core conclusion clear,
defensible, and reviewable.

## Final-size typography contract (mandatory)

Set the canvas to the intended publication dimensions **before** choosing typography,
line widths, marker sizes, spacing, or panel proportions. Create and export the figure
at approximately its final size; do not rely on substantial post-export scaling.

- Single-column width: approximately 89-90 mm (about 3.5 inches).
- Double-column width: approximately 180-183 mm (about 7.1-7.2 inches).
- Typeface: Arial throughout, with Helvetica as the preferred substitute when Arial is unavailable.
- Axis labels: 7 pt.
- Tick labels: 6-7 pt.
- Legend text: 6-7 pt.
- In-panel annotations: 6-7 pt.
- Panel labels: lowercase `a`, `b`, `c`, approximately 8 pt, bold.
- Avoid a title inside the plotting area unless it carries information that cannot be placed in the caption or panel descriptor.

Treat these as sizes at the final exported dimensions. If a 180 mm figure is reduced to
90 mm, a 10 pt label will appear approximately 5 pt; therefore resize the canvas and
re-render rather than scaling the finished artwork. Keep line widths, marker sizes,
gutters, and panel proportions consistent across manuscript figures at final size.
When `bbox_inches="tight"` changes the physical output dimensions, verify the exported
width and adjust the canvas or margins so the delivered file still matches the intended
publication width.

This contract overrides larger, legacy `figsize` and font-size numbers in supporting
examples. Treat those examples as layout patterns only and translate them to the final-size
contract before use.

## User-facing privacy rule

Do not disclose private local paths, private filenames, chat-attachment names, internal
reference filenames, template identifiers, or the provenance of private working materials
in user-facing replies, generated code comments, figure legends, reports, or manuscript
text. Use generic descriptions such as "the provided R template collection", "a private
working draft", or "the internal figure contract". Only reveal an exact path or source
file when the user explicitly asks for that audit trail.

## Python quick-start

**Python-only execution rule.** When the user has selected Python, do all figure
drawing, previewing, exporting, and visual QA in Python. Do not call R/ggplot2,
ComplexHeatmap, patchwork, or any R graphics device to create a temporary preview,
fallback export, or layout approximation. If Python or required Python plotting
packages are missing, stop before rendering and report the missing dependency. You
may still write the Python script, provide `pip`/environment install commands, or
ask permission to install dependencies, but do not cross-render the figure in R.

```python
import matplotlib as mpl
import matplotlib.pyplot as plt

mpl.rcParams.update({
    "font.family": "sans-serif",
    "font.sans-serif": ["Arial", "Helvetica", "sans-serif"],
    "svg.fonttype": "none",     # editable text in SVG
    "font.size": 7,
    "axes.labelsize": 7,
    "xtick.labelsize": 6,
    "ytick.labelsize": 6,
    "legend.fontsize": 6,
    "axes.spines.right": False,
    "axes.spines.top": False,
    "axes.linewidth": 0.6,
    "legend.frameon": False,
})

width_mm = 180.0   # use 89-90 for a single-column figure
height_mm = 120.0
fig, ax = plt.subplots(figsize=(width_mm / 25.4, height_mm / 25.4))

def add_panel_label(ax, label):
    ax.text(0.0, 1.02, label, transform=ax.transAxes,
            fontsize=8, fontweight="bold", ha="left", va="bottom")

def save_pub_py(fig, filename, dpi=600):
    # Prefer controlled margins over bbox_inches="tight" so physical size stays fixed.
    fig.savefig(f"{filename}.png", dpi=dpi)  # default: PNG
    fig.savefig(f"{filename}.svg")           # vector/editable when needed
    fig.savefig(f"{filename}.tiff", dpi=dpi)
```

Use `text.usetex = True` only when LaTeX is installed and math-rich labels are required.

## R quick-start

```r
library(ggplot2)
library(patchwork)

theme_set(
  theme_classic(base_size = 7, base_family = "Arial") +
    theme(
      axis.line = element_line(linewidth = 0.35, colour = "black"),
      axis.ticks = element_line(linewidth = 0.35, colour = "black"),
      axis.title = element_text(size = 7),
      axis.text = element_text(size = 6),
      legend.title = element_text(size = 6),
      legend.text = element_text(size = 6),
      strip.text = element_text(size = 7),
      plot.tag = element_text(size = 8, face = "bold"),
      panel.grid = element_blank()
    )
)

save_pub_r <- function(plot, filename, width_mm = 180, height_mm = 120, dpi = 600) {
  w <- width_mm / 25.4
  h <- height_mm / 25.4
  ragg::agg_png(paste0(filename, ".png"), width = w, height = h, units = "in", res = dpi)
  print(plot)
  dev.off()
  svglite::svglite(paste0(filename, ".svg"), width = w, height = h)
  print(plot)
  dev.off()
}
```

**Export policy**: PNG at 300–600 dpi is the default; add SVG only when editable
vector text is needed, and generate PDF only when the target journal explicitly
requires it.

## Default operating stance

- Start by classifying the requested figure into one of four archetypes:
  `quantitative grid`, `schematic-led composite`, `image plate + quant`, or `asymmetric mixed-modality figure`.
- Prefer one **hero panel** plus subordinate evidence panels over filling the canvas with equal-sized subplots.
- If the user asks for a single chart, still identify its role in the manuscript claim:
  discovery, mechanism, validation, comparison, robustness, or clinical/biological relevance.
- Keep the background white for plots and diagrams; switch to black only for microscopy / volume-rendering image plates.
- Prefer direct labels over legends when categories are spatially fixed or the legend would force unnecessary eye travel.
- Keep one restrained palette per figure: usually one neutral family, one signal family, and one accent family.
- Treat statistics, `n`, error-bar definitions, source-data traceability, and image-integrity notes as part of the figure,
  not as optional caption cleanup.
- When the user asks for broad `Nature` style rather than ML/NMI-specific style, read `references/nature-2026-observations.md` before choosing layout.

## When to load this skill

- Python or R figures for **papers, slides, or reports** targeting Nature, Science, Cell, NeurIPS, ICLR, or similar venues.
- Requests involving **grouped bars, trend lines, heatmaps, radar plots, multi-panel grids**, or **PNG/SVG/high-DPI** output.
- Any mention of "Nature style", "publication figure", "paper figure", "SCI figure", "R plotting template", or "high-quality scientific plot".
- Requests to improve a figure's logic, aesthetics, panel layout, figure legend, export quality, or journal-readiness.

## When NOT to load

- Plotly, Altair, Bokeh, or other interactive/web-first plotting.
- EDA-only plots without a publication target.
- Primary workflow is 3D, GIS, or non-scientific illustration tooling.
- Illustrator / Figma–first layout.

## Related files

| File | Open when |
|------|-----------|
| [references/figure-contract.md](references/figure-contract.md) | Need to convert a user request into core conclusion, evidence hierarchy, panel map, and review-risk checks |
| [references/backend-selection.md](references/backend-selection.md) | User has not chosen Python/R, asks for a recommendation, or a mixed Python/R workflow is possible |
| [references/r-workflow.md](references/r-workflow.md) | User chooses R or provides R scripts/templates/data |
| [references/r-template-index.md](references/r-template-index.md) | Need to adapt a user-provided or private R template collection without exposing source paths |
| [references/qa-contract.md](references/qa-contract.md) | Before final delivery, revision package, microscopy/blot figure, or journal-specific audit |
| [references/design-theory.md](references/design-theory.md) | Typography, color theory, layout rationale, export policy |
| [references/api.md](references/api.md) | Python PALETTE, helper function signatures, validation rules |
| [references/common-patterns.md](references/common-patterns.md) | Python layout patterns: hero panels, legend-only axes, dark image plates, asymmetric layouts |
| [references/nature-2026-observations.md](references/nature-2026-observations.md) | Real `Nature` page archetypes: schematic-led composites, dark image plates, clinical triptychs, asymmetric hero layouts |
| [references/tutorials.md](references/tutorials.md) | End-to-end walkthroughs: bars, trends, heatmaps |
| [references/chart-types.md](references/chart-types.md) | Radar, 3D sphere, fill_between, scatter patterns |
