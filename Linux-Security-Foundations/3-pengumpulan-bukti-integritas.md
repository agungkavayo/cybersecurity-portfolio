# Pengumpulan Bukti Digital dan Verifikasi Integritas File dengan Checksum

## Latar Belakang

Dalam forensik digital dan investigasi keamanan, bukti yang tidak dapat diverifikasi integritasnya adalah bukti yang tidak bernilai. Standar industri — baik dalam *incident response*, audit keamanan, maupun proses hukum — mensyaratkan bahwa setiap artefak digital yang dikumpulkan harus dapat dibuktikan bahwa ia tidak mengalami perubahan sejak saat pengumpulannya. Tanpa mekanisme ini, tidak ada jaminan bahwa bukti yang diserahkan kepada tim manajemen, auditor, atau penegak hukum identik dengan apa yang ditemukan di lapangan.

Proyek ini membangun dan mempraktikkan alur kerja pengelolaan bukti (*evidence handling*) yang terstruktur: dari penataan workspace hingga pembuatan arsip terkompresi dan verifikasi integritas menggunakan SHA256 checksum — sebuah praktik standar yang digunakan dalam investigasi forensik digital profesional.

---

## Tujuan

- Menata workspace menjadi area kerja yang terbagi berdasarkan status dan fungsi evidence
- Membuat inventaris file dan struktur direktori yang dapat diverifikasi
- Mengarsipkan seluruh bukti kerja awal ke dalam satu file terkompresi
- Menghasilkan SHA256 checksum sebagai bukti integritas kriptografis dari arsip tersebut
- Membangun kebiasaan kerja yang mencerminkan standar *chain of custody* dalam forensik digital

---

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Seluruh aktivitas evidence handling dilakukan di sini sebagai *primary workstation* |

---

## Konsep Dasar: Chain of Custody

*Chain of custody* adalah proses dokumentasi yang memastikan bahwa bukti digital dapat dilacak perjalanannya — dari pengumpulan pertama hingga presentasi akhir — tanpa celah yang memungkinkan pihak lain mempertanyakan keasliannya. Komponen utamanya meliputi:

1. **Siapa** yang mengumpulkan bukti (dicatat melalui `$USER` dan `date`)
2. **Kapan** bukti dikumpulkan (timestamp pada file)
3. **Apa** yang dikumpulkan (inventaris file)
4. **Bukti bahwa bukti tidak berubah** (SHA256 checksum)

Proyek ini mempraktikkan seluruh empat komponen tersebut.

---

## Langkah-Langkah yang Dilakukan

### 1. Penataan Struktur Workspace

Workspace yang tidak terstruktur adalah salah satu penyebab paling umum kehilangan bukti dalam investigasi. File yang tersebar tanpa kategori mempersulit penelusuran, menimbulkan kebingungan antara data mentah dan data yang sudah diolah, dan membuat pelaporan menjadi tidak efisien.

```bash
cd ~/module01
mkdir -p evidence/raw evidence/processed reports/archive
```

**Fungsi tiap subdirektori:**

| Direktori | Fungsi |
|-----------|--------|
| `evidence/raw/` | Data mentah langsung dari output tool, belum dimodifikasi |
| `evidence/processed/` | Data yang sudah difilter, diformat, atau dianalisis |
| `reports/archive/` | Arsip final yang siap didistribusikan atau disimpan jangka panjang |

### 2. Pembuatan Catatan Sesi dengan Metadata Analis

```bash
printf "# Module 01 Notes\n\n" > notes/session_notes.md
printf "Analyst: %s\nDate: %s\n" "$USER" "$(date)" >> notes/session_notes.md
```

Pencatatan nama analis (`$USER`) dan waktu sesi (`date`) langsung ke dalam file catatan memastikan bahwa setiap dokumen memiliki metadata yang dapat diverifikasi — siapa yang mengerjakan dan kapan.

### 3. Pembuatan Inventaris Workspace

```bash
find ~/module01 -maxdepth 2 -type d | sort | tee ~/module01/evidence/workspace_tree.txt
find ~/module01/notes -maxdepth 2 -type f | sort | tee ~/module01/evidence/notes_inventory.txt
```

`workspace_tree.txt` mendokumentasikan seluruh struktur direktori yang ada, sementara `notes_inventory.txt` mencatat semua file catatan yang telah dibuat. Kedua file ini akan menjadi referensi jika di kemudian hari ada pertanyaan tentang apa saja yang dikerjakan dalam sesi ini.

### 4. Pengarsipan Evidence ke File Terkompresi

```bash
tar -czf ~/module01/reports/module01_workspace_initial.tar.gz \
    ~/module01/notes \
    ~/module01/evidence
```

**Penjelasan flag `tar`:**

| Flag | Fungsi |
|------|--------|
| `-c` | *Create* — buat arsip baru |
| `-z` | Kompres menggunakan gzip |
| `-f` | Tentukan nama file output |

Penggunaan `tar.gz` memastikan bahwa seluruh struktur direktori, termasuk path relatif dan metadata file, tersimpan dengan benar di dalam arsip. File ini kemudian dapat dipindahkan, dibagikan, atau disimpan sebagai snapshot permanen dari kondisi workspace pada titik waktu tertentu.

### 5. Pembuatan SHA256 Checksum

```bash
sha256sum ~/module01/reports/module01_workspace_initial.tar.gz | tee ~/module01/evidence/archive_sha256.txt
ls -lh ~/module01/reports/module01_workspace_initial.tar.gz
```

SHA256 (*Secure Hash Algorithm 256-bit*) menghasilkan string 64 karakter heksadesimal yang unik untuk setiap file. Sifat kriptografis SHA256 memastikan bahwa:

- Perubahan sekecil apapun pada isi file (bahkan satu karakter) akan menghasilkan hash yang sepenuhnya berbeda
- Tidak ada dua file yang berbeda yang dapat menghasilkan hash yang sama (*collision resistance*)
- Hash tidak dapat dibalik untuk mendapatkan isi file asli (*one-way function*)

Dengan menyimpan hash ini, siapapun yang kemudian menerima file arsip tersebut dapat menjalankan `sha256sum` dan membandingkan hasilnya dengan nilai yang tersimpan — jika cocok, file terbukti tidak mengalami perubahan.

---

## Hasil dan Output

**Contoh isi `workspace_tree.txt`:**
```
/home/student/module01
/home/student/module01/evidence
/home/student/module01/evidence/raw
/home/student/module01/evidence/processed
/home/student/module01/notes
/home/student/module01/reports
/home/student/module01/reports/archive
/home/student/module01/tmp
```

**Contoh isi `archive_sha256.txt`:**
```
a3f92c1d4b8e7f2a1c9d0e5b6f3a8c2d1e4f7b9a0c3d6e9f2a5b8c1d4e7f0a2  module01_workspace_initial.tar.gz
```

**Verifikasi integritas (simulasi):**
```bash
sha256sum -c archive_sha256.txt
# Output: module01_workspace_initial.tar.gz: OK
```

---

## Temuan Utama

Proses ini mengungkap betapa mudahnya sebuah file dapat dimodifikasi tanpa jejak yang terlihat secara kasat mata. Tanpa checksum, tidak ada cara untuk membedakan antara file yang asli dan file yang telah diubah — ukuran file bisa saja sama, nama file sama, bahkan tanggal modifikasi bisa dimanipulasi. SHA256 adalah satu-satunya mekanisme sederhana yang memberikan jaminan kriptografis bahwa isi file tidak berubah.

Ukuran arsip yang dihasilkan juga memberikan gambaran tentang volume evidence yang dikumpulkan — informasi yang berguna untuk perencanaan kapasitas penyimpanan dalam investigasi skala besar.

---

## Apa yang Dipelajari

Pengelolaan bukti yang profesional bukan hanya soal "menyimpan file". Ini adalah proses yang terstruktur dengan tujuan yang jelas: memastikan bahwa setiap artefak dapat diverifikasi, dilacak, dan dipertanggungjawabkan. Standar ini berlaku baik dalam investigasi internal perusahaan maupun dalam proses hukum formal.

Kebiasaan membuat checksum setelah setiap pengumpulan evidence — seberapapun kecilnya — adalah investasi waktu yang sangat kecil namun dapat mencegah masalah besar di kemudian hari ketika integritas bukti dipertanyakan.

---

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `workspace_tree.txt` | `~/module01/evidence/` | Struktur direktori workspace |
| `notes_inventory.txt` | `~/module01/evidence/` | Inventaris file catatan |
| `archive_sha256.txt` | `~/module01/evidence/` | SHA256 checksum dari arsip evidence |
| `module01_workspace_initial.tar.gz` | `~/module01/reports/` | Arsip terkompresi seluruh evidence awal |
