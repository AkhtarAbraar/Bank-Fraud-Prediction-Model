# Kasus A: Deteksi Penipuan Pembukaan Rekening

Model klasifikasi untuk mendeteksi pengajuan rekening fraud pada dataset **Bank Account Fraud (BAF), Base variant**, dibatasi ke `month` 0–3. Target utama: **TPR pada FPR ≤ 5%**.

## Metode

Empat keluarga model dibandingkan: Logistic Regression (baseline linear), LightGBM, XGBoost, dan CatBoost, masing-masing dengan pencarian hyperparameter kecil (dua kandidat). Logistic Regression memakai median imputation + scaling untuk fitur numerik, serta most-frequent imputation + one-hot encoding untuk fitur kategorikal. Model tree memakai class weighting untuk kelas minoritas (fraud sekitar 1%) dan penanganan kategorikal serta nilai hilang bawaan pustakanya. `month` dan `device_fraud_count` (konstan) tidak dipakai sebagai fitur.

**Alasan memilih XGBoost sebagai model tersimpan**: pada pemilihan bulan 2, XGBoost memberi TPR@FPR5% tertinggi (53,42%; imbang dengan CatBoost 53,42% tetapi unggul ROC AUC 0,8866 vs 0,8837; LightGBM 52,42%; Logistic Regression 47,33%). Histogram tree menangani 260 ribu baris latih, enam kolom kategorikal dengan native categorical split, dan nilai hilang tanpa imputasi terpisah. Bulan 3 tidak memengaruhi pilihan ini.

## Pembagian data

Evaluasi tiga tahap berurutan waktu: fit kandidat di bulan 0–1 (260.060 baris, fraud 1,037%), pilih hyperparameter dan decision threshold di bulan 2 (136.979 baris, 0,875%), lalu laporkan metrik akhir sekali di bulan 3 (150.936 baris, 0,922%) yang tidak tersentuh. Pembagian ini meniru kondisi nyata: model memberi skor pada pengajuan yang lebih baru. Bulan 3 dikeluarkan dari EDA, kosakata kategori, fit preprocessing, tuning, penentuan threshold, dan pemilihan model.

## Hasil validasi

Threshold dipilih di bulan 2 pada batas FPR 5%, lalu dibekukan sebelum evaluasi bulan 3. Metrik bulan 3 memakai threshold beku tersebut, bukan threshold baru dari data uji.

| Model               | TPR@FPR5% bulan 2 | AUC bulan 3 | TPR@FPR5% bulan 3 | TPR bulan 3 (threshold beku) | FPR bulan 3 | Precision bulan 3 |
| ------------------- | ----------------: | ----------: | ----------------: | ---------------------------: | ----------: | ----------------: |
| Logistic Regression |            47.33% |      0.8898 |            51.15% |                       62.50% |       8.08% |             6.72% |
| LightGBM            |            52.42% |      0.9078 |            55.46% |                       67.24% |       8.79% |             6.65% |
| XGBoost             |            53.42% |      0.9134 |            57.83% |                       68.68% |       8.76% |             6.80% |
| CatBoost            |            53.42% |      0.9120 |            57.18% |                       68.53% |       8.65% |             6.87% |

FPR bulan 3 melampaui 5% untuk semua model, menunjukkan drift temporal. XGBoost mempertahankan AUC dan TPR tertinggi di holdout, dengan FPR setara model tree lain; CatBoost punya precision tertinggi. Pemilihan XGBoost hanya berdasarkan bulan 2; holdout mengonfirmasi, bukan menentukan.

## Catatan

`customer_age` berpotensi menjadi atribut sensitif. Hasil ini bukan evaluasi fairness dan tidak membuktikan ketiadaan dampak berbeda antar kelompok. Dataset BAF mencatat `-1` sebagai nilai hilang pada enam fitur tertentu; notebook mengubah sentinel tersebut menjadi NaN sebelum preprocessing.

## Glosarium istilah

**Metrik klasifikasi (label positif = fraud)**

| Istilah                    | Arti                                                                                                                                             | Rumus                    |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------ |
| TP (true positive)         | Fraud yang ditandai model.                                                                                                                       | —                        |
| FP (false positive)        | Non-fraud yang ikut ditandai. Biaya: review manual tambahan.                                                                                     | —                        |
| TN (true negative)         | Non-fraud yang tidak ditandai.                                                                                                                   | —                        |
| FN (false negative)        | Fraud yang lolos. Biaya paling mahal.                                                                                                            | —                        |
| TPR / Recall / Sensitivity | Porsi fraud yang berhasil ditangkap.                                                                                                             | TP / (TP + FN)           |
| FPR                        | Porsi non-fraud yang salah ditandai.                                                                                                             | FP / (FP + TN)           |
| TNR / Specificity          | Porsi non-fraud yang benar tidak ditandai.                                                                                                       | TN / (FP + TN) = 1 − FPR |
| FNR                        | Porsi fraud yang lolos.                                                                                                                          | FN / (TP + FN) = 1 − TPR |
| Precision                  | Porsi dari yang ditandai yang benar-benar fraud.                                                                                                 | TP / (TP + FP)           |
| Accuracy                   | Porsi prediksi benar keseluruhan. **Tidak dipakai di sini** karena fraud hanya ~1%, sehingga nilai akurasi tetap tinggi walau semua fraud lolos. | (TP + TN) / total        |

**Metrik dan konsep evaluasi**

| Istilah            | Deskripsi                                                                                                                                                           |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ROC curve          | Kurva TPR terhadap FPR di semua nilai threshold.                                                                                                                    |
| ROC AUC            | Luas di bawah kurva ROC (0,5 = acak, 1,0 = sempurna). Ringkasan kemampuan ranking, tidak bergantung threshold. Dipakai sebagai diagnostik sekunder dan tie-break.   |
| TPR@FPR5%          | TPR tertinggi yang bisa dicapai pada saat FPR ≤ 5%. Metrik target proyek ini: seberapa banyak fraud tertangkap pada batas false positive yang bisnis masih toleran. |
| Decision threshold | Nilai ambang probabilitas; skor ≥ threshold berarti ditandai fraud. Dipilih di bulan 2 pada batas FPR 5%, lalu dibekukan.                                           |
| Confusion matrix   | Tabel 2×2 berisi TP, FP, TN, FN pada threshold tertentu.                                                                                                            |

**Data dan protokol**

| Istilah                                                                          | Deskripsi                                                                                                                     |
| -------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| BAF (Bank Account Fraud)                                                         | Dataset pembukaan rekening berlabel fraud; variant `Base` dipakai di proyek ini.                                              |
| Sentinel `-1`                                                                    | Penanda nilai hilang pada enam fitur tertentu dalam datasheet BAF; diubah menjadi NaN. Nilai negatif di kolom lain tetap sah. |
| Fit months                                                                       | Bulan 0–1, dipakai untuk melatih kandidat model dan kosakata kategori.                                                        |
| Tuning month                                                                     | Bulan 2, dipakai memilih hyperparameter, model, dan decision threshold.                                                       |
| Holdout month                                                                    | Bulan 3, hanya dipakai sekali untuk pelaporan metrik akhir; tidak memengaruhi pilihan apa pun.                                |
| Drift temporal                                                                   | Pergeseran pola fraud antar bulan, sehingga FPR bulan 3 melampaui 5% walau threshold tidak diubah.                            |
| Class imbalance                                                                  | Kelas fraud sangat minoritas (~1%).                                                                                           |
| `class_weight="balanced"` / `scale_pos_weight` / `auto_class_weights="Balanced"` | Pembobotan kelas agar model lebih memperhatikan kelas fraud minoritas.                                                        |
| Native categorical handling                                                      | Kemampuan LightGBM/XGBoost/CatBoost memproses kolom kategorikal langsung tanpa one-hot encoding.                              |
| One-hot encoding                                                                 | Mengubah kategori menjadi kolom biner 0/1; dipakai hanya pada baseline Logistic Regression.                                   |

## Jalankan

Letakkan `Base.csv` di `data/Base.csv`, install paket dari `requirements.txt`, lalu jalankan notebook dari atas ke bawah. Notebook melatih dan menyimpan `model.pkl`. Untuk prediksi XGBoost, ubah input kategorikal ke pandas categorical dtype memakai kosakata kategori dan urutan kolom dari training; pola casting ada di sel XGBoost notebook.
