# Reproduksi Model Penyisihan — Tim datascape
## PeDaS 2026 | Babak Final

---

## Deskripsi

Paket ini berisi seluruh artefak yang diperlukan untuk mereproduksi prediksi terbaik tim **datascape** pada babak penyisihan PeDaS 2026.

- **Tugas**: Klasifikasi domain berbasis DNS log (IDADX dataset)  
- **Metrik**: Macro F1-score  
- **Skor lokal (OOF)**: ~0.975  
- **Skor leaderboard**: 0.975287141

---

## Struktur Folder

```
reproduksi-penyisihan/
├── README.md                     # File ini
├── datascape_ReproduksiPenyisihan.ipynb  # Notebook utama — jalankan Run All
├── predict_ready.csv             # Data input siap pakai (preprocessed)
├── models/
│   ├── xgb_fold0.pkl ... xgb_fold4.pkl    # 5 model XGBoost
│   ├── lgb_fold0.pkl ... lgb_fold4.pkl    # 5 model LightGBM
│   ├── cat_fold0.pkl ... cat_fold4.pkl    # 5 model CatBoost
│   ├── nlp_fold0.pkl ... nlp_fold4.pkl    # 5 model TF-IDF + NLP
│   └── le_label.pkl                       # Label encoder
├── weights/
│   ├── opt_weights.npy     # Bobot ensemble (Nelder-Mead)
│   └── best_alphas.npy     # Kalibrasi multiplier per kelas
└── submission_datascape_ReproduksiPenyisihan.csv   # Output prediksi (dihasilkan saat Run All)
```

---

## Cara Menjalankan

### Langkah

1. Pastikan semua folder `models/` dan `weights/` ada di direktori yang sama dengan notebook
2. Buka `datascape_ReproduksiPenyisihan.ipynb` di Jupyter Notebook / JupyterLab
3. Jalankan **Run All** (Kernel → Restart & Run All)
4. File `submission_datascape_ReproduksiPenyisihan.csv` akan terbentuk otomatis

### Catatan Penting

- **Tidak ada training ulang** — notebook hanya memuat model, menjalankan feature engineering, dan inferensi
- Seluruh path menggunakan path relatif (`./models/`, `./weights/`) sehingga portabel di lingkungan apapun
- Eksekusi penuh membutuhkan ~5–10 menit tergantung hardware

---

## Deskripsi Pipeline

### Feature Engineering (±280 fitur)

| Grup Fitur | Jumlah | Deskripsi |
|---|---|---|
| Lexical | ~60 | Panjang, entropi, jumlah digit/huruf/karakter khusus |
| N-gram | ~50 | Karakter 3/4-gram, bigram kata |
| Domain struktural | ~40 | TLD, SLD, subdomain depth, mixed-case |
| Reputasi domain | ~50 | TLD trustworthiness, pola phishing/malware/gambling |
| DNS protocol | ~30 | qtype, rcode, flag bits, EDNS, frame_len |
| Defacement pattern | ~50 | Pola HTML keywords, URL anomali, script injection |

### Ensemble Strategy

4 model × 5 fold = 20 base learner, blend probabilitas dengan bobot:
- XGBoost: 0.2874
- LightGBM: 0.2734  
- CatBoost: 0.2734
- TF-IDF/NLP: 0.1658

Diikuti kalibrasi per-kelas (alpha multiplier) via Powell optimization.

---

## Dependencies

Utama: `xgboost`, `lightgbm`, `catboost`, `scikit-learn`, `pandas`, `numpy`
