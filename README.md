# LOBICO Drug Sensitivity Analysis

This repository contains the data and scripts to perform logic-based modeling of drug sensitivity (LOBICO) using cell line genetic features and drug response measurements.

---

## Contents

### Python Scripts

* **`threshold.py`** – Compute binarization thresholds for drug responses following Knijnenburg et al. (2016).

  * Reads AUC values and their 95% confidence intervals from `AUC_upper_lower_bounds.xlsx`.
  * Upsamples AUCs and computes thresholds (`theta` and `b`) for defining sensitive vs resistant cell lines.
  * Generates plots of the distributions and thresholds.
  * Outputs threshold tables:

    * `AUC*_t0.1_log10.csv` – threshold values per drug.
    * `Samples*_t0.1_log10.csv` – summary of cell lines classified as sensitive/resistant.

* **Dependencies:** `numpy`, `pandas`, `scipy`, `matplotlib`, `sklearn`.

---

### MATLAB Scripts

1. **`LOBICO_crossval.m`** – LOBICO cross-validation pipeline

   * Loads AUC and mutation data.
   * Applies binarization threshold (from `threshold.py`) to define sensitive/resistant cell lines.
   * Performs nested cross-validation:

     * Outer loop: training-test split to evaluate performance.
     * Inner loop: k-fold cross-validation for hyperparameter tuning (model complexity `K` and `M`).
   * Uses CPLEX solver to infer Boolean logic models.
   * Outputs validation and testing results including confusion matrices, performance metrics, and inferred formulas.

2. **`balanced_crossval.m`** – Balanced k-fold cross-validation function

   * Splits samples into `k` folds, balancing sensitive vs resistant cell lines.
   * Returns testing and training indices for cross-validation.

3. **`parsexls2.m`** – Excel parser for drug and mutation data

   * Reads cell line names, AUC values, mutation matrix, and feature names from Excel sheets.
   * Used by `LOBICO_crossval.m` to load input data.

---

### Data Files

* **`Supplementary Table 5 LOBICO data for paper.xlsx`** – contains mutation matrices and drug AUCs for each cell line.
* **`AUC_upper_lower_bounds.xlsx`** – contains AUC values and their 95% confidence interval bounds for threshold computation.

---

### Workflow

1. **Compute thresholds**

   ```bash
   python threshold.py
   ```

   Generates threshold tables and distribution plots for each drug.

2. **Run LOBICO cross-validation**
   Open MATLAB and run:

   ```matlab
   LOBICO_crossval
   ```

   * Uses threshold tables from `threshold.py`.
   * Performs nested cross-validation and outputs performance metrics and inferred Boolean formulas.

---

### Python-to-MATLAB Workflow Diagram

```
AUC_upper_lower_bounds.xlsx   Supplementary Table 5 LOBICO data for paper.xlsx
                 │                         │
                 ▼                         ▼
          threshold.py  ──────────────►  threshold tables
                 │                             │
                 ▼                             ▼
             thresholds.csv                LOBICO_crossval.m
                 │                             │
                 └────────────► run LOBICO ──┘
```

* Python computes thresholds for sensitive/resistant classification.
* MATLAB uses thresholds to perform nested cross-validation and infer Boolean logic models.

---

### Notes

* The Python and MATLAB pipelines are tightly linked: thresholds computed in Python are required for MATLAB modeling.
* Random seeds are set for reproducibility.
* Plots generated in `threshold.py` show the upsampled AUC distributions, KDE, thresholds, and estimated variance.
* LOBICO requires a valid CPLEX installation for solving integer programming problems.
Short guide as to how figures & tables related to the threshold & LOBICO analyses in Chang et al. 2023 were created

---

Fig. 1E & Fig. 4A: run threshold.py (creates various image file outputs of the form AUC_f?_t0.1_log10.pdf)

Fig. 4B: run threshold.py (creates text file output Samples_t0.1_log10.csv from which Fig. 4B can be created)

Fig. 5B: 
1) Make sure LOBICO & Matlab are installed
2) Run threshold.py (creates AUC_t0.1_log10.csv which contains the binarization threshold)
3) Run LOBICO_crossval.m (runs LOBICO and outputs Validation_*.csv and Testing_*.csv files)
4) run plot_ROC_cross.py (creates pdf images)

Fig. 5C
1) run steps 1)-3) from Fig. 5B
2) run tables_ROC_cross.py (creates ROC_formulae_test_t0.1_log10_limit4.csv); file contains the Pareto optimal formulae (according to the test set) for each drug and training-test data split
3) run gene_importance.py (creates file gene_importance_t0.1_log10_limit4_ew.csv which provides the imporance (weight) for each gene)
4) run plot_gene_importance.py (creates the pdf image)

---

Supplementary Fig. 2: same as for Fig. 4B

Supplementary Fig. 3: same as for Fig. 5B

---

Supplementary Table 3: run threshold.py (creates text file AUC_t0.1_log10.csv from which Suppl. Table 3 can be created)

Supplementary Table 4: run threshold.py and look at text file output Samples_t0.1_log10.csv

Supplementary Table 6: Supplementary Table 6 corresponds to file ROC_formulae_test_t0.1_log10_limit4.csv, see discussion of Fig. 5C above

Supplementary Table 7: Supplementary Table 7 corresponds to file gene_importance_t0.1_log10_limit4_ew.csv, see discussion of Fig. 5C above
