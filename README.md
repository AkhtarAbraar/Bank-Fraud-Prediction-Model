# Kasus A: Deteksi Penipuan Pembukaan Rekening

Model klasifikasi untuk mendeteksi pengajuan rekening fraud pada dataset **Bank Account Fraud (BAF), Base variant**. Data dibatasi ke `month` 0–3. Target utama: **TPR pada FPR ≤ 5%**.

## Metode

Compared Logistic Regression, LightGBM, XGBoost, and CatBoost. Each model family receives a small hyperparameter search. Logistic Regression uses median imputation and scaling for numeric features, plus most-frequent imputation and one-hot encoding for categorical features. The tree models use class weighting and their supported categorical and missing-value handling. XGBoost is the saved model: it wins the month 2 tuning comparison (TPR 53.42% vs 52.42% for LightGBM), and month 3 never influenced that choice.

## Pembagian data

Chronological three-stage evaluation: fit candidate models on months 0 and 1, select hyperparameters and decision thresholds on month 2, and report final metrics once on untouched month 3. Preprocessing, category vocabularies, and threshold decisions exclude month 3. The test set is not used to choose the saved model.

## Hasil validasi

Thresholds were selected on month 2 with the target FPR limit, then frozen before month 3 evaluation. Month 3 metrics use those fixed thresholds, not new test-set thresholds.
| Model | Month 2 TPR at FPR 5% | Month 3 ROC AUC | Month 3 TPR | Month 3 FPR | Month 3 precision |
|---|---:|---:|---:|---:|---:|
| Logistic Regression | 47.33% | 0.8898 | 62.50% | 8.08% | 6.72% |
| LightGBM | 52.42% | 0.9078 | 67.24% | 8.79% | 6.65% |
| XGBoost | 53.42% | 0.9134 | 68.68% | 8.76% | 6.80% |
| CatBoost | 53.42% | 0.9120 | 68.53% | 8.65% | 6.87% |

Month 3 FPR exceeds five percent for every model, showing the effect of temporal drift. XGBoost has the highest holdout AUC, TPR, and FPR stays comparable to the other tree models, while CatBoost has the highest precision. XGBoost was selected on month 2 only; the holdout confirmed but did not drive the choice.

## Catatan

`customer_age` berpotensi menjadi atribut sensitif. Hasil ini bukan evaluasi fairness dan tidak membuktikan ketiadaan dampak berbeda antar kelompok. Dataset BAF mencatat `-1` sebagai nilai hilang pada enam fitur tertentu; notebook mengubah sentinel tersebut menjadi NaN.

## Jalankan

Place `Base.csv` at `data/Base.csv`, install packages from `requirements.txt`, then run the notebook from top to bottom. The notebook trains and exports `model.pkl`. For XGBoost predictions, cast categorical inputs to the pandas categorical dtype using the training category vocabulary and column order shown in the notebook.
