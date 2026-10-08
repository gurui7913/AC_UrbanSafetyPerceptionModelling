# Urban Safety Perception Modelling

Combining Space Syntax and street-view semantic segmentation to explore perceived safety in London.

**Gu Rui · UCL Bartlett · MSc Architectural Computation dissertation · 2025**

[中文说明](README.zh-CN.md) · [Data requirements](docs/DATA.md) · [Repository audit](docs/AUDIT.md)

This repository contains the original research notebooks, reorganized and documented in October 2026. It is a research archive with local-data prerequisites. The full data pipeline is not yet portable or reproducible from a fresh clone alone.

## Research and pipeline

The study relates Place Pulse 2.0 safety perception scores to street-network configuration and visible environmental features. The local modelling table contains **900 unique London location IDs**.

1. Prepare Place Pulse TrueSkill scores; filter observations with at least 12 comparisons and normalize within perceptual dimension.
2. Match London locations to Space Syntax street **lines**, extracting `INT2K` and `CH2K`.
3. Segment selected images with `nvidia/segformer-b0-finetuned-ade-512-512`.
4. Combine spatial and visual features by `location_id` into `merged_data.csv` (the original merge code is absent).
5. Fit OLS models and inspect coefficients, residuals, interactions, and extreme cases.

The outcome is perceived safety, not recorded crime or a causal measure of street safety.

## Features and actual model implementation

| Field | Meaning |
| --- | --- |
| `safer_Trueskill_Scores` | Normalized safety perception score, target range 0–10 |
| `INT2K` | Space Syntax integration at the 2 km radius |
| `CH2K` | Space Syntax choice at the 2 km radius |
| `green_view_ratio` | Percentage of image pixels classified as tree (4), grass (9), or plant (17) |
| `sky_visibility` | Percentage of image pixels classified as sky (2) |

Both visual features are **percentages (0–100)**. Flower pixels are not included by the original implementation. Segmentation logits are resized to the original image dimensions before classification.

`MergedData_Model_V2.ipynb` standardizes four predictors and fits three models:

| Model | Predictors, excluding intercept | Recorded in-sample R² | Adjusted R² |
| --- | ---: | ---: | ---: |
| Linear baseline | 4 | 0.017405 | 0.013013 |
| Degree-2 polynomial | 14 | 0.080896 | 0.066357 |
| Linear + pairwise interactions | 10 | 0.034764 | 0.023906 |

The interaction model includes all six pairwise products. The current code does not fit a separate fourth model using only two interactions. `Formula.ipynb` retains historical conceptual formulations and should be read alongside the implementation.

Values come from [the saved comparison table](results/model_comparison_results.csv). During the 2026 audit, all three R² values were independently recomputed with NumPy least squares from the existing local modelling table and agreed within `1e-10`. This verifies the reported fits; it does not validate upstream spatial matching or segmentation accuracy.

The models explain a small portion of the observed variation. There is no executed held-out or spatial cross-validation in these notebooks. Polynomial improvements are exploratory in-sample associations and do not establish predictive generalization, optimal design thresholds, or causal effects.

## Repository layout

```text
01_DataProcessing/          Original preparation, spatial matching and segmentation notebooks
02_DataAnalysis/            Original baseline, combined model, formulas and extreme-case notebooks
results/
  model_comparison_results.csv  Aggregate comparison exported by the original analysis
docs/
  DATA.md                   Exact inputs, schemas and manual preparation requirements
  AUDIT.md                  Audit findings and verification limits
  notebook-source-manifest.json  Source hashes and original local notebook locations
requirements.txt            Analysis dependencies inferred from imports
requirements-full.txt       Additional GIS / segmentation dependencies
```

## Notebook guide

| Stage | Notebook | Role |
| --- | --- | --- |
| 1 | [Place Pulse preparation](01_DataProcessing/20250521%20PP数据集合并简化.ipynb) | Prepare, filter and normalize scores |
| 2 | [Location_SaferScores_SS](01_DataProcessing/Location_SaferScores_SS.ipynb) | London subset, point-to-line matching, network feature export |
| 3 | [ImageSegmentation](01_DataProcessing/ImageSegmentation.ipynb) | Extract image greenery and sky percentages |
| 4 | [SS_Model_V1](02_DataAnalysis/SS_Model_V1.ipynb) | Spatial-only OLS baseline with two predictors |
| 5 | [MergedData_Model_V2](02_DataAnalysis/MergedData_Model_V2.ipynb) | Main combined spatial + visual analysis and model comparison |
| Reference | [Formula](02_DataAnalysis/Formula.ipynb) | Historical model equations |
| Supplement | [min_max](02_DataAnalysis/min_max.ipynb) | Extreme values of the spatial table and safety scores |

## Running locally

Use a separate Python environment. The audit used Python 3.10; the original dependency versions were not recorded. These dependency lists are installation guides, not a verified recreation of the original runtime.

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
jupyter lab
```

For GIS preprocessing and image segmentation, also install `requirements-full.txt`. The segmentation notebook downloads its model weights on first use.

Before running any notebook:

1. Obtain the inputs listed in [docs/DATA.md](docs/DATA.md).
2. Replace all hard-coded Windows input **and output** paths with paths on your machine. Several cells write to the original data folders.
3. If starting from raw data, resolve the coordinate-system issue described in [docs/AUDIT.md](docs/AUDIT.md) before regenerating the spatial table.
4. Prepare `merged_data.csv` explicitly by a validated one-to-one `location_id` join. Its creation is not implemented in the archived notebooks.
5. Restart the kernel and run the relevant notebook in order. For modelling only, use the prepared spatial table for V1 and the merged table for V2.

Notebook outputs and execution counts were cleared during organization; original executed copies remain in the local dissertation archive. Notebook source cells were preserved exactly.

## Data and reuse

The complete street-view image collection, raw TSVs, GIS layers, thesis drafts, and personal discussion documents are outside this repository. Previously uploaded legacy data and descriptive exports have been removed from the current GitHub tree and retained locally under ignored `data/legacy/` and `results/legacy/` folders. Existing Git history is preserved.

Data and pretrained weights remain subject to their respective providers' terms. No new open-source license is granted by this cleanup; the repository does not currently contain a standalone software license. Contact the author about reuse and cite the dissertation when referring to this research.

## Research foundations

- Place Pulse 2.0: Dubey et al., *Deep Learning the City: Quantifying Urban Perception at a Global Scale*, ECCV 2016.
- SegFormer: Xie et al., *SegFormer: Simple and Efficient Design for Semantic Segmentation with Transformers*, NeurIPS 2021.
- Space Syntax: Hillier and Hanson, *The Social Logic of Space*, 1984.
