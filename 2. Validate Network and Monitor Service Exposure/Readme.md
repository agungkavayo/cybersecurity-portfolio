## Tentang Portofolio Ini

Portofolio ini mendokumentasikan serangkaian proyek berbasis jaringan (*networking*) yang dikerjakan secara langsung (*hands-on*) di lingkungan lab terisolasi. Setiap proyek dirancang untuk mencerminkan cara kerja seorang *security analyst* atau *network engineer* profesional — mulai dari pembacaan baseline host, validasi service exposure, troubleshooting konektivitas yang terstruktur, analisis log dan packet capture, hingga penyusunan architecture assessment yang siap dibaca secara profesional.

Seluruh aktivitas dilakukan pada dua host lab dengan peran yang berbeda:

| Host | Peran |
|------|-------|
| **Ubuntu Lab** | *Primary analysis host* — baseline host, interface review, log analysis, packet summary, architecture notes |
| **Kali Linux** | *Validation workstation* — DNS check, service validation, localhost scanning, header inspection |
| **Local Targets** | *Safe practice target* — Juice Shop (`127.0.0.1:3000`) dan DVWA (`127.0.0.1:8081`) untuk exposure validation |

> ⚠️ **Catatan:** Seluruh aktivitas dilakukan **hanya** pada lingkungan lab yang telah disediakan dan terisolasi. Tidak ada load testing, perubahan service, atau eksperimen di luar scope localhost dan dataset praktikum.

---

## Daftar Proyek

| No. | Judul Proyek | Fokus Area | Tools Utama |
|-----|-------------|------------|-------------|
| 1 | [Membangun Workspace dan Membaca Baseline Network Stack Host](#proyek-1--membangun-workspace-dan-membaca-baseline-network-stack-host) | Host baseline, network stack | `ip addr`, `ip route`, `resolvectl`, `uname` |
| 2 | [Validasi Service Exposure dari Perspektif Internal dan Eksternal](#proyek-2--validasi-service-exposure-dari-perspektif-internal-dan-eksternal) | Service exposure, port validation | `ss`, `ps`, `nmap`, `curl` |
| 3 | [Troubleshooting Konektivitas Jaringan Secara Terstruktur dan Evidence-Based](#proyek-3--troubleshooting-konektivitas-jaringan-secara-terstruktur-dan-evidence-based) | Network troubleshooting | `ping`, `traceroute`, `mtr`, `netcat` |
| 4 | [Monitoring Operasional melalui Analisis Log Sistem dan Packet Capture](#proyek-4--monitoring-operasional-melalui-analisis-log-sistem-dan-packet-capture) | Log analysis, packet review | `journalctl`, `grep`, `tshark` |
| 5 | [Penyusunan Architecture Notes dan Network Segmentation Assessment](#proyek-5--penyusunan-architecture-notes-dan-network-segmentation-assessment) | Architecture thinking, segmentation | Dokumentasi teknis |
| 6 | [Network Case Assessment dan Jembatan ke Analisis Host yang Lebih Dalam](#proyek-6--network-case-assessment-dan-jembatan-ke-analisis-host-yang-lebih-dalam) | Security assessment, reporting | Dokumentasi profesional |

---

## Teknologi dan Tools

<p>
  <img src="https://img.shields.io/badge/OS-Ubuntu%20Linux-E95420?style=flat-square&logo=ubuntu&logoColor=white"/>
  <img src="https://img.shields.io/badge/OS-Kali%20Linux-557C94?style=flat-square&logo=kalilinux&logoColor=white"/>
  <img src="https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat-square&logo=gnubash&logoColor=white"/>
  <img src="https://img.shields.io/badge/Tool-Nmap-red?style=flat-square"/>
  <img src="https://img.shields.io/badge/Tool-tshark-1679A7?style=flat-square"/>
  <img src="https://img.shields.io/badge/Tool-curl-073551?style=flat-square"/>
  <img src="https://img.shields.io/badge/Tool-mtr%20%2F%20traceroute-informational?style=flat-square"/>
  <img src="https://img.shields.io/badge/Tool-netcat-yellow?style=flat-square"/>
</p>

---

## Kompetensi yang Dibangun

- **Network Stack Analysis** — Pembacaan interface, route, dan DNS resolver dari sisi host internal secara operasional
- **Service Exposure Validation** — Membedakan perspektif internal (`ss`) dan perspektif eksternal (`nmap`, `curl`) dalam menilai service yang berjalan
- **Structured Troubleshooting** — Mendiagnosis konektivitas secara berlapis menggunakan `ping`, `traceroute`, `mtr`, dan `netcat` secara berurutan
- **Operational Monitoring** — Membaca jurnal sistem, auth log, dan packet capture untuk mengidentifikasi pola perilaku yang relevan
- **Architecture Thinking** — Memetakan aset, trust boundary, segmentasi jaringan, dan exposure berdasarkan kondisi lingkungan nyata
- **Security Assessment Writing** — Menyusun laporan assessment yang menghubungkan observasi teknis dengan rekomendasi yang dapat ditindaklanjuti

---
---

# Proyek 1 — Membangun Workspace dan Membaca Baseline Network Stack Host

## Latar Belakang

Setiap investigasi jaringan yang profesional selalu dimulai dari satu titik yang sama: memahami kondisi host itu sendiri sebelum melihat ke luar. Analis yang langsung melakukan pemindaian target tanpa terlebih dahulu memahami baseline host mereka sendiri berisiko menarik kesimpulan yang keliru — port yang tampak "terbuka dari luar" mungkin sebenarnya adalah refleksi dari konfigurasi lokal, bukan eksposur nyata.

Proyek ini menetapkan workspace kerja yang terstruktur untuk seluruh rangkaian analisis jaringan dan mengumpulkan baseline *network stack* host Ubuntu sebagai titik referensi yang akan dipakai sepanjang modul. Baseline ini mencakup identitas sistem, konfigurasi interface jaringan, tabel routing aktif, dan resolver DNS yang digunakan.

## Tujuan

- Membangun struktur workspace yang terbagi berdasarkan domain teknis: baseline, services, troubleshooting, monitoring, dan architecture
- Merekam identitas host dan waktu sistem sebagai konteks awal sebelum observasi jaringan dimulai
- Membaca konfigurasi interface, alamat IP, tabel route, dan resolver DNS dari sisi host internal
- Membiasakan pola kerja di mana seluruh observasi dimulai dari host sendiri, bukan dari asumsi tentang target

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Seluruh aktivitas baseline dilakukan di sini sebagai *primary analysis host* |

## Langkah-Langkah yang Dilakukan

### 1. Pembangunan Struktur Workspace Modul 02

Workspace dibangun dengan subdirektori yang lebih spesifik dibandingkan modul sebelumnya, mencerminkan kompleksitas domain kerja yang bertambah:

```bash
mkdir -p ~/mod2/{evidence,reports,notes,baseline,services,troubleshooting,monitoring,architecture,tmp}
cd ~/mod2
pwd
ls -la
```

**Fungsi tiap direktori:**

| Direktori | Fungsi |
|-----------|--------|
| `evidence/baseline/` | Identitas host, interface, route, dan resolver |
| `evidence/services/` | Output service exposure dan process context |
| `evidence/troubleshooting/` | Hasil ping, traceroute, mtr, dan netcat |
| `evidence/monitoring/` | Journal, auth log, dan packet summary |
| `evidence/architecture/` | Cloud mapping dan segmentation notes |
| `reports/` | Assessment akhir dan final summary |

Pemisahan direktori berdasarkan domain teknis — bukan hanya "evidence" sebagai satu tempat — mencerminkan standar kerja *security analyst* profesional yang harus dapat menelusuri evidence berdasarkan konteksnya.

### 2. Perekaman Identitas Host dan Waktu Sistem

```bash
whoami  | tee ~/mod2/evidence/baseline/whoami.txt
hostname | tee ~/mod2/evidence/baseline/hostname.txt
date    | tee ~/mod2/evidence/baseline/date.txt
uname -a | tee ~/mod2/evidence/baseline/uname.txt
```

**Mengapa timestamp penting dalam investigasi jaringan:**

Setiap event jaringan — koneksi yang masuk, paket yang ditolak, autentikasi yang gagal — memiliki timestamp. Jika analis tidak mencatat waktu sistem saat pengumpulan data, tidak ada cara untuk menyelaraskan temuan dengan event log di sistem lain. Dalam investigasi insiden, perbedaan beberapa menit antara timestamp server yang berbeda dapat menyebabkan rekonstruksi urutan kejadian yang salah.

| Perintah | Output | Relevansi |
|----------|--------|-----------|
| `whoami` | Nama user aktif | Memastikan tidak bekerja dengan hak akses yang tidak sesuai |
| `hostname` | Nama host sistem | Identifikasi host dalam laporan multi-host |
| `date` | Tanggal dan waktu sistem saat ini | Timestamp referensi untuk seluruh evidence yang dikumpulkan |
| `uname -a` | Informasi kernel dan arsitektur | Konteks OS untuk interpretasi output jaringan |

### 3. Pembacaan Network Stack Host

```bash
ip addr   | tee ~/mod2/evidence/baseline/ip_addr.txt
ip route  | tee ~/mod2/evidence/baseline/route.txt
resolvectl status | tee ~/mod2/evidence/baseline/dns_check.txt
```

**Penjelasan tiap perintah dan informasi yang dikumpulkan:**

**`ip addr` — Interface dan Alamat IP:**

`ip addr` menampilkan seluruh interface jaringan yang terpasang pada host beserta alamat IP yang dikonfigurasi. Interface yang perlu diperhatikan:

| Interface | Keterangan |
|-----------|------------|
| `lo` | *Loopback* — `127.0.0.1`, selalu ada, hanya untuk komunikasi internal host |
| `eth0` / `ens*` | Interface ethernet utama — jalur komunikasi ke jaringan lab |
| `docker0` | Interface bridge Docker — muncul jika Docker aktif di host |

**`ip route` — Tabel Routing:**

Tabel routing menentukan ke mana paket akan diteruskan berdasarkan alamat tujuan. Komponen kunci:

| Komponen | Contoh | Keterangan |
|----------|--------|------------|
| Default route | `default via 10.10.10.1 dev eth0` | Gateway untuk semua traffic yang tidak cocok dengan route spesifik |
| Local subnet | `10.10.10.0/24 dev eth0` | Traffic ke subnet ini langsung dikirim tanpa melalui gateway |
| Loopback | `127.0.0.0/8 dev lo` | Traffic ke 127.x.x.x selalu diteruskan ke loopback |

**`resolvectl status` — DNS Resolver:**

`resolvectl` menampilkan konfigurasi DNS resolver yang aktif, termasuk DNS server yang digunakan per interface dan domain search yang dikonfigurasi. Informasi ini penting karena konfigurasi DNS yang berbeda antara interface dapat menyebabkan hasil resolusi yang tidak konsisten.

## Hasil dan Output

**Contoh output `ip_addr.txt`:**
```
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500
    link/ether 00:11:22:33:44:55 brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.21/24 brd 10.10.10.255 scope global eth0
```

**Contoh output `route.txt`:**
```
default via 10.10.10.1 dev eth0 proto dhcp
10.10.10.0/24 dev eth0 proto kernel scope link
127.0.0.0/8 dev lo scope host
```

**Contoh output `dns_check.txt`:**
```
Link 2 (eth0)
  Current DNS Server: 10.10.10.1
  DNS Servers: 10.10.10.1 8.8.8.8
```

## Temuan Utama

Baseline network stack mengungkap bahwa host Ubuntu menggunakan DNS resolver yang disediakan oleh gateway lab (`10.10.10.1`) sebagai server utama, dengan fallback ke DNS publik (`8.8.8.8`). Konfigurasi seperti ini relevan dalam konteks keamanan karena traffic DNS yang melalui resolver internal dapat dipantau atau dimodifikasi oleh administrator jaringan — informasi yang penting untuk diketahui sebelum melakukan validasi resolusi nama.

## Apa yang Dipelajari

Troubleshooting dan investigasi jaringan yang benar selalu dimulai dari host sendiri. Mengetahui interface mana yang aktif, route apa yang dikonfigurasi, dan DNS server mana yang digunakan adalah prasyarat untuk menginterpretasikan hasil tool jaringan lainnya dengan benar. Tanpa baseline ini, analis tidak dapat membedakan apakah sebuah anomali berasal dari konfigurasi host lokal atau dari infrastruktur jaringan yang lebih luas.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `whoami.txt` | `~/mod2/evidence/baseline/` | User aktif saat pengumpulan data |
| `hostname.txt` | `~/mod2/evidence/baseline/` | Nama host sistem |
| `date.txt` | `~/mod2/evidence/baseline/` | Timestamp referensi |
| `uname.txt` | `~/mod2/evidence/baseline/` | Informasi kernel dan arsitektur |
| `ip_addr.txt` | `~/mod2/evidence/baseline/` | Konfigurasi interface dan alamat IP |
| `route.txt` | `~/mod2/evidence/baseline/` | Tabel routing aktif |
| `dns_check.txt` | `~/mod2/evidence/baseline/` | Konfigurasi DNS resolver |

---
---

# Proyek 2 — Validasi Service Exposure dari Perspektif Internal dan Eksternal

## Latar Belakang

Salah satu kesalahan paling umum dalam penilaian keamanan adalah menyimpulkan bahwa sebuah service "aman" hanya karena tidak terlihat dari satu sudut pandang tertentu. Kenyataannya, perspektif internal host dan perspektif eksternal workstation dapat memberikan gambaran yang sangat berbeda tentang service yang sama. Sebuah port yang tampak tertutup dari dalam host mungkin terekspos ke jaringan, dan sebaliknya — port yang terbuka secara internal mungkin diblokir oleh firewall sebelum mencapai jaringan eksternal.

Proyek ini mempraktikkan pembacaan service exposure dari dua sudut pandang secara bersamaan: perspektif internal menggunakan `ss` dan `ps` pada Ubuntu, dan perspektif eksternal menggunakan `nmap` dan `curl` dari Kali Linux sebagai *validation workstation*.

## Tujuan

- Membaca port yang sedang *listening* beserta proses yang memilikinya dari sisi internal host
- Memvalidasi bagaimana service tersebut terlihat dari workstation eksternal menggunakan `nmap` dan `curl`
- Mengidentifikasi informasi yang hanya dapat dilihat dari perspektif internal dan yang hanya dari perspektif eksternal
- Mendokumentasikan kedua perspektif dalam format yang dapat digunakan sebagai dasar security assessment

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Membaca state internal — port listening dan process context |
| **Kali Linux** | Memvalidasi exposure — nmap scan dan HTTP header inspection |

## Langkah-Langkah yang Dilakukan

### 1. Pembacaan Port Listening dan Process Context dari Ubuntu

```bash
ss -tulpn | tee ~/mod2/evidence/services/ss_output.txt
ps -ef | grep -E "nginx|ssh|docker|python" | grep -v grep | tee ~/mod2/evidence/services/process_context.txt
```

**Penjelasan flag `ss -tulpn`:**

| Flag | Fungsi |
|------|--------|
| `-t` | Tampilkan socket TCP |
| `-u` | Tampilkan socket UDP |
| `-l` | Hanya socket yang sedang *listening* (menunggu koneksi masuk) |
| `-p` | Tampilkan nama dan PID proses yang memiliki socket |
| `-n` | Tampilkan alamat dan port dalam format numerik tanpa resolusi nama |

**Mengapa process context sama pentingnya dengan daftar port:**

Daftar port saja hanya menunjukkan *apa* yang berjalan. Process context (`ps -ef`) menunjukkan *siapa* yang menjalankannya — informasi yang krusial dalam security assessment. Sebuah port 8080 yang dimiliki oleh proses `python3 -m http.server` memiliki implikasi keamanan yang sangat berbeda dibandingkan dengan port 8080 yang dimiliki oleh `nginx` yang dikonfigurasi dengan benar.

**Pola filter yang digunakan:**

```bash
ps -ef | grep -E "nginx|ssh|docker|python" | grep -v grep
```

Operator `grep -v grep` digunakan untuk mengeluarkan proses `grep` itu sendiri dari hasil, sehingga output hanya berisi proses yang benar-benar relevan.

### 2. Validasi Service dari Kali sebagai Assessment Workstation

```bash
~/lab-targets-info
nmap -Pn -sV -p 3000,8081 127.0.0.1 -oN ~/mod2/evidence/services/nmap_local_targets.txt
curl -I http://127.0.0.1:3000 | tee  ~/mod2/evidence/services/http_headers_local.txt
curl -I http://127.0.0.1:8081 | tee -a ~/mod2/evidence/services/http_headers_local.txt
```

**Penjelasan flag `nmap`:**

| Flag | Fungsi |
|------|--------|
| `-Pn` | Skip host discovery — asumsikan host aktif tanpa mengirim ping |
| `-sV` | *Service version detection* — identifikasi aplikasi dan versinya |
| `-p 3000,8081` | Hanya pindai port yang ditentukan |
| `-oN` | Simpan output dalam format normal (human-readable) ke file |

**Informasi yang dapat diekstrak dari HTTP header (`curl -I`):**

| Header | Informasi yang Didapat | Relevansi Keamanan |
|--------|------------------------|-------------------|
| `Server` | Nama dan versi web server | Fingerprinting — dasar pencarian known vulnerabilities |
| `X-Powered-By` | Framework atau runtime yang digunakan | Mengekspos stack teknologi backend |
| `Set-Cookie` | Atribut cookie yang dikonfigurasi | Memeriksa `HttpOnly`, `Secure`, `SameSite` |
| `Content-Security-Policy` | Kebijakan keamanan konten | Menilai proteksi terhadap XSS |
| `X-Frame-Options` | Kebijakan framing halaman | Proteksi terhadap clickjacking |
| `Strict-Transport-Security` | Kebijakan HTTPS enforcement | Apakah HSTS diterapkan |

### 3. Dokumentasi Service Context Note

```bash
cat > ~/mod2/evidence/services/service_context_notes.md <<'EOF'
# Service Context Notes

## Perspektif Internal (Ubuntu — ss + ps)
- ss menampilkan port yang listening beserta PID dan nama proses pemiliknya
- ps memberikan konteks siapa yang menjalankan service tersebut
- Perspektif ini tidak dapat dilihat dari luar tanpa akses ke sistem

## Perspektif Eksternal (Kali — nmap + curl)
- nmap menunjukkan port mana yang benar-benar dapat dijangkau dari workstation
- curl mengungkap header HTTP yang diekspos oleh service
- Perspektif ini mencerminkan apa yang dapat dilihat oleh penyerang atau assessor

## Mengapa Kedua Perspektif Harus Selalu Dibandingkan
- Perbedaan antara keduanya mengindikasikan adanya firewall, NAT, atau konfigurasi binding
- Kesamaan antara keduanya mengkonfirmasi bahwa service benar-benar terekspos sesuai yang terlihat
EOF
```

## Hasil dan Output

**Contoh output `ss_output.txt`:**
```
Netid  State   Recv-Q Send-Q  Local Address:Port   Peer Address:Port  Process
tcp    LISTEN  0      128     0.0.0.0:22            0.0.0.0:*          users:(("sshd",pid=1234))
tcp    LISTEN  0      511     0.0.0.0:3000          0.0.0.0:*          users:(("node",pid=5678))
tcp    LISTEN  0      511     0.0.0.0:8081          0.0.0.0:*          users:(("nginx",pid=9012))
```

**Contoh output `nmap_local_targets.txt`:**
```
PORT     STATE SERVICE VERSION
3000/tcp open  http    Node.js Express framework
8081/tcp open  http    nginx 1.18.0
```

**Contoh output `http_headers_local.txt`:**
```
HTTP/1.1 200 OK
X-Powered-By: Express
X-Frame-Options: SAMEORIGIN
Set-Cookie: session=abc123; Path=/; HttpOnly
```

## Temuan Utama

Perbandingan antara output `ss` dan `nmap` mengkonfirmasi bahwa port 3000 dan 8081 benar-benar dapat dijangkau dari workstation validasi — tidak ada firewall yang memblokir di antara keduanya dalam lingkungan lab ini. Header `X-Powered-By: Express` yang terekspos pada port 3000 merupakan temuan yang patut dicatat: dalam lingkungan produksi, header ini sebaiknya dinonaktifkan karena memberikan informasi fingerprinting kepada penyerang.

Dari output `ss`, terlihat bahwa service pada port 3000 dijalankan oleh proses `node` — informasi yang tidak dapat diperoleh dari pemindaian `nmap` saja. Kombinasi keduanya memberikan gambaran lengkap tentang service yang berjalan.

## Apa yang Dipelajari

Service exposure validation yang benar membutuhkan dua perspektif. Perspektif internal memberikan informasi tentang *siapa* yang menjalankan service dan *bagaimana* konfigurasinya. Perspektif eksternal memberikan informasi tentang *apa* yang dapat dilihat dan dijangkau oleh pihak di luar sistem. Menggunakan hanya satu perspektif dapat menyebabkan penilaian risiko yang tidak akurat.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `ss_output.txt` | `~/mod2/evidence/services/` | Port listening dan process context dari Ubuntu |
| `process_context.txt` | `~/mod2/evidence/services/` | Detail proses service yang relevan |
| `nmap_local_targets.txt` | `~/mod2/evidence/services/` | Hasil nmap scan dari Kali |
| `http_headers_local.txt` | `~/mod2/evidence/services/` | HTTP response header dari kedua target lokal |
| `service_context_notes.md` | `~/mod2/evidence/services/` | Catatan perbandingan perspektif internal dan eksternal |

---
---

# Proyek 3 — Troubleshooting Konektivitas Jaringan Secara Terstruktur dan Evidence-Based

## Latar Belakang

Troubleshooting jaringan yang tidak terstruktur adalah sumber pemborosan waktu terbesar dalam operasi IT. Analis yang langsung mencoba berbagai solusi tanpa diagnosis yang sistematis berisiko memperbaiki hal yang bukan masalahnya, atau lebih buruk lagi — mengubah konfigurasi yang berfungsi dengan baik dan menciptakan masalah baru.

Proyek ini membangun dan mempraktikkan metode troubleshooting yang berlapis (*layered approach*): dimulai dari *loopback* untuk memverifikasi local stack, kemudian ke IP eksternal untuk memverifikasi konektivitas keluar, lalu ke nama domain untuk memverifikasi resolusi DNS, dan terakhir menggunakan `traceroute`, `mtr`, serta `netcat` untuk mendiagnosis masalah yang lebih spesifik. Setiap langkah memiliki tujuan yang jelas dan hasilnya disimpan sebagai evidence yang dapat dianalisis.

## Tujuan

- Membangun kebiasaan troubleshooting yang berlapis, terstruktur, dan evidence-based
- Membedakan failure domain: apakah masalah ada di local stack, routing, atau resolusi nama
- Menggunakan `traceroute` dan `mtr` untuk memetakan jalur jaringan dan kualitas tiap hop
- Menggunakan `netcat` untuk validasi port-level connectivity yang cepat dan tepat
- Mendokumentasikan temuan dalam route diagnosis yang dapat dibaca dan direproduksi

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Seluruh aktivitas troubleshooting dijalankan dari sini |

## Konsep Dasar: Troubleshooting Berlapis

Pendekatan berlapis dalam troubleshooting jaringan mengikuti model yang sederhana namun sangat efektif:

```
Layer 1: Local stack     → ping 127.0.0.1
         (apakah TCP/IP stack host berfungsi?)
             ↓
Layer 2: IP connectivity → ping 8.8.8.8
         (apakah paket dapat keluar dari host?)
             ↓
Layer 3: DNS resolution  → ping example.com
         (apakah nama domain dapat diresolvasi?)
             ↓
Layer 4: Path quality    → traceroute + mtr
         (di mana loss atau latency terjadi?)
             ↓
Layer 5: Port validation → netcat
         (apakah port tujuan benar-benar menerima koneksi?)
```

Dengan pendekatan ini, begitu satu layer gagal, analis dapat fokus pada layer tersebut tanpa perlu memeriksa layer di atasnya.

## Langkah-Langkah yang Dilakukan

### 1. Ping Matrix — Memisahkan Lapisan Masalah

```bash
printf "=== LOCALHOST ===\n"     | tee  ~/mod2/evidence/troubleshooting/ping_matrix.txt
ping -c 4 127.0.0.1             | tee -a ~/mod2/evidence/troubleshooting/ping_matrix.txt

printf "\n=== EXTERNAL IP ===\n" | tee -a ~/mod2/evidence/troubleshooting/ping_matrix.txt
ping -c 4 8.8.8.8               | tee -a ~/mod2/evidence/troubleshooting/ping_matrix.txt

printf "\n=== DNS NAME ===\n"    | tee -a ~/mod2/evidence/troubleshooting/ping_matrix.txt
ping -c 4 example.com           | tee -a ~/mod2/evidence/troubleshooting/ping_matrix.txt
```

**Interpretasi hasil ping matrix:**

| Skenario | Localhost | External IP | DNS Name | Diagnosis |
|----------|-----------|-------------|----------|-----------|
| Semua berhasil | ✅ | ✅ | ✅ | Konektivitas penuh — masalah bukan di sini |
| Hanya localhost berhasil | ✅ | ❌ | ❌ | Masalah routing atau gateway — tidak ada jalur keluar |
| Localhost & IP berhasil, DNS gagal | ✅ | ✅ | ❌ | Masalah DNS resolver — konektivitas ada tapi nama tidak bisa diresolvasi |
| Semua gagal | ❌ | ❌ | ❌ | Local stack bermasalah atau interface down |

### 2. Pemetaan Jalur dengan Traceroute dan MTR

```bash
traceroute 8.8.8.8 | tee ~/mod2/evidence/troubleshooting/traceroute_test.txt
mtr -rw -c 10 8.8.8.8 | tee ~/mod2/evidence/troubleshooting/mtr_test.txt
```

**Perbedaan `traceroute` dan `mtr`:**

| Aspek | `traceroute` | `mtr` |
|-------|-------------|-------|
| Cara kerja | Mengirim satu paket per hop, menampilkan hasilnya | Mengirim paket berulang ke setiap hop, mengukur statistik |
| Output | Daftar hop dengan RTT | Statistik per hop: loss%, avg, best, worst latency |
| Kegunaan | Memetakan jalur jaringan | Mendiagnosis intermittent loss atau jitter pada hop tertentu |
| Flag `-rw -c 10` | — | Report mode, wide output, 10 siklus pengukuran |

**Penjelasan flag `mtr -rw -c 10`:**

| Flag | Fungsi |
|------|--------|
| `-r` | *Report mode* — jalankan pengukuran dan tampilkan hasilnya setelah selesai |
| `-w` | *Wide* — tampilkan nama host penuh tanpa pemotongan |
| `-c 10` | Kirim 10 paket ke setiap hop untuk mendapatkan statistik yang lebih representatif |

**Cara membaca output MTR:**

```
Host                    Loss%   Snt   Last   Avg  Best  Wrst StDev
1. 10.10.10.1           0.0%    10    1.2    1.1   0.9   1.5   0.2
2. 192.168.1.1          0.0%    10    5.3    5.1   4.8   5.9   0.3
3. ???                  100.0%  10    0.0    0.0   0.0   0.0   0.0
4. 8.8.8.8              0.0%    10   12.4   12.1  11.8  12.9   0.3
```

Hop 3 menunjukkan `100%` loss namun hop 4 berhasil dicapai — ini adalah perilaku normal. Banyak router dikonfigurasi untuk tidak merespons ICMP TTL-exceeded (yang digunakan oleh `mtr`) namun tetap meneruskan paket. Ini bukan berarti ada masalah.

### 3. Validasi Port dengan Netcat

```bash
nc -vz 127.0.0.1 3000 | tee  ~/mod2/evidence/troubleshooting/nc_validation.txt
nc -vz 127.0.0.1 8081 | tee -a ~/mod2/evidence/troubleshooting/nc_validation.txt
```

**Penjelasan flag `nc -vz`:**

| Flag | Fungsi |
|------|--------|
| `-v` | *Verbose* — tampilkan detail koneksi |
| `-z` | *Zero-I/O mode* — hanya uji koneksi tanpa mengirim data |

**Interpretasi output netcat:**

| Output | Artinya |
|--------|---------|
| `Connection to 127.0.0.1 3000 port [tcp/*] succeeded!` | Port terbuka dan menerima koneksi TCP |
| `nc: connect to 127.0.0.1 port 3000 (tcp) failed: Connection refused` | Port tidak ada yang listen atau firewall menolak koneksi |
| *(tidak ada output, timeout)* | Port difilter — koneksi tidak diterima dan tidak ditolak secara eksplisit |

**Kapan menggunakan `netcat` vs `nmap`:**

`netcat` lebih ringan dan lebih cepat untuk validasi port tunggal. `nmap` lebih komprehensif untuk pemindaian multiple port dengan identifikasi service. Dalam alur kerja troubleshooting, `netcat` digunakan untuk konfirmasi cepat sebelum melanjutkan ke investigasi yang lebih dalam.

### 4. Route Diagnosis

```bash
cat > ~/mod2/evidence/troubleshooting/route_diagnosis.md <<'EOF'
# Route Diagnosis

## Observasi
- Localhost (127.0.0.1): dapat dijangkau — local stack berfungsi normal
- External IP (8.8.8.8): dapat dijangkau — routing keluar berfungsi
- DNS name (example.com): dapat dijangkau — resolusi nama berfungsi
- Traceroute: jalur keluar melewati gateway lab sebelum menuju internet
- MTR: tidak ada packet loss yang signifikan pada hop manapun

## Interpretasi Awal
Tidak ada failure domain yang terdeteksi pada sesi ini. Konektivitas berjalan normal
di seluruh lapisan: local stack, IP routing, dan DNS resolution.

## Catatan untuk Investigasi Lanjutan
- Jika ping localhost gagal: periksa network stack OS dengan `ip link show lo`
- Jika ping IP gagal tapi localhost OK: periksa default route dengan `ip route`
- Jika ping DNS gagal tapi IP OK: periksa resolver dengan `resolvectl status`
- Jika ada packet loss di MTR: identifikasi hop bermasalah dan laporkan ke tim jaringan
EOF
```

## Hasil dan Output

**Contoh output `ping_matrix.txt`:**
```
=== LOCALHOST ===
PING 127.0.0.1 (127.0.0.1): 56 bytes of data
64 bytes from 127.0.0.1: icmp_seq=0 ttl=64 time=0.045 ms
4 packets transmitted, 4 received, 0% packet loss

=== EXTERNAL IP ===
PING 8.8.8.8: 56 bytes of data
64 bytes from 8.8.8.8: icmp_seq=0 ttl=118 time=12.4 ms
4 packets transmitted, 4 received, 0% packet loss

=== DNS NAME ===
PING example.com (93.184.216.34): 56 bytes of data
64 bytes from 93.184.216.34: icmp_seq=0 ttl=56 time=45.2 ms
4 packets transmitted, 4 received, 0% packet loss
```

**Contoh output `nc_validation.txt`:**
```
Connection to 127.0.0.1 3000 port [tcp/*] succeeded!
Connection to 127.0.0.1 8081 port [tcp/*] succeeded!
```

## Temuan Utama

Dari ping matrix, ketiga lapisan konektivitas berjalan normal: local stack, IP routing, dan DNS resolution semuanya berfungsi. Perbedaan latency yang signifikan antara `ping 127.0.0.1` (~0.04ms) dan `ping example.com` (~45ms) adalah perilaku yang diharapkan — loopback tidak melalui jaringan fisik, sementara traffic ke internet melewati beberapa hop routing.

Validasi `netcat` mengkonfirmasi bahwa port 3000 dan 8081 benar-benar menerima koneksi TCP — bukan hanya "open" menurut pemindaian nmap, tetapi benar-benar responsif terhadap koneksi masuk.

## Apa yang Dipelajari

Troubleshooting yang terstruktur secara drastis mengurangi waktu yang dibutuhkan untuk menemukan akar masalah. Dengan mengikuti urutan lapisan dari dalam keluar, analis dapat dengan cepat mengisolasi *failure domain* dan fokus pada area yang bermasalah. Pola berpikir ini — sistematis, berlapis, evidence-based — adalah fondasi dari *network operations* dan *incident response* profesional.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `ping_matrix.txt` | `~/mod2/evidence/troubleshooting/` | Hasil ping ke tiga target berbeda |
| `traceroute_test.txt` | `~/mod2/evidence/troubleshooting/` | Peta jalur jaringan ke IP eksternal |
| `mtr_test.txt` | `~/mod2/evidence/troubleshooting/` | Statistik kualitas per hop |
| `nc_validation.txt` | `~/mod2/evidence/troubleshooting/` | Hasil validasi port-level dengan netcat |
| `route_diagnosis.md` | `~/mod2/evidence/troubleshooting/` | Interpretasi dan diagnosis tertulis |

---
---

# Proyek 4 — Monitoring Operasional melalui Analisis Log Sistem dan Packet Capture

## Latar Belakang

Monitoring yang efektif bukan tentang mengumpulkan sebanyak mungkin log — ini tentang mengetahui *di mana* harus mencari dan *apa* yang harus dicari. Dalam sebuah sistem Linux yang berjalan normal, ratusan event dicatat setiap menit. Tugas seorang *security analyst* adalah menyaring noise tersebut dan mengidentifikasi sinyal yang relevan: autentikasi yang gagal, service yang crash, koneksi yang tidak biasa, atau traffic yang tidak sesuai pola.

Proyek ini mempraktikkan tiga sumber monitoring yang saling melengkapi: journal sistem untuk event OS secara real-time, auth log untuk pola autentikasi, dan packet capture (PCAP) untuk observasi traffic di level jaringan. Kombinasi ketiganya memberikan gambaran yang jauh lebih komprehensif dibandingkan mengandalkan satu sumber saja.

## Tujuan

- Membaca journal sistem untuk mengidentifikasi event operasional yang relevan
- Menganalisis auth log sample untuk menemukan pola autentikasi yang mencurigakan
- Meninjau packet capture sample untuk memahami traffic yang melewati jaringan lab
- Menghubungkan temuan dari ketiga sumber monitoring menjadi konteks yang saling melengkapi

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Seluruh analisis monitoring dilakukan di sini |
| **Dataset `/opt/jccsah`** | Sumber PCAP sample dan auth log sample |

## Langkah-Langkah yang Dilakukan

### 1. Review Journal Sistem

```bash
journalctl -n 50 --no-pager | tee ~/mod2/evidence/monitoring/journal_review.txt
```

**Penjelasan flag `journalctl`:**

| Flag | Fungsi |
|------|--------|
| `-n 50` | Tampilkan 50 entri terbaru |
| `--no-pager` | Tampilkan output langsung ke terminal tanpa pagination |

**Jenis event yang perlu diperhatikan dalam journal:**

| Kata Kunci | Sumber Umum | Relevansi Keamanan |
|------------|-------------|-------------------|
| `Failed` | SSH, PAM, service | Percobaan akses yang gagal |
| `Started` / `Stopped` | systemd | Service yang baru berjalan atau dihentikan |
| `segfault` | kernel | Crash aplikasi yang bisa mengindikasikan eksploitasi |
| `UFW BLOCK` | firewall | Traffic yang diblokir |
| `CRON` | cron daemon | Task terjadwal yang berjalan |
| `sudo` | sudo | Penggunaan hak akses elevated |

### 2. Analisis Auth Log Sample

```bash
grep -i "failed\|accepted" ~/datasets/logs/linux/auth.log.sample | \
    tee ~/mod2/evidence/monitoring/auth_log_review.txt
```

**Mengapa `auth.log` adalah sumber penting dalam security monitoring:**

`auth.log` mencatat seluruh aktivitas autentikasi pada sistem Linux: login SSH, penggunaan `sudo`, autentikasi PAM, dan perubahan password. File ini adalah sumber pertama yang diperiksa dalam investigasi *unauthorized access* atau *brute force attack*.

**Pola yang dicari:**

| Pola | Contoh Entry | Indikasi |
|------|-------------|----------|
| Failed login berulang | `Failed password for root from 10.10.10.99 port 22` | Brute force SSH |
| Login berhasil setelah beberapa kali gagal | `Accepted password for admin` (setelah 5x Failed) | Credential stuffing berhasil |
| Login dari IP tidak dikenal | `Accepted publickey for user from 203.x.x.x` | Akses dari lokasi baru |
| Invalid user | `Invalid user admin from 10.10.10.99` | Enumerasi username |

**Contoh output yang dihasilkan:**
```bash
grep -i "failed\|accepted" auth.log.sample
```
```
May 15 10:23:11 ubuntu sshd[1234]: Failed password for root from 10.10.10.99 port 45231 ssh2
May 15 10:23:14 ubuntu sshd[1234]: Failed password for root from 10.10.10.99 port 45234 ssh2
May 15 10:23:17 ubuntu sshd[1234]: Failed password for root from 10.10.10.99 port 45237 ssh2
May 15 10:25:03 ubuntu sshd[1235]: Accepted publickey for student from 10.10.10.5 port 52100 ssh2
```

Tiga baris `Failed password` berturut-turut dari IP yang sama dalam selang waktu 3 detik adalah indikasi klasik *SSH brute force attack*.

### 3. Tinjauan PCAP Sample dengan tshark

```bash
tshark -r ~/datasets/pcap/localhost_web_lab.pcap | head -n 30 | \
    tee ~/mod2/evidence/monitoring/pcap_summary.txt
```

**Mengapa tshark digunakan alih-alih tcpdump:**

`tshark` adalah versi command-line dari Wireshark yang mampu melakukan *protocol dissection* secara otomatis. Output `tshark` jauh lebih mudah dibaca karena sudah menampilkan nama protokol, alamat sumber/tujuan, dan informasi layer aplikasi dalam format yang terstruktur.

**Informasi yang dapat diekstrak dari PCAP:**

| Kolom | Contoh | Keterangan |
|-------|--------|------------|
| Timestamp | `0.000000` | Waktu relatif paket dalam capture |
| Source | `127.0.0.1` | Alamat IP sumber |
| Destination | `127.0.0.1` | Alamat IP tujuan |
| Protocol | `HTTP`, `TCP`, `TLS` | Protokol yang digunakan |
| Info | `GET /login HTTP/1.1` | Detail payload (untuk protokol yang tidak terenkripsi) |

**Pola yang dicari dalam PCAP lab:**

- Repeated requests ke endpoint yang sama dalam waktu singkat (indikasi *scanning* atau *fuzzing*)
- Request ke path yang tidak umum (indikasi *directory traversal* atau *path enumeration*)
- Response code yang tidak normal (401 berulang = brute force, 500 = error yang mungkin disebabkan input berbahaya)
- Traffic yang tidak terduga ke port atau protokol yang seharusnya tidak aktif

### 4. Catatan Packet Notes

```bash
cat > ~/mod2/evidence/monitoring/packet_notes.md <<'EOF'
# Packet Notes

## Sumber Data
- PCAP sample dari dataset lab (bukan live capture)
- Aman digunakan karena tidak mengandung traffic jaringan produksi

## Fokus Analisis
- Port tujuan: mengidentifikasi service yang diakses
- Protocol flow: memahami urutan request-response
- Repeated request pattern: mendeteksi scanning atau brute force
- Response code: mengidentifikasi error atau anomali

## Hubungan dengan Sumber Monitoring Lain
- PCAP memberikan bukti traffic level jaringan
- Auth log memberikan bukti aktivitas autentikasi
- Journal memberikan bukti event OS
- Ketiganya saling melengkapi untuk rekonstruksi kejadian yang komprehensif
EOF
```

## Hasil dan Output

**Contoh output `pcap_summary.txt`:**
```
    1   0.000000 127.0.0.1 → 127.0.0.1    TCP 74 54321 → 3000 [SYN]
    2   0.000234 127.0.0.1 → 127.0.0.1    TCP 74 3000 → 54321 [SYN, ACK]
    3   0.000298 127.0.0.1 → 127.0.0.1    TCP 66 54321 → 3000 [ACK]
    4   0.001123 127.0.0.1 → 127.0.0.1    HTTP 234 GET / HTTP/1.1
    5   0.002891 127.0.0.1 → 127.0.0.1    HTTP 1098 HTTP/1.1 200 OK
```

Output ini menunjukkan TCP three-way handshake (SYN, SYN-ACK, ACK) yang normal, diikuti oleh HTTP GET request dan respons 200 OK — pola koneksi web yang sehat.

## Temuan Utama

Dari analisis auth log sample, ditemukan pola *brute force SSH* dari IP `10.10.10.99` dengan tiga percobaan gagal dalam interval 3 detik, sebelum akhirnya terdapat login berhasil menggunakan public key dari IP yang berbeda (`10.10.10.5`). Dua kejadian ini tidak terhubung — login berhasil menggunakan metode yang berbeda (public key vs password) dari IP yang berbeda — namun keduanya perlu didokumentasikan dalam konteks yang sama.

Analisis PCAP mengkonfirmasi traffic normal ke port 3000, konsisten dengan hasil validasi service yang dilakukan pada proyek sebelumnya.

## Apa yang Dipelajari

Monitoring yang efektif membutuhkan triangulasi dari beberapa sumber. Journal memberikan konteks OS-level, auth log memberikan konteks autentikasi, dan PCAP memberikan konteks traffic jaringan. Temuan yang hanya muncul di satu sumber mungkin merupakan false positive; temuan yang dikonfirmasi oleh dua atau lebih sumber memiliki tingkat kepercayaan yang jauh lebih tinggi.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `journal_review.txt` | `~/mod2/evidence/monitoring/` | 50 entri jurnal sistem terbaru |
| `auth_log_review.txt` | `~/mod2/evidence/monitoring/` | Baris failed dan accepted dari auth log sample |
| `pcap_summary.txt` | `~/mod2/evidence/monitoring/` | Ringkasan 30 paket pertama dari PCAP sample |
| `packet_notes.md` | `~/mod2/evidence/monitoring/` | Catatan konteks dan metodologi analisis packet |

---
---

# Proyek 5 — Penyusunan Architecture Notes dan Network Segmentation Assessment

## Latar Belakang

Pemahaman tentang jaringan yang berhenti di level interface dan port adalah pemahaman yang tidak lengkap. Seorang *security professional* yang berpengalaman tidak hanya tahu *bagaimana* jaringan terhubung, tetapi juga *mengapa* jaringan dikonfigurasi seperti itu, *di mana* batas kepercayaan (*trust boundary*) berada, dan *apa* risiko yang muncul dari konfigurasi tersebut.

Proyek ini mengubah observasi teknis yang telah dikumpulkan menjadi dokumentasi arsitektur yang dapat dikomunikasikan — sebuah keterampilan yang membedakan analis yang hanya bisa "menjalankan tool" dari analis yang dapat memberikan nilai nyata dalam diskusi desain sistem, tinjauan keamanan, atau incident response.

## Tujuan

- Memetakan aset dalam lingkungan lab beserta peran dan exposure masing-masing
- Mengidentifikasi trust boundary dan segmentasi jaringan berdasarkan kondisi nyata
- Menulis rekomendasi segmentasi yang berbasis observasi, bukan kalimat generik
- Membiasakan pola berpikir architecture yang akan terus relevan dalam konteks cloud dan enterprise

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Seluruh dokumentasi arsitektur disusun di sini |

## Konsep Dasar: Trust Boundary dan Segmentasi

**Trust boundary** adalah batas antara dua zona jaringan yang memiliki tingkat kepercayaan berbeda. Traffic yang melewati trust boundary memerlukan kontrol tambahan — firewall, autentikasi, atau enkripsi — karena data berpindah dari zona yang lebih dipercaya ke zona yang kurang dipercaya, atau sebaliknya.

**Segmentasi jaringan** adalah praktik memisahkan jaringan menjadi zona-zona yang lebih kecil untuk membatasi dampak jika salah satu zona dikompromikan (*blast radius reduction*).

## Langkah-Langkah yang Dilakukan

### 1. Cloud Mapping — Pemetaan Aset dan Exposure

```bash
cat > ~/mod2/evidence/architecture/cloud_mapping.md <<'EOF'
# Cloud and Virtual Network Mapping

## Inventaris Aset
| Aset | Alamat | Peran | Exposure Level |
|------|--------|-------|----------------|
| Ubuntu Lab | 10.10.10.21 | Primary analysis host | Internal only |
| Kali Linux | 10.10.10.22 | Validation workstation | Internal only |
| Juice Shop | 127.0.0.1:3000 | Target latihan web | Localhost only |
| DVWA | 127.0.0.1:8081 | Target latihan web | Localhost only |

## Jalur Akses yang Sah
- SSH: student → Ubuntu Lab dan Kali Linux (via password/key)
- Web targets: hanya dapat diakses dari localhost (tidak terekspos ke jaringan)

## Catatan Exposure
- Web targets yang terikat ke 127.0.0.1 tidak dapat dijangkau dari luar host
- Jika target terikat ke 0.0.0.0, seluruh interface termasuk jaringan lab dapat mengaksesnya
- Perbedaan binding address ini adalah perbedaan keamanan yang signifikan
EOF
```

**Mengapa perbedaan binding address penting:**

| Binding | Dapat diakses dari | Implikasi Keamanan |
|---------|-------------------|-------------------|
| `127.0.0.1:3000` | Hanya dari localhost | Aman — tidak terekspos ke jaringan |
| `0.0.0.0:3000` | Dari semua interface termasuk jaringan | Terekspos — semua host di jaringan dapat mengakses |

### 2. Network Segmentation Notes

```bash
cat > ~/mod2/evidence/architecture/network_segmentation_notes.md <<'EOF'
# Network Segmentation Notes

## Kondisi Saat Ini

### Isolasi User
- Akun student terisolasi di level home directory
- Tidak ada akses sudo — tindakan yang berdampak sistem memerlukan eskalasi ke instruktur
- Berbagi resource server (CPU, memori, disk) berarti aktivitas satu user dapat memengaruhi user lain

### Segmentasi Target
- Web targets sengaja terikat ke localhost untuk mencegah eksposur tidak sengaja ke jaringan lab
- Pemisahan peran Ubuntu (analisis) dan Kali (validasi) menciptakan logical boundary antar workstation

### Keterbatasan Segmentasi Saat Ini
- Tidak ada firewall yang memisahkan antar user di host yang sama
- Log dan proses sistem dapat dilihat oleh seluruh user di host yang sama (tergantung permission)
- Tidak ada enkripsi traffic antar host dalam jaringan lab internal

## Rekomendasi

### Jangka Pendek
- Pertahankan binding localhost untuk semua web target yang hanya digunakan untuk latihan lokal
- Pertahankan pemisahan peran Ubuntu dan Kali — jangan menjalankan tool validasi dari Ubuntu
- Dokumentasikan setiap pengecualian terhadap aturan ini

### Jangka Menengah (untuk environment produksi)
- Implementasikan segmentasi VLAN antara host analisis dan host target
- Tambahkan logging terpusat agar aktivitas dapat diaudit secara independen dari host
- Enkripsi traffic management menggunakan SSH atau VPN

### Referensi Arsitektur
- Model yang digunakan mencerminkan prinsip Zero Trust: tidak ada kepercayaan implisit berdasarkan lokasi jaringan
- Setiap akses harus diverifikasi, setiap komunikasi harus divalidasi
EOF
```

## Hubungan dengan Konsep Enterprise dan Cloud

Prinsip yang dipraktikkan dalam proyek ini secara langsung relevan dengan konsep keamanan enterprise dan cloud:

| Konsep Lab | Padanan Enterprise/Cloud |
|------------|--------------------------|
| Pemisahan Ubuntu dan Kali | Network segmentation antara management network dan production network |
| Web targets terikat ke localhost | Internal-only services dalam private subnet |
| Akun non-sudo | Principle of Least Privilege (PoLP) |
| Shared host dengan user terisolasi | Multi-tenant cloud environment |
| Pemisahan role antar host | Defense in Depth — tidak semua tool berjalan di satu host |

## Temuan Utama

Analisis arsitektur mengungkap bahwa lingkungan lab sudah menerapkan beberapa prinsip keamanan dengan benar: web targets terikat ke localhost (bukan `0.0.0.0`), akun student tidak memiliki hak sudo, dan ada pemisahan peran yang jelas antara host analisis dan host validasi.

Namun, keterbatasan yang ada — shared host, tidak ada firewall antar-user, tidak ada log terpusat — adalah batasan yang umum ditemukan dalam lingkungan pendidikan dan perlu dipahami sebagai area yang akan diperbaiki dalam arsitektur produksi nyata.

## Apa yang Dipelajari

Architecture thinking bukan keterampilan yang terpisah dari teknis — ini adalah kemampuan untuk melihat *mengapa* sistem dikonfigurasi seperti itu dan *apa* implikasi keamanannya. Analis yang dapat mengkomunikasikan temuan teknis dalam bahasa arsitektur akan jauh lebih bernilai dalam tim keamanan dibandingkan analis yang hanya dapat menjalankan tool dan melaporkan output mentahnya.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `cloud_mapping.md` | `~/mod2/evidence/architecture/` | Inventaris aset, jalur akses, dan catatan exposure |
| `network_segmentation_notes.md` | `~/mod2/evidence/architecture/` | Kondisi segmentasi saat ini dan rekomendasi |

---
---

# Proyek 6 — Network Case Assessment dan Jembatan ke Analisis Host yang Lebih Dalam

## Latar Belakang

Nilai seorang *security analyst* tidak ditentukan oleh banyaknya command yang dapat dijalankan, tetapi oleh kualitas interpretasi yang dihasilkan dari output tersebut. Assessment yang baik bukan ringkasan dari tool yang digunakan — ini adalah narasi teknis yang menjelaskan *apa yang ditemukan*, *apa artinya*, *apa risikonya*, dan *apa yang harus dilakukan*.

Proyek ini menutup seluruh rangkaian analisis jaringan dengan dua deliverable: sebuah *network case assessment* yang merangkum temuan dari semua proyek sebelumnya dalam format yang siap dibaca oleh tim teknis atau manajemen, dan sebuah *final summary* yang secara eksplisit menghubungkan hasil modul ini ke analisis yang lebih dalam di modul berikutnya.

## Tujuan

- Merangkum seluruh temuan dari proyek 1–5 dalam format assessment yang kohesif
- Mengidentifikasi risiko yang ditemukan dan memberikan rekomendasi yang dapat ditindaklanjuti
- Menuliskan jembatan eksplisit antara networking foundation dan analisis OS yang lebih dalam
- Membiasakan pola kerja di mana setiap sesi praktikum ditutup dengan deliverable yang bernilai

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Seluruh dokumentasi assessment disusun di sini |

## Langkah-Langkah yang Dilakukan

### 1. Network Case Assessment

```bash
cat > ~/mod2/reports/network_case_assessment.md <<'EOF'
# Network Case Assessment — Modul 02

## Ringkasan Eksekutif
Analisis jaringan pada lingkungan lab menunjukkan konfigurasi yang sesuai dengan prinsip keamanan dasar:
pemisahan peran host yang jelas, web target yang terisolasi di localhost, dan akun student tanpa
hak akses elevated. Tidak ditemukan anomali yang memerlukan tindakan segera dalam kondisi saat ini.

## Temuan Teknis

### 1. Baseline Host
- Interface dan routing berfungsi normal
- DNS resolver dikonfigurasi melalui gateway lab dengan fallback ke DNS publik
- Konfigurasi jaringan konsisten dengan desain lingkungan lab yang terisolasi

### 2. Service Exposure
- Port 22 (SSH): terbuka untuk akses remote yang sah
- Port 3000 (Juice Shop): terikat ke localhost — tidak terekspos ke jaringan
- Port 8081 (DVWA): terikat ke localhost — tidak terekspos ke jaringan
- Header HTTP mengekspos informasi teknologi (X-Powered-By, Server version)

### 3. Konektivitas
- Seluruh lapisan konektivitas berfungsi: local stack, IP routing, DNS resolution
- Tidak ada packet loss yang terdeteksi pada jalur ke internet
- Port target lokal (3000, 8081) dikonfirmasi menerima koneksi TCP

### 4. Monitoring
- Auth log sample menunjukkan pola brute force SSH dari IP eksternal (10.10.10.99)
- Login berhasil dari IP yang berbeda (10.10.10.5) menggunakan public key — bukan terkait brute force
- Traffic PCAP menunjukkan pola HTTP normal ke target lokal

### 5. Arsitektur
- Lingkungan menerapkan prinsip least privilege dan role separation
- Keterbatasan: tidak ada firewall antar-user, tidak ada log terpusat

## Risiko yang Teridentifikasi

| Risiko | Tingkat | Rekomendasi |
|--------|---------|-------------|
| Header server mengekspos versi teknologi | Rendah | Nonaktifkan X-Powered-By dan Server header di produksi |
| Brute force SSH dari IP eksternal (dalam sample) | Sedang | Implementasikan fail2ban atau rate limiting SSH |
| Shared host tanpa isolasi antar-user | Sedang | Gunakan container atau VM terpisah untuk lingkungan yang lebih terisolasi |
| Tidak ada log terpusat | Sedang | Implementasikan syslog terpusat untuk audit trail yang independen |

## Kesimpulan
Lingkungan lab berfungsi sesuai desain. Temuan utama yang perlu diingat untuk praktik nyata:
(1) selalu validasi service dari dua perspektif, (2) monitor auth log secara berkala, dan
(3) dokumentasikan arsitektur dalam bahasa yang dapat dikomunikasikan ke tim.
EOF
```

### 2. Final Summary — Jembatan ke Analisis Host yang Lebih Dalam

```bash
cat > ~/mod2/reports/final_summary.md <<'EOF'
# Final Summary — Jembatan ke Modul 03

## Apa yang Telah Dikerjakan di Modul 02

Modul ini membangun kemampuan untuk membaca dan menganalisis jaringan dari perspektif yang profesional:

1. **Baseline host** — memahami interface, route, dan resolver sebelum melihat ke luar
2. **Service exposure** — membedakan perspektif internal (ss, ps) dan eksternal (nmap, curl)
3. **Troubleshooting terstruktur** — mendiagnosis konektivitas berlapis tanpa tebakan
4. **Monitoring operasional** — membaca journal, auth log, dan PCAP sebagai sumber yang saling melengkapi
5. **Architecture thinking** — menerjemahkan observasi teknis ke dalam dokumentasi arsitektur

## Koneksi ke Modul 03 — Operating Systems & Scripting

Setiap area yang dikerjakan di Modul 02 memiliki kelanjutan langsung di Modul 03:

| Area Modul 02 | Kelanjutan di Modul 03 |
|---------------|------------------------|
| Service exposure (`ss`, `nmap`) | Relasi port–proses yang lebih dalam, analisis file descriptor |
| Process context (`ps`) | Process tree, parent-child relationship, zombie process |
| Auth log analysis | Log rotation, syslog, journald — cara log dikelola oleh OS |
| Network baseline | Hubungan antara network stack dan konfigurasi OS (firewall, kernel parameter) |
| Architecture notes | Automation — scripting untuk monitoring, alerting, dan reporting |

## Fondasi yang Siap Dibawa ke Modul 03

- Mampu membaca baseline host dari sisi jaringan
- Mampu memvalidasi service dari dua perspektif
- Mampu men-troubleshoot secara terstruktur dan evidence-based
- Mampu membaca log dan packet summary untuk konteks operasional
- Mampu mendokumentasikan temuan dalam format assessment yang profesional
EOF
```

## Struktur Assessment yang Digunakan

Assessment yang baik mengikuti struktur yang dapat diprediksi sehingga pembaca dapat menemukan informasi yang mereka butuhkan dengan cepat:

| Bagian | Fungsi | Pembaca Utama |
|--------|--------|---------------|
| Ringkasan Eksekutif | Gambaran singkat untuk pembaca yang tidak punya waktu membaca detail | Manajemen, pemimpin tim |
| Temuan Teknis | Detail observasi per area | Analis, engineer |
| Risiko yang Teridentifikasi | Tabel risiko dengan tingkat dan rekomendasi | Security manager, CISO |
| Kesimpulan | Poin utama yang perlu diingat | Semua pembaca |

## Temuan Utama

Proyek ini mengkonfirmasi bahwa kemampuan untuk *menulis* tentang temuan keamanan sama pentingnya dengan kemampuan untuk *menemukan* temuan tersebut. Assessment yang tidak dapat dikomunikasikan dengan baik kehilangan sebagian besar nilainya — tidak ada tindakan yang dapat diambil berdasarkan laporan yang tidak dapat dipahami oleh pembacanya.

## Apa yang Dipelajari

Menutup setiap sesi analisis dengan assessment tertulis adalah kebiasaan profesional yang memiliki dampak besar: (1) memaksa analis untuk benar-benar memahami apa yang ditemukan, bukan hanya mengumpulkan output, (2) menciptakan dokumentasi yang dapat digunakan kembali, dan (3) mempersiapkan analis untuk peran yang lebih senior di mana komunikasi adalah kompetensi inti.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `network_case_assessment.md` | `~/mod2/reports/` | Assessment lengkap dengan temuan, risiko, dan rekomendasi |
| `final_summary.md` | `~/mod2/reports/` | Ringkasan modul dan jembatan eksplisit ke Modul 03 |

---

