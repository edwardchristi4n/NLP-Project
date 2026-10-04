# Rencana Normalisasi & Aturan Baris Tanpa Isi — KelpNLP (UTS NLP)

> Status: **DRAFT untuk diskusi tim** (belum dieksekusi). Versi 0.1 — 4 Okt 2026.
> Konteks: domain tunggal KPAI vs PB Djarum, 1,515 teks (`in_domain=True`) dari 3,157 baris hasil cleaning.

---

## 1. Ringkasan keputusan yang diusulkan

| # | Keputusan | Status |
|---|---|---|
| 1 | Anotasi (multilabel **dan** NER) dilakukan di **`text_clean`**, bukan `text_norm` | Diusulkan, alasannya di §4 |
| 2 | `text_norm` dipertahankan sebagai **kolom turunan** untuk fitur klasifikasi, diperlakukan sebagai **variabel eksperimen** (ablation `clean` vs `norm`) di UAS | Diusulkan |
| 3 | Normalisasi dibuat **konservatif**; perbaiki dulu kesalahan yang sudah diketahui (`th` → `tahun`) | Diusulkan, dikerjakan sebelum sampling |
| 4 | Normalisasi **dibekukan (`norm v1.0`) sebelum split train/val/test dibuat** | Diusulkan |
| 5 | Baris **tanpa isi** dihapus sebelum sampling/anotasi; baris pendek yang masih bermakna **dipertahankan** | Perlu diputuskan tim (§6) |
| 6 | Kode dipisah ke notebook sendiri (bukan `.py`) | Disepakati |

---

## 2. Kondisi saat ini (fakta dari data)

- Pipeline: `text_raw` (audit) → `text_clean` (cleaning teknis, beku untuk anotasi) → `text_norm` (normalisasi, untuk fitur).
- Kamus normalisasi: 758 entri (hasil audit dari 912; entri berbahaya seperti `dpr`→`dapur`, `pt`→`patungan` sudah dibuang). Ditambah aturan: reduplikasi (`anak2`→`anak-anak`), tawa, pengurangan huruf berulang berbasis kosakata korpus.
- `text_norm` berbeda dari `text_clean` pada **96.3%** baris domain — jadi ini bukan perubahan kecil.
- Kosakata domain (1,515 teks, huruf kecil, tokenisasi `\w+`):

| Versi | Ukuran kosakata | Kata yang muncul 1× |
|---|---|---|
| `text_clean` | 5,718 | 56.1% |
| `text_norm` | 5,123 | 54.5% |

  Kosakata turun ±10%: bermanfaat, tetapi tidak mengubah karakter data.
- Kesalahan yang diketahui: `th` → `tahun` salah di ANC-2488. Beberapa hasil juga berupa tebakan (mis. "Dyaaaarrrrrrrrr" → "duar").
- Tidak ada stemming dan tidak ada penghapusan stopword (disengaja: negasi dan kata kasar adalah sinyal sentimen/abusive).

---

## 3. Apakah normalisasi perlu? Opsi dan trade-off

| | **A. Tanpa normalisasi** | **B. Normalisasi penuh/agresif** | **C. Konservatif + ablation (rekomendasi)** |
|---|---|---|---|
| Deskripsi | Hanya `text_clean` | Kamus besar + banyak aturan, `text_norm` dipakai sebagai satu-satunya input | `text_norm` konservatif, **dibandingkan** dengan `text_clean` di eksperimen |
| Kosakata / sparsity | Paling besar | Paling kecil | Turun sedang (~10% saat ini) |
| Risiko mengubah makna | Tidak ada | **Tertinggi** (label dibuat dari teks asli, model belajar dari teks berubah) | Rendah, terkontrol & diaudit |
| Cocok untuk model klasik (TF-IDF + LR/SVM/NB) | Kurang optimal | Biasanya terbaik *bila akurat* | Terbaik secara metodologis: diukur, bukan diasumsikan |
| Cocok untuk model pretrained (mis. IndoBERT) | Biasanya cukup / lebih baik | Manfaat kecil, risiko tetap | Terjawab lewat ablation |
| Memenuhi rubrik 5.2 ("normalisasi dijelaskan") | Lemah | Cukup | **Kuat** (ada alasan + bukti) |
| Biaya tambahan | Nol | Besar (kamus terus dirawat) | **Kecil** — kode sudah ada |
| Bahan analisis laporan | Tidak ada | Sedikit | **Ada** (selisih F1 clean vs norm, analisis error) |

Catatan: PPT (Text Classification, slide 23) hanya menyebut "Text normalization" sebagai satu langkah dalam alur; tidak ada aturan rinci. Acuan yang menilai adalah spesifikasi §4.3 dan rubrik 5.2 / 7.1.2.

---

## 4. Kenapa anotasi di `text_clean`, bukan `text_norm`

1. **Normalisasi bisa salah menebak** → annotator bisa melabeli teks yang tidak pernah ditulis penulisnya.
2. **Sinyal emosi/abusive hilang** (huruf berulang, KAPITAL, tawa) padahal sering penanda kemarahan.
3. **NER butuh offset yang stabil.** Normalisasi mengubah panjang teks (`anak2` → `anak-anak`) sehingga posisi karakter bergeser. Span NER harus menempel pada teks yang dilihat annotator.
4. **Keterlacakan** (rubrik 5.2): gold harus bisa ditelusuri ke teks asli.
5. **Tidak ada kerja dua kali.** `text_norm` dibuat otomatis; label menempel lewat `id`. Satu kali anotasi berlaku untuk kedua kolom.

Konsekuensi: gold dataset = `text_clean` + label. File split memuat `id, text_clean, text_norm, label…`; pasangan teks–label tidak mungkin tertukar karena berasal dari baris yang sama.

---

## 5. Strategi yang direkomendasikan (Opsi C)

### 5.1 Sekarang (sebelum sampling & anotasi)
1. Pisahkan kode ke **notebook preprocessing tersendiri** (cleaning → aturan baris tanpa isi → normalisasi → validasi).
2. Perbaiki entri kamus yang jelas salah (`th` → `tahun`); entri yang artinya bergantung konteks sebaiknya dibuang, bukan ditebak.
3. Terapkan aturan baris tanpa isi (§6), catat di `removed_rows.csv` dengan alasan.
4. Jalankan ulang normalisasi + semua validasi; perbarui `normalization_log.csv` dan `kamus_normalisasi.csv`.
5. **Audit konsistensi label–teks:** ambil sampel acak (mis. 50–100 baris) dan cek bahwa `text_norm` tidak mengubah makna yang menentukan label (sentimen, target, abusive). Catat temuan.

### 5.2 Saat anotasi
- Annotator hanya melihat `text_clean`. `text_norm` tidak ditampilkan di Label Studio.

### 5.3 Sebelum gold & split dibuat — **pembekuan**
- Beri versi `norm v1.0` dan catat tanggal + hash kamus. Setelah itu `text_norm` di file split **tidak boleh berubah**.
- Alasannya: mengubah normalisasi setelah split/training memang tidak mengubah label, tetapi **semua hasil eksperimen harus diulang** (reproducibility, rubrik 7.1.2).

### 5.4 Di UAS
- Ablation: model yang sama dilatih dengan input `text_clean` vs `text_norm`; laporkan selisih F1 (macro-F1, Hamming loss untuk multilabel).
- Jika selisih kecil/negatif, tulis apa adanya — itu hasil yang sah.
- Ide perbaikan dari analisis error dicatat sebagai saran pengembangan, bukan mengubah `norm v1.0` diam-diam.
- Deployment (Streamlit) harus memakai fungsi cleaning/normalisasi yang **identik** dengan notebook. Karena kode tetap di notebook, salin fungsinya ke aplikasi dan uji dengan beberapa contoh teks (hasil harus sama persis).

### 5.5 Yang sengaja **tidak** dilakukan
- Stemming dan stopword removal (merusak negasi/kata kasar; bila ingin dicoba, jadikan variabel eksperimen).
- Spell-checker otomatis (merusak nama, hashtag, slang).
- Lowercasing di dataset (dilakukan di vectorizer agar kapitalisasi tetap tersimpan di kedua kolom).

---

## 6. Aturan baris tanpa isi / terlalu pendek

Spesifikasi §4.3 mewajibkan pemeriksaan "teks yang terlalu pendek". Data domain (1,515 teks, median 16 kata):

| Ambang (jumlah kata di `text_clean`) | Baris terkena | Di antaranya abusive (label asli 2/3) |
|---|---|---|
| ≤ 1 | 9 | 1 |
| ≤ 2 | 34 | 8 |
| ≤ 3 | 77 | 13 |

Banyak komentar ≤3 kata adalah opini yang jelas dan bisa dilabeli, mis. "Kpai sontoloyo !!!!!!!", "Kpai banyak omong!", "#BUBARKAN KPAI !!!", "Udah offside KPAI", "pengalihan isu". Menghapus semua teks ≤3 kata berarti kehilangan 13 dari 160 teks abusive (~8%) — kelas minoritas yang paling dibutuhkan.

| Opsi | Aturan | Kelebihan | Kekurangan |
|---|---|---|---|
| **P1 (rekomendasi)** | Hapus baris **tanpa isi**: hanya tanda baca/emoji, atau 1 token tanpa hashtag/nama target/kata bermakna. Sisanya dipertahankan, annotator memakai `UNANNOTATABLE` bila tak bisa dilabeli | Tidak membuang opini singkat; abusive terjaga | Perlu definisi "tanpa isi" yang jelas (cek manual daftar baris terhapus) |
| P2 | Hapus ≤1 kata | Sederhana, mudah dijelaskan; hanya 9 baris | Bisa ikut menghapus 1 hashtag bermakna (mis. `#BUBARKANKPAISECEPATNYA`) |
| P3 | Hapus ≤3 kata | Sangat sederhana | Kehilangan 77 baris dan 13 abusive; menghapus banyak opini valid |

Yang dihapus = **seluruh baris** (satu komentar), bukan kata-katanya, dan dicatat di `removed_rows.csv`. Keputusan ini harus final **sebelum sampling**, supaya tenaga annotator tidak terbuang. Sisa data setelah penghapusan tetap jauh di atas kebutuhan 1,000 teks sampel.

---

## 7. Bahan diskusi dengan tim

1. Setuju memakai Opsi C (konservatif + ablation)? Atau tim lebih suka A (tanpa normalisasi) demi kesederhanaan?
2. Aturan baris pendek: P1, P2, atau P3? Jika P1, siapa yang mengecek daftar baris yang akan dihapus?
3. Siapa yang melakukan audit sampel konsistensi label–teks (§5.1 langkah 5) dan berapa ukuran sampelnya?
4. Kapan tanggal pembekuan `norm v1.0` (sebelum split)?
5. Deployment: setuju menyalin fungsi dari notebook ke Streamlit + uji kesamaan output?

---

## 8. Urutan kerja setelah diskusi

1. Notebook preprocessing terpisah + perbaikan kamus + aturan baris tanpa isi → CSV, log, validasi ulang.
2. Notebook sampling (stratified dengan oversampling kelas minoritas; justifikasi di `summarize.md` §6.1).
3. Guideline anotasi + konfigurasi label.
4. Pilot 50 teks → κ → revisi guideline → freeze v1.0 → anotasi penuh.
5. Bekukan `norm v1.0` → gold → split.
aude