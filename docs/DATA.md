# Data requirements and provenance

Paths below are relative to the local dissertation root (`08_Dissertation/`), not to the Git checkout. The original notebooks currently contain absolute Windows paths. Replace every input and output path before running elsewhere.

## Required local inputs

| Stage | Existing local input | Required fields / role |
| --- | --- | --- |
| Score preparation | `01_DataCollection/01 Place Pluse2.0 Dataset/00 place-pulse-dataset-images _OriginalDatasdet/` | `locations.tsv`, `places.tsv`, `qscores.tsv`, `studies.tsv`; later exploratory cells also read `votes.tsv` |
| London network | `02_DataProcessing/01_SpaceSyntax_London/London.gpkg` | Street line geometries with `INT2K`, `CH2K`; verify source CRS |
| Safety subset | `02_DataProcessing/00_ExtractData_PP/PP_Safer_Trueskill.csv` | Filtered and normalized safer dimension |
| Image segmentation | Original dataset `images/`, then `02_DataProcessing/02_ImageSegmentation/Selected_Images/` | Image filename stems correspond to `location_id` |
| Spatial-only model | `02_DataProcessing/01_SpaceSyntax_London/London_RegressionModel_wgs84.csv` | `location_id, lon, lat, safer_Trueskill_Scores, INT2K, CH2K` |
| Visual features | `02_DataProcessing/02_ImageSegmentation/green_view_ratio_and_sky_visibility_results.csv` | `location_id, green_view_ratio, sky_visibility` plus pixel counts and image metadata |
| Combined model | `02_DataProcessing/03_Multi-Linesr_Regression_Model/merged_data.csv` | `location_id, lon, lat, safer_Trueskill_Scores, green_view_ratio, sky_visibility, INT2K, CH2K` |

The score-preparation notebook uses an older Chinese spelling of the dataset directory; its root path does not match the current English directory name. Correct it explicitly. `London.gpkg` / `london.gpkg` also require consistent case on case-sensitive systems.

## Manual combined-table step

The archived code reads `merged_data.csv` but contains no code that creates it. To prepare it, join the spatial and segmentation tables on the underlying `location_id`, assert unique IDs on each side, check unmatched IDs, and retain the eight columns shown above. Do not join by row order.

The audit found 900 unique IDs in each of the three local input tables, identical ID sets, no empty fields, and exact numeric agreement of the merged spatial and visual columns with their source tables. This check does not certify the accuracy of the upstream extraction.

## What is included

- `data/legacy/safer_min12.csv`: previously tracked global safer-dimension export, moved from the preprocessing folder. It has 36,783 rows and is not the 900-row London modelling table.
- `results/model_comparison_results.csv`: aggregate three-model comparison copied from the existing local export.
- `results/legacy/`: descriptive statistics and extreme-case exports already present in the remote repository. The two summary files belong to different contexts and are kept separately.

## What stays local

Raw street-view images, raw TSV datasets, GIS layers, segmentation image folders, location-level modelling tables not previously tracked, thesis drafts, and discussion notes. The repository does not provide a full dataset download or grant rights to third-party data. Retain provider metadata and verify redistribution terms before any future dataset release.

## Sources identified in the archive

- Place Pulse 2.0 data and its original metadata files.
- Space Syntax Open Mapping GB v1, with provider metadata under the original collection folder.
- SegFormer-B0 ADE20K checkpoint: [model configuration](https://huggingface.co/nvidia/segformer-b0-finetuned-ade-512-512/blob/main/config.json).

The original software environment, exact source download dates, and model revision were not recorded in a reproducible lockfile.
