# Reproduksi Model Penyisihan — Tim datascape
## PeDaS 2026 | Babak Final
**Macro F1-Score Leaderboard:** 0.975287141

---

## 1. Ringkasan & Kepatuhan Batas Ukuran ZIP (10 MB)

Sesuai ketentuan Petunjuk Teknis Babak Final PeDaS 2026, berkas paket reproduksi yang diunggah ke formulir memiliki **batasan ukuran maksimal 10 MB**.

Arsitektur model tim **datascape** menggunakan *quad-branch ensemble* (20 base learners: 5 fold × [XGBoost + LightGBM + CatBoost + NLP TF-IDF Calibrated LinearSVC]) dengan total ukuran bobot tersimpan mencapai **~83 MB**, sehingga melebihi batas 10 MB jika seluruh bobot fisik disertakan langsung di dalam arsip ZIP.

### Solusi Transparan & Otomatis:
1. Seluruh bobot model tersimpan (`models/`) telah dipublikasikan secara terbuka pada mirror resmi GitHub:  
   👉 **https://github.com/ZakyFauzi/pedas2026-reproduksi-penyisihan**
2. Notebook `datascape_ReproduksiPenyisihan.ipynb` telah dilengkapi modul **Auto-Download Cerdas & Paralel** pada Sel 2.
3. Saat dewan juri membuka notebook dan menekan tombol **Run All**, notebook secara otomatis mendeteksi ketiadaan bobot model lokal dan langsung mengunduh seluruh 20 model secara multi-threaded (5 thread paralel via Fastly CDN GitHub) dalam waktu **~20–25 detik**.
4. Tidak diperlukan instalasi CLI tambahan (`curl`, `wget`, `gdown`, `unzip` ditiadakan; murni memanfaatkan pustaka standar Python `urllib` & `concurrent.futures`), sehingga dijamin **100% cross-platform** (Windows, Linux, macOS).
5. Paket ZIP yang dikumpulkan berukuran **hanya ~75 KB** (< 0.1 MB), sangat jauh di bawah batas 10 MB juknis panitia.

---

## 2. Struktur Direktori Paket Pengumpulan

```
pedas2026-reproduksi-penyisihan/
├── datascape_ReproduksiPenyisihan.ipynb  # Notebook utama — Jalankan "Run All"
├── predict_ready.csv                    # Data uji siap pakai (1.500 baris, 10 kolom asli)
├── requirements.txt                     # Daftar dependensi Python
├── README.md                            # Dokumentasi teknis & panduan eksekusi
├── weights/
│   ├── opt_weights.npy                  # Bobot blending optimal Nelder-Mead (160 bytes)
│   └── best_alphas.npy                  # Multiplier threshold kalibrasi per-kelas (184 bytes)
└── models/                              # Folder bobot model (diunduh otomatis saat Run All)
    ├── le_label.pkl                     # LabelEncoder (545 bytes)
    ├── xgb_fold0.pkl ... xgb_fold4.pkl  # 5 model XGBoost (~4.2 MB per fold)
    ├── lgb_fold0.pkl ... lgb_fold4.pkl  # 5 model LightGBM (~5.0 MB per fold)
    ├── cat_fold0.pkl ... cat_fold4.pkl  # 5 model CatBoost (~1.7 MB per fold)
    └── nlp_fold0.pkl ... nlp_fold4.pkl  # 5 model Calibrated LinearSVC (~5.9 MB per fold)
```

---

## 3. Cara Menjalankan (Zero-Configuration Run All)

### Opsi A: Eksekusi Langsung Paket ZIP (Rekomendasi Utama)
1. Ekstrak file ZIP `reproduksi-penyisihan-datascape.zip`.
2. Pasang dependensi yang dibutuhkan:
   ```bash
   pip install -r requirements.txt
   ```
3. Buka `datascape_ReproduksiPenyisihan.ipynb` pada Jupyter Notebook, JupyterLab, atau VS Code.
4. Pilih menu **Kernel → Restart & Run All**.
5. Notebook akan:
   - Mengunduh otomatis 20 model dari GitHub mirror (~25 detik).
   - Melakukan feature engineering deterministik (~280 fitur) pada `predict_ready.csv`.
   - Menjalankan inferensi 20 model, quad-branch probability blending, threshold calibration, dan domain threat routing.
   - Menghasilkan berkas `submission_datascape_ReproduksiPenyisihan.csv` dan `submission_output.csv` (1.500 baris format `id,category`).

### Opsi B: Clone Lengkap Bersama Bobot (Mode Offline)
Jika dewan juri menguji pada server/lingkungan yang terisolasi dari akses internet luar, repositori dapat di-clone langsung beserta seluruh bobot modelnya:
```bash
git clone https://github.com/ZakyFauzi/pedas2026-reproduksi-penyisihan.git
cd pedas2026-reproduksi-penyisihan
pip install -r requirements.txt
```
Lalu jalankan **Run All** pada `datascape_ReproduksiPenyisihan.ipynb`. Notebook akan langsung mendeteksi bahwa berkas lokal sudah lengkap dan langsung melakukan inferensi tanpa proses download.

---

## 4. Spesifikasi Pipeline & Metodologi

- **Tugas:** Klasifikasi ancaman keamanan domain berbasis DNS log (IDADX dataset).
- **Target Kelas (7):** `brand`, `fakeshop`, `malware`, `online gambling`, `other`, `phishing`, `spam`.
- **Feature Engineering (±280 Fitur):**
  - **Leksikal (60 fitur):** Shannon entropy, panjang string, rasio vokal-konsonan, digit ratio, token count.
  - **N-Gram & Subdomain (90 fitur):** Karakter 3/4-gram, dot depth, subdomain level, mixed-case 0x20 detection.
  - **Protokol DNS (30 fitur):** Query type, rcode status, flag bits, EDNS buffer size, frame length.
  - **Heuristik Reputasi & Defacement (100 fitur):** SLD categorization, keyword security scoring, TLD trustworthiness.
- **Ensemble Quad-Branch (5 Folds CV):**
  - XGBoost (bobot: 0.2874)
  - LightGBM (bobot: 0.2734)
  - CatBoost (bobot: 0.2734)
  - TF-IDF + Calibrated LinearSVC (bobot: 0.1658)
- **Pasca-Pengolahan:**
  - Optimasi bobot blend ensemble via Nelder-Mead.
  - Kalibrasi threshold probabilitas kelas via Powell Optimization (`best_alphas.npy`).
  - Domain threat routing untuk mitigasi salah klasifikasi pada domain perbankan/brand resmi.

---

## 5. Dependensi Lingkungan

- Python: `>= 3.10` (Diuji pada Python 3.12.14)
- Library:
  - `xgboost >= 2.0.0`
  - `lightgbm >= 4.0.0`
  - `catboost >= 1.2.0`
  - `scikit-learn >= 1.3.0`
  - `scipy >= 1.11.0`
  - `pandas >= 2.0.0`
  - `numpy >= 1.24.0`
  - `joblib >= 1.3.0`
  - `tqdm >= 4.66.0`
