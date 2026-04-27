# Identifikasi Indikator Kompromi pada Sampel Artefak dan Data Lab

## Latar Belakang

*Security awareness* yang sejati bukan tentang menghafal daftar ancaman atau mengikuti pelatihan tahunan. Ini tentang kemampuan untuk melihat artefak nyata — sebuah email, sebuah file konfigurasi, sebuah kumpulan data — dan secara kritis mengidentifikasi elemen-elemen yang menunjukkan adanya risiko, manipulasi, atau eksposur yang tidak semestinya.

Proyek ini mendekati keamanan dari sudut pandang praktis: menganalisis sampel email phishing menggunakan teknik pembacaan kritis yang sama yang digunakan oleh analis *threat intelligence*, melakukan pemindaian string berisiko pada dataset yang tersedia untuk mendeteksi potensi eksposur kredensial, dan mendokumentasikan prinsip-prinsip pengelolaan kata sandi yang operasional dan dapat diterapkan. Seluruh aktivitas berbasis pada artefak nyata di lingkungan lab — bukan skenario hipotesis.

---

## Tujuan

- Menganalisis sampel email phishing dan mengidentifikasi teknik rekayasa sosial (*social engineering*) yang digunakan
- Melakukan pemindaian berbasis regex pada dataset untuk menemukan string yang berpotensi mengekspos kredensial atau informasi sensitif
- Mendokumentasikan prinsip pengelolaan kata sandi yang profesional dan operasional
- Membangun kebiasaan berpikir kritis saat berhadapan dengan komunikasi dan artefak digital

---

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Seluruh analisis artefak dan pemindaian dilakukan di sini |

---

## Langkah-Langkah yang Dilakukan

### 1. Penemuan dan Tinjauan Sampel Email Phishing

Langkah pertama adalah menemukan artefak phishing yang tersedia di dataset lab, kemudian membacanya secara menyeluruh dengan pendekatan analitis — bukan sekadar membacanya sebagai email biasa.

```bash
# Temukan file phishing yang tersedia
find ~/datasets -type f | grep -i 'phish\|email'

# Baca dan simpan isi sampel sebagai evidence
cat ~/datasets/phishing/sample_email_1.txt | tee ~/module01/evidence/phishing_review.txt
```

**Framework Analisis Email Phishing:**

Saat menganalisis email yang mencurigakan, terdapat beberapa dimensi yang harus diperiksa secara sistematis:

#### A. Analisis Pengirim (*Sender Analysis*)

| Elemen yang Diperiksa | Tanda Bahaya (*Red Flag*) |
|-----------------------|--------------------------|
| Alamat email pengirim | Domain yang menyerupai domain legitimate (contoh: `paypa1.com` vs `paypal.com`) |
| Nama tampilan | Nama familiar yang tidak cocok dengan alamat email sebenarnya |
| Header email (via sumber) | Ketidakcocokan antara `From:`, `Reply-To:`, dan `Return-Path:` |

#### B. Analisis Konten (*Content Analysis*)

| Teknik Manipulasi | Contoh dalam Email Phishing | Mekanisme Psikologis |
|-------------------|-----------------------------|----------------------|
| **Urgensi palsu** | "Akun Anda akan ditutup dalam 24 jam" | Kepanikan mendorong tindakan impulsif |
| **Otoritas yang dipalsukan** | "Tim Keamanan Bank XYZ" | Kecenderungan patuh pada figur otoritas |
| **Permintaan informasi sensitif** | "Konfirmasi password Anda untuk verifikasi" | Organisasi legitimate tidak pernah meminta ini |
| **Ancaman konsekuensi** | "Jika tidak dikonfirmasi, akun akan diblokir" | Rasa takut mendorong kepatuhan tanpa berpikir |
| **Penawaran menggiurkan** | "Anda memenangkan hadiah senilai Rp 10 juta" | Keserakahan dan harapan menurunkan kewaspadaan |

#### C. Analisis Tautan (*Link Analysis*)

| Elemen | Yang Harus Diperiksa |
|--------|---------------------|
| URL yang ditampilkan | Apakah cocok dengan domain yang diklaim? |
| URL sebenarnya (hover) | Apakah berbeda dari yang ditampilkan? |
| Domain URL | Apakah menggunakan subdomain yang menyesatkan? (contoh: `paypal.malicious.com`) |
| Protokol | Apakah menggunakan HTTPS? (Meski HTTPS tidak menjamin keamanan konten) |

#### D. Indikator Teknis

```
Tanda-tanda phishing yang umum ditemukan dalam sampel:
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
   Kompromi satu akun tidak boleh berimbas pada akun lainnya.

2. PANJANG DAN KOMPLEKSITAS
   Panjang kata sandi (>16 karakter) lebih efektif daripada variasi simbol yang dangkal.
   Frasa sandi (passphrase) yang panjang lebih kuat dan lebih mudah diingat.

3. TIDAK ADA POLA YANG DAPAT DITEBAK
   Hindari informasi pribadi: tanggal lahir, nama, kota asal.
   Hindari pola keyboard: qwerty, 12345678, asdfgh.

4. PENGELOLAAN KREDENSIAL
   Gunakan password manager untuk menyimpan dan menghasilkan kata sandi.
   Jangan tulis kata sandi di sticky note, dokumen tidak terenkripsi, atau pesan teks.

5. ROTASI KREDENSIAL
   Ganti kata sandi segera jika ada indikasi kompromi.
   Aktifkan notifikasi untuk login dari lokasi atau perangkat baru.

6. AUTENTIKASI MULTI-FAKTOR (MFA)
   Aktifkan MFA pada seluruh akun yang mendukungnya.
   MFA secara signifikan mengurangi dampak kebocoran kata sandi.

7. KREDENSIAL LAB
   Kata sandi yang digunakan di lingkungan lab tidak boleh digunakan di sistem lain.
   Asumsikan seluruh aktivitas di lab dapat dimonitor dan diaudit.
EOF

cat ~/module01/evidence/password_strength_notes.txt
```

### 3. Pemindaian String Berisiko pada Dataset

Pemindaian berbasis regex digunakan untuk secara sistematis mencari string yang sering ditemukan dalam kasus eksposur kredensial, dokumen phishing, atau file konfigurasi yang tidak aman.

```bash
grep -RinE 'password|secret|token|urgent|verify|login|credential|api_key|passwd' \
    ~/datasets ~/samples 2>/dev/null | \
    tee -a ~/module01/evidence/risky_strings_findings.txt | \
    head -n 50
```

**Penjelasan flag `grep`:**

| Flag | Fungsi |
|------|--------|
| `-R` | Rekursif — telusuri seluruh subdirektori |
| `-i` | *Case-insensitive* — cocokkan `PASSWORD`, `password`, `Password` sekaligus |
| `-n` | Tampilkan nomor baris tempat kecocokan ditemukan |
| `-E` | Gunakan *Extended Regular Expression* untuk pola `pattern1\|pattern2` |

**Kata kunci yang dicari dan relevansinya:**

| Kata Kunci | Relevansi Keamanan |
|------------|-------------------|
| `password`, `passwd` | Potensi kredensial tersimpan dalam plaintext |
| `secret` | Secret key, API secret, atau token rahasia |
| `token` | API token, JWT, atau session token yang mungkin aktif |
| `api_key` | Kunci API yang dapat digunakan untuk mengakses layanan eksternal |
| `urgent`, `verify` | Bahasa phishing yang menciptakan urgensi |
| `login` | Halaman login, event login, atau proses autentikasi |
| `credential` | Referensi eksplisit ke informasi autentikasi |

---

## Temuan Utama

### Dari Analisis Phishing

Email phishing yang dianalisis menggunakan **tiga teknik manipulasi sekaligus** dalam satu pesan:
1. **Urgensi waktu** — batas waktu 24 jam yang menciptakan tekanan untuk bertindak cepat
2. **Otoritas yang dipalsukan** — mengklaim berasal dari institusi yang dikenal
3. **Permintaan kredensial eksplisit** — meminta pengguna untuk "memverifikasi" data login mereka

Kombinasi ketiga teknik ini adalah pola *triple-threat* yang paling umum ditemukan dalam kampanye phishing skala besar dan yang paling efektif menipu pengguna yang tidak waspada.

### Dari Pemindaian String Berisiko

Pemindaian menemukan beberapa file di dalam dataset yang mengandung string `password=` dan `token=` dalam format plaintext. Dalam skenario nyata, temuan seperti ini akan langsung diklasifikasikan sebagai:

- **Severity: High** jika file tersebut dapat diakses oleh pengguna yang tidak berwenang
- **Tindakan segera:** Enkripsi file, rotasi kredensial yang terekspos, audit akses log untuk mengetahui apakah file sudah diakses pihak yang tidak berwenang

---

## Apa yang Dipelajari

*Security awareness* yang efektif adalah kemampuan untuk menerjemahkan pengetahuan tentang ancaman menjadi tindakan analitis yang konkret. Seorang analis yang baik tidak hanya tahu bahwa "phishing itu berbahaya" — ia tahu **bagaimana mengidentifikasi teknik manipulasi spesifik** yang digunakan dalam sebuah pesan dan dapat **mendokumentasikannya secara sistematis**.

Pemindaian string berbasis regex yang dipraktikkan di sini adalah teknik yang sama yang digunakan dalam:
- *Incident response* untuk mencari kredensial yang mungkin terekspos setelah breach
- *Data Loss Prevention (DLP)* untuk mendeteksi informasi sensitif yang keluar melalui saluran yang tidak aman
- *Code review* untuk menemukan hardcoded credentials dalam repositori kode sumber

---

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `phishing_review.txt` | `~/module01/evidence/` | Salinan dan analisis sampel email phishing |
| `password_strength_notes.txt` | `~/module01/evidence/` | Dokumentasi prinsip pengelolaan kata sandi |
| `risky_strings_findings.txt` | `~/module01/evidence/` | Hasil pemindaian string berisiko pada dataset |
