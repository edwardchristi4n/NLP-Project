# PRD & Rencana Pengelolaan Dataset — Proyek UTS Pemrosesan Bahasa Alami

**Judul proyek:** Klasifikasi Sentimen Publik terhadap Polemik **KPAI vs PB Djarum** pada Komentar Berita Online
**Dataset sumber:** *Abusive Language Detection on Indonesian Online News Comments* (`id_abusive_news_comment`, dataset No. 7 di daftar resmi)
**Mata kuliah:** Pemrosesan Bahasa Alami (INFT41603) — Gasal 2026/2027
**Acuan:** `Spesifikasi_Proyek_UTS_UAS_PBA_2026_2027_Final_v2.pdf` + PPT Minggu 2 *Corpus Development, Processing and Understanding Text*
**Versi dokumen:** **v0.2** — sudah disesuaikan dengan hasil konsultasi dosen dan kesepakatan kelompok (domain tunggal, fokus sentimen). Taxonomy di-*freeze* menjadi v1.0 setelah *pilot annotation*.

> Catatan penamaan: semua file memakai awalan `KelpX_`. Ganti `X` dengan nomor kelompok.

### Riwayat perubahan
| Versi | Perubahan utama |
|---|---|
| v0.1 | Seluruh 15 berita; 9 label (jenis abusif + sikap); 7 tipe span (target + ekspresi) |
| **v0.2** | **Satu domain: KPAI vs PB Djarum** (saran dosen). **Fokus: sentimen positif/negatif** terhadap KPAI dan PB Djarum + `ABUSIVE`. **6 label multilabel. NER murni 4 tipe entitas** (`GOV_ORG`, `NONGOV_ORG`, `PERSON`, `GROUP`). Sampling 1.000 teks dari domain. |

---

## Daftar Isi
1. [Ringkasan Singkat](#1-ringkasan-singkat)
2. [Isi Folder & Struktur Proyek](#2-isi-folder--struktur-proyek)
3. [Analisis Dataset](#3-analisis-dataset)
4. [PRD (Product Requirements Document)](#4-prd-product-requirements-document)
5. [Skema Label: Multilabel Sentimen](#5-skema-label-multilabel-sentimen)
6. [Skema NER](#6-skema-ner)
7. [Hubungan Multilabel ↔ NER](#7-hubungan-multilabel--ner)
8. [Pipeline End-to-End UTS](#8-pipeline-end-to-end-uts)
9. [Rencana Detail per Tahap](#9-rencana-detail-per-tahap)
10. [Skema Data & Format File](#10-skema-data--format-file)
11. [Daftar Fungsi](#11-daftar-fungsi)
12. [Jadwal Kerja](#12-jadwal-kerja)
13. [Risiko & Mitigasi](#13-risiko--mitigasi)
14. [Checklist Pengumpulan UTS](#14-checklist-pengumpulan-uts)
15. [Jembatan ke UAS](#15-jembatan-ke-uas)

---

## 1. Ringkasan Singkat

| Hal | Isi |
|---|---|
| **Tujuan** | Mengklasifikasi **sentimen** komentar publik terhadap **KPAI** dan **PB Djarum** (positif/negatif), sekaligus mendeteksi komentar **abusif**, pada polemik audisi bulu tangkis 2019. |
| **Data** | 3.184 komentar berita (Kompas, Kaskus, Detik) dari 15 berita 2019 → cleaning → **3.157** teks → domain KPAI vs PB Djarum → **1.675** teks. |
| **Tugas A — Multilabel** | 6 label: `KPAI_POSITIVE`, `KPAI_NEGATIVE`, `PBDJARUM_POSITIVE`, `PBDJARUM_NEGATIVE`, `ABUSIVE`, `NEUTRAL`. Satu komentar boleh punya beberapa label. |
| **Tugas B — NER** | 4 tipe entitas: `GOV_ORG` (lembaga negara, mis. KPAI), `NONGOV_ORG` (organisasi non-pemerintah, mis. PB Djarum), `PERSON`, `GROUP` (kelompok orang, mis. anak-anak, atlet). |
| **Hubungan A–B** | NER menunjukkan **siapa** yang dibicarakan; multilabel menunjukkan **sikap** terhadapnya. `KPAI_*` ↔ `GOV_ORG`, `PBDJARUM_*` ↔ `NONGOV_ORG`. |
| **Target anotasi** | **1.000 teks** dari domain (minimal wajib 800): semua 188 teks calon abusif + 812 teks lain distratifikasi per utas berita. |
| **Status** | ✅ cleaning, seleksi domain, sampling, file tugas Label Studio, notebook, Streamlit, draf guideline v0.2 · ⬜ pilot, anotasi, IAA, gold, EDA final, laporan |

**Contoh dari corpus:**
```
Teks  : PB Djarum tuh bukan berjasa cari bibit aja tapi pembiayaan sponsor sampe jadi atlet sama pelatih juga..
        sayang kemampuan otaknya orang2 KPAI terbatas..
NER   : [PB Djarum]NONGOV_ORG ... [atlet]GROUP ... [KPAI]GOV_ORG
Label : [PBDJARUM_POSITIVE, KPAI_NEGATIVE, ABUSIVE]
```

---

## 2. Isi Folder & Struktur Proyek

```
Project_NLP/
├── Spesifikasi_Proyek_UTS_UAS_PBA_2026_2027_Final_v2.pdf   ← aturan, output wajib, rubrik
├── PRD_Rencana_Dataset_UTS_NLP.md                          ← dokumen ini
├── summarize.md                                            ← ringkasan singkat
├── Indonesian-Online-News-Comments/                        ← repo asli dataset (referensi, tidak diubah)
│   ├── Dataset/  (xlsx dataset + 2 kamus singkatan)
│   └── Code/     (7 notebook lama: crawling, BoW, MI, KNN/NB/SVM)
└── project_uts/                                            ← proyek kita (struktur spesifikasi 4.10)
    ├── dataset/
    │   ├── KelpX_dataset_original.xlsx        [WAJIB 1]
    │   ├── KelpX_dataset_clean.csv            [WAJIB 2]  semua 3.157 teks + kolom in_domain
    │   ├── KelpX_dataset_annotation.jsonl     [WAJIB 3]  dibuat setelah anotasi
    │   └── pendukung/                          log cleaning, sampel, kamus, file Label Studio, IAA
    ├── notebook/KelpX_dataset_EDA.ipynb       [WAJIB 5]  mandiri (semua fungsi di dalam)
    ├── guideline/
    │   ├── KelpX_annotation_guideline.pdf     [WAJIB 4]  (draf v0.2)
    │   ├── KelpX_annotation_guideline.md      sumber PDF
    │   ├── KelpX_taxonomy.json                daftar label & entitas (dibaca notebook + Streamlit)
    │   └── label_studio_config.xml            konfigurasi tool anotasi
    ├── streamlit/KelpX_dataset_explorer.py    [WAJIB 6]
    ├── report/                                [WAJIB 7]  (kerangka sudah ada)
    ├── code_pdf/                              [WAJIB 8]
    ├── KelpX_anggota.txt                      [WAJIB 9]
    ├── README.md, requirements.txt
```

### Fungsi file dari repo asli

| File | Fungsi untuk kita |
|---|---|
| `Abusive Language Detection ... .xlsx` | **Corpus utama**, disalin menjadi `KelpX_dataset_original.xlsx`. |
| `Acronym Words and Typos .xlsx` (698) + `kumpulan_singkatan_dan_kata_dasar.xls` (214) | Digabung dan dibersihkan menjadi `kamus_normalisasi.csv` (758 entri) untuk kolom `text_norm`. |
| 7 notebook `Code/` | **Tidak dipakai ulang.** Diambil pelajarannya: cleaning terlalu agresif (emoji dan tanda baca dibuang), *data leakage* (MI dihitung sebelum split), dan akurasi 87% yang menyesatkan karena model selalu menebak kelas mayoritas (macro-F1 ≈ 0,31). |

---

## 3. Analisis Dataset

### 3.1 Identitas
| Aspek | Temuan |
|---|---|
| Sumber | Kolom komentar Kompas, Kaskus, dan Detik; paper IEEE Xplore 9034620 |
| Periode | 2019 |
| Bahasa | Indonesia **informal**: singkatan (`yg`, `gak`), Betawi/Jawa (`ente`, `utekmu`), slang (`wkwk`, `gan`) |
| Kolom | `Kalimat` (teks), `label` (1 tidak abusif · 2 abusif tidak ofensif · 3 abusif & ofensif) |
| Metadata | Tidak ada ID, tanggal, atau sumber per baris |
| Lisensi | Tidak ada file LICENSE → hanya untuk akademik dan wajib sitasi |

### 3.2 Seluruh dataset (sebelum dipilih domain)

| Label asli | Jumlah | % |
|---|---:|---:|
| 1 tidak abusif | 2.789 | 87,6 |
| 2 abusif tidak ofensif | 110 | 3,5 |
| 3 abusif & ofensif | 285 | 9,0 |

Panjang teks: median 14 kata, maksimum 486 kata.

### 3.3 Noise dan cara penanganannya

| Noise | Baris | Contoh | Penanganan |
|---|---:|---|---|
| *Mojibake* (UTF-8 terbaca sebagai Mac Roman) | 348 | `ÔøΩ`, `üòÇ` | dipulihkan → 😂 |
| Emoji hilang permanen | 267 | `�` | dihapus (dicatat di flag) |
| Escape `%uXXXX` | 106 | `%uD83D%uDE02` | didekode → 😂 |
| Karakter tak terlihat | 48 | U+2063 sebelum `GOBLOK` | dihapus |
| Spasi/baris baru berlebih | 540 | `\n\n` | diseragamkan |
| URL / username | 2 / 2 | `www...`, `@user` | `[URL]` / `[USER]` |
| Duplikat (case-insensitive) | 22 | `#BubarkanKPAI` ×8 | dibuang (label tidak pernah bertentangan) |
| Tanpa huruf | 5 | `:(`, `1111` | dibuang |
| Kata sensor/samaran | ±20 | `beg0`, `b*rak`, `bgsd` | **dipertahankan** |
| Huruf & tanda baca berulang | ratusan | `Palsuuuu`, `!!!` | **dipertahankan** (sinyal emosi) |

**Hasil cleaning: 3.184 → 3.157 teks.**

### 3.4 Masalah label asli (alasan anotasi ulang)
1. **Single-label**, padahal spesifikasi meminta multilabel.
2. **Tidak ada informasi sentimen atau target**, hanya abusif atau tidak.
3. **Ada label salah.** Contohnya `Mulutmu sampah!!!` dan `Lembaga sampah. Bubar aja luh.` dilabeli "tidak abusif" (23 teks berkata kasar berlabel 1).
4. **Sangat tidak seimbang** (25:1).

### 3.5 Pemilihan domain: KPAI vs PB Djarum

Hasil konsultasi dosen: pilih **satu domain spesifik**. Topik ditentukan dari **posisi baris**, karena dataset tersusun berurutan per berita dan banyak komentar tidak menyebut topiknya.

| Topik berita | Jumlah teks | ≥ 800? |
|---|---:|---|
| **KPAI vs PB Djarum** | **1.821** | ✅ |
| Revisi UU KPK | 321 | ❌ |
| Tarif tol / Fadli Zon | 289 | ❌ |
| Indosat / Sandi | 191 | ❌ |
| HTI / Pemprov DKI | 151 | ❌ |
| Esia, Luna Maya, KPU, SBY, Ahmad Dhani, lainnya | < 140 masing-masing | ❌ |

Dari 1.821 teks di utas KPAI, **146** dibuang karena hanya membahas KPI atau sensor SpongeBob (berita lain yang terselip di rentang baris yang sama).

**Corpus domain final: 1.675 teks**, dalam 3 utas berurutan:

| Utas (`thread`) | Rentang baris | Jumlah |
|---|---|---:|
| 1 | ANC-1118 – ANC-1461 | 267 |
| 2 | ANC-1466 – ANC-2007 | 470 |
| 3 | ANC-2235 – ANC-3184 | 938 |

| Label asli di domain | Jumlah | % |
|---|---:|---:|
| 1 tidak abusif | 1.487 | 88,8 |
| 2 | 42 | 2,5 |
| 3 | 146 | 8,7 |

Panjang teks domain: median 16 kata, rata-rata 21,9 kata.

### 3.6 Bukti dari corpus untuk skema label & NER
Hitungan kata kunci di domain (perkiraan; dipakai sebagai bukti awal, bukan label):

| Hal | Teks yang memuat | Implikasi |
|---|---:|---|
| `kpai` | 932 | target sentimen utama → label `KPAI_*`, entitas `GOV_ORG` |
| `djarum`/`jarum` | 421 | target sentimen kedua → label `PBDJARUM_*`, entitas `NONGOV_ORG` |
| `anak`/`anak2`/`anak-anak` | 501 | kelompok yang paling dibicarakan → entitas `GROUP` |
| `atlet`/`pemain`/`bibit` | 146 | kelompok kedua → `GROUP` |
| `pemerintah`, `KPI`, `Komnas` | 45 / 34 / 24 | lembaga negara lain → `GOV_ORG` |
| `Susanto`, `Jokowi` | 15 / 13 | tokoh → `PERSON` |
| `bubar` (termasuk #BubarkanKPAI) | ±175 | sentimen negatif ke KPAI sangat banyak |
| ekspresi positif vs negatif | ±149 vs ±367 | opini publik berat sebelah → imbalance label |
| teks abusif (label asli 2+3) | 188 (11%) | label `ABUSIVE` |

Isu yang paling sering dibahas: rokok/iklan/logo (±450), peran Djarum (±420), prestasi bulu tangkis (±370), audisi/pembinaan (±250), kinerja KPAI (±220), eksploitasi anak (±115). Isu-isu ini menjadi **kategori yang termasuk** dalam setiap label (Bagian 5).

---

## 4. PRD (Product Requirements Document)

### 4.1 Latar belakang
Pada 2019, KPAI menilai Audisi Umum Beasiswa Bulu Tangkis PB Djarum sebagai eksploitasi anak karena memakai brand rokok, dan PB Djarum menghentikan audisinya. Polemik ini memicu ribuan komentar yang berisi dukungan, penolakan, dan makian. Dataset asli hanya menandai abusif atau tidak, sehingga **tidak bisa menjawab bagaimana sikap publik terhadap masing-masing pihak**.

### 4.2 Pernyataan masalah
1. **Task A — Multilabel (level teks):** *Bagaimana sikap komentar ini terhadap KPAI dan terhadap PB Djarum (positif/negatif), dan apakah bahasanya abusif?*
2. **Task B — NER (level kata):** *Entitas apa yang dibicarakan: lembaga negara, organisasi non-pemerintah, orang, atau kelompok orang?*
3. **Hubungan:** NER mengenali **siapa** yang dibicarakan, multilabel menentukan **sikap** terhadapnya (Bagian 7).

### 4.3 Tujuan UTS
| ID | Tujuan | Ukuran keberhasilan |
|---|---|---|
| G1 | Corpus domain bersih & terdokumentasi | Statistik cleaning + seleksi domain tersedia; setiap baris bisa dilacak lewat `id` |
| G2 | Taxonomy jelas & disetujui | 6 label + 4 entitas; definisi, kategori, contoh +/−/ambigu; di-*freeze* v1.0 setelah pilot |
| G3 | Gold dataset | ≥ 800 teks (target 1.000), validasi span 0 error |
| G4 | Anotasi konsisten | Fleiss' κ ≥ 0,60 untuk label utama; entity-F1 antarannotator ≥ 0,80 |
| G5 | EDA bermakna | Semua item spesifikasi 4.8 + interpretasi (sentimen dominan, imbalance) |
| G6 | Dataset explorer | Streamlit 5 menu tanpa error |
| G7 | Reproducible | Notebook *Restart & Run All* sukses dari folder ZIP |

### 4.4 Di luar cakupan UTS
- Melatih model (itu UAS).
- Menganotasi topik selain KPAI vs PB Djarum.
- Stemming atau stopword pada teks anotasi (hanya untuk kolom analisis `text_norm`).

### 4.5 Kebutuhan fungsional
| ID | Kebutuhan | Output |
|---|---|---|
| FR-01 | Muat dataset asli, beri ID `ANC-0001…` | `df_raw` |
| FR-02 | Cleaning C1–C11 dengan log per langkah | `KelpX_dataset_clean.csv`, `cleaning_log.csv`, `removed_rows.csv` |
| FR-03 | Tentukan berita asal dari posisi baris; pilih domain KPAI vs PB Djarum | kolom `segment`, `thread`, `in_domain` |
| FR-04 | Tiga versi teks: `text_raw`, `text_clean` (basis anotasi), `text_norm` (analisis) | kolom CSV |
| FR-05 | Sampling 1.000 teks dari domain + pembagian annotator | `sample_ids.csv`, `labelstudio_tasks_{A,B,C}.json` |
| FR-06 | Guideline anotasi | `KelpX_annotation_guideline.pdf` |
| FR-07 | Anotasi multilabel + NER dalam satu tool | export JSON per annotator |
| FR-08 | IAA multilabel & NER | `iaa_results.csv` |
| FR-09 | Adjudication + log | `adjudication_log.csv`, gold JSONL |
| FR-10 | Validasi entitas (offset, overlap, label valid, `NEUTRAL` eksklusif) | 0 error |
| FR-11 | EDA multilabel & NER | grafik + interpretasi |
| FR-12 | Streamlit explorer (Overview, Browse, Multilabel, NER, Filter/Search) | `KelpX_dataset_explorer.py` |

### 4.6 Kebutuhan non-fungsional
- **Reproducible:** path relatif, `SEED = 42`, `requirements.txt`, notebook mandiri.
- **Traceable:** baris gold → `id` → baris asli; semua keputusan cleaning, seleksi, dan adjudication tercatat.
- **Aman offset:** `text_clean` tidak berubah setelah anotasi dimulai (PPT slide 75).
- **Etis:** `@username` di-*mask*; peringatan konten kasar; sitasi dataset dan library.

### 4.7 Pemetaan ke rubrik UTS
| Kriteria | Bobot | Bagian dokumen |
|---|---:|---|
| Dataset Selection & Problem Formulation | 10% | 3, 4.2, 7 |
| Data Cleaning & Preparation | 10% | 3.3, 3.5, 9 (Tahap 2–4) |
| Multilabel Taxonomy Design | 15% | 5 |
| NER Taxonomy & Guideline | 15% | 6, 9 (Tahap 6) |
| Annotation Process & Adjudication | 15% | 9 (Tahap 7, 9) |
| Inter-Annotator Agreement | 10% | 9 (Tahap 8) |
| EDA Multilabel & NER | 15% | 9 (Tahap 10) |
| Dataset Explorer & Reproducibility | 10% | 9 (Tahap 11), 4.6 |

---

## 5. Skema Label: Multilabel Sentimen

Setiap komentar dinilai pada **tiga dimensi terpisah**, lalu diberi **semua** label yang berlaku:

| Dimensi | Label |
|---|---|
| Sikap terhadap **KPAI** | `KPAI_POSITIVE` / `KPAI_NEGATIVE` / (tidak bersikap) |
| Sikap terhadap **PB Djarum** | `PBDJARUM_POSITIVE` / `PBDJARUM_NEGATIVE` / (tidak bersikap) |
| **Nada bahasa** | `ABUSIVE` / (tidak abusif) |
| Bila tidak ada satu pun | `NEUTRAL` (eksklusif) |

### 5.1 Label dan kategori yang termasuk di dalamnya

#### `KPAI_POSITIVE` — mendukung KPAI
| Kategori yang termasuk | Contoh corpus |
|---|---|
| Membela KPAI atau tugasnya melindungi anak | `KPAI benar kok,mereka bertugas melindungi anak-anak,bukan masa depan mereka` |
| Menolak KPAI dibubarkan | `gak setuju kpai dibubarkan, kpai harus gantiin jarum rekrut pemain2 muda` |
| Setuju branding/iklan rokok dilarang | `Ban iklan rokok!!! Selamatkan generasi penerus dari rokok` (target tersirat) |
| Setuju audisi = eksploitasi anak | — |

**Bukan** `KPAI_POSITIVE`: pujian sarkastis seperti `dukung KPAI biar keliatan ada kerjaannya...` → `KPAI_NEGATIVE`.

#### `KPAI_NEGATIVE` — mengkritik/menolak KPAI
| Kategori yang termasuk | Contoh corpus |
|---|---|
| Kritik kinerja/keputusan | `KPAI gak becus....banyak anak2 dijalan terlantar, bisa nya cuma komen!` |
| Tuduhan cari sensasi/inkonsisten | `Sudah 50 tahun. Koq sekarang KPAI baru ribut ?` |
| Membandingkan dengan masalah anak lain yang diabaikan | `KPAI faker, eksploitasi dimananya? Lo mending ngurusin sinetron ga jelas aja` |
| Mengkhawatirkan prestasi bulu tangkis akibat KPAI | `Selamat tinggal medali emas Olimpiade dari cabang bulu tangkis, terima kasih kpd KPAI untuk jebloknya prestasi Indonesia` |
| Tuntutan pembubaran | `#BubarkanKPAI` · `KPAI? bubar aja` |
| Pertanyaan retoris/sarkasme | `KPAI sdh berbuat apa utk negara ini?` |

#### `PBDJARUM_POSITIVE` — mendukung PB Djarum
| Kategori yang termasuk | Contoh corpus |
|---|---|
| Apresiasi jasa pembinaan atlet | `Kalo nggk ada PB Djarum nggk mungkin lahir atlet bulutangkis yg hebat` |
| Menyayangkan audisi dihentikan | `Sangat disayangkan anak2 yang memang berniat di bidang bulutangkis` |
| Menilai logo/branding wajar | `ya wajar aja, namanya juga branding. terus mau duitnya, tp gk mau perusahaannya?` |
| Mendorong agar Djarum didukung | `Harusnya PB Djarum didukung, kalo perlu diwajibkan ...` |

#### `PBDJARUM_NEGATIVE` — mengkritik PB Djarum
| Kategori yang termasuk | Contoh corpus |
|---|---|
| Audisi dinilai promosi rokok terselubung | — (dicari saat pilot) |
| Kritik yayasan/CSR perusahaan rokok | `Dinegara ini hasil penjualan rokok yang notabene membunuh pemakainya dibuatkan yayasan sosial seolah olah ada nilai positif` |
| Kritik keputusan Djarum menghentikan audisi | — |

#### `ABUSIVE` — bahasa kasar
| Kategori yang termasuk | Contoh |
|---|---|
| Makian/kata kasar | `goblok`, `tolol`, `anjing`, `bangsat` |
| Hinaan kecerdasan/martabat | `sayang kemampuan otaknya orang2 KPAI terbatas..` · `gak punya otak` |
| Pelesetan atau sebutan merendahkan | `K pea I`, `onta`, `cebong` |
| Bentuk sensor/samaran | `beg0`, `b*rak`, `bgsd` |

**Bukan** `ABUSIVE`: kritik keras tanpa kata merendahkan (`KPAI tidak konsisten`).

#### `NEUTRAL` — tidak bersikap (eksklusif)
| Kategori yang termasuk | Contoh |
|---|---|
| Pertanyaan atau informasi tanpa sikap | `Jadi sebenarnya masalah brand /logo atau eksploitasi sih ??? Kmren baca krna eksploitasi ya klw gk salh` |
| Sikap hanya ke pihak lain (pemerintah, KPI, sinetron) | `Pemerintah aja diem, ngapain kita ribut2.` |

### 5.2 Aturan pemberian label
1. Sikap ke KPAI dan ke PB Djarum dinilai **terpisah**. `ABUSIVE` dinilai terpisah lagi.
2. **Target boleh tersirat**, karena semua komentar berasal dari polemik yang sama.
3. **Pro satu pihak ≠ otomatis anti pihak lain.** Beri label kedua pihak hanya jika keduanya terlihat.
4. Sarkasme dilabeli sesuai **maksud**, bukan kata-katanya.
5. `NEUTRAL` tidak boleh digabung dengan label lain.
6. Teks yang tidak bisa dipahami ditandai `UNANNOTATABLE` dan dikeluarkan dari gold.

Contoh kombinasi:

| Teks | Label |
|---|---|
| `Harusnya PB Djarum didukung` | `[PBDJARUM_POSITIVE]` |
| `KPAI GOBLOK!!!!!` | `[KPAI_NEGATIVE, ABUSIVE]` |
| `KPAI bisanya ngomong ga ada konstribusi, Djarum maju terus bangsa ini berterima kasih` | `[KPAI_NEGATIVE, PBDJARUM_POSITIVE]` |
| `PB Djarum tuh bukan berjasa cari bibit aja ... sayang kemampuan otaknya orang2 KPAI terbatas..` | `[PBDJARUM_POSITIVE, KPAI_NEGATIVE, ABUSIVE]` |

### 5.3 Perkiraan distribusi
Dari bukti kata kunci (3.6), diperkirakan `KPAI_NEGATIVE` dan `PBDJARUM_POSITIVE` dominan, sedangkan `KPAI_POSITIVE` dan `PBDJARUM_NEGATIVE` jarang. **Imbalance ini adalah temuan, bukan kesalahan**: opini publik memang berat sebelah. Imbalance dilaporkan di EDA dan ditangani di UAS (class weight, threshold per label). Label yang sangat jarang tetap dipertahankan kecuali dosen menyarankan penggabungan.

---

## 6. Skema NER

NER **hanya melabeli nama atau sebutan entitas**: siapa atau apa yang dibicarakan. Opini (`gak becus`), makian (`goblok`), dan kata kerja **tidak** masuk NER. Itu tugas multilabel. Skema tag: **BIO**, sehingga 4 tipe × 2 + `O` = 9 tag.

### 6.1 Tipe entitas dan teks yang masuk

| Tipe | Definisi | Teks yang **masuk** NER | Teks yang **tidak** masuk |
|---|---|---|---|
| `GOV_ORG` | lembaga negara/pemerintah | `KPAI`, `Kpai`, `KAPAI`, `K pea I` (pelesetan), `Komisi Perlindungan Anak Indonesia`, `KPI`, `pemerintah`, `Kemenpora`, `DPR`, `Komnas PA` | `negara` dalam arti umum |
| `NONGOV_ORG` | organisasi non-pemerintah: perusahaan, yayasan, klub, federasi, LSM | `PB Djarum`, `Djarum Foundation`, `Djarum`, `DJARUM`, `jarum` (jika berarti Djarum), `Sampoerna`, `PBSI`, `Lentera Anak` | `jarum` sebagai benda; `perusahaan rokok` (frasa umum) |
| `PERSON` | nama orang/tokoh | `Susanto`, `Jokowi`, `Pak Jokowi`, `Kevin`, `Marcus`, `Susi Susanti` | kata ganti `dia`, `mereka`, `lu`; sapaan `gan`, `bro` |
| `GROUP` | sebutan kelompok orang yang dibicarakan | `anak-anak`, `anak2`, `anak kecil`, `atlet muda`, `bibit atlet`, `pemain`, `orang tua`, `masyarakat`, `perokok` | `anak perusahaan`; kata `Anak` di dalam `Komisi Perlindungan Anak Indonesia` |

### 6.2 Aturan batas entitas
1. Nama resmi diambil **utuh**: `[Komisi Perlindungan Anak Indonesia]GOV_ORG`.
2. Gelar dan sapaan yang menempel ikut: `[Pak Jokowi]PERSON`.
3. Kata sifat bagian sebutan kelompok ikut (`[atlet muda]GROUP`). Kata penunjuk tidak ikut (`para [atlet]GROUP`).
4. Tanda baca di tepi tidak ikut: `KPAI!!!` → `[KPAI]GOV_ORG`.
5. Hashtag (`#BubarkanKPAI`) **tidak** dianotasi sebagai entitas.
6. Setiap kemunculan dianotasi. Entitas tidak boleh bertumpuk.

### 6.3 Contoh: teks mana yang masuk NER

```
Teks : KPAI gak becus....banyak anak2 dijalan terlantar, bisa nya cuma komen!
        ^^^^                    ^^^^^
NER  : [KPAI]GOV_ORG            [anak2]GROUP
BIO  : KPAI/B-GOV_ORG gak/O becus/O ..../O banyak/O anak2/B-GROUP dijalan/O terlantar/O ,/O ...
Label: [KPAI_NEGATIVE]
```

```
Teks : udah lah .... cari bibit dan kaderi sasi olah raga cabang apapun ranahnya pemerintah khususnya kemenpora
NER  : [pemerintah]GOV_ORG  [kemenpora]GOV_ORG
Label: [NEUTRAL]   ← ada entitas, tetapi tidak bersikap ke KPAI/PB Djarum
```

---

## 7. Hubungan Multilabel ↔ NER

| Label multilabel | Entitas terkait | Penjelasan |
|---|---|---|
| `KPAI_POSITIVE` / `KPAI_NEGATIVE` | `GOV_ORG` (KPAI) | sentimen ditujukan ke lembaga negara KPAI |
| `PBDJARUM_POSITIVE` / `PBDJARUM_NEGATIVE` | `NONGOV_ORG` (PB Djarum) | sentimen ditujukan ke organisasi swasta PB Djarum |
| `ABUSIVE` | `GOV_ORG` / `PERSON` / `GROUP` | sasaran makian |
| `NEUTRAL` | entitas apa saja | entitas disebut tanpa sikap |
| *(konteks)* | `GROUP` | siapa yang dibela atau dikhawatirkan (anak-anak vs atlet) |

Hubungan ini **tidak harus satu-ke-satu** (spesifikasi 4.4.3). Ada tiga kemungkinan yang dianalisis di EDA dan UAS:
- **Konsisten:** `KPAI_NEGATIVE` + entitas `GOV_ORG` "KPAI".
- **Sebagian:** `KPAI_NEGATIVE` tanpa entitas KPAI, karena targetnya tersirat (`Bubarkan saja!!`).
- **Kontradiktif:** `PBDJARUM_POSITIVE` tanpa entitas organisasi sama sekali → dicek apakah ada kesalahan.

---

## 8. Pipeline End-to-End UTS

```
┌───────────────────────────────────────────────────────────────────────────┐
│ TAHAP 0  Setup folder, requirements, seed                                 │
│ TAHAP 1  Corpus development & domain analysis   (PPT slide 24–34)         │
│ TAHAP 2  Data cleaning C1–C11                   (PPT slide 76–81)         │
│ TAHAP 3  Seleksi domain KPAI vs PB Djarum       (PPT slide 27 selection)  │
│ TAHAP 4  Text processing (kolom analisis)       (PPT slide 82–114)        │
│ TAHAP 5  Sampling 1.000 teks + bagi annotator                             │
│ TAHAP 6  Taxonomy + guideline → pilot 50 teks → revisi → FREEZE v1.0      │
│ TAHAP 7  Anotasi di Label Studio (NER + sentimen)                         │
│ TAHAP 8  Inter-annotator agreement                                        │
│ TAHAP 9  Adjudication → GOLD DATASET → validasi                           │
│ TAHAP 10 EDA multilabel + NER                                             │
│ TAHAP 11 Streamlit dataset explorer                                       │
│ TAHAP 12 Laporan, export kode PDF, ZIP                                    │
└───────────────────────────────────────────────────────────────────────────┘
```

**Tiga versi teks:**

| Kolom | Isi | Dipakai untuk |
|---|---|---|
| `text_raw` | teks asli | jejak audit (tidak pernah diubah) |
| `text_clean` | perbaikan teknis saja; kapital, emoji, dan tanda baca utuh | **anotasi**, explorer (dibekukan) |
| `text_norm` | lowercase + slang dinormalisasi | EDA kosakata, fitur TF-IDF di UAS |

---

## 9. Rencana Detail per Tahap

| # | Tahap | Cara / langkah | Output | Status |
|---|---|---|---|---|
| 0 | Setup | struktur folder spesifikasi 4.10, `requirements.txt`, seed 42 | folder `project_uts/` | ✅ |
| 1 | Corpus & domain analysis | muat xlsx, beri ID, cek label/panjang/noise, deteksi topik kata kunci | notebook §1 | ✅ |
| 2 | Cleaning | C1 mojibake → C2 `%u` → C3 `�` → C4 invisible → C5 NFC → C6 HTML → C7 URL → C8 user → C9 spasi → C10 tanpa huruf → C11 duplikat | `KelpX_dataset_clean.csv`, log | ✅ 3.157 teks |
| 3 | Seleksi domain | `assign_article_segment` (topik mayoritas ±12 baris) → `select_domain` (utas KPAI, buang KPI/SpongeBob) | kolom `segment`, `thread`, `in_domain` | ✅ 1.675 teks |
| 4 | Text processing | tokenisasi ber-offset, `text_norm` (lowercase, `anak2`→`anak-anak`, huruf berulang, kamus slang), stopword/stemming opsional | kolom analisis | ✅ |
| 5 | Sampling | semua 188 teks abusif + 812 teks lain per utas; 200 overlap + 267/267/266 | `sample_ids.csv`, `labelstudio_tasks_*.json` | ✅ |
| 6 | Taxonomy & guideline | draf v0.2 → **pilot 50 teks overlap** → hitung κ → revisi → freeze v1.0 | guideline PDF, `KelpX_taxonomy.json` | 🟡 draf |
| 7 | Anotasi | Label Studio: Langkah 1 blok entitas, Langkah 2 pilih label sentimen; kerja mandiri | `raw/annotator_{A,B,C}.json` | ⬜ |
| 8 | IAA | Fleiss κ & Krippendorff α per label; Cohen κ pairwise; NER entity-F1 exact/partial + κ token | `iaa_results.csv` | ⬜ (kode siap) |
| 9 | Adjudication & gold | mayoritas ≥2/3 otomatis; kasus *disagree* dibahas dan diedit di log; validasi 0 error; ≥800 teks | `KelpX_dataset_annotation.jsonl` | ⬜ (kode siap) |
| 10 | EDA | distribusi label, label/teks, co-occurrence, korelasi, imbalance; entitas per tipe/teks, panjang, top entitas, dominasi O, label × entitas | notebook §7–8 | ⬜ (kode siap) |
| 11 | Streamlit | Overview, Browse (highlight entitas), Multilabel, NER, Filter/Search | `KelpX_dataset_explorer.py` | ✅ |
| 12 | Pengumpulan | laporan, export kode PDF, anggota, uji ZIP | wajib #7–10 | ⬜ |

### Detail penting
- **Tahap 5 — alasan sampling purposive:** `ABUSIVE` hanya ±11% di domain. Mengambil semua teks abusif menaikkan proporsinya menjadi ±19% agar label itu cukup untuk IAA dan training. Distribusi utas tetap proporsional (16,8% / 28,3% / 54,9% vs populasi 15,9% / 28,1% / 56,0%).
- **Tahap 6 — pilot:** 50 teks overlap dianotasi ketiga anggota. Jika κ < 0,40, guideline direvisi dan pilot diulang.
- **Tahap 7 — konfigurasi tool:** `guideline/label_studio_config.xml`, `granularity="word"`, satu project per annotator. Label asli tidak ditampilkan ke annotator.
- **Tahap 8 — interpretasi κ (Landis & Koch):** ≤0,20 slight · 0,21–0,40 fair · 0,41–0,60 moderate · 0,61–0,80 substantial · >0,80 almost perfect.
- **Tahap 9 — gold hanya disimpan** bila validasi 0 error (offset cocok, entitas tidak bertumpuk, label valid, `NEUTRAL` eksklusif, label tidak kosong) dan jumlahnya ≥ 800 teks.

---

## 10. Skema Data & Format File

### 10.1 `KelpX_dataset_clean.csv` (3.157 baris)
| Kolom | Keterangan |
|---|---|
| `id` | `ANC-0001` … (urutan baris asli) |
| `text_raw`, `text_clean`, `text_norm` | tiga versi teks |
| `orig_label`, `orig_label_name` | label asli |
| `source`, `year`, `language` | metadata |
| `topic` | topik dari kata kunci |
| `segment` | berita asal dari posisi baris |
| `in_domain` | `True` jika termasuk domain KPAI vs PB Djarum |
| `thread` | nomor utas domain (1–3) |
| `n_chars`, `n_words`, `n_tokens`, `n_sentences` | statistik panjang |
| `had_mojibake`, `had_pct_escape`, `had_lost_emoji`, `had_invisible`, `had_url`, `had_user` | jejak cleaning |
| `annotation_status` | `not_annotated` / `in_sample` |

### 10.2 `KelpX_dataset_annotation.jsonl` (1 baris = 1 teks)
```json
{"id": "ANC-2630", "text": "KPAI gak becus....banyak anak2 dijalan terlantar, bisa nya cuma komen! Nyinyir! Giliran ada perusahaan yg mau membina bakat di nyinyirin! Kalo Djarum nyari bakat anak-anak di luar negeri nyinyir lagi!!!",
 "labels": ["KPAI_NEGATIVE", "PBDJARUM_POSITIVE"],
 "entities": [{"start": 0, "end": 4, "text": "KPAI", "label": "GOV_ORG"}, {"start": 25, "end": 30, "text": "anak2", "label": "GROUP"}, {"start": 143, "end": 149, "text": "Djarum", "label": "NONGOV_ORG"}, {"start": 162, "end": 171, "text": "anak-anak", "label": "GROUP"}],
 "meta": {"orig_label": 3, "thread": 3, "split_group": "single", "annotators": ["A"], "adjudicated": false, "taxonomy_version": "1.0", "guideline_version": "1.0"}}
```

### 10.3 File pendukung (`dataset/pendukung/`)
| File | Isi |
|---|---|
| `cleaning_log.csv`, `removed_rows.csv` | statistik C1–C11 dan baris yang dibuang |
| `kamus_normalisasi.csv`, `kamus_asli/` | kamus slang hasil gabungan + 2 kamus asli |
| `sample_ids.csv` | 1.000 ID terpilih + `thread` + strata + pembagian annotator |
| `label_studio/labelstudio_tasks_{A,B,C}.json` | task untuk diimpor ke Label Studio |
| `label_studio/raw/annotator_{A,B,C}.json` | export hasil anotasi |
| `adjudication_log.csv`, `iaa_results.csv` | keputusan adjudication & nilai agreement |

---

## 11. Daftar Fungsi

Semua fungsi didefinisikan **di dalam notebook**, di awal bagian tahapnya. Streamlit memuat salinan fungsi yang dibutuhkannya sendiri.

| Tahap | Fungsi | Kegunaan |
|---|---|---|
| Konfigurasi | `load_taxonomy`, `multilabel_names`, `entity_names`, `ensure_dirs` | membaca taxonomy dan menyiapkan folder |
| 1 Corpus | `load_original`, `assign_topic`, `add_metadata` | memuat data, ID, topik kata kunci, metadata |
| 2 Cleaning | `fix_mojibake`, `decode_pct_escape`, `remove_lost_emoji`, `remove_invisible`, `unicode_nfc`, `strip_html`, `mask_url`, `mask_user`, `normalize_whitespace`, `clean_text`, `apply_cleaning`, `drop_invalid_and_duplicates`, `build_cleaning_log`, `run_cleaning_pipeline` | C1–C11 + log |
| 3 Domain | `assign_article_segment`, `select_domain` | menentukan berita asal dari posisi baris; memilih domain KPAI vs PB Djarum |
| 4 Processing | `tokenize_with_offsets`, `tokenize`, `sentence_split`, `build_norm_dict`, `load_norm_dict`, `expand_reduplication`, `reduce_repeated_chars`, `normalize_text`, `remove_stopwords`, `stem`, `add_processing_columns` | tokenisasi, normalisasi, kamus |
| 5 Sampling | `stratified_purposive_sample`, `compare_distribution`, `make_overlap_split`, `export_to_labelstudio` | sampel 1.000, pembagian annotator, file task |
| 7/9 Anotasi | `parse_labelstudio_export`, `load_all_annotators`, `fix_span_whitespace`, `validate_spans`, `validation_report`, `spans_to_bio`, `bio_to_spans`, `build_adjudication_table`, `build_gold`, `gold_to_frame`, `entity_frame`, `save_jsonl`, `load_jsonl` | membaca hasil tool, validasi, BIO, adjudication, gold |
| 8 IAA | `multilabel_matrix`, `fleiss_kappa`, `fleiss_kappa_per_label`, `cohen_kappa_pairwise`, `krippendorff_alpha_per_label`, `span_agreement`, `span_agreement_pairwise`, `token_kappa`, `list_disagreements`, `full_iaa_report`, `interpret_kappa` | agreement multilabel & NER |
| 10 EDA | `label_stats`, `labels_per_text`, `cooccurrence_matrix`, `label_correlation`, `top_combinations`, `bio_frame`, `o_tag_ratio`, `entity_stats`, `top_spans`, `label_entity_crosstab` | statistik EDA |

---

## 12. Jadwal Kerja
| Minggu | Kegiatan | Output |
|---|---|---|
| 1 | ✅ cleaning, seleksi domain, sampling, draf taxonomy & guideline v0.2 | dataset clean, file task |
| 1–2 | Setup Label Studio, **pilot 50 teks**, hitung κ, revisi → freeze v1.0 | guideline v1.0 |
| 2–3 | Anotasi 200 teks overlap → IAA | `iaa_results.csv` |
| 3–4 | Anotasi 800 teks single (±267/orang, ±30 teks/hari) | export mentah |
| 4 | Adjudication + validasi → gold | `KelpX_dataset_annotation.jsonl` |
| 4–5 | EDA + interpretasi, screenshot Streamlit | notebook final |
| 5 | Laporan, export kode PDF, uji ZIP | ZIP final |

---

## 13. Risiko & Mitigasi
| Risiko | Mitigasi |
|---|---|
| `KPAI_POSITIVE` / `PBDJARUM_NEGATIVE` sangat jarang | dilaporkan sebagai temuan di EDA; UAS memakai class weight / threshold per label; konsultasi dosen bila < 30 contoh |
| Target tersirat membuat annotator berbeda pendapat | aturan "target boleh tersirat" + contoh di guideline; dibahas di pilot |
| Sarkasme | aturan "label sesuai maksud" + daftar contoh sarkasme |
| `jarum` (benda) vs Djarum | aturan konteks di guideline NER |
| IAA rendah | pilot + revisi guideline; diskusi kalibrasi |
| Offset entitas rusak | `text_clean` dibekukan; `validate_spans` wajib 0 error |
| Bias domain | ditulis sebagai keterbatasan: hanya komentator berita online 2019, satu polemik |
| Notebook tidak reproducible | path relatif, seed, notebook mandiri, uji dari ZIP |

---

## 14. Checklist Pengumpulan UTS
| No | File | Status |
|---|---|---|
| 1 | `dataset/KelpX_dataset_original.xlsx` | ✅ |
| 2 | `dataset/KelpX_dataset_clean.csv` | ✅ |
| 3 | `dataset/KelpX_dataset_annotation.jsonl` | ⬜ |
| 4 | `guideline/KelpX_annotation_guideline.pdf` | 🟡 draf v0.2 |
| 5 | `notebook/KelpX_dataset_EDA.ipynb` | ✅ (Bagian 5–8 menunggu anotasi) |
| 6 | `streamlit/KelpX_dataset_explorer.py` | ✅ |
| 7 | `report/KelpX_laporan_UTS.pdf` | ⬜ |
| 8 | `code_pdf/KelpX_code_export_UTS.pdf` | ⬜ |
| 9 | `KelpX_anggota.txt` | 🟡 |
| 10 | ZIP `Projek_UTS_PBA_Kelas_KEL.zip` | ⬜ |

Checklist proses: ☐ taxonomy disetujui dosen (freeze) · ☐ guideline lengkap · ☐ overlap, IAA, adjudication · ☐ gold, EDA, explorer jalan · ☐ notebook run-all tanpa error.

---

## 15. Jembatan ke UAS
- **Split 70/15/15** dari gold dengan *iterative multilabel stratification*; cek kebocoran duplikat antar-split.
- **Multilabel sentimen:** bandingkan ≥2 representasi (TF-IDF dari `text_norm` vs IndoBERT) dan ≥2 model (Logistic Regression Binary Relevance / Classifier Chain vs IndoBERT fine-tuning). Metrik: micro/macro/weighted-F1, Hamming Loss, subset accuracy, dan report per label. **Bukan accuracy saja.**
- **NER:** baseline CRF vs IndoBERT token classification (BIO 9 tag); evaluasi entity-level dengan `seqeval`; analisis boundary error vs label error (misalnya `GOV_ORG` vs `NONGOV_ORG`).
- **Integrated analysis:** prediksi multilabel + NER pada teks yang sama → cek konsisten / sebagian / kontradiktif (Bagian 7).
- **Streamlit UAS:** menu Prediksi Sentimen (probabilitas + threshold per label), Prediksi NER (highlight entitas), dan Integrated Analysis.

---

### Referensi
- Spesifikasi Proyek UTS–UAS PBA Gasal 2026/2027.
- PPT Minggu 2: *Pengembangan Korpus, Pemrosesan Teks, dan Pemahaman Teks*.
- Dataset & paper: *Abusive Language Detection on Indonesian Online News Comments*, IEEE Xplore 9034620.
- Landis, J. R. & Koch, G. G. (1977). *The Measurement of Observer Agreement for Categorical Data*.
- Library: pandas, scikit-learn, krippendorff, Sastrawi, matplotlib, seaborn, plotly, Streamlit, Label Studio.
