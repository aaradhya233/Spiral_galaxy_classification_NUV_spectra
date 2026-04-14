# Galaxy Bar Classification — ISRO Space Astronomy Group

Research project completed during internship at the Space Astronomy Group (SAG), URSC-ISRO.

**Task:** Binary classification of spiral galaxies (barred vs unbarred) from GALEX NUV FITS images using morphological feature extraction and classical ML classifiers.

---

## Repository Structure

```
galaxy-bar-classification/
├── notebooks/
│   ├── 00_chromeautomate.ipynb          — download GALEX NUV FITS images from NASA SkyView
│   ├── 01_testtrainsplit.ipynb          — stratified 80/20 split of 584 FITS files
│   ├── 02_testtrainlabel.ipynb          — morphological type string → binary label
│   ├── 03_preprocess.ipynb              — normalisation + 8× augmentation
│   ├── 04_cropped.ipynb                 — adaptive galaxy cutout + resize to 224×224
│   ├── 05_FeatureExtraction.ipynb       — 22 photometric features (train + test)
│   ├── 06_SVM.ipynb                     — RBF-SVM classifier
│   ├── 06_newfeatureextraction.ipynb    — centre-focused bar features (V5 pipeline only)
│   ├── 07_ForestClassify.ipynb          — Random Forest V1–V5
│   └── 08_confusionmatriximages.ipynb   — visual TP/TN/FP/FN galaxy grid audit
│
├── results/
│   ├── automatic_galaxy_detection_&_cropping/
│   ├── normalization_techniques_comparison/
│   ├── RF_version1_results/
│   ├── RF_version2_results/
│   ├── RF_version3_results/
│   ├── RF_version4_results/
│   ├── RF_version5_results/             — best model outputs
│   └── SVM_results/
│
├── README.md
├── requirements.txt
├── .gitignore
├── trainlabel.csv                       — ground truth labels (training set)
├── testlabel.csv                        — ground truth labels (test set)
├── train_galaxy_features_improved.csv   — final V5 feature set (training)
└── test_galaxy_features_improved.csv    — final V5 feature set (test)
```

---

## How to Run

> Steps 0–4 are shared. After that the pipeline splits into two tracks depending on which model you want to reproduce. Run **Track A** for RF V1–V4 and SVM, run **Track B** for RF V5 (best model).

### Shared Steps (run first, always)

| Step | Notebook | What it does |
|------|----------|-------------|
| 0 | `00_chromeautomate` | Automates GALEX NUV FITS downloads from NASA SkyView using a coordinates CSV. Requires ChromeDriver. Produces a `fits/` folder. |
| 1 | `01_testtrainsplit` | Splits 584 FITS files 80/20 stratified (random_state=42). Produces `train/`, `test/`, `train.csv`, `test.csv`. |
| 2 | `02_testtrainlabel` | Parses NED morphological type strings (SB/SAB → 1, SA → 0). Produces `trainlabel.csv`, `testlabel.csv`. |
| 3 | `03_preprocess` | Min-max normalisation (1–99th percentile clip) then 8× augmentation per image (4 rotations × horizontal flip). |
| 4 | `04_cropped` | Detects galaxy extent via radial flux profile + background sigma-clipping. Cuts out the galaxy and resizes to 224×224 using OpenCV. 96.4% success rate on the full dataset. |

---

### Track A — RF V1 to V4 and SVM

Use this track to reproduce the baseline through V4 results, or the SVM.

| Step | Notebook | What it does |
|------|----------|-------------|
| 5 | `05_FeatureExtraction` | Extracts 22 photometric features per galaxy. **Run Cell 1** on the `train/` folder → `galaxy_features_train.csv`. **Run Cell 2** on the `test/` folder → `galaxy_features_test.csv`. |
| 6 | `06_SVM` | RBF-SVM with GridSearchCV over C and gamma. Uses `galaxy_features_train/test.csv` from Step 5. Includes permutation feature importance. |
| 7 | `07_ForestClassify` | Random Forest versions 1–4, each in a separate cell. All use `galaxy_features_train/test.csv` from Step 5. Run cells in order: **Cell 1** = V1 (baseline), **Cell 2** = V2 (reweighted features), **Cell 3** = V3 (enhanced RF), **Cell 4** = V4 (32 engineered features + optimised hyperparameters). |

---

### Track B — RF V5 ★ Best Model

Use this track to reproduce the best result (70.09% accuracy, AUC-ROC ~0.77).

| Step | Notebook | What it does |
|------|----------|-------------|
| 5 | `05_FeatureExtraction` | Same as Track A — run Cell 1 (train) and Cell 2 (test) if not already done. |
| 6 | `06_newfeatureextraction` | Extracts centre-focused bar features targeting the inner 30–40% of galactic radius where bars actually exist. **Run Cell 2 only** (Cell 1 is an earlier simple-center attempt, superseded). Produces `train_galaxy_features_improved.csv` and `test_galaxy_features_improved.csv`. |
| 7 | `07_ForestClassify` | **Run Cell 0 only** (titled "CENTER-FOCUSED BAR DETECTION"). Uses `train/test_galaxy_features_improved.csv` from Step 6. |

---

### After either track

| Step | Notebook | What it does |
|------|----------|-------------|
| 8 | `08_confusionmatriximages` | Loads FITS files for each prediction category (TP/TN/FP/FN) and produces 2×2 visual galaxy grids for manual inspection. Point it at the predictions CSV output from whichever model you ran. |

---

## Features Extracted (22 total)

**Standard morphology:** DA, DV, Asymmetry (90°/180°), Clumpiness, Concentration, R50, R90, Gini, Ellipticity, Petrosian Radius

**Bar-specific:** Bar-Bulge Ratio (central elongation), Bar Fourier Strength (m=2 azimuthal mode), Central Concentration, Spiral Arm Strength, Angular Momentum Proxy

**Unbarred-specific:** Nuclear Concentration Ratio, Spiral Symmetry Score, Bar Ansae Test, Edge-On Indicator

**Derived:** Radial Profile Ratio (R90/R50), Effective Radius

---

## Results Summary

| Model | Accuracy | Unbarred Recall | Barred Recall | AUC-ROC |
|-------|----------|-----------------|---------------|---------|
| SVM (RBF) | 56.41% | 39.58% | 68.12% | 0.583 |
| RF Version 1 — Baseline | 60.68% | 43.75% | 72.46% | ~0.648 |
| RF Version 2 — Reweighted features | 64.96% | 45.83% | 78.26% | ~0.677 |
| RF Version 3 — Enhanced RF | 64.10% | 47.92% | 75.36% | ~0.663 |
| RF Version 4 — 32 engineered features | 68.38% | 54.17% | 78.26% | ~0.681 |
| **RF Version 5 — Centre-focused ★** | **70.09%** | **68.75%** | **71.01%** | **~0.77** |

**Key finding:** Adding unbarred-specific features (Nuclear Concentration Ratio, Bar Ansae Test, Spiral Symmetry Score) targeting the central galactic region improved minority class (unbarred) recall by **25 percentage points** from baseline to final model, while keeping barred recall stable.

**Top 3 discriminative features (permutation importance):**
1. Nuclear Concentration Ratio — 9.32%
2. Gini Coefficient — 8.88%
3. Bar Ansae Test — 8.59%

---

## Data

- **Images:** 584 GALEX NUV FITS images from NASA SkyView / MAST (not included due to file size)
- **Download:** Use `00_chromeautomate.ipynb` with your own `coordinates.csv`, or fetch directly from [mast.stsci.edu](https://mast.stsci.edu)
- **Labels:** Derived from NED / HyperLEDA morphological type strings
- **Split:** 467 training / 117 test (stratified, random_state=42)
- **Class distribution:** Barred 53.9% / Unbarred 45.7% / Unknown 0.4%

---

## Stack

Python 3.11, Astropy, Scikit-learn, Scikit-image, OpenCV, SciPy, NumPy, Matplotlib, Seaborn, Pandas, Selenium

---

## Author

**Aaradhya Sahu**  
Internship: Space Astronomy Group, URSC-ISRO  
GitHub: [github.com/aaradhya233](https://github.com/aaradhya233)
