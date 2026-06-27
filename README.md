Explainable ML for Kidney Dysfunction Prediction in Diabetic Foot Ulcer Patients

This repository holds the analysis code behind a study on predicting kidney dysfunction (KD) in diabetic foot ulcer (DFU) inpatients without relying on creatinine or urea — the two lab values that normally drive this diagnosis but are often delayed, missing, or simply not back from the lab yet when a patient is admitted.

The core question: how much of that diagnostic signal can be recovered from things a hospital already has on day one — CBC, electrolytes, inflammatory markers, comorbidities — and how much of the remaining gap does urea alone close once it comes back?

Background

The cohort is 734 DFU inpatients from Ardabil University of Medical Sciences (ethics approval IR.ARUMS.REC.1403.207), with kidney dysfunction present in roughly 44% of patients. Two prediction scenarios were built on the same engineered feature set:

- Scenario A — creatinine- and urea-free (26 features: CBC, derived inflammatory ratios, electrolytes, Wagner grade, heart disease)
- Scenario B — same as A, plus urea (27 features), still without creatinine

Both are benchmarked against a urea-only ablation model (to isolate exactly what urea contributes on its own) and a creatinine-only baseline (the conventional reference point). Everything is validated internally with 10-fold cross-validation, bootstrapped confidence intervals, and a chronological hold-out, then re-evaluated on a separate, genuinely external cohort of 62 patients that never touched model development.

What's in here

Three notebooks, each covering one stage of the analysis. They were written and run in Google Colab, so paths default to `/content/...` — change `DATA_PATH` (and the equivalent variables in the other two notebooks) if running elsewhere.

Scenario_A_and_B.ipynb
The main pipeline. Loads the development cohort, builds the engineered features (NLR, PLR, MLR, ELR, SII, AISI, HI, the potassium-to-sodium ratio, and an age-adjusted anemia score), then trains and compares seven classifiers — logistic regression, random forest, gradient boosting, XGBoost, LightGBM, a calibrated SVM, and an MLP — across both scenarios. Cross-validation, SMOTE (applied only inside the training folds, never on the held-out fold), bootstrap AUC confidence intervals, DeLong pairwise comparisons, SHAP, permutation importance, calibration curves, and decision curve analysis all come out of this one notebook, ending in a single combined PDF report. If the source Excel file isn't found at `DATA_PATH`, it falls back to a small synthetic dataset so the pipeline can still be run end to end — mainly there so anyone without access to the real patient data can still check that the code works.

Urea_Ablation_Analysis.ipynb
A deliberately minimal companion notebook: trains LightGBM on urea alone (5-fold CV) to see how much of Scenario B's performance comes from that single value versus everything else combined. There's also a short exploratory cell near the top for checking column names and target/urea candidates in a new file — that's leftover from when column naming wasn't yet consistent across spreadsheets, kept in because it's still occasionally useful.

External_validation.ipynb
Takes the models trained on the development cohort and scores them, unmodified, on a separate external case series. No retraining, no recalibration — the point is to see how AUC and calibration hold up outside the population the models were built on. Expects two files: the original development spreadsheet and a second external-cohort file.

Data

The patient-level Excel files aren't included in this repository. The dataset comes from a retrospective, IRB-approved cohort and isn't mine to redistribute — anyone needing access for replication should go through the corresponding author and the institution's data governance process, not GitHub.

If you just want to confirm the code runs, `Scenario_A_and_B.ipynb` will generate a synthetic stand-in automatically when it can't find the real file. It won't reproduce the actual numbers, but it exercises the full pipeline — feature engineering, all seven models, SHAP, the works.

Method summary

- Feature engineering: the standard inflammatory ratios (NLR/PLR/MLR/ELR/SII/AISI), a WBC-to-hemoglobin index, a potassium-to-sodium ratio, and an age-adjusted anemia score, all winsorized at the 99.5th percentile.
- Seven models per scenario, 10-fold stratified CV, SMOTE inside folds only, standardization fit per fold (never on the full dataset, to avoid leakage).
- Internal validation: pooled out-of-fold AUC/AUPRC/Brier/MCC/F1, 200-iteration bootstrap confidence intervals, chronological 80/20 temporal hold-out.
- Explainability: SHAP (TreeExplainer for the tree-based models, a gradient-boosted surrogate for SVM/MLP) plus permutation importance, checked against each other rather than trusted in isolation.
- Clinical utility: Youden-optimal thresholds, full sensitivity/specificity/PPV/NPV grids across thresholds, decision curve analysis.
- External validation on an independent 62-patient cohort, with no retraining.

For context, headline numbers: Scenario A lands around AUC 0.71 internally with no renal marker at all; adding urea (Scenario B) pushes that to roughly 0.85; a urea-only model gets to almost the same place by itself, which is the main point of the ablation — most of Scenario B's gain comes from urea, not the other 26 features. External AUCs are lower (around 0.61 and 0.80) and calibration outside the development cohort isn't great yet — recalibration would be needed before any of this is near clinically usable.

Running this

Built for Colab, so a few things need adjusting to run locally:
- Swap the `/content/...` paths for wherever your files actually live.
- Remove or guard the `from google.colab import files` import in the ablation notebook if you're not in Colab.
- You'll need: pandas, numpy, scikit-learn, imbalanced-learn, xgboost, lightgbm, shap, scipy, statsmodels, matplotlib, seaborn.

The random seed is fixed at 42 throughout, so reruns on the same input data should give you the same folds and the same bootstrap samples.

Status

This is the analysis code behind a manuscript currently in revision (dual-scenario development, the urea ablation, and external validation, all in one paper). The numbers in these notebooks match the out-of-fold and external results reported in that draft. If you're working on something related — DFU outcomes, creatinine-free CKD screening, anything in that space — feel free to open an issue, I'd be glad to compare notes.

Reuse

No formal license attached yet. If you want to build on this or reuse parts of the pipeline, open an issue or reach out first — this is tied to an unpublished study, so I'd rather know it's being used than have it float around unattributed.
