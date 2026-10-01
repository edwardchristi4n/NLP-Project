# PRD & Rencana Pengelolaan Dataset — Proyek UTS Pemrosesan Bahasa Alami

**Dataset:** *Abusive Language Detection on Indonesian Online News Comments* (`id_abusive_news_comment`, dataset No. 7 di daftar resmi)
**Mata kuliah:** Pemrosesan Bahasa Alami (INFT41603) — Gasal 2026/2027
**Fokus dokumen:** UTS (dataset development & anotasi), dengan catatan singkat menuju UAS
**Acuan:** `Spesifikasi_Proyek_UTS_UAS_PBA_2026_2027_Final_v2.pdf` + PPT Minggu 2 *Corpus Development, Processing and Understanding Text*
**Status:** Draf v0.1 — taxonomy masih **usulan** dan wajib dikonsultasikan ke dosen sebelum di-*freeze*

> Catatan penamaan: semua file memakai awalan `KelpX_`. Ganti `X` dengan nomor kelompok.

---

## Daftar Isi

1. [Ringkasan Singkat](#1-ringkasan-singkat)
2. [Isi Folder Project_NLP dan Fungsinya](#2-isi-folder-project_nlp-dan-fungsinya)
3. [Hasil Analisis Dataset](#3-hasil-analisis-dataset)
4. [PRD (Product Requirements Document)](#4-prd-product-requirements-document)
5. [Rancangan Taxonomy (Usulan Awal)](#5-rancangan-taxonomy-usulan-awal)
6. [Pipeline End-to-End UTS](#6-pipeline-end-to-end-uts)
7. [Rencana Detail per Tahap](#7-rencana-detail-per-tahap)
8. [Skema Data & Format File](#8-skema-data--format-file)
9. [Daftar Fungsi yang Akan Dibuat](#9-daftar-fungsi-yang-akan-dibuat)
10. [Jadwal Kerja Usulan](#10-jadwal-kerja-usulan)
11. [Risiko & Mitigasi](#11-risiko--mitigasi)
12. [Checklist Pengumpulan UTS](#12-checklist-pengumpulan-uts)
13. [Jembatan ke UAS](#13-jembatan-ke-uas)
14. [Lampiran: Kode Inti yang Sudah Diuji](#14-lampiran-kode-inti-yang-sudah-diuji)

---

## 1. Ringkasan Singkat

| Hal | Isi |
|---|---|
| **Apa datanya?** | 3.184 komentar pembaca berita online berbahasa Indonesia (Kompas, Kaskus, Detik) dari 15 berita populer tahun 2019. |
| **Label asli** | 1 kelas per komentar: `1` tidak abusive (87,6%), `2` abusive tapi tidak ofensif (3,5%), `3` abusive dan ofensif (9,0%). |
| **Masalah utama data** | Sangat tidak seimbang, label asli bersifat *single-label* (tidak bisa langsung dipakai), ada label yang keliru, ada karakter rusak (*mojibake*) di ±11% baris, dan emoji yang terkodekan salah. |
| **Apa yang harus dibuat di UTS?** | Merumuskan ulang data menjadi **2 tugas**: (A) *Multilabel Text Classification* dan (B) *NER/Span*, lalu membangun **gold dataset** lewat anotasi, inter-annotator agreement (IAA), adjudication, EDA, dan **Streamlit dataset explorer**. |
| **Usulan taxonomy** | Multilabel 9 label (jenis abusif + sikap/tujuan komentar). NER 7 tipe span (sasaran + ekspresi). Lihat Bagian 5. |
| **Hasil cleaning (prototipe)** | 3.184 → **3.157** teks unik dan layak; 932 baris diperbaiki teksnya. |
| **Target anotasi** | **1.000 teks** (minimal wajib 800), diambil dengan *stratified purposive sampling*. |
| **Output akhir UTS** | 9 file wajib + ZIP (lihat Bagian 12). |

---

## 2. Isi Folder Project_NLP dan Fungsinya

```
Project_NLP/
├── Spesifikasi_Proyek_UTS_UAS_PBA_2026_2027_Final_v2.pdf   ← aturan, output wajib, rubrik
└── Indonesian-Online-News-Comments/                        ← repo GitHub penulis asli dataset
    ├── README.md
    ├── Dataset/
    │   ├── Abusive Language Detection on Indonesian Online News Comments Dataset .xlsx
    │   ├── Acronym Words and Typos .xlsx
    │   └── kumpulan_singkatan_dan_kata_dasar.xls
    └── Code/
        ├── 0. Crawling Dataset.ipynb
        ├── 1. Building Dataset.ipynb
        ├── 2. Feature Extraction and Selection.ipynb
        ├── 3. KNN Classification.ipynb
        ├── 3. Naive Bayes Classification.ipynb
        ├── 3. SVM Classification.ipynb
        └── 4. K-Fold Cross Validation.ipynb
```

### 2.1 File data

| File | Isi | Ukuran | Fungsi dalam proyek kita |
|---|---|---|---|
| `Abusive Language Detection ... Dataset .xlsx` | Kolom `Kalimat` (teks komentar) dan `label` (1/2/3) | 3.184 baris × 2 kolom | **Corpus utama.** Disalin menjadi `KelpX_dataset_original.xlsx`. |
| `Acronym Words and Typos .xlsx` | Kolom `kata` → `kata sebenarnya` (singkatan, typo, huruf berulang). Contoh: `krna → karena`, `jelasssss → jelas`, `ente → kamu` | 698 pasangan | **Kamus normalisasi** untuk kolom `text_norm` (EDA & fitur UAS). Tidak dipakai untuk teks anotasi. |
| `kumpulan_singkatan_dan_kata_dasar.xls` | Singkatan → kata baku (tanpa header). Contoh: `sblm → sebelum`, `g → tidak` | 214 pasangan | Kamus normalisasi tambahan, **harus dibersihkan dulu** (lihat 3.6). |

### 2.2 Notebook kode dari penulis asli

| Notebook | Fungsi | Catatan penting untuk kita |
|---|---|---|
| `0. Crawling Dataset` | Mengambil komentar Detik lewat API GraphQL (`newcomment.detik.com`), per halaman, lalu disimpan ke CSV per judul berita. | Membuktikan asal data (penting untuk *domain analysis*). Output mentah masih berisi tag iklan HTML (`<ins ...>`) dan karakter `�`. |
| `1. Building Dataset` | Preprocessing: hapus emoji, simbol, angka, *newline* → lowercase → tokenisasi regex → deteksi singkatan (kata tanpa huruf vokal) → ganti singkatan pakai kamus + `kamusalay.csv` → hapus stopword → simpan `dataset.csv`. | Cleaning-nya **terlalu agresif** untuk tujuan kita: emoji, tanda baca, dan kapitalisasi dibuang, padahal itu penting untuk NER dan deteksi abusif (PPT slide 73–74). Juga memerlukan file yang tidak ada di repo (`dataset_stemming - Sheet1.csv`, `stopwords.txt`, `kamusalay.csv`). |
| `2. Feature Extraction and Selection` | Bag-of-Words (vocab 6.188 kata) → hitung *Mutual Information* per kata secara manual → simpan subset fitur 10%–90% teratas. | MI dihitung pada **seluruh data sebelum split** → *data leakage*. Kata dengan MI tertinggi: `goblok, tolol, bodoh, bego, alay, dungu` — berguna sebagai bukti leksikon abusif. |
| `3. KNN / Naive Bayes / SVM` | Split 70/30 acak, latih klasifier, cetak akurasi & F1. | Akurasi 86–87% **menyesatkan**: confusion matrix KNN & SVM menunjukkan model **selalu menebak kelas 1** (recall kelas 2 dan 3 = 0, macro-F1 ≈ 0,31). Bukti nyata dampak imbalance. |
| `4. K-Fold Cross Validation` | 10-fold CV untuk KNN dan GaussianNB. | Tanpa `random_state`, metrik hanya *accuracy* → tidak reproducible dan tidak cocok untuk data imbalance. |

**Kesimpulan:** kode lama **tidak dipakai ulang** sebagai pipeline. Yang diambil hanya (1) informasi asal-usul data, (2) kamus normalisasi, dan (3) pelajaran tentang imbalance dan leakage untuk dibahas di laporan.

---

## 3. Hasil Analisis Dataset

> Semua angka di bawah dihitung langsung dari file `.xlsx` (bukan perkiraan).

### 3.1 Identitas & domain

| Aspek | Temuan |
|---|---|
| Sumber | Kolom komentar Kompas, Kaskus, dan Detik. Paper: IEEE Xplore doc. 9034620 (README repo). |
| Periode | Berita populer tahun 2019. |
| Bahasa | Bahasa Indonesia **informal**: singkatan (`yg`, `gak`, `klo`), campuran Jawa/Betawi (`ente`, `ane`, `utekmu`), slang internet (`wkwk`, `gan`, `bray`), dan sedikit bahasa Inggris. |
| Domain | Opini publik tentang **politik, kebijakan, lembaga negara, bisnis, dan hiburan**. |
| Unit data | 1 komentar = 1 baris. |
| Metadata | **Tidak ada**: tidak ada ID, sumber per baris, tanggal, atau judul berita. Topik harus ditebak dari isi teks. |
| Lisensi | Repo tidak menyertakan file LICENSE → gunakan hanya untuk keperluan akademik dan **wajib sitasi** paper/repo. |

**15 berita sumber** (dari README): akuisisi Indosat oleh Sandi, tarif tol (Fadli Zon), Muslimah HTI di Pemprov DKI, KPU vs Tim IT Prabowo, vonis Ahmad Dhani, SBY–Ani, Luna Maya–Aladdin, nasib Esia, revisi UU KPK (2 berita), PB Djarum vs KPAI (3 berita), #bubarkanKPAI, kartun SpongeBob di GTV.

### 3.2 Distribusi label asli

| Label | Arti | Jumlah | Persen |
|---|---|---:|---:|
| 1 | *not abusive* | 2.789 | 87,59% |
| 2 | *abusive but not offensive* | 110 | 3,45% |
| 3 | *abusive and offensive* | 285 | 8,95% |
| | **Total** | **3.184** | 100% |

Rasio imbalance kelas terbesar : terkecil ≈ **25 : 1**.

### 3.3 Distribusi topik (deteksi kata kunci, perkiraan)

| Topik | Jumlah teks | Label 3 | Label 2 |
|---|---:|---:|---:|
| KPAI / PB Djarum / KPI / audisi bulu tangkis | 1.247 | 133 | 19 |
| Revisi UU KPK / Jokowi / korupsi | 163 | 5 | 1 |
| Indosat / Sandi / Qatar | 105 | 8 | 3 |
| Tarif tol / Fadli Zon | 104 | 13 | 3 |
| HTI / Pemprov DKI | 73 | 2 | 0 |
| SpongeBob / GTV / kartun | 56 | 3 | 2 |
| Esia / Bakrie | 47 | 1 | 2 |
| Lainnya (Luna Maya, KPU, SBY, Ahmad Dhani, dan teks tanpa kata kunci) | ±1.400 | ±132 | ±81 |

**Implikasi:** corpus didominasi isu **KPAI vs PB Djarum (±39%)**, dan entitas yang paling sering muncul adalah `KPAI` (905 teks), `Djarum` (362), `KPI` (118), `KPK` (110), `Jokowi` (90), `DPR` (60). Ini menjadi dasar tipe entitas `TARGET_ORG` dan `TARGET_PERSON`.

### 3.4 Panjang teks

| Ukuran | Min | Q1 | Median | Mean | Q3 | Max |
|---|---:|---:|---:|---:|---:|---:|
| Jumlah kata | 1 | 8 | 14 | 19,9 | 26 | 486 |
| Jumlah karakter | 2 | 47 | 88 | 129 | 164 | 3.162 |

- Panjang hampir sama di ketiga label (median 14–15 kata) → panjang bukan pembeda kelas.
- 15 teks > 128 kata (perlu perhatian *truncation* saat UAS memakai transformer).
- 29 teks hanya 1 kata, tetapi banyak yang tetap bermakna, misalnya `kampungan`, `Bahalul`, `guoblog...` → **jangan dibuang hanya karena pendek**.

### 3.5 Masalah kualitas teks (*noise*)

| Jenis masalah | Jumlah baris | Contoh | Penanganan |
|---|---:|---|---|
| *Mojibake* (UTF-8 terbaca sebagai Mac Roman) | 348 | `ÔøΩ`, `‚Å£‚Å£GOBLOK`, `üòÇ` | **Bisa dipulihkan 100%** dengan `s.encode('mac_roman').decode('utf-8')`. `üòÇ` kembali menjadi 😂. |
| Emoji hilang permanen (`�`, U+FFFD) | 267 | `...100%gak percaya,�` | Emoji asli sudah hilang sejak crawling → dihapus, dicatat di kolom *flag*. |
| Emoji ter-*escape* gaya JavaScript | 106 | `%uD83D%uDE02` | Didekode menjadi 😂. |
| Karakter tak terlihat (U+2063, dll.) | 48 | awal `GOBLOK` | Dihapus. |
| Baris baru / spasi berlebih | 540 | `...klo gt\n` | Diseragamkan menjadi 1 spasi. |
| URL | 2 | `www.isamku.com` | Diganti `[URL]`. |
| Username | 2 | `@khafidz99` | Diganti `[USER]` (privasi). |
| HTML | 0 (di file final) | — | Tetap dicek; di data crawling mentah ada `<ins ...>`. |
| Duplikat (setelah cleaning, *case-insensitive*) | 22 | `#BubarkanKPAI` ×8 varian | Dihapus, simpan kemunculan pertama. **Tidak ada duplikat yang labelnya bertentangan.** |
| Tanpa huruf sama sekali | 5 | `:(`, `1111`, `1 + 1 = 2` | Dihapus (tidak bisa dianotasi). |
| Hashtag | 86 | `#BubarkanKPAI`, `#savepbdjarum` | **Dipertahankan** (bukti tuntutan/ajakan). |
| Huruf berulang | 198 | `Palsuuuuuu`, `Bahluuull` | Dipertahankan di teks anotasi; dinormalkan hanya di `text_norm`. |
| Tanda baca berulang | 1.265 | `!!!`, `???` | Dipertahankan (sinyal emosi). |
| Kata kasar disensor/disamarkan | ±20 | `b*rak`, `bl**n`, `beg0`, `ng3w3`, `bgsd`, `KNTL` | Dipertahankan; aturan khusus di guideline NER. |
| Reduplikasi dengan angka | 564 | `anak2`, `orang2` | Dipertahankan di teks anotasi; `text_norm` → `anak-anak`. |

### 3.6 Masalah kamus normalisasi

| Temuan | Detail | Tindakan |
|---|---|---|
| Gabungan 2 kamus | 810 kata unik | Gabung jadi `kamus_normalisasi.csv` |
| Konflik makna | 12 kata punya 2 arti, mis. `krn → karena` vs `krn → kamu` (salah), `kl → kalau/kali`, `kt → kita/kata` | Pilih manual satu arti paling umum, atau buang kata ambigu |
| Nilai kosong | 37 entri di `kumpulan_singkatan...xls` berisi `'` (placeholder), mis. `kmrn`, `btw`, `dll` | Hapus atau isi manual (`kmrn → kemarin`) |
| Spasi di belakang | 15 nilai, mis. `aje → "aja "` | `strip()` |
| Cakupan | Kamus hanya mengenai ±7% token corpus | Wajar; normalisasi bukan inti UTS |

### 3.7 Label asli tidak cukup dan mengandung kesalahan

1. **Single-label**: satu komentar hanya punya satu kelas, sedangkan spesifikasi menuntut **multilabel**. Contoh `KPAI survei ke anak2 onta gurun mungkin, dasar KPAI totol. Kalo ada petisi pembubaran KPAI gw auto ikut` sekaligus mengandung hinaan, serangan identitas, dan ajakan.
2. **Label tidak konsisten**: 23 teks berlabel 1 (*not abusive*) mengandung kata kasar kuat, misalnya:
   - `Lembaga sampah. Bubar aja luh.` → label 1
   - `Mulutmu sampah!!!` → label 1
   - `Ngapain beli perusahaan yg mau ambruk dasar dungu` → label 1
3. **Leksikon kasar** (`goblok, tolol, bodoh, bego, dungu, sampah, kampret, ...`) muncul di 52% teks label 3, 32% label 2, dan hanya 4% label 1. Jadi leksikon berguna, tetapi tidak cukup: 48% teks label 3 abusif **tanpa** kata kasar umum (sindiran, ejekan, pelesetan seperti `KAPAI jadi tapai`).

> **Argumen untuk laporan (rubrik 5.1 & 5.3):** label asli hanya dipakai sebagai **referensi awal dan bahan sampling**. Taxonomy baru diturunkan dari pola isi corpus di atas, sehingga lebih kaya (multilabel) dan dapat dijelaskan dengan bukti.

---

## 4. PRD (Product Requirements Document)

### 4.1 Latar belakang

Komentar berita online banyak berisi hinaan, ejekan, dan serangan terhadap tokoh, lembaga, atau kelompok. Dataset asli hanya menjawab "abusif atau tidak". Moderasi nyata perlu tahu lebih banyak: **jenis abusifnya apa, siapa sasarannya, dan bagian kalimat mana yang menjadi buktinya**.

### 4.2 Pernyataan masalah

1. **Task A — Multilabel Text Classification (level teks):** *"Fenomena apa saja yang ada dalam komentar ini?"* Contohnya hinaan, kata kasar, ejekan, serangan identitas, ancaman, kritik, dukungan, ajakan, atau netral. Satu komentar boleh punya lebih dari satu label.
2. **Task B — NER/Span Classification (level span):** *"Bagian teks mana yang menjadi sasaran dan bukti fenomena tersebut?"* Contohnya nama tokoh, lembaga, kelompok, ekspresi kasar, tuntutan, isu, dan pujian.
3. **Hubungan A–B:** label multilabel harus bisa "dibuktikan" oleh span NER. Contoh: label `INSULT` ↔ span `ABUSIVE_EXPR` + `TARGET_ORG`.

### 4.3 Tujuan UTS (*goals*)

| ID | Tujuan | Ukuran keberhasilan |
|---|---|---|
| G1 | Corpus terdokumentasi dan bersih | Statistik sebelum/sesudah tersedia; setiap perubahan bisa ditelusuri via `id` & *flag* |
| G2 | Taxonomy multilabel & NER yang jelas | Disetujui dosen dan di-*freeze*; tiap label punya definisi, inclusion/exclusion, contoh +/−/ambigu |
| G3 | Gold dataset ≥ 800 teks unik | Target 1.000; 100% lolos validasi span |
| G4 | Anotasi konsisten | IAA multilabel (Fleiss' κ / Krippendorff α) ≥ 0,60 untuk label utama; NER entity-F1 antar-annotator ≥ 0,70 |
| G5 | EDA lengkap & bermakna | Semua item di spesifikasi 4.8 ada beserta interpretasi |
| G6 | Dataset explorer berjalan | Streamlit 5 menu, tanpa error, tanpa *hard-coded path* |
| G7 | Reproducible | Notebook *Restart & Run All* sukses dari folder ZIP |

### 4.4 Di luar cakupan UTS (*non-goals*)

- Melatih model klasifikasi atau NER (itu UAS).
- Stemming, stopword removal, atau normalisasi slang **pada teks anotasi**. Ini hanya untuk kolom turunan.
- Crawling data baru. Corpus memakai dataset resmi saja.

### 4.5 Pengguna / pemangku kepentingan

| Pengguna | Kebutuhan |
|---|---|
| Annotator (3 anggota) | Guideline jelas, tool anotasi mudah, pembagian kerja adil |
| Dosen penilai | Bukti keputusan, statistik, reproducibility, explorer yang bisa dicoba |
| Kelompok sendiri saat UAS | Gold dataset bersih dengan offset span valid, siap di-split |

### 4.6 Kebutuhan fungsional

| ID | Kebutuhan | Output |
|---|---|---|
| FR-01 | Memuat dataset asli dari path relatif dan memberi ID unik `ANC-0001…` | `df_raw` |
| FR-02 | Menjalankan cleaning bertahap dengan log jumlah baris terdampak per langkah | `KelpX_dataset_clean.csv`, tabel log |
| FR-03 | Menyimpan **3 versi teks**: `text_raw`, `text_clean` (basis anotasi), `text_norm` (analisis) | kolom CSV |
| FR-04 | Memberi metadata: topik (kata kunci), label asli, *flag* kualitas, jumlah token | kolom CSV |
| FR-05 | Sampling 1.000 teks secara terdokumentasi, dengan seed tetap | `sample_ids.csv` |
| FR-06 | Menyusun guideline anotasi lengkap | `KelpX_annotation_guideline.pdf` |
| FR-07 | Anotasi multilabel + span sekaligus dalam satu tool | export JSON per annotator |
| FR-08 | Menghitung IAA multilabel (per label & agregat) dan NER (exact & partial match) | tabel IAA |
| FR-09 | Adjudication dengan log keputusan | `adjudication_log.csv`, gold JSONL |
| FR-10 | Validasi span: offset cocok dengan substring, tidak overlap, tidak ada spasi di tepi, label valid | laporan validasi = 0 error |
| FR-11 | Konversi span → BIO untuk EDA token-level | statistik O vs entity |
| FR-12 | EDA multilabel & NER sesuai spesifikasi 4.8 | grafik + interpretasi di notebook |
| FR-13 | Streamlit explorer: Overview, Browse, Multilabel Analysis, NER Analysis, Filter/Search | `KelpX_dataset_explorer.py` |

### 4.7 Kebutuhan non-fungsional

- **Reproducible:** semua path relatif (`BASE_DIR = Path(__file__).parent` / `Path.cwd()`), `random_state=42`, dan `requirements.txt`.
- **Traceable:** setiap baris gold bisa dilacak kembali ke baris asli lewat `id`.
- **Aman offset:** teks anotasi (`text_clean`) **tidak boleh berubah** setelah anotasi dimulai (PPT slide 75).
- **Terdokumentasi:** setiap sel penting di notebook diawali Markdown (tujuan → keputusan → interpretasi).
- **Etis:** username di-*mask*; explorer diberi peringatan konten kasar; sitasi dataset, paper, dan library.

### 4.8 Pemetaan ke rubrik UTS

| Kriteria rubrik | Bobot | Bagian dokumen ini yang menjawab |
|---|---:|---|
| Dataset Selection & Problem Formulation | 10% | 3, 4.2, 5.3 |
| Data Cleaning & Preparation | 10% | 3.5, 7 (Tahap 2–3) |
| Multilabel Taxonomy Design | 15% | 5.1 |
| NER Taxonomy & Guideline | 15% | 5.2, 7 (Tahap 6) |
| Annotation Process & Adjudication | 15% | 7 (Tahap 7, 9) |
| Inter-Annotator Agreement | 10% | 7 (Tahap 8) |
| EDA Multilabel & NER | 15% | 7 (Tahap 10) |
| Dataset Explorer & Reproducibility | 10% | 7 (Tahap 11), 4.7 |

---

## 5. Rancangan Taxonomy (Usulan Awal)

> ⚠️ Ini **draf berbasis bukti dari corpus**, bukan final. Wajib dibahas 1× dengan dosen (spesifikasi 4.4.3), diuji lewat *pilot annotation*, lalu di-*freeze* dengan nomor versi.

### 5.1 Skema A — Multilabel (9 label)

Label disusun dalam **dua dimensi** supaya kombinasi label wajar terjadi:
- **Dimensi 1, jenis abusif:** PROFANITY, INSULT, MOCKERY, IDENTITY_ATTACK, THREAT
- **Dimensi 2, sikap/tujuan komentar:** CRITICISM, SUPPORT, CALL_TO_ACTION, NEUTRAL

| # | Label | Definisi operasional | Inclusion rule | Exclusion rule | Contoh positif (dari corpus) | Bukti di corpus |
|---|---|---|---|---|---|---|
| 1 | `PROFANITY` | Kata kasar, vulgar, atau cabul, termasuk bentuk sensor/samaran | Ada kata makian/vulgar (anjing, tai, bangsat, kntl, bgsd, b*rak) | Kata "anjing/babi" yang merujuk hewan secara harfiah | `Lah, kpi? kocak lu bgsd !` | Leksikon kasar ada di 306 teks |
| 2 | `INSULT` | Merendahkan kecerdasan, moral, fisik, atau martabat sasaran tertentu | Ada sasaran (orang/lembaga) + ungkapan merendahkan (goblok, dungu, otak gada isinya, sampah) | Kritik kinerja tanpa kata merendahkan → `CRITICISM` | `KPAI gak ada guna, sekumpulan orang gabut yg beg0, gak bisa mikir jernih.` | goblok 39, bodoh 37, tolol 28 teks |
| 3 | `MOCKERY` | Ejekan, sindiran, olok-olok, atau pelesetan nama tanpa makian eksplisit | Nada mengejek atau pelesetan (`KAPAI jadi tapai`), tawa sinis yang ditujukan ke sasaran | Humor netral tanpa sasaran; tawa biasa (`wkwk`) saja | `Saking pintar nya logika KAPAI jadi tapai...` | Mayoritas label asli 2; tawa/ejekan 108 teks |
| 4 | `IDENTITY_ATTACK` | Serangan berbasis identitas kelompok (politik, agama, etnis) memakai sebutan merendahkan | Ada sebutan kelompok yang merendahkan (cebong, kampret, kadrun, onta, antek aseng, PKI sebagai cap) | Menyebut nama kelompok/organisasi secara netral (`HTI dibubarkan pemerintah`) | `kampret2 jg pura2 gak punya wakil di dpr` | Sebutan kelompok 118 teks |
| 5 | `THREAT` | Ancaman atau keinginan kekerasan fisik terhadap sasaran | Ungkapan ingin/akan memukul, membunuh, membakar (tabok, gebuk, bunuh) yang ditujukan ke sasaran | Kata kekerasan dalam konteks berita (`pembunuhan suami di sinetron`) | `tetep pengen gue tabokin satu satu sih itu PKI eh, KPI` | Kata kekerasan 73 teks, ancaman nyata jauh lebih sedikit (perlu dicek di pilot) |
| 6 | `CRITICISM` | Kritik atau ketidaksetujuan terhadap kebijakan, kinerja, atau keputusan, dengan argumen/alasan | Menyatakan tidak setuju + alasan atau penilaian kinerja | Hanya makian tanpa substansi → `INSULT`/`PROFANITY` saja | `KPAI bukan melindungi anak Indonesia tapi merampas masa depan mereka yg bercita2...` | Kata kebijakan/kinerja 498 teks |
| 7 | `SUPPORT` | Dukungan, pembelaan, atau apresiasi terhadap tokoh, lembaga, atau kebijakan | Ada pujian/dukungan eksplisit (dukung, salut, terima kasih, bangga, harusnya didukung) | Pujian sarkastis → `MOCKERY` | `Harusnya PB Djarum didukung, kalo perlu diwajibkan...` | Kata dukungan 220 teks |
| 8 | `CALL_TO_ACTION` | Tuntutan atau ajakan agar pihak tertentu bertindak | Ada perintah/tuntutan/ajakan (bubarkan, petisi, copot, usut, mundur), termasuk hashtag tuntutan | Saran umum tanpa tuntutan ke pihak tertentu | `KPAI? bubar aja` · `#BubarkanKPAI` | Kata tuntutan 255 teks |
| 9 | `NEUTRAL` | Informasi, pertanyaan, atau komentar tanpa sikap dan tanpa unsur abusif | Tidak memenuhi label 1–8 | **Eksklusif**: tidak boleh digabung dengan label lain | `Ini lembaga negara. Pasti punya power dan ego kalo misal suratnya diabaikan.` | Pertanyaan/info umum |

**Aturan umum multilabel:**
- Berikan **semua** label yang terpenuhi. Contoh: `Lembaga sampah. Bubar aja luh.` → `[INSULT, CALL_TO_ACTION]`.
- `PROFANITY` dan `INSULT` boleh muncul bersama. `PROFANITY` = ada kata kasar; `INSULT` = ada sasaran yang direndahkan.
- `NEUTRAL` hanya bila tidak ada label lain. Teks *spam* atau tidak bisa dipahami tidak diberi label; teks itu ditandai `UNANNOTATABLE` dan dikeluarkan dari gold (dicatat jumlahnya).
- Kasus ambigu: putuskan berdasarkan **maksud keseluruhan** komentar (mengikuti PPT slide 49), lalu tulis keputusannya di log.

**Pemetaan kasar ke label asli** (untuk laporan, bukan aturan):
`label 3` ≈ INSULT / PROFANITY / IDENTITY_ATTACK / THREAT · `label 2` ≈ MOCKERY atau abusif ringan · `label 1` ≈ CRITICISM / SUPPORT / CALL_TO_ACTION / NEUTRAL, ditambah sebagian salah label.

> **Rencana cadangan:** jika setelah pilot `THREAT` < 30 contoh di 1.000 teks, gabungkan ke `INSULT` atau ubah menjadi `HARASSMENT`, dengan persetujuan dosen.

### 5.2 Skema B — NER / Span (7 tipe, skema tag BIO)

| # | Tipe span | Definisi | Contoh | Aturan boundary utama |
|---|---|---|---|---|
| 1 | `TARGET_PERSON` | Nama atau sebutan individu yang dibicarakan/disasar | `Jokowi`, `Sandi`, `Fadli Zon`, `Luna Maya`, `Pak Jokowi` | Sertakan gelar/sapaan yang menempel (`Pak`, `Bang`). Kata ganti (`lu`, `ente`, `dia`) **tidak** dianotasi. |
| 2 | `TARGET_ORG` | Lembaga, organisasi, perusahaan, atau institusi | `KPAI`, `PB Djarum`, `KPK`, `DPR`, `KPI`, `Indosat`, `Pemprov DKI` | Ambil nama lengkap yang tertulis (`PB Djarum`, bukan hanya `Djarum` jika ada `PB`). Pelesetan nama (`KAPAI`) tetap `TARGET_ORG`. |
| 3 | `TARGET_GROUP` | Kelompok sosial/politik yang disebut **secara netral** | `warga +62`, `ormas`, `pendukung 02`, `anak-anak atlet` | Sebutan kelompok yang **merendahkan** (cebong, kampret, onta) masuk `ABUSIVE_EXPR`, bukan di sini. |
| 4 | `ABUSIVE_EXPR` | Ekspresi kasar, hinaan, ejekan, slur kelompok, atau ancaman | `goblok`, `otak gada isinya`, `sampah masyarakat`, `onta gurun`, `tabokin` | Ambil **frasa evaluatif terpendek yang utuh maknanya**. Jangan sertakan target (`KPAI goblok` → `[KPAI]ORG [goblok]ABUSIVE`). Huruf berulang & sensor ikut dalam span (`Bahluuull`, `b*rak`). Tanda baca di ujung tidak ikut. |
| 5 | `DEMAND_EXPR` | Frasa tuntutan/ajakan bertindak | `bubar aja`, `petisi pembubaran`, `usut tuntas`, `#BubarkanKPAI` | Target di dalam frasa dipisah, kecuali hashtag satu token (diberi label utuh `DEMAND_EXPR`). |
| 6 | `ISSUE` | Isu, kebijakan, atau topik yang dikritik/dibela | `eksploitasi anak`, `revisi UU KPK`, `tarif tol`, `audisi bulutangkis` | Frasa benda inti; tanpa kata sifat penilai (`tarif tol mahal` → `[tarif tol]ISSUE`). |
| 7 | `PRAISE_EXPR` | Ekspresi pujian/dukungan | `salut`, `terima kasih`, `harusnya didukung`, `membanggakan negara` | Sama dengan `ABUSIVE_EXPR`: frasa terpendek, tanpa target. |

**Aturan NER umum:**
- **Tidak ada nested/overlap span** (BIO tidak mendukungnya). Jika bertabrakan, pilih span yang menjadi bukti label multilabel.
- Span harus tepat di batas token. Tidak boleh memotong kata, dan tidak boleh ada spasi di awal/akhir span.
- Entitas yang muncul berulang **dianotasi setiap kali muncul**.
- Total tag BIO: 7 × 2 + `O` = **15 tag**.

### 5.3 Hubungan multilabel ↔ NER

| Label multilabel | Span bukti yang diharapkan |
|---|---|
| PROFANITY | `ABUSIVE_EXPR` |
| INSULT | `ABUSIVE_EXPR` + `TARGET_PERSON/ORG/GROUP` |
| MOCKERY | `ABUSIVE_EXPR` (ejekan/pelesetan) + target |
| IDENTITY_ATTACK | `ABUSIVE_EXPR` (slur kelompok) |
| THREAT | `ABUSIVE_EXPR` (ancaman) + target |
| CRITICISM | `ISSUE` dan/atau target |
| SUPPORT | `PRAISE_EXPR` + target/`ISSUE` |
| CALL_TO_ACTION | `DEMAND_EXPR` + target |
| NEUTRAL | boleh tanpa span, atau hanya target/`ISSUE` |

**Contoh lengkap (dari corpus, sudah diuji dengan kode di Lampiran):**

```json
{
  "id": "ANC-xxxx",
  "text": "KPAI survei ke anak2 onta gurun mungkin, dasar KPAI totol. #BubarkanKPAI",
  "labels": ["IDENTITY_ATTACK", "INSULT", "CALL_TO_ACTION"],
  "entities": [
    {"start": 0,  "end": 4,  "text": "KPAI",          "label": "TARGET_ORG"},
    {"start": 21, "end": 31, "text": "onta gurun",    "label": "ABUSIVE_EXPR"},
    {"start": 47, "end": 51, "text": "KPAI",          "label": "TARGET_ORG"},
    {"start": 52, "end": 57, "text": "totol",         "label": "ABUSIVE_EXPR"},
    {"start": 59, "end": 72, "text": "#BubarkanKPAI", "label": "DEMAND_EXPR"}
  ]
}
```

Hasil BIO: `KPAI/B-TARGET_ORG survei/O ke/O anak2/O onta/B-ABUSIVE_EXPR gurun/I-ABUSIVE_EXPR mungkin/O ,/O dasar/O KPAI/B-TARGET_ORG totol/B-ABUSIVE_EXPR ./O #BubarkanKPAI/B-DEMAND_EXPR`

---

## 6. Pipeline End-to-End UTS

Pipeline menggabungkan **alur spesifikasi proyek** dengan **tahapan corpus development dan text processing di PPT Minggu 2**.

```
┌──────────────────────────────────────────────────────────────────────────┐
│ TAHAP 0  Setup folder, requirements, seed                                │
├──────────────────────────────────────────────────────────────────────────┤
│ TAHAP 1  CORPUS DEVELOPMENT  (PPT slide 24–34)                           │
│          tujuan & scope → sumber & etika → collection → metadata         │
├──────────────────────────────────────────────────────────────────────────┤
│ TAHAP 2  DATA CLEANING  (PPT slide 76–81, Spek 4.3)                      │
│          text_raw ──► text_clean          → KelpX_dataset_clean.csv      │
├──────────────────────────────────────────────────────────────────────────┤
│ TAHAP 3  TEXT PROCESSING untuk analisis  (PPT slide 82–114)              │
│          tokenization + normalization ──► text_norm (EDA saja)           │
├──────────────────────────────────────────────────────────────────────────┤
│ TAHAP 4  SAMPLING 1.000 teks                                             │
├──────────────────────────────────────────────────────────────────────────┤
│ TAHAP 5  TAXONOMY multilabel + NER ──► konsultasi dosen ──► FREEZE v1.0  │
├──────────────────────────────────────────────────────────────────────────┤
│ TAHAP 6  ANNOTATION GUIDELINE (+ pilot 50 teks)  (PPT slide 47–51)       │
├──────────────────────────────────────────────────────────────────────────┤
│ TAHAP 7  ANOTASI (Label Studio)                                          │
├──────────────────────────────────────────────────────────────────────────┤
│ TAHAP 8  INTER-ANNOTATOR AGREEMENT                                       │
├──────────────────────────────────────────────────────────────────────────┤
│ TAHAP 9  ADJUDICATION ──► GOLD DATASET ──► validasi span                 │
│          → KelpX_dataset_annotation.jsonl                                │
├──────────────────────────────────────────────────────────────────────────┤
│ TAHAP 10 EDA multilabel + NER  → KelpX_dataset_EDA.ipynb                 │
├──────────────────────────────────────────────────────────────────────────┤
│ TAHAP 11 STREAMLIT DATASET EXPLORER → KelpX_dataset_explorer.py          │
├──────────────────────────────────────────────────────────────────────────┤
│ TAHAP 12 Laporan, export kode PDF, anggota.txt, ZIP                      │
└──────────────────────────────────────────────────────────────────────────┘
```

**Prinsip kunci:** ada tiga versi teks dan masing-masing punya tugas berbeda.

| Kolom | Isi | Dipakai untuk | Boleh berubah setelah anotasi? |
|---|---|---|---|
| `text_raw` | Teks asli persis dari xlsx | Jejak audit | Tidak pernah |
| `text_clean` | Teks diperbaiki secara teknis (encoding, spasi, URL), tetapi kapital, tanda baca, emoji, dan kata tetap utuh | **Anotasi multilabel & NER**, explorer | **Tidak** (offset span bergantung padanya) |
| `text_norm` | lowercase + normalisasi slang + huruf berulang dikurangi (+ stopword opsional) | EDA kata, word cloud, nanti fitur TF-IDF di UAS | Ya (turunan, bisa dibuat ulang) |

---

## 7. Rencana Detail per Tahap

### Tahap 0 — Setup proyek

- Buat struktur folder sesuai spesifikasi:
  ```
  project_uts/
  ├── dataset/        (original, clean, annotation)
  ├── notebook/       (KelpX_dataset_EDA.ipynb)
  ├── guideline/      (KelpX_annotation_guideline.pdf)
  ├── streamlit/      (KelpX_dataset_explorer.py)
  ├── report/         (KelpX_laporan_UTS.pdf)
  ├── code_pdf/       (KelpX_code_export_UTS.pdf)
  ├── resources/      (kamus_normalisasi.csv, stopwords, label_studio_config.xml)  ← tambahan
  ├── requirements.txt                                                              ← tambahan
  └── KelpX_anggota.txt
  ```
- `requirements.txt`: `pandas, numpy, openpyxl, xlrd, matplotlib, seaborn, scikit-learn, statsmodels, krippendorff, Sastrawi, spacy, streamlit, plotly`.
- Tetapkan `SEED = 42` di awal notebook.

### Tahap 1 — Corpus Development (PPT slide 24–34)

| Langkah PPT | Penerapan di proyek |
|---|---|
| (1) Tujuan & ruang lingkup | Tujuan: mendeteksi jenis abusif, sasaran, dan buktinya dalam komentar berita. Bahasa: Indonesia informal. Domain: komentar berita politik/sosial. Periode: 2019. Unit: 1 komentar. |
| (2) Sumber, izin, etika | Dataset publik dari GitHub penulis (sitasi paper IEEE 9034620). Tidak ada nama akun komentator. Username di teks di-*mask*. Konten kasar diberi peringatan. |
| (3–4) Collection & selection | Collection sudah dilakukan penulis asli. Seleksi: buang duplikat & teks tanpa huruf; kriteria inklusi = berbahasa Indonesia dan bisa dipahami. |
| (5) Struktur & metadata | Lihat skema kolom di Bagian 8.1 (`id`, `source`, `topic`, `orig_label`, flag, `annotation_status`, dst.). |
| (6) Anotasi | Diperlukan, karena kedua tugas butuh label (Tahap 5–9). |
| (7) Quality control korpus | Cek kosong, duplikat, format, distribusi tidak seimbang, dan data di luar scope → tabel QC sebelum/sesudah. |
| (8) Dokumentasi & versioning | Simpan `corpus_v1.0` (clean), `taxonomy_v1.0`, `guideline_v1.x`, `gold_v1.0`, dengan `CHANGELOG` singkat di laporan. |

**Output:** `KelpX_dataset_original.xlsx` (salinan persis) + tabel profil corpus di notebook.

### Tahap 2 — Data Cleaning (Spek 4.3, PPT slide 76–81)

Urutan langkah, dengan hasil uji prototipe pada data asli:

| # | Langkah | Remove / Replace / Preserve | Baris terdampak |
|---|---|---|---:|
| C1 | Perbaiki *mojibake* Mac Roman → UTF-8 | Replace | 348 |
| C2 | Dekode escape `%uXXXX` menjadi emoji | Replace | 106 |
| C3 | Hapus `�` (emoji yang hilang permanen); set flag `had_lost_emoji` | Remove | 267 |
| C4 | Hapus karakter tak terlihat (U+200B–U+200F, U+2060–U+2064, U+FEFF) | Remove | 48 |
| C5 | Unicode NFC | Replace | — |
| C6 | Hapus tag HTML/entitas (`<...>`, `&nbsp;`) | Remove | 0 |
| C7 | URL → `[URL]` | Replace | 2 |
| C8 | `@username` → `[USER]` | Replace | 2 |
| C9 | Seragamkan spasi, tab, dan baris baru menjadi 1 spasi; `strip()` | Replace | 540 |
| C10 | Buang teks tanpa huruf | Remove row | 5 |
| C11 | Buang duplikat (*case-insensitive*, setelah C1–C9), simpan yang pertama | Remove row | 22 |
| — | **Dipertahankan:** kapitalisasi, tanda baca (termasuk `!!!`), emoji, hashtag, huruf berulang, angka, kata sensor | Preserve | — |

**Hasil:** 3.184 → **3.157 baris** (label 1: 2.764 · label 2: 109 · label 3: 284). Sebanyak 932 baris berubah teksnya, dan semuanya tercatat lewat flag.

**Kenapa tidak lowercase/stemming di sini?** Karena NER membutuhkan kapitalisasi (`KPAI`, `Jokowi`), sedangkan deteksi abusif membutuhkan emoji, tanda seru, dan bentuk asli kata kasar (PPT slide 73–74). Normalisasi dilakukan terpisah di Tahap 3.

**Output:** `KelpX_dataset_clean.csv` + tabel log cleaning (langkah, jumlah baris, contoh sebelum → sesudah).

### Tahap 3 — Text Processing untuk analisis (PPT slide 82–114)

Tahap ini hanya membuat **kolom turunan** dan statistik. Teks anotasi tidak disentuh.

| Proses | Alat | Hasil | Dipakai di |
|---|---|---|---|
| Sentence tokenization | spaCy `id_nusantara` (sesuai PPT) atau regex `[.!?]+` sebagai fallback | `n_sentences` | EDA |
| Word tokenization dengan offset | Regex tokenizer (Lampiran) — konsisten dengan BIO | `tokens`, `n_tokens` | EDA NER, BIO |
| Case normalization | `lower()` | — | `text_norm` |
| Normalisasi singkatan/slang | `kamus_normalisasi.csv` (gabungan 2 kamus yang sudah dibersihkan) | `yg → yang`, `gk → tidak` | `text_norm` |
| Reduplikasi angka | regex `(\w+)2\b → \1-\1` | `anak2 → anak-anak` | `text_norm` |
| Huruf berulang | kurangi ≥3 huruf sama menjadi 1, lalu cek kamus; jangan pukul rata (ingat `massa ≠ masa`, PPT slide 103) | `enaaak → enak` | `text_norm` |
| Stopword removal (opsional) | Sastrawi, **tanpa menghapus negasi** (`tidak, bukan, gak, jangan`) | — | word frequency / word cloud |
| Stemming (opsional) | Sastrawi | — | analisis kosakata saja |
| POS tagging (opsional) | spaCy `id_nusantara` | pola `ADJ` di sekitar target | bahan diskusi guideline span |

### Tahap 4 — Sampling

- **Target:** 1.000 teks. Buffer 200 di atas minimum 800 untuk menutup teks yang nanti ditandai `UNANNOTATABLE`.
- **Metode yang direkomendasikan:** *stratified purposive sampling* (seed 42)
  1. Ambil **semua** label asli 2 (109) dan 3 (284) = 393 teks, supaya label abusif cukup terwakili.
  2. Ambil **607** teks label 1 secara stratified berdasarkan `topic`, sebanding dengan proporsi topik di corpus.
  3. Hasilnya ±39% teks berpotensi abusif. Distribusi topik tetap terwakili.
- **Justifikasi untuk laporan:** dengan sampling proporsional murni, hanya ±125 teks abusif yang akan terbagi ke 5 label abusif. Itu terlalu jarang untuk anotasi, IAA, dan training UAS. Perubahan proporsi ini disengaja, didokumentasikan, dan **dikonfirmasi ke dosen**.
- **Alternatif:** jika dosen meminta distribusi asli, gunakan sampling proporsional atau anotasi seluruh 3.157 teks (beban jauh lebih besar).
- Simpan `sample_ids.csv` dan tampilkan perbandingan distribusi label asli & topik (populasi vs sampel) di notebook.

### Tahap 5 — Taxonomy & konsultasi dosen

1. Mulai dari usulan Bagian 5. Baca bersama ±100 teks acak untuk menguji apakah label cukup dan tidak tumpang tindih.
2. Hitung perkiraan frekuensi tiap label pada 100 teks itu. Label dengan < 3% kemunculan ditinjau ulang.
3. **Konsultasi 1× ke dosen** (wajib): bawa tabel label, definisi, boundary rules, dan 10 contoh anotasi.
4. Revisi → **freeze `taxonomy_v1.0`**. Perubahan setelah ini wajib disertai alasan, catatan versi, dan persetujuan dosen.

### Tahap 6 — Annotation Guideline (`KelpX_annotation_guideline.pdf`)

Isi minimal (Spek 4.5) beserta rencana isinya:

1. Tujuan anotasi & gambaran tugas (multilabel + span)
2. Definisi 9 label multilabel: definisi, inclusion, exclusion, ≥2–3 contoh positif, negatif, dan ambigu
3. Definisi 7 tipe span dengan contoh
4. Aturan pemberian multilabel (Bagian 5.1)
5. Aturan boundary span: multiword, tanda baca, huruf berulang, hashtag, kata sensor, target vs ekspresi, larangan nested
6. Aturan kasus ambigu & konflik antarlabel, contohnya:
   - sarkasme pujian (`Hebat, KPAI memang paling pintar`) → `MOCKERY`, bukan `SUPPORT`
   - kutipan ucapan orang lain yang kasar → dianotasi hanya jika komentator mengadopsinya
   - `PKI` sebagai cap ke orang → `IDENTITY_ATTACK`; `PKI` sebagai sejarah → tidak
7. Aturan teks tidak relevan/tidak jelas → `UNANNOTATABLE`
8. Format output & cara kerja tool
9. Riwayat versi guideline

**Pilot annotation (PPT slide 50):** ketiga anggota menganotasi **50 teks yang sama** secara independen → hitung agreement → diskusikan perbedaan → revisi guideline `v1.1` → (ulangi bila κ < 0,4).

### Tahap 7 — Proses Anotasi

- **Tool:** **Label Studio** (open source, mendukung multilabel *Choices* dan span *Labels* dalam satu tugas, export JSON dengan offset karakter). Alternatif: Doccano.
- **Konfigurasi Label Studio (inti):**
  ```xml
  <View>
    <Text name="text" value="$text_clean"/>
    <Labels name="ner" toName="text">
      <Label value="TARGET_PERSON"/><Label value="TARGET_ORG"/><Label value="TARGET_GROUP"/>
      <Label value="ABUSIVE_EXPR"/><Label value="DEMAND_EXPR"/><Label value="ISSUE"/><Label value="PRAISE_EXPR"/>
    </Labels>
    <Choices name="labels" toName="text" choice="multiple">
      <Choice value="PROFANITY"/><Choice value="INSULT"/><Choice value="MOCKERY"/>
      <Choice value="IDENTITY_ATTACK"/><Choice value="THREAT"/><Choice value="CRITICISM"/>
      <Choice value="SUPPORT"/><Choice value="CALL_TO_ACTION"/><Choice value="NEUTRAL"/>
      <Choice value="UNANNOTATABLE"/>
    </Choices>
  </View>
  ```
- **Pembagian kerja (3 annotator, 1.000 teks):**

  | Subset | Jumlah teks | Dikerjakan oleh | Fungsi |
  |---|---:|---|---|
  | Overlap | 200 (20%, stratified agar semua label utama muncul) | A, B, dan C | Hitung IAA |
  | Single | 800 | dibagi rata ±267 per orang | Memperluas gold |
  | **Beban per orang** | **±467 teks** | | |

- Setiap annotator bekerja **independen**, tanpa melihat label asli maupun hasil anotasi anggota lain.
- Catat `annotator_id` & waktu export → bukti kontribusi (rubrik 5.5).

### Tahap 8 — Inter-Annotator Agreement (Spek 4.7)

| Jenis | Metode | Cara hitung |
|---|---|---|
| Multilabel per label | **Fleiss' κ** (3 annotator) per label, karena tiap label adalah keputusan biner ya/tidak | `statsmodels.stats.inter_rater.fleiss_kappa` |
| Multilabel agregat | **Krippendorff's α** dengan jarak MASI (untuk himpunan label) atau rata-rata κ per label | `krippendorff` / `nltk.metrics.agreement` |
| Pairwise | Cohen's κ per pasangan annotator (A–B, A–C, B–C) | `sklearn.metrics.cohen_kappa_score` |
| NER | Entity-level P/R/F1 antarannotator (salah satu dianggap acuan) dengan **exact match** dan **partial/overlap match** | fungsi sendiri (Bagian 9) |
| NER token | Cohen's κ pada tag BIO per token | `cohen_kappa_score` |

**Interpretasi (Landis & Koch):** < 0,20 lemah · 0,21–0,40 cukup · 0,41–0,60 sedang · 0,61–0,80 kuat · > 0,80 hampir sempurna.

**Laporan wajib:** nilai agregat + per label/per tipe entity, 5–10 contoh *disagreement*, penyebabnya, dan perubahan guideline yang dihasilkan.

### Tahap 9 — Adjudication → Gold Dataset

1. **Subset overlap:** label diterima dengan *majority vote* (≥2 dari 3). Span diterima bila ≥2 annotator sepakat exact match. Sisanya dibahas bersama memakai guideline.
2. **Subset single:** dilakukan *spot-check* 10% oleh anggota lain. Kesalahan sistematis diperbaiki.
3. Catat setiap keputusan di `adjudication_log.csv` (`id, field, annotator_A, B, C, keputusan, alasan`).
4. Jalankan `validate_spans()` pada seluruh data (offset cocok, tidak overlap, tidak ada spasi di tepi, label ada di taxonomy, `NEUTRAL` eksklusif). **Target 0 error.**
5. Buang `UNANNOTATABLE` dan laporkan jumlahnya. Pastikan sisa ≥ 800 teks.
6. Simpan `KelpX_dataset_annotation.jsonl` (format di Bagian 8.2).

### Tahap 10 — EDA (Spek 4.8)

| Multilabel | NER |
|---|---|
| Jumlah teks & distribusi panjang (karakter, kata) | Distribusi jumlah token per teks |
| Frekuensi tiap label (bar chart) | Jumlah span per tipe entity |
| Jumlah label per teks (*label cardinality* & *density*) | Jumlah span per teks |
| Co-occurrence label (heatmap) | Distribusi panjang span (token) |
| Correlation matrix antarlabel (phi/Pearson pada one-hot) | Top-20 span tersering per tipe |
| Imbalance ratio (max/min) + label langka | Dominasi tag `O` vs entity (% token) + imbalance entity |
| *Tambahan:* label baru vs label asli (crosstab), label per topik | *Tambahan:* tipe span per label multilabel (bukti hubungan A–B) |

Setiap grafik diikuti **interpretasi & implikasi ke UAS**. Contoh: "label `THREAT` hanya 2% → perlu *class weight* atau *threshold tuning*", atau "tag `O` ±85% → evaluasi harus entity-level, bukan token accuracy".

### Tahap 11 — Streamlit Dataset Explorer (Spek 4.9)

| Halaman | Isi |
|---|---|
| **Overview** | Ringkasan corpus (sumber, jumlah sebelum/sesudah cleaning), tabel taxonomy & definisi, kartu statistik |
| **Browse Data** | Tabel teks + label + span; klik baris → teks dengan **highlight span berwarna per tipe** |
| **Multilabel Analysis** | Bar chart label, histogram jumlah label per teks, heatmap co-occurrence, filter label (AND/OR) |
| **NER Analysis** | Distribusi tipe span, top span per tipe, contoh teks dengan highlight |
| **Filter/Search** | Cari kata kunci, filter label, tipe entity, topik, dan label asli |

Teknis: baca JSONL dari path relatif, gunakan `@st.cache_data`, highlight dengan HTML `<mark>` berdasarkan offset `start/end`, dan tampilkan peringatan "konten mengandung bahasa kasar".

### Tahap 12 — Laporan & Pengumpulan

- `KelpX_laporan_UTS.pdf`: pendahuluan → domain analysis (Bagian 3) → problem formulation → cleaning (tabel log) → sampling → taxonomy → guideline (ringkas) → proses anotasi → IAA → adjudication → EDA → explorer → keterbatasan & bias → rencana UAS.
- `KelpX_code_export_UTS.pdf`: export notebook + `.py` (misalnya `jupyter nbconvert --to pdf` atau webpdf).
- `KelpX_anggota.txt`: nama, NIM, peran, tautan repo.
- ZIP: `Projek_UTS_PBA_Kelas_KEL.zip`.
- **Uji akhir:** ekstrak ZIP ke folder baru → *Restart & Run All* notebook → `streamlit run` → harus jalan tanpa error.

---

## 8. Skema Data & Format File

### 8.1 `KelpX_dataset_clean.csv`

| Kolom | Tipe | Contoh | Keterangan |
|---|---|---|---|
| `id` | str | `ANC-0002` | ID unik permanen (urutan baris asli) |
| `text_raw` | str | `‚Å£‚Å£GOBLOK,ngapain beli...` | Teks asli |
| `text_clean` | str | `GOBLOK,ngapain beli...` | **Basis anotasi** |
| `text_norm` | str | `goblok ngapain beli...` | Teks ternormalisasi |
| `orig_label` | int | `3` | Label asli |
| `orig_label_name` | str | `abusive_offensive` | |
| `source` | str | `kompas/kaskus/detik` | Tidak tercatat per baris di dataset asli |
| `topic` | str | `indosat_sandi` | Hasil deteksi kata kunci |
| `year` | int | `2019` | |
| `language` | str | `id` | |
| `n_chars`, `n_tokens` | int | | |
| `had_mojibake`, `had_pct_escape`, `had_lost_emoji`, `had_url`, `had_user` | bool | | Jejak cleaning |
| `is_duplicate`, `no_letters` | bool | | Baris ini dibuang (disimpan di log, tidak di CSV final) |
| `in_sample` | bool | | Masuk 1.000 sampel anotasi |
| `annotation_status` | str | `not_annotated` / `annotated` / `adjudicated` | PPT slide 28 |

### 8.2 `KelpX_dataset_annotation.jsonl` (1 baris = 1 teks)

```json
{"id": "ANC-0002",
 "text": "<text_clean>",
 "labels": ["INSULT", "CRITICISM"],
 "entities": [{"start": 0, "end": 6, "text": "GOBLOK", "label": "ABUSIVE_EXPR"},
              {"start": 20, "end": 27, "text": "indosat", "label": "TARGET_ORG"}],
 "meta": {"orig_label": 3, "topic": "indosat_sandi", "split_group": "overlap",
          "annotators": ["A", "B", "C"], "adjudicated": true,
          "taxonomy_version": "1.0", "guideline_version": "1.1"}}
```

Field `start`/`end` ditambahkan di luar contoh spesifikasi (yang hanya berisi `text` + `label`) agar span tidak ambigu ketika kata yang sama muncul lebih dari sekali.

### 8.3 File pendukung

| File | Isi |
|---|---|
| `resources/kamus_normalisasi.csv` | `kata, kata_baku, sumber` (gabungan & sudah dibersihkan) |
| `resources/label_studio_config.xml` | Konfigurasi tool |
| `dataset/cleaning_log.csv` | Langkah, jumlah baris, contoh |
| `dataset/sample_ids.csv` | ID terpilih + strata |
| `dataset/annotations_raw/annotator_{A,B,C}.json` | Export mentah per annotator |
| `dataset/adjudication_log.csv` | Keputusan adjudication |
| `dataset/iaa_results.csv` | Nilai agreement per label/entity |

---

## 9. Daftar Fungsi yang Akan Dibuat

Fungsi-fungsi ini dikelompokkan per modul. Bisa ditaruh di notebook, atau di file `utils.py` yang di-*import* oleh notebook dan Streamlit agar logikanya tidak ganda.

### 9.1 Loading & corpus

| Fungsi | Input → Output | Tujuan |
|---|---|---|
| `load_original(path)` | path xlsx → DataFrame + kolom `id` | Membaca dataset asli secara reproducible |
| `assign_topic(text)` | str → str | Memberi topik berbasis kata kunci 15 berita |
| `profile_corpus(df)` | DataFrame → dict/tabel | Jumlah, distribusi label, panjang, noise (Bagian 3) |

### 9.2 Cleaning (Tahap 2)

| Fungsi | Tujuan |
|---|---|
| `fix_mojibake(s)` | Mac Roman → UTF-8; kembalikan teks asli jika gagal |
| `decode_pct_escape(s)` | `%uD83D%uDE02` → 😂 |
| `remove_lost_emoji(s)` | Hapus U+FFFD |
| `remove_invisible(s)` | Hapus karakter zero-width/invisible |
| `strip_html(s)` | Hapus tag & entitas HTML |
| `mask_url_user(s)` | URL → `[URL]`, `@user` → `[USER]` |
| `normalize_whitespace(s)` | Spasi/baris baru → satu spasi |
| `clean_text(s) -> (clean, flags)` | Menjalankan C1–C9 berurutan dan mengembalikan flag tiap langkah |
| `drop_invalid_and_duplicates(df)` | C10–C11 + log baris yang dibuang |
| `build_cleaning_log(df_before, df_after)` | Tabel statistik sebelum/sesudah |

### 9.3 Text processing (Tahap 3)

| Fungsi | Tujuan |
|---|---|
| `load_norm_dict(paths)` | Gabung & bersihkan 2 kamus (strip, buang `'`, selesaikan konflik) |
| `normalize_text(s, norm_dict)` | lowercase, reduplikasi, huruf berulang, kamus slang → `text_norm` |
| `remove_stopwords(tokens, keep_negation=True)` | Sastrawi tanpa menghapus negasi |
| `tokenize_with_offsets(text)` | Token + posisi karakter (dasar BIO) |
| `sentence_split(text)` | spaCy `id_nusantara` / fallback regex |

### 9.4 Sampling (Tahap 4)

| Fungsi | Tujuan |
|---|---|
| `stratified_purposive_sample(df, n=1000, seed=42)` | Ambil semua label 2–3 + label 1 stratified per topik |
| `compare_distribution(pop, sample)` | Tabel/grafik populasi vs sampel |
| `make_overlap_split(sample, n_overlap=200)` | Tentukan subset overlap & bagian tiap annotator |
| `export_to_labelstudio(df, path)` | JSON task untuk import ke Label Studio |

### 9.5 Anotasi, IAA, adjudication (Tahap 7–9)

| Fungsi | Tujuan |
|---|---|
| `parse_labelstudio_export(path, annotator)` | JSON Label Studio → format internal `{id, text, labels, entities}` |
| `multilabel_matrix(records, label_list)` | Ubah ke matriks one-hot (teks × label) per annotator |
| `fleiss_kappa_per_label(annots)` | κ Fleiss per label + rata-rata |
| `cohen_kappa_pairwise(annots)` | κ Cohen tiap pasangan annotator |
| `krippendorff_alpha_masi(annots)` | α untuk himpunan label |
| `span_agreement(a, b, mode="exact"/"partial")` | P/R/F1 entity-level antarannotator |
| `token_kappa(a, b)` | κ pada tag BIO |
| `list_disagreements(annots)` | Daftar teks yang berbeda untuk dibahas |
| `majority_vote(annots)` | Usulan label/span awal untuk adjudication |
| `validate_spans(record, taxonomy)` | Offset, substring, overlap, spasi tepi, label valid |
| `save_jsonl(records, path)` / `load_jsonl(path)` | Simpan & baca gold dataset |

### 9.6 EDA (Tahap 10)

| Fungsi | Tujuan |
|---|---|
| `spans_to_bio(text, entities)` | Konversi ke token + tag BIO |
| `label_stats(df)` | Frekuensi, cardinality, density, imbalance ratio |
| `cooccurrence_matrix(Y)` / `label_correlation(Y)` | Heatmap hubungan label |
| `entity_stats(records)` | Jumlah per tipe, per teks, panjang span, top span |
| `o_tag_ratio(bio_tags)` | Proporsi `O` vs entity |
| `label_entity_crosstab(records)` | Bukti hubungan multilabel ↔ NER |

### 9.7 Streamlit (Tahap 11)

| Fungsi | Tujuan |
|---|---|
| `load_data()` (cached) | Baca JSONL + CSV via path relatif |
| `render_highlight(text, entities)` | HTML `<mark>` berwarna per tipe span |
| `page_overview()`, `page_browse()`, `page_multilabel()`, `page_ner()`, `page_search()` | Lima halaman wajib |

---

## 10. Jadwal Kerja Usulan

Jadwal ini relatif terhadap minggu mulai pengerjaan. Sesuaikan dengan tanggal UTS resmi.

| Minggu | Kegiatan | Penanggung jawab | Output |
|---|---|---|---|
| 1 | Setup folder, cleaning, profil corpus, sampling, draf taxonomy | Semua (cleaning: Anggota 1) | `dataset_clean.csv`, draf taxonomy |
| 1–2 | Baca 100 teks, revisi taxonomy, **konsultasi dosen**, freeze v1.0 | Semua | `taxonomy_v1.0` |
| 2 | Guideline v1.0 + setup Label Studio + **pilot 50 teks** + revisi v1.1 | Anggota 2 (guideline), Anggota 3 (tool) | guideline v1.1 |
| 2–3 | Anotasi overlap 200 teks → IAA awal | Semua | `iaa_results.csv` |
| 3–4 | Anotasi single 800 teks | Semua (±267/orang) | export mentah |
| 4 | Adjudication + validasi span → gold | Semua | `dataset_annotation.jsonl` |
| 4–5 | EDA + Streamlit explorer | Anggota 1 (EDA), Anggota 3 (Streamlit) | notebook, `.py` |
| 5 | Laporan, export PDF, uji ZIP dari folder kosong | Anggota 2 | ZIP final |

---

## 11. Risiko & Mitigasi

| Risiko | Dampak | Mitigasi |
|---|---|---|
| Label abusif terlalu jarang | Model UAS gagal (seperti kode asli yang selalu memprediksi kelas 1) | Sampling purposive; gabungkan label langka setelah pilot; EDA imbalance |
| IAA rendah (κ < 0,4) | Nilai IAA & gold turun | Pilot + revisi guideline; contoh ambigu diperbanyak; diskusi kalibrasi |
| Offset span rusak | Gold tidak bisa dipakai untuk NER | `text_clean` dibekukan sebelum anotasi; `validate_spans()` wajib 0 error |
| Sarkasme & konteks berita tidak terlihat | Ketidaksepakatan annotator | Aturan "maksud keseluruhan" + kolom `topic` ditampilkan di tool |
| Kamus normalisasi salah (`krn → kamu`) | `text_norm` keliru | Bersihkan kamus; jangan dipakai untuk teks anotasi |
| Notebook tidak reproducible | Nilai rubrik 5.8 turun | Path relatif, seed, requirements, uji dari ZIP |
| Beban anotasi besar | Terlambat | Mulai overlap lebih dulu; target harian ±30 teks/orang |
| Paparan konten kasar | Kenyamanan annotator | Sesi anotasi pendek; peringatan di explorer |
| Bias topik (39% KPAI) | Model bias ke entitas KPAI | Laporkan sebagai keterbatasan; stratifikasi topik saat sampling & split UAS |

---

## 12. Checklist Pengumpulan UTS

| No | File | Status |
|---|---|---|
| 1 | `dataset/KelpX_dataset_original.xlsx` | ☐ |
| 2 | `dataset/KelpX_dataset_clean.csv` | ☐ |
| 3 | `dataset/KelpX_dataset_annotation.jsonl` | ☐ |
| 4 | `guideline/KelpX_annotation_guideline.pdf` | ☐ |
| 5 | `notebook/KelpX_dataset_EDA.ipynb` (cleaning + EDA, run-all sukses) | ☐ |
| 6 | `streamlit/KelpX_dataset_explorer.py` | ☐ |
| 7 | `report/KelpX_laporan_UTS.pdf` | ☐ |
| 8 | `code_pdf/KelpX_code_export_UTS.pdf` | ☐ |
| 9 | `KelpX_anggota.txt` | ☐ |
| 10 | ZIP `Projek_UTS_PBA_Kelas_KEL.zip` | ☐ |

**Checklist proses (spesifikasi bagian 10):**
- ☐ Domain analysis selesai
- ☐ Taxonomy multilabel & NER disetujui dosen (freeze)
- ☐ Guideline lengkap & konsisten
- ☐ Overlap annotation, IAA, dan adjudication selesai
- ☐ Gold dataset, EDA, dan explorer bisa dijalankan
- ☐ Notebook UTS run-all tanpa error

---

## 13. Jembatan ke UAS

Pekerjaan UTS menentukan kemudahan UAS. Hal yang sudah disiapkan sejak sekarang:

- **Split 70/15/15** dari gold JSONL dengan *iterative multilabel stratification* (`iterstrat`) dan cek duplikat antar-split.
- **Multilabel:** bandingkan ≥2 representasi (TF-IDF dari `text_norm` vs IndoBERT embedding/fine-tuning) dan ≥2 model (mis. Logistic Regression Binary Relevance vs Classifier Chain/IndoBERT). Metrik: micro/macro/weighted-F1, Hamming Loss, dan subset accuracy. **Jangan hanya accuracy**, karena kode asli sudah membuktikan accuracy 87% bisa berarti model tidak belajar apa pun.
- **NER:** baseline CRF + IndoBERT token classification; konversi span → BIO memakai `spans_to_bio()` yang sama; evaluasi entity-level (`seqeval`) beserta analisis boundary error vs label error.
- **Streamlit UAS:** menu Prediksi Multilabel, Prediksi NER, dan Integrated Analysis (konsistensi label ↔ span, sesuai tabel 5.3).
- **Fitting** TF-IDF/MI/scaler hanya pada train set, supaya leakage seperti di notebook asli tidak terulang.

---

## 14. Lampiran: Kode Inti yang Sudah Diuji

Kode di bawah sudah dijalankan pada dataset asli dan menghasilkan angka di Bagian 3.5 dan 7.

### 14.1 Cleaning

```python
import re, unicodedata
import pandas as pd
from pathlib import Path

BASE = Path.cwd().parent            # sesuaikan: folder project_uts/
RAW = BASE / "dataset" / "KelpX_dataset_original.xlsx"

def fix_mojibake(s: str) -> str:
    """UTF-8 yang terbaca sebagai Mac Roman (mis. 'ÔøΩ', 'üòÇ') -> teks asli."""
    try:
        return s.encode("mac_roman").decode("utf-8")
    except (UnicodeEncodeError, UnicodeDecodeError):
        return s

def decode_pct_escape(s: str) -> str:
    """'%uD83D%uDE02' -> '😂' (escape gaya JavaScript, termasuk surrogate pair)."""
    def rep(m):
        cps = [int(x, 16) for x in re.findall(r"%u([0-9A-Fa-f]{4})", m.group(0))]
        return "".join(map(chr, cps)).encode("utf-16", "surrogatepass").decode("utf-16", "replace")
    return re.sub(r"(?:%u[0-9A-Fa-f]{4})+", rep, s)

STEPS = [
    ("mojibake",     fix_mojibake),
    ("pct_escape",   decode_pct_escape),
    ("lost_emoji",   lambda s: s.replace("�", "")),
    ("invisible",    lambda s: re.sub("[​-‏⁠-⁤﻿]", "", s)),
    ("unicode_nfc",  lambda s: unicodedata.normalize("NFC", s)),
    ("html",         lambda s: re.sub(r"<[^>]+>|&\w+;", " ", s)),
    ("url",          lambda s: re.sub(r"https?://\S+|www\.\S+", "[URL]", s)),
    ("user",         lambda s: re.sub(r"(?<!\w)@\w+", "[USER]", s)),
    ("whitespace",   lambda s: re.sub(r"\s+", " ", s).strip()),
]

def clean_text(s: str):
    s, flags = str(s), {}
    for name, fn in STEPS:
        new = fn(s)
        flags[f"had_{name}"] = new != s
        s = new
    return s, flags

df = pd.read_excel(RAW).rename(columns={"Kalimat": "text_raw", "label": "orig_label"})
df.insert(0, "id", [f"ANC-{i+1:04d}" for i in range(len(df))])
res = df["text_raw"].map(clean_text)
df["text_clean"] = res.str[0]
df = pd.concat([df, pd.DataFrame(res.str[1].tolist())], axis=1)

df["no_letters"]   = ~df["text_clean"].str.contains(r"[A-Za-z]")
df["is_duplicate"] = df["text_clean"].str.lower().duplicated(keep="first")
df_clean = df[~df["no_letters"] & ~df["is_duplicate"]].reset_index(drop=True)
print(len(df), "->", len(df_clean))   # 3184 -> 3157
```

### 14.2 Tokenisasi ber-offset, BIO, dan validasi span

```python
TOKEN_RE = re.compile(r"#\w+|\[URL\]|\[USER\]|\w+(?:[-']\w+)*|[^\w\s]")

def tokenize_with_offsets(text):
    return [(m.group(), m.start(), m.end()) for m in TOKEN_RE.finditer(text)]

def spans_to_bio(text, entities):
    toks = tokenize_with_offsets(text)
    tags = ["O"] * len(toks)
    for ent in sorted(entities, key=lambda e: e["start"]):
        inside = [i for i, (_, s, e) in enumerate(toks) if s >= ent["start"] and e <= ent["end"]]
        for k, i in enumerate(inside):
            tags[i] = ("B-" if k == 0 else "I-") + ent["label"]
    return [t for t, _, _ in toks], tags

def validate_spans(rec, allowed_labels=None):
    errs = []
    for e in rec["entities"]:
        if rec["text"][e["start"]:e["end"]] != e["text"]:
            errs.append(("offset_mismatch", e))
        if e["text"] != e["text"].strip():
            errs.append(("whitespace_boundary", e))
        if allowed_labels and e["label"] not in allowed_labels:
            errs.append(("invalid_label", e))
    ents = sorted(rec["entities"], key=lambda e: e["start"])
    for a, b in zip(ents, ents[1:]):
        if b["start"] < a["end"]:
            errs.append(("overlap", a, b))
    return errs
```

---

### Referensi

- Spesifikasi Proyek UTS–UAS PBA Gasal 2026/2027 (Indriasari & Purnomo).
- PPT Minggu 2: *Pengembangan Korpus, Pemrosesan Teks, dan Pemahaman Teks*.
- Repo & paper dataset: *Abusive Language Detection on Indonesian Online News Comments*, IEEE Xplore doc. 9034620.
- Kamus alay (dipakai kode asli): github.com/okkyibrohim/id-multi-label-hate-speech-and-abusive-language-detection
- Landis, J. R. & Koch, G. G. (1977). *The Measurement of Observer Agreement for Categorical Data*.
- Library: pandas, scikit-learn, statsmodels, krippendorff, Sastrawi, spaCy (`id_nusantara`), Label Studio, Streamlit.
