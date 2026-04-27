# Audit Izin File dan Tinjauan Metadata Sistem pada Lingkungan Linux

## Latar Belakang

Kesalahan konfigurasi izin file (*file permission misconfiguration*) adalah salah satu temuan paling umum dalam audit keamanan sistem Linux. File konfigurasi yang dapat dibaca oleh semua pengguna, skrip yang dapat dieksekusi oleh siapapun, atau direktori yang memberikan akses tulis kepada grup yang tidak seharusnya — semuanya adalah vektor yang dapat dieksploitasi untuk eskalasi privilase (*privilege escalation*) atau pencurian data.

Proyek ini membangun kemampuan untuk membaca, mengkonfigurasi, mengaudit, dan menginterpretasikan izin file Linux secara profesional. Tidak hanya pada level file individual, tetapi juga pada level path direktori secara keseluruhan — karena dalam praktik nyata, izin sebuah file hanya dapat dimaknai dalam konteks izin direktori induknya.

---

## Tujuan

- Mengkonfigurasi izin file menggunakan notasi oktal dan memahami implikasi keamanannya
- Menggunakan `ls -l`, `stat`, dan `namei` untuk membaca metadata dan izin secara komprehensif
- Memahami perbedaan antara izin file dan izin direktori dalam konteks akses efektif
- Menemukan file yang berpotensi relevan untuk analisis keamanan menggunakan `find` dengan filter tipe dan ekstensi
- Mendokumentasikan temuan dalam format yang siap digunakan dalam laporan audit

---

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Seluruh aktivitas audit izin dan metadata dilakukan di sini |

---

## Konsep Dasar: Sistem Izin Linux

Sebelum langkah-langkah dilakukan, penting untuk memahami sistem izin Linux yang digunakan sebagai referensi sepanjang proyek ini.

### Tiga Kelas Pengguna

| Kelas | Simbol | Keterangan |
|-------|--------|------------|
| Owner | `u` | Pemilik file — biasanya user yang membuatnya |
| Group | `g` | Grup yang diasosiasikan dengan file |
| Others | `o` | Semua pengguna lain di sistem |

### Tiga Jenis Izin

| Izin | Simbol | Nilai Oktal | Pada File | Pada Direktori |
|------|--------|-------------|-----------|----------------|
| Read | `r` | `4` | Dapat membaca isi file | Dapat melihat daftar isi direktori |
| Write | `w` | `2` | Dapat memodifikasi file | Dapat membuat/menghapus file di dalam direktori |
| Execute | `x` | `1` | Dapat menjalankan file sebagai program | Dapat masuk ke dalam direktori (`cd`) |

### Notasi Oktal

Nilai oktal dihitung dengan menjumlahkan nilai izin yang diberikan:

| Oktal | Biner | Izin | Keterangan |
|-------|-------|------|------------|
| `7` | `111` | `rwx` | Akses penuh |
| `6` | `110` | `rw-` | Baca dan tulis |
| `5` | `101` | `r-x` | Baca dan eksekusi |
| `4` | `100` | `r--` | Hanya baca |
| `0` | `000` | `---` | Tidak ada akses |

---

## Langkah-Langkah yang Dilakukan

### 1. Navigasi dan Pembacaan Struktur Direktori

```bash
printf "=== HOME ===\n" | tee ~/module01/evidence/filesystem_navigation.txt
pwd | tee -a ~/module01/evidence/filesystem_navigation.txt
ls -la ~ | tee -a ~/module01/evidence/filesystem_navigation.txt

printf "\n=== DATASETS ===\n" | tee -a ~/module01/evidence/filesystem_navigation.txt
find ~/datasets -maxdepth 2 -type d | sort | tee -a ~/module01/evidence/filesystem_navigation.txt

printf "\n=== STARTER ===\n" | tee -a ~/module01/evidence/filesystem_navigation.txt
find ~/starter -maxdepth 2 -type f | sort | tee -a ~/module01/evidence/filesystem_navigation.txt
```

`ls -la` menampilkan seluruh file termasuk file tersembunyi (yang diawali dengan `.`), lengkap dengan izin, jumlah *hard link*, owner, grup, ukuran, dan timestamp modifikasi terakhir.

### 2. Konfigurasi Izin File

Tiga file dibuat dan dikonfigurasi dengan izin yang berbeda untuk mencerminkan skenario nyata dalam pengelolaan file di server:

```bash
mkdir -p ~/module01/filesystem/labdata
cd ~/module01/filesystem/labdata
touch note.txt report.txt script.sh

chmod 600 note.txt    # Catatan sensitif — hanya owner
chmod 640 report.txt  # Laporan — owner bisa edit, grup hanya baca
chmod 750 script.sh   # Script — owner penuh, grup bisa jalankan
```

**Analisis izin tiap file:**

**`note.txt` — Mode `600` (`rw-------`):**
Hanya owner yang dapat membaca dan menulis. Tidak ada akses sama sekali untuk grup dan pengguna lain. Cocok untuk: file konfigurasi yang berisi password, private key, atau data sensitif lainnya.

**`report.txt` — Mode `640` (`rw-r-----`):**
Owner dapat membaca dan menulis. Anggota grup hanya dapat membaca. Pengguna lain tidak memiliki akses. Cocok untuk: laporan atau dokumen yang perlu dibagi dalam tim tanpa memberikan hak edit kepada seluruh anggota.

**`script.sh` — Mode `750` (`rwxr-x---`):**
Owner memiliki akses penuh. Anggota grup dapat membaca dan menjalankan skrip, namun tidak dapat memodifikasinya. Pengguna lain tidak memiliki akses. Cocok untuk: skrip otomasi yang perlu dijalankan oleh tim operasi namun tidak boleh dimodifikasi.

### 3. Audit Izin dengan `ls -l`

```bash
ls -l | tee ~/module01/evidence/permission_review.txt
```

**Output:**
```
-rw-------  1 student student   0 Apr 27 10:00 note.txt
-rw-r-----  1 student student   0 Apr 27 10:00 report.txt
-rwxr-x---  1 student student   0 Apr 27 10:00 script.sh
```

### 4. Inspeksi Metadata Detail dengan `stat`

```bash
stat note.txt report.txt script.sh | tee ~/module01/evidence/metadata_review.txt
```

`stat` mengungkap metadata yang tidak terlihat di `ls -l`, termasuk:

| Field | Keterangan |
|-------|------------|
| `Inode` | Nomor inode — identitas unik file di filesystem |
| `Links` | Jumlah *hard link* yang menunjuk ke inode ini |
| `Access` | Waktu terakhir file dibaca |
| `Modify` | Waktu terakhir isi file diubah |
| `Change` | Waktu terakhir metadata file diubah |
| `Birth` | Waktu file pertama kali dibuat (jika didukung filesystem) |

**Contoh output `stat` untuk `note.txt`:**
```
  File: note.txt
  Size: 0               Blocks: 0          IO Block: 4096   regular empty file
Device: fd00h/64768d    Inode: 131073      Links: 1
Access: (0600/-rw-------)  Uid: (1000/student)   Gid: (1000/student)
Access: 2024-04-27 10:00:00.000
Modify: 2024-04-27 10:00:00.000
Change: 2024-04-27 10:00:01.000
```

### 5. Audit Path Lengkap dengan `namei`

```bash
namei -l ~/module01/filesystem/labdata | tee -a ~/module01/evidence/metadata_review.txt
```

`namei -l` menelusuri setiap komponen dari path yang diberikan dan menampilkan izin di setiap level. Ini adalah tool yang sangat penting dalam audit keamanan karena **izin efektif sebuah file ditentukan oleh kombinasi izin file DAN semua direktori dalam pathnya**.

**Contoh output:**
```
f: /home/student/module01/filesystem/labdata
drwxr-xr-x root    root     /
drwxr-xr-x root    root     home
drwx------ student student  student
drwxr-xr-x student student  module01
drwxr-xr-x student student  filesystem
drwxr-xr-x student student  labdata
```

Jika salah satu direktori dalam path memiliki izin `---` untuk pengguna tertentu, pengguna tersebut tidak akan bisa mengakses file di dalamnya — bahkan jika file itu sendiri memiliki izin `777`.

### 6. Pencarian File Relevan untuk Analisis Keamanan

```bash
find ~/datasets ~/samples -maxdepth 3 -type f \
  \( -iname "*.log" -o -iname "*.json" -o -iname "*.pcap" \
     -o -iname "*.csv"  -o -iname "*.txt" \) | sort | \
  tee ~/module01/evidence/risky_strings_findings.txt
```

**Penjelasan flag `find`:**

| Flag/Opsi | Fungsi |
|-----------|--------|
| `-maxdepth 3` | Batasi pencarian hingga 3 level direktori ke dalam |
| `-type f` | Hanya cari file (bukan direktori atau symbolic link) |
| `-iname "*.log"` | Cocokkan nama file dengan ekstensi `.log` (tidak peka huruf besar/kecil) |
| `-o` | Operator OR — cocokkan salah satu dari kondisi |

---

## Skenario Risiko yang Ditemukan

Dari hasil audit, tiga skenario risiko umum dapat diidentifikasi:

**Skenario 1 — File konfigurasi dengan izin terlalu longgar:**
Jika `note.txt` yang berisi kredensial dikonfigurasi dengan `644` alih-alih `600`, seluruh pengguna di sistem dapat membacanya. Ini adalah temuan umum dalam audit server produksi.

**Skenario 2 — Skrip yang dapat dimodifikasi oleh grup:**
Jika `script.sh` dikonfigurasi dengan `770` alih-alih `750`, anggota grup dapat memodifikasi isinya — potensial untuk injeksi kode berbahaya (*script injection*).

**Skenario 3 — Direktori dengan izin yang tidak tepat:**
Jika direktori `labdata` memiliki izin `777`, pengguna manapun dapat membuat, menghapus, atau mengganti nama file di dalamnya — bahkan file yang mereka sendiri tidak memiliki izin untuk membacanya.

---

## Apa yang Dipelajari

Membaca izin file di Linux membutuhkan pemahaman yang lebih dari sekadar angka. Izin efektif sebuah file adalah hasil kombinasi dari izin file itu sendiri dan izin seluruh direktori dalam pathnya. Tool `namei -l` memberikan gambaran lengkap tentang hal ini dalam sekali jalankan.

Kemampuan audit izin seperti ini secara langsung relevan dalam skenario: *penetration testing* (mencari file yang salah konfigurasi), *incident response* (memverifikasi apakah file sistem penting telah dimodifikasi), dan *security hardening* (memastikan konfigurasi server memenuhi standar keamanan minimum).

---

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `filesystem_navigation.txt` | `~/module01/evidence/` | Output navigasi direktori home, datasets, dan starter |
| `permission_review.txt` | `~/module01/evidence/` | Output `ls -l` dengan detail izin tiga file uji |
| `metadata_review.txt` | `~/module01/evidence/` | Output `stat` dan `namei` untuk metadata dan path audit |
| `risky_strings_findings.txt` | `~/module01/evidence/` | Daftar file yang ditemukan melalui pencarian berbasis ekstensi |
