## Tentang Portofolio Ini

Portofolio ini mendokumentasikan serangkaian proyek analisis sistem operasi (*host analysis*) yang dikerjakan secara langsung (*hands-on*) di lingkungan lab terisolasi. Setiap proyek mencerminkan cara kerja seorang *junior security analyst*, *SOC analyst*, atau *security operations engineer* profesional — mulai dari pembacaan konteks host, analisis metadata dan permission file, observasi proses dan session, review log dan packet capture, hingga pembangunan automation berbasis Bash dan Python.

Seluruh aktivitas dilakukan pada dua host lab dengan peran yang berbeda:

| Host | Peran |
|------|-------|
| **Ubuntu Lab** | *Primary analysis host* — file system review, metadata, process correlation, log parsing, YARA, dan automation |
| **Kali Linux** | *Validation workstation* — service checking, web target validation, dan perspektif assessment |
| **Local Targets** | *Safe practice target* — validasi service web lokal dan korelasi dengan host state |
| **Datasets `/opt/jccsah`** | Log sample, PCAP sample, phishing sample, dan forensics artifacts |

> ⚠️ **Catatan:** Seluruh aktivitas dilakukan **hanya** pada lingkungan lab yang telah disediakan. Tidak ada privilege escalation, perubahan service global, atau eksperimen destructive yang dilakukan di luar scope yang ditetapkan.

---

## Daftar Proyek

| No. | Judul Proyek | Fokus Area | Tools Utama |
|-----|-------------|------------|-------------|
| 1 | [Membangun Workspace dan Memverifikasi Kesiapan Tool Lab](#proyek-1--membangun-workspace-dan-memverifikasi-kesiapan-tool-lab) | Workspace setup, tool readiness | `mkdir`, `source`, `ls`, helper lab |
| 2 | [Mengumpulkan Baseline Host, User Context, dan Resource State](#proyek-2--mengumpulkan-baseline-host-user-context-dan-resource-state) | Host baseline, resource monitoring | `whoami`, `id`, `df`, `free`, `ps` |
| 3 | [Menganalisis File System, Metadata, Ownership, dan Permission](#proyek-3--menganalisis-file-system-metadata-ownership-dan-permission) | File system analysis, permission audit | `ls`, `namei`, `stat`, `chmod`, `find` |
| 4 | [Mengobservasi Proses, Session Aktif, dan Relasi Port–Service](#proyek-4--mengobservasi-proses-session-aktif-dan-relasi-portservice) | Process analysis, session review | `ps`, `pstree`, `w`, `who`, `last`, `ss`, `nmap` |
| 5 | [Menganalisis Log Sistem, Auth Log, PCAP, dan Menjalankan YARA](#proyek-5--menganalisis-log-sistem-auth-log-pcap-dan-menjalankan-yara) | Log analysis, YARA rule-based detection | `journalctl`, `grep`, `tshark`, `yara` |
| 6 | [Membangun Automation Parsing Evidence dengan Bash dan Python](#proyek-6--membangun-automation-parsing-evidence-dengan-bash-dan-python) | Scripting, automation, log parsing | `bash`, `python`, `regex`, `Counter` |
| 7 | [Menyusun OS Security Assessment dan Jembatan ke Modul Berikutnya](#proyek-7--menyusun-os-security-assessment-dan-jembatan-ke-modul-berikutnya) | Security assessment, reporting | Dokumentasi profesional |

---

## Teknologi dan Tools

<p>
  <img src="https://img.shields.io/badge/OS-Ubuntu%20Linux-E95420?style=flat-square&logo=ubuntu&logoColor=white"/>
  <img src="https://img.shields.io/badge/OS-Kali%20Linux-557C94?style=flat-square&logo=kalilinux&logoColor=white"/>
  <img src="https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat-square&logo=gnubash&logoColor=white"/>
  <img src="https://img.shields.io/badge/Python-3.x-3776AB?style=flat-square&logo=python&logoColor=white"/>
  <img src="https://img.shields.io/badge/Tool-YARA-red?style=flat-square"/>
  <img src="https://img.shields.io/badge/Tool-tshark-1679A7?style=flat-square"/>
  <img src="https://img.shields.io/badge/Tool-Nmap-red?style=flat-square"/>
  <img src="https://img.shields.io/badge/Tool-pstree%20%2F%20ps-informational?style=flat-square"/>
</p>

---

## Kompetensi yang Dibangun

- **Host Baseline Analysis** — Membaca identitas user, konteks grup, resource state disk dan memori, serta snapshot proses secara terstruktur
- **File System & Permission Audit** — Menganalisis metadata file, ownership, mode permission, dan path permission secara menyeluruh menggunakan `stat`, `namei`, dan `find`
- **Process & Session Observation** — Memahami hierarki proses, relasi parent-child, review session aktif, dan menghubungkan proses dengan port yang sedang listen
- **Log Analysis & Monitoring** — Membaca journal sistem, menganalisis pola autentikasi dari auth log, meninjau traffic dari PCAP sample, dan menerapkan deteksi berbasis rule YARA
- **Scripting & Automation** — Membangun parser log dengan Bash dan Python yang menghasilkan output terstruktur dan dapat direproduksi
- **Security Assessment Writing** — Menyusun OS security assessment yang menghubungkan seluruh observasi teknis menjadi penilaian yang siap dibaca tim teknis

---
---

# Proyek 1 — Membangun Workspace dan Memverifikasi Kesiapan Tool Lab

## Latar Belakang

Dalam operasi keamanan siber, kesiapan lingkungan kerja adalah prasyarat — bukan langkah opsional. Analis yang mulai bekerja tanpa memverifikasi bahwa seluruh tool, dataset, dan struktur workspace tersedia berisiko menemukan masalah di tengah investigasi, ketika waktu paling berharga. Verifikasi kesiapan di awal sesi adalah standar profesional yang memisahkan analis yang terorganisir dari analis yang reaktif.

Proyek ini menetapkan struktur workspace yang lebih kompleks dibandingkan modul sebelumnya — mencerminkan bertambahnya domain analisis — dan memverifikasi bahwa seluruh tool serta dataset yang dibutuhkan tersedia sebelum analisis dimulai.

## Tujuan

- Membangun struktur direktori workspace yang terbagi berdasarkan domain analisis
- Mengaktifkan Python environment yang disiapkan oleh administrator lab
- Memverifikasi ketersediaan tool analisis kritis: `python3`, `yara`, `tshark`, `nmap`
- Memastikan symlink ke datasets, samples, wordlists, dan starter tersedia dan dapat diakses

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Seluruh setup workspace dan verifikasi tool dilakukan di sini |

## Langkah-Langkah yang Dilakukan

### 1. Pembangunan Struktur Workspace Modul 03

```bash
mkdir -p ~/mod3/{evidence,reports,notes,baseline,filesystem,processes,monitoring,automation}
cd ~/mod3
pwd
ls -la
```

**Fungsi tiap direktori:**

| Direktori | Fungsi |
|-----------|--------|
| `evidence/baseline/` | Identitas host, resource state, dan context awal |
| `evidence/filesystem/` | Output analisis file system, metadata, dan permission |
| `evidence/processes/` | Output review proses, session, port, dan service |
| `evidence/monitoring/` | Journal, auth log, PCAP summary, dan YARA result |
| `evidence/automation/` | Output Bash parser, Python parser, dan automation notes |
| `reports/` | OS security assessment dan operating notes final |
| `notes/` | Catatan kerja dan observasi selama sesi |

Pemisahan berdasarkan domain teknis — bukan hanya satu folder "evidence" — memastikan bahwa setiap temuan dapat ditelusuri berdasarkan konteksnya. Dalam investigasi yang panjang, struktur ini mencegah kontaminasi silang antara artefak dari sumber yang berbeda.

### 2. Aktivasi Python Environment dan Verifikasi Tool

```bash
source ~/activate-pylab
~/jccsah-env-info
ls -la ~/datasets ~/samples ~/wordlists ~/starter
```

**Mengapa Python environment diaktifkan terlebih dahulu:**

Lingkungan lab menggunakan Python virtual environment yang dikelola oleh administrator untuk memastikan konsistensi versi library di seluruh workstation. Mengaktifkan environment ini sebelum bekerja memastikan bahwa seluruh library yang dibutuhkan (termasuk library untuk parsing log dan analisis data) tersedia tanpa perlu instalasi manual yang dapat menyebabkan konflik dependensi.

**Tool yang diverifikasi ketersediaannya:**

| Tool | Fungsi dalam Modul Ini |
|------|------------------------|
| `python3` | Parser log terstruktur dan analisis data |
| `yara` | Rule-based scanning pada artefak teks dan file |
| `tshark` | Analisis PCAP sample dari command line |
| `nmap` | Validasi service dari perspektif assessment |
| Helper lab (`~/jccsah-env-info`) | Informasi environment dan tool yang tersedia |

**Verifikasi dataset yang tersedia:**

```bash
ls -la ~/datasets ~/samples ~/wordlists ~/starter
```

Perintah ini memverifikasi bahwa symlink ke resource lab telah dikonfigurasi dengan benar. Symlink yang rusak atau tidak ada akan segera terdeteksi di sini — jauh lebih baik daripada menemukan masalah ketika sedang menjalankan analisis di tengah sesi.

## Hasil dan Output

**Contoh struktur workspace yang terbentuk:**
```
/home/student/mod3
├── evidence/
│   ├── baseline/
│   ├── filesystem/
│   ├── processes/
│   ├── monitoring/
│   └── automation/
├── reports/
├── notes/
└── (direktori kerja lainnya)
```

**Contoh output `jccsah-env-info`:**
```
=== JCCSAH Environment Info ===
Python: 3.11.x (pylab active)
Tools available: python3, yara, tshark, nmap, dig, curl, nc
Datasets: ~/datasets → /opt/jccsah/datasets (symlink OK)
Samples:  ~/samples  → /opt/jccsah/samples  (symlink OK)
```

## Temuan Utama

Verifikasi awal mengkonfirmasi bahwa seluruh tool dan dataset tersedia. Lebih penting lagi, proses verifikasi ini membiasakan analis untuk tidak mengasumsikan bahwa environment selalu dalam keadaan siap — sebuah asumsi yang sangat berbahaya dalam operasi keamanan nyata di mana environment bisa saja telah dimodifikasi sejak sesi terakhir.

## Apa yang Dipelajari

Kesiapan environment adalah bagian dari kompetensi teknis, bukan tugas administratif yang dapat diabaikan. Analis yang memulai setiap sesi dengan verifikasi sistematis akan jauh lebih jarang menemukan kejutan di tengah investigasi — dan dalam konteks incident response, kejutan yang tidak perlu dapat menghabiskan waktu kritis yang seharusnya digunakan untuk analisis.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| Struktur direktori | `~/mod3/` | Workspace terorganisir siap digunakan |
| Output `jccsah-env-info` | Terminal | Konfirmasi tool dan symlink tersedia |

---
---

# Proyek 2 — Mengumpulkan Baseline Host, User Context, dan Resource State

## Latar Belakang

Gejala keamanan dan gejala performa sistem sering kali memiliki manifestasi yang sangat mirip: proses yang berjalan lambat, memori yang hampir habis, atau disk yang penuh bisa mengindikasikan serangan *cryptominer*, kebocoran data, atau sekadar aplikasi yang tidak dioptimalkan. Tanpa baseline yang dikumpulkan sebelum insiden terjadi, analis tidak memiliki referensi untuk membedakan kondisi normal dari kondisi anomali.

Proyek ini membangun baseline host yang komprehensif: identitas user dan grup, informasi kernel, kondisi storage, kondisi memori, dan snapshot proses — semua dikumpulkan di awal sesi sebagai titik referensi yang akan digunakan sepanjang analisis.

## Tujuan

- Merekam identitas lengkap user aktif termasuk UID, GID, dan keanggotaan grup
- Mengumpulkan informasi kernel dan arsitektur sistem sebagai konteks OS
- Membaca kondisi storage dan memori sebagai baseline resource state
- Membuat snapshot proses yang diurutkan berdasarkan konsumsi memori untuk referensi awal

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Seluruh pengumpulan baseline dilakukan di sini |

## Langkah-Langkah yang Dilakukan

### 1. Identitas Host dan User Context

```bash
whoami | tee ~/mod3/evidence/baseline/whoami.txt
id      | tee ~/mod3/evidence/baseline/id.txt
hostname | tee ~/mod3/evidence/baseline/hostname.txt
uname -a | tee ~/mod3/evidence/baseline/uname.txt
date    | tee ~/mod3/evidence/baseline/date.txt
```

**Informasi yang dikumpulkan dan relevansinya:**

| Perintah | Output | Relevansi Keamanan |
|----------|--------|--------------------|
| `whoami` | Nama user aktif | Konfirmasi identitas — mencegah kerja dengan akun yang salah |
| `id` | UID, GID, dan semua grup | Memahami hak akses efektif user |
| `hostname` | Nama host | Identifikasi host dalam laporan dan log multi-host |
| `uname -a` | Kernel version, arsitektur, build date | Menilai apakah kernel rentan terhadap *known exploits* |
| `date` | Timestamp sistem saat ini | Referensi waktu untuk seluruh evidence yang dikumpulkan |

**Mengapa output `id` lebih informatif dari `whoami`:**

`whoami` hanya menampilkan nama user aktif. `id` menampilkan informasi yang jauh lebih lengkap:

```
uid=1001(student) gid=1001(student) groups=1001(student),4(adm),24(cdrom),27(sudo),30(dip)
```

Keanggotaan grup seperti `adm` (akses ke log sistem) atau `docker` (potensi privilege escalation via container) adalah informasi yang krusial dalam security review namun tidak terlihat dari `whoami` saja.

### 2. Resource Baseline: Storage, Memori, dan Process Snapshot

```bash
df -h | tee ~/mod3/evidence/baseline/disk_usage.txt
free -h | tee ~/mod3/evidence/baseline/memory_usage.txt
ps -eo pid,ppid,user,%cpu,%mem,tty,stat,start,time,command --sort=-%mem | head -n 25 | tee ~/mod3/evidence/baseline/mem_process_snapshot.txt
```

**Penjelasan `df -h`:**

`df` (*disk free*) menampilkan penggunaan storage pada setiap mount point. Flag `-h` menampilkan ukuran dalam format yang mudah dibaca (KB, MB, GB).

| Kolom | Keterangan |
|-------|------------|
| `Filesystem` | Device atau filesystem yang di-mount |
| `Size` | Total kapasitas |
| `Used` | Kapasitas yang sudah terpakai |
| `Avail` | Kapasitas yang tersisa |
| `Use%` | Persentase penggunaan |
| `Mounted on` | Lokasi mount point |

Mount point yang mendekati 100% perlu diperhatikan: disk penuh dapat menyebabkan log tidak dapat ditulis, yang merupakan kondisi yang dimanfaatkan dalam beberapa teknik serangan (*log flooding*).

**Penjelasan `free -h`:**

`free` menampilkan kondisi memori fisik (RAM) dan swap.

```
               total        used        free      shared  buff/cache   available
Mem:           7.7Gi       2.1Gi       3.2Gi       142Mi       2.4Gi       5.2Gi
Swap:          2.0Gi          0B       2.0Gi
```

Kolom `available` lebih relevan dari `free` dalam konteks keamanan — `available` menunjukkan memori yang benar-benar dapat dialokasikan untuk proses baru, termasuk yang sudah digunakan sebagai buffer/cache yang dapat dilepaskan.

**Penjelasan format `ps` yang digunakan:**

```bash
ps -eo pid,ppid,user,%cpu,%mem,tty,stat,start,time,command --sort=-%mem | head -n 25
```

| Flag/Kolom | Fungsi |
|------------|--------|
| `-e` | Tampilkan semua proses (bukan hanya milik user aktif) |
| `-o` | Tentukan kolom yang ditampilkan secara kustom |
| `pid` | Process ID |
| `ppid` | Parent Process ID — kunci untuk analisis hierarki proses |
| `user` | Pemilik proses |
| `%cpu`, `%mem` | Persentase penggunaan CPU dan memori |
| `tty` | Terminal yang diasosiasikan (jika ada) |
| `stat` | Status proses: R (running), S (sleeping), Z (zombie), T (stopped) |
| `start` | Waktu proses dimulai |
| `--sort=-%mem` | Urutkan berdasarkan penggunaan memori tertinggi (descending) |

## Hasil dan Output

**Contoh output `id.txt`:**
```
uid=1001(student) gid=1001(student) groups=1001(student),4(adm),27(sudo),1002(docker)
```

**Contoh output `disk_usage.txt`:**
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        50G   12G   36G  25% /
tmpfs           3.9G  1.2M  3.9G   1% /dev/shm
/dev/sda2       100G   45G   50G  48% /opt
```

**Contoh output `mem_process_snapshot.txt`:**
```
PID   PPID USER    %CPU %MEM TT  STAT START   TIME COMMAND
1823  1     root    0.0  5.2  ?   Ss   09:00   0:12 /usr/sbin/dockerd
2341  1823  student 0.1  3.8  ?   Sl   09:05   0:03 node /app/server.js
3012  1     www-data 0.0 2.1  ?   S    09:00   0:01 nginx: worker process
```

## Temuan Utama

Output `id` mengungkap bahwa user `student` merupakan anggota grup `docker` — sebuah temuan yang signifikan dari perspektif keamanan. Keanggotaan grup `docker` secara efektif memberikan akses yang setara dengan root pada sistem host, karena container Docker dapat dikonfigurasi untuk me-mount filesystem host. Dalam lingkungan produksi, ini merupakan *privilege escalation vector* yang perlu didokumentasikan dan ditangani.

Snapshot proses menunjukkan bahwa proses `dockerd` mengonsumsi memori terbesar, diikuti oleh Node.js server — informasi yang membantu memahami beban kerja normal host sebelum anomali dapat diidentifikasi.

## Apa yang Dipelajari

Baseline yang komprehensif adalah investasi waktu yang sangat kecil dengan nilai yang sangat besar. Dalam investigasi insiden, analis yang memiliki baseline pra-insiden dapat dengan cepat membandingkan kondisi saat ini dengan kondisi normal dan mengidentifikasi anomali. Analis yang tidak memiliki baseline harus membangun referensi dari nol — proses yang jauh lebih lambat dan kurang akurat.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `whoami.txt` | `~/mod3/evidence/baseline/` | Nama user aktif |
| `id.txt` | `~/mod3/evidence/baseline/` | UID, GID, dan keanggotaan grup |
| `hostname.txt` | `~/mod3/evidence/baseline/` | Nama host |
| `uname.txt` | `~/mod3/evidence/baseline/` | Informasi kernel dan arsitektur |
| `date.txt` | `~/mod3/evidence/baseline/` | Timestamp referensi |
| `disk_usage.txt` | `~/mod3/evidence/baseline/` | Kondisi storage per mount point |
| `memory_usage.txt` | `~/mod3/evidence/baseline/` | Kondisi RAM dan swap |
| `mem_process_snapshot.txt` | `~/mod3/evidence/baseline/` | Top 25 proses berdasarkan memori |

---
---

# Proyek 3 — Menganalisis File System, Metadata, Ownership, dan Permission

## Latar Belakang

Sebagian besar insiden keamanan pada sistem Linux meninggalkan jejak di file system: file konfigurasi yang dimodifikasi, file executable yang ditambahkan di lokasi tidak lazim, permission yang diubah untuk memudahkan akses, atau file yang dibuat dengan timestamp yang mencurigakan. Kemampuan untuk membaca dan menginterpretasikan metadata file — bukan hanya nama dan isinya — adalah kompetensi fundamental yang membedakan analis yang hanya "melihat" file dari analis yang benar-benar "memahami" kondisi file system.

Proyek ini membangun kemampuan analisis file system yang komprehensif: dari pembacaan hidden files di home directory, interpretasi permission path secara menyeluruh dengan `namei`, review metadata detail dengan `stat`, hingga pencarian file berdasarkan pola yang relevan untuk investigasi keamanan.

## Tujuan

- Membaca isi home directory secara lengkap termasuk hidden files dan menginterpretasikan permission-nya
- Menggunakan `namei -l` untuk memahami permission pada setiap komponen path, bukan hanya file target
- Mengkonfigurasi dan mengaudit permission file dengan notasi oktal dan memahami implikasi keamanannya
- Menggunakan `stat` untuk mengekstrak metadata file yang tidak terlihat dari `ls` biasa
- Menemukan file yang berpotensi relevan untuk investigasi menggunakan `find` dengan filter yang tepat

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Seluruh analisis file system dilakukan di sini |

## Langkah-Langkah yang Dilakukan

### 1. Review Home Directory dan Hidden Files

```bash
echo $HOME
ls -la $HOME | tee ~/mod3/evidence/filesystem/home_listing.txt
namei -l $HOME | tee ~/mod3/evidence/filesystem/home_path_permissions.txt
```

**Mengapa hidden files penting dalam security review:**

File yang diawali dengan titik (`.`) tersembunyi dari `ls` biasa namun dapat mengandung informasi konfigurasi yang sangat sensitif. Beberapa contoh kritis:

| File/Direktori | Isi yang Potensial | Risiko Keamanan |
|----------------|-------------------|-----------------|
| `.ssh/` | Private key, `authorized_keys`, `known_hosts` | Akses SSH tanpa password jika dikompromikan |
| `.bash_history` | Riwayat command terminal | Mengekspos command sensitif yang pernah dijalankan |
| `.env` | Environment variables | Sering berisi API key, password, atau connection string |
| `.gitconfig` | Konfigurasi Git termasuk credential helper | Potensi kebocoran token repositori |
| `.netrc` | Credential untuk FTP/HTTP | Menyimpan username dan password dalam plaintext |

**Cara membaca output `namei -l`:**

`namei -l` menelusuri setiap komponen path dari root (`/`) hingga direktori target dan menampilkan permission serta owner di setiap level:

```
f: /home/student
drwxr-xr-x root    root    /
drwxr-xr-x root    root    home
drwx------ student student student
```

Dalam contoh ini, direktori `/home/student` memiliki mode `drwx------` yang berarti hanya owner (`student`) yang dapat masuk ke dalamnya. Ini adalah konfigurasi yang benar untuk home directory — mencegah user lain melihat atau mengakses file di dalamnya.

### 2. Konfigurasi dan Audit Permission File

```bash
mkdir -p ~/mod3/filesystem/labdata
cd ~/mod3/filesystem/labdata
touch note.txt report.txt script.sh

chmod 600 note.txt
chmod 640 report.txt
chmod 750 script.sh

ls -l | tee ~/mod3/evidence/filesystem/ownership_review.txt
stat note.txt report.txt script.sh | tee ~/mod3/evidence/filesystem/stat_review.txt
```

**Analisis keamanan tiap konfigurasi permission:**

**`note.txt` — Mode `600` (`-rw-------`):**

Hanya owner yang dapat membaca dan menulis. Tidak ada akses sama sekali untuk grup dan pengguna lain. Ini adalah mode yang tepat untuk file yang berisi informasi sensitif seperti kunci enkripsi, token API, atau catatan yang tidak boleh dibaca pihak lain.

**`report.txt` — Mode `640` (`-rw-r-----`):**

Owner dapat membaca dan menulis. Anggota grup hanya dapat membaca. Pengguna lain tidak memiliki akses. Cocok untuk dokumen yang perlu dibagikan dalam tim (*group*) tanpa memberikan hak modifikasi.

**`script.sh` — Mode `750` (`-rwxr-x---`):**

Owner memiliki akses penuh termasuk eksekusi. Anggota grup dapat membaca dan menjalankan skrip namun tidak dapat memodifikasinya. Pengguna lain tidak memiliki akses. Ini adalah konfigurasi standar untuk skrip otomasi yang digunakan oleh tim operasi.

**Tabel referensi notasi oktal:**

| Oktal | Biner | Simbol | Makna |
|-------|-------|--------|-------|
| `7` | `111` | `rwx` | Baca, tulis, dan eksekusi |
| `6` | `110` | `rw-` | Baca dan tulis |
| `5` | `101` | `r-x` | Baca dan eksekusi |
| `4` | `100` | `r--` | Hanya baca |
| `0` | `000` | `---` | Tidak ada akses |

**Informasi tambahan dari `stat` yang tidak ada di `ls -l`:**

```bash
stat note.txt
```

```
  File: note.txt
  Size: 0               Blocks: 0          IO Block: 4096
Device: fd00h           Inode: 131073      Links: 1
Access: (0600/-rw-------)  Uid: (1001/student)  Gid: (1001/student)
Access: 2024-04-27 10:00:00.123456789
Modify: 2024-04-27 10:00:00.123456789
Change: 2024-04-27 10:00:01.234567890
```

| Field `stat` | Relevansi Forensik |
|--------------|-------------------|
| `Inode` | Identitas unik file — berguna untuk melacak hard link |
| `Links` | Jumlah hard link — lebih dari 1 mengindikasikan file yang sama direferensikan dari beberapa path |
| `Access` (atime) | Kapan terakhir file dibaca — dapat digunakan untuk timeline rekonstruksi |
| `Modify` (mtime) | Kapan terakhir isi file diubah — sering menjadi fokus dalam forensik |
| `Change` (ctime) | Kapan terakhir metadata file diubah — tidak dapat dimanipulasi semudah mtime |

### 3. Pencarian File Berdasarkan Pola Relevan

```bash
find ~/datasets ~/samples -maxdepth 3 -type f \
  \( -iname "*.log" -o -iname "*.json" -o -iname "*.pcap" \
     -o -iname "*.csv"  -o -iname "*.txt" \) | sort | \
  tee ~/mod3/evidence/filesystem/find_sensitive_like.txt
```

**Ekstensi yang dicari dan relevansinya:**

| Ekstensi | Konten Umum | Nilai untuk Investigasi |
|----------|-------------|------------------------|
| `.log` | Event log sistem atau aplikasi | Rekonstruksi timeline aktivitas |
| `.pcap` | Packet capture jaringan | Analisis traffic dan deteksi anomali |
| `.json` | Data terstruktur, konfigurasi API | Potensi berisi token atau credential |
| `.csv` | Data tabular, export database | Potensi berisi data pengguna atau transaksi |
| `.txt` | Catatan, konfigurasi sederhana | Bervariasi — perlu inspeksi lebih lanjut |

## Temuan Utama

Review home directory mengungkap keberadaan direktori `.ssh/` yang mengandung file `authorized_keys`. Keberadaan file ini perlu diverifikasi: apakah public key yang terdaftar di dalamnya memang sesuai dengan pengguna yang berwenang, atau ada key yang tidak dikenal yang ditambahkan tanpa sepengetahuan pemilik akun.

Analisis `stat` mengungkap bahwa timestamp `ctime` (change time) berbeda dari `mtime` (modify time) pada beberapa file — perbedaan ini normal jika hanya metadata yang berubah (seperti permission), namun menjadi indikator yang perlu diselidiki jika tidak ada perubahan yang diharapkan.

## Apa yang Dipelajari

Analisis file system yang profesional bukan tentang membaca nama file — ini tentang membaca seluruh konteks: siapa yang memilikinya, kapan terakhir diakses atau dimodifikasi, bagaimana permission dikonfigurasi di setiap level path, dan apakah metadata tersebut konsisten dengan penggunaan yang diharapkan. Ketidakkonsistenan dalam metadata seringkali adalah sinyal pertama dari aktivitas yang mencurigakan.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `home_listing.txt` | `~/mod3/evidence/filesystem/` | Isi home directory termasuk hidden files |
| `home_path_permissions.txt` | `~/mod3/evidence/filesystem/` | Permission tiap komponen path home directory |
| `ownership_review.txt` | `~/mod3/evidence/filesystem/` | Output `ls -l` untuk tiga file uji |
| `stat_review.txt` | `~/mod3/evidence/filesystem/` | Metadata detail dari tiga file uji |
| `find_sensitive_like.txt` | `~/mod3/evidence/filesystem/` | Daftar file berdasarkan ekstensi yang relevan |

---
---

# Proyek 4 — Mengobservasi Proses, Session Aktif, dan Relasi Port–Service

## Latar Belakang

Proses adalah unit kerja sistem operasi yang paling mendasar. Setiap program yang berjalan, setiap service yang aktif, setiap koneksi jaringan yang terbuka — semuanya berakar pada satu atau lebih proses. Kemampuan untuk membaca hierarki proses, memahami relasi parent-child, mengidentifikasi session user yang aktif, dan menghubungkan proses dengan port jaringan adalah kompetensi inti dalam *incident response* dan *threat hunting*.

Proyek ini membangun kemampuan observasi proses secara menyeluruh: dari review proses berdasarkan resource usage, analisis hierarki proses dengan `pstree`, review session aktif dengan `w` dan `who`, hingga menghubungkan port yang listen dengan proses yang memilikinya — semua dari dua perspektif berbeda (internal dan eksternal).

## Tujuan

- Mengidentifikasi proses yang aktif dan mengonsumsi resource terbesar
- Membaca hierarki parent-child proses menggunakan `pstree` untuk memahami bagaimana proses dilahirkan
- Mereview session user yang sedang aktif dan riwayat login menggunakan `w`, `who`, dan `last`
- Menghubungkan port yang sedang listen dengan proses pemiliknya menggunakan `ss`
- Memvalidasi service dari perspektif eksternal menggunakan `nmap` dari Kali Linux

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Review proses, session, dan port dari perspektif internal |
| **Kali Linux** | Validasi service dari perspektif assessment eksternal |

## Langkah-Langkah yang Dilakukan

### 1. Review Proses Berdasarkan CPU dan Memory

```bash
ps -eo pid,ppid,user,%cpu,%mem,tty,stat,start,time,command --sort=-%cpu | head -n 25 | \
    tee ~/mod3/evidence/processes/ps_review.txt
```

**Kolom yang perlu diperhatikan dalam security context:**

| Kolom | Yang Dicari | Indikasi Anomali |
|-------|-------------|-----------------|
| `user` | Proses yang berjalan sebagai `root` tanpa alasan jelas | Privilege escalation atau backdoor |
| `ppid` | Proses dengan PPID yang tidak sesuai (misalnya shell sebagai anak dari web server) | Eksploitasi atau command injection |
| `%cpu` | Proses yang mengonsumsi CPU sangat tinggi secara konsisten | Cryptominer atau brute force |
| `%mem` | Proses dengan konsumsi memori yang terus bertambah | Memory leak atau data exfiltration dalam memori |
| `tty` | Proses interaktif yang berjalan dari terminal yang tidak dikenal | Session yang tidak sah |
| `stat` | Proses dengan status `Z` (zombie) | Proses yang tidak dibersihkan dengan benar |
| `command` | Nama command yang tidak dikenal atau menggunakan path tidak lazim | Malware atau tool penyerang |

### 2. Analisis Hierarki Proses dengan pstree

```bash
pstree -ap | head -n 80 | tee ~/mod3/evidence/processes/pstree_review.txt
```

**Penjelasan flag `pstree -ap`:**

| Flag | Fungsi |
|------|--------|
| `-a` | Tampilkan argument command line setiap proses |
| `-p` | Tampilkan PID setiap proses |

**Mengapa hierarki proses penting dalam investigasi:**

Dalam serangan berbasis *web shell* atau *command injection*, penyerang menjalankan perintah melalui proses yang menjadi anak (*child*) dari web server. Pola yang mencurigakan akan terlihat jelas di `pstree`:

```
nginx(1024)─┬─nginx(1025)
            └─nginx(1026)─┬─bash(5678)    ← MENCURIGAKAN: bash sebagai anak nginx
                           └─nc(5679)      ← SANGAT MENCURIGAKAN: netcat dijalankan dari nginx
```

Sebaliknya, hierarki yang normal untuk web server:
```
nginx(1024)─┬─nginx(1025)
            └─nginx(1026)
```

### 3. Review Session User Aktif dan Riwayat Login

```bash
w    | tee ~/mod3/evidence/processes/w_output.txt
who  | tee ~/mod3/evidence/processes/who_output.txt
last -n 10 | tee ~/mod3/evidence/processes/last_output.txt
```

**Perbandingan `w` vs `who`:**

| Aspek | `w` | `who` |
|-------|-----|-------|
| Informasi yang ditampilkan | User, terminal, login time, idle time, CPU usage, command aktif | User, terminal, login time, IP asal |
| Kegunaan | Melihat apa yang sedang dilakukan user saat ini | Melihat siapa yang login dan dari mana |
| Relevansi keamanan | Mendeteksi aktivitas mencurigakan dari user aktif | Mendeteksi login dari IP yang tidak dikenal |

**Cara membaca output `w`:**
```
USER     TTY      FROM             LOGIN@   IDLE JCPU PCPU WHAT
student  pts/0    10.10.10.5       10:00    0.00s 0.05s 0.01s w
alice    pts/1    10.10.10.99      09:45    5:23  0.12s 0.12s vim /etc/passwd
```

Baris kedua perlu diselidiki: user `alice` dari IP `10.10.10.99` sedang membuka `/etc/passwd` menggunakan `vim`. Meski membaca `/etc/passwd` adalah operasi yang legal, menggunakannya dengan `vim` dari IP yang tidak biasa patut dicatat.

**Cara membaca `last`:**

`last` membaca file `/var/log/wtmp` yang mencatat seluruh login dan logout. Ini adalah sumber yang berguna untuk rekonstruksi timeline aktivitas user.

```
student  pts/0        10.10.10.5     Sun Apr 27 10:00   still logged in
alice    pts/1        10.10.10.99    Sun Apr 27 09:45   still logged in
bob      pts/2        10.10.10.7     Sat Apr 26 22:30 - 23:15  (00:45)
```

### 4. Relasi Port dan Proses — Perspektif Internal dan Eksternal

```bash
# Ubuntu Lab — perspektif internal
ss -tulpn | tee ~/mod3/evidence/processes/ss_review.txt

# Kali Linux — perspektif assessment eksternal
nmap -sV -Pn -p 3000,8081 127.0.0.1 -oN ~/mod3/evidence/processes/nmap_targets.txt
```

**Menghubungkan output `ss` dengan informasi proses:**

```
Netid  State   Local Address:Port  Process
tcp    LISTEN  0.0.0.0:3000        users:(("node",pid=2341,fd=19))
tcp    LISTEN  0.0.0.0:8081        users:(("nginx",pid=3012,fd=6))
tcp    LISTEN  0.0.0.0:22          users:(("sshd",pid=1100,fd=3))
```

Kolom `Process` di output `ss -p` langsung menunjukkan nama proses dan PID yang memiliki socket tersebut. Dengan PID ini, analis dapat menggunakan `ps` untuk mendapatkan informasi lengkap tentang proses tersebut — termasuk user yang menjalankannya, kapan dimulai, dan argument command line-nya.

## Temuan Utama

Analisis `pstree` mengungkap bahwa proses `node` (yang melayani port 3000) adalah anak dari proses `dockerd` — konsisten dengan ekspektasi bahwa aplikasi dijalankan dalam container Docker. Ini adalah pola yang normal.

Review `last` menunjukkan bahwa user `bob` memiliki sesi login singkat dari IP `10.10.10.7` pada malam hari (22:30). Meski tidak dapat langsung disimpulkan sebagai ancaman, login di luar jam kerja normal dari IP yang berbeda perlu didokumentasikan untuk korelasi dengan temuan log lainnya.

## Apa yang Dipelajari

Observasi proses yang efektif membutuhkan tiga lapisan: *apa* yang berjalan (`ps`), *bagaimana* relasi antar proses (`pstree`), dan *siapa* yang sedang aktif (`w`, `who`, `last`). Ketiga lapisan ini saling melengkapi — temuan di satu lapisan hampir selalu memerlukan verifikasi di lapisan lain sebelum kesimpulan dapat ditarik.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `ps_review.txt` | `~/mod3/evidence/processes/` | Top 25 proses berdasarkan CPU |
| `pstree_review.txt` | `~/mod3/evidence/processes/` | Hierarki proses 80 baris pertama |
| `w_output.txt` | `~/mod3/evidence/processes/` | Session aktif dengan aktivitas saat ini |
| `who_output.txt` | `~/mod3/evidence/processes/` | User yang login beserta terminal dan IP |
| `last_output.txt` | `~/mod3/evidence/processes/` | 10 riwayat login terakhir |
| `ss_review.txt` | `~/mod3/evidence/processes/` | Port listening dengan proses pemilik |
| `nmap_targets.txt` | `~/mod3/evidence/processes/` | Hasil validasi service dari Kali |

---
---

# Proyek 5 — Menganalisis Log Sistem, Auth Log, PCAP, dan Menjalankan YARA

## Latar Belakang

Log adalah memori sistem operasi. Setiap autentikasi, setiap perubahan konfigurasi, setiap koneksi jaringan — semua dicatat dalam berbagai sumber log yang berbeda. Tantangan bagi seorang *security analyst* bukan kekurangan data, melainkan kemampuan untuk menavigasi banyak sumber log, mengidentifikasi pola yang signifikan, dan menghubungkan temuan dari satu sumber dengan sumber lainnya untuk membangun gambaran yang komprehensif.

Proyek ini mengintegrasikan empat sumber analisis yang berbeda namun saling melengkapi: journal sistem untuk event OS, auth log untuk aktivitas autentikasi, PCAP sample untuk traffic jaringan, dan YARA untuk deteksi berbasis rule pada artefak teks. Kombinasi keempat sumber ini mencerminkan alur kerja monitoring operasional yang digunakan dalam SOC (*Security Operations Center*) nyata.

## Tujuan

- Membaca dan menginterpretasikan event dari journal sistem menggunakan `journalctl`
- Menganalisis pola autentikasi dari auth log sample untuk mengidentifikasi aktivitas mencurigakan
- Meninjau traffic dari PCAP sample menggunakan `tshark` dan menghubungkannya dengan temuan lain
- Menulis rule YARA sederhana dan menerapkannya pada sampel artefak untuk mendeteksi string berisiko

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Seluruh analisis log, PCAP, dan YARA dilakukan di sini |
| **Dataset** | Auth log sample, PCAP sample, dan phishing sample dari `~/datasets` |

## Langkah-Langkah yang Dilakukan

### 1. Review Journal Sistem dan Auth Log Sample

```bash
journalctl -n 50 --no-pager | tee ~/mod3/evidence/monitoring/journal_review.txt

grep -i "failed\|accepted" ~/datasets/logs/linux/auth.log.sample | \
    tee ~/mod3/evidence/monitoring/auth_log_review.txt
```

**Sumber log yang relevan di sistem Linux:**

| Sumber | Lokasi | Isi |
|--------|--------|-----|
| systemd journal | `journalctl` | Event OS, service start/stop, kernel messages |
| Auth log | `/var/log/auth.log` | Login, sudo, PAM, SSH |
| Syslog | `/var/log/syslog` | Event aplikasi dan sistem umum |
| Kern log | `/var/log/kern.log` | Pesan kernel termasuk hardware events |
| UFW log | `/var/log/ufw.log` | Traffic yang diblokir firewall |

**Pola deteksi dalam auth log:**

| Pola | Regex / String | Indikasi |
|------|----------------|----------|
| Brute force SSH | `Failed password .* from <IP>` berulang dalam interval singkat | Percobaan otomatis menebak password |
| Successful login setelah banyak gagal | `Failed` berulang diikuti `Accepted` dari IP yang sama | Brute force berhasil |
| Invalid username | `Invalid user <name> from <IP>` | Enumerasi username |
| Root login attempt | `Failed password for root from` | Upaya langsung mengakses root |
| Sudo usage | `sudo: <user> : COMMAND=` | Pencatatan penggunaan hak elevated |

### 2. Tinjauan PCAP Sample dengan tshark

```bash
tshark -r ~/datasets/pcap/localhost_web_lab.pcap | head -n 20 | \
    tee ~/mod3/evidence/monitoring/pcap_review.txt
```

**Analisis mendalam output tshark:**

`tshark` mendekode protokol secara otomatis dan menampilkan ringkasan setiap frame:

```
1   0.000000 127.0.0.1 → 127.0.0.1    TCP  74  54321 → 3000 [SYN] Seq=0
2   0.000234 127.0.0.1 → 127.0.0.1    TCP  74  3000 → 54321 [SYN, ACK]
3   0.000298 127.0.0.1 → 127.0.0.1    TCP  66  54321 → 3000 [ACK]
4   0.001123 127.0.0.1 → 127.0.0.1    HTTP 234 GET /rest/user/login HTTP/1.1
5   0.002891 127.0.0.1 → 127.0.0.1    HTTP 89  HTTP/1.1 401 Unauthorized
6   0.003201 127.0.0.1 → 127.0.0.1    HTTP 234 GET /rest/user/login HTTP/1.1
7   0.004112 127.0.0.1 → 127.0.0.1    HTTP 89  HTTP/1.1 401 Unauthorized
```

**Interpretasi PCAP di atas:**

Frame 4-7 menunjukkan dua request ke `/rest/user/login` yang keduanya mendapat respons `401 Unauthorized`. Ini adalah pola yang bisa mengindikasikan:
- Percobaan login manual yang gagal
- *Automated credential testing* — terutama jika pola ini berulang puluhan atau ratusan kali

Hubungkan dengan auth log: jika ada `Failed password` di auth log pada timestamp yang sama dengan request ini di PCAP, kedua sumber saling mengkonfirmasi temuan yang sama.

### 3. Deteksi Berbasis Rule dengan YARA

```bash
cat > ~/mod3/tmp/basic_keywords.yar <<'EOF'
rule suspicious_keywords {
    meta:
        description = "Mendeteksi kata kunci yang umum ditemukan dalam phishing atau credential exposure"
        author      = "Student Lab"
        date        = "2024-04-27"
    strings:
        $a = "urgent"   nocase
        $b = "verify"   nocase
        $c = "password" nocase
    condition:
        any of them
}
EOF

yara ~/mod3/tmp/basic_keywords.yar ~/datasets/phishing/sample_email_1.txt | \
    tee ~/mod3/evidence/monitoring/yara_result.txt
```

**Anatomi rule YARA:**

```yara
rule <nama_rule> {
    meta:        // Metadata — tidak mempengaruhi deteksi, hanya dokumentasi
    strings:     // Pola yang dicari (string, hex, regex)
    condition:   // Logika kapan rule dianggap match
}
```

**Bagian `strings` — jenis pola yang didukung:**

| Tipe | Contoh | Kegunaan |
|------|--------|----------|
| String teks | `$a = "urgent" nocase` | Mencari kata kunci teks (case-insensitive dengan `nocase`) |
| Hex pattern | `$b = { 4D 5A 90 00 }` | Mencari byte sequence (signature executable) |
| Regex | `$c = /password\s*=/i` | Mencari pola kompleks dengan ekspresi reguler |

**Bagian `condition` — contoh logika:**

| Kondisi | Artinya |
|---------|---------|
| `any of them` | Match jika minimal satu string ditemukan |
| `all of them` | Match hanya jika semua string ditemukan |
| `2 of them` | Match jika minimal 2 string ditemukan |
| `$a and $b` | Match jika $a DAN $b keduanya ditemukan |
| `$a at 0` | Match jika $a ditemukan di offset 0 (awal file) |

**Contoh output YARA:**
```
suspicious_keywords /home/student/datasets/phishing/sample_email_1.txt
```

Output ini menunjukkan bahwa rule `suspicious_keywords` berhasil mendeteksi setidaknya satu dari string yang dicari dalam file phishing sample — mengkonfirmasi bahwa file tersebut mengandung kata kunci yang relevan dengan social engineering.

## Temuan Utama

**Dari auth log:** Ditemukan 7 event `Failed password for root` dari IP `10.10.10.99` dalam rentang 30 detik — frekuensi yang mengindikasikan *automated brute force*. Menariknya, tidak ada `Accepted` login dari IP tersebut, yang berarti serangan tidak berhasil menembus autentikasi.

**Dari PCAP:** Traffic ke endpoint `/rest/user/login` menunjukkan pola request yang berulang dengan respons 401 — mengkonfirmasi adanya percobaan otomatis yang berkorelasi dengan temuan auth log.

**Dari YARA:** Rule berhasil mendeteksi string `"urgent"` dan `"password"` dalam sampel email phishing, mengkonfirmasi bahwa artefak tersebut mengandung karakteristik teknik social engineering yang umum.

**Korelasi ketiga sumber:** IP `10.10.10.99` muncul di auth log sebagai sumber brute force, dan pola request di PCAP konsisten dengan traffic otomatis. Ini adalah contoh bagaimana triangulasi dari beberapa sumber meningkatkan kepercayaan temuan secara signifikan.

## Apa yang Dipelajari

Analisis log yang efektif bukan tentang membaca satu sumber secara mendalam — ini tentang menghubungkan sinyal dari beberapa sumber untuk membangun gambaran yang koheren. Satu `Failed password` di auth log bisa jadi typo biasa. Tujuh `Failed password` dalam 30 detik dari IP yang sama, dikonfirmasi oleh pola traffic di PCAP, adalah temuan yang memerlukan tindakan.

YARA memperkenalkan konsep *rule-based thinking* — sebuah pendekatan yang scalable karena rule yang sama dapat dijalankan pada ribuan file sekaligus, memungkinkan deteksi yang konsisten tanpa review manual satu per satu.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `journal_review.txt` | `~/mod3/evidence/monitoring/` | 50 event jurnal sistem terbaru |
| `auth_log_review.txt` | `~/mod3/evidence/monitoring/` | Baris failed dan accepted dari auth log sample |
| `pcap_review.txt` | `~/mod3/evidence/monitoring/` | Ringkasan 20 frame pertama dari PCAP sample |
| `yara_result.txt` | `~/mod3/evidence/monitoring/` | Hasil deteksi YARA pada phishing sample |

---
---

# Proyek 6 — Membangun Automation Parsing Evidence dengan Bash dan Python

## Latar Belakang

Dalam operasi keamanan skala nyata, volume log yang perlu dianalisis jauh melampaui kemampuan review manual. Seorang *security analyst* yang hanya dapat membaca log secara manual — baris per baris — akan segera kewalahan dalam menghadapi ribuan event per jam yang dihasilkan sistem produksi. Automation adalah solusinya: parser yang dibangun dengan baik dapat memproses ribuan baris log dalam hitungan detik dan menghasilkan ringkasan yang langsung dapat ditindaklanjuti.

Proyek ini membangun dua parser yang saling melengkapi: parser Bash untuk triage cepat dan deteksi berbasis grep yang dapat langsung dijalankan dari terminal, dan parser Python untuk analisis yang lebih terstruktur dengan output yang lebih kaya informasi. Keduanya bekerja pada dataset yang sama — auth log sample — sehingga hasilnya dapat dibandingkan dan divalidasi.

## Tujuan

- Membangun Bash parser yang dapat memfilter, mengelompokkan, dan meringkas pola dalam auth log
- Membangun Python parser yang menggunakan regex dan Counter untuk analisis yang lebih terstruktur
- Membandingkan output kedua pendekatan dan memahami kapan masing-masing lebih sesuai digunakan
- Memahami konsep dan posisi cron dalam konteks automation keamanan operasional

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Seluruh scripting dan automation dilakukan di sini |

## Langkah-Langkah yang Dilakukan

### 1. Bash Parser untuk Auth Log Sample

```bash
cat > ~/mod3/tmp/auth_parser.sh <<'EOF'
#!/usr/bin/env bash
# auth_parser.sh
# Parser sederhana untuk mengekstrak pola failed login dari auth log
# Bagian dari Modul 03 — Operating Systems & Scripting

LOG=~/datasets/logs/linux/auth.log.sample

if [ ! -f "$LOG" ]; then
    echo "[ERROR] File log tidak ditemukan: $LOG"
    exit 1
fi

echo "=== Failed Login Events ==="
grep -i "Failed password" "$LOG"

echo ""
echo "=== Source IP Summary (diurutkan berdasarkan frekuensi) ==="
grep -i "Failed password" "$LOG" | awk '{print $(NF-3), $(NF-1)}' | sort | uniq -c | sort -rn

echo ""
echo "=== Total Failed Login: $(grep -c -i 'Failed password' "$LOG") events"
EOF

chmod +x ~/mod3/tmp/auth_parser.sh
~/mod3/tmp/auth_parser.sh | tee ~/mod3/evidence/automation/bash_parser_output.txt
```

**Penjelasan pipeline `awk '{print $(NF-3), $(NF-1)}'`:**

Pada baris auth log seperti:
```
May 15 10:23:11 ubuntu sshd[1234]: Failed password for root from 10.10.10.99 port 45231 ssh2
```

`NF` (Number of Fields) adalah jumlah total kolom. Dengan memilih `$(NF-3)` dan `$(NF-1)`, kita mengekstrak:
- `$(NF-3)` → `10.10.10.99` (alamat IP sumber)
- `$(NF-1)` → `45231` (nomor port)

Pipeline `sort | uniq -c | sort -rn` kemudian mengelompokkan dan menghitung frekuensi kemunculan setiap kombinasi IP-port, diurutkan dari yang paling sering muncul.

**Contoh output `bash_parser_output.txt`:**
```
=== Failed Login Events ===
May 15 10:23:11 ubuntu sshd[1234]: Failed password for root from 10.10.10.99 port 45231 ssh2
May 15 10:23:14 ubuntu sshd[1234]: Failed password for root from 10.10.10.99 port 45234 ssh2
May 15 10:23:17 ubuntu sshd[1234]: Failed password for root from 10.10.10.99 port 45237 ssh2

=== Source IP Summary (diurutkan berdasarkan frekuensi) ===
      7 10.10.10.99 ssh2
      1 10.10.10.5  ssh2

=== Total Failed Login: 8 events
```

### 2. Python Parser untuk Auth Log Sample

```bash
cat > ~/mod3/tmp/auth_parser.py <<'EOF'
"""
auth_parser.py
Parser terstruktur untuk analisis pola failed login pada auth log.
Bagian dari Modul 03 — Operating Systems & Scripting
"""

import re
from collections import Counter
from pathlib import Path

# Konfigurasi
LOG_PATH = Path.home() / 'datasets' / 'logs' / 'linux' / 'auth.log.sample'
PATTERN  = re.compile(r'Failed password .* from (\S+) port (\S+)')

def parse_auth_log(log_path: Path) -> dict:
    """Parsing auth log dan kembalikan ringkasan terstruktur."""
    ip_counter   = Counter()
    port_counter = Counter()
    failed_lines = []

    for line in log_path.read_text().splitlines():
        match = PATTERN.search(line)
        if match:
            ip, port = match.group(1), match.group(2)
            ip_counter[ip]     += 1
            port_counter[port] += 1
            failed_lines.append(line)

    return {
        'total_failed'   : len(failed_lines),
        'ip_summary'     : dict(ip_counter.most_common()),
        'port_summary'   : dict(port_counter.most_common(5)),
        'failed_entries' : failed_lines,
    }

def main():
    if not LOG_PATH.exists():
        print(f'[ERROR] File log tidak ditemukan: {LOG_PATH}')
        return

    result = parse_auth_log(LOG_PATH)

    print('=== Ringkasan Analisis Auth Log ===')
    print(f"Total failed login  : {result['total_failed']}")
    print(f"IP paling aktif     : {result['ip_summary']}")
    print(f"Port sumber teratas : {result['port_summary']}")

    # Identifikasi IP dengan percobaan terbanyak
    if result['ip_summary']:
        top_ip    = list(result['ip_summary'].keys())[0]
        top_count = result['ip_summary'][top_ip]
        if top_count >= 3:
            print(f'\n[PERINGATAN] IP {top_ip} melakukan {top_count} percobaan login gagal')
            print('Kemungkinan aktivitas brute force — perlu investigasi lebih lanjut')

if __name__ == '__main__':
    main()
EOF

python ~/mod3/tmp/auth_parser.py | tee ~/mod3/evidence/automation/python_parser_output.txt
```

**Penjelasan komponen Python yang digunakan:**

| Komponen | Fungsi |
|----------|--------|
| `re.compile(pattern)` | Mengkompilasi regex untuk efisiensi — penting jika memproses file besar |
| `Counter` | Menghitung frekuensi kemunculan secara otomatis dari setiap sumber `collections` |
| `Path.home()` | Mengembalikan path home directory secara portabel (bekerja di semua OS) |
| `.most_common()` | Mengurutkan Counter dari yang paling sering ke yang paling jarang |
| `match.group(1)` | Mengekstrak capturing group pertama dari regex match |

**Contoh output `python_parser_output.txt`:**
```
=== Ringkasan Analisis Auth Log ===
Total failed login  : 8
IP paling aktif     : {'10.10.10.99': 7, '10.10.10.5': 1}
Port sumber teratas : {'45231': 1, '45234': 1, '45237': 1, '45240': 1, '45243': 1}

[PERINGATAN] IP 10.10.10.99 melakukan 7 percobaan login gagal
Kemungkinan aktivitas brute force — perlu investigasi lebih lanjut
```

### 3. Catatan Konsep Automation Berkala (Cron)

```bash
cat > ~/mod3/evidence/automation/cron_notes.md <<'EOF'
# Cron Notes — Konsep dan Posisi dalam Automation Keamanan

## Apa itu Cron
Cron adalah job scheduler Unix yang menjalankan perintah atau script secara otomatis
pada jadwal yang ditentukan. Format jadwal: menit jam hari bulan hari-minggu.

## Contoh Skenario Automation Keamanan

| Task | Jadwal Cron | Contoh Command |
|------|------------|----------------|
| Parsing auth log setiap jam | `0 * * * *` | `~/mod3/tmp/auth_parser.sh >> ~/mod3/logs/hourly_auth.log` |
| Checksum file kritis harian | `0 6 * * *` | `sha256sum /etc/passwd >> ~/checksums/daily.txt` |
| Backup evidence mingguan | `0 2 * * 0` | `tar -czf ~/backup/evidence_$(date +%Y%m%d).tar.gz ~/mod3/evidence` |

## Batasan dalam Lingkungan Lab
- Akun student non-sudo — tidak dapat memodifikasi cron system-wide
- Fokus modul ini adalah memahami konsep dan kapabilitas cron
- Implementasi automation di lingkungan produksi memerlukan review dan approval

## Prinsip Automation yang Aman
- Automation harus idempotent — dijalankan berkali-kali menghasilkan output yang sama
- Setiap script otomatis harus memiliki logging yang mencatat waktu dan hasilnya
- Error handling harus jelas — script tidak boleh gagal diam-diam tanpa notifikasi
- Automation keamanan harus di-review sebelum deployment untuk mencegah false positive
EOF
```

## Perbandingan Bash vs Python untuk Log Parsing

| Aspek | Bash Parser | Python Parser |
|-------|------------|---------------|
| **Kecepatan setup** | Sangat cepat — langsung dijalankan | Perlu penulisan script dengan struktur |
| **Keterbacaan output** | Terbatas — tergantung formatting manual | Terstruktur — mudah dikustomisasi |
| **Error handling** | Terbatas | Komprehensif dengan try/except |
| **Regex capability** | Terbatas (grep/awk) | Penuh dengan modul `re` |
| **Data structures** | Tidak ada (hanya teks) | Dict, Counter, List, dll. |
| **Portabilitas** | Terbatas di Linux/macOS | Lintas platform |
| **Penggunaan ideal** | Triage cepat, one-liner, pipeline sederhana | Analisis terstruktur, reporting, integrasi sistem |

Kesimpulan: gunakan Bash ketika kecepatan dan kesederhanaan adalah prioritas. Gunakan Python ketika akurasi, struktur output, dan kemampuan pengembangan lebih lanjut lebih penting.

## Temuan Utama

Kedua parser menghasilkan temuan yang konsisten: IP `10.10.10.99` melakukan 7 percobaan login gagal — proporsi yang jauh di atas threshold normal dan mengindikasikan aktivitas otomatis. Python parser menambahkan nilai dengan secara otomatis mengidentifikasi dan memberikan peringatan untuk IP yang melampaui ambang batas, tanpa perlu interpretasi manual.

## Apa yang Dipelajari

Automation yang efektif bukan tentang menggantikan analis — ini tentang membebaskan analis dari pekerjaan repetitif sehingga mereka dapat fokus pada investigasi yang memerlukan penilaian manusia. Parser yang dibangun di proyek ini adalah fondasi dari apa yang kemudian berkembang menjadi sistem alerting, SIEM rule, atau threat hunting script yang lebih kompleks.

## Referensi File Bukti dan Script

| File | Lokasi | Isi |
|------|--------|-----|
| `bash_parser_output.txt` | `~/mod3/evidence/automation/` | Output Bash parser: failed events dan IP summary |
| `python_parser_output.txt` | `~/mod3/evidence/automation/` | Output Python parser: ringkasan terstruktur dan peringatan |
| `cron_notes.md` | `~/mod3/evidence/automation/` | Konsep cron dan prinsip automation keamanan |
| `auth_parser.sh` | `~/mod3/tmp/` | Script Bash parser |
| `auth_parser.py` | `~/mod3/tmp/` | Script Python parser |

---
---

# Proyek 7 — Menyusun OS Security Assessment dan Jembatan ke Modul Berikutnya

## Latar Belakang

Seorang *security analyst* yang hanya dapat menjalankan tool namun tidak dapat mengkomunikasikan temuannya memiliki nilai yang sangat terbatas dalam sebuah tim. Kemampuan untuk mengubah sekumpulan output teknis menjadi assessment yang kohesif, berstruktur, dan dapat ditindaklanjuti adalah kompetensi yang membedakan analis junior dari analis yang benar-benar efektif.

Proyek terakhir ini menutup seluruh rangkaian analisis sistem operasi dengan dua deliverable profesional: sebuah *OS security assessment* yang merangkum seluruh temuan dari enam proyek sebelumnya dalam format yang siap dibaca oleh tim teknis atau manajemen, dan *operating notes* yang secara eksplisit menghubungkan fondasi yang telah dibangun dengan kapabilitas yang akan dikembangkan di modul berikutnya.

## Tujuan

- Merangkum seluruh temuan teknis dari proyek 1–6 dalam format assessment yang terstruktur dan profesional
- Mengidentifikasi risiko yang ditemukan, menilai tingkat keparahannya, dan memberikan rekomendasi konkret
- Mendokumentasikan prinsip kerja yang harus dibawa ke modul berikutnya sebagai *operating principles*
- Membiasakan pola kerja di mana setiap sesi analisis selalu ditutup dengan deliverable yang bernilai

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Seluruh penulisan assessment dilakukan di sini |

## Langkah-Langkah yang Dilakukan

### 1. OS Security Assessment

```bash
cat > ~/mod3/reports/os_security_assessment.md <<'EOF'
# OS Security Assessment — Modul 03

## Ringkasan Eksekutif
Analisis sistem operasi pada Ubuntu Lab menunjukkan konfigurasi yang sebagian besar sesuai
dengan prinsip keamanan dasar. Ditemukan beberapa temuan yang memerlukan perhatian, terutama
terkait keanggotaan grup yang memberikan hak akses elevated secara tidak langsung dan adanya
indikasi brute force SSH dari IP eksternal dalam dataset log yang ditinjau.

## Temuan Teknis per Domain

### 1. Host Baseline dan User Context
- User student adalah anggota grup `docker` — memberikan akses efektif setara root via container
- Kernel versi terkini diverifikasi — tidak ada known critical vulnerability yang teridentifikasi
- Resource state dalam kondisi normal: disk dan memori tidak mendekati kapasitas penuh

### 2. File System dan Permission
- Home directory dikonfigurasi dengan mode `drwx------` — isolasi user yang benar
- File yang dibuat selama sesi dikonfigurasi dengan permission yang sesuai (600, 640, 750)
- Hidden files di home directory perlu di-audit secara berkala, terutama `.ssh/authorized_keys`

### 3. Process dan Session
- Hierarki proses konsisten dengan ekspektasi — tidak ada proses anak yang mencurigakan
- Ditemukan sesi login singkat dari user `bob` pada jam tidak biasa dari IP yang berbeda
- Port 3000 dan 8081 diverifikasi aktif dan terikat ke localhost sesuai ekspektasi

### 4. Log dan Monitoring
- Auth log menunjukkan 7 percobaan failed login SSH dari IP 10.10.10.99 dalam 30 detik
- Pola PCAP mengkonfirmasi traffic otomatis ke endpoint login — berkorelasi dengan auth log
- YARA berhasil mendeteksi string social engineering dalam phishing sample

### 5. Automation
- Bash dan Python parser berhasil dibangun dan menghasilkan output yang konsisten
- Kedua parser mengkonfirmasi IP 10.10.10.99 sebagai sumber aktivitas brute force

## Risiko yang Teridentifikasi

| Temuan | Tingkat Risiko | Rekomendasi |
|--------|---------------|-------------|
| Keanggotaan grup `docker` untuk user student | Tinggi | Evaluasi apakah akses Docker diperlukan; gunakan rootless Docker jika diperlukan |
| SSH brute force dari 10.10.10.99 (dalam sample) | Sedang | Implementasikan `fail2ban` atau rate limiting SSH; pertimbangkan key-only authentication |
| Login user `bob` di jam tidak biasa | Rendah | Investigasi lebih lanjut dengan korelasi log lain; dokumentasikan sebagai anomali |
| Hidden files tidak diaudit secara berkala | Rendah | Jadwalkan review berkala `~/.ssh/authorized_keys` dan file konfigurasi lainnya |

## Kesimpulan
Sistem beroperasi secara normal namun memiliki beberapa area yang memerlukan penguatan.
Temuan paling kritis adalah keanggotaan grup `docker` yang memberikan hak akses elevated
secara implisit. Temuan lainnya bersifat monitoring dan perlu ditindaklanjuti dengan
investigasi yang lebih dalam menggunakan automation yang telah dibangun.
EOF
```

### 2. Operating Notes — Jembatan ke Modul Berikutnya

```bash
cat > ~/mod3/reports/operating_notes.md <<'EOF'
# Operating Notes — Prinsip Kerja dan Jembatan ke Modul 04

## Prinsip Kerja yang Harus Dibawa ke Modul Berikutnya

1. **Evidence tetap terorganisir** — struktur direktori berbasis domain teknis, bukan satu folder
2. **Ubuntu sebagai primary analysis host** — jangan menjalankan analisis di Kali kecuali untuk validasi
3. **Kali sebagai validation workstation** — perspektif eksternal hanya dari sini
4. **Bash dan Python digunakan bersama** — Bash untuk kecepatan, Python untuk struktur
5. **Proses, log, dan metadata harus dikorelasikan** — tidak ada sumber tunggal yang cukup
6. **Setiap sesi ditutup dengan assessment** — output teknis tanpa interpretasi tidak bernilai

## Apa yang Telah Dibangun di Modul 03

| Kompetensi | Dibangun Melalui |
|------------|-----------------|
| Host baseline collection | Proyek 1 dan 2 |
| File system dan permission audit | Proyek 3 |
| Process dan session analysis | Proyek 4 |
| Log, PCAP, dan YARA analysis | Proyek 5 |
| Bash dan Python scripting | Proyek 6 |
| Security assessment writing | Proyek 7 |

## Koneksi ke Modul 04 — Scripting dan Automation Lanjutan

| Fondasi Modul 03 | Pengembangan di Modul 04 |
|------------------|--------------------------|
| Bash parser sederhana | Shell scripting yang lebih kompleks dengan flow control |
| Python parser dengan Counter dan regex | Modul Python, class, dan integrasi API |
| YARA rule sederhana | Rule YARA yang lebih kompleks untuk deteksi malware |
| Cron concept | Implementasi scheduled task dalam automation pipeline |
| Auth log analysis | Log aggregation dan correlation engine |

## Fondasi yang Siap Dibawa ke Modul 04

- File system dan permission dipahami secara operasional
- Process hierarchy dapat dibaca dan diinterpretasikan
- Log dari berbagai sumber dapat dikorelasikan
- Bash dan Python siap dipakai untuk automation
- Assessment writing sebagai habit — bukan langkah tambahan
EOF
```

## Struktur Assessment yang Profesional

Assessment yang baik mengikuti struktur yang dapat diprediksi sehingga berbagai pembaca dapat menemukan informasi yang mereka butuhkan dengan cepat:

| Bagian | Fungsi | Pembaca Utama |
|--------|--------|---------------|
| **Ringkasan Eksekutif** | Gambaran keseluruhan dalam 3-4 kalimat | Manajemen, pemimpin tim |
| **Temuan Teknis per Domain** | Detail observasi yang terorganisir | Analis, engineer, SOC |
| **Tabel Risiko** | Tingkat risiko dan rekomendasi yang dapat ditindaklanjuti | Security manager |
| **Kesimpulan** | Poin utama yang perlu diingat | Semua pembaca |

**Mengapa domain-based organization lebih baik dari chronological organization:**

Assessment yang disusun berdasarkan urutan kronologis ("pertama saya menjalankan ini, kemudian itu") membebani pembaca dengan detail yang tidak perlu. Assessment yang disusun berdasarkan domain teknis (baseline, file system, process, dll.) memungkinkan pembaca langsung menuju area yang relevan dengan tanggung jawabnya.

## Temuan Utama

Proyek ini mengkonfirmasi bahwa kualitas assessment berbanding lurus dengan kualitas analisis sebelumnya. Analis yang mengumpulkan evidence secara terstruktur, menjalankan korelasi antar sumber, dan mendokumentasikan temuan sepanjang sesi akan dapat menghasilkan assessment yang komprehensif dalam waktu singkat. Analis yang bekerja secara tidak terorganisir akan kesulitan merekonstruksi apa yang ditemukan bahkan setelah sesi selesai.

## Apa yang Dipelajari

Menutup setiap modul dengan assessment tertulis bukan formalitas — ini adalah cara analis memverifikasi bahwa mereka benar-benar memahami apa yang ditemukan, bukan hanya berhasil menjalankan perintah. Assessment yang tidak dapat ditulis adalah tanda bahwa pemahaman belum cukup dalam. Assessment yang ditulis dengan jelas adalah bukti kompetensi yang dapat dikomunikasikan kepada tim dan manajemen.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `os_security_assessment.md` | `~/mod3/reports/` | Assessment lengkap dengan temuan, risiko, dan rekomendasi |
| `operating_notes.md` | `~/mod3/reports/` | Prinsip kerja dan jembatan eksplisit ke Modul 04 |
