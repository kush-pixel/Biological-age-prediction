Data expectations

DNAm matrix (training & evaluation):

Orientation can be CpGs × samples or samples × CpGs.

Values should be β in [0,1] after preprocessing.

Metadata (per sample): must include an age column. If available, sex, ethnicity, and tissue are used for descriptive checks and subgroup metrics.

External cohort: features may not perfectly match the training set; the notebook uses ColumnAligner to reindex to the training schema and fill missing CpGs with training means.

How to run (by cell number)

Cell 0 — Imports & config
All libraries (NumPy, pandas, matplotlib, SciPy, scikit-learn) and plotting config. Sets a random SEED for reproducibility.

Cell 1 — Load external evaluation cohort (GSE157131)
Loads DNAm matrix + metadata (kept strictly for evaluation).

Cell 2 — Evaluation sanity
Shapes, missingness, and age histogram. Expect near-zero missingness; mid-to-older age concentration is normal.

Cell 3 — Peek evaluation DNAm
Quick head to confirm IDs and matrix orientation before alignment.

Cell 4 — Load training cohort
Loads whole-blood training DNAm + metadata (the set used for fitting).

Cell 6 — Training coverage
Age bins, sex/ethnicity counts, and age histogram. Densest in midlife → smallest errors there, larger at the tails.

Cell 7 — β-value distribution & missingness (train)
β values bounded in [0,1] with expected shape; ~0% high-missing CpGs → inputs are clean.

Cell 8 — PCA colored by age
Smooth age gradient across PCs → age is a leading signal; no dominant batch islands.

Cell 9 — CpG–age correlations (Spearman |ρ|)
Many probes show strong monotonic trends → consistent with clock biology (descriptive, not inferential).

Cell 11 — Single-model benchmarks (test split)

Ridge (reduced CpGs) — Linear + L2; best single model on test.

PCR — PCs by variance; can miss predictive directions.

PLS — Components maximizing covariance with age; decent but behind Ridge here.

Random Forest — Nonlinear; lags in ultra-wide, collinear CpGs.
Residuals vs age show mild regression-to-mean (calibration slope < 1). Bin-MAE clarifies tail errors.

Cell 13 — Stacked ensemble (Ridge + PLS)
Out-of-fold stacking blends complementary errors → small but real lift over Ridge; similar, mild calibration compression; small subgroup MAE gaps.

Cell 14 — ColumnAligner
Reorders evaluation CpGs to training schema and fills missing CpGs with training means → prevents silent misalignment across studies.

Cell 17 — External cohort evaluation (GSE157131)
No retraining; align + score. Ridge and Stack remain strong; PLS degrades. Same gentle calibration compression as internal → effect from cohort age distribution, not overfitting.

Models in brief

Ridge regression (with reduced CpG panel): linear, L2-regularized; handles p≫n and collinearity well; workhorse clock here.

PCR (Principal Components Regression): regress on top variance PCs; simple but can discard predictive low-variance directions.

PLS (Partial Least Squares): supervised components that covary with age; tuning-sensitive; behind Ridge in this setup.

Random Forest: nonlinear, interaction-capable; tends to underperform on ultra-wide, highly correlated CpGs without heavy feature engineering.

Stacked ensemble: OOF blending of Ridge + PLS for a modest accuracy gain without new bias patterns.

Results (typical from this notebook)
| Dataset / Model                    | RMSE (yrs) |  MAE (yrs) |          R² / r | Notes                                               |
| ---------------------------------- | ---------: | ---------: | --------------: | --------------------------------------------------- |
| **Test split** — Ridge             | \~**4.39** | \~**3.27** | **R² \~ 0.925** | Best single model; slope < 1 (mild mean regression) |
| **Test split** — Stack (Ridge+PLS) | \~**4.26** | \~**3.16** | **R² \~ 0.929** | Small but real lift                                 |
| **External (GSE157131)** — Ridge   | \~**4.00** |          — |   **r \~ 0.94** | Strong generalization after alignment               |
| **External (GSE157131)** — Stack   | \~**4.00** |          — |   **r \~ 0.94** | Matches Ridge externally                            |
| **External (GSE157131)** — PLS     | \~**6.82** |          — |   **r \~ 0.72** | Degrades more off-study                             |


Numbers may vary slightly across runs; patterns are stable.

Calibration & fairness

Calibration slope < 1 (both internal & external) → slight over-prediction at young ages and under-prediction at old ages (driven by age distributions).

Subgroup MAE (e.g., by sex) shows small gaps in this run; continue to monitor if deployment skews.

Add your own data

Prepare:

train_dnam (β matrix), train_meta (must include age)

eval_dnam, eval_meta (for external cohort)

Update file paths in Cells 1 & 4.

Ensure age units are years.

Run the notebook; ColumnAligner will handle feature order for the external set.

Troubleshooting

KeyError when subsetting DNAm: orientation mismatch (samples vs CpGs).

Use the orientation-robust EDA snippets (already included) and rely on ColumnAligner before evaluation.

Worse-than-expected external metrics: verify that alignment ran, and check the age range mismatch; expect larger tail errors if your external set has more extremes.

Memory issues: reduce CpG subset sizes for EDA plots (e.g., 20k probes), and prefer float32 where safe.

Reproducibility

Global SEED = 42 (NumPy + random) is set in Cell 0.

KFold/shuffles in scikit-learn use the same seed.

License & attribution

Replace this with your chosen license.

External cohort evaluated here: GSE157131 (public DNAm dataset).

Built with NumPy, pandas, matplotlib, SciPy, and scikit-learn.
