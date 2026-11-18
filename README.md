# Kasus A — Deteksi Penipuan Pembukaan Rekening

Model klasifikasi untuk mendeteksi pengajuan rekening fraud pada dataset **Bank Account Fraud (BAF), Base variant**. Data dibatasi ke `month` 0–3. Target utama: **TPR pada FPR ≤ 5%**.

## Metode

Dibandingkan Logistic Regression (median imputation + scaling untuk fitur numerik, most-frequent imputation + one-hot encoding untuk fitur kategorikal) dengan LightGBM. Model utama LightGBM memakai `scale_pos_weight` untuk menangani ketidakseimbangan kelas dan mendukung nilai hilang. `month` tidak dipakai sebagai fitur; `device_fraud_count` dikeluarkan karena konstan.

## Pembagian data

Pembagian berbasis waktu: latih pada bulan 0–2 (397.039 pengajuan) dan validasi pada bulan 3 (150.936 pengajuan). Split ini meniru penilaian pengajuan yang lebih baru, bukan split acak yang dapat mencampur periode waktu. Validasi berisi 1.392 fraud.

## Hasil validasi

LightGBM mencapai **TPR@FPR5% = 52,23%**, pada **FPR = 4,9818%** (threshold 0,801168). ROC AUC diagnostik: Logistic Regression 0,8913; LightGBM 0,8992. TPR dihitung pada titik ROC terakhir yang memenuhi FPR ≤ 5%, tanpa interpolasi.

## Catatan

`customer_age` berpotensi menjadi atribut sensitif. Hasil ini bukan evaluasi fairness dan tidak membuktikan ketiadaan dampak berbeda antar kelompok. Dataset BAF mencatat `-1` sebagai nilai hilang pada enam fitur tertentu; notebook mengubah sentinel tersebut menjadi NaN.

## Jalankan

Simpan `Base.csv` di `data/Base.csv`, install paket dari `requirements.txt`, lalu jalankan notebook dari atas ke bawah. Notebook menghasilkan `model.pkl`. Untuk prediksi LightGBM, kategorikal harus diubah ke pandas category memakai kategori dan urutan yang dipakai saat training; gunakan pola casting dalam sel LightGBM notebook.
