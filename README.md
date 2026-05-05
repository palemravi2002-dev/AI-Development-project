# Lumbar Spine Curvature Analysis Pipeline
### Normal vs. Achondroplasia — Automated Radiograph Analysis

---

## Overview

This project implements a fully automated, classical computer-vision pipeline for quantitative analysis of lumbar spine radiographs. It compares spinal curvature, vertebral morphology, and local texture features between a **normal** cohort and an **achondroplasia** cohort, producing per-image diagnostic reports and a full cohort dashboard all without requiring deep-learning infrastructure or manual annotation beyond a single ROI definition.

---

## Pipeline Stages

| # | Stage | Description |
|---|---|---|
| 1 | Dataset loading | Scans `data/normal/` and `data/achondroplasia/` for `.png`, `.jpg`, `.jpeg` |
| 2 | ROI definition | Draw once interactively or use default; cached to `output/roi.json` |
| 3 | Grayscale preprocessing | CLAHE contrast enhancement + Gaussian denoising + polarity check |
| 4 | Segmentation | Otsu threshold + morphological closing/opening + connected components |
| 5 | Skeletonisation | Zhang–Suen thinning to medial axis |
| 6 | Spline fitting | Cubic UnivariateSpline on centerline skeleton points |
| 7 | Curvature estimation | Frenet–Serret curvature κ and Cobb angle proxy |
| 8 | Severity grading | Normal / Mild / Moderate / Severe (Cobb angle bands) |
| 9 | Zoning | L1–L5 vertebral label assignment |
| 10 | SIFT | Scale-Invariant Feature Transform keypoint extraction |
| 11 | ORB | Oriented FAST and Rotated BRIEF keypoint extraction |
| 12 | Density heatmap | Class-stratified spatial keypoint density maps |
| 13 | Statistical comparison | Mann–Whitney U + Welch t-test across cohorts |
| 14 | Dashboard | Full cohort 5×4 panel composite figure |
---

## Project Structure

```
project/
│
├── data/
│   ├── normal/                 
│   └── achondroplasia/         
│
├── output/
│   ├── roi.json                 
│   ├── results.csv              
│   ├── statistics.csv           
│   ├── preprocessing.png
│   ├── segmentation.png
│   ├── skeletonization.png
│   ├── spline_fitting.png
│   ├── severity_grading.png
│   ├── zoning.png
│   ├── sift_features.png
│   ├── orb_features.png
│   ├── density_heatmap.png
│   ├── statistics.png
│   ├── dashboard.png
│   
│       
│
└── spine_analysis.ipynb
```

---

## Requirements

### Python Version
Python **3.9** or higher is required.

### Dependencies

Install all dependencies with:

```bash
pip install numpy scipy scikit-image opencv-python matplotlib pandas seaborn pillow notebook
```

| Package | Minimum Version | Purpose |
|---|---|---|
| `numpy` | 1.24 | Array computation |
| `scipy` | 1.10 | Spline fitting, statistics |
| `scikit-image` | 0.21 | Segmentation, skeletonisation, morphology |
| `opencv-python` | 4.8 | Image I/O, CLAHE, SIFT, ORB |
| `matplotlib` | 3.7 | Plotting, interactive ROI widget |
| `pandas` | 2.0 | Results tables, CSV export |
| `seaborn` | 0.12 | Statistical violin/box plots |
| `pillow` | 9.0 | Additional image format support |
| `notebook` | 6.5 | Jupyter Notebook execution environment |

### Hardware
- **CPU:** Any modern multi-core processor (no GPU required)
- **RAM:** ≥ 8 GB recommended for cohorts of 76+ high-resolution images
- **Storage:** ≥ 500 MB for images, notebook, and all outputs

---

## Quick Start

### Step 1 — Prepare your data

```
data/
├── normal/
│   ├── spine01.png
│   ├── spine02.jpg
│   └── ...
└── achondroplasia/
    ├── achron-001.jpg
    ├── achron-002.jpg
    └── ...
```

Supported formats: `.png`, `.jpg`, `.jpeg` (case-insensitive).

> **Image polarity:** The pipeline expects **bright anatomy on a dark background**
> (standard X-ray convention). If your images are inverted, the preprocessing
> step detects and corrects this automatically using a median-intensity check.

---

### Step 2 — Launch the notebook

```bash
jupyter notebook spine_analysis.ipynb
```

Then run all cells: **Kernel → Restart & Run All**

---

### Step 3 — ROI definition (Cell 5)

The ROI covers the **lumbar spine only (L1–L5)**.

| Environment | Behaviour |
|---|---|
| Interactive (Qt / Tk backend) | A window opens on the first image — drag to draw the lumbar rectangle, then close the window |
| Headless / server / VS Code | Default ROI is applied automatically: `x=0.20, y=0.40, w=0.60, h=0.55` (fractional) |

To force interactive mode, add this to Cell 1:
```python
%matplotlib qt   # or: %matplotlib tk
```

The ROI is saved to `output/roi.json` after the first run and reused on all subsequent runs. To redraw, delete `output/roi.json` and re-run Cell 5.

---

### Step 4 — Review outputs

All outputs are written to the `output/` directory automatically.

| File | Description |
|---|---|
| `output/results.csv` | Per-image: Cobb angle, grade, zone angles, curvature, SIFT/ORB counts |
| `output/statistics.csv` | Group comparison: means, Mann–Whitney p-values, significance flags |
| `output/dashboard.png` | Full cohort composite dashboard |
| `output/reports/report_<name>.png` | Single-image diagnostic report for every image |

---

## Outputs Explained

### Per-image Results (`results.csv`)

| Column | Description |
|---|---|
| `path` | Full file path |
| `label` | `normal` or `achondroplasia` |
| `cobb_deg` | Estimated Cobb angle (degrees) |
| `grade` | Severity: Normal / Mild / Moderate / Severe |
| `n_vertebrae` | Number of vertebrae detected (max 5) |
| `mean_curv` | Mean centerline curvature κ |
| `sift_n` | Number of SIFT keypoints detected |
| `orb_n` | Number of ORB keypoints detected |
| `angle_L1` … `angle_L5` | Endplate orientation per lumbar zone (degrees) |

### Severity Grading (Cobb Angle)

| Grade | Cobb Angle Range | Clinical Implication |
|---|---|---|
| Normal | 0° – 10° | Within physiological range |
| Mild | 10° – 25° | Monitoring recommended |
| Moderate | 25° – 40° | Bracing may be indicated |
| Severe | ≥ 40° | Surgical consultation advised |

### Diagnostic Report Panels

| Panel | Content |
|---|---|
| A | Preprocessed lumbar ROI (denoised grayscale) |
| B | Segmentation overlay (green = detected vertebrae) |
| C | Centerline spline (orange) overlaid on ROI |
| D | Lumbar zones L1–L5 with colour-coded bounding boxes |
| E | Curvature profile κ(s) along the spine |
| F | Cobb angle severity gauge |
| G | SIFT keypoints (top 100, cyan) |
| H | ORB keypoints (top 100, orange) |
| I | Text summary: file, class, Cobb angle, grade, zone angles, feature counts |

---

## Demo Mode

If **no images** are found in `data/normal/` or `data/achondroplasia/`, the pipeline automatically enters **Demo Mode**:

- Generates 3 synthetic normal X-rays (lordosis 15°–21°)
- Generates 3 synthetic achondroplastic X-rays (lordosis 35°–45°)
- Saves them into the data directories
- Runs the full pipeline on the synthetic images

This allows you to verify the pipeline is working correctly before loading real data.

---

## Reproducing Results

- Global random seed: `np.random.seed(42)` (Cell 1)
- ROI coordinates are cached in `output/roi.json` — identical crop applied on every run
- All processing steps are deterministic given fixed inputs and a fixed ROI

To fully reset and re-run from scratch:
```bash
rm -rf output/
jupyter nbconvert --to notebook --execute spine_analysis_pipeline.ipynb
```

---

## Known Limitations

- **Cobb angle accuracy:** The endplate-orientation method (Method A) can produce inflated angles when vertebral bodies appear heavily tilted or when the image orientation deviates from a true lateral projection. Ground-truth landmark annotation would improve accuracy.
- **Zone detection:** In severely deformed or low-contrast images, fewer than 5 vertebrae may be detected. The pipeline labels only what is found (e.g., L1–L3 if only 3 components pass the size filter).
- **No classifier:** The current version is a quantitative characterisation tool. A supervised classifier (SVM, random forest) trained on the output feature vectors is a recommended next step.
- **Single ROI for all images:** The fractional ROI is designed to generalise across resolutions but may need adjustment if images have very different field-of-view or patient positioning.

---

## Citation

If you use this pipeline in your work, please cite the following foundational methods:

- Otsu, N. (1979). A threshold selection method from gray-level histograms. *IEEE Transactions on Systems, Man, and Cybernetics*, 9(1), 62–66.
- Lowe, D.G. (2004). Distinctive image features from scale-invariant keypoints. *International Journal of Computer Vision*, 60(2), 91–110.
- Rublee, E. et al. (2011). ORB: An efficient alternative to SIFT or SURF. *ICCV 2011*, 2564–2571.
- Cobb, J.R. (1948). Outline for the study of scoliosis. *AAOS Instructional Course Lectures*, 5, 261–275.
- Zhang, R. (2025). A state-of-the-art survey of deep learning for lumbar spine image analysis. *AI Medicine*, 1(1), 3.

---

## License

This project is released for academic and research use. Not validated for clinical diagnosis. Always consult a qualified medical professional for clinical interpretation of spinal radiographs.

---

*Last updated: May 2026*
