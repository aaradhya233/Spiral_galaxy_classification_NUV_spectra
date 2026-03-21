# Galaxy Bar Classification — ISRO Space Astronomy Group

Research project completed during internship at the Space Astronomy Group (SAG), URSC-ISRO.

**Task:** Binary classification of spiral galaxies (barred vs unbarred) from GALEX NUV FITS images using morphological feature extraction and classical ML classifiers.

---

## Repository Structure

```
galaxy-bar-classification/
├── notebooks/
│   ├── 01_testtrainsplit.ipynb          — stratified 80/20 split of FITS files
│   ├── 02_testtrainlabel.ipynb          — morphological type string → binary label
│   ├── 03_preprocess.ipynb              — min-max normalisation + 8× augmentation
│   ├── 04_cropped.ipynb                 — adaptive galaxy cutout + resize to 224×224
│   ├── 05_FeatureExtraction_train.ipynb — 22 photometric features (training set)
│   ├── 05b_FeatureExtraction_test.ipynb — 22 photometric features (test set)
│   ├── 06_newfeatureextraction.ipynb    — centre-focused bar features (V2 improved)
│   ├── 07_ForestClassify.ipynb          — Random Forest with weighting + threshold tuning
│   ├── 08_SVM.ipynb                     — RBF-SVM with GridSearchCV
│   └── 09_confusionmatriximages.ipynb   — visual TP/TN/FP/FN galaxy grid audit
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
├── trainlabel.csv                       — ground truth labels (training)
├── testlabel.csv                        — ground truth labels (test)
├── train_galaxy_features_improved.csv   — final V5 feature set (training)
└── test_galaxy_features_improved.csv    — final V5 feature set (test)
```

---

## Pipeline — Run Notebooks in Order

| Step | Notebook | Description |
|------|----------|-------------|
| 1 | `01_testtrainsplit` | Split 584 FITS files 80/20 stratified; save `train.csv` / `test.csv` |
| 2 | `02_testtrainlabel` | Parse NED morphological type strings → binary label (SB/SAB=1, SA=0) |
| 3 | `03_preprocess` | Min-max normalisation; 8× augmentation (4 rotations × horizontal flip) |
| 4 | `04_cropped` | Adaptive galaxy detection via radial profile analysis; intelligent cutout; resize to 224×224 (96.4% success rate) |
| 5 | `05/05b_FeatureExtraction` | Extract 22 photometric features per galaxy for train and test sets |
| 6 | `06_newfeatureextraction` | Centre-focused bar feature extraction — targets inner 30–40% of galactic radius where bars actually exist |
| 7 | `07_ForestClassify` | Random Forest with feature weighting, 7-fold CV, optimal threshold search |
| 8 | `08_SVM` | RBF-SVM with GridSearchCV + permutation feature importance |
| 9 | `09_confusionmatriximages` | Load FITS files per TP/TN/FP/FN category; produce 2×2 visual grids for manual audit |

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
| CNN + Ring Features | 66.67% | 18.75% | 100% | — |
| RF Version 1 (Baseline) | 60.68% | 43.75% | 72.46% | ~0.648 |
| RF Version 2 (Weighted features) | 64.96% | 45.83% | 78.26% | ~0.677 |
| RF Version 3 (22 features) | 64.10% | 47.92% | 75.36% | ~0.663 |
| RF Version 4 (32 engineered features) | 68.38% | 54.17% | 78.26% | ~0.681 |
| **RF Version 5 (Centre-focused) ★** | **70.09%** | **68.75%** | **71.01%** | **~0.77** |

**Key finding:** Adding unbarred-specific features (Nuclear_Concentration_Ratio, Bar_Ansae_Test, Spiral_Symmetry_Score) targeting the central galactic region improved minority class (unbarred) recall by **25 percentage points** from baseline to final model.

**Top 3 discriminative features (from feature importance analysis):**
1. Nuclear Concentration Ratio — 9.32%
2. Gini Coefficient — 8.88%
3. Bar Ansae Test — 8.59%

---

## Data

- **Images:** 584 GALEX NUV FITS images from NASA MAST public archive (not included due to file size)
- **Download:** [mast.stsci.edu](https://mast.stsci.edu)
- **Labels:** Derived from NED / HyperLEDA morphological type strings (SB/SAB = barred, SA = unbarred)
- **Split:** 467 training / 117 test (stratified)
- **Class distribution:** Barred 53.9% / Unbarred 45.7% / Unknown 0.4%

---

## Stack

Python 3.11, Astropy, Scikit-learn, Scikit-image, OpenCV, SciPy, NumPy, Matplotlib, Seaborn, Pandas

---

## Author

**Aaradhya Sahu**  
Internship: Space Astronomy Group, URSC-ISRO, Oct–Nov 2025  
GitHub: [github.com/aaradhya233](https://github.com/aaradhya233)
