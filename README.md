# Dynamic ROI-constrained UAV multispectral estimation of single-plant LAI in trellis-trained kiwifruit

Public reference code and data-access information for the study accepted in **Computers and Electronics in Agriculture**:

> **Dynamic ROI-constrained UAV multispectral estimation of single-plant LAI in trellis-trained kiwifruit**  
> Wenjie Li, Hongen Liu, Linghuan Ouyang, Qian Chen, Tianqi Lv, Jiali Li, Xintao Lin, Chen Yao, Yongqiang Zheng, Jianping Qian

## What this repository provides

This repository provides a **cleaned reference implementation of the key algorithmic workflow** used in the paper. It is intended to help readers understand, inspect, adapt, and compare the method rather than serve as a frozen archive of every project file generated during manuscript development.

The public release includes:

- pseudo-invariant-feature (PIF) relative radiometric calibration;
- RTK/tree-anchor-guided dynamic single-plant ROI extraction;
- local spectral purity screening with parameter **Q**;
- percentile-based spatial boundary control with parameter **P**;
- vegetation-index and NIR texture feature extraction;
- temporal-window analysis;
- Top-K feature selection;
- ROI-strategy comparison;
- grouped cross-validation model benchmarking;
- orchard-scale LAI mapping;
- anonymized/illustrative input templates;
- an external link to the large UAV multispectral imagery archive.

The key dynamic ROI setting reported in the paper is **P90_Q70**. The public code also exposes the main boundary parameters used in the study: initial exploration radius = **2.0 m**, `Rmin = 0.2 m`, `Rmax = 2.6 m`, fallback radius = **1.5 m**, and minimum valid vegetation pixels = **30**.

## Related patent application

The core adaptive single-plant canopy ROI method associated with this repository is also the subject of a **Chinese invention patent application (patent pending)**:

- **Title:** 棚架果园单株冠层 ROI 的自适应提取方法及装置  
  *Adaptive extraction method and device for single-plant canopy ROI in trellis orchards*
- **Application number:** **202610722903.3**
- **Filing date:** **May 25, 2026**
- **Applicant:** Institute of Agricultural Resources and Regional Planning, Chinese Academy of Agricultural Sciences
- **Current status:** The application was accepted by CNIPA on May 25, 2026 and **passed preliminary examination on June 2, 2026**. Following approval of the request for early publication, it entered the **publication preparation procedure**. A request for substantive examination was also included among the filing documents received by CNIPA. The application is currently pending and has **not yet been granted**.

Official CNIPA notices documenting the above status are included in the [`patent/`](patent/) directory. See [`patent/README.md`](patent/README.md) for details.

> **Patent notice:** The source code is released for scientific communication, academic reference, method comparison, and research collaboration. Public availability of this reference implementation does not imply transfer or waiver of patent rights. Please contact the authors regarding commercial use or technology transfer.

## Repository structure

```text
.
├── README.md
├── requirements.txt
├── USAGE_NOTICE.md
├── CITATION.cff
├── code/
│   ├── 00_fit_relative_radiometric_calibration.py
│   ├── 01_dynamic_roi_extraction.py
│   ├── 02_extract_vi_texture_features.py
│   ├── 03_temporal_window_ablation.py
│   ├── 04_feature_selection_topk.py
│   ├── 05_roi_strategy_ablation.py
│   ├── 06_model_benchmark_groupcv.py
│   └── 07_orchard_lai_mapping.py
├── examples/
│   ├── example_samples_anonymized.csv
│   ├── example_imagery_config.csv
│   ├── paper_imagery_config_template.csv
│   ├── example_pif_calibration_template.csv
│   └── example_roi_strategy_config.csv
├── data_access/
│   └── UAV_DATA.md
├── patent/
│   ├── README.md
│   ├── CN2026107229033_Acceptance_Notice_2026-05-25.pdf
│   └── CN2026107229033_Preliminary_Examination_Passed_2026-06-02.pdf
└── docs/
    ├── METHOD_WORKFLOW.md
    └── REPOSITORY_SCOPE.md
```

## Data availability

### UAV multispectral imagery

The UAV orthomosaics are approximately **6 GB** and are not stored directly in the GitHub repository.

**Quark Cloud:** https://pan.quark.cn/s/da72cf8f8f82

The archive is named `UAV-data.zip` and contains the May, August, and September orchard imagery. Each monthly folder includes the four Mavic 3 Multispectral bands used in the study: **Green, Red, RedEdge, and NIR**.

More details are provided in [`data_access/UAV_DATA.md`](data_access/UAV_DATA.md).

### Ground LAI measurements and additional data

The complete ground-measured LAI dataset and project-specific intermediate files are not bundled in this public repository. The CSV files in `examples/` are anonymized/illustrative templates intended to document input formats.

For additional research data, methodological questions, or collaboration, please contact:

**Dr. Wenjie Li**  
**Email: 17395844880@163.com**

## Reproducibility scope

The code is released primarily for **method transparency and academic reference**. It documents the key processing logic and parameterization used in the paper, but the example files alone are not intended to exactly reproduce every numerical result, table, or figure in the article.

Exact numerical reproduction may require the complete ground-reference measurements, original project-specific intermediate tables, the full UAV data, and a closely matched software environment. See [`docs/REPOSITORY_SCOPE.md`](docs/REPOSITORY_SCOPE.md) for details.

## Installation

Clone or download the repository, create a Python environment, and install the dependencies:

```bash
pip install -r requirements.txt
```

`xgboost` is optional and is used only when the XGBoost model is included in the model benchmark.

## Input files

### 1. Sample-tree table

A CSV or XLSX table with at least:

- `Tree_ID` or `ID`: anonymized tree identifier;
- `X`/`Y` or `E`/`N`: projected tree-center coordinates;
- `Date`: acquisition date;
- `LAI_Observed`: measured LAI, required for model development but not for ROI delineation alone.

See `examples/example_samples_anonymized.csv`.

### 2. Imagery configuration

A CSV table containing:

- `Date`;
- `Green`, `Red`, `RedEdge`, `NIR`: paths to co-registered single-band orthomosaics;
- optional band-wise calibration coefficients `*_a` and `*_b`, where:

```text
reflectance = a * DN + b
```

`examples/paper_imagery_config_template.csv` contains the band-wise calibration coefficients reported in the paper. Replace the raster paths with the locations of your downloaded imagery.

### 3. Optional PIF calibration table

For independent datasets, `code/00_fit_relative_radiometric_calibration.py` can fit band-wise relative calibration coefficients from a table containing:

- `Date`;
- `Band`;
- `PIF`;
- `DN`;
- `TargetReflectance`.

See `examples/example_pif_calibration_template.csv`.

## Typical workflow

### Step 0. Relative radiometric calibration (optional if coefficients are already available)

```bash
python code/00_fit_relative_radiometric_calibration.py \
  --input examples/example_pif_calibration_template.csv \
  --output outputs/calibration/calibration_coefficients.csv
```

### Step 1. Dynamic single-plant ROI extraction

```bash
python code/01_dynamic_roi_extraction.py \
  --samples examples/example_samples_anonymized.csv \
  --imagery-config examples/paper_imagery_config_template.csv \
  --output-dir outputs/dynamic_roi \
  --sample-crs EPSG:4546 \
  --p 90 \
  --q 70
```

### Step 2. Spectral and NIR texture feature extraction

```bash
python code/02_extract_vi_texture_features.py \
  --samples examples/example_samples_anonymized.csv \
  --imagery-config examples/paper_imagery_config_template.csv \
  --output outputs/features/single_plant_lai_features.csv \
  --sample-crs EPSG:4546 \
  --p 90 \
  --q 70
```

### Step 3. Temporal-window analysis

```bash
python code/03_temporal_window_ablation.py \
  --input outputs/features/single_plant_lai_features.csv \
  --output-dir outputs/temporal_ablation
```

### Step 4. Feature selection

```bash
python code/04_feature_selection_topk.py \
  --input outputs/features/single_plant_lai_features.csv \
  --output-dir outputs/feature_selection \
  --optimal-k 29
```

### Step 5. ROI-strategy comparison

`code/05_roi_strategy_ablation.py` compares **precomputed feature tables** generated under alternative ROI definitions. Edit `examples/example_roi_strategy_config.csv` so each row points to the corresponding feature table.

```bash
python code/05_roi_strategy_ablation.py \
  --strategy-config examples/example_roi_strategy_config.csv \
  --output-dir outputs/roi_ablation \
  --top-k 29
```

### Step 6. Machine-learning model comparison

```bash
python code/06_model_benchmark_groupcv.py \
  --input outputs/features/single_plant_lai_features.csv \
  --features outputs/feature_selection/selected_features_topK.txt \
  --output-dir outputs/model_benchmark
```

When `Tree_ID` is present, the script uses grouped cross-validation to keep repeated observations from the same tree in the same fold.

### Step 7. Orchard-scale LAI mapping

```bash
python code/07_orchard_lai_mapping.py \
  --feature-table outputs/features/single_plant_lai_features.csv \
  --features outputs/feature_selection/selected_features_topK.txt \
  --imagery-config examples/paper_imagery_config_template.csv \
  --date 2024-09-28 \
  --output-dir outputs/lai_mapping
```

The default GBDT parameters in the mapping script match the optimal configuration reported in the paper. The wall-to-wall texture layers are efficient moving-window approximations used for mapping and are not identical to the sample-level ROI GLCM calculation.

## Notes for reuse

- The dynamic ROI workflow assumes known or pre-established tree-center anchors from RTK, a tree inventory, row interpolation, or manual/semi-automatic correction.
- The method is designed to define an effective single-plant observation unit under trellis overlap; it is not a fully unsupervised tree-detection algorithm.
- Replace the anonymized coordinates in the example sample table with your own projected coordinates before running the spatial scripts.
- Large raster products, derived outputs, and model files are ignored by `.gitignore` and should not be committed unless intentionally required.

## Citation

If you use or adapt this code, please cite the associated paper. The final DOI can be added to this README and `CITATION.cff` after the article is formally assigned its publication metadata.

## Contact

For data requests, technical questions, or collaboration:

**Dr. Wenjie Li**  
**17395844880@163.com**
