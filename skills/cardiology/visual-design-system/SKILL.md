---
name: visual-design-system
description: "Publication-grade design tokens and utilities for Nature/JACC/NEJM quality graphics. Use when generating charts, infographics, animations, or any visual content that requires medical-journal-grade color palettes, typography, accessibility-validated contrast ratios, and consistent branding. Provides Python token APIs, colorblind-safe palettes, G2 chart templates, AntV infographic templates, Vizzu animation presets, and Manim integration for animated medical explainers."
---

# Visual Design System

**Purpose:** Publication-grade design tokens and utilities for Nature/JACC/NEJM quality graphics.

**Status:** Phase 3.1 In Progress - Manim Animations

---

## Which Tool Should I Use?

Use this decision table to pick the right backend before writing any code:

| Task | Best Tool | Output |
|------|-----------|--------|
| Publication figure (bar, line, forest plot) | **Plotly** or **drawsvg** | PNG (300 DPI) |
| Infographic card / social slide | **Satori** or **Component Library** | PNG, SVG |
| Medical diagram (heart, ECG, flowchart) | **drawsvg** | PNG, SVG |
| Clinical trial / drug mechanism template | **SVG Templates** | PNG, SVG |
| Treatment pathway / CONSORT / PRISMA | **Architecture Diagrams** | PNG |
| Animated mechanism / Kaplan-Meier / ECG | **Manim** | MP4 |
| Any of the above with a unified Python API | **Component Library** | PNG, SVG, HTML |

**Quick rule:** if it's a static publication figure → Plotly/drawsvg; if it's a polished card or social asset → Satori/Component; if it needs motion → Manim.

---

## Publication Figure Workflow (Create → Validate → Export)

For any publication-grade output, follow this validated sequence:

```python
from skills.cardiology.visual_design_system.tokens import (
    get_color, get_accessible_pair, validate_contrast, get_contrast_ratio
)
from cardiology_visual_system.scripts.plotly_charts import create_comparison_bars, save_chart

# 1. Create the figure using token colors
treatment, control = get_accessible_pair("treatment_control")
fig = create_comparison_bars(
    categories=["Primary", "Secondary"],
    group1_values=[12.3, 8.5],
    group2_values=[18.7, 14.2],
    group1_name="Treatment",
    group2_name="Placebo",
    title="Clinical Trial Results"
)

# 2. Validate contrast before export
if not validate_contrast(treatment, "#ffffff"):
    ratio = get_contrast_ratio(treatment, "#ffffff")
    # Fix: swap to a pre-validated pair
    treatment, control = get_accessible_pair("benefit_risk")

# 3. Validate DPI / scale — always use scale=4 for 300 DPI
# fig.write_image("results.png", scale=4)  # direct Plotly
save_chart(fig, "results.png")            # auto 300 DPI (scale=4)
```

**If validation fails:**

| Problem | Cause | Fix |
|---------|-------|-----|
| `validate_contrast` returns `False` | Ratio < 4.5:1 | Swap to a pre-validated pair via `get_accessible_pair()` |
| Render produces no output | Missing dependency | Run `pip install drawsvg cairosvg` or `pip install diagrams` |
| Manim scene not found | Not in catalog | Run `python scripts/render_manim.py --list` and check `scene_catalog.json` |
| 300 DPI export looks blurry | Wrong scale | Use `scale=4` in `fig.write_image()` or `save_chart(fig, path)` |
| Font not applied | System font missing | Tokens fall back to Arial; check `tokens.get_font_family("primary")` |

---

## Quick Start

```python
from skills.cardiology.visual_design_system.tokens import (
    get_tokens,
    get_color,
    get_accessible_pair,
    validate_contrast,
)

# Get a specific color
navy = get_color("primary.navy")  # "#1e3a5f"

# Get a colorblind-safe pair for treatment vs control
treatment, control = get_accessible_pair("treatment_control")

# Validate accessibility
is_safe = validate_contrast("#1e3a5f", "#ffffff")  # True (9.2:1 ratio)

# Get full tokens object
tokens = get_tokens()
palette = tokens.get_color_palette("categorical")  # 7 colorblind-safe colors
```

---

## Design Philosophy

This system enforces **Nature journal standards** for all visual output:

| Standard | Requirement | How We Enforce |
|----------|-------------|----------------|
| **Fonts** | Helvetica/Arial only | Token system + validation |
| **Font sizes** | 5-8pt for figures | Pre-defined size scale |
| **Contrast** | WCAG AA (4.5:1 min) | Automated validation |
| **Colorblind** | No red-green only | Paul Tol palettes |
| **Resolution** | 300 DPI minimum | Export presets |
| **Shadows** | None in figures | Disabled by default |

---

## Token Categories (Overview)

Full token reference: `references/color_palettes.md` and `references/nature_guidelines.md`.

### Colors

```python
# Primary
get_color("primary.navy")    # "#1e3a5f"
get_color("primary.blue")    # "#2d6a9f"
get_color("primary.teal")    # "#48a9a6"

# Semantic
get_color("semantic.success")  # "#2e7d32"
get_color("semantic.warning")  # "#e65100"
get_color("semantic.danger")   # "#c62828"
get_color("semantic.neutral")  # "#546e7a"

# Colorblind-safe palettes
tokens.get_color_palette("categorical")      # 7 Paul Tol colors
tokens.get_color_palette("sequential_blue")  # 5-step blue ramp
tokens.get_color_palette("diverging")        # blue ← neutral → red

# Pre-validated accessible pairs
t, c    = get_accessible_pair("treatment_control")    # ('#0077bb', '#ee7733')
b, r    = get_accessible_pair("benefit_risk")         # ('#009988', '#cc3311')
i, p    = get_accessible_pair("intervention_placebo") # ('#0077bb', '#bbbbbb')

# Clinical outcome colors
tokens.get_clinical_color("mortality")           # "#b2182b"
tokens.get_clinical_color("hospitalization")     # "#ef8a62"
tokens.get_clinical_color("symptom_improvement") # "#67a9cf"

# Forest plot colors
colors = tokens.get_forest_plot_colors()
# keys: point_estimate, confidence_interval, null_line, summary_diamond,
#       heterogeneity_low, heterogeneity_moderate, heterogeneity_high
```

### Typography

```python
tokens.get_font_family("primary")   # "Helvetica, Arial, sans-serif"
tokens.get_font_family("monospace") # "Courier New, Courier, monospace"

# Figure elements (Nature 5-8pt standard)
tokens.get_font_size("figure_elements", "panel_label")  # 8pt
tokens.get_font_size("figure_elements", "axis_title")   # 7pt
tokens.get_font_size("figure_elements", "axis_tick")    # 6pt

# Infographic / social media
tokens.get_font_size("infographic_elements", "headline")   # 24pt
tokens.get_font_size("social_media", "carousel_stat")      # 48pt
```

### Spacing & Strokes

```python
# 4px base grid
tokens.get_spacing("xs")  # "4px"  |  tokens.get_spacing("sm")  # "8px"
tokens.get_spacing("md")  # "12px" |  tokens.get_spacing("lg")  # "16px"

tokens.get_stroke_width("hairline")  # "0.5px"
tokens.get_stroke_width("thin")      # "1px"
tokens.get_stroke_width("regular")   # "1.5px"
```

---

## Validation

### CLI

```bash
python scripts/token_validator.py                  # Full validation
python scripts/token_validator.py --contrast-report
python scripts/token_validator.py --json
```

### Programmatic

```python
from tokens.index import validate_contrast, get_contrast_ratio

validate_contrast("#1e3a5f", "#ffffff")              # True  (WCAG AA)
validate_contrast("#1e3a5f", "#ffffff", level="AAA") # True  (WCAG AAA)
get_contrast_ratio("#1e3a5f", "#ffffff")             # 9.2
```

---

## Plotly (Phase 1.4)

Standard publication charts with automatic token integration and 300 DPI export.

```python
from cardiology_visual_system.scripts.plotly_charts import (
    create_comparison_bars, save_chart,
)

fig = create_comparison_bars(
    categories=["Primary", "Secondary"],
    group1_values=[12.3, 8.5],
    group2_values=[18.7, 14.2],
    group1_name="Treatment",
    group2_name="Placebo",
    title="Clinical Trial Results"
)
save_chart(fig, "results.png")  # Auto 300 DPI (scale=4)
```

```bash
# CLI
cd skills/cardiology/cardiology-visual-system/scripts
python plotly_charts.py demo --quality-report
python plotly_charts.py demo --png --output-dir ../outputs
python plotly_charts.py bar -d data.csv -o chart.png
```

**Direct template usage:**
```python
from tokens.index import get_plotly_template
import plotly.io as pio

pio.templates["publication"] = get_plotly_template()
fig = px.bar(data, template="publication")
fig.write_image("chart.png", scale=4)
```

---

## Satori Infographic Pipeline (Phase 1.2)

Generates PNG/SVG infographic cards from structured data via a Node.js renderer.

**Templates:** `stat-card`, `comparison`, `process-flow`, `trial-summary`, `key-finding`

```bash
cd satori/
node renderer.js --list
node renderer.js --template stat-card \
  --data '{"value": "26%", "label": "Mortality Reduction", "source": "PARADIGM-HF"}' \
  -o ../outputs/stat-card.png
```

```python
from scripts.generate_infographic import generate_stat_card, generate_trial_summary

generate_stat_card(
    "26%", "Mortality Reduction",
    sublabel="HR 0.74, 95% CI 0.65-0.85",
    source="PARADIGM-HF",
    output="outputs/stat-card.png"
)

generate_trial_summary(
    "DAPA-HF", "HFrEF patients", "Dapagliflozin 10mg",
    "CV death or HF hospitalization",
    0.74, "0.65-0.85", "<0.001",
    nnt=21,
    output="outputs/trial.png"
)
```

Output: 1200×630px PNG at 2x scale. Custom dimensions: pass `width=` / `height=` to any generator or `--width`/`--height` to the CLI.

---

## drawsvg Pipeline (Phase 1.3)

Pure Python SVG for medical diagrams and charts. No Node.js required.

```bash
pip install drawsvg cairosvg
```

**Modules:** `medical_diagrams` (heart, ECG, conduction, organ icons), `data_charts` (bar, grouped bar, line, forest plot), `process_flows` (algorithm, patient journey, study flow, simple flow).

```python
from drawsvg.medical_diagrams import ecg_wave
from drawsvg.data_charts import forest_plot
from drawsvg.process_flows import study_flow

# ECG waveform
svg = ecg_wave(wave_type="normal", show_labels=True, title="Normal Sinus Rhythm")
svg.save_png("ecg.png")

# Forest plot
studies = [
    {"name": "DAPA-HF",         "estimate": 0.74, "lower": 0.65, "upper": 0.85, "weight": 60},
    {"name": "EMPEROR-Reduced", "estimate": 0.75, "lower": 0.65, "upper": 0.86, "weight": 50},
    {"name": "DELIVER",         "estimate": 0.82, "lower": 0.73, "upper": 0.92, "weight": 70},
]
svg = forest_plot(studies=studies, title="SGLT2 Inhibitors in HF", show_pooled=True)
svg.save_png("forest_plot.png")
# → For component-based forest plot with backend selection, see Component Library below.

# CONSORT study flow
svg = study_flow(enrollment=1500, randomized=1200,
    groups=[
        {"name": "Treatment", "allocated": 600, "discontinued": 45, "analyzed": 555},
        {"name": "Control",   "allocated": 600, "discontinued": 52, "analyzed": 548},
    ],
    title="DAPA-HF Study Flow"
)
svg.save_png("study_flow.png")
# → For CONSORT diagrams with auto-routing, see Architecture Diagrams below.
```

For full parameter reference (all `wave_type` values, `highlight_chamber` options, etc.) see `svg_diagrams/`.

---

## Component Library (Phase 2.1)

Unified Python API wrapping Satori, Plotly, and drawsvg with consistent interfaces.

**Components:** `StatCard`, `ComparisonChart`, `ForestPlot`, `Timeline`, `ProcessFlow`, `DataTable`

```python
from components import StatCard, ForestPlot, ComparisonChart, DataTable

card = StatCard(value="26%", label="Mortality Reduction",
                sublabel="HR 0.74, 95% CI 0.65-0.85", source="PARADIGM-HF")
card.render("stat_card.png")                    # auto backend
card.render("stat_card.png", backend="satori")  # infographic style
card.render("stat_card.png", backend="drawsvg") # publication style

# Forest plot with backend selection (see drawsvg section for raw SVG alternative)
plot = ForestPlot(studies=[...], title="SGLT2i in HF", x_label="Hazard Ratio (95% CI)")
plot.render("forest.png", backend="plotly")

table = DataTable(
    title="Baseline Characteristics",
    headers=["Characteristic", "Treatment (n=500)", "Control (n=500)", "P-value"],
    rows=[["Age, years", "65.2 ± 12.1", "64.8 ± 11.9", "0.62"]],
    footer="Values are mean ± SD or n (%)"
)
table.render("baseline.png")
```

**Backend selection:**

| Backend | Best For |
|---------|----------|
| `satori` | Infographic cards, social media |
| `plotly` | Interactive charts, data viz |
| `drawsvg` | Publication figures, diagrams |

**Resolution config:**
```python
from components.base import RenderConfig
config = RenderConfig(width=1200, height=630, quality="print")  # 300 DPI
card = StatCard(value="42%", label="Test", config=config)
```

---

## SVG Infographic Templates (Phase 2.2)

lxml-based SVG placeholder replacement for five standard medical layouts.

**Templates:** `trial_results`, `drug_mechanism`, `patient_stats`, `before_after`, `risk_factors`

Full field reference: see `TEMPLATE_REFERENCE.md` in `svglue_templates/`.

```bash
cd svglue_templates/
python template_renderer.py --list
python template_renderer.py trial_results --demo -o output.svg
python template_renderer.py trial_results --demo --png --scale 2 -o output.svg
```

```python
from svglue_templates.template_renderer import render_template, save_svg, save_png
from pathlib import Path

svg = render_template("trial_results", {
    "trial_name": "PARADIGM-HF",
    "primary_hr": "0.80",
    "primary_ci": "95% CI: 0.73-0.87",
    "primary_p": "P < 0.001",
    "source": "McMurray JJV et al. N Engl J Med. 2014",
})
save_svg(svg, Path("trial_results.svg"))
save_png(svg, Path("trial_results.svg"), scale=2)  # 1600×1200 PNG
```

---

## Architecture Diagrams (Phase 2.3)

Publication-grade clinical pathways and research flow diagrams using `mingrammer/diagrams`.

```bash
pip install diagrams
brew install graphviz  # macOS
```

**Modules:** `treatment_pathways` (HF, ACS, AF algorithms), `research_flows` (CONSORT, PRISMA, methodology), `healthcare_arch` (hospital system, cardiology dept, data pipeline).

```python
from arch_diagrams.treatment_pathways import create_heart_failure_pathway
from arch_diagrams.research_flows import create_consort_diagram

create_heart_failure_pathway(output_path="outputs/hf_pathway", format="png")
# → GDMT initiation → ACEi/ARNi → Beta-blocker → MRA → SGLT2i → Device therapy

# CONSORT diagram (see drawsvg section for pure-Python SVG alternative)
create_consort_diagram(
    enrolled=500, randomized=400,
    treatment_n=200, control_n=200,
    treatment_completed=180, control_completed=175,
    treatment_analyzed=200, control_analyzed=200,
    output_path="outputs/consort", format="png"
)
```

**Diagram color coding:**

| Element | Color | Meaning |
|---------|-------|---------|
| Assessment | Blue (#2d6a9f) | Diagnostics |
| Decision | Orange (#e65100) | Stratification |
| Treatment | Green (#2e7d32) | Active therapy |
| Danger/Critical | Red (#c62828) | ICU, exclusions |

```bash
python arch_diagrams/treatment_pathways.py  # all pathways
python arch_diagrams/research_flows.py      # all research flows
python arch_diagrams/healthcare_arch.py     # all architecture diagrams
```

---

## Manim Animations (Phase 3.1)

Educational animations for mechanisms, survival curves, and ECG fundamentals.

```bash
python -m venv .venv-manim
.venv-manim/bin/python -m pip install manim
```

**Key scenes:**

| Key | Scene Class | Description |
|-----|-------------|-------------|
| `mechanism` | `MechanismOfActionScene` | 4-step mechanism flow with outcome callout |
| `kaplan_meier` | `KaplanMeierScene` | Stepwise survival curves + HR label |
| `ecg_wave` | `ECGWaveScene` | Normal sinus rhythm with labels |

Full catalog: `manim_animations/scene_catalog.json` — categories include cardiometabolic, ACS/CAD, arrhythmia, imaging/DX, statistics, devices, anatomy.

```bash
cd skills/cardiology/visual-design-system

python scripts/render_manim.py --list
python scripts/render_manim.py mechanism     --quality m --format mp4
python scripts/render_manim.py kaplan_meier --quality h --preview
python scripts/render_manim.py ecg_wave     --quality l --manim-bin .venv-manim
```

- Outputs → `outputs/manim/`
- Colors and fonts sourced from design tokens via `manim_animations/theme.py`
- Carousel slides with `animation_scene` route to Manim via `carousel-generator-v2`

---

## Directory Structure

```
visual-design-system/
├── SKILL.md
├── tokens/
│   ├── index.py            # Main token loader
│   ├── colors.json
│   ├── typography.json
│   ├── spacing.json
│   └── shadows.json
├── scripts/
│   ├── token_validator.py
│   ├── generate_infographic.py
│   └── render_manim.py
├── satori/                 # Phase 1.2 - React → SVG → PNG
├── svg_diagrams/           # Phase 1.3 - Pure Python SVG
├── components/             # Phase 2.1 - Component Library
├── svglue_templates/       # Phase 2.2 - SVG Templates
│   └── TEMPLATE_REFERENCE.md  # Full field docs for all 5 templates
├── arch_diagrams/          # Phase 2.3 - Architecture Diagrams
├── manim_animations/       # Phase 3.1 - Manim scenes + catalog
├── references/
│   ├── nature_guidelines.md
│   └── color_palettes.md
└── outputs/
```

---

## References

- [Nature Figure Guidelines](https://www.nature.com/nature/for-authors/preparing-your-submission)
- [Paul Tol's Colorblind-Safe Palettes](https://personal.sron.nl/~pault/)
- [WCAG 2.1 Contrast Requirements](https://www.w3.org/WAI/WCAG21/Understanding/contrast-minimum)
- [Satori Documentation](https://github.com/vercel/satori)
- `references/nature_guidelines.md` — full Nature journal figure standards
- `references/color_palettes.md` — complete palette definitions
- `svglue_templates/TEMPLATE_REFERENCE.md` — all template field definitions

---

*Last Updated: 2026-01-01*
*Maintainer: Dr. Shailesh Singh*
