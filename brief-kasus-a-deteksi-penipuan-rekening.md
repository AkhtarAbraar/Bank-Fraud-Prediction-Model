# Brief Project: Kasus A - Deteksi Penipuan Pembukaan Rekening Bank

## 1. Konteks

Sebuah bank digital menerima ribuan pengajuan buka rekening online setiap hari. Sebagian kecil diajukan oleh penipu, misalnya memakai identitas palsu atau untuk menampung uang hasil kejahatan. Tim risiko ingin setiap pengajuan baru diberi skor: seberapa mungkin pengajuan tersebut penipuan.

## 2. Tujuan

Menangkap sebanyak mungkin penipu, dengan syarat nasabah jujur yang ikut tertandai paling banyak 5%.

Setiap nasabah jujur yang tertandai berarti calon pelanggan hilang dan tambahan kerja verifikasi manual.

## 3. Metrik Utama

**TPR pada FPR 5%**: persentase penipu yang tertangkap saat nasabah jujur yang salah tertandai dibatasi 5%.

- TPR (True Positive Rate) = penipu yang tertangkap / total penipu
- FPR (False Positive Rate) = nasabah jujur yang tertandai / total nasabah jujur
- Angka ini dilaporkan pada **data validasi**.

## 4. Data

| Item | Keterangan |
|---|---|
| Dataset | Bank Account Fraud (BAF), NeurIPS 2022 |
| Varian | **Base** saja |
| Subset | Baris dengan kolom `month` bernilai **0-3** |
| Unit data | Satu baris = satu pengajuan rekening |
| Label | Satu kolom label penipuan |
| Sumber unduh | https://www.kaggle.com/datasets/sgpjesus/bank-account-fraud-dataset-neurips-2022 |
| Penjelasan kolom | Datasheet di https://github.com/feedzai/bank-account-fraud |

## 5. Aturan

- Metode bebas, asal bisa dilatih dan dijalankan.
- Cara membagi data latih dan validasi ditentukan sendiri, **lalu alasannya ditulis**.
- Aturan berlaku untuk kedua kasus (Kasus A dan Kasus B).

## 6. Yang Dikumpulkan

Link GitHub Repository yang berisi:

1. **Notebook** yang bisa dijalankan dari atas ke bawah tanpa error.
2. **File model** yang sudah disimpan dan bisa dimuat ulang.
3. **README** maksimal 1 halaman, memuat:
   - metode yang dipakai
   - alasan memilih metode tersebut
   - cara membagi data
   - skor di data validasi (TPR pada FPR 5%)
4. **requirements.txt**

## 7. Checklist Pengerjaan

- [ ] Unduh dataset Base dari Kaggle, filter `month` 0-3
- [ ] Baca datasheet untuk memahami kolom (terutama kolom dengan nilai hilang dan kolom sensitif)
- [ ] EDA: proporsi penipuan, nilai hilang, distribusi fitur
- [ ] Tentukan strategi split data dan catat alasannya
- [ ] Bangun baseline, lalu model utama
- [ ] Hitung TPR pada FPR 5% di data validasi
- [ ] Simpan model dan uji muat ulang di sesi baru
- [ ] Pastikan notebook jalan bersih dari atas ke bawah (Restart & Run All)
- [ ] Tulis README (maks. 1 halaman)
- [ ] Buat `requirements.txt` dengan versi paket
- [ ] Push ke GitHub dan cek ulang repo dari clone baru

## 8. Catatan Teknis (saran, bukan bagian dari soal)

Verifikasi detailnya di datasheet resmi sebelum dipakai.

- **Kelas sangat tidak seimbang**: penipuan hanya sebagian kecil dari data. Pertimbangkan class weight atau `scale_pos_weight`, dan jangan pakai akurasi sebagai ukuran.
- **Split data**: karena ada kolom `month`, pembagian berdasarkan waktu (misalnya latih di bulan 0-2, validasi di bulan 3) meniru kondisi nyata, yaitu model memberi skor pada pengajuan yang lebih baru. Alternatifnya stratified split acak, yang lebih stabil tetapi kurang realistis. Pilih satu dan jelaskan alasannya.
- **Kolom `month`**: sebaiknya tidak dipakai sebagai fitur.
- **Nilai hilang**: beberapa kolom di BAF memakai kode khusus (misalnya -1) untuk nilai hilang. Cek di datasheet.
- **Menghitung TPR pada FPR 5%**: ambil dari kurva ROC (`sklearn.metrics.roc_curve`), yaitu TPR pada titik terakhir yang FPR-nya masih ≤ 0.05.
- **Model yang umum dipakai untuk data tabular**: gradient boosting (LightGBM, XGBoost, CatBoost) biasanya kuat sebagai baseline.
- **Kolom sensitif**: BAF memuat atribut seperti usia. Pertimbangkan dampaknya terhadap keadilan model dan sebutkan singkat di README bila relevan.
