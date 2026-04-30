## Tentang Portofolio Ini

Portofolio ini mendokumentasikan serangkaian proyek *applied security* yang mengintegrasikan prinsip-prinsip inti keamanan siber ke dalam workflow yang dapat dijalankan secara langsung. Setiap proyek dirancang untuk mencerminkan cara berpikir seorang *security analyst* profesional dalam menilai sebuah environment: memetakan aset dan kontrol, membaca ancaman dari artefak nyata, memvalidasi identitas dan akses, meninjau bukti kriptografis, memeriksa eksposur aplikasi web, dan menyusun penilaian berorientasi *incident response*.

Seluruh aktivitas dilakukan pada dua host lab dengan peran yang berbeda:

| Host | Peran |
|------|-------|
| **Ubuntu Lab** | *Primary analysis host* — risk notes, control review, identity context, hash & certificate check, secure coding review, log & incident evidence, assessment writing |
| **Kali Linux** | *Validation workstation* — header validation, localhost web exposure, endpoint inspection, service verification, attacker-aware perspective dalam scope aman |
| **Local Targets** | *Safe practice target* — Juice Shop (`127.0.0.1:3000`) dan DVWA (`127.0.0.1:8081`) untuk validasi kontrol dan perilaku aplikasi |
| **Datasets `/opt/jccsah`** | Auth log, Apache log, phishing sample, PCAP, OSINT sample, starter scripts, report templates |

> ⚠️ **Catatan:** Seluruh aktivitas dilakukan **hanya** pada lingkungan lab yang telah disediakan. Tidak ada eksploitasi aktif, perubahan konfigurasi sistem, atau pengujian di luar dataset, local targets, dan scope yang sudah ditetapkan.

---

## Daftar Proyek

| No. | Judul Proyek | Fokus Area | Tools & Konsep |
|-----|-------------|------------|----------------|
| 1 | [Memetakan Aset, Kontrol, dan Prinsip Keamanan pada Environment Lab](#proyek-1--memetakan-aset-kontrol-dan-prinsip-keamanan-pada-environment-lab) | Security principles, asset mapping | CIA triad, Defense in Depth |
| 2 | [Menganalisis Threat Landscape dan Attack Surface Berdasarkan Artefak Nyata](#proyek-2--menganalisis-threat-landscape-dan-attack-surface-berdasarkan-artefak-nyata) | Threat analysis, risk context | `grep`, phishing analysis, OSINT review |
| 3 | [Mereview Konteks Identitas, Session, dan Least Privilege](#proyek-3--mereview-konteks-identitas-session-dan-least-privilege) | IAM, session review, access control | `whoami`, `id`, `groups`, `w`, `who` |
| 4 | [Memverifikasi Integritas File dan Meninjau Bukti Kriptografis](#proyek-4--memverifikasi-integritas-file-dan-meninjau-bukti-kriptografis) | Cryptography, hash verification, TLS | `sha256sum`, `openssl` |
| 5 | [Memvalidasi Security Header dan Eksposur Aplikasi Web](#proyek-5--memvalidasi-security-header-dan-eksposur-aplikasi-web) | Web security, AppSec, header analysis | `curl`, `whatweb`, OWASP principles |
| 6 | [Menyusun Incident Timeline dan Case Assessment Berorientasi Respons](#proyek-6--menyusun-incident-timeline-dan-case-assessment-berorientasi-respons) | Incident thinking, response planning | Timeline analysis, evidence correlation |
| 7 | [Menulis Security Assessment Komprehensif dan Jembatan ke Blue Team](#proyek-7--menulis-security-assessment-komprehensif-dan-jembatan-ke-blue-team) | Assessment writing, operational documentation | Dokumentasi profesional |

---

## Teknologi dan Tools

<p>
  <img src="https://img.shields.io/badge/OS-Ubuntu%20Linux-E95420?style=flat-square&logo=ubuntu&logoColor=white"/>
  <img src="https://img.shields.io/badge/OS-Kali%20Linux-557C94?style=flat-square&logo=kalilinux&logoColor=white"/>
  <img src="https://img.shields.io/badge/Tool-OpenSSL-721412?style=flat-square"/>
  <img src="https://img.shields.io/badge/Tool-curl%20%2F%20whatweb-073551?style=flat-square"/>
  <img src="https://img.shields.io/badge/Tool-sha256sum-informational?style=flat-square"/>
  <img src="https://img.shields.io/badge/Framework-CIA%20Triad-blue?style=flat-square"/>
  <img src="https://img.shields.io/badge/Framework-OWASP-red?style=flat-square"/>
  <img src="https://img.shields.io/badge/Framework-Defense%20in%20Depth-green?style=flat-square"/>
</p>

---

## Kompetensi yang Dibangun

- **Security Principles** — Menerapkan CIA triad, defense in depth, dan least privilege dalam konteks environment lab yang nyata, bukan hanya definisi teoritis
- **Threat Analysis** — Menganalisis artefak phishing dan OSINT sample untuk mengidentifikasi indikator ancaman yang konkret dan relevan
- **Identity & Access Management** — Membaca konteks user, keanggotaan grup, dan informasi session sebagai dasar evaluasi access control
- **Applied Cryptography** — Menggunakan `sha256sum` untuk verifikasi integritas evidence dan `openssl` untuk membaca metadata sertifikat TLS secara operasional
- **Web Application Security** — Memvalidasi security headers dan fingerprint aplikasi dari dua perspektif berbeda untuk menilai efektivitas kontrol
- **Incident-Oriented Thinking** — Menyusun timeline observasi, mendokumentasikan response priorities, dan menghasilkan case assessment yang siap dibaca tim teknis
- **Security Assessment Writing** — Mengintegrasikan seluruh observasi teknis menjadi penilaian yang komprehensif, proporsional, dan dapat ditindaklanjuti

---
---

# Proyek 1 — Memetakan Aset, Kontrol, dan Prinsip Keamanan pada Environment Lab

## Latar Belakang

Salah satu kesalahan paling umum dalam pendekatan keamanan siber adalah langsung berfokus pada ancaman sebelum memahami apa yang perlu dilindungi. Daftar ancaman yang panjang dan komprehensif tidak berguna jika tidak dikaitkan dengan aset spesifik yang ada dalam lingkungan yang sedang dinilai. Sebaliknya, pemahaman yang mendalam tentang aset, nilai, dan kontrol yang sudah ada memungkinkan analis untuk memprioritaskan ancaman secara proporsional — bukan reaktif terhadap semua kemungkinan ancaman sekaligus.

Proyek ini membangun fondasi *security reasoning* yang benar: dimulai dari pemetaan aset dan kontrol yang ada, kemudian menerjemahkan prinsip-prinsip inti keamanan (CIA triad dan defense in depth) ke dalam konteks lingkungan lab yang sedang digunakan — bukan sebagai abstraksi akademis, melainkan sebagai referensi yang akan digunakan sepanjang modul.

## Tujuan

- Membangun struktur workspace yang terbagi berdasarkan domain keamanan yang berbeda
- Memetakan aset, area bernilai tinggi, dan kontrol yang sudah ada dalam environment lab
- Menerjemahkan CIA triad ke dalam implikasi konkret untuk lingkungan yang sedang digunakan
- Mendokumentasikan bagaimana defense in depth diterapkan — atau tidak diterapkan — dalam konfigurasi lab saat ini

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Seluruh dokumentasi security reasoning dilakukan di sini |

## Langkah-Langkah yang Dilakukan

### 1. Pembangunan Struktur Workspace Modul 05

```bash
mkdir -p ~/mod5/{evidence,reports,notes,principles,threats,iam,crypto,web,incident,tmp}
cd ~/mod5
pwd
ls -la
```

**Signifikansi struktur direktori dalam konteks security reasoning:**

Berbeda dengan modul-modul sebelumnya yang mengorganisir evidence berdasarkan *tool* atau *aktivitas*, modul ini mengorganisir berdasarkan *domain keamanan*. Ini mencerminkan cara kerja seorang *security analyst* yang berpengalaman: setiap temuan langsung dikategorikan berdasarkan domain yang relevan (principles, threats, IAM, crypto, web, incident) sehingga assessment akhir dapat disusun dengan struktur yang kohesif.

| Direktori | Domain Keamanan |
|-----------|----------------|
| `evidence/principles/` | Security principles — CIA, defense in depth, asset mapping |
| `evidence/threats/` | Threat landscape — phishing, attack surface, risk context |
| `evidence/iam/` | Identity & access — user context, session, privilege |
| `evidence/crypto/` | Cryptography — hash, certificate, TLS metadata |
| `evidence/web/` | Web security — headers, fingerprinting, AppSec observations |
| `evidence/incident/` | Incident thinking — timeline, response notes, case assessment |

### 2. Pemetaan Aset dan Kontrol

```bash
cat > ~/mod5/evidence/principles/asset_control_map.md <<'EOF'
# Asset and Control Map

## Inventaris Aset

| Aset | Tipe | Nilai | Lokasi |
|------|------|-------|--------|
| Ubuntu Lab | Host | Tinggi | 10.10.10.21 |
| Kali Linux | Host | Tinggi | 10.10.10.22 |
| Juice Shop (`:3000`) | Aplikasi web | Sedang | `127.0.0.1:3000` |
| DVWA (`:8081`) | Aplikasi web | Sedang | `127.0.0.1:8081` |
| Shared venv & tooling | Resource | Sedang | `/opt/jccsah/venvs/` |
| Datasets (log, pcap, phishing) | Data | Tinggi | `/opt/jccsah/datasets/` |
| Student credentials | Identitas | Kritis | Akun OS student |
| Evidence & reports | Data | Tinggi | `~/mod5/` |

## Area Bernilai Tinggi
- **Student credentials** — kompromi akun memberi akses ke seluruh workspace
- **Shared venv dan tooling** — modifikasi dapat memengaruhi seluruh student
- **Local target access path** — jalur akses ke target latihan yang harus tetap terkontrol
- **Evidence dan reports** — integritas harus dijaga untuk nilai forensik

## Kontrol yang Sudah Ada

| Kontrol | Implementasi | Tujuan |
|---------|-------------|--------|
| Non-sudo student account | Konfigurasi OS | Membatasi dampak kompromi akun |
| Localhost binding web targets | Network config | Mencegah eksposur ke jaringan |
| Shared datasets di lokasi terkontrol | Filesystem permission | Konsistensi dan auditabilitas |
| Pemisahan peran Ubuntu-Kali | Prosedur kerja | Mengurangi kebingungan scope |
EOF
cat ~/mod5/evidence/principles/asset_control_map.md
```

**Mengapa pemetaan aset dilakukan sebelum analisis ancaman:**

Tanpa pemetaan aset yang jelas, analisis ancaman menjadi tidak terarah. Mengetahui bahwa "student credentials" adalah aset berkritikalitas tinggi — bukan sekadar "ada akun user" — mengubah cara analis mengevaluasi ancaman: brute force SSH bukan hanya "serangan jaringan" tetapi ancaman langsung terhadap aset berkritikalitas tinggi yang perlu diprioritaskan penanganannya.

### 3. CIA Triad dalam Konteks Lab

```bash
cat > ~/mod5/evidence/principles/cia_notes.md <<'EOF'
# CIA Triad — Konteks Lab

## Confidentiality (Kerahasiaan)
**Yang dilindungi:** home directory student, credentials, dan dataset sensitif.
**Kontrol yang ada:** mode `drwx------` pada home directory, akun non-sudo.
**Gap yang teridentifikasi:** shared host berarti proses sistem dapat dilihat oleh seluruh user
melalui `ps` — tidak semua aktivitas benar-benar tersembunyi.

## Integrity (Integritas)
**Yang dilindungi:** file evidence, log sample, dan report yang dikumpulkan.
**Kontrol yang ada:** SHA256 checksum untuk verifikasi evidence.
**Gap yang teridentifikasi:** tidak ada immutable logging — evidence dapat dimodifikasi
tanpa mekanisme deteksi otomatis jika checksum tidak diperiksa secara berkala.

## Availability (Ketersediaan)
**Yang dilindungi:** shared lab resources (venv, datasets, tooling) harus tetap dapat
digunakan oleh seluruh student.
**Kontrol yang ada:** resource sharing yang dikelola administrator.
**Gap yang teridentifikasi:** shared host rentan terhadap resource exhaustion jika satu
student menjalankan proses yang terlalu berat.
EOF
```

**Nilai CIA triad dalam evaluasi kontrol:**

CIA triad bukan hanya kerangka klasifikasi — ini adalah alat evaluasi. Dengan memetakan setiap aset ke dimensi CIA yang paling relevan, analis dapat dengan cepat mengidentifikasi kontrol mana yang melindungi dimensi mana, dan dimensi mana yang tidak memiliki kontrol yang memadai (*control gap*).

### 4. Defense in Depth dalam Konteks Lab

```bash
cat > ~/mod5/evidence/principles/defense_in_depth_notes.md <<'EOF'
# Defense in Depth — Implementasi dalam Lab

## Layer yang Teridentifikasi

| Layer | Implementasi | Dampak jika Dikompromikan |
|-------|-------------|--------------------------|
| Account separation | Non-sudo student account | Kompromi terbatas pada user-space |
| Network binding | Localhost-only web targets | Tidak terekspos ke jaringan lab |
| Environment isolation | Shared venv administrator-managed | Konsistensi library, tidak dapat dimodifikasi student |
| Dataset isolation | Dataset di lokasi terkontrol | Artefak latihan tidak terpengaruh aktivitas student |
| Role separation | Ubuntu analisis, Kali validasi | Mencegah percampuran scope yang berbahaya |

## Prinsip yang Diterapkan
- **Jika satu layer gagal, layer berikutnya masih memberikan perlindungan**
- Kompromi akun student tidak langsung memberikan akses root (non-sudo)
- Kompromi akses jaringan tidak langsung memberikan akses ke web targets (localhost binding)

## Keterbatasan Defense in Depth Saat Ini
- Tidak ada network segmentation antar student (shared host)
- Tidak ada log terpusat yang independen dari host
- Tidak ada alerting otomatis untuk perilaku anomali
EOF
```

## Hasil dan Output

Tiga file yang dihasilkan — `asset_control_map.md`, `cia_notes.md`, dan `defense_in_depth_notes.md` — membentuk *security baseline document* yang menjadi referensi untuk seluruh analisis berikutnya dalam modul ini. Setiap temuan ancaman, setiap review identitas, dan setiap observasi web security akan dirujuk kembali ke baseline ini.

## Temuan Utama

Pemetaan aset mengungkap bahwa "student credentials" adalah aset berkritikalitas tertinggi dalam environment ini — kompromi akun student memberikan akses ke workspace, evidence, dan potensi pivot ke resource shared. Namun, kontrol yang ada (non-sudo, localhost binding, pemisahan peran) sudah membentuk beberapa lapisan defense in depth yang efektif untuk lingkungan latihan.

Gap yang paling signifikan adalah tidak adanya immutable logging dan alerting otomatis — kelemahan yang umum dalam lingkungan pendidikan namun perlu dipahami sebagai area yang perlu diperkuat dalam deployment produksi nyata.

## Apa yang Dipelajari

*Security reasoning* yang efektif dimulai dari pemahaman tentang apa yang perlu dilindungi, bukan dari pengetahuan tentang semua ancaman yang ada. Dengan baseline aset dan kontrol yang jelas, analis dapat membuat keputusan prioritas yang rasional — bukan reaktif terhadap setiap ancaman yang disebutkan dalam laporan industri.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `asset_control_map.md` | `~/mod5/evidence/principles/` | Inventaris aset, area bernilai tinggi, dan kontrol yang ada |
| `cia_notes.md` | `~/mod5/evidence/principles/` | CIA triad dalam konteks lab dengan gap yang teridentifikasi |
| `defense_in_depth_notes.md` | `~/mod5/evidence/principles/` | Layer defense in depth dan keterbatasannya |

---
---

# Proyek 2 — Menganalisis Threat Landscape dan Attack Surface Berdasarkan Artefak Nyata

## Latar Belakang

*Threat landscape* adalah gambaran komprehensif tentang ancaman yang relevan terhadap sebuah environment — bukan daftar semua jenis serangan yang pernah ada, melainkan ancaman yang memiliki *capability*, *opportunity*, dan *intent* untuk memengaruhi aset spesifik dalam konteks yang sedang dinilai. Analis yang membaca threat landscape dari artefak nyata — bukan dari abstraksi — memiliki kemampuan penilaian risiko yang jauh lebih akurat.

Proyek ini menganalisis ancaman langsung dari dua sumber konkret: sampel email phishing yang mencerminkan *social engineering threat* yang paling umum, dan dataset OSINT yang memberikan gambaran tentang informasi yang dapat dikumpulkan tentang suatu target. Hasil analisis kemudian digunakan untuk mendokumentasikan *attack surface* yang spesifik terhadap environment lab.

## Tujuan

- Mengekstrak indikator ancaman konkret dari artefak phishing dan dataset OSINT menggunakan teknik filtering yang tepat
- Menghubungkan temuan artefak dengan jenis ancaman yang relevan dalam konteks environment lab
- Mendokumentasikan attack surface berdasarkan observasi teknis, bukan asumsi generik
- Membiasakan pola berpikir bahwa ancaman harus dinilai dalam konteks aset spesifik, bukan secara abstrak

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Analisis artefak phishing dan OSINT, dokumentasi attack surface |

## Langkah-Langkah yang Dilakukan

### 1. Review Artefak Phishing dan Ekstraksi Threat Indicator

```bash
cat ~/datasets/phishing/sample_email_1.txt | tee ~/mod5/evidence/threats/phishing_findings.txt

grep -RinE 'urgent|verify|vpn|password|login' \
    ~/datasets/phishing ~/datasets/osint 2>/dev/null | \
    tee ~/mod5/evidence/threats/threat_review.txt
```

**Mengapa keyword-based review adalah langkah pertama yang tepat:**

Sebelum melakukan analisis mendalam terhadap artefak, review berbasis keyword memungkinkan analis untuk dengan cepat mengidentifikasi artefak mana yang mengandung sinyal yang relevan dan mana yang dapat dikesampingkan. Ini adalah teknik *triage* — bukan analisis final.

**Keyword yang dicari dan relevansinya:**

| Keyword | Mengapa Relevan dalam Konteks Phishing |
|---------|----------------------------------------|
| `urgent` | Menciptakan tekanan waktu yang mendorong tindakan impulsif |
| `verify` | Memancing korban untuk "memverifikasi" informasi melalui saluran palsu |
| `vpn` | Sering digunakan dalam spear phishing yang menarget karyawan IT |
| `password` | Indikasi permintaan kredensial yang tidak seharusnya ada dalam komunikasi resmi |
| `login` | Mengarahkan ke halaman login palsu (*credential harvesting*) |

**Teknik analisis mendalam setelah keyword review:**

Setelah keyword review mengidentifikasi artefak yang relevan, analisis mendalam pada `phishing_findings.txt` mencakup:

| Dimensi Analisis | Yang Dicari | Mengapa Penting |
|-----------------|-------------|-----------------|
| **Pengirim** | Ketidakcocokan antara nama tampilan dan domain email | Spoofing identity |
| **Subject line** | Kata yang menciptakan urgensi atau otoritas palsu | Social engineering trigger |
| **Body content** | Kombinasi urgency + authority + request | Triple-threat pattern |
| **Links** | Domain yang tidak cocok dengan klaim pengirim | Phishing redirect |
| **Call to action** | Permintaan yang tidak akan pernah dibuat oleh organisasi legitimate | Credential harvesting |

### 2. Dokumentasi Attack Surface

```bash
cat > ~/mod5/evidence/threats/attack_surface_notes.md <<'EOF'
# Attack Surface — Environment Lab

## Permukaan yang Dapat Diobservasi dari Luar

| Surface | Aksesibel dari | Tingkat Risiko |
|---------|---------------|----------------|
| SSH (port 22) | Jaringan lab | Sedang — credential-based auth |
| Web targets (3000, 8081) | Localhost only | Rendah — tidak terekspos jaringan |
| Shared dataset access | Pengguna dengan akun valid | Rendah — read-only untuk student |
| User-space tooling | Akun student aktif | Rendah — non-sudo |

## Faktor Risiko Kunci

### Credential Misuse
Akun student yang dikompromikan memberikan akses ke seluruh workspace dan potensi
pivot ke resource shared. Vektor utama: phishing, brute force SSH, credential reuse.

### Weak Review of Phishing-like Content
Student yang tidak terlatih mengenali social engineering dapat menjadi vektor kompromi
melalui email phishing atau manipulasi sosial lainnya.

### Over-broad Trust in Local Tools or Data
Mengasumsikan bahwa tool atau data yang tersedia di lab selalu aman dan tidak dimodifikasi
adalah asumsi yang berbahaya dalam lingkungan shared.

### Shared Resource Exhaustion
Proses yang terlalu berat dari satu student dapat memengaruhi ketersediaan resource
untuk seluruh pengguna shared host — denial of service yang tidak disengaja.

## Attack Path yang Paling Realistis (dalam konteks lab)
1. Phishing → kompromi credential → SSH access → akses workspace student
2. Brute force SSH → akses langsung tanpa perlu phishing
3. Credential reuse → akun yang sama dipakai di sistem lain yang lebih lemah

## Yang TIDAK Menjadi Attack Surface (karena kontrol yang ada)
- Web targets tidak dapat dijangkau dari luar host (localhost binding)
- Eskalasi privilege ke root terbatas oleh non-sudo account
- Modifikasi tooling tidak dapat dilakukan tanpa akses administrator
EOF
cat ~/mod5/evidence/threats/attack_surface_notes.md
```

**Perbedaan antara attack surface dan threat:**

*Attack surface* adalah totalitas titik-titik di mana sistem dapat diserang. *Threat* adalah kemungkinan kejadian yang memanfaatkan bagian dari attack surface tersebut. Memahami attack surface membantu analis memprioritaskan kontrol: mempersempit attack surface (misalnya dengan localhost binding) lebih efektif daripada mencoba mencegah semua kemungkinan ancaman.

## Hasil dan Output

**Contoh isi `phishing_findings.txt` (potongan):**
```
From: security-team@urgent-verify.com
To: employee@company.com
Subject: URGENT: Verify Your VPN Credentials Immediately

Dear User,

Your VPN access will be suspended within 24 hours unless you verify your login
credentials at the link below. This is a mandatory security compliance requirement.
```

**Contoh isi `threat_review.txt`:**
```
phishing/sample_email_1.txt:1:URGENT: Verify Your VPN Credentials
phishing/sample_email_1.txt:5:verify your login credentials
phishing/sample_email_1.txt:7:password reset required
```

## Temuan Utama

Analisis phishing sample mengungkap penggunaan *triple-threat pattern* yang klasik: urgensi waktu ("24 hours"), otoritas yang dipalsukan ("security compliance requirement"), dan permintaan kredensial eksplisit ("verify your login credentials"). Kombinasi ketiganya adalah pola yang terbukti efektif dalam melewati kewaspadaan korban yang tidak terlatih.

Dokumentasi attack surface mengidentifikasi SSH sebagai satu-satunya *surface* yang benar-benar terekspos ke jaringan. Semua surface lainnya sudah dikontrol melalui localhost binding atau pemisahan akun. Ini berarti strategi proteksi harus diprioritaskan pada pengamanan SSH (key-based auth, fail2ban, rate limiting) dan kesadaran phishing.

## Apa yang Dipelajari

Membaca ancaman dari artefak nyata — bukan dari daftar teoritis — menghasilkan penilaian risiko yang jauh lebih akurat dan dapat ditindaklanjuti. Phishing sample yang dianalisis secara sistematis mengungkap pola yang dapat langsung digunakan sebagai bahan pelatihan *security awareness*, sementara dokumentasi attack surface yang spesifik memberikan panduan prioritas kontrol yang jelas.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `phishing_findings.txt` | `~/mod5/evidence/threats/` | Isi lengkap phishing sample untuk analisis |
| `threat_review.txt` | `~/mod5/evidence/threats/` | Hasil grep keyword dari phishing dan OSINT dataset |
| `attack_surface_notes.md` | `~/mod5/evidence/threats/` | Dokumentasi attack surface spesifik environment lab |

---
---

# Proyek 3 — Mereview Konteks Identitas, Session, dan Least Privilege

## Latar Belakang

Identity and Access Management (IAM) adalah fondasi dari hampir seluruh model keamanan modern. Sebagian besar insiden keamanan yang berhasil — baik yang dilakukan oleh penyerang eksternal maupun *insider threat* — pada akhirnya melibatkan penggunaan identitas yang tidak sah atau penyalahgunaan hak akses yang berlebihan. Konfigurasi "least privilege" bukan hanya best practice — ini adalah *blast radius reducer*: membatasi seberapa jauh kerusakan yang dapat ditimbulkan jika sebuah akun dikompromikan.

Proyek ini membangun kebiasaan membaca dan mengevaluasi konteks identitas secara sistematis: siapa yang sedang bekerja, hak akses apa yang dimiliki, dari mana mereka terhubung, dan apakah konfigurasi privilege sudah sesuai dengan prinsip least privilege. Semua pertanyaan ini dimulai dari perintah sederhana yang hasilnya seringkali lebih informatif dari yang disadari.

## Tujuan

- Membaca konteks identitas lengkap termasuk UID, GID, dan seluruh keanggotaan grup
- Mereview session aktif dan informasi login untuk mendeteksi potensi akses yang tidak sah
- Menerjemahkan konfigurasi least privilege dari model lab ke prinsip yang berlaku di lingkungan produksi
- Memahami mengapa kontrol berbasis identitas adalah kontrol pertama yang harus dievaluasi dalam setiap security review

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Review identitas dan session dilakukan di sini |

## Langkah-Langkah yang Dilakukan

### 1. Review Identity Context

```bash
whoami | tee  ~/mod5/evidence/iam/iam_context.txt
id      | tee -a ~/mod5/evidence/iam/iam_context.txt
groups  | tee -a ~/mod5/evidence/iam/iam_context.txt
```

**Mengapa `id` adalah perintah paling informatif untuk IAM review:**

`id` menampilkan informasi yang tidak dapat dilihat dari `whoami` saja:

```
uid=1001(student) gid=1001(student) groups=1001(student),4(adm),27(sudo),998(docker)
```

Setiap grup yang tercantum memiliki implikasi keamanan yang berbeda:

| Grup | Implikasi Keamanan |
|------|-------------------|
| `adm` | Akses baca ke log sistem (`/var/log/`) — bermanfaat untuk analisis, tapi juga mengekspos informasi sistem |
| `sudo` | Dapat menjalankan perintah sebagai root — **sangat signifikan** dalam security review |
| `docker` | Akses ke Docker daemon — **efektif setara root** via container privilege escalation |
| `www-data` | Akses ke web server files — relevan jika ada web server berjalan |

**Red flags dalam output `id` yang perlu segera ditindaklanjuti:**

Jika `id` menunjukkan keanggotaan `sudo` atau `docker` untuk akun yang seharusnya non-privileged, ini adalah temuan yang harus didokumentasikan dan dilaporkan karena memberikan hak akses yang jauh melampaui kebutuhan operasional normal.

### 2. Review Session Aktif

```bash
w   | tee  ~/mod5/evidence/iam/session_access_review.txt
who | tee -a ~/mod5/evidence/iam/session_access_review.txt
```

**Informasi yang dapat diekstrak dari review session:**

**Dari `w`:**
```
USER     TTY      FROM             LOGIN@   IDLE JCPU PCPU WHAT
student  pts/0    10.10.10.5       10:00    0.00s 0.05s 0.01s w
alice    pts/1    10.10.10.99      09:45    15:23 0.12s 0.12s bash
```

Analisis baris per baris:
- `student` dari `10.10.10.5` dengan idle time `0.00s` — sedang aktif bekerja, IP konsisten dengan lab
- `alice` dari `10.10.10.99` dengan idle time `15:23` — tidak aktif selama 15 menit dari IP yang perlu diverifikasi

**Pertanyaan yang harus dijawab dari review session:**

| Pertanyaan | Relevansi Keamanan |
|------------|-------------------|
| Apakah semua user yang login dikenal? | Mendeteksi unauthorized access |
| Apakah IP asal konsisten dengan lokasi yang diharapkan? | Mendeteksi akses dari lokasi anomali |
| Apakah ada session yang aktif di luar jam kerja? | Mendeteksi after-hours access |
| Apakah ada multiple session dari user yang sama? | Potensi credential sharing atau akses paralel |

### 3. Dokumentasi Least Privilege

```bash
cat > ~/mod5/evidence/iam/least_privilege_notes.md <<'EOF'
# Least Privilege — Implementasi dan Implikasi

## Definisi Operasional
Least privilege berarti setiap entitas (user, proses, service) hanya memiliki hak akses
yang minimum yang diperlukan untuk menjalankan fungsinya — tidak lebih.

## Implementasi dalam Lab

| Kontrol | Implementasi | Prinsip yang Diterapkan |
|---------|-------------|------------------------|
| Non-sudo student account | Akun tidak dapat `sudo` | Hak akses sesuai kebutuhan operasional |
| Localhost-only web targets | Target tidak terekspos jaringan | Akses dibatasi ke yang membutuhkan |
| Read-only dataset access | Student tidak dapat mengubah dataset | Integritas resource shared terjaga |
| Pemisahan Ubuntu-Kali | Tool berbeda di host berbeda | Scope kerja dibatasi sesuai peran |

## Mengapa Least Privilege Mengurangi Blast Radius

Jika akun student dikompromikan:
- Tanpa least privilege: penyerang memiliki root access → seluruh sistem terancam
- Dengan least privilege (non-sudo): penyerang terbatas pada user-space → dampak terbatas

## Kesenjangan (Gap) yang Teridentifikasi
- Shared host tanpa container isolation berarti proses sistem masih dapat dilihat antar user
- Tidak ada time-based access control (jam akses tidak dibatasi)
- Tidak ada MFA untuk SSH — password-based auth masih memungkinkan brute force

## Rekomendasi untuk Lingkungan Produksi
- Implementasikan key-based SSH authentication (nonaktifkan password auth)
- Pertimbangkan implementasi MFA untuk akses SSH
- Audit keanggotaan grup secara berkala — grup `docker` harus dievaluasi ulang
- Implementasikan session timeout untuk terminal idle
EOF
cat ~/mod5/evidence/iam/least_privilege_notes.md
```

## Temuan Utama

Review `id` mengkonfirmasi bahwa akun student tidak memiliki akses `sudo` — kontrol yang sudah benar dan konsisten dengan prinsip least privilege. Namun, jika keanggotaan `docker` ditemukan, ini adalah temuan yang perlu diescalate karena grup `docker` memberikan kemampuan privilege escalation ke root melalui container.

Review session dengan `w` mengungkap bahwa seluruh session aktif berasal dari IP dalam subnet lab — tidak ada akses dari IP eksternal yang mencurigakan. Ini adalah baseline yang perlu dipantau secara berkala.

## Apa yang Dipelajari

IAM bukan hanya konsep tentang SSO, MFA, atau LDAP — ini dimulai dari pemahaman siapa yang sedang aktif di sistem saat ini, apa yang bisa mereka lakukan, dan dari mana mereka terhubung. Perintah sederhana seperti `id`, `w`, dan `who` memberikan gambaran IAM yang sangat informatif dalam hitungan detik, dan menjadi titik awal yang wajib dalam setiap security review atau incident investigation.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `iam_context.txt` | `~/mod5/evidence/iam/` | Output `whoami`, `id`, dan `groups` |
| `session_access_review.txt` | `~/mod5/evidence/iam/` | Output `w` dan `who` — session aktif |
| `least_privilege_notes.md` | `~/mod5/evidence/iam/` | Dokumentasi implementasi dan gap least privilege |

---
---

# Proyek 4 — Memverifikasi Integritas File dan Meninjau Bukti Kriptografis

## Latar Belakang

Kriptografi dalam konteks operasi keamanan bukan tentang implementasi algoritma enkripsi — ini tentang menggunakan *output* kriptografi sebagai alat kerja yang konkret: hash untuk verifikasi integritas, sertifikat untuk membangun kepercayaan, dan metadata TLS untuk menilai konfigurasi keamanan. Analis yang hanya memahami kriptografi secara teoritis namun tidak dapat membaca output `sha256sum` atau `openssl x509` dengan percaya diri belum memiliki kompetensi operasional yang lengkap.

Proyek ini membangun kemampuan *applied cryptography* yang langsung dapat digunakan dalam operasi keamanan: menggunakan SHA256 untuk verifikasi integritas evidence, membaca metadata sertifikat TLS untuk menilai konfigurasi kepercayaan, dan yang terpenting — mengetahui apa yang dapat dan tidak dapat dibuktikan oleh setiap jenis bukti kriptografis.

## Tujuan

- Menggunakan `sha256sum` untuk menghasilkan dan mendokumentasikan hash dari multiple artefak sebagai evidence integrity baseline
- Membaca metadata sertifikat TLS menggunakan `openssl` dan menginterpretasikan informasi yang dikandungnya
- Mendokumentasikan batasan dari setiap jenis bukti kriptografis — apa yang dapat dibuktikan dan apa yang tidak
- Membiasakan pola kerja di mana integritas evidence selalu diverifikasi sebelum dijadikan basis kesimpulan

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Hash verification untuk artefak lokal |
| **Kali Linux** | Certificate review menggunakan `openssl` terhadap target eksternal |

## Langkah-Langkah yang Dilakukan

### 1. Hash Verification untuk Multiple Artefak

```bash
sha256sum ~/datasets/phishing/sample_email_1.txt | tee  ~/mod5/evidence/crypto/hash_verification.txt
sha256sum ~/datasets/logs/linux/auth.log.sample   | tee -a ~/mod5/evidence/crypto/hash_verification.txt
sha256sum ~/datasets/pcap/localhost_web_lab.pcap  | tee -a ~/mod5/evidence/crypto/hash_verification.txt || true
```

**Mengapa tiga artefak berbeda dipilih:**

Memverifikasi hash pada artefak dengan tipe yang berbeda (email sample, log file, PCAP) menunjukkan bahwa hash verification adalah teknik yang agnostik terhadap tipe file — ia bekerja identik pada semua jenis file. Ini penting untuk membangun intuisi bahwa hash adalah *universal identifier* untuk konten file, terlepas dari formatnya.

**Cara menggunakan hash verification dalam praktik forensik:**

```bash
# Langkah 1: Catat hash saat evidence pertama kali dikumpulkan
sha256sum evidence_file.txt > evidence_file.txt.sha256

# Langkah 2: Setelah transfer atau setelah investigasi selesai, verifikasi
sha256sum -c evidence_file.txt.sha256
# Output yang diharapkan: evidence_file.txt: OK
# Output yang mengindikasikan masalah: evidence_file.txt: FAILED
```

**Apa yang hash DAPAT dan TIDAK DAPAT buktikan:**

| Dapat Dibuktikan | Tidak Dapat Dibuktikan |
|-----------------|------------------------|
| File tidak berubah sejak hash dibuat | File belum pernah dimodifikasi sebelum hash dibuat |
| Dua file dengan hash yang sama memiliki konten identik | Kapan file terakhir diakses |
| File yang ditransfer identik dengan file asli | Siapa yang membuat atau memodifikasi file |
| Integritas file tidak terganggu | Apakah konten file adalah bukti yang valid secara hukum |

### 2. Certificate Review dengan OpenSSL

```bash
echo | openssl s_client -connect example.com:443 \
    -servername example.com 2>/dev/null | \
    openssl x509 -noout -subject -issuer -dates -fingerprint | \
    tee ~/mod5/evidence/crypto/certificate_review.txt
```

**Penjelasan pipeline OpenSSL:**

| Komponen | Fungsi |
|----------|--------|
| `echo \|` | Mengirim input kosong untuk langsung menutup koneksi setelah handshake |
| `openssl s_client -connect example.com:443` | Membuat koneksi TLS dan mengambil sertifikat |
| `-servername example.com` | SNI (Server Name Indication) — penting untuk server yang host multiple domain |
| `2>/dev/null` | Sembunyikan diagnostic output dari s_client |
| `openssl x509 -noout` | Parse sertifikat (dari stdin) tanpa menampilkan raw data |
| `-subject` | Tampilkan subject (CN, O, C) |
| `-issuer` | Tampilkan issuer (Certificate Authority) |
| `-dates` | Tampilkan `notBefore` dan `notAfter` (masa berlaku) |
| `-fingerprint` | Tampilkan fingerprint SHA1 dari sertifikat |

**Cara membaca dan menginterpretasikan output:**

```
subject=CN = www.example.org, O = Internet Corporation for Assigned Names and Numbers, C = US
issuer=CN = DigiCert TLS RSA SHA256 2020 CA1, O = DigiCert Inc, C = US
notBefore=Jan 13 00:00:00 2023 GMT
notAfter=Feb 13 23:59:59 2024 GMT
SHA1 Fingerprint=A0:B8:56:73:...
```

**Analisis setiap field:**

| Field | Nilai Contoh | Interpretasi |
|-------|-------------|--------------|
| `subject CN` | `www.example.org` | Domain yang disertifikasi — harus cocok dengan domain yang diakses |
| `subject O` | `Internet Corporation...` | Organisasi pemilik sertifikat |
| `issuer CN` | `DigiCert TLS RSA SHA256 2020 CA1` | Certificate Authority yang menerbitkan |
| `notBefore` | `Jan 13 00:00:00 2023` | Sertifikat belum valid sebelum tanggal ini |
| `notAfter` | `Feb 13 23:59:59 2024` | Sertifikat tidak valid setelah tanggal ini |
| `Fingerprint` | `A0:B8:56:73:...` | Identitas unik sertifikat ini — dapat diverifikasi dengan pihak lain |

**Red flags dalam sertifikat:**

| Kondisi | Implikasi |
|---------|-----------|
| `notAfter` sudah lewat | Sertifikat expired — TLS tidak dapat dipercaya |
| `subject CN` tidak cocok domain | Certificate mismatch — kemungkinan MITM atau konfigurasi salah |
| Issuer tidak dikenal | Self-signed atau untrusted CA — kepercayaan tidak dapat diverifikasi |
| Fingerprint berbeda dari yang diharapkan | Kemungkinan sertifikat telah diganti |

### 3. Catatan Operasional Cryptographic Evidence

```bash
cat > ~/mod5/evidence/crypto/openssl_notes.txt <<'EOF'
# Crypto Notes — Implikasi Operasional

## SHA256 Hash
- Membuktikan: konten file tidak berubah sejak hash dibuat
- Tidak membuktikan: file belum pernah dimodifikasi sebelum hash dibuat
- Penggunaan: chain of custody, evidence integrity, deduplication, malware identification

## Sertifikat TLS
- Subject: siapa yang diklaim memiliki domain ini
- Issuer: CA yang menjamin klaim tersebut
- Validity dates: masa berlaku jaminan tersebut
- Fingerprint: identitas unik sertifikat ini

## Apa yang HTTPS / TLS TIDAK menjamin
- Keamanan konten aplikasi di balik HTTPS
- Tidak adanya kerentanan di sisi server
- Legitimasi organisasi di balik domain
- Keamanan data setelah sampai di server

## Implikasi untuk Security Assessment
HTTPS hanya membuktikan bahwa komunikasi antara browser dan server terenkripsi.
Aplikasi yang berjalan di belakang HTTPS masih dapat memiliki SQL injection, XSS,
broken authentication, atau kerentanan lainnya. TLS adalah transport security,
bukan application security.
EOF
cat ~/mod5/evidence/crypto/openssl_notes.txt
```

## Hasil dan Output

**Contoh isi `hash_verification.txt`:**
```
a3f92c1d4b8e7f2a1c9d0e5b6f3a8c2d...  /home/student/datasets/phishing/sample_email_1.txt
8b2e4f9c1a7d3e6b0f5c8a2d4e7f1b9c...  /home/student/datasets/logs/linux/auth.log.sample
5c9f2a8d1e4b7c0f3a6d9e2f5b8c1a4d...  /home/student/datasets/pcap/localhost_web_lab.pcap
```

**Contoh isi `certificate_review.txt`:**
```
subject=CN = www.example.org, O = Internet Corporation for Assigned Names and Numbers, C = US
issuer=CN = DigiCert TLS RSA SHA256 2020 CA1, O = DigiCert Inc, C = US
notBefore=Jan 13 00:00:00 2023 GMT
notAfter=Feb 13 23:59:59 2024 GMT
SHA1 Fingerprint=A0:B8:56:73:8B:93:AA:3E:7A:88:1A:5A:88:86:FA:FF:AB:62:6B:61
```

## Temuan Utama

Review hash menghasilkan baseline integritas untuk tiga artefak utama yang digunakan sepanjang modul. Nilai hash ini akan menjadi referensi: jika di kemudian hari hash artefak yang sama berbeda dari yang tercatat di sini, ini adalah indikator bahwa file telah dimodifikasi.

Review sertifikat mengungkap bahwa `example.com` menggunakan sertifikat yang diterbitkan oleh `DigiCert` — CA yang diakui secara global. Ini mengkonfirmasi validitas trust chain. Catatan penting dari `openssl_notes.txt`: keberadaan sertifikat valid ini tidak membuktikan bahwa aplikasi yang berjalan di domain tersebut aman dari kerentanan aplikasi.

## Apa yang Dipelajari

Kriptografi dalam operasi keamanan bukan tentang implementasi — ini tentang interpretasi. SHA256 hash yang dihasilkan dalam hitungan detik memberikan bukti integritas yang jauh lebih kuat dari "saya yakin file ini tidak berubah". Sertifikat TLS yang dibaca dengan `openssl` memberikan informasi kepercayaan yang tidak terlihat dari browser. Dan yang paling penting: memahami *batas* dari setiap bukti kriptografis mencegah analis dari membuat klaim yang lebih luas dari yang dapat dibuktikan.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `hash_verification.txt` | `~/mod5/evidence/crypto/` | SHA256 hash dari tiga artefak utama |
| `certificate_review.txt` | `~/mod5/evidence/crypto/` | Metadata sertifikat TLS `example.com` |
| `openssl_notes.txt` | `~/mod5/evidence/crypto/` | Catatan operasional tentang implikasi cryptographic evidence |

---
---

# Proyek 5 — Memvalidasi Security Header dan Eksposur Aplikasi Web

## Latar Belakang

Keamanan aplikasi web sering kali dinilai dari dua ekstrem yang sama-sama tidak akurat: "aman karena menggunakan HTTPS" atau "tidak aman karena ditemukan kerentanan X". Realitasnya jauh lebih nuansif. Security headers memberikan *sinyal* tentang kontrol yang dikonfigurasi, namun kehadiran header saja tidak menjamin bahwa seluruh aspek keamanan aplikasi sudah ditangani dengan benar.

Proyek ini memvalidasi web application security dari dua perspektif yang berbeda: perspektif internal (Ubuntu) dan perspektif assessment (Kali), menggunakan local targets sebagai objek validasi yang aman dan terkontrol. Fokusnya adalah membangun kemampuan membaca dan menginterpretasikan bukti keamanan aplikasi secara proporsional — tidak terlalu optimistis, tidak terlalu pesimistis.

## Tujuan

- Mengumpulkan dan menginterpretasikan security headers dari local targets menggunakan `curl -I`
- Menggunakan `whatweb` untuk fingerprinting ringan yang mengungkap teknologi di balik aplikasi
- Mendokumentasikan observasi AppSec yang proporsional — berdasarkan apa yang terlihat, bukan asumsi
- Memahami batasan header-based assessment dan kapan analisis yang lebih dalam diperlukan

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Kali Linux** | Validasi security headers dan fingerprinting dari perspektif assessment |
| **Ubuntu Lab** | Dokumentasi appsec observations |

## Langkah-Langkah yang Dilakukan

### 1. Validasi Security Headers dengan curl

```bash
curl -I http://127.0.0.1:3000 | tee  ~/mod5/evidence/web/http_security_headers.txt
curl -I http://127.0.0.1:8081 | tee -a ~/mod5/evidence/web/http_security_headers.txt
```

**Security headers yang diperiksa dan implikasinya:**

| Header | Jika Ada | Jika Tidak Ada |
|--------|----------|----------------|
| `Content-Security-Policy` | Mencegah XSS dengan membatasi sumber konten | Aplikasi mungkin rentan terhadap XSS |
| `X-Frame-Options` | Mencegah clickjacking | Halaman dapat di-embed dalam iframe oleh pihak lain |
| `Strict-Transport-Security` | Memaksa HTTPS untuk koneksi berikutnya | Downgrade attack ke HTTP dimungkinkan |
| `X-Content-Type-Options: nosniff` | Mencegah MIME type sniffing | Browser mungkin menginterpretasikan konten secara salah |
| `Referrer-Policy` | Mengontrol informasi yang dikirim dalam Referer header | Kebocoran URL internal ke domain eksternal |
| `Permissions-Policy` | Mengontrol akses ke browser features (kamera, mikrofon) | Over-permissive browser API access |

**Header yang mengindikasikan informasi (perlu diperhatikan):**

| Header | Contoh | Implikasi |
|--------|--------|-----------|
| `Server` | `nginx/1.18.0` | Mengekspos versi software — target untuk version-specific exploits |
| `X-Powered-By` | `Express` | Mengekspos framework backend |
| `Set-Cookie` tanpa `HttpOnly` | `session=abc123; Path=/` | Cookie dapat dibaca oleh JavaScript — rentan XSS cookie theft |
| `Set-Cookie` tanpa `Secure` | `session=abc123; Path=/; HttpOnly` | Cookie dikirim melalui HTTP — rentan MITM |

### 2. Fingerprinting dengan WhatWeb

```bash
whatweb http://127.0.0.1:3000 | tee  ~/mod5/evidence/web/local_target_review.txt || true
whatweb http://127.0.0.1:8081 | tee -a ~/mod5/evidence/web/local_target_review.txt || true
```

**Mengapa `|| true` digunakan:**

`|| true` memastikan bahwa jika `whatweb` tidak tersedia di Kali, perintah tidak menghasilkan error yang menghentikan eksekusi script. Ini adalah pola *graceful degradation* — jika tool tidak tersedia, catat absensinya dan lanjutkan dengan tool yang ada.

**Informasi yang dapat dikumpulkan `whatweb`:**

`whatweb` melakukan *passive fingerprinting* — menganalisis HTTP headers, HTML content, dan pola respons untuk mengidentifikasi:
- Framework web (Express, Django, Rails, Laravel)
- Web server (nginx, Apache, IIS)
- CMS yang digunakan (WordPress, Joomla, Drupal)
- Library JavaScript (jQuery versi tertentu, Bootstrap)
- Metainformasi lain yang terekspos dalam HTML

Semua informasi ini dapat digunakan oleh penyerang untuk mencari kerentanan yang spesifik terhadap teknologi yang teridentifikasi.

### 3. AppSec Observations yang Proporsional

```bash
cat > ~/mod5/evidence/web/appsec_observations.md <<'EOF'
# AppSec Observations — Berdasarkan Evidence

## Metodologi
Observasi ini dibatasi pada apa yang dapat dilihat dari header HTTP dan fingerprinting
ringan. Klaim keamanan atau ketidakamanan yang melampaui evidence yang ada harus
dihindari.

## Apa yang Terlihat (Observable)

### Port 3000 — Juice Shop (Node.js/Express)
- Status: merespons dengan 200 OK
- Framework: Node.js Express (dari X-Powered-By)
- Security headers: perlu diverifikasi dari output curl
- Server version: tidak diekspos (lebih baik dari mengekspos versi)

### Port 8081 — DVWA (nginx)
- Status: merespons dengan 200 OK
- Web server: nginx versi terekspos dalam header Server
- Security headers: perlu diverifikasi dari output curl

## Apa yang TIDAK Dapat Disimpulkan dari Header Saja

| Tidak Terlihat | Mengapa Penting |
|----------------|----------------|
| Kualitas autentikasi dan otorisasi | Header tidak mencerminkan logika auth di dalam aplikasi |
| Validasi input dan output | Header tidak menunjukkan apakah XSS/SQLi dimungkinkan |
| Keamanan session management | Sebagian terlihat dari cookie flags, tapi tidak lengkap |
| Keamanan konfigurasi backend | Tidak terlihat dari permukaan |
| Kerentanan logic business | Sama sekali tidak terlihat dari header |

## Pertanyaan untuk Analisis Lebih Dalam

1. Apakah cookie menggunakan `HttpOnly` dan `Secure` flag?
2. Apakah `Content-Security-Policy` dikonfigurasi dengan benar?
3. Apakah ada endpoint yang tidak memerlukan autentikasi yang seharusnya terlindungi?
4. Bagaimana aplikasi menangani input yang tidak valid?

## Kesimpulan Proporsional
Kedua aplikasi dapat dijangkau dari localhost dan merespons sesuai ekspektasi. Kehadiran
atau ketidakhadiran security header tertentu adalah sinyal, bukan bukti keamanan atau
ketidakamanan yang definitif. Analisis yang lebih mendalam memerlukan code review,
authenticated testing, atau penetration testing yang terkontrol.
EOF
cat ~/mod5/evidence/web/appsec_observations.md
```

## Hasil dan Output

**Contoh isi `http_security_headers.txt`:**
```
HTTP/1.1 200 OK
X-Powered-By: Express
X-Frame-Options: SAMEORIGIN
X-Content-Type-Options: nosniff
Content-Type: text/html; charset=utf-8

HTTP/1.1 200 OK
Server: nginx/1.18.0
Content-Type: text/html;charset=UTF-8
```

**Analisis perbandingan kedua target:**

| Header | Port 3000 (Juice Shop) | Port 8081 (DVWA) | Penilaian |
|--------|----------------------|-----------------|-----------|
| `X-Frame-Options` | `SAMEORIGIN` ✅ | Tidak ada ⚠️ | DVWA lebih rentan clickjacking |
| `X-Content-Type-Options` | `nosniff` ✅ | Tidak ada ⚠️ | DVWA lebih rentan MIME sniffing |
| `Server` version | Tidak diekspos ✅ | `nginx/1.18.0` ⚠️ | DVWA mengekspos versi |
| `Content-Security-Policy` | Tidak ada ⚠️ | Tidak ada ⚠️ | Kedua target tidak memiliki CSP |

## Temuan Utama

Perbandingan security headers antara Juice Shop dan DVWA mengungkap perbedaan yang signifikan meskipun keduanya adalah target latihan. Juice Shop mengimplementasikan beberapa security header dasar yang tidak ada di DVWA. Ini adalah pengingat bahwa kualitas keamanan aplikasi bervariasi bahkan di antara aplikasi yang dibuat untuk tujuan yang sama — dan bahwa header-based assessment dapat mengungkap perbedaan tersebut dengan cepat.

Ketiadaan `Content-Security-Policy` di kedua target adalah temuan yang konsisten dan relevan: tanpa CSP, aplikasi tidak memiliki mekanisme untuk mencegah eksekusi script yang tidak sah dari domain eksternal.

## Apa yang Dipelajari

*Maturity* dalam AppSec assessment berarti mengetahui batas dari setiap teknik yang digunakan. Header-based assessment adalah *fast and cheap* — dapat dilakukan dalam hitungan detik dan tidak memerlukan autentikasi. Namun batasannya jelas: banyak kerentanan aplikasi yang tidak dapat terdeteksi dari headers saja. Memahami batas ini dan mendokumentasikannya secara eksplisit adalah tanda profesionalisme dalam security assessment.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `http_security_headers.txt` | `~/mod5/evidence/web/` | HTTP headers dari kedua local targets |
| `local_target_review.txt` | `~/mod5/evidence/web/` | Output fingerprinting `whatweb` |
| `appsec_observations.md` | `~/mod5/evidence/web/` | Observasi AppSec yang proporsional berbasis evidence |

---
---

# Proyek 6 — Menyusun Incident Timeline dan Case Assessment Berorientasi Respons

## Latar Belakang

*Incident response* yang efektif bukan tentang bereaksi secepat mungkin — ini tentang bereaksi dengan *urutan yang benar*. Analis yang langsung mengubah konfigurasi sistem begitu menemukan anomali berisiko menghapus bukti, memperburuk situasi, atau membuat perubahan yang tidak dapat dipertanggungjawabkan. Sebaliknya, analis yang mengikuti urutan kerja yang terstruktur — preserve → analyze → contain → remediate — menghasilkan respons yang dapat diaudit, direproduksi, dan dipelajari.

Proyek ini membangun *incident thinking* — kemampuan untuk melihat serangkaian observasi teknis sebagai potensi incident, menyusunnya dalam timeline yang logis, dan mendokumentasikan prioritas respons yang benar. Ini adalah jembatan yang menghubungkan seluruh analisis teknis dari proyek-proyek sebelumnya menjadi pola pikir operasional yang siap dibawa ke modul blue team.

## Tujuan

- Menyusun timeline sederhana dari urutan observasi yang telah dilakukan sebagai simulasi incident timeline
- Mendokumentasikan prioritas respons yang benar — apa yang harus dilakukan pertama dan apa yang tidak boleh dilakukan terburu-buru
- Mengintegrasikan temuan dari seluruh proyek sebelumnya menjadi case assessment yang kohesif
- Membiasakan pola pikir bahwa incident response dimulai dari preservasi evidence, bukan dari tindakan

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Seluruh dokumentasi incident thinking dilakukan di sini |

## Langkah-Langkah yang Dilakukan

### 1. Pembangunan Incident Timeline

```bash
cat > ~/mod5/evidence/incident/incident_timeline.md <<'EOF'
# Incident Timeline — Modul 05

## Konteks
Timeline ini merekonstruksi urutan observasi yang dilakukan selama sesi analisis,
diformat sebagai simulasi incident timeline untuk membiasakan pola dokumentasi yang
digunakan dalam incident response nyata.

## Timeline Observasi

| Waktu | Aktivitas | Host | Temuan |
|-------|-----------|------|--------|
| T+00 | Workspace setup dan asset mapping | Ubuntu | Aset dan kontrol terdokumentasi |
| T+15 | Phishing sample review | Ubuntu | Triple-threat pattern teridentifikasi di email sample |
| T+25 | Attack surface mapping | Ubuntu | SSH sebagai satu-satunya surface yang terekspos ke jaringan |
| T+35 | Identity dan session review | Ubuntu | Akun non-sudo, session dari IP lab — normal |
| T+45 | Hash verification | Ubuntu | Baseline integritas tiga artefak terdokumentasi |
| T+55 | Certificate review | Kali | TLS valid, trust chain terverifikasi |
| T+65 | Web header validation | Kali | Security headers bervariasi antara dua target |
| T+75 | AppSec observations | Ubuntu | CSP tidak ada di kedua target, X-Frame-Options hanya di Juice Shop |
| T+85 | Incident assessment | Ubuntu | Control gaps dan risk themes terdokumentasi |

## Pola yang Teridentifikasi dari Timeline
1. Tidak ada tanda kompromi aktif yang terdeteksi dalam sesi ini
2. Control gaps yang ada bersifat konfigurasi, bukan indikasi serangan yang sedang berlangsung
3. Phishing sample mengandung IOC yang perlu dimasukkan ke threat intelligence

## Catatan Metodologi
Timeline yang terstruktur memungkinkan analis lain untuk mereproduksi investigasi ini,
mengidentifikasi langkah yang mungkin terlewat, dan memverifikasi temuan secara independen.
EOF
cat ~/mod5/evidence/incident/incident_timeline.md
```

**Mengapa format tabel digunakan dalam timeline:**

Timeline dalam format tabel memungkinkan pembaca untuk dengan cepat melihat urutan kronologis, host yang digunakan di setiap langkah, dan temuan yang dihasilkan. Dalam incident response nyata, timeline seperti ini yang pertama kali diminta oleh manajemen atau tim hukum untuk memahami apa yang terjadi dan kapan.

### 2. Response Notes — Prioritas yang Benar

```bash
cat > ~/mod5/evidence/incident/response_notes.md <<'EOF'
# Response Notes — Urutan Prioritas yang Benar

## PERTAMA: Preserve dan Dokumentasi
- Jangan mengubah apa pun sebelum evidence terdokumentasi
- Catat hash dari file yang relevan sebelum setiap analisis
- Dokumentasikan context host dan waktu dengan timestamp yang akurat
- Catat siapa yang melakukan apa dan kapan

## KEDUA: Confirm dan Validate
- Konfirmasi bahwa anomali yang terlihat adalah masalah nyata, bukan artifact analisis
- Validasi dari perspektif yang berbeda (Ubuntu vs Kali) sebelum menarik kesimpulan
- Bedakan antara: kondisi normal yang tidak familiar vs kondisi abnormal yang nyata

## KETIGA: Analyze Scope
- Tentukan apakah masalah terbatas pada satu host, satu user, atau lebih luas
- Identifikasi apakah ada indikasi lateral movement atau data exfiltration
- Jangan overclaim — batasi penilaian pada evidence yang ada

## KEEMPAT: Contain (jika diperlukan)
- Dalam lab: laporkan ke instruktur sebelum mengubah konfigurasi apapun
- Dalam produksi: ikuti runbook incident response yang sudah ditetapkan
- Isolasi tanpa menghancurkan evidence

## YANG TIDAK BOLEH DILAKUKAN PERTAMA KALI

| Tindakan Salah | Mengapa Berbahaya |
|----------------|-------------------|
| Langsung mengubah konfigurasi | Menghapus evidence, membuat perubahan yang tidak dapat diaudit |
| Mengasumsikan kompromi tanpa bukti | Menyebabkan respons yang tidak proporsional |
| Overclaim keamanan dari satu artefak | Kepercayaan palsu yang berbahaya |
| Menjalankan remediation tanpa dokumentasi | Tidak dapat diaudit, tidak dapat dipelajari |

## Trigger untuk Eskalasi
- Ditemukan akun yang tidak dikenal dalam output `w` atau `who`
- Hash file kritis berbeda dari baseline yang terdokumentasi
- Proses dengan parent yang mencurigakan dalam output `pstree`
- Traffic ke IP eksternal yang tidak diharapkan dalam PCAP
EOF
cat ~/mod5/evidence/incident/response_notes.md
```

### 3. Final Case Assessment

```bash
cat > ~/mod5/evidence/incident/final_case_assessment.md <<'EOF'
# Final Case Assessment — Modul 05

## Ringkasan Eksekutif
Environment menunjukkan pemisahan peran yang wajar antara Ubuntu dan Kali, dengan
kontrol dasar yang sudah berfungsi. Tidak ditemukan indikasi kompromi aktif. Beberapa
control gaps teridentifikasi yang relevan untuk perbaikan dalam lingkungan produksi.

## Temuan per Domain

### Security Principles
- CIA triad terpenuhi secara dasar: confidentiality melalui permission, integrity melalui
  checksum, availability melalui resource sharing yang terkontrol
- Defense in depth berfungsi dengan 5 layer teridentifikasi
- Control gap: tidak ada immutable logging dan alerting otomatis

### Threat & Attack Surface
- Attack surface yang terekspos ke jaringan terbatas pada SSH (port 22)
- Phishing sample mengandung triple-threat pattern yang perlu digunakan sebagai materi
  awareness training
- Risk utama: credential misuse melalui phishing atau brute force SSH

### Identity & Access
- Akun non-sudo: kontrol benar dan konsisten
- Session review: tidak ada anomali — seluruh session dari IP lab yang diharapkan
- Control gap: tidak ada MFA untuk SSH, tidak ada session timeout

### Cryptography
- Hash baseline terdokumentasi untuk tiga artefak utama
- TLS certificate valid dengan trust chain yang terverifikasi
- Pemahaman kritis: TLS tidak menjamin keamanan aplikasi di baliknya

### Web Security
- Juice Shop: beberapa security header ada, CSP tidak ada
- DVWA: minimal security headers, versi nginx terekspos
- Control gap: tidak ada CSP di kedua aplikasi

## Rekomendasi Prioritas

| Prioritas | Rekomendasi | Dampak |
|-----------|-------------|--------|
| Tinggi | Key-based SSH auth (nonaktifkan password auth) | Eliminasi brute force risk |
| Tinggi | Implementasikan fail2ban untuk SSH | Rate limiting otomatis |
| Sedang | Tambahkan CSP di aplikasi web | Proteksi XSS |
| Sedang | Sembunyikan Server header di nginx | Kurangi fingerprinting surface |
| Rendah | Implementasikan session timeout | Kurangi idle session risk |

## Kesimpulan
Environment dalam kondisi yang sesuai untuk tujuan latihan dengan kontrol yang wajar.
Untuk deployment produksi, rekomendasi di atas perlu diimplementasikan. Fondasi
security reasoning yang dibangun di modul ini siap dikembangkan ke blue team operations
dan incident response yang lebih operasional.
EOF
cat ~/mod5/evidence/incident/final_case_assessment.md
```

## Temuan Utama

Case assessment mengintegrasikan temuan dari seluruh enam domain analisis menjadi gambaran yang kohesif: environment dalam kondisi baik untuk latihan, namun memiliki beberapa control gaps yang relevan untuk deployment produksi. Temuan paling kritis adalah kombinasi password-based SSH authentication tanpa MFA — vektor yang paling realistis untuk kompromi dalam environment seperti ini.

Timeline yang disusun menunjukkan bahwa seluruh sesi analisis dapat direproduksi secara terstruktur dalam waktu sekitar 85 menit, dengan urutan yang logis dari asset mapping hingga incident assessment.

## Apa yang Dipelajari

*Incident thinking* bukan hanya relevan saat insiden terjadi — ini adalah pola pikir yang harus diterapkan dalam setiap sesi analisis. Mendokumentasikan urutan aktivitas sebagai timeline, merekam prioritas respons yang benar, dan mengintegrasikan temuan dari berbagai domain menjadi case assessment yang kohesif adalah kompetensi yang membedakan analis yang dapat dipercaya dalam situasi tekanan tinggi dari analis yang hanya efektif dalam kondisi normal.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `incident_timeline.md` | `~/mod5/evidence/incident/` | Timeline terstruktur seluruh observasi dalam sesi |
| `response_notes.md` | `~/mod5/evidence/incident/` | Prioritas respons yang benar dan yang tidak boleh dilakukan |
| `final_case_assessment.md` | `~/mod5/evidence/incident/` | Integrasi temuan dari semua domain dengan rekomendasi |

---
---

# Proyek 7 — Menulis Security Assessment Komprehensif dan Jembatan ke Blue Team

## Latar Belakang

Modul 05 adalah titik konvergensi dalam rangkaian praktikum ini: seluruh fondasi teknis dari modul-modul sebelumnya — context host, networking, OS analysis, Python automation — bertemu dengan prinsip-prinsip inti keamanan siber untuk menghasilkan *security reasoning* yang utuh. Proyek ini menutup konvergensi tersebut dengan dua deliverable yang berfungsi sebagai bukti kematangan analitis: assessment komprehensif yang dapat dipertahankan kepada audiens teknis, dan bridge notes yang secara eksplisit memetakan fondasi yang telah dibangun ke kebutuhan operasi keamanan yang lebih spesifik di modul berikutnya.

## Tujuan

- Menyusun module assessment yang merangkum seluruh temuan dalam format yang dapat dipertahankan
- Mendokumentasikan prinsip operasional yang harus menjadi kebiasaan kerja permanen
- Memetakan secara eksplisit bagaimana setiap kompetensi yang dibangun di modul ini relevan untuk blue team, incident response, dan red team operations
- Membiasakan pola penutupan sesi yang profesional: selalu dengan deliverable yang bernilai

## Langkah-Langkah yang Dilakukan

### 1. Module 05 Assessment

```bash
cat > ~/mod5/reports/module05_assessment.md <<'EOF'
# Module 05 Assessment — Cybersecurity Core Concepts & Applied Practice

## Ringkasan Eksekutif
Modul ini berhasil mengintegrasikan konsep-konsep inti keamanan siber ke dalam workflow
yang dapat dijalankan. Security reasoning tidak lagi abstrak — setiap prinsip (CIA, least
privilege, defense in depth, threat landscape) telah dikaitkan langsung dengan environment
dan artefak yang konkret.

## Pencapaian per Domain

| Domain | Status | Highlight |
|--------|--------|-----------|
| Security Principles | ✅ Selesai | Asset map, CIA kontekstual, defense in depth layers terdokumentasi |
| Threat Analysis | ✅ Selesai | Triple-threat phishing pattern teridentifikasi, attack surface terpetakan |
| IAM Review | ✅ Selesai | User context, session, least privilege terdokumentasi dengan gap |
| Cryptography | ✅ Selesai | Hash baseline terdokumentasi, TLS certificate terverifikasi |
| Web Security | ✅ Selesai | Security headers dibandingkan, AppSec observations proporsional |
| Incident Thinking | ✅ Selesai | Timeline tersusun, response priorities terdokumentasi, case assessment final |

## Prinsip Operasional yang Terbentuk

1. **Security dimulai dari aset, bukan ancaman** — tanpa pemetaan aset, penilaian ancaman tidak terarah
2. **Evidence sebelum kesimpulan** — tidak ada klaim keamanan atau ketidakamanan tanpa evidence
3. **Perspektif ganda selalu diperlukan** — Ubuntu untuk analisis, Kali untuk validasi
4. **Batas bukti harus selalu didokumentasikan** — hash, header, dan certificate membuktikan sesuatu yang spesifik, bukan segalanya
5. **Incident thinking adalah kebiasaan, bukan mode darurat** — timeline dan prioritas respons selalu relevan

## Takeaway Utama
Cybersecurity yang efektif adalah kombinasi dari: memahami apa yang dilindungi (asset),
mengetahui apa yang mengancam (threat), mengevaluasi apa yang sudah ada (control),
dan mengetahui apa yang masih kurang (gap). Semua itu harus didukung oleh evidence
yang terstruktur dan dapat dipertahankan.
EOF
cat ~/mod5/reports/module05_assessment.md
```

### 2. Bridge Notes ke Modul Berikutnya

```bash
cat > ~/mod5/reports/module05_bridge_notes.md <<'EOF'
# Module 05 Bridge Notes — Menuju Blue Team dan Red Team

## Prinsip yang Harus Dibawa ke Modul Berikutnya

1. **Control thinking matters** — setiap observasi harus dievaluasi dalam konteks kontrol yang ada
2. **Identity dan trust boundaries matter** — siapa yang memiliki akses ke apa selalu relevan
3. **Evidence harus dipreservasi sebelum kesimpulan dibuat** — urutan ini tidak boleh dibalik
4. **Header, hash, certificate, dan log adalah sinyal, bukan kebenaran final** — triangulasi selalu diperlukan
5. **Dokumentasi bukan formalitas** — ini adalah mekanisme akuntabilitas profesional

## Koneksi ke Modul 06 — Blue Team & Defensive Security

| Fondasi Modul 05 | Penggunaan di Modul 06 |
|------------------|------------------------|
| CIA triad dan control assessment | Framework evaluasi efektivitas kontrol defensif |
| Auth log analysis | SIEM rule development dan alert tuning |
| Hash verification | EDR dan file integrity monitoring |
| Incident timeline | SOAR playbook dan incident runbook |
| AppSec observations | Web Application Firewall (WAF) rule development |
| Threat indicator extraction | Threat Intelligence Platform (TIP) integration |

## Koneksi ke Modul 07 — Red Team Perspective

| Fondasi Modul 05 | Penggunaan di Modul 07 |
|------------------|------------------------|
| Attack surface mapping | Reconnaissance scope definition |
| Phishing IOC identification | Social engineering simulation framework |
| Security header analysis | Web application testing methodology |
| Least privilege gaps | Privilege escalation pathway assessment |
| Certificate review | TLS/HTTPS misconfiguration testing |

## Fondasi yang Siap untuk Modul 06 dan 07

- Security reasoning berbasis evidence — bukan intuisi atau asumsi
- Asset-centric threat assessment — ancaman dinilai dalam konteks aset
- Control gap documentation — gap yang teridentifikasi siap menjadi target perbaikan (blue) atau target pengujian (red)
- Incident-oriented documentation — pola kerja yang siap untuk SOC dan DFIR
- Proportional AppSec assessment — tidak overclaim, tidak underclaim
EOF
cat ~/mod5/reports/module05_bridge_notes.md
```

## Signifikansi Assessment dalam Konteks Profesional

Assessment yang dihasilkan dalam proyek ini mencerminkan standar dokumentasi yang diharapkan dalam peran profesional keamanan siber:

| Komponen | Standar yang Dipenuhi |
|----------|----------------------|
| Ringkasan eksekutif | Dapat dibaca oleh non-technical stakeholder |
| Temuan per domain | Terstruktur untuk technical reviewer |
| Prinsip operasional | Dapat dijadikan standard operating procedure |
| Bridge notes | Menunjukkan pemahaman tentang roadmap pengembangan kompetensi |

## Temuan Utama

Assessment mengkonfirmasi bahwa seluruh domain keamanan yang ditargetkan telah dianalisis dengan pendekatan yang berbasis evidence dan proporsional. Yang paling signifikan adalah terbentuknya pola kerja yang konsisten sepanjang modul: setiap observasi dikaitkan dengan aset spesifik, setiap kesimpulan dibatasi pada evidence yang ada, dan setiap temuan didokumentasikan dengan konteks yang cukup untuk dapat direproduksi oleh analis lain.

## Apa yang Dipelajari

Assessment yang baik bukan ringkasan dari semua yang dilakukan — ini adalah narasi yang menjawab pertanyaan: *apa yang ditemukan, apa artinya, apa risikonya, dan apa yang harus dilakukan?* Kemampuan untuk menjawab keempat pertanyaan ini secara kohesif dan berbasis evidence adalah kompetensi inti yang akan terus digunakan dalam seluruh karir di bidang keamanan siber.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `module05_assessment.md` | `~/mod5/reports/` | Assessment komprehensif dengan pencapaian per domain dan prinsip operasional |
| `module05_bridge_notes.md` | `~/mod5/reports/` | Koneksi eksplisit ke Modul 06 (Blue Team) dan Modul 07 (Red Team) |
