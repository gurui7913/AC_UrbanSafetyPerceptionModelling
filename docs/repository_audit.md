# Repository and dissertation archive audit

Audit date: 2026-10-08. Scope: folder inventory, notebook version comparison, static code review, selected modelling-table integrity checks, and independent reconstruction of the three reported model fits. This is not a full scientific peer review or an end-to-end rerun.

## Findings and actions

| Finding | Evidence | Action / status |
| --- | --- | --- |
| Duplicate notebook copies | Seven notebooks have matching hashes across `03_Codes/`, `03_Codes/MyCode/`, and the submission code folder; GitHub cell sources are identical too | Canonical Git checkout established; original copies retained as historical material |
| README entry point was incorrect | V1 fits only two spatial predictors; V2 fits combined features and three variants | Rewritten notebook guide |
| README described four fitted models | V2 fits baseline, degree-2 polynomial, and all six pairwise interactions | Corrected to three fitted models; conceptual Formula notebook retained |
| README visual definitions disagreed with code | Code uses tree, grass, plant and multiplies ratios by 100 | Corrected classes and percentage units |
| Results were overinterpreted | Model comparison uses in-sample fitted values; cross-validation is imported but not executed | README now states exploratory associations and validation limits |
| Absolute machine paths | All computational notebooks reference the original Windows dissertation folders | Documented required path edits; source code preserved |
| Stale preparation path | Score-preparation root uses an older Chinese directory spelling | Documented in DATA.md; correction required before raw rerun |
| Missing merge implementation | V2 reads `merged_data.csv`; no preparation cell writes that file | Documented validated join contract; existing merged values checked |
| Geographic nearest-neighbor join | Location notebook cell 15 converts lines to EPSG:4326 and calls `sjoin_nearest` | Unresolved research issue; recompute in a suitable projected CRS before claiming a corrected dataset |
| Ties silently discarded | Same cell drops duplicate location IDs after the nearest join | Unresolved; define a stable tie policy and preserve matching diagnostics |
| Stored out-of-order failure | V2 cell 13 had a saved `NameError` for `original_r2`, which is assigned in earlier cell 10 | Outputs cleared; no claim that full sequential execution was tested |
| Office lock accidentally tracked | `~$scores.tsv` was in the preprocessing folder | Removed from active repository; local copy retained; ignore rule added |
| Unstructured CSV exports | Source data and outputs were intermingled with notebooks | Existing tables moved to `data/legacy/` and `results/legacy/` |
| No environment lock / standalone license | Original repo contained neither | Added inferred dependency lists; original environment and reuse terms remain unresolved |

The CRS finding follows the [GeoPandas nearest-join documentation](https://geopandas.org/en/stable/docs/reference/api/geopandas.sjoin_nearest.html): distance calculations use CRS units and geographic CRS distances are inaccurate. A suitable projected London CRS, such as EPSG:27700 after checking both input CRSs, should be used for distance matching; export coordinates separately in WGS84. This may change street assignments and downstream results, so the archived computation was not silently altered during organization.

## Verified in this audit

- Seven corresponding notebooks have identical source cells in the remote repository and original local code folders.
- Every code cell parses as Python without a syntax error.
- Spatial, segmentation, and merged inputs each have 900 rows and 900 unique location IDs, identical ID sets, and no empty CSV fields.
- Every spatial and visual numeric value in the merged table matches its corresponding source row.
- Independent NumPy least-squares fits, using standardized predictors and the exact interaction construction, reproduce all three exported R² values within `1e-10`: 0.0174048703, 0.0808961481, and 0.0347636577.

After organization, notebook schema, source preservation, relative documentation links, and Git whitespace/content checks are verified before publication.

## Verification limits

No raw-data preprocessing, GIS rematching, model download, GPU inference, or full notebook execution was performed. Dependency installation was not tested as a complete environment. The independent fit check validates model calculations on the existing local CSV, not feature provenance, semantic accuracy, geographic matching, causal validity, or out-of-sample performance. Notebook markdown retains historical interpretations; use the corrected README and this audit when assessing claims.

## Organization policy

The public Git checkout is the active code copy. The surrounding dissertation directory remains the archival home of source data, thesis versions, drawings, and submitted supplementary material. Its local root README provides navigation. Original historical folders remain intact to preserve references and submission provenance; they should not be edited as the active project.

No raw images, new location-level inputs, thesis drafts, or discussion files were added to Git. Existing public data exports remain traceable through Git history. Clearing notebook outputs does not remove their previous versions from Git history.

## Follow-up: current GitHub tree cleanup

On 2026-10-08, the user requested removal of previously uploaded residual files from the current GitHub tree, with all local files retained and no history rewrite. The four CSV exports under `data/legacy/` and `results/legacy/` were untracked and the folders added to `.gitignore`. Their local SHA-256 hashes were checked before and after untracking. The code notebooks and aggregate `results/model_comparison_results.csv` remain tracked. This follow-up supersedes the earlier policy of retaining legacy exports in the public working tree; their historical Git versions remain available.

## Follow-up: consistent repository naming

On 2026-10-08, the seven notebooks were grouped under `notebooks/01_data_preparation/` and `notebooks/02_model_analysis/`, with numbered lowercase English names. Documentation and the full dependency filename were also normalized to underscores. The complete old-to-new mapping is in [file_rename_map.csv](file_rename_map.csv). Notebook files were renamed without changing their bytes or source cells. The source manifest continues to point to the unchanged original local snapshots, while its active notebook paths use the new names. README links and local interview-preparation links were updated. Legacy CSV files remain local and ignored.
