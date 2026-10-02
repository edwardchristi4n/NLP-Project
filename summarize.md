# Ringkasan Proyek UTS NLP — Sentimen Publik pada Polemik KPAI vs PB Djarum

> Ringkasan singkat (v0.3 — pindah fokus ke folder `UTS_NLP/`, tim **KelpNLP**). Detail lengkap ada di `PRD_Rencana_Dataset_UTS_NLP.md`. Notebook kerja: `UTS_NLP/notebook/KelpNLP_dataset_EDA.ipynb`.

---

## 1. Arah Proyek

**Tujuan:** mengklasifikasi **sentimen** komentar berita terhadap **KPAI** dan **PB Djarum** (positif/negatif), sekaligus mendeteksi komentar **abusif**, pada polemik audisi bulu tangkis 2019.

| Tugas | Pertanyaan yang dijawab | Output |
|---|---|---|
| **A. Multilabel (sentimen)** | Bagaimana sikap komentar terhadap KPAI dan PB Djarum, dan apakah abusif? | `[KPAI_NEGATIVE, PBDJARUM_POSITIVE, ABUSIVE]` |
| **B. NER** | Entitas apa yang dibicarakan? | `"KPAI"` → GOV_ORG, `"PB Djarum"` → NONGOV_ORG, `"atlet"` → GROUP |

```
Teks  : PB Djarum tuh bukan berjasa cari bibit aja tapi pembiayaan sponsor sampe jadi atlet sama pelatih juga..
        sayang kemampuan otaknya orang2 KPAI terbatas..
NER   : [PB Djarum]NONGOV_ORG ... [atlet]GROUP ... [KPAI]GOV_ORG
Label : [PBDJARUM_POSITIVE, KPAI_NEGATIVE, ABUSIVE]
```

- **UTS:** membangun gold dataset (≥800 teks) melalui analisis domain (4.2), cleaning (4.3), anotasi, IAA, adjudication, EDA, dan Streamlit explorer.
- **UAS:** melatih model sentimen multilabel dan model NER, lalu men-*deploy* keduanya.

### 6 label multilabel
| Label | Kategori yang termasuk |
|---|---|
| `KPAI_POSITIVE` | membela KPAI; setuju branding rokok dilarang; audisi = eksploitasi anak; menolak KPAI dibubarkan |
| `KPAI_NEGATIVE` | kritik kinerja/keputusan KPAI; tuduhan cari sensasi; membandingkan dengan masalah anak lain; khawatir prestasi turun; #BubarkanKPAI; sarkasme ke KPAI |
| `PBDJARUM_POSITIVE` | apresiasi pembinaan atlet; menyayangkan audisi dihentikan; menilai logo wajar |
| `PBDJARUM_NEGATIVE` | audisi = promosi rokok terselubung; kritik yayasan/branding rokok |
| `ABUSIVE` | makian, hinaan, ejekan merendahkan, termasuk bentuk sensor (`beg0`) dan pelesetan (`K pea I`) |
| `NEUTRAL` | tidak bersikap ke KPAI maupun PB Djarum (eksklusif) |

Aturan kunci: sikap ke KPAI dan ke PB Djarum dinilai **terpisah**, target boleh **tersirat**, dan sarkasme dilabeli sesuai **maksud**.

### NER — 4 tipe entitas
NER hanya melabeli **nama atau sebutan entitas**. Opini dan makian **tidak** masuk NER.

| Tipe | Teks yang masuk NER |
|---|---|
| `GOV_ORG` | `KPAI`, `Kpai`, `KAPAI`, `Komisi Perlindungan Anak Indonesia`, `KPI`, `pemerintah`, `Kemenpora` |
| `NONGOV_ORG` | `PB Djarum`, `Djarum Foundation`, `Djarum`, `jarum` (= Djarum), `PBSI`, `Sampoerna` |
| `PERSON` | `Susanto`, `Jokowi`, `Kevin`, `Marcus` (kata ganti tidak masuk) |
| `GROUP` | `anak-anak`, `anak2`, `atlet muda`, `bibit atlet`, `orang tua`, `masyarakat`, `perokok` |

**Hubungan:** `KPAI_*` ↔ `GOV_ORG` · `PBDJARUM_*` ↔ `NONGOV_ORG` · `ABUSIVE` ↔ sasaran makian.

---

## 2. Ringkasan Dataset

| Tahap | Jumlah teks |
|---|---:|
| Dataset asli (15 berita 2019) | 3.184 |
| Setelah cleaning | 3.157 |
| Segmen berita KPAI vs PB Djarum (deteksi posisi baris, ±12 baris) | 1.836 |
| Dibuang: blok berita lain yang ikut tersenggol (mis. KPI/SpongeBob) | sedang divalidasi |
| **Corpus domain final** | **±1.675** *(masih tahap 4.3, lihat catatan)* |
| **Target sampel untuk anotasi** | **≥1.000** |

> **Catatan status angka domain:** analisis 4.2 (di notebook `KelpNLP_dataset_EDA.ipynb`) sudah mengonfirmasi segmen "KPAI vs PB Djarum" adalah satu-satunya topik di atas 800 teks (1.836 teks sebelum dibersihkan dari blok berita lain yang ikut tersenggol oleh jendela ±12 baris). Angka final **1.675** berasal dari seleksi domain berbasis kata kunci (pendekatan awal) dan **belum final** — tahap 4.3 (cleaning) akan memakai seleksi berbasis **rentang baris** (bukan kata kunci) untuk membuang blok berita lain secara lebih presisi, sehingga angka ini kemungkinan direvisi sedikit saat 4.3 dikerjakan dan dikonfirmasi.

**Mengapa domain KPAI vs PB Djarum?** Topik ini satu-satunya yang punya lebih dari 800 teks. Topik terbesar berikutnya, revisi UU KPK, hanya 321 teks.

**Masalah dataset asli:** label hanya abusif atau tidak (single-label, tanpa sentimen atau target); ada label salah (`Mulutmu sampah!!!` berlabel "tidak abusif"); sangat tidak seimbang (±25:1); tidak ada metadata topik (topik ditentukan dari posisi baris, karena data tersusun per berita, bukan dari kolom tersendiri).

**Noise yang ditangani (dikonfirmasi di notebook §4.2.6):** mojibake (348 baris), escape `%u` (106), emoji hilang permanen setelah perbaikan (267), karakter tak terlihat (48), baris baru di dalam sel (294), spasi ganda (165), URL (3), username (2), duplikat persis/case-insensitive (±35), teks tanpa huruf (5). Yang **dipertahankan**: kapital, emoji, tanda baca berulang (`!!!`), hashtag, huruf berulang, dan kata sensor (`beg0`, `K pea I`) — karena ikut membawa makna sentimen/emosi.

**Dugaan distribusi label:** opini publik berat ke pro-Djarum/anti-KPAI, jadi `KPAI_POSITIVE` dan `PBDJARUM_NEGATIVE` akan jarang. Bukti awal dari kata kunci (§4.2.7): menyerang KPAI 13,3%, membela Djarum 5,6%, vs membela KPAI hanya 0,7% dan mengkritik Djarum 1,5%. Imbalance ini akan dilaporkan lebih lengkap di EDA setelah anotasi.

---

## 3. Alur Kerja

| # | Tahap | Di mana | Status |
|---|---|---|---|
| 1 | Judul, deskripsi proyek | notebook — sel awal | ✅ |
| 2 | 4.2 Dataset Selection & Domain Analysis (sumber data, bahasa/keinformalan, jumlah/struktur, label asli, distribusi awal, masalah kualitas, potensi multilabel & NER) | notebook §4.2.1–4.2.8 | ✅ selesai |
| 3 | 4.3 Data cleaning (encoding, dedup, seleksi domain berbasis rentang baris, kolom `in_domain`/`segment`/`thread`) | notebook §4.3 | ⬜ belum dikerjakan (hasil analisis sudah ada, kode belum ditulis) |
| 4 | Text processing (tokenisasi ber-offset, `text_norm`) | notebook | ⬜ |
| 5 | Sampling ≥1.000 teks (purposive: semua abusif + indikasi pro-KPAI/anti-Djarum + proporsional per utas) | notebook | ⬜ |
| 6 | Guideline + pilot | `guideline/` | 🟡 draf v0.2 (di `project_uts/`, perlu disalin/disesuaikan ke `UTS_NLP/`) |
| 7 | Anotasi (Label Studio) | A, B, C | ⬜ |
| 8 | IAA (Fleiss/Cohen κ, Krippendorff α, entity-F1) | notebook | ⬜ |
| 9 | Adjudication & gold dataset | notebook | ⬜ |
| 10 | EDA multilabel & NER | notebook | ⬜ |
| 11 | Streamlit explorer | `streamlit/` | 🟡 ada versi di `project_uts/`, perlu disesuaikan ke dataset domain `UTS_NLP` final |
| 12 | Laporan & ZIP | `report/` | ⬜ |

**Cara menjalankan (saat ini):**
```bash
cd UTS_NLP
jupyter lab notebook/KelpNLP_dataset_EDA.ipynb        # Kernel → Restart & Run All
```

---

## 4. Library

| Library | Kegunaan |
|---|---|
| pandas, numpy | olah tabel, statistik, matriks |
| openpyxl, xlrd | baca `.xlsx` dan `.xls` |
| re, unicodedata, json *(bawaan)* | regex cleaning/tokenisasi, normalisasi Unicode, file JSON/JSONL |
| Sastrawi | stopword & stemming Bahasa Indonesia (opsional, untuk analisis) |
| scikit-learn | Cohen's κ; model di UAS |
| krippendorff | Krippendorff's α |
| matplotlib, seaborn | grafik EDA & heatmap (dipakai di §4.2) |
| plotly, streamlit | grafik interaktif & aplikasi explorer |
| jupyterlab, nbconvert | menjalankan & export notebook |
| Label Studio *(tool)* | anotasi sentimen + NER |
| spaCy `id_nusantara` *(opsional)* | sentence split / POS (PPT minggu 2) |

---

## 5. Langkah Berikutnya
1. Tulis dan jalankan **§4.3 Data Cleaning** di notebook `UTS_NLP/notebook/KelpNLP_dataset_EDA.ipynb`: perbaikan encoding, dedup (kunci ternormalisasi), seleksi domain berbasis **rentang baris** (bukan kata kunci), kolom `in_domain`/`segment`/`thread`, tabel audit sebelum/sesudah — sekaligus mengonfirmasi ulang angka final corpus domain (saat ini masih tertulis ±1.675, sedang divalidasi).
2. Sampling purposive ≥1.000 teks (semua abusif + indikasi pro-KPAI/anti-Djarum + proporsional per utas, seed tetap agar reproducible).
3. Pindahkan/selaraskan `guideline/` dan `streamlit/` dari `project_uts/` ke `UTS_NLP/` dengan penamaan tim **KelpNLP** dan NIM anggota (240712822, 240712828, 240712846).
4. Setup Label Studio → **pilot 50 teks overlap** → hitung κ → revisi guideline → freeze v1.0.
5. Anotasi penuh → IAA → adjudication → gold dataset → EDA + interpretasi.
6. Laporan, export kode PDF, lalu ZIP.

---

## 6. Klarifikasi Desain — Cleaning sampai Anotasi (catatan diskusi 02-10-2026)

### 6.1 Alur jumlah teks (bukan dipotong ke 1.000 saat cleaning)
1. **Cleaning: 3.184 → 3.157.** Semua baris disimpan di `KelpX_dataset_clean.csv`; tidak ada pemotongan topik di tahap ini.
2. **Seleksi domain: 3.157 → ±1.675.** Flag `in_domain=True` untuk utas KPAI vs PB Djarum (thread 1: 267, thread 2: 470, thread 3: 938; minus ±146 teks KPI/SpongeBob yang tersenggol jendela ±12 baris).
3. **Sampling anotasi: 1.675 → 1.000.** Semua 188 teks abusif + 812 teks lain proporsional per utas. Dibagi 200 overlap (A+B+C, untuk IAA) + 800 single (267/267/266). Diawali pilot 50 overlap → hitung κ → revisi → freeze v1.0. Target gold ≥800 (target 1.000 sebagai cadangan `UNANNOTATABLE`).

### 6.2 File output (satu clean CSV + flag, bukan banyak CSV)
- `KelpX_dataset_original.xlsx` — 3.184 asli.
- `KelpX_dataset_clean.csv` — 3.157 baris + kolom `in_domain`, `thread`, `segment`, `annotation_status`.
- `KelpX_dataset_annotation.jsonl` — ±1.000 gold.
- Pendukung: `cleaning_log.csv`, `removed_rows.csv`, `sample_ids.csv`, `labelstudio_tasks_{A,B,C}.json`, `iaa_results.csv`, `adjudication_log.csv`. Tidak anotasi semua karena keterbatasan waktu (±30 teks/hari/orang) dan spesifikasi hanya mewajibkan ≥800.

### 6.3 Dampak distribusi skew pro-Djarum / anti-KPAI
- Bukti awal §4.2.7: serang KPAI 13,3%, bela Djarum 5,6% vs bela KPAI 0,7%, kritik Djarum 1,5%. `KPAI_POSITIVE` dan `PBDJARUM_NEGATIVE` diprediksi langka.
- Keputusan: **biarkan natural, jangan dibalance paksa** — skew adalah temuan opini riil 2019, bukan cacat. Purposive sampling hanya menjamin support minimal label langka, bukan mengubah proporsi publik.
- Konsekuensi: IAA label langka tidak stabil (kappa paradox → lapor per-label κ + % agreement); model UAS bias ke mayoritas (wajib macro-F1, per-label PR, class weight / focal loss); waspadai korelasi semu `KPAI_NEGATIVE ≈ PBDJARUM_POSITIVE` dan `ABUSIVE ≈ KPAI_NEGATIVE`.

### 6.4 Cakupan 6 label multilabel + 4 tipe NER
- Multilabel (`KPAI_POSITIVE/NEGATIVE`, `PBDJARUM_POSITIVE/NEGATIVE`, `ABUSIVE`, `NEUTRAL` eksklusif) + `UNANNOTATABLE` bersifat exhaustive: semua komentar punya kombinasi valid. Sikap ke pihak lain (pemerintah, KPI, sinetron) sengaja masuk `NEUTRAL`.
- NER (`GOV_ORG`, `NONGOV_ORG`, `PERSON`, `GROUP` → 9 tag BIO) hanya untuk nama/sebutan; komentar tanpa entitas tetap valid (dominasi `O` normal).
- Proyeksi: bagus untuk `KPAI_NEGATIVE`, `PBDJARUM_POSITIVE`, `GOV_ORG`, `NONGOV_ORG`, `GROUP`; sedang untuk `ABUSIVE`/`NEUTRAL`; lemah untuk `KPAI_POSITIVE`, `PBDJARUM_NEGATIVE`, `PERSON`. Tidak perlu tambah label baru.
- Kriteria freeze v1.0: `UNANNOTATABLE` ≤5% dan tidak ada pola disagree "tidak ada label yang cocok" pada pilot 50.
