# Sampling Plan — KelpNLP (UTS NLP)

> Instruksi eksekusi untuk Claude. Output plan ini dipakai untuk membuat `notebook §4.4` (satu-satunya kode sampling yang sah). Jangan ubah cleaning/normalisasi.

## 1. Tujuan

Pilih **1.000 teks** dari corpus domain untuk dianotasi (minimal wajib 800). Cadangan 200 untuk `UNANNOTATABLE` / data rusak.

## 2. Input (jangan diubah)

* File: `dataset/KelpNLP_dataset_clean.csv` (3.157 baris × 23 kolom).
* Kolom kunci: `id` (`ANC-0001`…), `text_clean` (teks anotasi), `text_norm` (abaikan saat sampling), `orig_label` (1/2/3), `in_domain` (True/False), `thread` (0/1/2/3), `annotation_status` (semua `not_annotated`).
* Domain (`in_domain=True`): **1.515 teks** — utas 1: 178 (`ANC-1123–ANC-1300`), utas 2: 400 (`ANC-1599–ANC-2007`), utas 3: 937 (`ANC-2243–ANC-3184`).
* `SEED = 42` untuk semua random. Rerun notebook harus hasilkan file identik.

## 3. Definisi strata (dalam domain saja)

| Strata | Kriteria | Perkiraan |
|---|---|---|
| `abusif` | `orig_label IN (2,3)` | 160 teks (30 / 49 / 81 per utas 1/2/3) — **ambil semua** |
| `langka` | regex pro-KPAI / anti-Djarum di `text_clean` (lowercase), di luar yang sudah `abusif` | ±39 teks |
| `umum` | sisanya | ±1.316 teks |

Regex `langka` (catat pola persis di notebook, hitung overlap dengan `abusif`):

```
pro_kpai = ban iklan rokok|selamatkan generasi|dukung kpai|setuju.*kpai|kpai benar|kpai tepat|melindungi anak|eksploitasi anak|tutup pabrik|larang.*rokok
anti_djarum = promosi rokok|brand.*rokok|merk rokok|logo.*rokok|csr.*rokok|yayasan.*rokok|tanpa pamrih|brainwash|eksploitasi.*audisi
```

## 4. Aturan eksklusi / reduksi (sebelum random)

1. **Near-duplicate `#BubarkanKPAI`**: ada ±7 varian (`#BubarkanKPAI`, `Bubarkan KPAI`, `#BUBARKAN KPAI !!!`, …). Sisakan **1–2 varian** (pilih yang terpanjang / paling representatif), sisanya jangan masuk sampel. Catat ID yang dibuang di notebook.
2. **Blok SpongeBob (35 komentar ANC-1301–1598 yang menyebut KPAI/Djarum)**: sudah `in_domain=False`. Tetap di luar sampel.
3. **Teks ≤2 kata (34 teks) dan >100 kata (18 teks, maks 486)**: tetap boleh masuk sampel. Jangan filter panjang. Guideline yang menangani via `UNANNOTATABLE`.
4. Yang dipakai ke Label Studio/Prodigy adalah **`text_clean`**, bukan `text_norm`. Offset NER dihitung pada `text_clean`.

## 5. Pengambilan sisa sampel

* Kebutuhan: 1.000 − (160 `abusif` + `langka` unik setelah dedup near-duplicate) = **±800–810 teks `umum`**.
* Ambil acak **proporsional per utas** dengan seed: utas 1 ±12% (178/1515), utas 2 ±26% (400/1515), utas 3 ±62% (937/1515).
* Bulatkan agar total pas 1.000.

## 6. Pembagian annotator

| Grup | Jumlah | Isi |
|---|---|---|
| `overlap` (A+B+C) | 200 | stratifikasi utas + **wajib memuat** proporsi `abusif` dan semua `langka`. 50 di antaranya = pilot (tandai `is_pilot=True`) |
| `single_A` | 267 | sisa, acak |
| `single_B` | 267 | sisa, acak |
| `single_C` | 266 | sisa, acak |

Total: 200 + 267 + 267 + 266 = 1.000.

## 7. Output

1. `dataset/pendukung/sample_ids.csv` — kolom persis: `id,thread,strata,split_group,annotator,is_pilot`. `split_group` ∈ {`overlap`,`single`}, `annotator` ∈ {`ABC`,`A`,`B`,`C`}.
2. `dataset/pendukung/label_studio/labelstudio_tasks_{A,B,C}.json` — tiap task: `{"id","text": <text_clean>, "meta": {"thread":…, "split_group":…}}`. Annotator A dapat `overlap` + `single_A`, dst. Jangan sertakan `orig_label` / `text_norm` (blind).
3. Jangan ubah `KelpNLP_dataset_clean.csv` kecuali kolom `annotation_status` → `in_sample` untuk 1.000 ID terpilih (opsional, catat di notebook bila dilakukan).

## 8. Verifikasi (wajib tampil di notebook)

* Total 1.000; overlap 200 + single 800 (267/267/266).
* Semua 160 `abusif` ikut.
* Proporsi utas sampel vs populasi selisih <2pp.
* `sample_ids.csv` stabil bila notebook di-Restart & Run All ulang (cek hash / sort + compare).
* Contoh 3 baris per file task ditampilkan.

## 9. Larangan

* Dilarang mengubah kode/fungsi cleaning (C1–C11), kamus normalisasi, atau isi `text_clean`/`text_norm`.
* Dilarang memakai `text_norm` untuk kriteria strata atau isi task.
* Dilarang menambah notebook baru — kode sampling di `notebook/KelpNLP_dataset_EDA.ipynb §4.4` saja.
