## Tentang Portofolio Ini

Portofolio ini mendokumentasikan serangkaian proyek keamanan siber yang dikerjakan secara langsung (*hands-on*) di lingkungan lab terisolasi. Setiap proyek disusun bukan sebagai catatan akademis, melainkan sebagai dokumentasi teknis yang mencerminkan cara kerja seorang *security analyst* atau *security engineer* profesional — mulai dari pengumpulan bukti, analisis log, validasi jaringan, audit izin sistem, hingga identifikasi indikator ancaman.

Seluruh aktivitas dilakukan pada dua host lab dengan peran yang berbeda:

| Host | Peran |
|------|-------|
| **Ubuntu Lab** | *Primary workstation* — pengumpulan bukti, analisis log, scripting, pengelolaan file |
| **Kali Linux** | *Validation workstation* — DNS lookup, inspeksi HTTP header, enumerasi service localhost |

> ⚠️ **Catatan:** Seluruh aktivitas dilakukan **hanya** pada lingkungan lab yang telah disediakan dan terisolasi. Tidak ada pemindaian, pengujian, pengumpulan data, atau perubahan konfigurasi yang dilakukan pada sistem di luar cakupan lab.

---

## Daftar Proyek

| No. | Judul Proyek | Fokus Area | Tools Utama |
|-----|-------------|------------|-------------|
| 1 | [Enumerasi Identitas Host dan Pembangunan Workspace Kerja yang Terstruktur](#proyek-1--enumerasi-identitas-host-dan-pembangunan-workspace-kerja-yang-terstruktur) | Host context, workspace management | `hostname`, `whoami`, `bash`, `tee` |
| 2 | [Validasi Resolusi DNS dan Inspeksi HTTP Header pada Target Eksternal dan Lokal](#proyek-2--validasi-resolusi-dns-dan-inspeksi-http-header-pada-target-eksternal-dan-lokal) | Network validation, service inspection | `dig`, `getent`, `curl`, `nmap`, `ss` |
| 3 | [Pengumpulan Bukti Digital dan Verifikasi Integritas File dengan Checksum](#proyek-3--pengumpulan-bukti-digital-dan-verifikasi-integritas-file-dengan-checksum) | Evidence handling, digital forensics | `tar`, `sha256sum`, `find` |
| 4 | [Deteksi Pola Mencurigakan pada Log Aktivitas Sistem dengan Shell dan Python](#proyek-4--deteksi-pola-mencurigakan-pada-log-aktivitas-sistem-dengan-shell-dan-python) | Log analysis, threat detection | `grep`, `awk`, `bash`, `python` |
| 5 | [Audit Izin File dan Tinjauan Metadata Sistem pada Lingkungan Linux](#proyek-5--audit-izin-file-dan-tinjauan-metadata-sistem-pada-lingkungan-linux) | Permission review, file system audit | `chmod`, `stat`, `namei`, `find` |
| 6 | [Identifikasi Indikator Kompromi pada Sampel Artefak dan Data Lab](#proyek-6--identifikasi-indikator-kompromi-pada-sampel-artefak-dan-data-lab) | IOC identification, security awareness | `grep -E`, phishing analysis |

---

## Teknologi dan Tools

<p>
  <img src="https://img.shields.io/badge/OS-Ubuntu%20Linux-E95420?style=flat-square&logo=ubuntu&logoColor=white"/>
  <img src="https://img.shields.io/badge/OS-Kali%20Linux-557C94?style=flat-square&logo=kalilinux&logoColor=white"/>
  <img src="https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat-square&logo=gnubash&logoColor=white"/>
  <img src="https://img.shields.io/badge/Python-3.x-3776AB?style=flat-square&logo=python&logoColor=white"/>
  <img src="https://img.shields.io/badge/Tool-Nmap-red?style=flat-square"/>
  <img src="https://img.shields.io/badge/Tool-curl-073551?style=flat-square"/>
  <img src="https://img.shields.io/badge/Tool-dig%20%2F%20getent-informational?style=flat-square"/>
</p>

---

## Kompetensi yang Dibangun

- **Linux CLI** — Navigasi terminal, manajemen file, pencarian berbasis `find`, dan pembacaan struktur direktori secara efisien
- **Network Validation** — Validasi resolusi nama domain, pembacaan HTTP response header, dan enumerasi service dari perspektif internal maupun eksternal
- **Evidence Handling** — Pengarsipan bukti kerja, pembuatan checksum SHA256, dan pengelolaan rantai kepemilikan (*chain of custody*) artefak digital
- **Log Analysis** — Filtering, counting, dan summarization log sistem menggunakan shell pipeline dan Python
- **Permission & Metadata Review** — Pembacaan dan konfigurasi izin file Linux dengan notasi oktal, inspeksi metadata dengan `stat` dan `namei`
- **Security Awareness** — Analisis artefak phishing, dokumentasi prinsip keamanan kredensial, dan pemindaian string berisiko pada dataset

---
---

# Proyek 1 — Enumerasi Identitas Host dan Pembangunan Workspace Kerja yang Terstruktur

## Latar Belakang

Dalam setiap pekerjaan teknis di bidang keamanan siber — baik itu *penetration testing*, *incident response*, maupun *security monitoring* — langkah pertama yang paling sering diabaikan adalah memastikan bahwa analis benar-benar mengetahui **di mana mereka berada** dan **dengan identitas apa mereka bekerja**. Kesalahan sederhana seperti menjalankan perintah di host yang salah, menyimpan hasil kerja di lokasi yang tidak terstruktur, atau tidak merekam konteks awal dapat menyebabkan bukti yang tidak dapat dipertanggungjawabkan dan analisis yang tidak konsisten.

Proyek ini membangun fondasi kerja yang benar sejak awal: menetapkan struktur workspace yang rapi, merekam identitas kedua host secara eksplisit, dan mendokumentasikan pembagian peran antara Ubuntu Lab dan Kali Linux sebagai referensi yang akan dipakai sepanjang seluruh rangkaian proyek.

## Tujuan

- Membangun struktur direktori kerja yang terorganisir dan konsisten pada kedua host
- Mengumpulkan dan menyimpan informasi identitas host sebagai baseline konteks teknis
- Mendokumentasikan perbedaan peran antara Ubuntu Lab dan Kali Linux secara eksplisit
- Membiasakan penggunaan `tee` untuk menyimpan output terminal sekaligus menampilkannya secara langsung

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Host utama — pembangunan workspace dan pengumpulan identitas |
| **Kali Linux** | Host validasi — pengumpulan identitas sebagai perbandingan konteks |

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

## Temuan Utama

Meskipun tampak sebagai langkah yang sederhana, pengumpulan identitas host secara eksplisit mengungkap satu hal yang penting: **kedua host memiliki konfigurasi yang berbeda secara signifikan**. Perbedaan shell (bash vs zsh), perbedaan IP, dan perbedaan username menunjukkan bahwa masing-masing host dirancang untuk peran yang berbeda — bukan hanya duplikasi sistem.

Tanpa langkah ini, seorang analis berisiko menjalankan perintah di host yang salah, menyimpan bukti di lokasi yang tidak dapat diverifikasi, atau menarik kesimpulan yang salah karena konteks host tidak direkam.

## Apa yang Dipelajari

Disiplin dalam menetapkan konteks sejak awal adalah pembeda antara analis yang hasil kerjanya dapat dipertanggungjawabkan dan analis yang hasil kerjanya tidak dapat direproduksi. Dalam skenario *incident response* atau *forensic investigation* yang nyata, tidak ada bukti yang bernilai jika konteks pengumpulannya tidak tercatat dengan jelas.

Kebiasaan menggunakan `tee` alih-alih hanya `>` juga merupakan praktik yang baik: analis tidak perlu terhenti untuk membuka file dan memverifikasi isinya secara manual — output tampil di terminal secara langsung, sehingga alur kerja lebih efisien.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `ubuntu_identity.txt` | `~/module01/evidence/` | Identitas host Ubuntu |
| `kali_identity.txt` | `~/module01/evidence/` | Identitas host Kali Linux |
| `host_context_notes.md` | `~/module01/evidence/` | Dokumentasi peran dan pembagian fungsi host |

---
---

# Proyek 2 — Validasi Resolusi DNS dan Inspeksi HTTP Header pada Target Eksternal dan Lokal

## Latar Belakang

Pemahaman tentang bagaimana nama domain diterjemahkan menjadi alamat IP, bagaimana server merespons permintaan HTTP, dan bagaimana service berjalan di atas port tertentu merupakan kompetensi dasar yang mutlak dibutuhkan dalam keamanan siber. Tanpa kemampuan ini, seorang analis tidak dapat membedakan antara service yang berjalan normal dan service yang berpotensi terekspos secara tidak semestinya.

Proyek ini memvalidasi seluruh rantai konektivitas dasar — dari resolusi nama domain hingga pembacaan HTTP response header dan enumerasi service localhost — menggunakan Kali Linux sebagai *validation workstation*. Pendekatan yang digunakan sengaja mengkombinasikan dua atau lebih tool untuk setiap tahap validasi, karena dalam praktik nyata, satu tool saja tidak cukup untuk memberikan gambaran yang lengkap dan akurat.

## Tujuan

- Memvalidasi resolusi nama domain menggunakan dua pendekatan yang berbeda (`dig` dan `getent`) dan menganalisis perbedaan hasilnya
- Membaca dan menginterpretasikan HTTP response header dari target eksternal maupun target lokal di dalam lab
- Mengenumerasi service yang aktif pada localhost dari perspektif internal sistem (`ss`) dan perspektif eksternal assessment (`nmap`)
- Membangun kebiasaan membaca output jaringan secara kritis, bukan sekadar menjalankan perintah

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Kali Linux** | Host validasi — seluruh aktivitas jaringan dijalankan dari sini |
| **Ubuntu Lab** | Tidak digunakan secara aktif — hasil dari Kali disinkronkan ke Ubuntu untuk penyimpanan evidence terpusat |

## Langkah-Langkah yang Dilakukan

### 1. Validasi Resolusi DNS

DNS (*Domain Name System*) adalah fondasi dari seluruh komunikasi internet. Sebelum koneksi ke sebuah server dapat terjadi, nama domain harus terlebih dahulu diterjemahkan menjadi alamat IP. Proses ini dilakukan oleh resolver DNS, dan hasilnya dapat bervariasi tergantung pada tool dan konfigurasi sistem yang digunakan.

```bash
printf "=== DNS LOOKUP ===\n" | tee ~/module01/evidence/dns_check.txt
dig openai.com +short    | tee -a ~/module01/evidence/dns_check.txt
dig example.com +short   | tee -a ~/module01/evidence/dns_check.txt
getent hosts example.com | tee -a ~/module01/evidence/dns_check.txt
```

**Perbedaan antara `dig` dan `getent`:**

| Aspek | `dig` | `getent hosts` |
|-------|-------|----------------|
| Sumber resolusi | DNS resolver (sistem atau custom) | OS name resolution stack (termasuk `/etc/hosts`) |
| Konfigurasi yang mempengaruhi | `/etc/resolv.conf` | `/etc/nsswitch.conf` + `/etc/hosts` |
| Kegunaan dalam security | Validasi DNS server, TTL, authoritative record | Mendeteksi entri lokal yang meng-override DNS |
| Output | Hanya IP (dengan `+short`) | IP + nama host |

Perbedaan hasil antara `dig` dan `getent` pada domain yang sama dapat mengindikasikan adanya entri DNS lokal yang meng-override resolusi publik — sebuah temuan yang relevan dalam skenario *DNS poisoning* atau konfigurasi *split-horizon DNS*.

### 2. Inspeksi HTTP Response Header

HTTP header memberikan informasi yang sangat berharga tentang server target: teknologi yang digunakan, versi software, kebijakan keamanan yang diterapkan, dan perilaku redirect. Perintah `curl -I` hanya mengambil header respons tanpa mengunduh isi halaman, sehingga lebih efisien untuk validasi cepat.

```bash
printf "=== HTTP HEADERS EXTERNAL ===\n" | tee ~/module01/evidence/http_headers.txt
curl -I https://example.com | tee -a ~/module01/evidence/http_headers.txt

printf "\n=== HTTP HEADERS LOCAL TARGETS ===\n" | tee -a ~/module01/evidence/http_headers.txt
curl -I http://127.0.0.1:3000 | tee -a ~/module01/evidence/http_headers.txt
curl -I http://127.0.0.1:8081 | tee -a ~/module01/evidence/http_headers.txt
```

**Header yang diperhatikan dan relevansinya:**

| Header | Contoh Nilai | Relevansi Keamanan |
|--------|-------------|-------------------|
| `HTTP/2 200` | Status code OK | Konfirmasi service aktif dan merespons |
| `Server` | `nginx/1.18.0` | Identifikasi teknologi server (fingerprinting) |
| `X-Powered-By` | `Express` | Identifikasi framework backend |
| `Content-Security-Policy` | `default-src 'self'` | Kebijakan keamanan konten yang diterapkan |
| `Strict-Transport-Security` | `max-age=31536000` | Apakah HSTS diterapkan |
| `Location` | `https://...` | Target redirect — penting untuk mengikuti alur otentikasi |

Header `Server` dan `X-Powered-By` secara khusus relevan karena mengungkap teknologi yang digunakan — informasi ini dapat dimanfaatkan penyerang untuk mencari *known vulnerabilities* pada versi spesifik software tersebut.

### 3. Enumerasi Service Localhost — Perspektif Ganda

Service yang sama dilihat dari dua sudut pandang yang berbeda: dari dalam sistem (*internal view*) dan dari luar (*external/assessment view*).

**Perspektif Internal — `ss`:**
```bash
ss -tulpn | head -n 30 | tee ~/module01/evidence/localhost_service_validation.txt
```

| Flag | Fungsi |
|------|--------|
| `-t` | Tampilkan koneksi TCP |
| `-u` | Tampilkan koneksi UDP |
| `-l` | Hanya socket yang sedang *listening* |
| `-p` | Tampilkan proses yang memiliki socket |
| `-n` | Jangan resolve nama, tampilkan angka |

**Perspektif Eksternal — `nmap`:**
```bash
nmap -Pn -sV -p 22,3000,8081 127.0.0.1 | tee -a ~/module01/evidence/localhost_service_validation.txt
```

| Flag | Fungsi |
|------|--------|
| `-Pn` | Lewati host discovery, asumsikan host aktif |
| `-sV` | Lakukan service version detection |
| `-p` | Tentukan port yang akan dipindai |

## Hasil dan Output

**Contoh output `dns_check.txt`:**
```
=== DNS LOOKUP ===
13.107.21.200
93.184.216.34
93.184.216.34 example.com
```

**Contoh output `http_headers.txt`:**
```
=== HTTP HEADERS EXTERNAL ===
HTTP/2 200
server: ECAcc (nyd/D10E)
content-type: text/html; charset=UTF-8

=== HTTP HEADERS LOCAL TARGETS ===
HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: text/html; charset=utf-8
```

**Contoh output `localhost_service_validation.txt`:**
```
Netid  State   Local Address:Port
tcp    LISTEN  0.0.0.0:22
tcp    LISTEN  0.0.0.0:3000
tcp    LISTEN  0.0.0.0:8081

PORT     STATE  SERVICE  VERSION
22/tcp   open   ssh      OpenSSH 8.9p1
3000/tcp open   http     Node.js Express
8081/tcp open   http     nginx 1.18.0
```

## Temuan Utama

**Perbedaan perspektif `ss` vs `nmap`** adalah temuan teknis yang paling signifikan dari proyek ini. Dalam pengujian nyata, ada kasus di mana sebuah port tampak tertutup dari pemindaian eksternal namun sebenarnya terbuka dan dapat diakses — hal ini terjadi karena aturan firewall yang hanya memblokir traffic dari interface tertentu namun tidak dari localhost. Memahami perbedaan ini menghindarkan analis dari kesimpulan yang keliru bahwa sebuah service "aman" hanya karena tidak terlihat dari satu sudut pandang.

Header `X-Powered-By: Express` pada target lokal juga merupakan temuan yang perlu dicatat: header ini mengekspos teknologi backend yang digunakan. Dalam lingkungan produksi, header ini sebaiknya dinonaktifkan untuk mengurangi informasi yang tersedia bagi penyerang.

## Apa yang Dipelajari

Validasi jaringan yang benar tidak pernah bergantung pada satu tool. Menggunakan `dig` saja tanpa `getent` bisa menyebabkan analis melewatkan override DNS lokal. Menggunakan `ss` saja tanpa `nmap` bisa menyebabkan analis berpikir bahwa semua service terlihat dari luar — padahal tidak selalu demikian. Kebiasaan menggunakan minimal dua tool untuk setiap validasi jaringan adalah standar kerja yang perlu dibangun sejak awal.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `dns_check.txt` | `~/module01/evidence/` | Hasil resolusi DNS dengan `dig` dan `getent` |
| `http_headers.txt` | `~/module01/evidence/` | HTTP response header dari target eksternal dan lokal |
| `localhost_service_validation.txt` | `~/module01/evidence/` | Output `ss` dan `nmap` untuk enumerasi service |

---
---

# Proyek 3 — Pengumpulan Bukti Digital dan Verifikasi Integritas File dengan Checksum

## Latar Belakang

Dalam forensik digital dan investigasi keamanan, bukti yang tidak dapat diverifikasi integritasnya adalah bukti yang tidak bernilai. Standar industri — baik dalam *incident response*, audit keamanan, maupun proses hukum — mensyaratkan bahwa setiap artefak digital yang dikumpulkan harus dapat dibuktikan tidak mengalami perubahan sejak saat pengumpulannya. Tanpa mekanisme ini, tidak ada jaminan bahwa bukti yang diserahkan kepada tim manajemen, auditor, atau penegak hukum identik dengan apa yang ditemukan di lapangan.

Proyek ini membangun dan mempraktikkan alur kerja pengelolaan bukti (*evidence handling*) yang terstruktur: dari penataan workspace hingga pembuatan arsip terkompresi dan verifikasi integritas menggunakan SHA256 checksum — sebuah praktik standar yang digunakan dalam investigasi forensik digital profesional.

## Tujuan

- Menata workspace menjadi area kerja yang terbagi berdasarkan status dan fungsi evidence
- Membuat inventaris file dan struktur direktori yang dapat diverifikasi
- Mengarsipkan seluruh bukti kerja awal ke dalam satu file terkompresi
- Menghasilkan SHA256 checksum sebagai bukti integritas kriptografis dari arsip tersebut
- Membangun kebiasaan kerja yang mencerminkan standar *chain of custody* dalam forensik digital

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Seluruh aktivitas evidence handling dilakukan di sini sebagai *primary workstation* |

## Konsep Dasar: Chain of Custody

*Chain of custody* adalah proses dokumentasi yang memastikan bahwa bukti digital dapat dilacak perjalanannya — dari pengumpulan pertama hingga presentasi akhir — tanpa celah yang memungkinkan pihak lain mempertanyakan keasliannya. Komponen utamanya meliputi:

1. **Siapa** yang mengumpulkan bukti (dicatat melalui `$USER` dan `date`)
2. **Kapan** bukti dikumpulkan (timestamp pada file)
3. **Apa** yang dikumpulkan (inventaris file)
4. **Bukti bahwa bukti tidak berubah** (SHA256 checksum)

## Langkah-Langkah yang Dilakukan

### 1. Penataan Struktur Workspace

```bash
cd ~/module01
mkdir -p evidence/raw evidence/processed reports/archive
```

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

### 4. Pengarsipan Evidence ke File Terkompresi

```bash
tar -czf ~/module01/reports/module01_workspace_initial.tar.gz \
    ~/module01/notes \
    ~/module01/evidence
```

| Flag | Fungsi |
|------|--------|
| `-c` | *Create* — buat arsip baru |
| `-z` | Kompres menggunakan gzip |
| `-f` | Tentukan nama file output |

### 5. Pembuatan SHA256 Checksum

```bash
sha256sum ~/module01/reports/module01_workspace_initial.tar.gz | tee ~/module01/evidence/archive_sha256.txt
ls -lh ~/module01/reports/module01_workspace_initial.tar.gz
```

SHA256 (*Secure Hash Algorithm 256-bit*) menghasilkan string 64 karakter heksadesimal yang unik untuk setiap file. Sifat kriptografis SHA256 memastikan bahwa:

- Perubahan sekecil apapun pada isi file akan menghasilkan hash yang sepenuhnya berbeda
- Tidak ada dua file berbeda yang dapat menghasilkan hash yang sama (*collision resistance*)
- Hash tidak dapat dibalik untuk mendapatkan isi file asli (*one-way function*)

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

**Verifikasi integritas:**
```bash
sha256sum -c archive_sha256.txt
# Output: module01_workspace_initial.tar.gz: OK
```

## Temuan Utama

Proses ini mengungkap betapa mudahnya sebuah file dapat dimodifikasi tanpa jejak yang terlihat secara kasat mata. Tanpa checksum, tidak ada cara untuk membedakan antara file yang asli dan file yang telah diubah — ukuran file bisa saja sama, nama file sama, bahkan tanggal modifikasi bisa dimanipulasi. SHA256 adalah satu-satunya mekanisme sederhana yang memberikan jaminan kriptografis bahwa isi file tidak berubah.

## Apa yang Dipelajari

Pengelolaan bukti yang profesional bukan hanya soal "menyimpan file". Ini adalah proses yang terstruktur dengan tujuan yang jelas: memastikan bahwa setiap artefak dapat diverifikasi, dilacak, dan dipertanggungjawabkan. Kebiasaan membuat checksum setelah setiap pengumpulan evidence adalah investasi waktu yang sangat kecil namun dapat mencegah masalah besar ketika integritas bukti dipertanyakan.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `workspace_tree.txt` | `~/module01/evidence/` | Struktur direktori workspace |
| `notes_inventory.txt` | `~/module01/evidence/` | Inventaris file catatan |
| `archive_sha256.txt` | `~/module01/evidence/` | SHA256 checksum dari arsip evidence |
| `module01_workspace_initial.tar.gz` | `~/module01/reports/` | Arsip terkompresi seluruh evidence awal |

---
---

# Proyek 4 — Deteksi Pola Mencurigakan pada Log Aktivitas Sistem dengan Shell dan Python

## Latar Belakang

Log sistem adalah sumber informasi paling kaya dalam keamanan siber. Hampir setiap serangan siber — mulai dari *brute force login*, *lateral movement*, hingga *data exfiltration* — meninggalkan jejak di log. Kemampuan untuk membaca log secara efisien, menyaring sinyal yang relevan dari kebisingan (*noise*), dan mengidentifikasi pola yang mencurigakan adalah kompetensi inti seorang *security analyst*.

Proyek ini membangun fondasi analisis log menggunakan dua pendekatan yang saling melengkapi: shell scripting dengan `grep` dan `awk` untuk filtering cepat dan otomasi keputusan berbasis kondisi, serta Python untuk analisis yang lebih terstruktur dan dapat dikembangkan.

## Tujuan

- Membuat dataset log simulasi yang mencerminkan aktivitas sistem nyata
- Memfilter event berdasarkan tingkat keparahan dan jenis aktivitas menggunakan shell pipeline
- Membangun logika keputusan otomatis berbasis ambang batas dengan Bash scripting
- Mengembangkan parser log terstruktur dengan Python yang menghasilkan ringkasan analitis
- Memahami kapan menggunakan shell dan kapan menggunakan Python dalam konteks analisis log

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Seluruh aktivitas parsing dan scripting dilakukan di sini |

## Dataset Log

```bash
cat > ~/module01/tmp/activity.log <<'EOF'
INFO login user=alice src=10.10.10.5
WARN failed_login user=bob src=10.10.10.7
WARN failed_login user=bob src=10.10.10.7
INFO file_upload user=alice path=/uploads/q1.xlsx
WARN failed_login user=charlie src=10.10.10.9
EOF
```

| Kolom | Contoh | Keterangan |
|-------|--------|------------|
| Level | `INFO`, `WARN` | Tingkat keparahan event |
| Event type | `login`, `failed_login`, `file_upload` | Jenis aktivitas yang terjadi |
| User | `user=alice` | Identitas pengguna yang terlibat |
| Source/Path | `src=10.10.10.5` | Alamat IP sumber atau path file |

## Langkah-Langkah yang Dilakukan

### 1. Filtering Event dengan `grep`

```bash
grep "failed_login" ~/module01/tmp/activity.log | tee ~/module01/evidence/activity_filtering.txt
```

**Output:**
```
WARN failed_login user=bob src=10.10.10.7
WARN failed_login user=bob src=10.10.10.7
WARN failed_login user=charlie src=10.10.10.9
```

### 2. Agregasi dan Penghitungan dengan `awk`

```bash
awk '{print $3,$4}' ~/module01/tmp/activity.log | sort | uniq -c | tee -a ~/module01/evidence/activity_filtering.txt
```

**Output:**
```
2 user=bob src=10.10.10.7
1 user=alice src=10.10.10.5
1 user=alice path=/uploads/q1.xlsx
1 user=charlie src=10.10.10.9
```

Kolom angka di kiri menunjukkan frekuensi kemunculan — `2` untuk kombinasi `user=bob src=10.10.10.7` langsung menunjukkan adanya percobaan berulang dari entitas yang sama.

### 3. Logika Deteksi Otomatis dengan Bash

```bash
cat > ~/module01/tmp/check_failed_logins.sh <<'EOF'
#!/usr/bin/env bash
LOG_FILE=~/module01/tmp/activity.log
THRESHOLD=2
COUNT=$(grep -c "failed_login" "$LOG_FILE")

echo "=== Laporan Deteksi Failed Login ==="
echo "File log     : $LOG_FILE"
echo "Total event  : $COUNT"
echo "Ambang batas : $THRESHOLD"
echo ""

if [ "$COUNT" -ge "$THRESHOLD" ]; then
    echo "[PERINGATAN] Review diperlukan: terdeteksi event failed_login berulang"
    grep "failed_login" "$LOG_FILE"
else
    echo "[AMAN] Tidak ada pola failed_login yang mencurigakan"
fi
EOF

chmod +x ~/module01/tmp/check_failed_logins.sh
~/module01/tmp/check_failed_logins.sh | tee ~/module01/evidence/shell_logic_output.txt
```

**Output:**
```
=== Laporan Deteksi Failed Login ===
File log     : /home/student/module01/tmp/activity.log
Total event  : 3
Ambang batas : 2

[PERINGATAN] Review diperlukan: terdeteksi event failed_login berulang
WARN failed_login user=bob src=10.10.10.7
WARN failed_login user=bob src=10.10.10.7
WARN failed_login user=charlie src=10.10.10.9
```

### 4. Parser Log Terstruktur dengan Python

```python
# activity_parser.py
from collections import Counter
from pathlib import Path

log = Path.home() / 'module01' / 'tmp' / 'activity.log'
users = Counter()
events = Counter()
failed_entries = []
failed = 0

for line in log.read_text().splitlines():
    parts = line.split()
    if parts:
        events[parts[0]] += 1
    for part in parts:
        if part.startswith('user='):
            users[part.split('=', 1)[1]] += 1
    if 'failed_login' in line:
        failed += 1
        failed_entries.append(line)

print('=== Ringkasan Analisis Log ===')
print(f'failed_login_count : {failed}')
print(f'user_activity      : {dict(users)}')
print(f'event_level_summary: {dict(events)}')

if failed >= 2:
    print('\n[PERINGATAN] Percobaan login gagal berulang terdeteksi:')
    for entry in failed_entries:
        print(f'  {entry}')
```

```bash
python ~/module01/tmp/activity_parser.py | tee ~/module01/evidence/python_logic_output.txt
```

**Output:**
```
=== Ringkasan Analisis Log ===
failed_login_count : 3
user_activity      : {'alice': 2, 'bob': 2, 'charlie': 1}
event_level_summary: {'INFO': 2, 'WARN': 3}

[PERINGATAN] Percobaan login gagal berulang terdeteksi:
  WARN failed_login user=bob src=10.10.10.7
  WARN failed_login user=bob src=10.10.10.7
  WARN failed_login user=charlie src=10.10.10.9
```

## Perbandingan Shell vs Python

| Aspek | Shell (`grep`, `awk`, `bash`) | Python |
|-------|-------------------------------|--------|
| Kecepatan setup | Sangat cepat, langsung dijalankan | Perlu menulis script terlebih dahulu |
| Keterbacaan output | Terbatas, tergantung formatting manual | Lebih terstruktur dan mudah dikustomisasi |
| Kemampuan pengembangan | Terbatas untuk logika kompleks | Sangat fleksibel, dapat dikembangkan menjadi modul |
| Penggunaan ideal | Filtering cepat, one-liner, otomasi sederhana | Analisis terstruktur, laporan, integrasi data |

## Temuan Utama

Dari analisis log, ditemukan bahwa **user `bob` melakukan dua percobaan login yang gagal dari IP yang sama (`10.10.10.7`) dalam satu sesi**. Ini adalah pola klasik *brute-force* atau *credential stuffing*. Dalam lingkungan produksi, pola ini seharusnya memicu alert otomatis dan potensi blokir sementara pada IP sumber. User `charlie` juga mencatat satu event `failed_login` yang perlu dipantau sebagai anomali.

## Apa yang Dipelajari

Shell dan Python bukan alat yang bersaing — keduanya memiliki tempat yang berbeda dalam alur kerja analis keamanan. Pola berpikir yang dibangun di sini — ekstrak, hitung, bandingkan dengan ambang batas, keluarkan keputusan — adalah fondasi dari hampir semua sistem deteksi ancaman, mulai dari skrip sederhana hingga platform SIEM (*Security Information and Event Management*) yang kompleks.

## Referensi File Bukti dan Script

| File | Lokasi | Isi |
|------|--------|-----|
| `activity.log` | `~/module01/tmp/` | Dataset log simulasi |
| `activity_filtering.txt` | `~/module01/evidence/` | Output grep dan awk filtering |
| `shell_logic_output.txt` | `~/module01/evidence/` | Output script deteksi Bash |
| `python_logic_output.txt` | `~/module01/evidence/` | Output parser Python |
| `check_failed_logins.sh` | `~/module01/tmp/` | Script deteksi otomatis Bash |
| `activity_parser.py` | `~/module01/tmp/` | Parser log Python |

---
---

# Proyek 5 — Audit Izin File dan Tinjauan Metadata Sistem pada Lingkungan Linux

## Latar Belakang

Kesalahan konfigurasi izin file (*file permission misconfiguration*) adalah salah satu temuan paling umum dalam audit keamanan sistem Linux. File konfigurasi yang dapat dibaca oleh semua pengguna, skrip yang dapat dieksekusi oleh siapapun, atau direktori yang memberikan akses tulis kepada grup yang tidak seharusnya — semuanya adalah vektor yang dapat dieksploitasi untuk eskalasi privilase (*privilege escalation*) atau pencurian data.

## Tujuan

- Mengkonfigurasi izin file menggunakan notasi oktal dan memahami implikasi keamanannya
- Menggunakan `ls -l`, `stat`, dan `namei` untuk membaca metadata dan izin secara komprehensif
- Memahami perbedaan antara izin file dan izin direktori dalam konteks akses efektif
- Menemukan file yang berpotensi relevan untuk analisis keamanan menggunakan `find`
- Mendokumentasikan temuan dalam format yang siap digunakan dalam laporan audit

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Seluruh aktivitas audit izin dan metadata dilakukan di sini |

## Konsep Dasar: Sistem Izin Linux

### Tiga Kelas Pengguna

| Kelas | Simbol | Keterangan |
|-------|--------|------------|
| Owner | `u` | Pemilik file — biasanya user yang membuatnya |
| Group | `g` | Grup yang diasosiasikan dengan file |
| Others | `o` | Semua pengguna lain di sistem |

### Notasi Oktal

| Oktal | Biner | Izin | Keterangan |
|-------|-------|------|------------|
| `7` | `111` | `rwx` | Akses penuh |
| `6` | `110` | `rw-` | Baca dan tulis |
| `5` | `101` | `r-x` | Baca dan eksekusi |
| `4` | `100` | `r--` | Hanya baca |
| `0` | `000` | `---` | Tidak ada akses |

## Langkah-Langkah yang Dilakukan

### 1. Navigasi dan Pembacaan Struktur Direktori

```bash
printf "=== HOME ===\n" | tee ~/module01/evidence/filesystem_navigation.txt
pwd | tee -a ~/module01/evidence/filesystem_navigation.txt
ls -la ~ | tee -a ~/module01/evidence/filesystem_navigation.txt

printf "\n=== DATASETS ===\n" | tee -a ~/module01/evidence/filesystem_navigation.txt
find ~/datasets -maxdepth 2 -type d | sort | tee -a ~/module01/evidence/filesystem_navigation.txt
```

### 2. Konfigurasi Izin File

```bash
mkdir -p ~/module01/filesystem/labdata
cd ~/module01/filesystem/labdata
touch note.txt report.txt script.sh

chmod 600 note.txt    # Catatan sensitif — hanya owner
chmod 640 report.txt  # Laporan — owner bisa edit, grup hanya baca
chmod 750 script.sh   # Script — owner penuh, grup bisa jalankan
```

**Analisis izin tiap file:**

| File | Mode | Simbol | Penjelasan | Kasus Penggunaan |
|------|------|--------|------------|-----------------|
| `note.txt` | `600` | `rw-------` | Hanya owner bisa baca dan tulis | File berisi password, private key |
| `report.txt` | `640` | `rw-r-----` | Owner baca/tulis, grup hanya baca | Laporan yang dibagi dalam tim |
| `script.sh` | `750` | `rwxr-x---` | Owner penuh, grup bisa baca dan eksekusi | Skrip otomasi tim operasi |

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

| Field | Keterangan |
|-------|------------|
| `Inode` | Nomor inode — identitas unik file di filesystem |
| `Links` | Jumlah *hard link* yang menunjuk ke inode ini |
| `Access` | Waktu terakhir file dibaca |
| `Modify` | Waktu terakhir isi file diubah |
| `Change` | Waktu terakhir metadata file diubah |

### 5. Audit Path Lengkap dengan `namei`

```bash
namei -l ~/module01/filesystem/labdata | tee -a ~/module01/evidence/metadata_review.txt
```

`namei -l` menelusuri setiap komponen dari path yang diberikan dan menampilkan izin di setiap level. Ini sangat penting karena **izin efektif sebuah file ditentukan oleh kombinasi izin file DAN semua direktori dalam pathnya**.

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

### 6. Pencarian File Relevan untuk Analisis Keamanan

```bash
find ~/datasets ~/samples -maxdepth 3 -type f \
  \( -iname "*.log" -o -iname "*.json" -o -iname "*.pcap" \
     -o -iname "*.csv"  -o -iname "*.txt" \) | sort | \
  tee ~/module01/evidence/risky_strings_findings.txt
```

## Skenario Risiko yang Ditemukan

| Skenario | Konfigurasi Bermasalah | Dampak Potensial |
|----------|----------------------|-----------------|
| File konfigurasi terlalu terbuka | `note.txt` dengan mode `644` alih-alih `600` | Seluruh pengguna di sistem dapat membaca kredensial |
| Skrip dapat dimodifikasi grup | `script.sh` dengan mode `770` alih-alih `750` | Injeksi kode berbahaya oleh anggota grup |
| Direktori dengan izin berlebihan | `labdata` dengan mode `777` | Siapapun dapat membuat atau menghapus file di dalamnya |

## Apa yang Dipelajari

Membaca izin file di Linux membutuhkan pemahaman yang lebih dari sekadar angka. Izin efektif sebuah file adalah hasil kombinasi dari izin file itu sendiri dan izin seluruh direktori dalam pathnya. Kemampuan audit ini secara langsung relevan dalam: *penetration testing* (mencari file yang salah konfigurasi), *incident response* (memverifikasi file sistem penting), dan *security hardening* (memastikan konfigurasi server memenuhi standar keamanan minimum).

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `filesystem_navigation.txt` | `~/module01/evidence/` | Output navigasi direktori home, datasets, dan starter |
| `permission_review.txt` | `~/module01/evidence/` | Output `ls -l` dengan detail izin tiga file uji |
| `metadata_review.txt` | `~/module01/evidence/` | Output `stat` dan `namei` untuk metadata dan path audit |
| `risky_strings_findings.txt` | `~/module01/evidence/` | Daftar file yang ditemukan melalui pencarian berbasis ekstensi |

---
---

# Proyek 6 — Identifikasi Indikator Kompromi pada Sampel Artefak dan Data Lab

## Latar Belakang

*Security awareness* yang sejati bukan tentang menghafal daftar ancaman atau mengikuti pelatihan tahunan. Ini tentang kemampuan untuk melihat artefak nyata — sebuah email, sebuah file konfigurasi, sebuah kumpulan data — dan secara kritis mengidentifikasi elemen-elemen yang menunjukkan adanya risiko, manipulasi, atau eksposur yang tidak semestinya.

Proyek ini mendekati keamanan dari sudut pandang praktis: menganalisis sampel email phishing menggunakan teknik pembacaan kritis yang sama yang digunakan oleh analis *threat intelligence*, melakukan pemindaian string berisiko pada dataset untuk mendeteksi potensi eksposur kredensial, dan mendokumentasikan prinsip pengelolaan kata sandi yang operasional.

## Tujuan

- Menganalisis sampel email phishing dan mengidentifikasi teknik rekayasa sosial (*social engineering*) yang digunakan
- Melakukan pemindaian berbasis regex pada dataset untuk menemukan string yang berpotensi mengekspos kredensial
- Mendokumentasikan prinsip pengelolaan kata sandi yang profesional dan operasional
- Membangun kebiasaan berpikir kritis saat berhadapan dengan komunikasi dan artefak digital

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Seluruh analisis artefak dan pemindaian dilakukan di sini |

## Langkah-Langkah yang Dilakukan

### 1. Penemuan dan Tinjauan Sampel Email Phishing

```bash
find ~/datasets -type f | grep -i 'phish\|email'
cat ~/datasets/phishing/sample_email_1.txt | tee ~/module01/evidence/phishing_review.txt
```

**Framework Analisis Email Phishing:**

#### A. Analisis Pengirim

| Elemen yang Diperiksa | Tanda Bahaya (*Red Flag*) |
|-----------------------|--------------------------|
| Alamat email pengirim | Domain yang menyerupai domain legitimate (contoh: `paypa1.com` vs `paypal.com`) |
| Nama tampilan | Nama familiar yang tidak cocok dengan alamat email sebenarnya |
| Header email | Ketidakcocokan antara `From:`, `Reply-To:`, dan `Return-Path:` |

#### B. Analisis Konten

| Teknik Manipulasi | Contoh | Mekanisme Psikologis |
|-------------------|--------|----------------------|
| **Urgensi palsu** | "Akun Anda akan ditutup dalam 24 jam" | Kepanikan mendorong tindakan impulsif |
| **Otoritas yang dipalsukan** | "Tim Keamanan Bank XYZ" | Kecenderungan patuh pada figur otoritas |
| **Permintaan informasi sensitif** | "Konfirmasi password Anda untuk verifikasi" | Organisasi legitimate tidak pernah meminta ini |
| **Ancaman konsekuensi** | "Jika tidak dikonfirmasi, akun akan diblokir" | Rasa takut mendorong kepatuhan tanpa berpikir |
| **Penawaran menggiurkan** | "Anda memenangkan hadiah senilai Rp 10 juta" | Harapan menurunkan kewaspadaan |

#### C. Indikator Teknis Phishing

```
✗ Domain pengirim tidak cocok dengan organisasi yang diklaim
✗ Bahasa yang menciptakan urgensi tidak wajar
✗ Permintaan eksplisit untuk memasukkan kredensial
✗ Link yang mengarah ke domain yang tidak relevan
✗ Attachment yang tidak diminta
✗ Sapaan generik ("Pelanggan yang terhormat") alih-alih nama penerima
```

### 2. Dokumentasi Prinsip Pengelolaan Kata Sandi

```bash
cat > ~/module01/evidence/password_strength_notes.txt <<'EOF'
=== Prinsip Pengelolaan Kata Sandi ===

1. KEUNIKAN
   Gunakan kata sandi yang berbeda untuk setiap sistem dan layanan.

2. PANJANG DAN KOMPLEKSITAS
   Panjang kata sandi (>16 karakter) lebih efektif daripada variasi simbol yang dangkal.
   Frasa sandi (passphrase) yang panjang lebih kuat dan lebih mudah diingat.

3. TIDAK ADA POLA YANG DAPAT DITEBAK
   Hindari informasi pribadi: tanggal lahir, nama, kota asal.

4. PENGELOLAAN KREDENSIAL
   Gunakan password manager. Jangan tulis kata sandi di dokumen tidak terenkripsi.

5. ROTASI KREDENSIAL
   Ganti kata sandi segera jika ada indikasi kompromi.

6. AUTENTIKASI MULTI-FAKTOR (MFA)
   Aktifkan MFA pada seluruh akun yang mendukungnya.

7. KREDENSIAL LAB
   Kata sandi lab tidak boleh digunakan di sistem lain.
EOF
```

### 3. Pemindaian String Berisiko pada Dataset

```bash
grep -RinE 'password|secret|token|urgent|verify|login|credential|api_key|passwd' \
    ~/datasets ~/samples 2>/dev/null | \
    tee -a ~/module01/evidence/risky_strings_findings.txt | \
    head -n 50
```

| Flag | Fungsi |
|------|--------|
| `-R` | Rekursif — telusuri seluruh subdirektori |
| `-i` | *Case-insensitive* — cocokkan `PASSWORD`, `password`, `Password` sekaligus |
| `-n` | Tampilkan nomor baris tempat kecocokan ditemukan |
| `-E` | Gunakan *Extended Regular Expression* |

**Kata kunci yang dicari dan relevansinya:**

| Kata Kunci | Relevansi Keamanan |
|------------|-------------------|
| `password`, `passwd` | Potensi kredensial tersimpan dalam plaintext |
| `secret` | Secret key, API secret, atau token rahasia |
| `token` | API token, JWT, atau session token yang mungkin aktif |
| `api_key` | Kunci API yang dapat digunakan untuk mengakses layanan eksternal |
| `urgent`, `verify` | Bahasa phishing yang menciptakan urgensi |
| `credential` | Referensi eksplisit ke informasi autentikasi |

## Temuan Utama

**Dari analisis phishing:** Email yang dianalisis menggunakan **tiga teknik manipulasi sekaligus** dalam satu pesan — urgensi waktu, otoritas yang dipalsukan, dan permintaan kredensial eksplisit. Kombinasi ini adalah pola *triple-threat* yang paling umum ditemukan dalam kampanye phishing skala besar.

**Dari pemindaian string berisiko:** Ditemukan beberapa file di dalam dataset yang mengandung string `password=` dan `token=` dalam format plaintext. Dalam skenario nyata, temuan seperti ini diklasifikasikan sebagai **Severity: High** dan memerlukan tindakan segera: enkripsi file, rotasi kredensial, dan audit akses log.

## Apa yang Dipelajari

*Security awareness* yang efektif adalah kemampuan menerjemahkan pengetahuan tentang ancaman menjadi tindakan analitis yang konkret. Pemindaian string berbasis regex yang dipraktikkan di sini adalah teknik yang sama yang digunakan dalam *incident response* untuk mencari kredensial yang terekspos, *Data Loss Prevention (DLP)* untuk mendeteksi informasi sensitif, dan *code review* untuk menemukan hardcoded credentials dalam repositori kode sumber.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `phishing_review.txt` | `~/module01/evidence/` | Salinan dan analisis sampel email phishing |
| `password_strength_notes.txt` | `~/module01/evidence/` | Dokumentasi prinsip pengelolaan kata sandi |
| `risky_strings_findings.txt` | `~/module01/evidence/` | Hasil pemindaian string berisiko pada dataset |
