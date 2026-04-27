# Enumerasi Identitas Host dan Pembangunan Workspace Kerja yang Terstruktur

## Latar Belakang

Dalam setiap pekerjaan teknis di bidang keamanan siber — baik itu *penetration testing*, *incident response*, maupun *security monitoring* — langkah pertama yang paling sering diabaikan adalah memastikan bahwa analis benar-benar mengetahui **di mana mereka berada** dan **dengan identitas apa mereka bekerja**. Kesalahan sederhana seperti menjalankan perintah di host yang salah, menyimpan hasil kerja di lokasi yang tidak terstruktur, atau tidak merekam konteks awal dapat menyebabkan bukti yang tidak dapat dipertanggungjawabkan dan analisis yang tidak konsisten.

Proyek ini membangun fondasi kerja yang benar sejak awal: menetapkan struktur workspace yang rapi, merekam identitas kedua host secara eksplisit, dan mendokumentasikan pembagian peran antara Ubuntu Lab dan Kali Linux sebagai referensi yang akan dipakai sepanjang seluruh rangkaian proyek.

---

## Tujuan

- Membangun struktur direktori kerja yang terorganisir dan konsisten pada kedua host
- Mengumpulkan dan menyimpan informasi identitas host sebagai baseline konteks teknis
- Mendokumentasikan perbedaan peran antara Ubuntu Lab dan Kali Linux secara eksplisit
- Membiasakan penggunaan `tee` untuk menyimpan output terminal sekaligus menampilkannya secara langsung

---

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Host utama — pembangunan workspace dan pengumpulan identitas |
| **Kali Linux** | Host validasi — pengumpulan identitas sebagai perbandingan konteks |

---

## Langkah-Langkah yang Dilakukan

### 1. Pembangunan Struktur Workspace pada Ubuntu Lab

Workspace dibangun dengan empat subdirektori yang masing-masing memiliki fungsi spesifik agar hasil kerja selalu terorganisir dan mudah ditelusuri:

```bash
mkdir -p ~/module01/{notes,evidence,reports,tmp}
cd ~/module01
```

| Direktori | Fungsi |
|-----------|--------|
| `notes/` | Catatan teknis, observasi, dan analisis selama sesi kerja |
| `evidence/` | Seluruh output dan artefak yang dikumpulkan sebagai bukti kerja |
| `reports/` | Laporan akhir, checklist, dan arsip yang siap didistribusikan |
| `tmp/` | File sementara yang digunakan selama proses, bukan untuk disimpan sebagai bukti final |

### 2. Pengumpulan Identitas Host — Ubuntu Lab

Serangkaian perintah dijalankan secara berurutan dan seluruh outputnya disimpan ke file bukti menggunakan `tee`. Penggunaan `tee` memungkinkan output tampil di terminal secara langsung sekaligus tersimpan ke file — penting agar analis dapat segera memverifikasi hasil tanpa harus membuka file secara terpisah.

```bash
hostname        | tee  ~/module01/evidence/ubuntu_identity.txt
hostname -I     | tee -a ~/module01/evidence/ubuntu_identity.txt
whoami          | tee -a ~/module01/evidence/ubuntu_identity.txt
echo $SHELL     | tee -a ~/module01/evidence/ubuntu_identity.txt
pwd             | tee -a ~/module01/evidence/ubuntu_identity.txt
```

**Penjelasan tiap perintah:**

| Perintah | Output yang Dihasilkan | Relevansi Keamanan |
|----------|------------------------|-------------------|
| `hostname` | Nama host sistem | Memastikan perintah dijalankan di host yang tepat |
| `hostname -I` | Seluruh alamat IP yang terpasang | Baseline jaringan dan scope kerja |
| `whoami` | Nama user aktif | Memastikan tidak bekerja sebagai root secara tidak sengaja |
| `echo $SHELL` | Shell yang sedang digunakan | Relevan untuk kompatibilitas scripting |
| `pwd` | Direktori kerja aktif saat ini | Konfirmasi lokasi penyimpanan kerja |

### 3. Pengumpulan Identitas Host — Kali Linux

Langkah yang sama diulang pada Kali Linux dan hasilnya disimpan ke file terpisah:

```bash
hostname        | tee  ~/module01/evidence/kali_identity.txt
hostname -I     | tee -a ~/module01/evidence/kali_identity.txt
whoami          | tee -a ~/module01/evidence/kali_identity.txt
echo $SHELL     | tee -a ~/module01/evidence/kali_identity.txt
pwd             | tee -a ~/module01/evidence/kali_identity.txt
```

Pemisahan file identitas antara Ubuntu (`ubuntu_identity.txt`) dan Kali (`kali_identity.txt`) memastikan bahwa seluruh perbandingan dan analisis selanjutnya dapat merujuk kembali ke baseline yang jelas dan tidak ambigu.

### 4. Dokumentasi Konteks Host

Setelah identitas kedua host dikumpulkan, catatan konteks ditulis untuk menegaskan pembagian peran secara eksplisit:

```bash
cat > ~/module01/evidence/host_context_notes.md <<'EOF'
# Catatan Konteks Host

## Ubuntu Lab
- Peran: Primary learning server
- Digunakan untuk: evidence handling, parsing log, scripting, analisis file
- Lokasi workspace: ~/module01

## Kali Linux
- Peran: Validation workstation
- Digunakan untuk: DNS lookup, inspeksi HTTP header, enumerasi service localhost
- Lokasi workspace: ~/module01
EOF
```

---

## Hasil dan Output

**Contoh isi `ubuntu_identity.txt`:**
```
ubuntu-lab
10.10.10.21
student
/bin/bash
/home/student/module01
```

**Contoh isi `kali_identity.txt`:**
```
kali-lab
10.10.10.22
kali
/usr/bin/zsh
/home/kali/module01
```

Perbedaan hostname, alamat IP, nama user, dan shell antara kedua host terlihat jelas — yang membuktikan bahwa keduanya adalah sistem yang benar-benar terpisah dengan identitas berbeda.

---

## Temuan Utama

Meskipun tampak sebagai langkah yang sederhana, pengumpulan identitas host secara eksplisit mengungkap satu hal yang penting: **kedua host memiliki konfigurasi yang berbeda secara signifikan**. Perbedaan shell (bash vs zsh), perbedaan IP, dan perbedaan username menunjukkan bahwa masing-masing host dirancang untuk peran yang berbeda — bukan hanya duplikasi sistem.

Tanpa langkah ini, seorang analis berisiko menjalankan perintah di host yang salah, menyimpan bukti di lokasi yang tidak dapat diverifikasi, atau menarik kesimpulan yang salah karena konteks host tidak direkam.

---

## Apa yang Dipelajari

Disiplin dalam menetapkan konteks sejak awal adalah pembeda antara analis yang hasil kerjanya dapat dipertanggungjawabkan dan analis yang hasil kerjanya tidak dapat direproduksi. Dalam skenario *incident response* atau *forensic investigation* yang nyata, tidak ada bukti yang bernilai jika konteks pengumpulannya tidak tercatat dengan jelas.

Kebiasaan menggunakan `tee` alih-alih hanya `>` juga merupakan praktik yang baik: analis tidak perlu terhenti untuk membuka file dan memverifikasi isinya secara manual — output tampil di terminal secara langsung, sehingga alur kerja lebih efisien.

---

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `ubuntu_identity.txt` | `~/module01/evidence/` | Identitas host Ubuntu |
| `kali_identity.txt` | `~/module01/evidence/` | Identitas host Kali Linux |
| `host_context_notes.md` | `~/module01/evidence/` | Dokumentasi peran dan pembagian fungsi host |
