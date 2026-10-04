# Pedoman Anotasi (Annotation Guideline) — KelpNLP

| | |
|---|---|
| **Proyek** | UTS–UAS Pemrosesan Bahasa Alami, Gasal 2026/2027 |
| **Kelompok** | KelpNLP |
| **Domain** | komentar berita tentang polemik KPAI vs PB Djarum (2019) |
| **Versi** | 0.3 (draf, 4 Oktober 2026) |
| **Alat anotasi** | Prodigy (anotasi.uajy.ac.id) |

> **Status:** draf untuk ditinjau tim dan dosen. Setelah uji coba (pilot) 50 teks, pedoman ini direvisi lalu dibekukan menjadi versi 1.0. Setelah beku, aturan tidak boleh diubah tanpa catatan versi.
>
> Semua contoh diambil dari data asli dan ditulis apa adanya, dengan kode komentar (misalnya ANC-3000). Karena itu dokumen ini memuat kata kasar.

**Inti pedoman dalam lima kalimat**

1. Baca satu komentar sampai habis, lalu kerjakan dua hal di layar yang sama: sorot nama pihak (NER) dan centang label sikap (multilabel).
2. Sikap terhadap KPAI dan sikap terhadap PB Djarum dinilai **terpisah**. Bahasa abusif dinilai terpisah lagi.
3. Yang dinilai adalah **maksud penulis**, bukan kata per kata. Sindiran dilabeli sesuai maksudnya.
4. NER hanya menyorot **nama atau sebutan pihak**. Opini dan makian tidak disorot.
5. Kalau ragu, ikuti aturan di bab 5, beri tanda *flag*, dan tulis catatan. Jangan menebak.

---

## 1. Pendahuluan dan tujuan

### 1.1 Latar belakang
Pada 2019, KPAI (Komisi Perlindungan Anak Indonesia) menilai Audisi Umum Beasiswa Bulu Tangkis PB Djarum mengandung unsur eksploitasi anak, karena anak peserta memakai atribut bermerek Djarum yang identik dengan rokok. PB Djarum lalu mengumumkan penghentian audisi umum. Pembaca berita ramai berkomentar: sebagian besar membela PB Djarum dan menyerang KPAI, sebagian kecil sebaliknya, dan banyak yang memakai kata kasar.

### 1.2 Data yang dianotasi
- **Satu unit anotasi = satu komentar pembaca.** Teks yang ditampilkan adalah teks hasil pembersihan (`text_clean`): ejaan, huruf kapital, emoji, dan tanda baca penulis dipertahankan.
- Komentar berasal dari tiga berita. Judul berita ditampilkan di bawah teks sebagai konteks.

| Berita | Isi | Hal yang sering disinggung komentar |
|---|---|---|
| 1 | "KPAI Survei Ke Anak Yang Mana??" | survei tentang anak yang mengaitkan Djarum dengan rokok |
| 2 | "#bubarkanKPAI Trending di Twitter" | tagar pembubaran KPAI; banyak komentar menyapa "Bu" (komisioner KPAI) |
| 3 | "PB Djarum Hentikan Audisi Bulutangkis" | penghentian audisi; KPAI dianggap "cuci tangan" |

- Label dari dataset asli tidak ditampilkan dan tidak dipakai.

### 1.3 Tujuan
Hasil anotasi menjadi **gold dataset** untuk melatih dua model di UAS:

| Tugas | Pertanyaan yang dijawab | Bentuk hasil |
|---|---|---|
| **A. Multilabel** | Bagaimana sikap komentar terhadap KPAI dan terhadap PB Djarum, dan apakah bahasanya abusif? | satu atau lebih label per komentar |
| **B. NER** | Pihak mana yang disebut di dalam komentar? | potongan teks yang disorot + tipe entitas |

### 1.4 Prinsip umum
1. **Baca sampai habis** sebelum memberi label, termasuk komentar panjang.
2. **Nilai maksud penulis** dari sudut pandang pembaca umum Indonesia.
3. **Konteks berita boleh dipakai** untuk memahami siapa yang dimaksud (lihat aturan M2).
4. **Kerjakan sendiri.** Jangan berdiskusi tentang teks yang sedang dianotasi sampai kesepakatan antar-annotator selesai dihitung.
5. **Jangan menebak.** Kalau dua pembacaan sama kuat, pakai aturan bab 5 dan beri *flag*.

---

## 2. Skema label

### 2.1 Label multilabel (tingkat komentar)

| Label | Arti singkat |
|---|---|
| `KPAI_POSITIVE` | mendukung, membela, atau membenarkan KPAI |
| `KPAI_NEGATIVE` | mengkritik, menyindir, atau menolak KPAI |
| `PBDJARUM_POSITIVE` | mendukung, membela, atau mengapresiasi PB Djarum dan audisinya |
| `PBDJARUM_NEGATIVE` | mengkritik PB Djarum, audisinya, atau pemakaian merek rokoknya |
| `ABUSIVE` | memakai makian, hinaan, atau sebutan merendahkan |
| `NEUTRAL` | bisa dipahami, tetapi tidak bersikap ke KPAI maupun PB Djarum dan tidak abusif |
| `UNANNOTATABLE` | tidak bisa dipahami atau tidak bisa dinilai (lihat bab 6) |

Satu komentar boleh menerima beberapa label sekaligus. `NEUTRAL` dan `UNANNOTATABLE` selalu berdiri sendiri.

### 2.2 Tipe entitas NER (tingkat kata)

| Tipe | Arti singkat | Contoh |
|---|---|---|
| `GOV_ORG` | lembaga negara atau pemerintah | `KPAI`, `pemerintah`, `kemenpora`, `Komnas HAM` |
| `NONGOV_ORG` | organisasi bukan pemerintah: perusahaan, yayasan, klub, federasi, LSM | `PB Djarum`, `Djarum Foundation`, `Sampoerna`, `lentera anak` |
| `PERSON` | nama orang | `Susanto`, `Kevin Sanjaya`, `Pak Jokowi` |
| `GROUP` | sebutan kelompok orang yang menjadi pihak dalam polemik | `anak-anak`, `atlet`, `orang tua`, `masyarakat`, `perokok` |

### 2.3 Hubungan multilabel dan NER
NER menjawab **"siapa yang disebut"**. Multilabel menjawab **"bagaimana sikap penulis terhadap pihak itu"**.

| Label multilabel | Entitas yang biasanya menjadi sasarannya |
|---|---|
| `KPAI_POSITIVE`, `KPAI_NEGATIVE` | `GOV_ORG` yang merujuk ke KPAI; `PERSON` pejabat KPAI |
| `PBDJARUM_POSITIVE`, `PBDJARUM_NEGATIVE` | `NONGOV_ORG` yang merujuk ke PB Djarum atau Djarum |
| `ABUSIVE` | sasaran makian, sering berupa `GOV_ORG` atau `PERSON` |
| `NEUTRAL` | tidak ada sasaran sikap; entitas tetap boleh ada |

Hubungan ini **tidak satu-ke-satu**:

- Label bisa diberikan tanpa entitas. `kurang kerjaan atau kurang jatah ya` (ANC-3109) tidak menyebut nama, tetapi berlabel `KPAI_NEGATIVE`.
- Entitas bisa ada tanpa label sikap. `Sanggup ga pemerintah bina atlit dini tanpa seponsor` (ANC-2606) memuat `[pemerintah]` dan `[atlit]`, tetapi berlabel `NEUTRAL`.

---

## 3. Aturan label multilabel

### 3.1 Urutan berpikir
Jawab lima pertanyaan ini secara berurutan untuk setiap komentar.

```
1. Apakah teks bisa dipahami?
      Tidak  -> UNANNOTATABLE, selesai.
2. Apakah penulis bersikap terhadap KPAI?
      Mendukung -> KPAI_POSITIVE        Mengkritik -> KPAI_NEGATIVE
3. Apakah penulis bersikap terhadap PB Djarum?
      Mendukung -> PBDJARUM_POSITIVE    Mengkritik -> PBDJARUM_NEGATIVE
4. Apakah ada makian, hinaan, atau sebutan merendahkan?
      Ya -> ABUSIVE
5. Tidak ada satu pun label dari langkah 2-4?
      -> NEUTRAL
```

### 3.2 Aturan umum

**M1. Pilih semua label yang berlaku.** Langkah 2, 3, dan 4 dinilai sendiri-sendiri.

**M2. Sasaran boleh tersirat**, tetapi hanya dalam dua keadaan:

- komentar menilai **tindakan yang dalam polemik ini hanya dilakukan satu pihak**, atau
- komentar memakai **sebutan pengganti** yang jelas.

| Petunjuk di teks | Sasaran |
|---|---|
| melarang, menegur, menuduh eksploitasi, survei, cari panggung, cari sensasi, kurang kerjaan, bikin ruwet tanpa solusi, cuci tangan, ngeles, "selama ini ke mana saja" | KPAI |
| "komisi", "lembaga ini", "Bu"/"Ibu" (komisioner KPAI), "ketua" | KPAI |
| audisi, beasiswa, sponsor, logo atau merek di kaus, pembinaan atlet, menghentikan audisi | PB Djarum |

Kalau sasaran masih bisa dibaca ke dua arah, **jangan beri label pihak**.

**M3. Mendukung satu pihak tidak otomatis menolak pihak lain.** Label untuk pihak kedua hanya diberikan kalau sikap terhadap pihak itu memang ada di teks.

- `sebagai sponsor wajarlah...saja kalau djarum memasang berbagai banner Djarum. Djarum tidak salah....` (ANC-1604) → `PBDJARUM_POSITIVE` saja.
- `KPAI bisanya ngomong ga ada konstribusi, Djarum maju terus bangsa ini berterima kasih puluhan tahun kiprah pembinaan sejak dini atlet Bulu tangkis` (ANC-3149) → `KPAI_NEGATIVE` + `PBDJARUM_POSITIVE`.

**M4. Sindiran dilabeli sesuai maksudnya.** Pujian yang jelas menyindir adalah label negatif. Tanda sindiran: lanjutan kalimat yang menjatuhkan, logika yang sengaja dilebih-lebihkan, atau tantangan agar KPAI menggantikan peran Djarum.

**M5. Ikuti inti komentar.** Pengakuan singkat sebelum kata "tapi" atau "cuma" tidak dihitung sebagai sikap positif.

- `Mungkin niatnya baik KPAI, cuman kurang main cantik n elegan aja` (ANC-1131) → `KPAI_NEGATIVE` saja.

Positif dan negatif untuk pihak yang sama hanya diberikan bersamaan kalau komentar memuat dua penilaian yang sama kuat. Ini jarang; beri *flag*.

**M6. Sikap terhadap pejabat KPAI dihitung sebagai sikap terhadap KPAI.** Kritik ke ketua atau komisioner KPAI → `KPAI_NEGATIVE`. Sikap terhadap pihak lain (pemerintah, Presiden, KPI, Komnas HAM, Kak Seto, sinetron) **tidak** dihitung.

**M7. `ABUSIVE` menilai bahasa, bukan sikap.** Label ini boleh berdiri sendiri kalau sasaran makiannya bukan KPAI atau PB Djarum, atau tidak jelas.

**M8. `NEUTRAL` dan `UNANNOTATABLE` tidak boleh digabung** dengan label apa pun.

### 3.3 Aturan per label

#### KPAI_POSITIVE
- **Definisi:** komentar mendukung, membela, atau membenarkan KPAI atau tindakannya dalam polemik ini.
- **Beri label jika:** menyatakan dukungan atau pujian yang tulus; membenarkan alasan KPAI (merek rokok tidak pantas di kegiatan anak, audisi adalah eksploitasi anak, KPAI mengikuti undang-undang); menolak KPAI dibubarkan atau membela KPAI dari hujatan.
- **Jangan beri jika:** pujiannya menyindir (M4); komentar hanya menyebut KPAI; komentar anti-rokok secara umum tanpa menyinggung iklan, merek, atau audisi.

| Jenis | Contoh | Keputusan |
|---|---|---|
| Positif | `Jangan dong, kan fungsinya bagus melindungi anak-anak di NKRI dari eksploitasi. Itu kan misi organisasi KPAI.` (ANC-1149) | `KPAI_POSITIVE` |
| Positif | `Kayanya KPAI ngga salah kalau mengacu ke Undang Undangnya` (ANC-2521) | `KPAI_POSITIVE` |
| Positif | `Ban iklan rokok!!! Selamatkan generasi penerus dari rokok` (ANC-1931) | `KPAI_POSITIVE` (membenarkan alasan KPAI soal iklan rokok) |
| Negatif | `dukung KPAI biar keliatan ada kerjaannya...` (ANC-3125) | **bukan** positif → `KPAI_NEGATIVE` (lanjutannya menjatuhkan) |
| Negatif | `KPAI benar kok,mereka bertugas melindungi anak-anak,bukan masa depan mereka` (ANC-1188) | **bukan** positif → `KPAI_NEGATIVE` ("bukan masa depan mereka" adalah sindiran) |
| Ambigu | `SALUT UTK KPAI` (ANC-2291) | Tidak ada tanda sindiran → dibaca apa adanya → `KPAI_POSITIVE`, beri *flag* |

#### KPAI_NEGATIVE
- **Definisi:** komentar mengkritik, menyindir, menolak, atau menuntut pembubaran KPAI.
- **Beri label jika:** mengkritik keputusan atau kinerja KPAI; menuduh cari panggung, cari sensasi, atau cuci tangan; membandingkan dengan masalah anak lain yang dianggap diabaikan; menyalahkan KPAI atas dampak buruk (prestasi turun, mimpi anak hilang); menantang KPAI menggantikan peran Djarum; memakai tagar pembubaran KPAI.
- **Jangan beri jika:** yang dikritik hanya pihak lain (pemerintah, KPI, Komnas HAM); komentar hanya bertanya untuk mendapat informasi.

| Jenis | Contoh | Keputusan |
|---|---|---|
| Positif | `Sudah 50 tahun. Koq sekarang KPAI baru ribut ?` (ANC-2549) | `KPAI_NEGATIVE` |
| Positif | `KPAI...bukannya mencari solusi..tapi nambah masalah saja...sudah jelas ada lihak swasta yang mau terlibat ngedidik anak...ini malah cari sensasi...` (ANC-2700) | `KPAI_NEGATIVE` |
| Positif (tersirat) | `kemaren aja ribut2 pelanggaran hak anak, sekarang udh dihentikan malah cuci tangan.. maunya gmana sih???` (ANC-3106) | `KPAI_NEGATIVE` (M2: "cuci tangan") |
| Positif (tersirat) | `Mending urusin guru yg diserang ortu yg berantem tuh bu dari pada urusin bulutangkis` (ANC-1914) | `KPAI_NEGATIVE` (M2: "bu" = komisioner KPAI) |
| Negatif | `Sanggup ga pemerintah bina atlit dini tanpa seponsor` (ANC-2606) | **bukan** `KPAI_NEGATIVE` → `NEUTRAL` (yang ditanya pemerintah) |
| Ambigu | `Pegawai KPAI apa tidak merokok ?` (ANC-1818) | Pertanyaan menyindir, bukan mencari informasi → `KPAI_NEGATIVE` |
| Ambigu | `gak setuju kpai dibubarkan, kpai harus gantiin jarum rekrut pemain2 muda` (ANC-2312) | Menuntut KPAI menanggung akibat (M4) → `KPAI_NEGATIVE`, beri *flag* |

#### PBDJARUM_POSITIVE
- **Definisi:** komentar mendukung, membela, atau mengapresiasi PB Djarum, Djarum Foundation, atau audisinya.
- **Beri label jika:** mengapresiasi jasa pembinaan atlet; menilai logo atau merek di audisi wajar atau tidak salah; menyetujui sikap PB Djarum; meminta PB Djarum didukung.
- **Jangan beri jika:** komentar hanya menyebut nama Djarum tanpa menilai; komentar hanya menyerang KPAI tanpa menilai Djarum (M3).

| Jenis | Contoh | Keputusan |
|---|---|---|
| Positif | `sebagai sponsor wajarlah...saja kalau djarum memasang berbagai banner Djarum. Djarum tidak salah....` (ANC-1604) | `PBDJARUM_POSITIVE` |
| Positif | `Djarum sudah keluar duit banyak buat bea siswa, mencetak atlit yang membanggakan bangsa, masih di salahin.` (ANC-2542) | `PBDJARUM_POSITIVE` |
| Positif (tersirat) | `ya wajar aja, namanya juga branding. terus mau duitnya, tp gk mau perusahaannya?` (ANC-1953) | `PBDJARUM_POSITIVE` (M2: "branding") |
| Negatif | `Sudah 50 tahun. Koq sekarang KPAI baru ribut ?` (ANC-2549) | **bukan** `PBDJARUM_POSITIVE`; hanya `KPAI_NEGATIVE` |
| Ambigu | `Semoga KPAI bisa membuat audisi bulatangkis sekelas Djarum.` (ANC-2753) | Menantang KPAI dan menjadikan Djarum ukuran yang baik → `KPAI_NEGATIVE` + `PBDJARUM_POSITIVE` |

#### PBDJARUM_NEGATIVE
- **Definisi:** komentar mengkritik PB Djarum, Djarum Foundation, audisinya, atau pemakaian merek rokoknya.
- **Beri label jika:** menilai audisi sebagai promosi rokok; meminta Djarum melepas nama atau logo; meragukan niat Djarum; mengkritik sikap Djarum menghentikan audisi ("baper", "ngambek").
- **Jangan beri jika:** komentar anti-rokok secara umum tanpa menyinggung Djarum, audisi, atau merek; tantangan "tutup saja pabriknya kalau berani" (itu sindiran ke pihak yang melarang).

| Jenis | Contoh | Keputusan |
|---|---|---|
| Positif | `kalau benar2 tanpa pamrih. Jarum gausah pasang iklan di event anak2. mau ngerusak mental anak2?` (ANC-2532) | `PBDJARUM_NEGATIVE` |
| Positif | `brainwash merk rokok memang hebat. solusi gampang ganti nama Djarum saja gak mau.` (ANC-1961) | `PBDJARUM_NEGATIVE` |
| Positif (tersirat) | `Masalahnya logo logo itu dilihat jutaan orang dari berbagai usia di media elekteonik (brand awareness). Padahal itu produk yg merusak kesehatan.` (ANC-2552) | `PBDJARUM_NEGATIVE` (M2: "logo") |
| Negatif | `Tutup saja pabrik Rokok Djarumny kalau berani.` … `CSR Djarum sangat membantu anak anak aktif di bulutangkis` … `KPAI sangat sempit pemikirannya` (ANC-2977) | **bukan** `PBDJARUM_NEGATIVE` → `PBDJARUM_POSITIVE` + `KPAI_NEGATIVE` |
| Negatif | `Yang bener itu tutup ajah perusahaan rokok di indonesia..., bikin orang boros dan merusak kesehatan...` (ANC-2631) | **bukan** `PBDJARUM_NEGATIVE` → `NEUTRAL` (anti-rokok umum) |
| Ambigu | `bisa diambil jalan tengah. coba pb djarum ganti nama. tdk pakai merk rokok.` (ANC-2715) | Meminta Djarum melepas merek rokok → `PBDJARUM_NEGATIVE` |

#### ABUSIVE
- **Definisi:** komentar memakai makian, hinaan, atau sebutan yang merendahkan orang atau lembaga, kepada siapa pun.
- **Uji sederhana:** *kalau kata itu diucapkan langsung kepada seseorang di depan umum, apakah itu makian atau hinaan, bukan sekadar kritik?*

| Termasuk `ABUSIVE` | Contoh kata atau ungkapan |
|---|---|
| Makian dan kata kasar | anjing, bangsat, tai, kampret, bacot, ndasmu |
| Hinaan terhadap kecerdasan atau kewarasan | goblok, tolol, bego, bodoh, dungu, dongo, bloon, geblek, pekok, gila, "mikir pake dengkul", "tidak punya otak" |
| Sebutan yang merendahkan martabat | sampah, benalu, sontoloyo, "mulut comberan" |
| Pelesetan nama yang menghina | `K pea I`, `KOMISI PENGERDIL ANAK INDONESIA` |
| Bentuk tersensor atau ejaan lain dari kata di atas | `beg0`, `TOL*L`, `gublokk`, `bangsad` |

| **Bukan** `ABUSIVE` (kritik keras terhadap tindakan) | Contoh kata atau ungkapan |
|---|---|
| Penilaian kinerja atau sikap | tidak becus, tidak berguna, unfaedah, kurang kerjaan, cari panggung, cari sensasi, ngawur, aneh, plin-plan |
| Ejekan ringan tentang sikap | lebay, baper, alay, nyinyir, munafik, faker, sok pahlawan |

| Jenis | Contoh | Keputusan |
|---|---|---|
| Positif | `KPAI GOBLOK!!!!!` (ANC-3000) | `KPAI_NEGATIVE` + `ABUSIVE` |
| Positif | `KPAI gak ada guna, sekumpulan orang gabut yg beg0, gak bisa mikir jernih.` (ANC-2859) | `KPAI_NEGATIVE` + `ABUSIVE` (bentuk sensor tetap dihitung) |
| Positif | `Itulah KPAI. Mikirnya pake dengkul, jadi dangkal` (ANC-1762) | `KPAI_NEGATIVE` + `ABUSIVE` |
| Negatif | `KPAI kurang kerjaan. KPAI cari sensasi saja.` (ANC-3006) | **bukan** `ABUSIVE` → `KPAI_NEGATIVE` saja |
| Negatif | `KPAI LEBAY...` (ANC-2802) | **bukan** `ABUSIVE` → `KPAI_NEGATIVE` saja |
| Ambigu | `Bodoh, ..............` (ANC-1916) | Ada hinaan, sasaran tidak jelas → `ABUSIVE` saja (M7) |
| Ambigu | `kalo presidennya ahok uda abis tuh digoblok-goblokin dianjing-anjingin orang2 KPAI` (ANC-1209) | Penulis sendiri memakai kata kasar untuk membayangkan makian ke KPAI → `KPAI_NEGATIVE` + `ABUSIVE` |

Kata kasar yang hanya dikutip untuk ditolak ("jangan bilang goblok dong") tidak dihitung `ABUSIVE`.

#### NEUTRAL
- **Definisi:** komentar bisa dipahami, tetapi tidak menunjukkan sikap yang jelas terhadap KPAI maupun PB Djarum, dan tidak abusif.
- **Beri label jika:** pertanyaan untuk mendapat informasi; pernyataan informatif; harapan tanpa menyalahkan siapa pun; sikap yang hanya tertuju ke pihak lain; komentar tentang forum atau komentator lain tanpa makian.
- **Jangan beri jika:** ada sasaran tersirat menurut M2; ada kata abusif (pakai `ABUSIVE`); teks tidak bisa dipahami (pakai `UNANNOTATABLE`).

| Jenis | Contoh | Keputusan |
|---|---|---|
| Positif | `Mungkin bisa dijelaskan dulu apa artinya exploitasi?` (ANC-2349) | `NEUTRAL` |
| Positif | `rezim jokowi makin ngaco... pembinaan anak usia dini cuma omong kosong...` (ANC-2745) | `NEUTRAL` (sikap hanya ke pemerintah) |
| Positif | `nah loh kena skak deh yaaaa... nice thread gannn... positif dan informatif yaaa... ditunggu thread lainnya...` (ANC-1252) | `NEUTRAL` (komentar tentang forum) |
| Negatif | `kurang kerjaan atau kurang jatah ya` (ANC-3109) | **bukan** `NEUTRAL` → `KPAI_NEGATIVE` (M2: "kurang kerjaan" menilai pihak yang melarang) |
| Ambigu | `Seharusya iklan produk tembakau di media juga dihapuskan. Logikanya sih gitu` (ANC-2401) | Bisa mendukung, bisa menyindir → `NEUTRAL`, beri *flag* |
| Ambigu | `Semoga diberi kemudahan Jangan sampai mimpi anak bangsa menjadi atlit bulu tangkis sirna` … (ANC-1775) | Harapan, tidak menyalahkan siapa pun → `NEUTRAL` |

### 3.4 Kombinasi label

| Kombinasi | Sah? | Catatan |
|---|---|---|
| `KPAI_NEGATIVE` + `PBDJARUM_POSITIVE` | Ya | kombinasi paling umum |
| `KPAI_NEGATIVE` + `ABUSIVE` | Ya | |
| `KPAI_POSITIVE` + `PBDJARUM_NEGATIVE` | Ya | |
| `KPAI_NEGATIVE` + `PBDJARUM_NEGATIVE` | Ya | mengkritik kedua pihak |
| `ABUSIVE` saja | Ya | makian tanpa sasaran KPAI atau Djarum |
| `KPAI_POSITIVE` + `KPAI_NEGATIVE` | Jarang | hanya bila dua penilaian sama kuat; beri *flag* |
| `NEUTRAL` + label lain | **Tidak** | |
| `UNANNOTATABLE` + label lain | **Tidak** | |
| Tanpa label sama sekali | **Tidak** | setiap komentar minimal punya satu label |

---

## 4. Aturan NER

### 4.1 Apa yang disorot
Sorot **nama atau sebutan pihak** saja. Yang **tidak** disorot: opini (`gak becus`, `didukung`), makian (`goblok`), kata kerja, kata ganti (`dia`, `mereka`, `kalian`, `lu`, `ente`, `anda`), dan sapaan tanpa nama (`bu`, `pak`, `gan`, `bro`, `bos`).

Prodigy memecah teks menjadi **token** (kata dan tanda baca). Sorotan selalu mengikuti batas token.

### 4.2 Aturan per tipe entitas
Pada contoh di bawah, tanda kurung siku `[ ]` menandai bagian yang disorot.

#### GOV_ORG — lembaga negara atau pemerintah
| Jenis | Contoh | Keputusan |
|---|---|---|
| Positif | `[KPAI], [KPPU], [Komnas ham] adalah contoh lembaga yang lebih banyak mudaratnya.` (ANC-2444) | tiga entitas `GOV_ORG` |
| Positif | `ranahnya [pemerintah] khususnya [kemenpora]` (ANC-2511) | dua entitas `GOV_ORG` |
| Positif | `" [Komisi Perlindungan Anak Indonesia] "` (ANC-1155) | nama resmi diambil utuh |
| Negatif | `dasar lembaga goblok` (ANC-1210) | "lembaga" kata umum → tidak disorot |
| Negatif | `sama2 benalu negara!` (ANC-2729) | "negara" bukan lembaga → tidak disorot |
| Ambigu | `dasar [K pea I] !` (ANC-2576) | pelesetan nama KPAI → tetap `GOV_ORG`, tiga token disorot |
| Ambigu | `[KPI] cari panggung..biar dibilang ada kerja..` (ANC-2504) | salah tulis KPAI atau memang KPI, keduanya lembaga negara → `GOV_ORG` |

#### NONGOV_ORG — organisasi bukan pemerintah
| Jenis | Contoh | Keputusan |
|---|---|---|
| Positif | `gak bisa membedakan [PB Djarum], [Djarum Foundation] dan Pabrik Rokok [Djarum]` (ANC-2329) | tiga entitas `NONGOV_ORG` |
| Positif | `itu [kpai] sama [lentera anak] kok tiba-tiba men-judge [pb djarum]` (ANC-1245) | `lentera anak` (LSM) → `NONGOV_ORG` |
| Positif | `[Jarum] tdk mnyuruh [anak-anak] merokok` (ANC-1859) | "Jarum" = Djarum → `NONGOV_ORG`; "anak-anak" → `GROUP` |
| Negatif | `Sialnya [KPAI] tidak bisa membedakan mana Yayasan dan mana bisnis...` (ANC-2360) | "Yayasan" sebutan umum tanpa nama → tidak disorot; begitu juga `perusahaan rokok` dan `sponsor` |
| Ambigu | `jawabnya [jarum] itu y jarum jahit` (ANC-1645) | "jarum" pertama adalah nama yang ditanyakan (Djarum) → disorot; "jarum jahit" adalah benda → tidak disorot |
| Ambigu | `merokok [jarum] super ada juga merokok [sampoerna]` (ANC-1682) | merek rokok yang sama dengan nama perusahaan → sorot **nama perusahaannya saja** |

#### PERSON — nama orang
| Jenis | Contoh | Keputusan |
|---|---|---|
| Positif | `Menjadi seperti [Kevin Sanjaya] dan [Hendra Setiawan] adalah mimpi` (ANC-2597) | nama lengkap diambil utuh |
| Positif | `ya sudah [pak susanto] nanti [kpai] saja yang menggantikan [pb djarum]` (ANC-2779) | sapaan yang menempel pada nama ikut disorot |
| Positif | `[Susanto] [KPAI] hanya bisa cari masalah saja` (ANC-2973) | dua entitas berdampingan: `PERSON` lalu `GOV_ORG` |
| Negatif | `Ketua [KPAI] lagi cari panggung.` (ANC-2484) | jabatan tanpa nama → "Ketua" tidak disorot |
| Negatif | `KE MANA AJA BU?` (ANC-1915) | sapaan tanpa nama → tidak disorot |
| Ambigu | `mudah2n ada solusi dr Menpora` (ANC-3004) | "Menpora" adalah jabatan, bukan nama → tidak disorot |

#### GROUP — kelompok orang yang menjadi pihak dalam polemik
Agar konsisten, `GROUP` dibatasi pada **lima kelompok** berikut, dan yang disorot hanya **kata intinya**.

| Kelompok | Kata inti yang disorot |
|---|---|
| Anak dan generasi muda | anak, anak-anak, anak2, anak anak, bocah, generasi |
| Atlet | atlet, atlit, pemain, pebulutangkis, pebulu tangkis, olahragawan |
| Orang tua | orang tua, ortu |
| Masyarakat umum | masyarakat, rakyat, netizen, warganet, publik |
| Perokok | perokok |

| Jenis | Contoh | Keputusan |
|---|---|---|
| Positif | `mimpi untuk [anak2] Indonesia` (ANC-2597) | kata inti saja; "Indonesia" tidak ikut |
| Positif | `takut ya diserang [netizen]` (ANC-2835) | `GROUP` |
| Positif | `sampe jadi [atlet] sama pelatih juga` (ANC-2385) | "pelatih" di luar lima kelompok → tidak disorot |
| Negatif | `orang2 [KPAI] terbatas` (ANC-2385) | "orang2" kata umum → hanya `KPAI` yang disorot |
| Negatif | `ah anak saya ditanya [jarum]` (ANC-1676) | "anak saya" satu orang tertentu, bukan kelompok → tidak disorot; "jarum" = Djarum tetap disorot |
| Ambigu | `Komisi Perlindungan Anak Indonesia` | "Anak" bagian dari nama lembaga → tidak disorot terpisah (aturan B7) |

### 4.3 Aturan batas sorotan

| No | Aturan | Contoh |
|---|---|---|
| B1 | **Nama resmi lebih dari satu kata disorot utuh** sebagai satu entitas. | `[Komisi Perlindungan Anak Indonesia]`, `[PB Djarum]`, `[Djarum Foundation]`, `[Komnas HAM]` |
| B2 | **Tanda baca di tepi tidak ikut.** | `KPAI!!!!!` → `[KPAI]` |
| B3 | **Sapaan yang menempel pada nama ikut; jabatan tidak ikut.** | `[Pak Jokowi]`, `[kak seto]`, tetapi `ketua [KPAI]` |
| B4 | **Bentuk ulang disorot utuh**, apa pun cara menulisnya. | `[anak-anak]`, `[anak2]`, `[anak anak]` |
| B5 | **Kata keterangan di sekitar `GROUP` tidak ikut.** | `para [atlet] muda`, `[anak2] indonesia`, `[generasi] penerus` |
| B6 | **Setiap kemunculan disorot**, termasuk nama yang sama yang muncul berulang. | `[KPAI] kurang kerjaan. [KPAI] cari sensasi saja.` |
| B7 | **Tidak boleh bertumpuk.** Kalau ada dua kemungkinan, pilih nama resmi yang paling panjang. | `[Komisi Perlindungan Anak Indonesia]`, bukan `Komisi Perlindungan [Anak] Indonesia` |
| B8 | **Salah ketik, huruf berulang, dan akhiran yang menempel tetap disorot** satu token utuh. | `[djarumnya]`, `[kpainya]`, `[KPAIIIII]`, `[KAPAI]` |
| B9 | **Tagar slogan tidak disorot.** Tagar yang isinya hanya nama, disorot. | `#BubarkanKPAI` tidak disorot; `#KPAI` disorot; pada `#Bubarkan [KPAI]` kata `KPAI` disorot |
| B10 | **Nama yang menempel dengan kata lain tanpa spasi** tetap disorot satu token utuh. Potongannya dirapikan otomatis saat pembuatan gold dataset. | `merokok?kpai`, `djarum.tp` |

Yang juga **tidak** disorot: nama tempat dan negara (`Indonesia`, `Kudus`), kata `negara` dan `bangsa`, nama media sosial (`twitter`, `IG`), dan nama orang atau kelompok di dalam makian (`anak setan`).

### 4.4 Daftar keputusan cepat

| Teks | Tipe | Catatan |
|---|---|---|
| KPAI, Kpai, kpai, KAPAI, K pea I, Komisi Perlindungan Anak Indonesia | `GOV_ORG` | termasuk pelesetan |
| pemerintah, kemenpora, KPI, KPPU, Komnas HAM, BPJS | `GOV_ORG` | |
| PB Djarum, PB.Djarum, Djarum, Jarum (= Djarum), PT Djarum, Djarum Foundation | `NONGOV_ORG` | |
| Sampoerna, Gudang Garam, BCA, Blibli, PBSI, Lentera Anak, Komnas PA, HTI | `NONGOV_ORG` | Komnas PA adalah LSM, bukan lembaga negara |
| Susanto, Sitti Hikmawati, Jokowi, Ahok, Kak Seto, Kevin Sanjaya, Hendra Setiawan, Rudi Hartono | `PERSON` | |
| anak, anak2, anak-anak, atlet, atlit, pemain, orang tua, masyarakat, netizen, perokok, generasi | `GROUP` | kata inti saja |
| Menpora, Presiden, ketua, komisioner, pegawai, bu, pak | tidak disorot | jabatan atau sapaan tanpa nama |
| negara, bangsa, Indonesia, lembaga, komisi, yayasan, sponsor, perusahaan rokok | tidak disorot | kata umum atau tempat |

Kalau menemukan nama yang tidak ada di daftar: tentukan tipenya dengan definisi di 4.2, beri *flag*, dan tulis di catatan agar daftar ini bisa dilengkapi.

### 4.5 Skema BIO
Annotator **tidak** menulis tag BIO. Annotator cukup menyorot; notebook yang mengubah sorotan menjadi tag BIO.

- `B-` = token pertama entitas, `I-` = token lanjutan entitas yang sama, `O` = bukan entitas.
- Dengan 4 tipe entitas, ada 9 tag: `B-`/`I-` untuk tiap tipe, ditambah `O`.

```
Token : Sangat  disayangkan  ...  KPAI       lebay  ...  PB            Djarum        baperan
Tag   : O       O            O    B-GOV_ORG  O      O    B-NONGOV_ORG  I-NONGOV_ORG  O
```

Karena itu batas sorotan harus tepat: selisih satu token saja menghasilkan tag yang berbeda.

---

## 5. Kasus khusus dan konflik antarlabel

| Kasus | Contoh | Keputusan dan alasan |
|---|---|---|
| **Sindiran dengan pujian palsu** | `dukung KPAI biar keliatan ada kerjaannya...` (ANC-3125) | `KPAI_NEGATIVE`. Lanjutan kalimat menjatuhkan. |
| **Sindiran dengan logika berlebihan** | `sepertinya KPAI ingin yang ikutan audisi umurnya 40+ jadi ga termasuk eksploitasi anak` (ANC-1280) | `KPAI_NEGATIVE`. |
| **Tantangan agar KPAI menggantikan Djarum** | `Semoga KPAI bisa menciptakan pebulu tangkis Indonesia yg hebat!!!` (ANC-3119) | `KPAI_NEGATIVE`. Pola sindiran yang umum di data ini. |
| **Pujian tanpa tanda sindiran** | `#GOODJOB KPAI` (ANC-2277) | `KPAI_POSITIVE`, beri *flag*. Tanpa tanda sindiran, teks dibaca apa adanya. |
| **Pertanyaan retoris** | `KPAI sdh berbuat apa utk negara ini?` (ANC-3071) | `KPAI_NEGATIVE`. Bukan mencari informasi. |
| **Pertanyaan sungguhan** | `Mungkin bisa dijelaskan dulu apa artinya exploitasi?` (ANC-2349) | `NEUTRAL`. |
| **Mengutip lalu membantah** | `Lho katanya PB Djarum cuma meng eksploitasi anak anak. Gimana sih....????` (ANC-2610) | Nilai sikap penulis, bukan isi kutipan → `KPAI_NEGATIVE`. |
| **Mengkritik kedua pihak** | `Sangat disayangkan...KPAI lebay...PB Djarum baperan....mudah2n ada solusi dr Menpora` (ANC-3004) | `KPAI_NEGATIVE` + `PBDJARUM_NEGATIVE`. "lebay" dan "baperan" bukan `ABUSIVE`. |
| **Dua sikap berlawanan dalam satu komentar** | `saya mendukung kpai, tindakan kpai sudah tepat, sudah saatnya perusahaan rokok tidak menjadi motor pengembangan olahraga di Indonesia.` (ANC-1937) | `KPAI_POSITIVE` + `PBDJARUM_NEGATIVE`. |
| **Sikap hanya ke pihak lain** | `rezim jokowi makin ngaco... pembinaan anak usia dini cuma omong kosong...` (ANC-2745) | `NEUTRAL` (M6). `[jokowi]` tetap disorot sebagai `PERSON`. |
| **Pihak lain disebut bersama KPAI** | `KPAI apa sdh disusupi paham2 HTI atau Ormas2 radikal lainya....pemerintah harus cek ulang personel2 nya....` (ANC-1715) | `KPAI_NEGATIVE`. |
| **Menyalahkan akibat tanpa menyebut nama** | `Bye bye prestasi bulutangkis....` (ANC-2613) | `KPAI_NEGATIVE`, beri *flag*. Dampak buruk dipandang akibat tindakan KPAI. |
| **Anti-rokok secara umum** | `Yang bener itu tutup ajah perusahaan rokok di indonesia...` (ANC-2631) | `NEUTRAL`. Tidak menyinggung KPAI, Djarum, audisi, atau merek. |
| **Tagar salah ketik** | `#bubarkanKPI` (ANC-1696) | `KPAI_NEGATIVE`. Di berita tentang KPAI, yang dimaksud adalah KPAI. Tagar tidak disorot (B9). |
| **Menyerang komentator lain** | `Disini pendukung KPAI mana?` … `mana biar gua sembur kasih pencerahan` (ANC-2310) | Meremehkan pendukung KPAI → `KPAI_NEGATIVE`. Tidak ada kata abusif, jadi tanpa `ABUSIVE`. Kalau ada makian ke komentator lain → tambah `ABUSIVE`. |
| **Komentar sangat pendek** | `Cuci tangan...` (ANC-2889) | Tetap dilabeli kalau sasaran jelas menurut M2 → `KPAI_NEGATIVE`. |
| **Komentar sangat panjang** | lebih dari 100 kata | Baca sampai habis. Label mengikuti seluruh isi; semua nama tetap disorot. |
| **Salah ketik dan singkatan** | `KPAI sdh berbuat apa utk negara ini?` | Baca sesuai maksud penulis. Tidak perlu diperbaiki. |
| **Campuran bahasa daerah** | `Bukannya cari solusi buat anak berbakat malah bikin ruwet...iki piye` (ANC-3018) | Sebagian besar bahasa Indonesia → tetap dilabeli (`KPAI_NEGATIVE`). |

**Kalau dua aturan bertentangan**, pakai urutan ini: (1) bisa dipahami atau tidak, (2) maksud penulis menurut M4 dan M5, (3) sasaran menurut M2, (4) kalau masih ragu → label yang lebih hati-hati (`NEUTRAL`) dan beri *flag*.

---

## 6. Teks tidak relevan, tidak jelas, atau tidak dapat dianotasi

Bedakan dua keadaan ini:

| | `NEUTRAL` | `UNANNOTATABLE` |
|---|---|---|
| Teks bisa dipahami? | Ya | Tidak, atau maksudnya tidak bisa ditentukan |
| Ada sikap ke KPAI atau PB Djarum? | Tidak | Tidak bisa dinilai |
| Masuk gold dataset? | Ya | Tidak (dikeluarkan, jumlahnya dilaporkan) |

**Beri `UNANNOTATABLE` jika:**

| Keadaan | Contoh |
|---|---|
| Teks tidak bermakna | `Dyaaaarrrrrrrrr....` (ANC-3117) |
| Seluruh atau hampir seluruh teks berbahasa daerah atau asing | `sakarepmu wae....ramelumelu.....` (ANC-2448) · `PLAY STUPID GAMES WIN STUPID PRIZES.` (ANC-3022) |
| Maksud hanya bisa dipahami dengan melihat komentar lain | `Emang susah...` (ANC-2624) |
| Spam, iklan, atau teks rusak | *(belum ditemukan contohnya di data domain)* |

**Aturan tambahan**

- Komentar `UNANNOTATABLE` **tidak disorot** entitasnya.
- **Di luar topik tetapi bisa dipahami** → `NEUTRAL`, bukan `UNANNOTATABLE`.
- Jangan memakai `UNANNOTATABLE` untuk menghindari kasus sulit. Kasus sulit diselesaikan dengan bab 5 dan *flag*.
- Kalau `UNANNOTATABLE` melebihi 5% pada uji coba, pedoman ini harus ditinjau ulang.

---

## 7. Alur kerja di Prodigy dan kontrol kualitas

> Nama tombol di bawah mengikuti tampilan standar Prodigy. Kalau tampilan di anotasi.uajy.ac.id berbeda, ikuti fungsi tombolnya.

### 7.1 Langkah untuk setiap komentar
1. **Baca** seluruh komentar dan lihat judul beritanya.
2. **Cek dulu:** bisa dipahami? Kalau tidak → centang `UNANNOTATABLE`, tekan **Accept**, lanjut.
3. **NER:** pilih tipe entitas, lalu sorot kata yang sesuai. Ulangi sampai semua nama tersorot. Salah sorot → klik sorotan untuk menghapusnya.
4. **Multilabel:** centang semua label yang berlaku, mengikuti urutan berpikir 3.1.
5. **Periksa:** apakah kombinasi sah (3.4)? Apakah sikap ke KPAI dan ke PB Djarum sudah dinilai terpisah?
6. **Ragu?** Beri *flag* dan tulis alasan singkat di kolom catatan.
7. Tekan **Accept** untuk menyimpan.

### 7.2 Arti tombol
| Tombol | Kapan dipakai |
|---|---|
| **Accept** | Selalu, setelah anotasi selesai, termasuk untuk `NEUTRAL` dan `UNANNOTATABLE`. |
| **Reject** | **Tidak dipakai.** |
| **Ignore** | Hanya kalau teks gagal tampil karena masalah teknis. Laporkan ke tim. |
| **Undo** | Untuk kembali ke komentar sebelumnya bila salah menekan. |
| ***Flag*** | Kasus ragu atau kasus yang belum diatur pedoman. |

`UNANNOTATABLE` harus dicentang sebagai label lalu di-**Accept**, bukan di-**Ignore**, supaya tetap terhitung dalam kesepakatan antar-annotator.

### 7.3 Kontrol kualitas
1. **Uji coba (pilot):** 50 komentar dikerjakan ketiga annotator. Kesepakatan dihitung, perbedaan dibahas, pedoman direvisi, lalu dibekukan menjadi versi 1.0.
2. **Anotasi tumpang-tindih (overlap):** sebagian komentar dikerjakan ketiga annotator untuk menghitung kesepakatan (rencana: 200 komentar). Sisanya dibagi rata dan dikerjakan satu annotator.
3. **Mandiri:** annotator tidak tahu komentar mana yang overlap dan tidak membahas isi komentar sebelum kesepakatan dihitung.
4. **Sesi kerja:** masuk dengan akun atau sesi sendiri. Istirahat setiap kira-kira 50 komentar.
5. **Ukuran kesepakatan:** multilabel dihitung per label (Fleiss' kappa dan persentase kesepakatan); NER dihitung dengan F1 tingkat entitas antar-annotator.
6. **Adjudikasi:** untuk multilabel, label yang dipilih minimal 2 dari 3 annotator diterima. Untuk NER, sorotan yang sama persis pada minimal 2 annotator diterima. Sisanya, dan semua komentar ber-*flag*, diputuskan bersama dan dicatat.
7. **Perubahan pedoman** setelah versi 1.0 harus dicatat di bab 8 beserta alasannya.

---

## 8. Riwayat versi dan keputusan yang perlu dikonfirmasi

| Versi | Tanggal | Perubahan |
|---|---|---|
| 0.1 | — | Draf awal (9 label abusif/sikap, 7 span). Diganti. |
| 0.2 | — | Domain KPAI vs PB Djarum; 6 label sikap; NER 4 tipe entitas. |
| 0.3 | 4 Okt 2026 | Disusun ulang menjadi 8 bab; alat anotasi Prodigy; tiap label punya definisi, aturan beri/jangan, contoh positif, negatif, dan ambigu; batas `ABUSIVE` diperjelas; `GROUP` dibatasi lima kelompok dan kata inti; aturan tagar dan token menempel; contoh ANC-1188 dikoreksi menjadi sindiran. |
| 1.0 | — | Dibekukan setelah uji coba 50 komentar dan tinjauan dosen. |

**Keputusan di versi 0.3 yang perlu dikonfirmasi tim dan dosen sebelum uji coba**

| No | Keputusan | Alternatif |
|---|---|---|
| 1 | Tagar slogan (`#BubarkanKPAI`) tidak disorot sebagai entitas | disorot utuh sebagai `GOV_ORG` |
| 2 | `GROUP` dibatasi lima kelompok dan hanya kata inti | semua sebutan kelompok orang, termasuk kata sifatnya |
| 3 | Jabatan tanpa nama (`Menpora`, `Presiden`, `ketua`) tidak disorot | disorot sebagai `PERSON` atau `GOV_ORG` |
| 4 | `munafik`, `faker`, `lebay`, `baper` bukan `ABUSIVE` | dihitung `ABUSIVE` |
| 5 | Pujian tanpa tanda sindiran dibaca apa adanya | dianggap sindiran karena konteks berita |
| 6 | Teks yang seluruhnya berbahasa daerah → `UNANNOTATABLE` | tetap dilabeli bila annotator paham |
| 7 | Menyalahkan akibat tanpa menyebut nama (`Bye bye prestasi bulutangkis....`) → `KPAI_NEGATIVE` | `NEUTRAL` |

---

## Lampiran A. Contoh anotasi lengkap

#### Contoh 1 (ANC-2385)
```
Teks  : PB Djarum tuh bukan berjasa cari bibit aja tapi pembiayaan sponsor sampe
        jadi atlet sama pelatih juga.. sayang kemampuan otaknya orang2 KPAI
        terbatas..
NER   : [PB Djarum] NONGOV_ORG   [atlet] GROUP   [KPAI] GOV_ORG
Label : PBDJARUM_POSITIVE, KPAI_NEGATIVE, ABUSIVE
Alasan: memuji jasa PB Djarum; menghina kecerdasan orang KPAI.
```

#### Contoh 2 (ANC-2597)
```
Teks  : Menjadi seperti Kevin Sanjaya dan Hendra Setiawan adalah mimpi untuk anak2
        Indonesia, jago bulutangkis dan banyak uang (kaya), sekarang mimpi itu
        dimatikan oleh KPAI
NER   : [Kevin Sanjaya] PERSON   [Hendra Setiawan] PERSON   [anak2] GROUP   [KPAI] GOV_ORG
Label : KPAI_NEGATIVE
Alasan: menyalahkan KPAI; tidak ada penilaian terhadap PB Djarum; tidak ada kata abusif.
```

#### Contoh 3 (ANC-2576)
```
Teks  : anak2 juga tidak boleh sekolah karena eksploitasi, kumpulin semua anak2 di
        KPAI supaya di urusnya, dasar K pea I !
NER   : [anak2] GROUP   [anak2] GROUP   [KPAI] GOV_ORG   [K pea I] GOV_ORG
Label : KPAI_NEGATIVE, ABUSIVE
Alasan: sindiran dengan logika berlebihan; "K pea I" pelesetan nama yang menghina.
```

#### Contoh 4 (ANC-2511)
```
Teks  : udah lah .... cari bibit dan kaderi sasi olah raga cabang apapun ranahnya
        pemerintah khususnya kemenpora .... gitu aja kok repot
NER   : [pemerintah] GOV_ORG   [kemenpora] GOV_ORG
Label : NEUTRAL
Alasan: sikap hanya tertuju ke pemerintah; entitas tetap disorot.
```

#### Contoh 5 (ANC-1937)
```
Teks  : saya mendukung kpai, tindakan kpai sudah tepat, sudah saatnya perusahaan
        rokok tidak menjadi motor pengembangan olahraga di Indonesia. Rokok itu
        meracuni bangsa, larang peredarannya & tutup pabriknya
NER   : [kpai] GOV_ORG   [kpai] GOV_ORG
Label : KPAI_POSITIVE, PBDJARUM_NEGATIVE
Alasan: dukungan tulus ke KPAI; menolak peran perusahaan rokok dalam pembinaan olahraga.
```

#### Contoh 6 (ANC-3117)
```
Teks  : Dyaaaarrrrrrrrr....
NER   : (tidak ada)
Label : UNANNOTATABLE
Alasan: teks tidak bermakna.
```

---

## Lampiran B. Ringkasan satu halaman

**Urutan kerja:** baca → bisa dipahami? → sorot nama → centang label → cek kombinasi → *flag* bila ragu → Accept.

| Pertanyaan | Jawaban → label |
|---|---|
| Tidak bisa dipahami? | `UNANNOTATABLE` (sendiri, tanpa sorotan) |
| Sikap ke KPAI? | dukung → `KPAI_POSITIVE` · kritik atau sindir → `KPAI_NEGATIVE` |
| Sikap ke PB Djarum? | dukung → `PBDJARUM_POSITIVE` · kritik → `PBDJARUM_NEGATIVE` |
| Ada makian atau hinaan? | `ABUSIVE` |
| Tidak ada semuanya? | `NEUTRAL` (sendiri) |

**Ingat untuk multilabel**

- Sasaran tersirat dihitung hanya bila tindakannya khas satu pihak atau ada sebutan pengganti (M2).
- Membela Djarum tidak otomatis menyerang KPAI, dan sebaliknya (M3).
- Sindiran → label negatif. Pujian tanpa tanda sindiran → dibaca apa adanya, beri *flag*.
- Kritik keras bukan `ABUSIVE`. `ABUSIVE` = makian, hinaan kecerdasan, sebutan merendahkan, pelesetan menghina.

**Ingat untuk NER**

- Sorot nama saja: `GOV_ORG`, `NONGOV_ORG`, `PERSON`, `GROUP`.
- Nama resmi utuh; tanda baca tidak ikut; setiap kemunculan disorot; tidak bertumpuk.
- `GROUP` hanya kata inti dari lima kelompok: anak, atlet, orang tua, masyarakat, perokok.
- Tidak disorot: kata ganti, sapaan, jabatan tanpa nama, tagar slogan, tempat, makian.

---

## Lampiran C. Pengaturan label di Prodigy

Label dimasukkan persis seperti di bawah (huruf besar, garis bawah).

| Bagian layar | Label |
|---|---|
| Sorotan teks (NER) | `GOV_ORG`, `NONGOV_ORG`, `PERSON`, `GROUP` |
| Pilihan ganda (boleh lebih dari satu) | `KPAI_POSITIVE`, `KPAI_NEGATIVE`, `PBDJARUM_POSITIVE`, `PBDJARUM_NEGATIVE`, `ABUSIVE`, `NEUTRAL`, `UNANNOTATABLE` |
| Kolom teks | catatan annotator |

- Pilihan label sikap harus diatur sebagai **pilihan ganda**, bukan pilihan tunggal.
- Teks yang diunggah adalah kolom `text_clean`. Kode komentar (`ANC-xxxx`) dan judul berita disertakan sebagai keterangan, supaya hasil anotasi bisa dicocokkan kembali dengan dataset.
- Hasil ekspor Prodigy diubah oleh notebook menjadi format gold dataset:

```json
{"id": "ANC-3000", "text": "KPAI GOBLOK!!!!!",
 "labels": ["KPAI_NEGATIVE", "ABUSIVE"],
 "entities": [{"start": 0, "end": 4, "text": "KPAI", "label": "GOV_ORG"}]}
```
