# Deteksi Pola Mencurigakan pada Log Aktivitas Sistem dengan Shell dan Python

## Latar Belakang

Log sistem adalah sumber informasi paling kaya dalam keamanan siber. Hampir setiap serangan siber — mulai dari *brute force login*, *lateral movement*, hingga *data exfiltration* — meninggalkan jejak di log. Kemampuan untuk membaca log secara efisien, menyaring sinyal yang relevan dari kebisingan (*noise*), dan mengidentifikasi pola yang mencurigakan adalah kompetensi inti seorang *security analyst*.

Proyek ini membangun fondasi analisis log menggunakan dua pendekatan yang saling melengkapi: shell scripting dengan `grep` dan `awk` untuk filtering cepat dan otomasi keputusan berbasis kondisi, serta Python untuk analisis yang lebih terstruktur dan dapat dikembangkan. Keduanya dipraktikkan pada dataset log yang dirancang untuk mencerminkan pola nyata yang ditemukan dalam *security monitoring*.

---

## Tujuan

- Membuat dataset log simulasi yang mencerminkan aktivitas sistem nyata
- Memfilter event berdasarkan tingkat keparahan dan jenis aktivitas menggunakan shell pipeline
- Membangun logika keputusan otomatis berbasis ambang batas dengan Bash scripting
- Mengembangkan parser log terstruktur dengan Python yang menghasilkan ringkasan analitis
- Memahami kapan menggunakan shell dan kapan menggunakan Python dalam konteks analisis log

---

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Seluruh aktivitas parsing dan scripting dilakukan di sini |

---

## Dataset Log

Log simulasi dibuat dengan struktur yang mencerminkan format log sistem nyata:

```bash
cat > ~/module01/tmp/activity.log <<'EOF'
INFO login user=alice src=10.10.10.5
WARN failed_login user=bob src=10.10.10.7
WARN failed_login user=bob src=10.10.10.7
INFO file_upload user=alice path=/uploads/q1.xlsx
WARN failed_login user=charlie src=10.10.10.9
EOF
```

**Struktur setiap baris log:**

| Kolom | Contoh | Keterangan |
|-------|--------|------------|
| Level | `INFO`, `WARN` | Tingkat keparahan event |
| Event type | `login`, `failed_login`, `file_upload` | Jenis aktivitas yang terjadi |
| User | `user=alice` | Identitas pengguna yang terlibat |
| Source/Path | `src=10.10.10.5` | Alamat IP sumber atau path file |

---

## Langkah-Langkah yang Dilakukan

### 1. Filtering Event dengan `grep`

Langkah pertama dalam analisis log adalah menyaring hanya baris yang relevan untuk ditinjau lebih lanjut. `grep` memungkinkan filtering berbasis pola teks secara instan, tanpa perlu membaca seluruh file secara manual.

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

Setelah mengetahui *event apa* yang terjadi, langkah selanjutnya adalah mengetahui *seberapa sering* event tersebut terjadi per entitas (user dan IP). `awk` digunakan untuk mengekstrak kolom tertentu, yang kemudian diagregasi dengan `sort` dan `uniq -c`.

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

Filtering manual berguna untuk analisis satu waktu, namun dalam operasi keamanan yang nyata, deteksi harus dapat berjalan secara otomatis dan memberikan output keputusan yang jelas. Script Bash berikut mengimplementasikan logika sederhana: jika jumlah event `failed_login` mencapai atau melebihi ambang batas tertentu, keluarkan peringatan.

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
    echo ""
    echo "--- Detail Event ---"
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

--- Detail Event ---
WARN failed_login user=bob src=10.10.10.7
WARN failed_login user=bob src=10.10.10.7
WARN failed_login user=charlie src=10.10.10.9
```

### 4. Parser Log Terstruktur dengan Python

Python digunakan untuk menghasilkan ringkasan yang lebih terstruktur, dapat direproduksi, dan mudah dikembangkan menjadi komponen yang lebih besar. Parser ini mengekstrak jumlah event, ringkasan per user, dan distribusi level event.

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
        events[parts[0]] += 1          # Hitung level (INFO/WARN)
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

---

## Perbandingan Shell vs Python

| Aspek | Shell (`grep`, `awk`, `bash`) | Python |
|-------|-------------------------------|--------|
| Kecepatan setup | Sangat cepat, langsung dijalankan | Perlu menulis script terlebih dahulu |
| Keterbacaan output | Terbatas, tergantung formatting manual | Lebih terstruktur dan mudah dikustomisasi |
| Kemampuan pengembangan | Terbatas untuk logika kompleks | Sangat fleksibel, dapat dikembangkan menjadi modul |
| Integrasi dengan tools lain | Sangat baik via pipe (`\|`) | Baik via library dan API |
| Penggunaan ideal | Filtering cepat, one-liner, otomasi sederhana | Analisis terstruktur, laporan, integrasi data |

---

## Temuan Utama

Dari analisis log, ditemukan bahwa **user `bob` melakukan dua percobaan login yang gagal dari IP yang sama (`10.10.10.7`) dalam satu sesi**. Ini adalah pola klasik *brute-force* atau *credential stuffing* — di mana penyerang mencoba berbagai kombinasi kredensial secara berulang. Dalam lingkungan produksi, pola ini seharusnya memicu alert otomatis dan potensi blokir sementara pada IP sumber.

User `charlie` juga mencatat satu event `failed_login` — lebih rendah dari ambang batas untuk peringatan berulang, namun tetap perlu dicatat sebagai anomali yang perlu dipantau.

---

## Apa yang Dipelajari

Shell dan Python bukan alat yang bersaing — keduanya memiliki tempat yang berbeda dalam alur kerja analis keamanan. Shell unggul untuk inspeksi cepat dan otomasi pipeline yang ringan. Python unggul untuk analisis yang dapat direproduksi, dapat dikembangkan, dan menghasilkan output yang lebih terstruktur untuk pelaporan.

Pola berpikir yang dibangun di sini — ekstrak, hitung, bandingkan dengan ambang batas, keluarkan keputusan — adalah fondasi dari hampir semua sistem deteksi ancaman, mulai dari skrip sederhana hingga platform SIEM (*Security Information and Event Management*) yang kompleks.

---

## Referensi File Bukti dan Script

| File | Lokasi | Isi |
|------|--------|-----|
| `activity.log` | `~/module01/tmp/` | Dataset log simulasi |
| `activity_filtering.txt` | `~/module01/evidence/` | Output grep dan awk filtering |
| `shell_logic_output.txt` | `~/module01/evidence/` | Output script deteksi Bash |
| `python_logic_output.txt` | `~/module01/evidence/` | Output parser Python |
| `check_failed_logins.sh` | `~/module01/tmp/` | Script deteksi otomatis Bash |
| `activity_parser.py` | `~/module01/tmp/` | Parser log Python |
