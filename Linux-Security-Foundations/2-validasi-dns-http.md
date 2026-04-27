# Validasi Resolusi DNS dan Inspeksi HTTP Header pada Target Eksternal dan Lokal

## Latar Belakang

Pemahaman tentang bagaimana nama domain diterjemahkan menjadi alamat IP, bagaimana server merespons permintaan HTTP, dan bagaimana service berjalan di atas port tertentu merupakan kompetensi dasar yang mutlak dibutuhkan dalam keamanan siber. Tanpa kemampuan ini, seorang analis tidak dapat membedakan antara service yang berjalan normal dan service yang berpotensi terekspos secara tidak semestinya.

Proyek ini memvalidasi seluruh rantai konektivitas dasar — dari resolusi nama domain hingga pembacaan HTTP response header dan enumerasi service localhost — menggunakan Kali Linux sebagai *validation workstation*. Pendekatan yang digunakan sengaja mengkombinasikan dua atau lebih tool untuk setiap tahap validasi, karena dalam praktik nyata, satu tool saja tidak cukup untuk memberikan gambaran yang lengkap dan akurat.

---

## Tujuan

- Memvalidasi resolusi nama domain menggunakan dua pendekatan yang berbeda (`dig` dan `getent`) dan menganalisis perbedaan hasilnya
- Membaca dan menginterpretasikan HTTP response header dari target eksternal maupun target lokal di dalam lab
- Mengenumerasi service yang aktif pada localhost dari perspektif internal sistem (`ss`) dan perspektif eksternal assessment (`nmap`)
- Membangun kebiasaan membaca output jaringan secara kritis, bukan sekadar menjalankan perintah

---

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Kali Linux** | Host validasi — seluruh aktivitas jaringan dijalankan dari sini |
| **Ubuntu Lab** | Tidak digunakan secara aktif — hasil dari Kali akan disinkronkan ke Ubuntu untuk penyimpanan evidence terpusat |

---

## Langkah-Langkah yang Dilakukan

### 1. Validasi Resolusi DNS

DNS (*Domain Name System*) adalah fondasi dari seluruh komunikasi internet. Sebelum koneksi ke sebuah server dapat terjadi, nama domain harus terlebih dahulu diterjemahkan menjadi alamat IP. Proses ini dilakukan oleh resolver DNS, dan hasilnya dapat bervariasi tergantung pada tool dan konfigurasi sistem yang digunakan.

```bash
printf "=== DNS LOOKUP ===\n" | tee ~/module01/evidence/dns_check.txt
dig openai.com +short   | tee -a ~/module01/evidence/dns_check.txt
dig example.com +short  | tee -a ~/module01/evidence/dns_check.txt
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

Ini adalah bagian yang paling informatif dari proyek ini. Service yang sama dilihat dari dua sudut pandang yang berbeda: dari dalam sistem (*internal view*) dan dari luar (*external/assessment view*).

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

---

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
age: 526065
cache-control: max-age=604800
content-type: text/html; charset=UTF-8
server: ECAcc (nyd/D10E)

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

---

## Temuan Utama

**Perbedaan perspektif `ss` vs `nmap`** adalah temuan teknis yang paling signifikan dari proyek ini. Dalam pengujian nyata, ada kasus di mana sebuah port tampak tertutup dari pemindaian eksternal namun sebenarnya terbuka dan dapat diakses — hal ini terjadi karena aturan firewall yang hanya memblokir traffic dari interface tertentu namun tidak dari localhost. Memahami perbedaan ini menghindarkan analis dari kesimpulan yang keliru bahwa sebuah service "aman" hanya karena tidak terlihat dari satu sudut pandang.

Header `X-Powered-By: Express` pada target lokal juga merupakan temuan yang perlu dicatat: header ini mengekspos teknologi backend yang digunakan. Dalam lingkungan produksi, header ini sebaiknya dinonaktifkan untuk mengurangi informasi yang tersedia bagi penyerang.

---

## Apa yang Dipelajari

Validasi jaringan yang benar tidak pernah bergantung pada satu tool. Menggunakan `dig` saja tanpa `getent` bisa menyebabkan analis melewatkan override DNS lokal. Menggunakan `ss` saja tanpa `nmap` bisa menyebabkan analis berpikir bahwa semua service terlihat dari luar — padahal tidak selalu demikian.

Kebiasaan menggunakan minimal dua tool untuk setiap validasi jaringan adalah standar kerja yang perlu dibangun sejak awal dan akan terus relevan pada proyek-proyek yang lebih kompleks.

---

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `dns_check.txt` | `~/module01/evidence/` | Hasil resolusi DNS dengan `dig` dan `getent` |
| `http_headers.txt` | `~/module01/evidence/` | HTTP response header dari target eksternal dan lokal |
| `localhost_service_validation.txt` | `~/module01/evidence/` | Output `ss` dan `nmap` untuk enumerasi service |
