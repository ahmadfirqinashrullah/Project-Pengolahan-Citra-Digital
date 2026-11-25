# Pra-pemrosesan Citra & Baseline Tesseract OCR untuk Tulisan Tangan

Proyek mata kuliah **Pengolahan Citra Digital**. Repositori ini berisi pra-pemrosesan citra berbasis **OpenCV** dan **scikit-image**, lalu mengevaluasi akurasi **Tesseract OCR** pada tulisan tangan di **kertas polos** maupun **kertas bergaris**. Akurasi diukur dengan **CER** (*Character Error Rate*) dan **WER** (*Word Error Rate*).

---

## Daftar Isi

1. [Struktur Direktori](#1-struktur-direktori)
2. [Kelas Dataset](#2-kelas-dataset)
3. [Alur Pemrosesan](#3-alur-pemrosesan)
4. [Ablation Study](#4-ablation-study)
5. [Cara Menjalankan](#5-cara-menjalankan)
6. [Format `ground_truth.csv`](#6-format-ground_truthcsv)
7. [Keluaran](#7-keluaran)

---

## 1. Struktur Direktori

```text
Pengolahan Citra Digital/
├── Dataset/
│   ├── Kertas_Polos-Baris/       # Kelas 1: potongan 1 baris, kertas polos
│   ├── Kertas_Bergaris-Baris/    # Kelas 2: potongan 1 baris, kertas bergaris
│   ├── Kertas_Bergaris/          # Kelas 3: halaman penuh / multi-baris
│   ├── Sekunder/                 # (Opsional) subset dataset publik, mis. IAM Handwriting
│   └── ground_truth.csv          # Teks asli tiap citra, untuk menghitung CER & WER
├── Notebook/
│   └── PCD.ipynb                 # Notebook eksperimen utama
├── hasil_prapemrosesan/          # Dibuat otomatis: citra bersih & laporan CSV
└── README.md
```

> **Penting:** nama folder di dalam `Dataset/` harus sesuai dengan daftar di atas (memakai *underscore*, mis. `Kertas_Polos-Baris`). Notebook tetap menerima penulisan dengan spasi (`Kertas Polos-Baris`) atau berkas `.zip` dengan nama yang sama, tetapi nama dengan *underscore* adalah yang dipakai di Google Drive proyek ini.

---

## 2. Kelas Dataset

| Kelas | Folder | `class_name` | Tata letak | Garis buku |
|:-----:|--------|--------------|------------|:----------:|
| 1 | `Kertas_Polos-Baris` | `polos_single_line` | 1 baris | Tidak |
| 2 | `Kertas_Bergaris-Baris` | `bergaris_single_line` | 1 baris | Ya |
| 3 | `Kertas_Bergaris` | `multi_line` | Banyak baris | Dideteksi otomatis |
| – | `Sekunder` (opsional) | `sekunder` | 1 baris | Tidak |

Untuk Kelas 3, keberadaan garis buku dideteksi otomatis per citra, sehingga halaman polos yang ikut tercampur tetap diproses dengan benar.

---

## 3. Alur Pemrosesan

```text
[Citra Mentah (BGR)]
        │
        ▼
[1. Grayscale & Normalisasi Iluminasi] ──► Estimasi latar (Morphological Closing + Median Blur)
        │
        ▼
[2. Reduksi Noise] ─────────────────────► Median Filter 3×3
        │
        ▼
[3. Binarisasi] ────────────────────────► Otsu (setelah normalisasi) / Sauvola (multi-baris)
        │
        ▼
[4. Hapus Garis Kondisional] ───────────► Jika has_lines=True: Opening kernel horizontal + Closing penyambung
        │
        ▼
[5. Pembersihan Komponen Terhubung] ────► Hapus bintik kecil, sisa garis, dan artefak tepi
        │
        ▼
[6. Koreksi Kemiringan (Deskewing)] ────► Rotasi optimal berdasarkan variansi proyeksi horizontal
        │
        ▼
[7. Segmentasi Baris Viterbi] ──────────► Jika layout_type='multi_line': pemotongan jalur biaya minimum
        │
        ▼
[8. Cropping & Padding Margin Putih] ───► Border putih 25 px (tinta = 0, latar = 255)
        │
        ▼
[9. Tesseract OCR & Evaluasi CER/WER] ──► Ablation Study Exp 0 (baseline) s.d. Exp 4 (final)
```

---

## 4. Ablation Study

Setiap eksperimen menambah satu tahap pra-pemrosesan di atas eksperimen sebelumnya, sehingga kontribusi tiap tahap dapat dibandingkan.

| Eksperimen | Masukan ke Tesseract |
|:----------:|----------------------|
| Exp 0 | Citra mentah (baseline) |
| Exp 1 | + Grayscale, normalisasi iluminasi, dan *denoising* |
| Exp 2 | + Binarisasi |
| Exp 3 | + Penghapusan garis dan pembersihan komponen terhubung |
| Exp 4 | + *Deskewing*, segmentasi baris, *cropping*, dan *padding* (hasil akhir) |

Mode segmentasi halaman Tesseract: `--psm 7` untuk citra 1 baris dan `--psm 6` untuk citra multi-baris (Exp 0–3). Exp 4 selalu memakai `--psm 7` karena citra sudah dipotong per baris. Bahasa OCR: `ind+eng`.

---

## 5. Cara Menjalankan

### Google Colab (disarankan)

1. Unggah folder proyek ke Google Drive, mis. `MyDrive/Semester 7/Pengolahan Citra Digital/`.
2. Buka `Notebook/PCD.ipynb` di Colab.
3. Jalankan sel instalasi paket di bagian paling atas (hapus tanda `#`):

   ```bash
   !apt-get update -qq && apt-get install -y -qq tesseract-ocr tesseract-ocr-ind tesseract-ocr-eng
   !pip install -q opencv-python numpy matplotlib scikit-image pandas pytesseract jiwer
   ```

4. Jalankan seluruh sel secara berurutan. Drive akan di-*mount* otomatis, dan folder proyek dicari di salah satu lokasi berikut:
   - `MyDrive/Semester 7/Pengolahan Citra Digital`
   - `MyDrive/Pengolahan Citra Digital`
   - `MyDrive/Semester 7/PCD`

   Jika folder proyek berada di tempat lain, ubah daftar `kandidat_colab` pada sel kedua.

### Lokal

Instal Tesseract (beserta paket bahasa `ind` dan `eng`) dan paket Python di atas, lalu jalankan notebook dari folder `Notebook/` atau dari folder induk proyek. Folder proyek terdeteksi otomatis selama `Dataset/` berada di dalamnya.

### Jika folder dataset tidak ditemukan

Notebook akan mencetak pesan `[!] Folder/zip '...' tidak ditemukan` beserta isi folder `Dataset/` saat itu. Cocokkan nama folder pada `KONFIG_KELAS` (sel ketiga) dengan nama folder yang sebenarnya.

---

## 6. Format `ground_truth.csv`

Berkas ini berada di `Dataset/ground_truth.csv`. Jika belum ada, notebook membuat templatenya otomatis; tinggal isi kolom `ground_truth` dengan teks asli tiap citra.

| Kolom | Keterangan |
|-------|------------|
| `image_id` | ID unik, mis. `polos_single_line_01_01` |
| `class_name` | Nama kelas (lihat bagian 2) |
| `folder_asal` | Nama folder asal citra |
| `file` | Nama berkas citra |
| `ground_truth` | Teks asli pada citra |

Pencocokan dilakukan berdasarkan `image_id`, atau berdasarkan pasangan `folder_asal` + `file` bila `image_id` tidak cocok. Citra dengan `ground_truth` kosong tetap diproses, tetapi CER/WER-nya tidak dihitung.

---

## 7. Keluaran

Semua hasil disimpan di `hasil_prapemrosesan/`:

| Berkas / Folder | Isi |
|-----------------|-----|
| `<class_name>/<kode>.png` | Citra akhir (tinta hitam, latar putih, margin 25 px) |
| `<class_name>/<kode>_barisNN.png` | Potongan per baris untuk citra multi-baris |
| `ringkasan_prapemrosesan.csv` | Ambang Otsu, piksel garis yang dihapus, komponen yang dibuang, sudut *deskew*, jumlah baris |
| `hasil_evaluasi_ocr.csv` | Prediksi OCR serta CER/WER tiap eksperimen (Exp 0–4) per citra |
