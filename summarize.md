# Ringkasan Proyek UTS NLP — Sentimen Publik pada Polemik KPAI vs PB Djarum

> Ringkasan singkat (v0.6 — cleaning ✅ dan normalisasi ✅ selesai; **berikutnya: sampling dan persiapan anotasi**, tim **KelpNLP**). Notebook kerja: `UTS_NLP/notebook/KelpNLP_dataset_EDA.ipynb`. Status kesiapan anotasi ada di **§6.6**. Rencana eksekusi siap-jalan ada di **§5**.
>
> Catatan: `PRD_Rencana_Dataset_UTS_NLP.md` belum diselaraskan dan masih memakai angka lama (domain 1.675, nama file `KelpX`). Bila berbeda, yang benar adalah dokumen ini dan notebook.

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

- **UTS:** membangun gold dataset (≥800 teks) melalui analisis domain (4.2), cleaning dan normalisasi (4.3), anotasi, IAA, adjudication, EDA, dan Streamlit explorer.
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
| Setelah cleaning (5 tanpa huruf + 22 duplikat dibuang) | 3.157 |
| **Corpus domain KPAI vs PB Djarum** (`in_domain = True`) | **1.515** |
| Di luar domain (tetap disimpan, `in_domain = False`) | 1.642 |
| **Target sampel untuk anotasi** | **1.000** (minimal wajib 800) |

**Corpus domain per utas** (batas baris sudah diverifikasi dengan membaca baris di sekitar batas):

| Utas | Berita asal | Rentang | Jumlah | Berlabel abusif (label asli 2–3) |
|---|---|---|---:|---:|
| 1 | no. 10 — KPAI Survei Ke Anak Yang Mana?? | ANC-1123 – ANC-1300 | 178 | 30 |
| 2 | no. 12 — #bubarkanKPAI Trending di Twitter | ANC-1599 – ANC-2007 | 400 | 49 |
| 3 | no. 15 — PB Djarum Hentikan Audisi Bulutangkis | ANC-2243 – ANC-3184 | 937 | 81 |
| | **Total** | | **1.515** | **160** |

> **Mengapa bukan 1.675 lagi?** Angka lama (1.675; utas 267/470/938) berasal dari seleksi berbasis kata kunci, dan masih memuat ±150 komentar berita **SpongeBob/KPI** (berita no. 11, ANC-1301–1598) yang letaknya di antara dua berita KPAI. Seleksi final memakai **batas baris tiap berita**, sehingga blok itu keluar seluruhnya. Notebook §4.2 masih menampilkan 1.836 sebagai *segmen awal* (sebelum cleaning, masih memuat blok SpongeBob); angka finalnya ditetapkan di §4.3.5.

**Mengapa domain KPAI vs PB Djarum?** Topik ini satu-satunya yang punya lebih dari 800 teks. Topik terbesar berikutnya, revisi UU KPK, hanya ±330 teks.

**Masalah dataset asli:** label hanya abusif atau tidak (single-label, tanpa sentimen atau target); ada label salah (`Mulutmu sampah!!!` berlabel "tidak abusif"); sangat tidak seimbang (±25:1); tidak ada metadata topik (topik ditentukan dari posisi baris, karena data tersusun per berita).

**Noise yang ditangani di cleaning (jumlah baris terdampak):** mojibake (348), escape `%u` (106), karakter hilang `�` (267), karakter tak terlihat (48), escape `\/` (9), URL (2), username (1), spasi/baris baru (540), teks tanpa huruf (5, dibuang), duplikat (22, dibuang). Yang **dipertahankan** di `text_clean`: kapital, emoji, tanda baca berulang (`!!!`), hashtag, huruf berulang, dan kata sensor (`beg0`, `K pea I`).

**Dugaan distribusi label** (indikasi kata kunci, dihitung pada 1.515 teks domain): menyerang KPAI 14,7%, membela PB Djarum 6,5%, vs membela KPAI hanya 0,9% dan mengkritik PB Djarum 1,8%. Opini publik berat ke pro-Djarum/anti-KPAI, jadi `KPAI_POSITIVE` dan `PBDJARUM_NEGATIVE` akan jarang. (Notebook §4.2.7 menampilkan angka segmen awal: 13,3% / 5,6% / 0,7% / 1,5%; polanya sama.)

---

## 3. Alur Kerja

| # | Tahap | Di mana | Status |
|---|---|---|---|
| 1 | Judul, deskripsi proyek | notebook — sel awal | ✅ |
| 2 | 4.2 Dataset Selection & Domain Analysis | notebook §4.2.1–4.2.8 | ✅ |
| 3 | 4.3 Data cleaning C1–C11 + seleksi domain berbasis batas baris + validasi | notebook §4.3.1–4.3.9 | ✅ 3.184 → 3.157; domain 1.515; `text_clean` dibekukan |
| 4 | 4.3 (lanjutan) Normalisasi → `text_norm` + validasi akhir | notebook §4.3.10–4.3.19 | ✅ selesai (lihat §6.5) |
| 5 | Sampling 1.000 teks domain + pembagian annotator + file task Label Studio | notebook §4.4 (baru, spesifikasi di §5 Tugas S1) | ⬜ **berikutnya — eksekusi pertama** |
| 6 | Guideline + taxonomy + konfigurasi Label Studio | `UTS_NLP/guideline/` (spesifikasi di §5 Tugas S2) | 🟡 draf v0.2 masih di `project_uts/` (nama `KelpX`); **belum ada di `UTS_NLP/`** |
| 7 | Pilot 50 teks → κ → revisi → freeze v1.0 | A, B, C | ⬜ |
| 8 | Anotasi penuh (Label Studio) | A, B, C | ⬜ |
| 9 | IAA (Fleiss/Cohen κ, Krippendorff α, entity-F1) | notebook | ⬜ |
| 10 | Adjudication & gold dataset | notebook | ⬜ |
| 11 | EDA multilabel & NER | notebook | ⬜ |
| 12 | Streamlit explorer | `streamlit/` | 🟡 ada versi di `project_uts/`, perlu disesuaikan |
| 13 | Laporan, export kode PDF, ZIP | `report/`, `code_pdf/` | ⬜ |

**Cara menjalankan:**
```bash
cd UTS_NLP
jupyter lab notebook/KelpNLP_dataset_EDA.ipynb        # Kernel → Restart & Run All
```
Notebook membuat ulang semua file di `dataset/`. Selama kodenya tidak diubah, hasilnya identik setiap kali dijalankan.

**Pemetaan bagian notebook (untuk dipilah saat menyusun laporan):**

| Bagian notebook | Isi | Bagian laporan / rubrik |
|---|---|---|
| §4.2.1–4.2.8 | sumber, bahasa, struktur, label asli, distribusi, masalah kualitas, potensi | Dataset & domain analysis (5.1) |
| §4.3.1–4.3.9 | pemeriksaan awal, C1–C11, seleksi domain, statistik, validasi cleaning | Data cleaning (5.2) |
| §4.3.10–4.3.19 | daftar proteksi, audit kamus, normalisasi, audit 20 sampel, validasi akhir | Data preparation / normalisasi (5.2) |

---

## 4. Library

| Library | Kegunaan |
|---|---|
| pandas, numpy | olah tabel, statistik, matriks |
| openpyxl, xlrd | baca `.xlsx` dan `.xls` (xlrd wajib untuk kamus `kumpulan_singkatan_dan_kata_dasar.xls`) |
| re, unicodedata, html, hashlib, itertools, json *(bawaan)* | regex cleaning/normalisasi, normalisasi Unicode, pengecekan file |
| Sastrawi | stopword & stemming Bahasa Indonesia (opsional; belum dipakai) |
| scikit-learn | Cohen's κ; model di UAS |
| krippendorff | Krippendorff's α |
| matplotlib, seaborn | grafik EDA & heatmap (dipakai di §4.2) |
| plotly, streamlit | grafik interaktif & aplikasi explorer |
| jupyterlab, nbconvert | menjalankan & export notebook |
| Label Studio *(tool)* | anotasi sentimen + NER |

---

## 5. Langkah Berikutnya (siap eksekusi — urutan wajib)

> Aturan eksekusi: kerjakan S1 → S2 → S3 berurutan. S4–S6 menunggu manusia (anotasi di Label Studio). Setelah anotasi dimulai, **kode cleaning tidak boleh diubah** (merusak offset NER); kode normalisasi/kamus masih boleh diperbaiki.

### S1. Sampling 1.000 teks (notebook §4.4 baru)
- Input: `dataset/KelpNLP_dataset_clean.csv` (kolom `text_clean`, `in_domain`, `thread`, `orig_label`), `SEED = 42`.
- Langkah: (a) tandai strata wajib = 160 teks `orig_label IN (2,3)` di domain + ±39 teks indikasi langka (regex pro-KPAI/anti-Djarum, catat pola di notebook); (b) dari 7 near-duplicate `#BubarkanKPAI` sisakan 1–2 varian; (c) sisa ±810 diambil acak proporsional per utas (±12% / 26% / 62%) dengan seed; (d) bagi menjadi 200 overlap (stratifikasi utas + pastikan memuat abusif dan langka; 50 di antaranya = pilot) + 800 single (267/267/266); (e) simpan `dataset/pendukung/sample_ids.csv` (kolom `id, thread, strata, split_group, annotator`) dan `dataset/pendukung/label_studio/labelstudio_tasks_{A,B,C}.json` berisi **`text_clean`** (bukan `text_norm`).
- Verifikasi: total 1.000; 200 overlap + 800 single; tiap utas proporsional; semua 160 abusif ikut; `sample_ids.csv` stabil bila notebook dijalankan ulang.

### S2. Guideline + taxonomy + konfigurasi (`UTS_NLP/guideline/`)
- Salin dari `project_uts/guideline/` lalu ganti `KelpX` → `KelpNLP`; tulis `KelompokNLP_…txt` (nama, NIM, peran A/B/C).
- Isi guideline v1.0-draf (Bahasa Indonesia, 7 bagian adaptasi): kontrol dokumen + glosari; ruang lingkup (`text_clean` dianotasi); skema 6 multilabel + 4 NER dengan definisi eksklusif + contoh korpus asli/ambigu; mekanik Label Studio (langkah 1 blok entitas → langkah 2 label, blind label asli); QA (pilot 50, κ ≥ 0,60 label utama, entity-F1 ≥ 0,80, κ < 0,40 = revisi + ulang); etika (`[USER]/[URL]`, peringatan konten kasar, skew = temuan); lampiran gold + kesalahan umum.
- Keputusan yang harus dikunci di guideline: `NEUTRAL` eksklusif; `UNANNOTATABLE` ≤5%; target boleh tersirat; sarkasme ikut maksud; `KPAI` dalam `#BubarkanKPAI` tidak dientitas; `jarum` benda ≠ Djarum; `K pea I`/`beg0` = proteksi + sinyal `ABUSIVE`.
- Output: `UTS_NLP/guideline/KelpNLP_annotation_guideline.md` (+ PDF wajib), `KelpNLP_taxonomy.json`, `label_studio_config.xml`.

### S3. Selaraskan PRD dengan angka final
- Di `PRD_Rencana_Dataset_UTS_NLP.md` ganti angka lama (domain 1.675, utas 267/470/938, 188 abusif) dengan angka final dokumen ini (domain **1.515**, utas **178/400/937**, **160** abusif). Tanpa ini laporan dan sidang membingungkan.

### S4–S6 (menunggu manusia)
4. Setup Label Studio → **pilot 50 overlap** → hitung κ → revisi → freeze v1.0. 5. Anotasi penuh → IAA → adjudication → gold (`KelpNLP_dataset_annotation.jsonl`, target ≥800). 6. EDA + Streamlit + laporan + export PDF + ZIP.

---

## 6. Klarifikasi Desain — Cleaning sampai Anotasi

### 6.1 Alur jumlah teks
1. **Cleaning: 3.184 → 3.157.** ✅ Semua baris bersih disimpan di `KelpNLP_dataset_clean.csv`; tidak ada pemotongan topik di tahap ini.
2. **Seleksi domain: 3.157 → 1.515.** ✅ Ditandai `in_domain = True` dengan `thread` 1/2/3 = 178 / 400 / 937. Baris di luar domain tidak dihapus (`thread = 0`).
3. **Sampling anotasi: 1.515 → 1.000.** ⬜ *Rencana, belum dijalankan.* Rencana lama ("188 abusif + 812 lain") tidak berlaku lagi karena teks berlabel abusif di domain hanya **160**. Rencana yang disesuaikan:
   - ambil **semua 160** teks berlabel abusif;
   - ambil teks berindikasi label langka (pro-KPAI / anti-Djarum; ±39 teks menurut kata kunci, sebagian tumpang tindih dengan yang abusif);
   - sisanya diambil acak **proporsional per utas** (±12% / 26% / 62%), *seed* tetap;
   - dibagi 200 overlap (A+B+C, untuk IAA) + 800 single (267/267/266);
   - diawali pilot 50 overlap → hitung κ → revisi → freeze v1.0.
   Target gold ≥800; 1.000 memberi cadangan untuk teks `UNANNOTATABLE`.

### 6.2 File output (satu clean CSV + penanda, bukan banyak CSV)
| File | Status | Isi |
|---|---|---|
| `dataset/KelpNLP_dataset_original.xlsx` | ✅ | 3.184 baris asli, tidak diubah |
| `dataset/KelpNLP_dataset_clean.csv` | ✅ | 3.157 baris × 23 kolom: `id`, `text_raw`, `text_clean`, `text_norm`, label asli, metadata, `topic`, `segment`, `in_domain`, `thread`, panjang teks, 7 penanda `had_*`, `annotation_status` |
| `dataset/pendukung/cleaning_log.csv` | ✅ | jumlah baris terdampak tiap langkah C1–C11 + contoh |
| `dataset/pendukung/removed_rows.csv` | ✅ | 27 baris yang dibuang + alasannya |
| `dataset/pendukung/kamus_normalisasi.csv` | ✅ | 758 entri: `kata`, `kata_baku`, `sumber` |
| `dataset/pendukung/normalization_log.csv` | ✅ | semua penggantian: `jenis`, `asal`, `hasil`, `jumlah` |
| `dataset/pendukung/sample_ids.csv`, `label_studio/labelstudio_tasks_{A,B,C}.json` | ⬜ | dibuat di tahap sampling |
| `dataset/pendukung/iaa_results.csv`, `adjudication_log.csv` | ⬜ | dibuat setelah anotasi |
| `dataset/KelpNLP_dataset_annotation.jsonl` | ⬜ | gold dataset (±1.000 teks) |

Tidak semua teks dianotasi karena keterbatasan waktu dan spesifikasi hanya mewajibkan ≥800.

### 6.3 Dampak distribusi skew pro-Djarum / anti-KPAI
- Indikasi kata kunci pada 1.515 teks domain: serang KPAI 14,7%, bela Djarum 6,5% vs bela KPAI 0,9%, kritik Djarum 1,8%. `KPAI_POSITIVE` dan `PBDJARUM_NEGATIVE` diprediksi langka.
- Keputusan: **biarkan natural, jangan dibalance paksa** — skew adalah temuan opini riil 2019, bukan cacat. Purposive sampling hanya menjamin support minimal label langka, bukan mengubah proporsi publik.
- Konsekuensi: IAA label langka tidak stabil (kappa paradox → lapor per-label κ + % agreement); model UAS bias ke mayoritas (wajib macro-F1, per-label PR, class weight / focal loss); waspadai korelasi semu `KPAI_NEGATIVE ≈ PBDJARUM_POSITIVE` dan `ABUSIVE ≈ KPAI_NEGATIVE`.

### 6.4 Cakupan 6 label multilabel + 4 tipe NER
- Multilabel (`KPAI_POSITIVE/NEGATIVE`, `PBDJARUM_POSITIVE/NEGATIVE`, `ABUSIVE`, `NEUTRAL` eksklusif) + `UNANNOTATABLE` bersifat exhaustive: semua komentar punya kombinasi valid. Sikap ke pihak lain (pemerintah, KPI, sinetron) sengaja masuk `NEUTRAL`.
- NER (`GOV_ORG`, `NONGOV_ORG`, `PERSON`, `GROUP` → 9 tag BIO) hanya untuk nama/sebutan; komentar tanpa entitas tetap valid (dominasi `O` normal).
- Proyeksi: bagus untuk `KPAI_NEGATIVE`, `PBDJARUM_POSITIVE`, `GOV_ORG`, `NONGOV_ORG`, `GROUP`; sedang untuk `ABUSIVE`/`NEUTRAL`; lemah untuk `KPAI_POSITIVE`, `PBDJARUM_NEGATIVE`, `PERSON`. Tidak perlu tambah label baru.
- Kriteria freeze v1.0: `UNANNOTATABLE` ≤5% dan tidak ada pola disagree "tidak ada label yang cocok" pada pilot 50.

### 6.5 Normalisasi `text_norm` — ✅ SELESAI (03-10-2026)

**Apa itu:** kolom turunan dari `text_clean` dengan ejaan kata umum yang diseragamkan. Dipakai untuk analisis kosakata di EDA dan fitur model (mis. TF-IDF) di UAS. **Bukan** teks yang dianotasi.

**Tiga versi teks:**

| Kolom | Isi | Dipakai untuk |
|---|---|---|
| `text_raw` | teks asli | jejak audit |
| `text_clean` | noise teknis diperbaiki; kapital, emoji, tanda baca utuh | **anotasi** dan offset NER (dibekukan) |
| `text_norm` | huruf kecil + ejaan kata umum diseragamkan | EDA kosakata, fitur model |

**Yang dikerjakan (notebook §4.3.10–4.3.19):**

| Bagian | Isi |
|---|---|
| 4.3.10 Daftar proteksi | 44 kata + 9 frasa entitas (`kpai`, `djarum`, `pb djarum`, `dpr`, `jokowi`, …), semua hashtag dan `@sebutan`, `[URL]`/`[USER]`, bentuk sensor/campuran huruf-angka (`beg0`, `b*rak`, `17T`) |
| 4.3.11 Audit kamus bawaan | 912 entri dari 2 kamus diperiksa: nilai kosong, kata ganda, 8 kata bernilai bertentangan, entri keliru |
| 4.3.12 Kamus final | 912 → **758** entri: saringan otomatis → buang kata proteksi → daftar buang (audit) → perbaikan nilai (27) → tambahan kata informal (79) → rantai diteruskan (`ajaa → aja → saja`) |
| 4.3.13 Fungsi normalisasi | per kata: huruf kecil → proteksi → reduplikasi → tawa → kamus → huruf berulang (+ cek kamus lagi) |
| 4.3.14 Menjalankan | pada semua 3.157 teks; setiap penggantian dicatat |
| 4.3.15 Audit 20 sampel | sebelum → sesudah dibaca langsung |
| 4.3.16 Validasi normalisasi | 18 pengujian, semua lolos |
| 4.3.17 Simpan | CSV + kolom `text_norm`, `kamus_normalisasi.csv`, `normalization_log.csv` |
| 4.3.18 Validasi akhir | 19 pengujian pada file yang dibaca ulang dari disk, semua lolos |

**Aturan normalisasi per kata:**

| Urutan | Aturan | Contoh |
|---|---|---|
| N1 | huruf kecil | `KPAI` → `kpai` |
| N2 | proteksi: dilewati tanpa diubah | `pb djarum`, `#bubarkankpai`, `[URL]`, `beg0`, `b*rak`, `k pea i` |
| N3 | reduplikasi angka | `anak2` → `anak-anak`, `org2` → `orang-orang`, `anak2nya` → `anak-anaknya` |
| N4 | tawa disatukan | `hahahaha` → `haha`, `wkwkwkwk` → `wkwk` |
| N5 | kamus | `yg` → `yang`, `gak`/`ga`/`gk` → `tidak`, `udah` → `sudah`, `aja` → `saja` |
| N6 | huruf berulang, lalu cek kamus lagi | `Palsuuuu` → `palsu`, `ancurrr` → `hancur` |

**Perubahan dari rencana awal (dan alasannya):**

| Rencana awal | Yang dijalankan | Alasan |
|---|---|---|
| saring kamus dengan daftar entitas saja | + daftar buang dan daftar perbaikan hasil audit | kamus bawaan memuat entri keliru di luar daftar entitas: `pt → patungan` (merusak "PT Djarum"), `pp → pulang pergi` (PP 109), `sd → sampai dengan` (anak SD), `sby → surabaya` (sebagian adalah SBY), `minyak → ga mendidik`. ±120 kata akan salah diganti |
| kamus = gabungan 2 kamus bawaan | + 79 kata informal tambahan | `gak`, `ga`, `aja`, `udah`, `kalo`, `tuh` tidak ada di kamus bawaan; tanpa tambahan, contoh di rencana sendiri tidak ternormalisasi. Tambahan ini menyumbang hampir setengah dari semua penggantian |
| urutan: kamus → reduplikasi → huruf berulang | reduplikasi → tawa → kamus → huruf berulang → kamus lagi | `org2`, `jgn2`, `trusss` baru bisa dicari di kamus setelah dipecah/dikurangi |
| — | aturan tawa | kamus hanya memuat panjang tawa tertentu, sehingga `hahaha` dan `hahahaha` berakhir berbeda |
| huruf berulang → 1 huruf | 1 atau 2 huruf, dipilih dari kosakata corpus; angka dan tanda baca tidak disentuh | `maaaf` harus menjadi `maaf`; `1000000` tidak boleh berubah |
| kata pendek ikut dinormalisasi | `tu`, `ni`, `dah`, `kaya`, `ad` sengaja **tidak** dimasukkan | artinya bergantung konteks |

**Hasil:**

| Hal | Hasil |
|---|---|
| Teks yang berubah | 2.371 dari 3.157 (75,1%); di domain 1.171 dari 1.515 (77,3%) |
| Kata yang diganti | 8.735: kamus 7.587 · reduplikasi 975 · tawa 108 · huruf berulang 65 |
| Kosakata | 9.240 → 8.534 kata unik (−7,6%) |
| Kata umum terkumpul | `tidak` 344 → 1.551 · `saja` 187 → 701 · `sudah` 251 → 529 · `anak-anak` 48 → 310 · `atlet` 80 → 141 |
| Entitas | tidak ada yang berkurang; `kpai` 1.135 → 1.136, `djarum` 561 → 566 (salah ketiknya ikut terkumpul) |
| `text_clean` | tidak berubah |

**Keterbatasan yang diketahui:**
- **Kamus tidak melihat konteks.** Pada audit 20 sampel ada satu salah arti: `th` di ANC-2488 berarti `tuh` tetapi diganti `tahun`.
- **Reduplikasi berimbuhan diulang utuh:** `bertahun2` → `bertahun-bertahun`, bukan `bertahun-tahun`.
- **Reduplikasi bertanda kutip** (`org"`, `atlet"`) tidak ditangani.
- **Kata informal yang jarang** dan tidak ada di kamus tetap seperti aslinya (`biayain`, `doank`, `kalu`).
- **Bentuk singkat kata kasar** (`bgsd`, `kntl`) tidak diurai, sesuai keputusan proteksi.
- Keterbatasan ini **hanya berdampak pada `text_norm`** (analisis kosakata dan fitur model), tidak pada anotasi.

### 6.6 Kesiapan anotasi — apakah sudah aman?

**Jawaban singkat: datanya sudah aman untuk dianotasi, tetapi anotasi belum bisa dimulai.** Teks yang akan dianotasi sudah final dan tervalidasi. Yang belum ada adalah sampel, guideline di folder ini, dan pilot.

**Yang sudah aman (dijamin oleh validasi di notebook):**

| Hal | Jaminan |
|---|---|
| Teks anotasi (`text_clean`) final | dibekukan; normalisasi tidak mengubahnya; 21 pengujian cleaning + 19 pengujian akhir lolos |
| Offset karakter stabil | tidak ada karakter tak terlihat, baris baru, atau spasi ganda; satu komentar = satu baris |
| Sinyal label utuh | kapital, emoji, `!!!`, hashtag, dan kata sensor tidak diubah; jumlah sebutan `kpai`/`djarum` sama dengan teks asli |
| Teks unik | tidak ada duplikat (abaikan kapital) |
| Jumlah cukup | 1.515 teks domain ≥ 800; cukup untuk sampel 1.000 |
| Bisa ditelusuri | setiap teks punya ID permanen yang menunjuk ke baris dataset asli |
| Bisa diulang | notebook dijalankan ulang menghasilkan file yang identik |
| Belum ada label | `annotation_status = not_annotated` untuk semua baris |

**Yang belum ada (harus selesai sebelum anotasi dimulai):**

| # | Hal | Kondisi sekarang |
|---|---|---|
| 1 | **Sampling** | belum dijalankan; belum ada `sample_ids.csv` dan file task Label Studio |
| 2 | **Guideline, taxonomy, konfigurasi Label Studio** | belum ada folder `guideline/` di `UTS_NLP/`; draf v0.2 masih di `project_uts/` dengan nama `KelpX` |
| 3 | **Pilot 50 teks** | belum; guideline masih draf dan belum diuji antar-annotator |
| 4 | **File anggota** | `KelompokNLP_…txt` masih kosong |

**Keputusan kecil yang sebaiknya diambil saat sampling/guideline:**
- **35 komentar di blok SpongeBob yang ikut menyebut KPAI/Djarum** sekarang di luar domain (konteksnya sensor KPI). Barisnya masih ada di CSV bila ingin dimasukkan.
- **7 near-duplicate di domain** (`#BubarkanKPAI`, `Bubarkan KPAI`, `#BUBARKAN KPAI !!!`): sebaiknya tidak semuanya masuk sampel.
- **Teks sangat pendek dan sangat panjang:** 34 teks domain berisi ≤2 kata dan 18 teks berisi >100 kata (maksimum 486). Guideline perlu aturan untuk keduanya (mis. kapan boleh `UNANNOTATABLE`).
- **Hashtag sebagai entitas:** apakah `KPAI` di dalam `#BubarkanKPAI` diblok sebagai `GOV_ORG` atau tidak, harus diputuskan di guideline sebelum pilot.

**Aturan yang harus dipegang selama anotasi:**
1. Yang diimpor ke Label Studio adalah **`text_clean`**, bukan `text_norm`. Offset entitas dihitung pada `text_clean`.
2. Setelah anotasi dimulai, **kode cleaning tidak boleh diubah**. Perubahan aturan cleaning mengubah `text_clean` dan merusak offset entitas yang sudah dianotasi. Kode normalisasi dan kamus masih boleh diperbaiki karena hanya memengaruhi `text_norm`.
3. Sampel dikunci dengan *seed* dan disimpan sebagai daftar ID, supaya tidak berubah bila notebook dijalankan ulang.
