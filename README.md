# Urban Safety Perception Modelling

**Combining Space Syntax and Street-View Semantic Segmentation to Model Non-Linear Safety Perception in London Streetscapes**

> UCL Bartlett School of Architecture · MSc Architectural Computation — Dissertation  
> **Gu Rui** · 2025

> **My contribution:** Research design · Feature engineering pipeline · Regression modelling · Statistical diagnostics · Extreme case analysis

---

## Table of Contents

- [Research Question](#research-question)
- [Pipeline Architecture](#pipeline-architecture)
- [Data Sources](#data-sources)
- [Feature Extraction](#feature-extraction)
  - [Space Syntax Metrics](#space-syntax-metrics)
  - [Street-View Semantic Segmentation](#street-view-semantic-segmentation)
  - [Multicollinearity Diagnostics](#multicollinearity-diagnostics)
- [Regression Models](#regression-models)
  - [Model 1 — Multiple Linear Regression (Baseline)](#model-1--multiple-linear-regression-baseline)
  - [Model 2 — Polynomial Regression (Degree 2)](#model-2--polynomial-regression-degree-2)
  - [Model 3 — Interaction Effect Model](#model-3--interaction-effect-model)
  - [Model 4 — Enhanced Linear Model with Targeted Interactions](#model-4--enhanced-linear-model-with-targeted-interactions)
- [Extreme Case Analysis](#extreme-case-analysis)
- [Results](#results)
- [Repository Structure](#repository-structure)
- [How to Run](#how-to-run)
- [Future Work](#future-work)

---

## Research Question

Standard urban safety research treats spatial configuration and visual environment as **separate explanatory systems** — Space Syntax studies quantify connectivity but ignore what pedestrians actually see; street-view studies capture visual impressions but ignore structural accessibility. This project bridges that gap:

> **How do spatial configuration (connectivity, accessibility) and visual environment (greenery, sky openness) jointly influence pedestrians' perceived safety in London streetscapes — and do these relationships follow linear or non-linear patterns?**

We operationalize this through a four-predictor regression framework using crowdsourced pairwise safety scores (Place Pulse 2.0) as ground truth. We hypothesize that **polynomial terms and cross-modal interaction effects** will reveal threshold dynamics invisible to linear models — specifically that mid-range values of integration and visual openness, not extremes, correlate with highest perceived safety.

---

## Pipeline Architecture

```
Stage 1                Stage 2                  Stage 3               Stage 4
Crowdsourced     ──►   Feature               ──► Regression      ──►  Extreme Case
Safety Scores          Extraction                 Modelling            Analysis

• Place Pulse 2.0      • Space Syntax metrics    • 4 model variants    • Min/max per variable
• TrueSkill scores       INT2K, CH2K             • Linear baseline     • CSV export
• ~900 London            2km radius              • Polynomial Deg 2    • Qualitative
  locations           • SegFormer-B0               + interactions        streetscape
• Georeferenced          ADE20K 150-class        • VIF + residual        validation
  points                 segmentation              diagnostics
                       • green_view_ratio
                         sky_visibility
```

---

## Data Sources

| Source | Description | Format |
|--------|-------------|--------|
| **Place Pulse 2.0** (MIT Media Lab) | Crowdsourced pairwise safety comparisons across global cities; TrueSkill algorithm converts wins/losses to continuous scores | CSV · `safer_Trueskill_Scores` per location ID |
| **Space Syntax Open Mapping** | Pre-computed axial map metrics for the UK road network (OS Meridian 2); Integration and Choice at configurable radii | CSV · matched by nearest-node to Place Pulse coordinates |
| **Street View Images** | Static SVIs at each Place Pulse location; processed locally via SegFormer-B0 | JPEG · pixel-level semantic class maps |

**Study area:** ~900 georeferenced locations across London, spanning central business districts, mixed-use corridors, and residential neighbourhoods. London selected for urban complexity, historical street-network depth, and open data availability.

---

## Feature Extraction

### Space Syntax Metrics

Space Syntax quantifies how structurally integrated or traversed each street segment is within the wider network, independently of Euclidean distance.

| Metric | Symbol | Definition | Radius |
|--------|--------|------------|--------|
| Integration | `INT2K` | Normalised inverse mean depth — how easily all other segments are reached from this one | 2 km |
| Choice | `CH2K` | Betweenness centrality — how often this segment appears on shortest paths between all pairs | 2 km |

- **High INT2K** → segment is topologically central, easily reachable (commercial corridors, high streets)
- **High CH2K** → segment is heavily traversed by through-movement (arterial routes)
- Metrics sourced from **Space Syntax Open Mapping**, matched to Place Pulse coordinates via nearest-node spatial join

### Street-View Semantic Segmentation

Visual features derived from per-pixel semantic segmentation using **SegFormer-B0** fine-tuned on ADE20K (150 semantic categories, `nvidia/segformer-b0-finetuned-ade-512-512`):

```python
from transformers import SegformerFeatureExtractor, SegformerForSemanticSegmentation

extractor = SegformerFeatureExtractor.from_pretrained(
    "nvidia/segformer-b0-finetuned-ade-512-512"
)
model = SegformerForSemanticSegmentation.from_pretrained(
    "nvidia/segformer-b0-finetuned-ade-512-512"
)

inputs = extractor(images=image, return_tensors="pt")
outputs = model(**inputs)
logits = outputs.logits  # shape: (1, 150, H/4, W/4)
seg_map = logits.argmax(dim=1)  # class per pixel
```

**Derived features:**

| Feature | ADE20K Classes | Computation |
|---------|---------------|-------------|
| `green_view_ratio` | `tree` (4), `grass` (9), `plant` (17), `flower` (66) | `green_pixels / total_pixels` |
| `sky_visibility` | `sky` (2) | `sky_pixels / total_pixels` |

### Multicollinearity Diagnostics

Before modelling, VIF and Pearson correlation were computed across all four predictors:

| Predictor Pair | Pearson r | Interpretation |
|----------------|-----------|----------------|
| INT2K — CH2K | **0.681** | High collinearity; both measure network centrality at 2km |
| INT2K — green_view_ratio | −0.12 | Weak negative (urban cores have less greenery) |
| INT2K — sky_visibility | −0.09 | Weak negative |
| green_view_ratio — sky_visibility | 0.18 | Weak positive |

**Implication:** CH2K and INT2K cannot be treated as independent predictors in a linear model. The high INT2K–CH2K correlation informs polynomial model interpretation — quadratic CH2K terms partially absorb INT2K variance.

---

## Regression Models

All models use `safer_Trueskill_Scores` as the dependent variable $Y$.  
Predictors are standardized via `StandardScaler` before fitting.

**Variable notation:**

| Symbol | Variable |
|--------|----------|
| $X_1$ | `INT2K` — Integration at 2km |
| $X_2$ | `CH2K` — Choice at 2km |
| $X_3$ | `green_view_ratio` |
| $X_4$ | `sky_visibility` |
| $Y$ | `safer_Trueskill_Scores` |

---

### Model 1 — Multiple Linear Regression (Baseline)

$$Y = \beta_0 + \beta_1 X_1 + \beta_2 X_2 + \beta_3 X_3 + \beta_4 X_4 + \varepsilon$$

Fitted via OLS (`statsmodels.OLS`). Establishes individual linear predictor effects and serves as the baseline for non-linear model comparison. Residual diagnostics (Shapiro-Wilk normality test, Q-Q plot, heteroscedasticity check) run post-fit.

---

### Model 2 — Polynomial Regression (Degree 2)

$$Y = \beta_0 + \sum_{i=1}^{4} \beta_i X_i + \sum_{i=1}^{4} \gamma_i X_i^2 + \sum_{i < j} \delta_{ij} X_i X_j + \varepsilon$$

Generated via `sklearn.preprocessing.PolynomialFeatures(degree=2, include_bias=False)`, then fitted with OLS.

**Term count:** 4 linear + 4 quadratic + 6 cross-product = **14 terms**

The quadratic terms $\gamma_i X_i^2$ are the key test of the inverted-U hypothesis — a negative $\gamma_i$ coefficient indicates a safety peak at mid-range $X_i$ values.

---

### Model 3 — Interaction Effect Model

Retains only the six theoretically motivated cross-product terms, dropping quadratic terms:

$$Y = \beta_0 + \sum_{i=1}^{4} \beta_i X_i + \delta_{13}(X_1 X_3) + \delta_{14}(X_1 X_4) + \delta_{23}(X_2 X_3) + \delta_{24}(X_2 X_4) + \delta_{34}(X_3 X_4) + \delta_{12}(X_1 X_2) + \varepsilon$$

Rationale: tests whether spatial-visual cross-modal interactions (e.g. `INT2K × green_view_ratio`) explain safety variance beyond individual effects, without the overfitting risk of full polynomial expansion.

---

### Model 4 — Enhanced Linear Model with Targeted Interactions

Combines linear predictors with only the two highest-signal interaction terms identified from Model 3:

$$Y = \beta_0 + \sum_{i=1}^{4} \beta_i X_i + \delta_{13}(X_1 X_3) + \delta_{34}(X_3 X_4) + \varepsilon$$

Selected interactions:
- `INT2K × green_view_ratio` ($\delta_{13}$): connectivity amplifies safety return from greenery
- `green_view_ratio × sky_visibility` ($\delta_{34}$): co-occurring openness (sky + green) produces stronger safety signal than either alone

---

## Extreme Case Analysis

`min_max.ipynb` identifies the real London locations corresponding to extreme values of each predictor and the safety score. Outputs a CSV for qualitative streetscape inspection.

**Selected extreme cases:**

| Case | Safety Score | INT2K | CH2K | Interpretation |
|------|:---:|:---:|:---:|----------------|
| Max safety (9.64) | — | ~179 | ~16,248 | Mid-range INT2K; moderate connectivity, not over-integrated |
| Min safety (0.81) | — | ~32 | ~373 | Low connectivity + likely enclosed visual field |
| Max INT2K = Max CH2K | 4.32 | 677.2 | 201,083 | Over-integrated commercial zone; excess connectivity suppresses safety |

**Key insight:** The highest-INT2K location is not the safest — it scores only 4.32. This directly validates the inverted-U hypothesis from Model 2: beyond a threshold, greater network centrality correlates with reduced perceived safety (busy arterials, impersonal commercial strips).

---

## Results

### Model Comparison

| Model | Terms | R² | Key Insight |
|-------|:-----:|:--:|-------------|
| Linear Baseline | 4 | ~0.017 | Weak individual effects; INT2K dominant (r = 0.072) |
| Polynomial Deg 2 | 14 | **~0.081** | ~5× improvement; quadratic terms confirm inverted-U |
| Interaction Only | 10 | Intermediate | Cross-modal terms add signal beyond linear |
| Enhanced Linear | 6 | Intermediate | Parsimonious; `INT2K × green` and `green × sky` most informative |

### Emergent Patterns

| Pattern | Evidence |
|---------|----------|
| Inverted-U: Integration | Negative $\gamma_1$ coefficient in Model 2; max-INT2K case scores 4.32 |
| Inverted-U: Greenery | Negative $\gamma_3$ coefficient; dense canopy can signal isolation |
| Inverted-U: Sky visibility | Negative $\gamma_4$ coefficient; overexposed open spaces feel unsafe |
| Cross-modal interaction | $\delta_{13}$ (`INT2K × green`) significant; connectivity amplifies greenery's safety effect |
| CH2K near-zero main effect | r ≈ 0 with safety; variance absorbed by INT2K due to collinearity (r = 0.681) |

---

## Repository Structure

```
02_DataAnalysis/
├── Formula.ipynb              # Mathematical model formulations & variable notation
│                                - All 4 model equations in LaTeX
│                                - Variable definitions and units
│
├── SS_Model_V1.ipynb          # Main regression analysis pipeline
│                                - Data loading & StandardScaler normalization
│                                - Model 1: OLS linear regression
│                                - Model 2: PolynomialFeatures + OLS
│                                - Model 3 & 4: custom interaction term construction
│                                - Pearson correlation matrix & heatmap
│                                - VIF multicollinearity table
│                                - Residual diagnostics (Shapiro-Wilk, Q-Q, scatter)
│                                - Coefficient bar plots per model
│
└── min_max.ipynb              # Extreme value identification & export
                                 - Per-variable min/max location lookup
                                 - Safety score at extreme cases
                                 - CSV export for qualitative case study
```

---

## How to Run

### Prerequisites

```bash
pip install pandas numpy matplotlib seaborn statsmodels scikit-learn scipy transformers torch
```

### Data Setup

Ensure the following files are available in your data directory:

```
data/
├── place_pulse_london.csv      # Place Pulse 2.0 London subset
│                                 columns: location_id, lat, lon, safer_Trueskill_Scores
├── space_syntax_london.csv     # Space Syntax Open Mapping export
│                                 columns: location_id, INT2K, CH2K
└── visual_features.csv         # SegFormer-B0 outputs (pre-computed)
                                  columns: location_id, green_view_ratio, sky_visibility
```

> **Note:** Street-view images must be pre-processed through SegFormer-B0 to generate `visual_features.csv`. See feature extraction code in `SS_Model_V1.ipynb` Section 2.

### Running the Analysis

**Step 1 — Check model formulations:**
```
Open Formula.ipynb
```
Reference for all variable definitions and mathematical notation used in the modelling notebooks.

**Step 2 — Run regression pipeline:**
```
Open SS_Model_V1.ipynb → Run All
```
Executes data merging, normalization, all four regression models, diagnostic plots, and coefficient visualizations in sequence.

**Step 3 — Extreme case export:**
```
Open min_max.ipynb → Run All
```
Outputs `extreme_cases.csv` with location IDs, predictor values, and safety scores for qualitative streetscape inspection.

### Reproducing SegFormer Feature Extraction

```python
from transformers import SegformerFeatureExtractor, SegformerForSemanticSegmentation
import torch, numpy as np
from PIL import Image

extractor = SegformerFeatureExtractor.from_pretrained(
    "nvidia/segformer-b0-finetuned-ade-512-512"
)
model = SegformerForSemanticSegmentation.from_pretrained(
    "nvidia/segformer-b0-finetuned-ade-512-512"
)
model.eval()

# ADE20K class indices
GREEN_CLASSES = {4, 9, 17, 66}   # tree, grass, plant, flower
SKY_CLASS     = {2}               # sky

def extract_visual_features(image_path):
    img = Image.open(image_path).convert("RGB")
    inputs = extractor(images=img, return_tensors="pt")
    with torch.no_grad():
        logits = model(**inputs).logits
    seg = logits.argmax(dim=1).squeeze().numpy()
    total = seg.size
    green = np.isin(seg, list(GREEN_CLASSES)).sum() / total
    sky   = np.isin(seg, list(SKY_CLASS)).sum()   / total
    return {"green_view_ratio": green, "sky_visibility": sky}
```

---

## Future Work

- **Chinese city extension** — rebuild Space Syntax metrics for target cities (e.g. Guangzhou, Shenzhen) using **OSMnx** + **momepy**; replace Place Pulse 2.0 with locally-collected pairwise comparisons or CLIP-based scoring to address cross-cultural perception bias
- **Spatially-aware models** — replace global OLS with **Geographically Weighted Regression (GWR)** or spatial lag models to account for spatial autocorrelation in safety scores
- **Dynamic SVIs** — incorporate time-of-day and seasonal variation to capture temporal safety perception shifts
- **End-to-end pipeline** — replace manual CSV merging with a unified geospatial pipeline (GeoPandas + PostGIS) for scalable city-level deployment

---

## References

1. Dubey, A. et al. "Deep Learning the City: Quantifying Urban Perception at a Global Scale." *ECCV*, 2016. *(Place Pulse 2.0)*
2. Xie, E. et al. "SegFormer: Simple and Efficient Design for Semantic Segmentation with Transformers." *NeurIPS*, 2021.
3. Hillier, B. & Hanson, J. *The Social Logic of Space*. Cambridge University Press, 1984. *(Space Syntax foundations)*
4. Salesses, P. et al. "The Collaborative Image of The City: Mapping the Inequality of Urban Perception." *PLOS ONE*, 2013.

---

## License

Developed as part of the UCL Bartlett MSc Architectural Computation dissertation programme.  
Please cite the original dissertation if you use this code or methodology.
