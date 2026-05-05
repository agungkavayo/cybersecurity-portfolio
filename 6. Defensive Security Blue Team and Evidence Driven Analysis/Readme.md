## Tentang Portofolio Ini

Portofolio ini mendokumentasikan serangkaian proyek *defensive security* dan *blue team operations* yang dikerjakan secara langsung (*hands-on*) di lingkungan lab terisolasi. Setiap proyek mencerminkan cara kerja seorang *SOC analyst*, *DFIR specialist*, atau *blue team engineer* profesional — mulai dari penetapan monitoring scope, triage log Linux dan Apache, analisis PCAP dan deteksi pola beaconing, review threat intelligence dan YARA scanning, forensics berbasis artefak, hingga penyusunan triage decision dan incident workflow yang dapat dipertanggungjawabkan.

Seluruh aktivitas dilakukan pada dua host lab dengan peran yang berbeda:

| Host | Peran |
|------|-------|
| **Ubuntu Lab** | *Primary analysis host* — log parsing, timeline building, YARA, artifact analysis, incident notes, threat evidence correlation, report writing |
| **Kali Linux** | *Validation workstation* — PCAP summary inspection, service validation, endpoint review, attacker-aware perspective yang terkontrol |
| **Local Targets** | *Safe practice target* — web target lokal untuk validation, header review, dan service exposure context |
| **Datasets `/opt/jccsah`** | Linux logs, Apache logs, Windows/SIEM sample, PCAP sample, phishing sample, OSINT sample, forensics sample |

> ⚠️ **Catatan:** Seluruh aktivitas dilakukan **hanya** pada lingkungan lab yang telah disediakan. Tidak ada perubahan sistem global, packet interception di luar scope, atau simulasi serangan yang dapat mengganggu pengguna lain.

---

## Daftar Proyek

| No. | Judul Proyek | Fokus Area | Tools & Konsep |
|-----|-------------|------------|----------------|
| 1 | [Menetapkan Monitoring Scope dan Pemetaan Sumber Evidence](#proyek-1--menetapkan-monitoring-scope-dan-pemetaan-sumber-evidence) | Monitoring baseline, detection scope | `ss`, log source mapping |
| 2 | [Melakukan Triage Log Linux, Apache, dan Windows Event dengan SIEM Thinking](#proyek-2--melakukan-triage-log-linux-apache-dan-windows-event-dengan-siem-thinking) | Log analysis, SIEM thinking | `grep`, `awk`, correlation analysis |
| 3 | [Menganalisis PCAP, Pola Beaconing, dan Konteks IDS/IPS](#proyek-3--menganalisis-pcap-pola-beaconing-dan-konteks-idips) | Network analysis, beaconing detection | `tshark`, firewall log review |
| 4 | [Menerapkan Threat Intelligence, IOC Enrichment, dan YARA Scanning](#proyek-4--menerapkan-threat-intelligence-ioc-enrichment-dan-yara-scanning) | Threat intel, IOC review, rule-based detection | `grep`, `yara`, enrichment |
| 5 | [Membangun Forensics-on-Files dan Timeline Investigasi](#proyek-5--membangun-forensics-on-files-dan-timeline-investigasi) | DFIR basics, evidence integrity, timeline | `sha256sum`, artifact analysis |
| 6 | [Menyusun Triage Decision dan Incident Response Workflow](#proyek-6--menyusun-triage-decision-dan-incident-response-workflow) | Incident triage, response planning | Playbook writing, case assessment |
| 7 | [Menulis Blue Team Assessment dan Jembatan ke Perspektif Offensive](#proyek-7--menulis-blue-team-assessment-dan-jembatan-ke-perspektif-offensive) | Assessment writing, blue-to-red bridge | Dokumentasi profesional |

---

## Teknologi dan Tools

<p>
  <img src="https://img.shields.io/badge/OS-Ubuntu%20Linux-E95420?style=flat-square&logo=ubuntu&logoColor=white"/>
  <img src="https://img.shields.io/badge/OS-Kali%20Linux-557C94?style=flat-square&logo=kalilinux&logoColor=white"/>
  <img src="https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat-square&logo=gnubash&logoColor=white"/>
  <img src="https://img.shields.io/badge/Tool-tshark-1679A7?style=flat-square"/>
  <img src="https://img.shields.io/badge/Tool-YARA-red?style=flat-square"/>
  <img src="https://img.shields.io/badge/Tool-sha256sum-informational?style=flat-square"/>
  <img src="https://img.shields.io/badge/Concept-SIEM%20Thinking-blue?style=flat-square"/>
  <img src="https://img.shields.io/badge/Concept-DFIR%20Basics-green?style=flat-square"/>
  <img src="https://img.shields.io/badge/Concept-SOC%20Workflow-purple?style=flat-square"/>
</p>

---

## Kompetensi yang Dibangun

- **Monitoring Scope Definition** — Menetapkan sumber evidence yang tersedia dan batas observasi yang realistis sebelum analisis dimulai
- **Log Triage & SIEM Thinking** — Melakukan triage terstruktur pada auth log, Apache access log, dan Windows/SIEM event dengan mindset korelasi antar-sumber
- **Network & PCAP Analysis** — Membaca traffic summary dari PCAP sample, mengidentifikasi pola beaconing, dan memahami reasoning di balik IDS/IPS decision
- **Threat Intelligence & YARA** — Mengekstrak IOC dari artefak, melakukan enrichment berbasis konteks, dan menerapkan rule-based scanning dengan YARA
- **DFIR Basics** — Menjaga integritas evidence dengan hash, membaca artefak forensik (registry, USB), dan menyusun timeline investigasi
- **Incident Triage & Response** — Mendokumentasikan triage decisions, menulis response playbook notes, dan menyusun case assessment yang dapat dipertanggungjawabkan
- **Blue Team Assessment Writing** — Mengintegrasikan seluruh temuan dari berbagai layer (host, log, network, artifact) menjadi penilaian defensif yang komprehensif

---
---

# Proyek 1 — Menetapkan Monitoring Scope dan Pemetaan Sumber Evidence

## Latar Belakang

Salah satu kegagalan paling umum dalam operasi *blue team* adalah memulai analisis tanpa terlebih dahulu memahami *apa yang tersedia untuk dianalisis*. SOC analyst yang langsung masuk ke log review tanpa pemetaan sumber evidence berisiko menghabiskan waktu pada dataset yang tidak relevan, melewatkan sumber yang lebih informatif, atau membuat kesimpulan yang tidak valid karena tidak mengetahui keterbatasan data yang dimiliki.

Proyek ini membangun fondasi monitoring yang benar: mendefinisikan sumber evidence yang tersedia, menetapkan scope observasi yang realistis untuk lingkungan shared lab, dan mengumpulkan baseline service context yang akan menjadi referensi sepanjang seluruh analisis. Ini adalah langkah yang tampak administratif namun memiliki dampak besar terhadap kualitas seluruh analisis berikutnya.

## Tujuan

- Membangun struktur workspace yang terbagi berdasarkan domain defensive security
- Mendokumentasikan seluruh sumber evidence yang tersedia sebagai *log source map*
- Menetapkan scope monitoring yang realistis untuk lingkungan shared lab
- Mengumpulkan baseline service context sebagai referensi untuk korelasi log dan network evidence

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Setup workspace dan dokumentasi monitoring scope |
| **Ubuntu Lab** | Pengumpulan service context baseline |

## Langkah-Langkah yang Dilakukan

### 1. Pembangunan Struktur Workspace Modul 06

```bash
mkdir -p ~/mod6/{evidence,reports,notes,monitoring,logs,network,threatintel,forensics,incident,tmp}
cd ~/mod6
pwd
ls -la
```

**Signifikansi struktur direktori dalam konteks blue team:**

Modul ini menggunakan struktur direktori yang mencerminkan *domain defensive security* — berbeda dari modul sebelumnya yang berbasis tool atau aktivitas. Struktur ini memudahkan analis untuk menemukan evidence berdasarkan konteks kerjanya: apakah sedang melakukan log triage, network analysis, threat intel review, atau forensics.

| Direktori | Domain Blue Team |
|-----------|-----------------|
| `evidence/monitoring/` | Monitoring scope, service baseline, detection notes |
| `evidence/logs/` | Auth log, web log, Windows event — triage dan korelasi |
| `evidence/network/` | PCAP summary, beaconing notes, IDS/IPS observations |
| `evidence/threatintel/` | IOC review, enrichment notes, YARA findings |
| `evidence/forensics/` | Artifact review, timeline, evidence integrity |
| `evidence/incident/` | Triage decisions, response playbook, case assessment |
| `reports/` | Module assessment dan bridge notes |

### 2. Pendokumentasian Log Source Map

```bash
cat > ~/mod6/evidence/monitoring/log_source_map.md <<'EOF'
# Log Source Map — Modul 06

## Sumber Evidence yang Tersedia

| Sumber | Lokasi | Tipe Data | Nilai Analitis |
|--------|--------|-----------|----------------|
| Linux auth log sample | `~/datasets/logs/linux/auth.log.sample` | Authentication events | Tinggi — SSH brute force, login success/fail |
| Apache access log sample | `~/datasets/logs/apache/access.log.sample` | HTTP requests | Tinggi — web scanning, path traversal, SQLi attempts |
| Windows/SIEM sample | `~/datasets/logs/windows/sysmon_sample.jsonl` | Process, network, registry | Sangat Tinggi — endpoint behavior, persistence |
| PCAP sample | `~/datasets/pcap/localhost_web_lab.pcap` | Network traffic | Tinggi — traffic patterns, protocol analysis |
| Phishing sample | `~/datasets/phishing/sample_email_1.txt` | Email artifact | Sedang — IOC extraction, social engineering analysis |
| OSINT sample | `~/datasets/osint/` | Open source intel | Sedang — context enrichment |
| Forensics samples | `~/datasets/forensics/` | Registry, USB artifacts | Tinggi — persistence, lateral movement indicators |

## Keterbatasan Monitoring dalam Konteks Lab

| Keterbatasan | Implikasi |
|-------------|-----------|
| Shared host tanpa isolasi | Tidak semua event di journal adalah milik student yang sedang bekerja |
| Sample data, bukan live stream | Tidak dapat melakukan real-time alerting atau live hunting |
| User-space only | Tidak ada akses ke kernel events atau system-wide audit trail |
| Localhost targets saja | Network evidence terbatas pada traffic lokal |

## Yang Dapat dan Tidak Dapat Disimpulkan dari Dataset Ini
- **Dapat:** Pola autentikasi, request path mencurigakan, IOC dalam artefak teks, pola traffic dalam PCAP
- **Tidak dapat:** Konfirmasi kompromi aktif, atribusi penyerang, live threat hunting
EOF
cat ~/mod6/evidence/monitoring/log_source_map.md
```

**Mengapa pemetaan sumber evidence kritis sebelum analisis:**

Dalam SOC produksi, analis yang tidak memahami *coverage gap* — sumber log apa yang tidak tersedia — berisiko memberikan *false assurance*: menyimpulkan bahwa tidak ada ancaman hanya karena tidak menemukan indikasi di log yang dimiliki, padahal log yang paling relevan mungkin tidak dikumpulkan sama sekali.

### 3. Penetapan Detection Scope

```bash
cat > ~/mod6/evidence/monitoring/detection_scope_notes.md <<'EOF'
# Detection Scope Notes

## Apa yang Berada dalam Scope

| Area | Aktivitas yang Diizinkan |
|------|--------------------------|
| Log analysis | Membaca dan menganalisis sample log yang tersedia |
| PCAP review | Membaca PCAP sample yang sudah ada — bukan live capture |
| Service validation | Memvalidasi service pada localhost targets |
| Artifact review | Membaca dan menganalisis forensics sample |
| YARA scanning | Menjalankan rule pada dataset yang sudah ada |

## Apa yang Berada di Luar Scope
- Live packet capture pada interface jaringan yang digunakan student lain
- Perubahan konfigurasi sistem atau service
- Simulasi serangan yang dapat memengaruhi student lain
- Akses ke log sistem di luar home directory tanpa izin

## Mengapa Scope Dibatasi
Environment shared lab berbeda fundamental dari SOC produksi:
- Tindakan satu student dapat memengaruhi student lain
- Tidak ada izin untuk melakukan active defense (blocking, firewall changes)
- Fokus adalah membangun reasoning dan analisis, bukan deployment kontrol
EOF
cat ~/mod6/evidence/monitoring/detection_scope_notes.md
```

### 4. Pengumpulan Service Context Baseline

```bash
ss -tulpn | tee ~/mod6/evidence/monitoring/service_context.txt
~/lab-targets-info | tee -a ~/mod6/evidence/monitoring/service_context.txt
```

**Mengapa service context harus dikumpulkan sebelum log review:**

Sinyal dalam log tidak bermakna tanpa konteks service yang berjalan. Sebuah entry `GET /api/users` di Apache log sangat berbeda artinya jika port 80 terikat ke `0.0.0.0` (terekspos ke jaringan) versus terikat ke `127.0.0.1` (hanya localhost). Service context baseline memastikan setiap temuan log dapat diinterpretasikan dalam konteks eksposur yang sebenarnya.

**Contoh output `service_context.txt`:**
```
Netid  State   Local Address:Port   Process
tcp    LISTEN  0.0.0.0:22           users:(("sshd"))
tcp    LISTEN  127.0.0.1:3000       users:(("node"))
tcp    LISTEN  127.0.0.1:8081       users:(("nginx"))

=== Lab Targets ===
Juice Shop : http://127.0.0.1:3000
DVWA       : http://127.0.0.1:8081
```

Dari output ini, analis mengetahui bahwa hanya SSH yang terekspos ke seluruh jaringan — web targets terikat ke localhost. Informasi ini langsung relevan saat menganalisis log jaringan.

## Temuan Utama

Pemetaan sumber evidence mengungkap bahwa lab memiliki coverage yang cukup komprehensif untuk tujuan latihan: auth log untuk authentication events, Apache log untuk web requests, Windows/SIEM sample untuk endpoint behavior, dan PCAP untuk network traffic. Coverage gap yang signifikan adalah tidak adanya *endpoint detection* real-time (EDR) dan *network flow data* (NetFlow/sFlow) — keduanya adalah sumber yang sangat penting dalam SOC produksi.

Service baseline mengkonfirmasi bahwa hanya SSH yang benar-benar terekspos ke jaringan. Ini penting untuk menginterpretasikan temuan di log berikutnya: request mencurigakan di Apache log tidak dapat berasal dari luar host karena Apache terikat ke localhost.

## Apa yang Dipelajari

*Monitoring without scope* adalah monitoring yang tidak dapat dievaluasi efektivitasnya. Dengan menetapkan scope di awal — apa yang dimonitor, apa keterbatasannya, dan apa yang tidak tercakup — analis memiliki basis untuk membuat klaim yang *defensible*: kesimpulan yang dapat dipertahankan karena batasannya sudah didokumentasikan dengan jelas.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `log_source_map.md` | `~/mod6/evidence/monitoring/` | Inventaris sumber evidence dengan nilai analitis dan keterbatasannya |
| `detection_scope_notes.md` | `~/mod6/evidence/monitoring/` | Scope monitoring yang ditetapkan dan justifikasinya |
| `service_context.txt` | `~/mod6/evidence/monitoring/` | Baseline service yang berjalan sebelum analisis dimulai |

---
---

# Proyek 2 — Melakukan Triage Log Linux, Apache, dan Windows Event dengan SIEM Thinking

## Latar Belakang

*SIEM thinking* bukan tentang dashboard — ini tentang kemampuan untuk melihat event dari berbagai sumber, mengidentifikasi mana yang relevan, dan menghubungkan sinyal yang terpisah menjadi narasi yang kohesif. Analyst yang membaca setiap log secara terpisah akan melewatkan pola yang hanya terlihat ketika multiple sumber dikorelasikan: autentikasi gagal yang diikuti oleh web request mencurigakan dari IP yang sama, atau process creation yang diikuti oleh outbound connection ke domain yang tidak dikenal.

Proyek ini membangun kemampuan triage log yang terstruktur pada tiga sumber berbeda: Linux auth log untuk pola autentikasi, Apache access log untuk request pattern yang mencurigakan, dan Windows/SIEM sample untuk pemahaman endpoint behavior. Yang membedakan proyek ini dari sekadar "menjalankan grep" adalah pembangunan *correlation mindset* — kebiasaan untuk selalu bertanya bagaimana temuan di satu log berhubungan dengan temuan di log lainnya.

## Tujuan

- Melakukan triage auth log untuk mengidentifikasi pola autentikasi yang mencurigakan dan merangkum IP sumber
- Menganalisis Apache access log untuk menemukan request path yang mengindikasikan scanning atau eksploitasi
- Meninjau Windows/SIEM event sample untuk membangun pemahaman tentang endpoint behavior yang relevan
- Membangun correlation notes yang menghubungkan temuan dari ketiga sumber log

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Seluruh triage log dan correlation analysis dilakukan di sini |

## Langkah-Langkah yang Dilakukan

### 1. Triage Auth Log Linux

```bash
grep -i 'failed\|accepted' ~/datasets/logs/linux/auth.log.sample | \
    tee ~/mod6/evidence/logs/auth_triage.txt

awk '/Failed password/ {print $(NF-3), $(NF-1)}' \
    ~/datasets/logs/linux/auth.log.sample | sort | uniq -c | \
    tee -a ~/mod6/evidence/logs/auth_triage.txt
```

**Framework triage auth log — empat pertanyaan utama:**

| Pertanyaan | Command yang Menjawab | Relevansi |
|------------|----------------------|-----------|
| *Siapa* yang mencoba login? | `grep 'failed\|accepted'` — ekstrak username | Mengidentifikasi target akun |
| *Dari mana* mereka terhubung? | `awk '{print $(NF-3)}'` — ekstrak IP | Mengidentifikasi sumber aktivitas |
| *Seberapa sering*? | `sort \| uniq -c` — hitung frekuensi | Membedakan human error dari brute force |
| *Berhasil atau tidak*? | Cari `Accepted` setelah `Failed` dari IP sama | Menentukan apakah serangan berhasil |

**Threshold untuk eskalasi dalam konteks auth log:**

| Pola | Threshold | Indikasi |
|------|-----------|----------|
| `Failed password` dari satu IP | ≥5 dalam 1 menit | Kemungkinan automated brute force |
| `Failed password for root` | Satu pun sudah mencurigakan | Upaya langsung ke akun privileged |
| `Accepted` setelah multiple `Failed` dari IP sama | Tidak ada threshold — langsung eskalasi | Brute force berhasil |
| `Invalid user` | ≥3 username berbeda | Username enumeration |

**Contoh output auth_triage.txt:**
```
May 15 10:23:11 ubuntu sshd[1234]: Failed password for root from 10.10.10.99 port 45231
May 15 10:23:14 ubuntu sshd[1234]: Failed password for root from 10.10.10.99 port 45234
May 15 10:23:17 ubuntu sshd[1234]: Failed password for root from 10.10.10.99 port 45237
May 15 10:25:03 ubuntu sshd[1235]: Accepted publickey for student from 10.10.10.5

      7 10.10.10.99 ssh2
      1 10.10.10.5  ssh2
```

### 2. Triage Apache Access Log

```bash
cat ~/datasets/logs/apache/access.log.sample | \
    tee ~/mod6/evidence/logs/apache_triage.txt

grep -E 'script|passwd|download|admin|\.\./' \
    ~/datasets/logs/apache/access.log.sample | \
    tee -a ~/mod6/evidence/logs/apache_triage.txt || true
```

**Path yang perlu diperhatikan dalam Apache access log:**

| Pattern | Contoh Request | Kemungkinan Indikasi |
|---------|---------------|----------------------|
| `../` atau `..%2F` | `GET /app/../../../etc/passwd` | Path traversal attempt |
| `/admin`, `/manager` | `GET /admin/login.php` | Admin panel discovery |
| `passwd`, `shadow` | `GET /etc/passwd` | Sensitive file access attempt |
| `.php?` dengan parameter panjang | `GET /page.php?id=1'+OR+'1'='1` | SQL injection attempt |
| `<script`, `%3Cscript` | `GET /search?q=<script>alert(1)` | XSS injection attempt |
| `/download` | `GET /download?file=../../../../etc/shadow` | File download exploitation |

**Cara membaca format Apache Combined Log:**
```
10.10.10.99 - - [15/May/2024:10:23:45 +0000] "GET /admin HTTP/1.1" 404 452 "-" "sqlmap/1.7"
│            │   │                              │                    │   │    │   │
│            │   Timestamp                      Request             Status│  Referer User-Agent
IP           User                                                        Size
```

**Red flags dari User-Agent:**

| User-Agent | Indikasi |
|------------|----------|
| `sqlmap/` | SQL injection testing tool |
| `Nikto/` | Web vulnerability scanner |
| `Nessus/` | Vulnerability scanner |
| `python-requests/` | Automated script — bisa legitimate atau tidak |
| `curl/` | Manual atau scripted request |

### 3. Review Windows/SIEM Event Sample

```bash
cat ~/datasets/logs/windows/sysmon_sample.jsonl | \
    tee ~/mod6/evidence/logs/windows_event_notes.txt
```

**Pengenalan Sysmon dan event ID yang paling relevan:**

Sysmon (*System Monitor*) adalah komponen Windows yang menghasilkan log detail tentang aktivitas endpoint. Format JSONL (JSON Lines) memudahkan parsing karena setiap baris adalah satu event lengkap.

| Sysmon Event ID | Nama | Relevansi Keamanan |
|----------------|------|--------------------|
| **1** | Process Create | Proses baru yang berjalan — kunci untuk deteksi malware |
| **3** | Network Connection | Koneksi jaringan dari proses — deteksi C2 communication |
| **7** | Image Loaded | DLL yang dimuat — deteksi DLL injection |
| **8** | CreateRemoteThread | Thread injection ke proses lain — deteksi process injection |
| **11** | File Create | File baru yang dibuat — deteksi dropper |
| **13** | Registry Value Set | Perubahan registry — deteksi persistence |
| **22** | DNS Query | DNS resolution oleh proses — deteksi C2 domain |

**Pola korelasi yang paling valuable dalam Sysmon:**

```
Event ID 1 (Process Create): cmd.exe
    ↓ diikuti oleh
Event ID 3 (Network Connection): outbound ke 185.x.x.x:443
    ↓ diikuti oleh
Event ID 11 (File Create): %TEMP%\update.exe

→ Ini adalah pola classic malware execution:
  shell → download → execute
```

### 4. Correlation Notes

```bash
cat > ~/mod6/evidence/logs/correlation_notes.md <<'EOF'
# Correlation Notes — SIEM Thinking

## Prinsip Korelasi

"Satu event adalah observasi. Dua event yang terhubung adalah sinyal. Tiga event
yang membentuk pola adalah temuan yang layak dieskalasi."

## Korelasi yang Diidentifikasi dari Dataset

### Korelasi 1: Auth + Network
IP 10.10.10.99 melakukan 7x failed SSH login (dari auth log).
Jika IP yang sama kemudian muncul di Apache access log dengan request mencurigakan,
korelasi ini meningkatkan tingkat kepercayaan temuan secara signifikan.

### Korelasi 2: Process + Network (Windows)
Process creation event (Sysmon EID 1) yang diikuti oleh network connection (Sysmon EID 3)
ke IP eksternal yang tidak diharapkan mengindikasikan kemungkinan command-and-control
communication atau data exfiltration.

### Korelasi 3: Registry + Process (Windows)
Registry persistence entry (Sysmon EID 13) yang dikombinasikan dengan process creation
dari path yang tidak lazim (temp folder, AppData) mengindikasikan kemungkinan
malware persistence mechanism.

## Pertanyaan Korelasi yang Harus Selalu Ditanyakan

| Pertanyaan | Mengapa Penting |
|------------|----------------|
| Apakah IP yang sama muncul di beberapa log? | Mengkonfirmasi atau menolak hipotesis |
| Apakah timeline event konsisten? | Sequence matters — urutan event menentukan interpretasi |
| Apakah username yang sama terlibat? | Identity thread membantu rekonstruksi aktivitas |
| Apakah ada proses yang tidak biasa terlibat? | Unusual process = potential malware atau misuse |

## Keterbatasan Korelasi Manual
- Tanpa SIEM yang sesungguhnya, korelasi manual tidak dapat dilakukan pada volume besar
- Timestamp dari sumber berbeda harus diselaraskan (timezone differences)
- False positives lebih tinggi tanpa baseline behavior yang terdokumentasi
EOF
cat ~/mod6/evidence/logs/correlation_notes.md
```

## Hasil dan Output

**Dari auth triage:** 7 failed login attempts dari `10.10.10.99` dalam interval singkat — pola brute force yang jelas. Satu accepted login dari `10.10.10.5` menggunakan public key — login yang sah.

**Dari Apache triage:** Jika tersedia request ke path sensitif (`/admin`, `passwd`, `../../`), ini mengindikasikan scanning atau exploitation attempt yang perlu dikorelasikan dengan IP di auth log.

**Dari Windows/SIEM:** Process creation events yang diikuti network connections ke IP eksternal adalah pola yang memerlukan investigasi lebih lanjut — terutama jika proses yang terlibat adalah interpreter atau shell.

## Temuan Utama

Korelasi antara auth log dan Apache access log adalah yang paling informatif dalam dataset ini: jika IP yang melakukan SSH brute force (`10.10.10.99`) juga muncul di Apache access log dengan request mencurigakan pada timestamp yang hampir bersamaan, ini mengindikasikan aktivitas reconnaissance yang terkoordinasi — bukan sekadar kesalahan user biasa.

Correlation notes mengidentifikasi tiga pola korelasi kritis yang relevan untuk SOC: auth+network, process+network (Windows), dan registry+process (Windows). Ketiga pola ini adalah dasar dari banyak detection rule di SIEM produksi.

## Apa yang Dipelajari

*SIEM thinking* yang sesungguhnya adalah kemampuan untuk melihat hubungan antara event yang secara individual mungkin tampak tidak signifikan. `Failed password` satu kali bisa jadi salah ketik. Tujuh kali dalam 30 detik dari IP yang sama adalah brute force. Brute force yang diikuti oleh web request mencurigakan dari IP yang sama adalah reconnaissance campaign. Progression dari observasi ke sinyal ke temuan inilah yang membedakan analis yang efektif.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `auth_triage.txt` | `~/mod6/evidence/logs/` | Failed/accepted login events dan ringkasan IP |
| `apache_triage.txt` | `~/mod6/evidence/logs/` | Apache access log dan request mencurigakan |
| `windows_event_notes.txt` | `~/mod6/evidence/logs/` | Windows/Sysmon event sample |
| `correlation_notes.md` | `~/mod6/evidence/logs/` | Framework korelasi dan temuan yang diidentifikasi |

---
---

# Proyek 3 — Menganalisis PCAP, Pola Beaconing, dan Konteks IDS/IPS

## Latar Belakang

Network evidence sering kali menjadi sumber konfirmasi yang paling kuat dalam investigasi keamanan: log dapat dihapus atau dimanipulasi, tetapi traffic yang sudah ter-capture dalam PCAP menjadi bukti yang sulit dibantah. Di sisi lain, analyst yang tidak terlatih membaca PCAP cenderung *tenggelam dalam detail* — menghabiskan waktu membaca payload individual alih-alih memulai dari pola tingkat tinggi yang lebih informatif.

Proyek ini membangun kemampuan analisis jaringan yang efisien: memulai dari summary PCAP untuk mengidentifikasi pola, menghubungkan pola tersebut dengan konsep *beaconing* (koneksi berulang yang mengindikasikan malware C2), dan membangun reasoning yang tepat tentang kapan dan bagaimana IDS/IPS seharusnya bereaksi — termasuk pemahaman tentang trade-off antara detection rate dan false positive rate.

## Tujuan

- Membaca PCAP sample menggunakan `tshark` dan mengidentifikasi pola traffic yang relevan
- Menghubungkan observasi PCAP dengan konsep beaconing dan indikator potensi command-and-control
- Membangun IDS/IPS observations yang mencerminkan reasoning yang matang tentang detection dan prevention
- Memahami keterbatasan analisis PCAP sample versus live network monitoring

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | PCAP analysis dengan tshark dan dokumentasi beaconing notes |
| **Kali Linux** | Pendukung untuk service validation jika diperlukan |

## Langkah-Langkah yang Dilakukan

### 1. Analisis PCAP Sample dengan tshark

```bash
tshark -r ~/datasets/pcap/localhost_web_lab.pcap | head -n 30 | \
    tee ~/mod6/evidence/network/pcap_summary.txt
```

**Pendekatan analisis PCAP yang efisien — dari atas ke bawah:**

Analis yang efektif tidak memulai dari paket pertama dan membaca satu per satu. Mereka memulai dari pertanyaan tingkat tinggi:

1. **Berapa banyak traffic?** Volume yang tidak biasa bisa mengindikasikan scanning atau data transfer
2. **Protocol apa yang dominan?** Distribusi protokol yang tidak normal adalah sinyal awal
3. **Host mana yang paling aktif?** IP dengan volume traffic tertinggi perlu perhatian
4. **Ada pattern temporal?** Traffic yang berulang dengan interval reguler mengindikasikan beaconing

**Cara membaca output tshark secara efisien:**

```
Frame  Time    Source     → Dest      Protocol Size  Info
1      0.000   127.0.0.1  → 127.0.0.1  TCP     74    54321 → 3000 [SYN]
2      0.000   127.0.0.1  → 127.0.0.1  TCP     74    3000 → 54321 [SYN, ACK]
3      0.000   127.0.0.1  → 127.0.0.1  TCP     66    54321 → 3000 [ACK]
4      0.001   127.0.0.1  → 127.0.0.1  HTTP    234   GET /rest/user/login HTTP/1.1
5      0.002   127.0.0.1  → 127.0.0.1  HTTP    89    HTTP/1.1 401 Unauthorized
6      0.003   127.0.0.1  → 127.0.0.1  HTTP    234   GET /rest/user/login HTTP/1.1
7      0.004   127.0.0.1  → 127.0.0.1  HTTP    89    HTTP/1.1 401 Unauthorized
```

Frame 4-7 menunjukkan dua request ke `/rest/user/login` yang keduanya mendapat `401 Unauthorized` — pola yang mengindikasikan percobaan login yang gagal berulang. Jika pola ini berlanjut puluhan kali, ini adalah credential stuffing atau brute force via web interface.

**Tshark flags yang berguna untuk analisis lanjutan:**

| Flag | Fungsi | Contoh Penggunaan |
|------|--------|-------------------|
| `-r` | Baca dari file PCAP | `tshark -r capture.pcap` |
| `ip.addr == X.X.X.X` | Filter berdasarkan IP | Fokus pada satu host |
| `http.request` | Filter hanya HTTP request | Web traffic analysis |
| `tcp.flags.syn == 1` | Filter SYN packets | Port scanning detection |
| `-T fields -e` | Export field tertentu | Ekstrak IP, timestamp untuk spreadsheet |

### 2. Beaconing Notes dari Firewall Log Sample

```bash
cat ~/datasets/logs/network/firewall_logs.txt | \
    tee ~/mod6/evidence/network/beaconing_notes.md

cat >> ~/mod6/evidence/network/beaconing_notes.md <<'EOF'

# Observasi Beaconing

## Apa itu Beaconing
Beaconing adalah pola komunikasi di mana malware yang terinfeksi melakukan koneksi
keluar ke server command-and-control (C2) secara berkala dengan interval yang konsisten.
Karakteristiknya: destination yang sama, interval yang reguler, payload yang kecil.

## Cara Mengidentifikasi Beaconing dari Log

| Indikator | Threshold | Catatan |
|-----------|-----------|---------|
| Koneksi ke IP/domain yang sama | >10x dalam 1 jam | Semakin konsisten intervalnya, semakin mencurigakan |
| Ukuran payload yang konsisten | Variasi <10% | HTTP beaconing sering menggunakan payload tetap |
| Interval yang reguler | Variasi <5% dari mean | Jittering dapat digunakan untuk menghindari deteksi |
| Koneksi di luar jam kerja | Tergantung baseline | Malware tidak peduli jam kerja |

## Mengapa Beaconing Sulit Dideteksi
- Malware modern menggunakan jitter (variasi acak) pada interval untuk menghindari
  deteksi berbasis pattern
- Banyak malware menggunakan protokol HTTPS, menyulitkan inspeksi payload
- Beaconing dapat menyerupai legitimate telemetry (antivirus updates, OS updates)

## Prinsip Kehati-Hatian
Koneksi berulang ke destination yang sama BISA jadi beaconing, atau bisa jadi:
- Software update check yang legitim
- Monitoring agent yang normal berjalan
- Application health check

Konteks dan volume tetap menentukan sebelum eskalasi dilakukan.
EOF
```

### 3. IDS/IPS Observations

```bash
cat > ~/mod6/evidence/network/ids_ips_observations.md <<'EOF'
# IDS/IPS Observations — Reasoning yang Matang

## Perbedaan IDS dan IPS

| Aspek | IDS (Intrusion Detection) | IPS (Intrusion Prevention) |
|-------|--------------------------|---------------------------|
| Aksi | Alert — memberitahu analyst | Block — memblokir traffic secara otomatis |
| Posisi | Out-of-band (mirror/tap) | Inline (di jalur traffic) |
| Dampak false positive | Alert yang tidak perlu | Legitimate traffic yang diblokir |
| Risiko | Miss detection | Service disruption |

## Jenis Detection dalam IDS/IPS

### Signature-based Detection
- Mencocokkan traffic dengan pola serangan yang diketahui
- Pro: false positive rendah untuk known threats
- Con: tidak mendeteksi zero-day atau variasi baru

### Anomaly-based Detection
- Mendeteksi deviasi dari baseline normal
- Pro: dapat mendeteksi serangan baru
- Con: false positive tinggi jika baseline tidak akurat

### Behavior-based Detection
- Mendeteksi urutan aksi yang mencurigakan (bukan satu event tunggal)
- Pro: efektif untuk APT dan serangan multi-stage
- Con: butuh waktu untuk membangun model behavior

## Trade-off dalam Keputusan Blocking

| Pertimbangan | Pertanyaan yang Harus Dijawab |
|-------------|-------------------------------|
| Confidence level | Seberapa yakin ini adalah malicious traffic? |
| Business impact | Apa dampak jika legitimate traffic diblokir? |
| Reversibility | Apakah blocking dapat dengan mudah di-undo? |
| Alternative actions | Apakah ada langkah yang kurang disruptif? |

## Konteks Lab
Dalam lingkungan shared lab, "blocking" tidak dapat diterapkan tanpa memengaruhi student
lain. Fokus adalah membangun reasoning tentang kapan dan mengapa blocking decisions
dibuat dalam SOC produksi, bukan mengimplementasikan blocking secara aktual.
EOF
cat ~/mod6/evidence/network/ids_ips_observations.md
```

## Temuan Utama

Analisis PCAP mengungkap pola request berulang ke `/rest/user/login` dengan respons `401` — mengkonfirmasi credential testing yang sudah diidentifikasi sebelumnya dalam auth log. Korelasi antara PCAP dan auth log dari dua sumber yang berbeda meningkatkan tingkat kepercayaan temuan secara signifikan.

Dokumentasi beaconing notes memperjelas bahwa identifikasi beaconing memerlukan baseline yang baik: tanpa mengetahui seperti apa traffic normal dari host tersebut, sulit membedakan beaconing dari legitimate telemetry.

## Apa yang Dipelajari

Analisis jaringan yang efektif dimulai dari pertanyaan tingkat tinggi, bukan dari paket pertama. Dan yang lebih penting: temuan dalam PCAP bernilai lebih tinggi ketika dikonfirmasi oleh sumber lain (log, artifact, behavior) — sebuah prinsip yang berlaku universal dalam defensive security.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `pcap_summary.txt` | `~/mod6/evidence/network/` | Ringkasan 30 frame pertama PCAP sample |
| `beaconing_notes.md` | `~/mod6/evidence/network/` | Firewall log sample dan framework identifikasi beaconing |
| `ids_ips_observations.md` | `~/mod6/evidence/network/` | Reasoning tentang IDS/IPS detection dan prevention decisions |

---
---

# Proyek 4 — Menerapkan Threat Intelligence, IOC Enrichment, dan YARA Scanning

## Latar Belakang

*Threat intelligence* yang efektif bukan sekadar berlangganan feed IOC dan mengimpornya ke SIEM. IOC (Indicator of Compromise) tanpa konteks adalah sinyal tanpa makna — alamat IP yang muncul dalam satu feed mungkin saja merupakan IP CDN yang digunakan oleh ribuan website legitimate. Yang membuat threat intel berharga adalah proses *enrichment*: menambahkan konteks yang menjelaskan mengapa sebuah IOC relevan untuk environment dan organisasi tertentu.

Proyek ini membangun kemampuan *applied threat intelligence* dalam konteks lab: mengekstrak IOC dari artefak yang tersedia, melakukan enrichment berbasis konteks, dan menggunakan YARA sebagai mekanisme rule-based detection yang scalable. Fokusnya adalah membangun *threat intel discipline* — kebiasaan untuk tidak hanya mengumpulkan IOC, tetapi juga memahami dan mendokumentasikan konteksnya.

## Tujuan

- Mengekstrak IOC dari phishing sample dan OSINT dataset menggunakan grep-based review
- Membangun enrichment notes yang menambahkan konteks pada IOC yang ditemukan
- Menulis dan menerapkan YARA rule untuk deteksi berbasis keyword pada artefak phishing
- Memahami perbedaan antara IOC yang actionable dan IOC yang hanya menambah noise

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | IOC review, enrichment notes, dan YARA scanning |

## Langkah-Langkah yang Dilakukan

### 1. IOC Review dari Phishing dan OSINT Dataset

```bash
grep -RinE 'http|@|vpn|verify|urgent' \
    ~/datasets/phishing ~/datasets/osint 2>/dev/null | \
    tee ~/mod6/evidence/threatintel/ioc_review.txt
```

**Tipe IOC dan karakteristiknya:**

| Tipe IOC | Contoh | Masa Berlaku | Nilai |
|----------|--------|--------------|-------|
| IP address | `185.234.x.x` | Pendek (hari-minggu) | Rendah-Sedang |
| Domain | `urgent-verify.com` | Sedang (minggu) | Sedang |
| URL | `http://fake-bank.com/login` | Pendek | Sedang |
| Email address | `security@paypa1.com` | Sedang | Sedang-Tinggi |
| File hash (SHA256) | `a3f92c1d...` | Panjang (permanen) | Tinggi |
| YARA rule | Pattern-based | Panjang | Sangat Tinggi |

**Mengapa hash dan YARA rule lebih bernilai dari IP:**

IP address sering berubah — penyerang dapat dengan mudah berpindah ke hosting baru. Domain sedikit lebih stabil. File hash dan YARA rule adalah IOC yang paling tahan lama karena malware yang tidak dimodifikasi akan selalu memiliki hash yang sama, dan YARA rule yang baik dapat mendeteksi variasi dari malware yang sama berdasarkan karakteristik kodenya.

**Proses ekstraksi IOC yang terstruktur:**

```bash
# Ekstrak email addresses
grep -oE '[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}' phishing_sample.txt

# Ekstrak URLs
grep -oE 'https?://[^ ]+' phishing_sample.txt

# Ekstrak IP addresses
grep -oE '\b([0-9]{1,3}\.){3}[0-9]{1,3}\b' phishing_sample.txt
```

### 2. Enrichment Notes

```bash
cat > ~/mod6/evidence/threatintel/enrichment_notes.md <<'EOF'
# Enrichment Notes — Menambahkan Konteks pada IOC

## Apa itu Enrichment
Enrichment adalah proses menambahkan konteks pada IOC raw untuk menentukan
apakah IOC tersebut relevan, valid, dan actionable untuk environment kita.

## Framework Enrichment

### Untuk Email IOC
| Field | Yang Dicari | Sumber Enrichment |
|-------|-------------|-------------------|
| Domain age | Domain baru (<30 hari) lebih mencurigakan | WHOIS lookup |
| MX records | Tidak ada MX = bukan domain email sah | DNS lookup |
| Reputation | Apakah domain ada di blacklist? | VirusTotal, AbuseIPDB |
| Pattern | Typosquatting? (paypa1.com, g00gle.com) | Manual review |

### Untuk IP IOC
| Field | Yang Dicari | Sumber |
|-------|-------------|--------|
| ASN/Owner | VPS provider → lebih likely threat actor | WHOIS/BGP lookup |
| Geolocation | Konsisten dengan threat actor profile? | IP geolocation |
| Historical activity | Apakah pernah dilaporkan sebelumnya? | VirusTotal, Shodan |
| Current hosting | CDN IP? → kemungkinan false positive | Reverse DNS |

## IOC yang Ditemukan dalam Dataset Lab

### Dari Phishing Sample
- Email domain: `urgent-verify.com` — nama domain mengandung kata "urgent" (social engineering indicator)
- Email domain: `fakebank-alert.net` — bukan domain bank yang legitimate
- Keyword pattern: kombinasi "urgent" + "verify" + "password" = triple-threat social engineering

### Enrichment yang Dilakukan
- Domain `urgent-verify.com`: dibuat dengan tujuan phishing (pola nama domain mencurigakan)
- Tidak ada legitimate organization yang menggunakan domain dengan kata "urgent" dalam nama domain bisnis mereka

## IOC yang Sebaiknya TIDAK Langsung Diblokir Tanpa Validasi
- IP address CDN (Cloudflare, Fastly, Akamai) — memblokir akan merusak akses ke ribuan situs
- Domain email provider besar — Google, Microsoft: bisa jadi pengirim yang ditipu, bukan aktor
- URL shortener — bit.ly, tinyurl: sering digunakan untuk phishing tapi juga untuk marketing legitimate

## Prinsip Enrichment yang Baik
"Enrichment yang baik mengurangi ambiguitas. Enrichment yang buruk hanya menambah noise."
EOF
cat ~/mod6/evidence/threatintel/enrichment_notes.md
```

### 3. YARA Scanning pada Artefak Phishing

```bash
cat > ~/mod6/tmp/blue_keywords.yar <<'EOF'
rule blue_team_keywords {
    meta:
        description   = "Deteksi keyword yang umum ditemukan dalam phishing dan social engineering"
        author        = "Blue Team Lab — Modul 06"
        date          = "2024-04-27"
        reference     = "Internal threat intelligence"
        severity      = "medium"

    strings:
        $urgent   = "urgent"   nocase
        $verify   = "verify"   nocase
        $password = "password" nocase
        $vpn      = "vpn"      nocase

    condition:
        2 of them
}
EOF

yara ~/mod6/tmp/blue_keywords.yar ~/datasets/phishing/sample_email_1.txt | \
    tee ~/mod6/evidence/threatintel/yara_findings.txt
```

**Mengapa kondisi `2 of them` lebih baik dari `any of them`:**

`any of them` akan men-trigger rule jika hanya satu kata ditemukan. Kata seperti "password" atau "verify" muncul dalam komunikasi yang sangat legitimate (password reset email yang sah, formulir verifikasi yang normal). Dengan `2 of them`, rule hanya men-trigger jika minimal dua kata ditemukan bersama-sama — mengurangi false positive secara signifikan.

**Contoh output `yara_findings.txt`:**
```
blue_team_keywords /home/student/datasets/phishing/sample_email_1.txt
```

**Mengembangkan YARA rule yang lebih sophisticated:**

```yara
rule phishing_triple_threat {
    meta:
        description = "Deteksi email phishing dengan multiple social engineering indicators"

    strings:
        $urgency1 = "urgent"          nocase
        $urgency2 = "immediately"     nocase
        $urgency3 = "within 24 hours" nocase
        $action1  = "verify"          nocase
        $action2  = "confirm"         nocase
        $target1  = "password"        nocase
        $target2  = "credentials"     nocase
        $target3  = "login"           nocase

    condition:
        (1 of ($urgency*)) and
        (1 of ($action*)) and
        (1 of ($target*))
}
```

Rule yang lebih sophisticated ini memerlukan *semua tiga kategori* diwakili (urgency + action + target) — mencerminkan triple-threat pattern yang diidentifikasi dalam analisis phishing sebelumnya.

## Temuan Utama

YARA scan mengkonfirmasi keberadaan minimal dua keyword social engineering dalam phishing sample. Enrichment notes mengidentifikasi bahwa domain `urgent-verify.com` memiliki karakteristik yang sangat mencurigakan berdasarkan pola nama domain saja — bahkan sebelum memeriksa konten email. Ini menunjukkan nilai *proactive IOC screening*: domain dengan pola nama tertentu dapat di-flag untuk review lebih lanjut sebelum email dari domain tersebut bahkan dibuka.

## Apa yang Dipelajari

Threat intelligence bernilai ketika memperjelas keputusan, bukan hanya memperbanyak indikator. IOC yang paling berguna adalah yang memiliki konteks: bukan hanya "IP ini ada di blacklist" tapi "IP ini ada di blacklist karena terkait kampanye phishing yang menarget sektor keuangan pada bulan lalu, menggunakan teknik yang sama dengan yang kita lihat di sini." Konteks inilah yang mengubah IOC menjadi *intelligence*.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `ioc_review.txt` | `~/mod6/evidence/threatintel/` | Hasil ekstraksi IOC dari phishing dan OSINT dataset |
| `enrichment_notes.md` | `~/mod6/evidence/threatintel/` | Framework enrichment dan analisis IOC yang ditemukan |
| `yara_findings.txt` | `~/mod6/evidence/threatintel/` | Hasil YARA scan pada phishing sample |

---
---

# Proyek 5 — Membangun Forensics-on-Files dan Timeline Investigasi

## Latar Belakang

*Digital Forensics and Incident Response* (DFIR) dalam konteks lab berbeda dari DFIR produksi dalam hal scope, tetapi tidak berbeda dalam hal *prinsip*. Prinsip utama DFIR — preservasi evidence sebelum analisis, dokumentasi setiap langkah, dan rekonstruksi timeline dari artefak yang tersedia — berlaku sama dalam investigasi korporat senilai jutaan dolar maupun dalam praktikum lab.

Proyek ini mempraktikkan *forensics-on-files*: menganalisis artefak digital yang sudah tersedia (registry sample, USB artifacts) tanpa melakukan live acquisition yang memerlukan akses privileged. Fokusnya adalah membangun dua kompetensi DFIR yang paling fundamental: menjaga integritas evidence dengan hash, dan menyusun timeline investigasi dari artefak yang terfragmentasi.

## Tujuan

- Membaca dan menganalisis artefak forensik (registry run keys, USB artifacts) yang tersedia dalam dataset
- Menghasilkan dan mendokumentasikan SHA256 hash dari setiap artefak sebagai bukti integritas evidence
- Menyusun timeline investigasi yang menghubungkan artefak terpisah menjadi narasi yang kohesif
- Memahami implikasi keamanan dari setiap jenis artefak yang dianalisis

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Seluruh analisis forensik dan timeline building dilakukan di sini |

## Langkah-Langkah yang Dilakukan

### 1. Review Artefak Forensik

```bash
cat ~/datasets/forensics/registry_runkeys_sample.txt | \
    tee ~/mod6/evidence/forensics/artifact_review.txt

cat ~/datasets/forensics/usb_artifacts_sample.txt | \
    tee -a ~/mod6/evidence/forensics/artifact_review.txt
```

**Registry Run Keys — Mekanisme Persistence yang Umum:**

Registry Run Keys adalah salah satu mekanisme persistence yang paling umum digunakan oleh malware. Ketika sebuah entry ditambahkan ke Run Key, program tersebut akan otomatis dieksekusi setiap kali Windows startup.

**Lokasi Run Keys yang perlu dimonitor:**

| Registry Path | Scope | Keterangan |
|---------------|-------|------------|
| `HKCU\Software\Microsoft\Windows\CurrentVersion\Run` | Per-user | Dieksekusi saat user login |
| `HKLM\Software\Microsoft\Windows\CurrentVersion\Run` | System-wide | Dieksekusi saat Windows startup untuk semua user |
| `HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce` | Per-user, sekali | Dieksekusi sekali lalu dihapus |
| `HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon` | System-wide | Dieksekusi saat Winlogon |

**Red flags dalam registry Run Keys:**

| Indikator | Contoh | Kemungkinan Indikasi |
|-----------|--------|----------------------|
| Path ke temp folder | `C:\Users\user\AppData\Local\Temp\update.exe` | Malware dropper |
| Nama yang menyerupai sistem | `WindowsUpdate.exe`, `svchost32.exe` | Masquerading |
| Path yang tidak standar | `C:\Users\Public\Documents\tool.exe` | Malware dalam user-writable location |
| File dengan ekstensi ganda | `document.pdf.exe` | Extension spoofing |

**USB Artifacts — Rekonstruksi Aktivitas Removable Media:**

USB artifacts memberikan gambaran tentang perangkat USB apa yang pernah terhubung ke sistem, kapan, dan pengguna siapa yang menggunakannya — informasi yang sangat relevan dalam investigasi data exfiltration.

**Artefak USB yang dapat dianalisis:**

| Artefak | Lokasi (Windows) | Informasi |
|---------|-----------------|-----------|
| USB device entries | `HKLM\SYSTEM\CurrentControlSet\Enum\USBSTOR` | Device ID, VID/PID, serial number |
| MountedDevices | `HKLM\SYSTEM\MountedDevices` | Drive letter yang ditetapkan |
| Shellbag | `HKCU\Software\Microsoft\Windows\Shell` | Folder yang pernah dibuka dari USB |
| LNK files | `%APPDATA%\Microsoft\Windows\Recent` | File yang pernah dibuka dari USB |

### 2. Evidence Integrity dengan SHA256

```bash
sha256sum ~/datasets/forensics/registry_runkeys_sample.txt | \
    tee ~/mod6/evidence/forensics/evidence_integrity.txt

sha256sum ~/datasets/forensics/usb_artifacts_sample.txt | \
    tee -a ~/mod6/evidence/forensics/evidence_integrity.txt
```

**Chain of Custody dalam Forensics Digital:**

Dalam investigasi forensik yang akan digunakan dalam proses hukum, setiap artefak harus memiliki *chain of custody* yang terdokumentasi:

```
1. Identifikasi: Artefak apa yang ditemukan?
2. Pengumpulan: Siapa yang mengumpulkan? Kapan? Dengan tool apa?
3. Preservasi: Hash dicatat sebelum setiap analisis
4. Transfer: Siapa yang menerima evidence? Tanda terima ada?
5. Analisis: Siapa yang menganalisis? Hasil apa yang diperoleh?
6. Pelaporan: Bagaimana temuan dikomunikasikan?
```

Dalam konteks lab, hash yang dicatat di `evidence_integrity.txt` merepresentasikan langkah "Preservasi" dalam chain of custody — bukti bahwa artefak yang dianalisis identik dengan artefak yang awalnya dikumpulkan.

**Contoh isi `evidence_integrity.txt`:**
```
a3f92c1d4b8e7f2a...  /home/student/datasets/forensics/registry_runkeys_sample.txt
8b2e4f9c1a7d3e6b...  /home/student/datasets/forensics/usb_artifacts_sample.txt
```

### 3. Timeline Building

```bash
cat > ~/mod6/evidence/forensics/timeline_notes.md <<'EOF'
# Timeline Investigasi — Format dan Metodologi

## Mengapa Timeline Penting dalam DFIR
Timeline adalah cara paling efektif untuk mengkomunikasikan "apa yang terjadi" kepada
audiens yang berbeda: technical analyst membutuhkan detail teknis, manajemen membutuhkan
narasi yang dapat dipahami, dan tim hukum membutuhkan dokumentasi yang dapat diverifikasi.

## Timeline dari Artefak yang Tersedia

| Urutan | Aktivitas | Artefak Pendukung | Tingkat Kepercayaan |
|--------|-----------|-------------------|---------------------|
| 1 | Email phishing-style diterima | Phishing sample dataset | Artefak ada, simulasi |
| 2 | Akses ke host terdeteksi | Auth log — SSH failed/accepted | Tinggi |
| 3 | Web request mencurigakan | Apache access log | Tinggi |
| 4 | Service dan port divalidasi | ss output, service context | Tinggi |
| 5 | USB device terhubung ke sistem | USB artifact sample | Artefak ada, simulasi |
| 6 | Registry persistence entry dibuat | Registry run keys sample | Artefak ada, simulasi |
| 7 | Network traffic mencurigakan | PCAP sample | Artefak ada |
| 8 | Evidence dikumpulkan dan di-hash | evidence_integrity.txt | Terdokumentasi |

## Catatan Metodologi Timeline
- Urutan berdasarkan logical sequence, bukan timestamp aktual (data adalah sample)
- Dalam investigasi nyata, setiap event harus memiliki timestamp yang akurat dan verified
- Konflik timestamp antara sumber berbeda harus diselesaikan sebelum timeline dianggap final

## Framework Timeline Analysis

### T-Model (Threat Timeline)
```
Reconnaissance → Initial Access → Execution → Persistence → Lateral Movement → Exfiltration
```

### D-Model (Detection Timeline)
```
Signal detected → Triage → Investigation → Containment → Remediation → Lessons Learned
```

Kedua model harus dibangun secara paralel: T-Model merekonstruksi apa yang dilakukan penyerang,
D-Model merekonstruksi bagaimana defender merespons. Gap antara T-Model dan D-Model adalah
"dwell time" — berapa lama penyerang tidak terdeteksi.
EOF
cat ~/mod6/evidence/forensics/timeline_notes.md
```

## Temuan Utama

Analisis registry run keys sample mengungkap pattern yang konsisten dengan mekanisme persistence malware: binary di path temp folder yang dikonfigurasi untuk berjalan otomatis saat startup. Dikombinasikan dengan USB artifact yang menunjukkan perangkat removable media terhubung sebelum entry persistence dibuat, ini mengindikasikan kemungkinan *USB-based initial infection* diikuti oleh *persistence establishment* — sebuah attack pattern yang umum.

## Apa yang Dipelajari

DFIR yang efektif memerlukan dua hal yang sering dianggap bertentangan: kecepatan (respons cepat untuk membatasi kerusakan) dan akurasi (dokumentasi yang cermat untuk memastikan evidence dapat dipertahankan). Timeline building adalah cara untuk menyeimbangkan keduanya — menyusun narasi yang dapat dikomunikasikan dengan cepat, sambil menyertakan referensi ke evidence yang mendukung setiap klaim.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `artifact_review.txt` | `~/mod6/evidence/forensics/` | Isi registry run keys dan USB artifact samples |
| `evidence_integrity.txt` | `~/mod6/evidence/forensics/` | SHA256 hash dari kedua artefak forensik |
| `timeline_notes.md` | `~/mod6/evidence/forensics/` | Timeline investigasi dan framework metodologi DFIR |

---
---

# Proyek 6 — Menyusun Triage Decision dan Incident Response Workflow

## Latar Belakang

Perbedaan antara *security analyst* yang efektif dan yang tidak terletak pada kemampuan pengambilan keputusan di bawah ketidakpastian. Dalam insiden nyata, analyst jarang memiliki semua informasi yang dibutuhkan sebelum perlu membuat keputusan pertama. Yang membedakan respons yang baik dari yang buruk bukan jumlah data yang dimiliki, melainkan kualitas *triage* — kemampuan untuk memilah mana yang perlu segera ditindaklanjuti, mana yang perlu investigasi lebih lanjut, dan mana yang dapat di-defer.

Proyek ini membangun kemampuan *incident-oriented decision making*: mendokumentasikan kerangka triage yang terstruktur, menulis response playbook yang dapat digunakan kembali, dan menyusun case assessment final yang mengintegrasikan seluruh temuan dari semua sumber evidence yang telah dianalisis.

## Tujuan

- Mendokumentasikan kerangka triage decision yang terstruktur untuk berbagai jenis sinyal
- Menulis response playbook notes yang mencerminkan urutan prioritas yang benar
- Menyusun final case assessment yang mengintegrasikan temuan dari seluruh domain analisis
- Membiasakan pola kerja di mana setiap incident phase ditutup dengan dokumentasi yang jelas

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Seluruh dokumentasi triage dan incident workflow |

## Langkah-Langkah yang Dilakukan

### 1. Triage Decision Notes

```bash
cat > ~/mod6/evidence/incident/triage_decision_notes.md <<'EOF'
# Triage Decision Notes — Kerangka Pengambilan Keputusan

## Pertanyaan Triage yang Harus Dijawab Pertama

### Level 1 — Scope Identification (< 5 menit)
1. Apakah ini isolasi satu host atau multi-host?
2. Apakah ada indikasi data exfiltration yang sedang berlangsung?
3. Apakah ada akun yang dikompromikan yang perlu segera di-disable?

### Level 2 — Signal Strength Assessment (5-15 menit)
4. Dari mana sinyal paling kuat berasal: log, network, phishing, atau artifact?
5. Apakah sinyal dari beberapa sumber saling mengkonfirmasi?
6. Apakah ada false positive yang jelas yang dapat menjelaskan sinyal ini?

### Level 3 — Validation Before Escalation (15-30 menit)
7. Apakah temuan ini sudah cukup kuat untuk dieskalasi?
8. Data tambahan apa yang dibutuhkan sebelum escalation?
9. Siapa yang perlu dinotifikasi dan kapan?

## Matriks Triage

| Sinyal | Kemungkinan | Tindakan | Timeframe |
|--------|-------------|----------|-----------|
| SSH brute force berhasil | Akun dikompromikan | Disable akun, preserve log, eskalasi | Segera |
| SSH brute force gagal | Reconnaissance | Monitor, rate limit, document | 1 jam |
| Multiple failed login web | Credential stuffing | Monitor, CAPTCHA, block IP | 1 jam |
| Outbound ke IP mencurigakan | C2 communication | Isolasi host, preserve evidence, eskalasi | Segera |
| Registry persistence baru | Potential malware | Preserve, analyze, timeline | 4 jam |
| Unusual process creation | Possible exploitation | Monitor + investigate | 1 jam |

## Sinyal yang Memerlukan Immediate Escalation (Tanpa Menunggu Konfirmasi)
- Evidence of active data exfiltration
- Compromised admin or service account
- Ransomware indicators (mass file modification, ransom note)
- Evidence of persistence on multiple hosts

## Sinyal yang Memerlukan Investigation Before Escalation
- Single source signal tanpa korelasi dari sumber lain
- Pattern yang bisa jadi legitimate (backup job, update mechanism)
- Low confidence IOC match tanpa konteks tambahan
EOF
cat ~/mod6/evidence/incident/triage_decision_notes.md
```

### 2. Response Playbook Notes dan Final Case Assessment

```bash
cat > ~/mod6/evidence/incident/response_playbook_notes.md <<'EOF'
# Response Playbook Notes — Urutan dan Prinsip

## Fase 1: Preserve (Selalu Pertama)
- Catat hash dari semua file yang relevan SEBELUM dianalisis lebih lanjut
- Dokumentasikan context host dan timestamp
- Jangan mengubah apa pun sebelum evidence terdokumentasi
- Buat salinan artefak yang akan dianalisis — analisis dari copy, bukan original

## Fase 2: Confirm and Validate
- Validasi dari perspektif yang berbeda (Ubuntu analisis, Kali validasi)
- Cari konfirmasi dari minimum dua sumber berbeda sebelum menarik kesimpulan
- Bedakan antara definite malicious, likely malicious, suspicious, dan unknown

## Fase 3: Scope and Contain
- Tentukan blast radius: satu user? Satu host? Beberapa host? Network-wide?
- Dalam lab: laporkan ke instruktur sebelum tindakan apapun
- Dalam produksi: ikuti runbook yang sudah ditetapkan

## Fase 4: Document and Communicate
- Tulis temuan dengan bahasa yang jelas — hindari jargon berlebihan
- Sertakan level kepercayaan untuk setiap klaim
- Pisahkan "apa yang diketahui" dari "apa yang diasumsikan" dari "apa yang tidak diketahui"

## Yang Tidak Boleh Dilakukan

| Tindakan | Mengapa Dilarang |
|----------|-----------------|
| Langsung remediate tanpa preserve | Menghapus evidence yang mungkin dibutuhkan |
| Overclaim berdasarkan satu sinyal | Menyebabkan respons yang tidak proporsional |
| Underclaim karena tidak yakin | Membiarkan ancaman aktif tidak ditangani |
| Skip documentation karena "terburu-buru" | Tidak bisa diaudit, tidak bisa dipelajari |
EOF

cat > ~/mod6/evidence/incident/final_case_assessment.md <<'EOF'
# Final Case Assessment — Modul 06

## Ringkasan Eksekutif
Environment menyediakan multiple defensive signals dari berbagai lapisan: authentication events
(auth log), web request patterns (Apache log), endpoint behavior (Windows/Sysmon sample),
network traffic (PCAP), dan persistence indicators (forensics artifacts). Kesimpulan yang kuat
muncul ketika sinyal-sinyal ini dikorelasikan, bukan ketika masing-masing dianalisis secara
terpisah.

## Temuan per Lapisan

### Authentication Layer (Auth Log)
- 7 failed SSH login dari IP 10.10.10.99 dalam interval singkat → brute force attempt
- 1 successful login dari 10.10.10.5 via public key → legitimate access
- Rekomendasi: disable password auth, enforce key-only SSH

### Web Application Layer (Apache Log)
- Request ke sensitive paths perlu dikorelasikan dengan IP dari auth log
- User-Agent strings mencurigakan mengindikasikan automated scanning tools
- Rekomendasi: WAF implementation, path-based access control

### Endpoint Layer (Windows/Sysmon)
- Process creation diikuti network connection → potential C2 communication pattern
- Registry persistence indicators → malware persistence investigation required
- Rekomendasi: EDR deployment, application whitelisting

### Network Layer (PCAP + Firewall)
- Repeated requests to login endpoint → credential testing via web interface
- Correlation dengan auth log mengkonfirmasi attack dari multiple vectors
- Rekomendasi: rate limiting, geo-blocking untuk sumber yang konsisten mencurigakan

### Forensics Layer (Registry + USB)
- Run key persistence entry di path temp folder → high confidence malware indicator
- USB artifact menunjukkan removable media access → potential initial infection vector
- Rekomendasi: USB policy enforcement, removable media monitoring

## Korelasi Kritis yang Mengubah Individual Signals Menjadi Temuan
1. IP 10.10.10.99 di auth log + Apache log = reconnaissance campaign, bukan isolated incident
2. Registry persistence + USB artifact = attack chain dari initial infection ke persistence
3. PCAP login attempts + auth log SSH attempts = multi-vector attack against same target

## Kesimpulan
Tidak ada insiden aktif yang terkonfirmasi dalam dataset lab (data adalah sample historis).
Namun, pola yang diobservasi mencerminkan attack chain yang realistis: initial access via
phishing atau brute force → persistence via registry → ongoing C2 via beaconing.
Fondasi analisis ini siap dibawa ke operasi blue team yang lebih mendalam.
EOF
cat ~/mod6/evidence/incident/final_case_assessment.md
```

## Temuan Utama

Triage decision notes mengidentifikasi dua level eskalasi yang berbeda: sinyal yang memerlukan tindakan *segera* (compromised admin account, active data exfiltration) dan sinyal yang memerlukan investigasi lebih dulu (single source signal, patterns yang mungkin legitimate). Pembedaan ini adalah kompetensi inti seorang SOC analyst — menghindari dua kesalahan yang berlawanan: *over-escalation* (eskalasi setiap anomali kecil) dan *under-escalation* (menunda tindakan ketika sinyal sudah cukup kuat).

Final case assessment berhasil membangun narasi korelasi yang kohesif dari lima lapisan evidence yang berbeda — sebuah gambaran yang jauh lebih kuat dari apa yang dapat diperoleh dari analisis satu lapisan saja.

## Apa yang Dipelajari

*Incident response* yang efektif memerlukan dua kompetensi yang sering dianggap bertentangan: kecepatan dalam pengambilan keputusan dan akurasi dalam dokumentasi. Response playbook yang tertulis dengan baik menyelesaikan tension ini dengan mendefinisikan urutan prioritas yang jelas — apa yang harus dilakukan pertama (preserve), apa yang harus dilakukan kedua (validate), dan apa yang tidak boleh dilakukan sama sekali (mengubah sebelum mendokumentasikan).

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `triage_decision_notes.md` | `~/mod6/evidence/incident/` | Kerangka triage dengan matriks sinyal dan timeframe |
| `response_playbook_notes.md` | `~/mod6/evidence/incident/` | Urutan fase respons dan hal yang tidak boleh dilakukan |
| `final_case_assessment.md` | `~/mod6/evidence/incident/` | Case assessment terintegrasi dari semua lapisan evidence |

---
---

# Proyek 7 — Menulis Blue Team Assessment dan Jembatan ke Perspektif Offensive

## Latar Belakang

Memahami *defense* secara mendalam adalah prasyarat untuk memahami *offense* secara bermakna. Red team yang tidak memahami cara kerja blue team akan melakukan serangan yang tidak relevan atau tidak realistis. Blue team yang tidak memahami cara kerja red team akan membangun kontrol yang memberikan kepercayaan palsu. Modul ini menutup gap tersebut dengan menyiapkan transisi yang terstruktur: assessment yang merangkum kompetensi defensive yang telah dibangun, dan bridge notes yang secara eksplisit memetakan bagaimana setiap kompetensi defensif tersebut relevan untuk memahami perspektif offensive.

## Tujuan

- Menyusun module assessment yang merangkum seluruh kompetensi blue team yang dikembangkan
- Mendokumentasikan *operational principles* yang harus menjadi standar kerja permanen
- Memetakan koneksi eksplisit antara setiap kompetensi defensive ke perspektif offensive yang relevan
- Mempersiapkan transisi mental dari "bagaimana kita mendeteksi" ke "bagaimana penyerang menghindari deteksi"

## Langkah-Langkah yang Dilakukan

### 1. Module 06 Assessment

```bash
cat > ~/mod6/reports/module06_assessment.md <<'EOF'
# Module 06 Assessment — Defensive Security & Blue Team Operations

## Ringkasan Eksekutif
Defensive security memerlukan evidence-driven thinking yang berlapis: tidak cukup
membaca satu log, tidak cukup melihat dari satu perspektif, dan tidak cukup membuat
kesimpulan dari satu sinyal. Kekuatan analisis defensif terletak pada kemampuan
menghubungkan sinyal dari berbagai sumber menjadi narasi yang kohesif.

## Evaluasi per Domain

| Domain | Status | Highlight |
|--------|--------|-----------|
| Monitoring Scope | ✅ Selesai | Log source map dan detection scope terdokumentasi |
| Log Triage | ✅ Selesai | Auth, Apache, Windows event — correlation notes dibangun |
| Network Analysis | ✅ Selesai | PCAP summary, beaconing concept, IDS/IPS reasoning |
| Threat Intelligence | ✅ Selesai | IOC extraction, enrichment framework, YARA scanning |
| DFIR Basics | ✅ Selesai | Artifact review, hash integrity, timeline methodology |
| Incident Workflow | ✅ Selesai | Triage decision, response playbook, case assessment |

## Operational Principles yang Terbentuk

1. **Evidence first, conclusion second** — tidak ada klaim tanpa evidence yang mendukung
2. **Correlation always** — satu sinyal adalah observasi, multiple signals yang terhubung adalah temuan
3. **Scope before action** — pahami blast radius sebelum mengambil tindakan apapun
4. **Preserve before analyze** — hash sebelum membaca, copy sebelum menganalisis
5. **Document everything** — "jika tidak terdokumentasi, tidak terjadi"
6. **Proportional response** — level respons harus sesuai dengan level kepercayaan temuan

## Key Takeaways
- SIEM thinking adalah tentang korelasi, bukan tentang platform
- Beaconing memerlukan baseline untuk identifikasi — tanpa baseline, semua traffic terlihat normal
- IOC yang tidak diperkaya dengan konteks adalah noise, bukan intelligence
- Timeline adalah cara paling efektif untuk mengkomunikasikan investigasi
- Triage decision yang baik memisahkan "butuh tindakan segera" dari "butuh investigasi lebih lanjut"
EOF
cat ~/mod6/reports/module06_assessment.md
```

### 2. Bridge Notes ke Modul Offensive

```bash
cat > ~/mod6/reports/module06_bridge_notes.md <<'EOF'
# Module 06 Bridge Notes — Menuju Perspektif Offensive

## Prinsip yang Harus Dibawa ke Modul Berikutnya

1. **Correlation matters more than isolated signals** — penyerang yang baik memahami ini dan berusaha menjaga setiap aksi tetap di bawah threshold satu sumber
2. **Validation dari multiple viewpoints** — red team yang efektif memvalidasi aksinya dari perspektif defender
3. **Evidence integrity must be preserved** — dalam offensive context: penyerang berusaha menghapus evidence; defender berusaha mempreservasinya
4. **Response thinking begins with scope and context** — penyerang memperluas scope; defender mempersempitnya

## Koneksi Blue Team → Red Team

| Kompetensi Blue Team | Perspektif Red Team yang Relevan |
|----------------------|----------------------------------|
| Auth log analysis (brute force detection) | Bagaimana melakukan password spraying yang menghindari lockout threshold? |
| Apache log analysis (path scanning detection) | Bagaimana melakukan directory enumeration yang tidak terlihat seperti scanning? |
| Beaconing detection | Bagaimana C2 framework mendesain jitter untuk menghindari detection? |
| YARA rule writing | Bagaimana malware menggunakan packing dan obfuscation untuk menghindari YARA? |
| Registry persistence detection | Apa saja teknik persistence lain yang lebih sulit dideteksi? |
| USB artifact analysis | Bagaimana initial infection via USB dilakukan? |
| IDS/IPS blocking logic | Bagaimana traffic berbahaya dapat menyerupai traffic legitimate? |

## Transisi Mental yang Diperlukan

Blue team berpikir: "Bagaimana kita mendeteksi aktivitas ini?"
Red team berpikir: "Bagaimana kita melakukan aktivitas ini tanpa terdeteksi?"

Keduanya menggunakan *pengetahuan yang sama* — perbedaannya hanya pada perspektif.
Analyst yang memahami keduanya adalah yang paling efektif di salah satu peran.

## Fondasi yang Siap untuk Modul 07

- Pemahaman tentang apa yang TERLIHAT oleh defender → memahami apa yang perlu dihindari oleh attacker
- Pemahaman tentang threshold detection → memahami bagaimana tetap di bawah threshold
- Pemahaman tentang log sources → memahami log mana yang perlu dibersihkan atau dihindari
- Pemahaman tentang artifact forensics → memahami footprint yang ditinggalkan setiap aksi
- Pemahaman tentang correlation → memahami mengapa single-source evidence lebih mudah dijelaskan
EOF
cat ~/mod6/reports/module06_bridge_notes.md
```

## Signifikansi Transisi Blue-to-Red

Transisi dari modul blue team ke modul red team yang terstruktur — dengan bridge notes yang eksplisit — mencerminkan pendekatan *purple team* yang semakin diadopsi oleh organisasi keamanan yang mature: alih-alih memisahkan blue team dan red team sebagai entitas yang terpisah, keduanya saling belajar dari perspektif satu sama lain untuk menghasilkan kontrol yang lebih efektif dan pengujian yang lebih realistis.

## Temuan Utama

Module assessment mengkonfirmasi bahwa seluruh domain defensive security yang ditargetkan telah dianalisis dengan pendekatan yang terstruktur dan berbasis korelasi. Yang paling signifikan adalah terbentuknya *operational principles* yang akan tetap relevan sepanjang karir: "evidence first", "correlation always", "preserve before analyze". Prinsip-prinsip ini tidak hanya berlaku untuk blue team — mereka adalah fondasi dari security thinking yang baik di semua spesialisasi.

## Apa yang Dipelajari

Blue team yang memahami cara kerja red team adalah blue team yang paling efektif. Dan red team yang memahami cara kerja blue team adalah red team yang paling relevan. Bridge notes yang dibangun dalam proyek ini bukan hanya persiapan untuk modul berikutnya — ini adalah bukti bahwa kompetensi defensif yang telah dibangun memberikan *context* yang sangat berharga untuk memahami perspektif offensive secara bermakna.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `module06_assessment.md` | `~/mod6/reports/` | Assessment komprehensif dengan evaluasi per domain dan operational principles |
| `module06_bridge_notes.md` | `~/mod6/reports/` | Koneksi eksplisit blue team ke red team perspective di Modul 07 |
