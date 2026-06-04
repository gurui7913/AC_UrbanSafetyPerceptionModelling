# Urban Safety Perception Modelling
**Track the Space, Decode the Fear**

## Project Overview
This project investigates how spatial configuration and visual environment 
jointly shape pedestrians' perception of safety in urban streetscapes.
By combining Space Syntax metrics with street-view semantic segmentation, 
we built an analytical pipeline that models perceived safety from both 
2D connectivity structure and 3D visual features.

**Research Question:** How do spatial configuration (connectivity, accessibility) 
and visual environment (greenery, sky openness) jointly influence 
pedestrians' perceived safety in London's urban streetscapes — 
and do these relationships follow linear or non-linear patterns?

---

## Pipeline Overview

- Crowdsourced Safety Data Collection (Place Pulse 2.0 · ~900 London locations)
- Space Syntax Metric Extraction (Integration INT2K + Choice CH2K · 2km radius)
- Street-View Semantic Segmentation (SegFormer-B0 · ADE20K)
- Visual Feature Derivation (Green View Ratio + Sky Visibility)
- Multi-Variable Regression Modelling (Linear → Polynomial → Interaction)
- Extreme Case Analysis (Min/Max identification for qualitative validation)

---

## Methods

### Data Collection
- Selected ~900 georeferenced locations across London from **Place Pulse 2.0** (MIT Media Lab)
- Safety perception scores derived via **TrueSkill** pairwise comparison algorithm
- Space Syntax metrics sourced from **Space Syntax Open Mapping** (OS Meridian 2 road network)
- Street View Images processed locally via **SegFormer-B0** (ADE20K fine-tune, Hugging Face)

### Feature Extraction
- **Spatial features:** INT2K (Integration) and CH2K (Choice) quantify 
  how connected and how traversed each street segment is at 2km radius
- **Visual features:** SegFormer-B0 pixel-classifies each SVI into 150 semantic 
  categories; green_view_ratio and sky_visibility derived from class proportions
- **Multicollinearity check:** VIF and Pearson correlation confirm INT2K–CH2K 
  collinearity (r = 0.681), informing model variable selection

### Regression Models
- **Linear baseline** establishes individual predictor effects
- **Polynomial regression (Degree 2)** adds quadratic and cross-product terms 
  to capture non-linear and interaction effects
- **Interaction-focused model** isolates theoretically motivated term pairs 
  (e.g. INT2K × green_view_ratio, green_view_ratio × sky_visibility)
- **Enhanced linear model** combines linear terms with targeted interaction terms 
  selected from domain knowledge

### Extreme Case Analysis
- Identified locations with maximum/minimum values for each variable
- Exported to CSV for qualitative streetscape comparison
- Validates model direction: high-INT2K + closed-sky → suppressed safety scores

---

## Key Findings
- Identified **non-linear threshold effects**: moderate integration, greenery, 
  and sky openness associate with highest safety scores; extremes in either 
  direction reduce perceived safety
- **Inverted-U relationships** across all four predictors — over-integrated 
  commercial corridors and visually enclosed spaces both score lower than mid-range environments
- **Cross-modal interaction effects**: spatial connectivity (INT2K) shows 
  stronger positive effect when co-occurring with sufficient visual openness 
  and green coverage
- **INT2K as dominant predictor** (r = 0.072); CH2K contributes negligible 
  independent variance (r ≈ 0), likely due to high collinearity with INT2K

---

## Limitations
- Moderate dataset size (~900 locations) limits generalisation across city types
- Place Pulse 2.0 reflects a global, non-London-specific crowd; local 
  cultural safety norms may introduce bias
- Static SVIs do not capture temporal variation (day/night, seasonal change)
- SegFormer-B0 trained on ADE20K; domain shift may affect fine-grained 
  streetscape categories

## Future Work
- Extend pipeline to Chinese cities using OSMnx-derived Space Syntax + 
  local pairwise perception data
- Incorporate dynamic SVIs and temporal safety variation
- Replace polynomial regression with spatially-aware models 
  (GWR, spatial lag) to account for geographic clustering

---

## Tech Stack
- Python, Jupyter Notebook
- HuggingFace Transformers (SegFormer-B0)
- statsmodels (OLS, VIF diagnostics)
- scikit-learn (PolynomialFeatures, StandardScaler)
- pandas / numpy / matplotlib / seaborn / scipy

## Dependencies
```bash
pip install pandas numpy matplotlib seaborn statsmodels scikit-learn scipy transformers
```

---

## Repository Structure
```
02_DataAnalysis/
├── Formula.ipynb           # Mathematical model formulations & notation reference
├── SS_Model_V1.ipynb       # Main regression analysis
│                             - Linear & polynomial regression
│                             - Correlation, VIF, residual diagnostics
│                             - Distribution & scatter visualisation
└── min_max.ipynb           # Extreme value identification & CSV export
```

---

## License

This project is for academic research purposes. Please cite the original dissertation if you use this code or methodology.

## Acknowledgements

- [Place Pulse 2.0](http://pulse.media.mit.edu/) — MIT Media Lab
- [Space Syntax Open Mapping](https://www.spacesyntax.net/)
- [SegFormer](https://huggingface.co/nvidia/segformer-b0-finetuned-ade-512-512) — NVIDIA / Hugging Face

---

## Team
- **Rui Gu** — Research Design, Feature Engineering, Statistical Modelling, Interpretation

*UCL Bartlett · MSc Architectural Computation · Dissertation, 2025*
