## Tentang Portofolio Ini

Portofolio ini mendokumentasikan serangkaian proyek *programming*, *automation*, dan *security tooling* yang dikerjakan secara langsung (*hands-on*) di lingkungan lab terisolasi. Setiap proyek dirancang untuk mencerminkan cara kerja seorang *security engineer* atau *SOC automation developer* profesional — mulai dari pengelolaan Python environment, pembangunan workflow automasi endpoint, penerapan prinsip *secure coding*, parsing IOC dari artefak nyata, hingga pengembangan mini tool keamanan yang menghasilkan output terstruktur dan dapat dipakai ulang.

Seluruh aktivitas dilakukan pada dua host lab dengan peran yang berbeda:

| Host | Peran |
|------|-------|
| **Ubuntu Lab** | *Primary development and analysis host* — Python scripting, file parsing, log processing, secure coding, dan report generation |
| **Kali Linux** | *Validation workstation* — endpoint testing, header validation, service verification, dan assessment output checking |
| **Local Targets** | *Safe practice target* — Juice Shop (`127.0.0.1:3000`) dan DVWA (`127.0.0.1:8081`) untuk pengujian automation yang aman |
| **Datasets `/opt/jccsah`** | Log sample, phishing sample, PCAP sample, starter code, dan report template |

> ⚠️ **Catatan:** Seluruh aktivitas dilakukan **hanya** pada lingkungan lab yang telah disediakan. Tidak ada brute force, exploit workflow, mass scanning, atau automation ke target di luar scope yang disediakan.

---

## Daftar Proyek

| No. | Judul Proyek | Fokus Area | Tools Utama |
|-----|-------------|------------|-------------|
| 1 | [Menyiapkan Python Environment dan Workspace Automation](#proyek-1--menyiapkan-python-environment-dan-workspace-automation) | Environment setup, workspace management | `python`, `pip`, `venv`, `pathlib` |
| 2 | [Menerapkan Python Foundations untuk Operasi Keamanan](#proyek-2--menerapkan-python-foundations-untuk-operasi-keamanan) | File I/O, structured data, argument handling | `json`, `argparse`, `pathlib` |
| 3 | [Membangun Automation Workflow untuk Endpoint Validation](#proyek-3--membangun-automation-workflow-untuk-endpoint-validation) | Endpoint automation, HTTP validation | `requests`, `curl`, `nmap` |
| 4 | [Menerapkan Secure Coding pada Script Keamanan](#proyek-4--menerapkan-secure-coding-pada-script-keamanan) | Input validation, parameterized query, error handling | `urllib`, `sqlite3`, `try/except` |
| 5 | [Membangun Threat-Aware Script untuk Analisis IOC dan Artefak](#proyek-5--membangun-threat-aware-script-untuk-analisis-ioc-dan-artefak) | IOC parsing, hash review, YARA detection | `re`, `sha256sum`, `yara` |
| 6 | [Mengembangkan Mini Tool Keamanan dengan Output Terstruktur](#proyek-6--mengembangkan-mini-tool-keamanan-dengan-output-terstruktur) | Tool building, structured reporting | `requests`, `pathlib`, `curl` |
| 7 | [Menyusun Automation Assessment dan Jembatan ke Modul Berikutnya](#proyek-7--menyusun-automation-assessment-dan-jembatan-ke-modul-berikutnya) | Assessment writing, operational documentation | Dokumentasi profesional |

---

## Teknologi dan Tools

<p>
  <img src="https://img.shields.io/badge/OS-Ubuntu%20Linux-E95420?style=flat-square&logo=ubuntu&logoColor=white"/>
  <img src="https://img.shields.io/badge/OS-Kali%20Linux-557C94?style=flat-square&logo=kalilinux&logoColor=white"/>
  <img src="https://img.shields.io/badge/Python-3.x-3776AB?style=flat-square&logo=python&logoColor=white"/>
  <img src="https://img.shields.io/badge/Library-requests-informational?style=flat-square"/>
  <img src="https://img.shields.io/badge/Library-pathlib%20%2F%20json-informational?style=flat-square"/>
  <img src="https://img.shields.io/badge/Library-argparse%20%2F%20sqlite3-informational?style=flat-square"/>
  <img src="https://img.shields.io/badge/Tool-YARA-red?style=flat-square"/>
  <img src="https://img.shields.io/badge/Tool-curl%20%2F%20nmap-073551?style=flat-square"/>
</p>

---

## Kompetensi yang Dibangun

- **Python Environment Management** — Mengaktifkan dan memverifikasi virtual environment, mencatat path interpreter, dan memastikan konsistensi lintas sesi kerja
- **Security-Oriented Python** — Menggunakan `pathlib`, `json`, `argparse`, dan `requests` untuk membangun script yang jelas input-output-nya dan mudah dijalankan ulang
- **Endpoint Automation** — Membangun workflow validasi endpoint yang menyimpan output terstruktur dan dapat dibandingkan dengan observasi manual dari workstation lain
- **Secure Coding Practices** — Menerapkan input validation, parameterized query, dan error handling yang informatif namun tidak membocorkan detail berlebihan
- **Threat-Aware Scripting** — Mengekstrak IOC dari artefak teks, melakukan hash review untuk verifikasi integritas, dan menerapkan YARA rule untuk deteksi berbasis pola
- **Security Tool Building** — Mengembangkan mini tool yang memiliki input yang jelas, error handling yang tepat, dan output yang tersimpan serta dapat dipakai ulang
- **Automation Assessment Writing** — Menyusun penilaian automation readiness yang menghubungkan seluruh output teknis menjadi narasi yang siap dibaca tim

---
---

# Proyek 1 — Menyiapkan Python Environment dan Workspace Automation

## Latar Belakang

Dalam pengembangan automation untuk keamanan siber, kegagalan lingkungan kerja (*environment failure*) adalah salah satu penyebab paling umum dari script yang berjalan dengan benar di satu mesin namun gagal di mesin lain. Masalah ini — yang dikenal sebagai *"works on my machine"* — sering disebabkan oleh perbedaan versi Python, library yang tidak terinstal, atau path interpreter yang berbeda antara sesi kerja.

Virtual environment adalah solusi standar untuk masalah ini: sebuah isolasi Python yang memastikan bahwa seluruh dependency yang dibutuhkan tersedia dengan versi yang tepat, terlepas dari konfigurasi Python global di sistem. Proyek ini membangun fondasi environment yang akan digunakan sepanjang seluruh rangkaian proyek automation di modul ini.

## Tujuan

- Membangun struktur workspace yang memisahkan source code, evidence, dan report secara jelas
- Mengaktifkan Python virtual environment yang disiapkan administrator lab dan memverifikasi konfigurasinya
- Mencatat path interpreter dan pip sebagai dokumentasi environment baseline
- Memverifikasi ketersediaan seluruh resource lab yang akan digunakan sepanjang modul

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Seluruh setup environment dan workspace dilakukan di sini |

## Langkah-Langkah yang Dilakukan

### 1. Pembangunan Struktur Workspace Modul 04

```bash
mkdir -p ~/mod4/{evidence,reports,notes,baseline,foundations,automation,secure_coding,analysis,tmp}
cd ~/mod4
pwd
ls -la
```

**Fungsi tiap direktori:**

| Direktori | Fungsi |
|-----------|--------|
| `evidence/baseline/` | Python version, path interpreter, dan workspace inventory |
| `evidence/foundations/` | Output demo File I/O, JSON, dan argparse |
| `evidence/automation/` | Output endpoint checker dan catatan automation workflow |
| `evidence/secure_coding/` | Output demo input validation, query aman, dan error handling |
| `evidence/analysis/` | IOC parser output, hash review, dan YARA result |
| `reports/` | Automation assessment dan bridge notes |
| `tmp/` | File script sementara selama pengembangan |

Pemisahan antara `tmp/` (untuk script yang sedang dikembangkan) dan `evidence/` (untuk output yang sudah terverifikasi) adalah praktik penting dalam automation: script yang belum divalidasi tidak boleh dianggap sebagai bukti kerja yang dapat dipertanggungjawabkan.

### 2. Aktivasi Python Environment dan Pencatatan Baseline

```bash
source ~/activate-pylab
python --version | tee ~/mod4/evidence/baseline/python_version.txt
which python     | tee ~/mod4/evidence/baseline/venv_paths.txt
which pip        | tee -a ~/mod4/evidence/baseline/venv_paths.txt
~/jccsah-env-info
```

**Mengapa mencatat path interpreter penting:**

Perbedaan antara `/usr/bin/python3` (sistem) dan `/opt/jccsah/venvs/pylab/bin/python` (virtual environment lab) bukan hanya perbedaan path — ini perbedaan dalam library yang tersedia. Script yang dijalankan dengan interpreter yang salah akan gagal dengan error `ModuleNotFoundError` yang membingungkan jika tidak dipahami konteksnya.

**Verifikasi yang dilakukan oleh `~/jccsah-env-info`:**

| Yang Diverifikasi | Relevansi |
|-------------------|-----------|
| Versi Python aktif | Memastikan kompatibilitas dengan script yang akan ditulis |
| Library yang tersedia | Memastikan `requests`, `yara`, dan dependency lain tersedia |
| Symlink datasets | Memastikan data input tersedia sebelum script dijalankan |
| Helper tools | Memverifikasi `tshark`, `nmap`, dan tool pendukung lainnya |

### 3. Inventaris Workspace dan Resource

```bash
find ~/mod4 -maxdepth 2 -type d | sort | tee ~/mod4/evidence/baseline/module04_workspace_tree.txt
ls -la ~/datasets ~/starter ~/wordlists
```

`find` dengan `-maxdepth 2` menghasilkan inventaris struktur dua level yang cukup detail untuk menunjukkan organisasi workspace tanpa menghasilkan output yang terlalu panjang. File `module04_workspace_tree.txt` ini menjadi referensi yang dapat digunakan untuk memverifikasi bahwa tidak ada direktori yang terlewat atau tidak sengaja dibuat di lokasi yang salah.

## Hasil dan Output

**Contoh isi `python_version.txt`:**
```
Python 3.11.4
```

**Contoh isi `venv_paths.txt`:**
```
/opt/jccsah/venvs/pylab/bin/python
/opt/jccsah/venvs/pylab/bin/pip
```

Kedua path mengarah ke direktori virtual environment (`/opt/jccsah/venvs/pylab/`) — bukan ke sistem Python global. Ini mengkonfirmasi bahwa script yang dijalankan akan menggunakan library yang tepat.

**Contoh isi `module04_workspace_tree.txt`:**
```
/home/student/mod4
/home/student/mod4/analysis
/home/student/mod4/automation
/home/student/mod4/baseline
/home/student/mod4/evidence
/home/student/mod4/foundations
/home/student/mod4/notes
/home/student/mod4/reports
/home/student/mod4/secure_coding
/home/student/mod4/tmp
```

## Temuan Utama

Verifikasi path interpreter mengkonfirmasi bahwa seluruh sesi Python akan menggunakan virtual environment lab, bukan Python sistem. Ini penting karena library seperti `requests` dan `yara-python` hanya tersedia di virtual environment dan tidak diinstal di Python sistem — perbedaan yang akan menyebabkan error jika interpreter yang salah digunakan.

Inventaris workspace mengkonfirmasi bahwa seluruh 9 direktori berhasil dibuat dengan benar dan siap digunakan.

## Apa yang Dipelajari

Environment management bukan langkah administratif yang dapat diabaikan — ini adalah fondasi teknis yang menentukan apakah script akan berjalan dengan benar. Mendokumentasikan environment baseline (path, versi, library) di awal sesi memastikan bahwa jika ada masalah di kemudian hari, analis memiliki referensi untuk debugging yang akurat.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `python_version.txt` | `~/mod4/evidence/baseline/` | Versi Python yang aktif |
| `venv_paths.txt` | `~/mod4/evidence/baseline/` | Path interpreter dan pip |
| `module04_workspace_tree.txt` | `~/mod4/evidence/baseline/` | Struktur direktori workspace |

---
---

# Proyek 2 — Menerapkan Python Foundations untuk Operasi Keamanan

## Latar Belakang

Python menawarkan ratusan library dan ribuan cara untuk menyelesaikan setiap masalah. Namun dalam konteks operasi keamanan siber, sebagian besar pekerjaan sehari-hari dapat diselesaikan dengan subset kecil dari kemampuan Python: membaca dan menulis file, memproses data terstruktur, dan menerima parameter input dari pengguna. Menguasai tiga komponen ini dengan benar dan efisien adalah fondasi yang jauh lebih berharga daripada mengetahui ratusan library secara dangkal.

Proyek ini memfokuskan pengembangan pada tiga fondasi Python yang paling relevan untuk operasi keamanan: penanganan file dengan `pathlib`, pemrosesan data terstruktur dengan `json`, dan penanganan argument input dengan `argparse`. Ketiganya akan digunakan berulang kali dalam seluruh proyek berikutnya.

## Tujuan

- Membiasakan penggunaan `pathlib` sebagai cara modern dan aman untuk menangani path file lintas platform
- Menggunakan `json` untuk menyimpan dan membaca output terstruktur yang dapat diproses oleh tool lain
- Mengimplementasikan `argparse` untuk membuat script yang menerima parameter input secara eksplisit
- Membangun kebiasaan menyimpan seluruh output ke file evidence, bukan hanya menampilkannya di terminal

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Seluruh pengembangan script foundations dilakukan di sini |

## Langkah-Langkah yang Dilakukan

### 1. File I/O dan Structured Data dengan JSON

```python
# json_yaml_demo.py
import json
from pathlib import Path

sample = {
    'host'   : 'ubuntu-lab',
    'service': 'local-target',
    'ports'  : [3000, 8081],
    'status' : 'ready'
}

out = Path.home() / 'mod4' / 'evidence' / 'foundations' / 'json_yaml_notes.txt'
out.write_text(json.dumps(sample, indent=2))
print(out.read_text())
```

```bash
python ~/mod4/tmp/json_yaml_demo.py
```

**Mengapa `pathlib` lebih baik dari string concatenation untuk path:**

| Pendekatan | Contoh | Masalah |
|------------|--------|---------|
| String concatenation | `'/home/' + user + '/mod4/evidence'` | Tidak portabel, rentan typo, tidak ada validasi |
| `os.path.join()` | `os.path.join('/home', user, 'mod4', 'evidence')` | Lebih baik, tapi masih verbose |
| `pathlib.Path` | `Path.home() / 'mod4' / 'evidence'` | Portabel, mudah dibaca, memiliki metode built-in |

**Keunggulan `pathlib.Path` dalam konteks security scripting:**

```python
p = Path.home() / 'mod4' / 'evidence' / 'output.txt'
p.write_text(content)          # Menulis file
p.read_text()                  # Membaca file
p.exists()                     # Memeriksa keberadaan file
p.parent.mkdir(parents=True)   # Membuat direktori parent jika belum ada
p.stat()                       # Membaca metadata file
```

**Mengapa JSON lebih baik dari output teks acak:**

Output JSON dapat dibaca kembali oleh script lain menggunakan `json.loads()`, diproses oleh tool seperti `jq` di terminal, atau diintegrasikan ke dalam sistem lain. Output teks acak memerlukan parsing custom setiap kali digunakan kembali — sebuah pemborosan yang terakumulasi dalam workflow yang kompleks.

**Contoh output `json_yaml_notes.txt`:**
```json
{
  "host": "ubuntu-lab",
  "service": "local-target",
  "ports": [3000, 8081],
  "status": "ready"
}
```

### 2. Argument Handling dengan argparse

```python
# argparse_demo.py
import argparse

parser = argparse.ArgumentParser(description='Simple endpoint checker')
parser.add_argument('--target', required=True, help='Target URL to check')
args = parser.parse_args()

print(f'Target received: {args.target}')
```

```bash
python ~/mod4/tmp/argparse_demo.py --target http://127.0.0.1:3000 | \
    tee ~/mod4/evidence/foundations/argparse_demo.txt
```

**Mengapa `argparse` penting dalam security scripting:**

Script yang menggunakan *hardcoded values* — URL, path file, atau parameter lain yang ditulis langsung dalam kode — memiliki masalah fundamental: untuk digunakan pada target yang berbeda, kode harus diedit. Ini berarti:

1. Risiko tidak sengaja mengubah logika script saat mengedit nilai
2. Tidak dapat dijalankan dengan mudah dalam batch atau pipeline
3. Sulit diaudit karena perubahan target tidak tercatat di parameter, melainkan di dalam kode

`argparse` menyelesaikan semua masalah ini dengan memindahkan konfigurasi dari kode ke parameter runtime:

```bash
# Hardcoded — harus edit kode untuk mengubah target
python checker.py

# argparse — target ditentukan saat menjalankan, kode tidak berubah
python checker.py --target http://127.0.0.1:3000
python checker.py --target http://127.0.0.1:8081
python checker.py --target http://internal-app.lab:9000
```

**Fitur `argparse` yang berguna dalam security tools:**

| Fitur | Contoh | Kegunaan |
|-------|--------|----------|
| `required=True` | `add_argument('--target', required=True)` | Memastikan parameter wajib selalu diberikan |
| `default=` | `add_argument('--timeout', default=5)` | Nilai default yang masuk akal |
| `type=int` | `add_argument('--port', type=int)` | Validasi tipe secara otomatis |
| `help=` | `add_argument('--target', help='Target URL')` | Dokumentasi built-in (`--help`) |
| `choices=` | `add_argument('--method', choices=['GET','POST'])` | Membatasi nilai yang diterima |

### 3. File Input dan Output yang Terkendali

```python
# file_io_demo.py
from pathlib import Path

src = Path.home() / 'datasets' / 'phishing' / 'sample_email_1.txt'
out = Path.home() / 'mod4' / 'evidence' / 'foundations' / 'file_io_review.txt'

text = src.read_text(errors='ignore')
out.write_text(text[:400])
print(out.read_text())
```

```bash
python ~/mod4/tmp/file_io_demo.py
```

**Mengapa `errors='ignore'` digunakan:**

File yang berasal dari sumber yang tidak diketahui (email phishing, log dari sistem lain, artefak forensik) sering mengandung karakter yang tidak valid dalam encoding UTF-8. Tanpa `errors='ignore'`, Python akan melempar `UnicodeDecodeError` dan script berhenti. Dengan `errors='ignore'`, karakter yang tidak dapat didekode akan dilewati — perilaku yang aman untuk analisis teks.

**Pembatasan output ke 400 karakter pertama:**

Dalam analisis artefak, menyimpan seluruh konten file besar sebagai evidence seringkali tidak perlu dan dapat menghabiskan storage. Menyimpan potongan awal (preview) sudah cukup untuk mendokumentasikan bahwa file telah dibaca dan untuk menunjukkan karakteristik kontennya.

## Hasil dan Output

**Contoh output `argparse_demo.txt`:**
```
Target received: http://127.0.0.1:3000
```

**Contoh output `file_io_review.txt`:**
```
From: security-team@urgent-verify.com
To: employee@company.com
Subject: URGENT: Your account will be suspended

Dear Valued Customer,

We have detected suspicious activity on your account. You must verify
your credentials immediately to avoid suspension. Click the link below...
```

## Temuan Utama

Demo `file_io_review.txt` langsung mengungkap beberapa indikator phishing yang terlihat jelas bahkan dari 400 karakter pertama: domain pengirim (`urgent-verify.com`) yang tidak cocok dengan klaim organisasi legitimate, kata "URGENT" dalam subject, dan bahasa yang menciptakan urgensi dalam body email. Ini menunjukkan bahwa bahkan preview singkat dari sebuah artefak sudah dapat memberikan nilai analitis yang signifikan.

## Apa yang Dipelajari

Tiga fondasi yang dipraktikkan — `pathlib`, `json`, `argparse` — bukanlah fitur yang menarik secara teknis. Namun ketiganya adalah komponen yang akan muncul di hampir setiap script security yang ditulis dengan benar. Menguasai ketiganya dengan baik jauh lebih berharga daripada mengetahui banyak library secara dangkal.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `json_yaml_notes.txt` | `~/mod4/evidence/foundations/` | Output JSON terformat dari demo structured data |
| `argparse_demo.txt` | `~/mod4/evidence/foundations/` | Output demo argument handling |
| `file_io_review.txt` | `~/mod4/evidence/foundations/` | Potongan 400 karakter pertama dari phishing sample |

---
---

# Proyek 3 — Membangun Automation Workflow untuk Endpoint Validation

## Latar Belakang

Validasi endpoint — memeriksa apakah sebuah service aktif, merespons dengan benar, dan mengembalikan status yang diharapkan — adalah aktivitas yang dilakukan berulang kali dalam operasi keamanan. Melakukannya secara manual satu per satu efisien untuk satu atau dua target, namun tidak scalable ketika jumlah target bertambah atau ketika validasi harus dilakukan secara berkala.

Automation memecahkan masalah scalability ini, namun dengan satu syarat yang sering diabaikan: output automation harus dapat dipercaya dan dapat divalidasi. Script yang menghasilkan output yang tidak dapat diverifikasi oleh tool lain — atau yang tidak menyimpan hasilnya untuk referensi di kemudian hari — tidak jauh berbeda kualitasnya dengan validasi manual yang tidak dicatat.

Proyek ini membangun workflow validasi endpoint yang lengkap: script Python yang mengecek status setiap target dan menyimpan hasilnya dalam format yang dapat dibaca ulang, diikuti oleh validasi manual dari Kali Linux untuk mengkonfirmasi bahwa output script konsisten dengan observasi independen.

## Tujuan

- Membangun endpoint checker yang menangani multiple target, timeout, dan error secara bersih
- Menyimpan output dalam format yang dapat dibaca ulang dan digunakan sebagai evidence
- Memvalidasi hasil script dari perspektif eksternal menggunakan Kali Linux
- Mendokumentasikan pola kerja automation yang dapat direplikasi dan diperluas

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Pengembangan dan eksekusi endpoint checker |
| **Kali Linux** | Validasi independen hasil endpoint checker |

## Langkah-Langkah yang Dilakukan

### 1. Endpoint Checker dengan Python

```python
# endpoint_checker.py
import requests
from pathlib import Path

TARGETS = [
    'http://127.0.0.1:3000',
    'http://127.0.0.1:8081'
]

out = Path.home() / 'mod4' / 'evidence' / 'automation' / 'requests_output.txt'

lines = []
for target in TARGETS:
    try:
        r = requests.get(target, timeout=5)
        lines.append(f'{target} | status={r.status_code} | len={len(r.text)}')
    except requests.exceptions.ConnectionError:
        lines.append(f'{target} | error=ConnectionRefused')
    except requests.exceptions.Timeout:
        lines.append(f'{target} | error=Timeout')
    except Exception as e:
        lines.append(f'{target} | error={type(e).__name__}: {e}')

out.write_text('\n'.join(lines))
print(out.read_text())
```

```bash
python ~/mod4/tmp/endpoint_checker.py
```

**Analisis komponen script:**

**`timeout=5`:**
Timeout adalah parameter yang wajib ada dalam setiap HTTP request dalam script automation. Tanpa timeout, script akan menunggu selamanya jika server tidak merespons — memblokir eksekusi dan menghasilkan script yang tampak "hang" tanpa pesan error yang jelas.

**Exception handling yang spesifik:**

| Exception | Penyebab | Pesan yang Dihasilkan |
|-----------|----------|----------------------|
| `ConnectionError` | Port tertutup atau host tidak dapat dijangkau | `error=ConnectionRefused` |
| `Timeout` | Server tidak merespons dalam batas waktu | `error=Timeout` |
| `Exception` (generic) | Error tidak terduga lainnya | `error=NamaException: detail` |

Menangkap exception secara spesifik (bukan hanya `except Exception`) memungkinkan script menghasilkan pesan error yang informatif dan dapat langsung dipahami tanpa perlu membaca traceback Python yang panjang.

**Format output yang dipilih:**

```
http://127.0.0.1:3000 | status=200 | len=4523
http://127.0.0.1:8081 | status=200 | len=1892
```

Format dengan delimiter `|` dipilih karena dapat diproses dengan `awk` atau `cut` di shell, sehingga output ini dapat langsung digunakan dalam pipeline shell tanpa perlu script tambahan.

### 2. Validasi Hasil dari Kali Linux

```bash
curl -I http://127.0.0.1:3000 | tee  ~/mod4/evidence/automation/endpoint_validation.txt
curl -I http://127.0.0.1:8081 | tee -a ~/mod4/evidence/automation/endpoint_validation.txt
nmap -Pn -sV -p 3000,8081 127.0.0.1 | tee -a ~/mod4/evidence/automation/endpoint_validation.txt
```

**Tujuan validasi dari Kali:**

Script Python yang berjalan di Ubuntu mengakses endpoint dari dalam host yang sama. Validasi dari Kali memberikan perspektif yang berbeda: apakah service yang sama dapat dijangkau dari workstation yang terpisah? Jika ada perbedaan antara hasil script Ubuntu dan validasi Kali, ini mengindikasikan adanya konfigurasi firewall, binding address yang berbeda, atau masalah jaringan yang perlu diselidiki.

**Matriks perbandingan yang diharapkan:**

| Endpoint | Python (Ubuntu) | curl (Kali) | nmap (Kali) | Status |
|----------|----------------|-------------|-------------|--------|
| `:3000` | status=200 | HTTP 200 OK | open/http | ✅ Konsisten |
| `:8081` | status=200 | HTTP 200 OK | open/http | ✅ Konsisten |

Ketika ketiganya menunjukkan hasil yang konsisten, kepercayaan terhadap status endpoint meningkat secara signifikan.

### 3. Dokumentasi Automation Workflow

```bash
cat > ~/mod4/evidence/automation/automation_notes.md <<'EOF'
# Automation Workflow Notes

## Pembagian Peran dalam Workflow Ini
- **Ubuntu**: Pengembangan dan eksekusi script Python
- **Kali**: Validasi independen hasil script dari perspektif workstation assessment

## Prinsip Output yang Baik
- Simpan seluruh output ke file, jangan hanya tampilkan di terminal
- Gunakan format yang dapat diproses ulang (JSON, delimiter yang konsisten)
- Sertakan timestamp dalam output untuk referensi waktu

## Prinsip Error Handling dalam Automation
- Tangkap exception secara spesifik — hindari bare `except:`
- Tulis pesan error yang cukup informatif untuk debugging tanpa terlalu verbose
- Script harus tetap berjalan meski satu target gagal (jangan berhenti di exception pertama)

## Validasi Silang (Cross-Validation)
- Output automation selalu divalidasi dengan observasi manual dari perspektif yang berbeda
- Perbedaan antara hasil script dan hasil manual adalah sinyal untuk diselidiki, bukan diabaikan
EOF
```

## Hasil dan Output

**Contoh isi `requests_output.txt`:**
```
http://127.0.0.1:3000 | status=200 | len=4523
http://127.0.0.1:8081 | status=200 | len=1892
```

**Contoh isi `endpoint_validation.txt`:**
```
HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: text/html; charset=utf-8

HTTP/1.1 200 OK
Server: nginx/1.18.0
Content-Type: text/html

PORT     STATE SERVICE VERSION
3000/tcp open  http    Node.js Express framework
8081/tcp open  http    nginx 1.18.0
```

## Temuan Utama

Validasi silang antara output Python script dan validasi Kali menghasilkan hasil yang konsisten: kedua endpoint merespons dengan status 200 dan dapat dijangkau dari kedua host. Header dari `curl` mengungkap informasi tambahan yang tidak dikumpulkan oleh script Python sederhana: `X-Powered-By: Express` dan versi nginx yang digunakan — informasi yang relevan untuk fingerprinting.

Ini menunjukkan bahwa script Python dan `curl`/`nmap` bukan alat yang saling menggantikan, melainkan saling melengkapi: script Python unggul dalam kecepatan, konsistensi, dan kemampuan menyimpan output; `curl` dan `nmap` unggul dalam detail inspeksi yang lebih dalam.

## Apa yang Dipelajari

Automation workflow yang baik bukan hanya tentang *menulis script yang bekerja* — ini tentang membangun sistem di mana hasilnya dapat dipercaya karena telah divalidasi dari perspektif yang berbeda. Script yang tidak pernah divalidasi secara independen adalah script yang belum terbukti andal.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `requests_output.txt` | `~/mod4/evidence/automation/` | Output endpoint checker — status dan panjang respons |
| `endpoint_validation.txt` | `~/mod4/evidence/automation/` | Validasi dari Kali: curl headers dan nmap scan |
| `automation_notes.md` | `~/mod4/evidence/automation/` | Dokumentasi prinsip dan pola automation workflow |

---
---

# Proyek 4 — Menerapkan Secure Coding pada Script Keamanan

## Latar Belakang

Ironi dalam keamanan siber adalah bahwa tool yang dibangun untuk melindungi sistem sering kali memiliki kerentanan yang sama dengan sistem yang mereka lindungi. Script yang tidak memvalidasi input dapat dimanipulasi untuk memproses data berbahaya. Query database yang menggunakan string concatenation rentan terhadap SQL injection. Error handling yang terlalu verbose dapat membocorkan informasi sensitif tentang struktur sistem kepada penyerang.

*Secure coding* bukan tentang menghasilkan kode yang sempurna — ini tentang membangun kebiasaan kecil yang secara konsisten mengurangi permukaan serangan pada setiap script yang ditulis. Proyek ini mempraktikkan tiga prinsip secure coding yang paling relevan dalam konteks script keamanan: validasi input, parameterized query, dan error handling yang informatif namun tidak berlebihan.

## Tujuan

- Mengimplementasikan validasi URL menggunakan `urllib.parse` untuk mencegah input yang tidak valid
- Menggunakan parameterized query dalam SQLite untuk mencegah SQL injection
- Membangun error handling yang memberikan pesan yang berguna tanpa membocorkan detail implementasi
- Memahami mengapa ketiga prinsip ini relevan bahkan untuk script kecil yang dijalankan secara internal

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Seluruh demo secure coding dilakukan di sini |

## Langkah-Langkah yang Dilakukan

### 1. Input Validation — Validasi URL Sebelum Digunakan

```python
# input_validation_demo.py
from urllib.parse import urlparse

def validate_target(url: str) -> bool:
    """Validasi bahwa URL memiliki skema yang diizinkan dan host yang valid."""
    parsed = urlparse(url)
    return parsed.scheme in {'http', 'https'} and bool(parsed.netloc)

samples = [
    'http://127.0.0.1:3000',    # Valid
    '127.0.0.1:8081',           # Tidak valid — tidak ada skema
    'ftp://example.com',        # Tidak valid — skema tidak diizinkan
    'file:///etc/passwd',       # Tidak valid — skema berbahaya
    'https://lab.internal',     # Valid
]

for s in samples:
    status = '✓ VALID' if validate_target(s) else '✗ INVALID'
    print(f'{status:12} {s}')
```

```bash
python ~/mod4/tmp/input_validation_demo.py | tee ~/mod4/evidence/secure_coding/input_validation_notes.txt
```

**Mengapa validasi input kritis bahkan untuk script internal:**

Script keamanan sering menerima input dari sumber yang tidak sepenuhnya terpercaya: file konfigurasi, variabel environment, parameter command line, atau bahkan output dari script lain. Tanpa validasi, URL seperti `file:///etc/passwd` atau `javascript:alert(1)` dapat diteruskan ke fungsi yang seharusnya hanya menerima HTTP/HTTPS URL — dengan konsekuensi yang tidak terduga.

**Apa yang dilakukan `urlparse`:**

```python
from urllib.parse import urlparse
result = urlparse('http://127.0.0.1:3000/api/users')
# result.scheme   = 'http'
# result.netloc   = '127.0.0.1:3000'
# result.path     = '/api/users'
# result.hostname = '127.0.0.1'
# result.port     = 3000
```

`urlparse` mendekomposisi URL menjadi komponen-komponennya, memungkinkan validasi yang presisi terhadap bagian mana pun dari URL.

**Contoh output:**
```
✓ VALID      http://127.0.0.1:3000
✗ INVALID    127.0.0.1:8081
✗ INVALID    ftp://example.com
✗ INVALID    file:///etc/passwd
✓ VALID      https://lab.internal
```

### 2. Parameterized Query — Mencegah SQL Injection

```python
# safe_query_demo.py
import sqlite3
from pathlib import Path

db = Path.home() / 'mod4' / 'tmp' / 'users.db'
conn = sqlite3.connect(db)
cur  = conn.cursor()

# Setup tabel
cur.execute('CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, username TEXT)')
cur.execute('DELETE FROM users')
cur.executemany('INSERT INTO users (username) VALUES (?)', [('alice',), ('bob',)])

# Query yang aman menggunakan parameterized query
user_id = '1'
cur.execute('SELECT * FROM users WHERE id = ?', (user_id,))
print('Hasil query aman:', cur.fetchall())

conn.commit()
conn.close()
```

```bash
python ~/mod4/tmp/safe_query_demo.py | tee ~/mod4/evidence/secure_coding/safe_query_demo.txt
```

**Perbandingan query tidak aman vs aman:**

**Query tidak aman (string concatenation):**
```python
# JANGAN LAKUKAN INI
query = f"SELECT * FROM users WHERE id = {user_id}"
cur.execute(query)
```

Jika `user_id` berisi `1 OR 1=1`, query menjadi:
```sql
SELECT * FROM users WHERE id = 1 OR 1=1
```
Yang akan mengembalikan **semua** baris dalam tabel — SQL injection klasik.

**Query aman (parameterized):**
```python
# LAKUKAN INI
cur.execute('SELECT * FROM users WHERE id = ?', (user_id,))
```

Dengan parameterized query, `user_id` diperlakukan sebagai **data literal**, bukan sebagai bagian dari perintah SQL. String `1 OR 1=1` akan dicari secara harfiah sebagai nilai ID, bukan diinterpretasikan sebagai perintah SQL.

**Prinsip yang sama berlaku di semua database:**

| Database | Placeholder | Contoh |
|----------|-------------|--------|
| SQLite | `?` | `WHERE id = ?` |
| MySQL/PostgreSQL | `%s` | `WHERE id = %s` |
| SQLAlchemy ORM | Named params | `WHERE id = :user_id` |

### 3. Error Handling yang Informatif namun Tidak Berlebihan

```python
# error_handling_demo.py
from pathlib import Path

try:
    data = Path.home() / 'datasets' / 'nonexistent' / 'missing.txt'
    print(data.read_text())
except FileNotFoundError:
    print('Input file not found. Review dataset path before rerunning the script.')
except PermissionError:
    print('Permission denied. Ensure the file is readable by the current user.')
except Exception as e:
    print(f'Unexpected error: {type(e).__name__}. Contact lab administrator.')
```

```bash
python ~/mod4/tmp/error_handling_demo.py | tee ~/mod4/evidence/secure_coding/error_handling_notes.txt
```

**Spektrum error handling dan risikonya:**

| Pendekatan | Contoh Output | Masalah |
|------------|---------------|---------|
| Terlalu sedikit | *(tidak ada output, script diam)* | Tidak dapat di-debug, tidak ada informasi untuk tindak lanjut |
| Tepat | `"Input file not found. Review dataset path."` | Informatif, dapat ditindaklanjuti, tidak membocorkan detail |
| Terlalu banyak | `"FileNotFoundError: [Errno 2] No such file or directory: '/home/student/datasets/nonexistent/missing.txt'"` | Membocorkan path absolut sistem, berpotensi membantu penyerang |

**Contoh output:**
```
Input file not found. Review dataset path before rerunning the script.
```

Pesan ini memberikan informasi yang cukup bagi pengguna yang sah untuk menyelesaikan masalah (cek path dataset), namun tidak membocorkan detail yang dapat dimanfaatkan jika output ini ter-log atau terekspos.

## Temuan Utama

Ketiga prinsip secure coding yang dipraktikkan — validasi input, parameterized query, dan error handling yang tepat — memiliki satu tema yang sama: **jangan percaya input secara implisit dan jangan mengekspos implementasi secara berlebihan**. Ketiganya dapat diterapkan dengan overhead kode yang minimal namun memberikan perlindungan yang signifikan.

Demo `input_validation` secara khusus mengungkap bahwa URL `file:///etc/passwd` yang diberikan sebagai input akan diklasifikasikan sebagai tidak valid dan tidak akan diproses lebih lanjut — sebuah proteksi sederhana yang mencegah script dimanipulasi untuk membaca file sistem yang seharusnya tidak diakses.

## Apa yang Dipelajari

*Secure coding* bukan tentang menulis kode yang lebih kompleks — ini tentang menulis kode yang lebih sadar. Setiap input yang diterima tanpa validasi adalah permukaan serangan yang potensial. Setiap query yang dibentuk dengan string concatenation adalah kerentanan yang menunggu untuk dieksploitasi. Setiap error message yang terlalu detail adalah informasi yang membantu penyerang memahami sistem. Kebiasaan kecil yang dipraktikkan secara konsisten inilah yang membangun postur keamanan yang nyata.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `input_validation_notes.txt` | `~/mod4/evidence/secure_coding/` | Hasil klasifikasi valid/tidak valid dari 5 URL sampel |
| `safe_query_demo.txt` | `~/mod4/evidence/secure_coding/` | Output parameterized query SQLite |
| `error_handling_notes.txt` | `~/mod4/evidence/secure_coding/` | Output error handling yang informatif |

---
---

# Proyek 5 — Membangun Threat-Aware Script untuk Analisis IOC dan Artefak

## Latar Belakang

*Indicator of Compromise* (IOC) adalah artefak yang mengindikasikan bahwa sebuah sistem mungkin telah dikompromikan atau sedang menjadi target serangan. IOC dapat berupa alamat IP, domain, URL, alamat email, hash file, atau pola teks tertentu yang ditemukan dalam komunikasi atau file yang mencurigakan.

Ekstraksi IOC secara manual dari artefak seperti email phishing, log sistem, atau dokumen yang mencurigakan adalah pekerjaan yang repetitif dan rawan terlewat. Script yang dirancang dengan *threat awareness* — pemahaman tentang apa yang dicari dan mengapa — dapat mengotomatisasi ekstraksi ini dan menghasilkan ringkasan yang dapat langsung digunakan dalam investigasi.

Proyek ini membangun tiga komponen threat-aware scripting: IOC parser untuk mengekstrak email dan keyword dari artefak teks, hash review untuk verifikasi integritas file, dan YARA rule untuk deteksi berbasis pola yang dapat diterapkan pada banyak file sekaligus.

## Tujuan

- Membangun IOC parser menggunakan regex untuk mengekstrak email dan keyword yang relevan dari artefak
- Menggunakan `sha256sum` untuk menghasilkan hash sebagai identitas kriptografis file evidence
- Menulis dan menerapkan YARA rule untuk deteksi pola berbasis keyword pada artefak
- Memahami hubungan antara ketiga teknik ini dalam alur kerja *threat intelligence* dan *incident response*

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Seluruh analisis IOC, hash review, dan YARA dijalankan di sini |

## Langkah-Langkah yang Dilakukan

### 1. IOC Parser — Ekstraksi Email dan Keyword dengan Regex

```python
# ioc_parser.py
import re
from pathlib import Path

src  = Path.home() / 'datasets' / 'phishing' / 'sample_email_1.txt'
text = src.read_text(errors='ignore')

# Ekstraksi alamat email
email_pattern = re.compile(r'[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}')
emails = email_pattern.findall(text)

# Deteksi keyword berbahaya
danger_keywords = ['urgent', 'verify', 'vpn', 'password', 'suspended', 'click']
found_keywords  = [w for w in text.split() if any(kw in w.lower() for kw in danger_keywords)]

print('=== IOC Extraction Results ===')
print(f'Emails found   : {emails}')
print(f'Danger keywords: {found_keywords[:10]}')  # Batasi tampilan ke 10 pertama
```

```bash
python ~/mod4/tmp/ioc_parser.py | tee ~/mod4/evidence/analysis/ioc_parser_output.txt
```

**Regex untuk ekstraksi email — analisis pola:**

```
[A-Za-z0-9._%+-]+  →  local part (karakter yang valid sebelum @)
@                   →  karakter @ literal
[A-Za-z0-9.-]+     →  domain name
\.                  →  titik literal
[A-Za-z]{2,}       →  TLD (minimal 2 karakter: .com, .id, .net, dll.)
```

**Keterbatasan regex untuk ekstraksi email:**

Regex ini tidak sempurna — email dengan format yang tidak biasa (menggunakan karakter Unicode atau format khusus) mungkin tidak terdeteksi. Namun untuk analisis awal pada artefak phishing, regex ini sudah cukup untuk mengidentifikasi kandidat IOC yang perlu diselidiki lebih lanjut.

**Mengapa keyword-based extraction berguna:**

Penyerang dalam kampanye phishing cenderung menggunakan kata-kata tertentu yang terbukti efektif dalam memancing korban: "urgent", "verify", "suspended", "password reset". Keberadaan beberapa kata ini dalam satu dokumen meningkatkan probabilitas bahwa dokumen tersebut adalah artefak phishing — sebuah sinyal yang dapat digunakan untuk *triage* otomatis.

**Contoh output `ioc_parser_output.txt`:**
```
=== IOC Extraction Results ===
Emails found   : ['security-team@urgent-verify.com', 'noreply@fakebank-alert.net']
Danger keywords: ['URGENT:', 'verify', 'suspended', 'password', 'click', 'urgent']
```

Dua domain yang ditemukan — `urgent-verify.com` dan `fakebank-alert.net` — adalah IOC yang dapat langsung dimasukkan ke dalam threat intelligence platform atau digunakan untuk query di SIEM.

### 2. Hash Review — Verifikasi Integritas File

```bash
sha256sum ~/datasets/phishing/sample_email_1.txt | tee  ~/mod4/evidence/analysis/hash_review.txt
sha256sum ~/datasets/logs/linux/auth.log.sample   | tee -a ~/mod4/evidence/analysis/hash_review.txt
```

**Kegunaan SHA256 hash dalam operasi keamanan:**

| Use Case | Cara Kerja |
|----------|------------|
| **Evidence integrity** | Hash sebelum dan sesudah investigasi — jika berbeda, file telah dimodifikasi |
| **Malware identification** | Hash file mencurigakan, bandingkan dengan database threat intelligence |
| **Deduplication** | Identifikasi file yang identik berdasarkan hash, bukan nama |
| **Chain of custody** | Dokumentasi hash pada setiap transfer evidence |
| **IOC sharing** | Hash adalah format IOC yang universal dan tidak mengandung data sensitif |

**Mengapa SHA256, bukan MD5 atau SHA1:**

MD5 dan SHA1 telah terbukti memiliki kelemahan kriptografis (*collision vulnerability*) — dua file berbeda dapat menghasilkan hash yang sama, yang berarti hash tidak dapat diandalkan sebagai identitas unik. SHA256 saat ini adalah standar minimum yang diterima dalam konteks forensik dan keamanan.

**Contoh output `hash_review.txt`:**
```
a3f92c1d4b8e7f2a1c9d0e5b6f3a8c2d1e4f7b9a0...  /home/student/datasets/phishing/sample_email_1.txt
8b2e4f9c1a7d3e6b0f5c8a2d4e7f1b9c3a6d8e0f2...  /home/student/datasets/logs/linux/auth.log.sample
```

### 3. YARA Rule — Deteksi Berbasis Pola

```bash
cat > ~/mod4/tmp/basic_keywords.yar <<'EOF'
rule basic_keywords {
    meta:
        description = "Deteksi keyword yang umum dalam phishing dan social engineering"
        author      = "Student Lab — Modul 04"
        date        = "2024-04-27"
        severity    = "low"

    strings:
        $urgent   = "urgent"   nocase
        $verify   = "verify"   nocase
        $password = "password" nocase

    condition:
        any of them
}
EOF

yara ~/mod4/tmp/basic_keywords.yar ~/datasets/phishing/sample_email_1.txt | \
    tee ~/mod4/evidence/analysis/yara_helper_output.txt
```

**Perbedaan YARA dengan grep biasa:**

| Aspek | `grep` | YARA |
|-------|--------|------|
| Definisi pola | Inline saat dijalankan | Dalam file rule yang terpisah dan dapat di-versioning |
| Multiple patterns | Memerlukan multiple `grep` atau regex kompleks | Satu rule dapat berisi banyak string dengan logika kondisional |
| Skalabilitas | Sulit dikelola jika patterns banyak | Rule dapat dikelompokkan, di-import, dan dishare |
| Metadata | Tidak ada | Rule dapat menyertakan metadata (author, date, severity) |
| Output | Hanya baris yang cocok | Nama rule + file yang match — lebih mudah untuk reporting |
| Ekosistem | Standar Unix | Standar industri keamanan — digunakan di Virus Total, SIEM, EDR |

**Contoh output `yara_helper_output.txt`:**
```
basic_keywords /home/student/datasets/phishing/sample_email_1.txt
```

Output ini mengkonfirmasi bahwa setidaknya satu string yang didefinisikan dalam rule (`urgent`, `verify`, atau `password`) ditemukan dalam file phishing sample.

## Temuan Utama

IOC parser berhasil mengekstrak dua domain email yang mencurigakan dari phishing sample: `urgent-verify.com` dan `fakebank-alert.net`. Kedua domain ini menggunakan pola penamaan yang umum dalam phishing — menggabungkan kata-kata yang menciptakan urgensi (`urgent`, `alert`) dengan kata-kata yang menyerupai nama institusi (`verify`, `bank`).

Kombinasi ketiga teknik — IOC extraction, hash verification, dan YARA scanning — menciptakan alur kerja analisis artefak yang komprehensif: IOC extraction menemukan *apa* yang mencurigakan, hash verification memastikan *integritas* artefak, dan YARA memberikan *konfirmasi berbasis rule* yang dapat direproduksi dan dishare dengan tim lain.

## Apa yang Dipelajari

*Threat-aware scripting* adalah tentang membangun tool dengan memahami konteks ancamannya: mengapa IOC tertentu relevan, mengapa integritas file harus diverifikasi, dan mengapa deteksi berbasis rule lebih scalable daripada review manual. Pemahaman konteks ini adalah yang membedakan script keamanan yang benar-benar berguna dari script yang hanya "bekerja secara teknis".

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `ioc_parser_output.txt` | `~/mod4/evidence/analysis/` | Email dan keyword yang diekstrak dari phishing sample |
| `hash_review.txt` | `~/mod4/evidence/analysis/` | SHA256 hash dari dua file dataset |
| `yara_helper_output.txt` | `~/mod4/evidence/analysis/` | Hasil deteksi YARA pada phishing sample |

---
---

# Proyek 6 — Mengembangkan Mini Tool Keamanan dengan Output Terstruktur

## Latar Belakang

Perbedaan antara *script* dan *tool* terletak pada kesiapan penggunaannya: script adalah kode yang bekerja; tool adalah kode yang dapat digunakan kembali oleh orang lain (atau oleh diri sendiri di masa depan) tanpa perlu membaca dan memahami implementasinya terlebih dahulu. Tool yang baik memiliki input yang jelas, proses yang dapat diprediksi, dan output yang berguna — bahkan jika orang yang menggunakannya bukan orang yang menulisnya.

Proyek ini mengintegrasikan seluruh komponen yang telah dipraktikkan — file I/O dengan `pathlib`, HTTP request dengan `requests`, error handling yang tepat, dan output terstruktur — menjadi satu mini tool yang memiliki karakteristik tool yang sesungguhnya. Hasilnya kemudian divalidasi dari Kali Linux untuk memverifikasi konsistensinya.

## Tujuan

- Mengintegrasikan seluruh komponen Python yang telah dipelajari menjadi satu tool yang kohesif
- Menghasilkan output yang disimpan ke file laporan yang dapat dipakai ulang
- Memvalidasi output tool dari perspektif Kali Linux untuk memastikan konsistensi
- Memahami karakteristik yang membedakan tool yang siap pakai dari script yang hanya bekerja sekali

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Pengembangan dan eksekusi mini tool |
| **Kali Linux** | Validasi output tool dari perspektif assessment |

## Langkah-Langkah yang Dilakukan

### 1. Pengembangan Mini Tool

```python
# mini_tool.py
"""
Mini Security Tool — Endpoint Summary Generator
Mengecek status endpoint yang ditentukan dan menghasilkan laporan ringkas.
"""

import requests
from datetime import datetime
from pathlib import Path

TARGETS = [
    'http://127.0.0.1:3000',
    'http://127.0.0.1:8081'
]

out = Path.home() / 'mod4' / 'reports' / 'mini_tool_report.txt'

def check_endpoint(url: str, timeout: int = 5) -> dict:
    """Cek satu endpoint dan kembalikan hasilnya sebagai dictionary."""
    try:
        r = requests.get(url, timeout=timeout)
        return {
            'url'        : url,
            'status'     : r.status_code,
            'length'     : len(r.text),
            'server'     : r.headers.get('Server', 'unknown'),
            'powered_by' : r.headers.get('X-Powered-By', 'unknown'),
        }
    except requests.exceptions.ConnectionError:
        return {'url': url, 'status': 'error', 'reason': 'ConnectionRefused'}
    except requests.exceptions.Timeout:
        return {'url': url, 'status': 'error', 'reason': 'Timeout'}
    except Exception as e:
        return {'url': url, 'status': 'error', 'reason': str(e)}

# Jalankan pengecekan
timestamp = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
results   = [check_endpoint(t) for t in TARGETS]

# Format output
lines = [f'=== Endpoint Summary Report ===', f'Generated: {timestamp}', '']
for r in results:
    if r.get('status') == 'error':
        lines.append(f"[ERROR]  {r['url']} — {r['reason']}")
    else:
        lines.append(f"[OK]     {r['url']}")
        lines.append(f"         Status: {r['status']} | Length: {r['length']} bytes")
        lines.append(f"         Server: {r['server']} | Powered-By: {r['powered_by']}")
    lines.append('')

report = '\n'.join(lines)
out.write_text(report)
print(report)
```

```bash
python ~/mod4/tmp/mini_tool.py
```

**Karakteristik tool yang baik — yang diterapkan dalam mini tool ini:**

| Karakteristik | Implementasi |
|---------------|-------------|
| Input yang jelas | Daftar target didefinisikan di satu tempat (`TARGETS`) yang mudah ditemukan |
| Output yang tersimpan | Laporan disimpan ke `mini_tool_report.txt` — bukan hanya ditampilkan di terminal |
| Timestamp | Setiap laporan mencatat kapan dihasilkan untuk referensi waktu |
| Error yang informatif | Error message jelas tanpa traceback Python yang panjang |
| Fungsi yang terpisah | `check_endpoint()` dapat digunakan kembali atau dimodifikasi secara independen |
| Komentar docstring | Fungsi memiliki docstring yang menjelaskan tujuannya |

**Mengapa timestamp penting dalam laporan tool:**

Laporan tanpa timestamp tidak dapat dikaitkan dengan kondisi sistem pada waktu tertentu. Jika laporan menunjukkan status 200 namun service kemudian dilaporkan down, tidak ada cara untuk mengetahui kapan perubahan terjadi tanpa timestamp yang tercatat.

### 2. Validasi Output dari Kali Linux

```bash
# Baca laporan yang dihasilkan tool
cat ~/mod4/reports/mini_tool_report.txt

# Validasi independen dari Kali
curl -I http://127.0.0.1:3000 | head -n 5
curl -I http://127.0.0.1:8081 | head -n 5
```

**Tujuan validasi dari perspektif Kali:**

Mini tool mengumpulkan header `Server` dan `X-Powered-By` dari respons HTTP. Validasi dari Kali menggunakan `curl -I` memungkinkan perbandingan langsung: apakah nilai yang dicatat oleh tool identik dengan yang terlihat dari workstation assessment?

Jika ada perbedaan — misalnya tool mencatat `Server: nginx/1.18.0` namun `curl` dari Kali menunjukkan nilai yang berbeda — ini mengindikasikan kemungkinan adanya reverse proxy, load balancer, atau konfigurasi yang menyebabkan respons berbeda tergantung sumber request.

## Hasil dan Output

**Contoh isi `mini_tool_report.txt`:**
```
=== Endpoint Summary Report ===
Generated: 2024-04-27 10:45:23

[OK]     http://127.0.0.1:3000
         Status: 200 | Length: 4523 bytes
         Server: unknown | Powered-By: Express

[OK]     http://127.0.0.1:8081
         Status: 200 | Length: 1892 bytes
         Server: nginx/1.18.0 | Powered-By: unknown
```

**Perbandingan dengan output `curl -I` dari Kali:**
```
HTTP/1.1 200 OK
X-Powered-By: Express
Server: (tidak ada dalam header — konsisten dengan "unknown")

HTTP/1.1 200 OK
Server: nginx/1.18.0
(X-Powered-By tidak ada — konsisten dengan "unknown")
```

Kedua output konsisten — tool mengumpulkan data yang akurat dan dapat dipercaya.

## Temuan Utama

Mini tool berhasil mengungkap bahwa meskipun kedua endpoint merespons dengan status 200, karakteristiknya berbeda: port 3000 dijalankan oleh Node.js Express (terlihat dari header `X-Powered-By`) dan tidak mengekspos informasi server, sementara port 8081 menggunakan nginx dan tidak mengekspos framework backend. Perbedaan ini relevan untuk assessment karena masing-masing teknologi memiliki profil kerentanan yang berbeda.

## Apa yang Dipelajari

Membangun tool yang benar-benar dapat digunakan kembali membutuhkan disiplin lebih dari sekadar membuat script yang bekerja. Timestamp, output yang tersimpan, error handling yang jelas, dan fungsi yang terpisah bukan "fitur tambahan" — ini adalah komponen yang menentukan apakah tool dapat diandalkan dalam operasi nyata.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `mini_tool_report.txt` | `~/mod4/reports/` | Laporan endpoint summary dengan timestamp |
| `mini_tool.py` | `~/mod4/tmp/` | Source code mini tool |

---
---

# Proyek 7 — Menyusun Automation Assessment dan Jembatan ke Modul Berikutnya

## Latar Belakang

Setiap rangkaian kerja teknis yang tidak ditutup dengan penilaian yang terstruktur berisiko kehilangan nilainya. Script yang berjalan dengan baik namun tidak didokumentasikan, tool yang menghasilkan output yang bagus namun tidak dievaluasi dalam konteks operasional, dan automation yang tidak divalidasi dari perspektif yang berbeda — semuanya adalah kerja yang belum tuntas.

Proyek penutup ini menyatukan seluruh output dari enam proyek sebelumnya menjadi dua deliverable: sebuah *automation assessment* yang mengevaluasi kesiapan automation yang telah dibangun secara jujur dan konstruktif, dan *bridge notes* yang secara eksplisit menghubungkan kapabilitas yang telah dikembangkan dengan kebutuhan modul-modul selanjutnya.

## Tujuan

- Mengevaluasi seluruh output automation dari modul ini secara terstruktur dalam format assessment
- Mengidentifikasi apa yang sudah berjalan dengan baik dan apa yang perlu diperbaiki atau dikembangkan lebih lanjut
- Mendokumentasikan prinsip operasional yang harus dibawa ke modul berikutnya
- Menegaskan kesinambungan antara automation yang dibangun di sini dengan kebutuhan *cyber core*, *blue team*, dan *threat hunting*

## Lingkungan Kerja

| Host | Peran dalam Proyek Ini |
|------|------------------------|
| **Ubuntu Lab** | Seluruh penulisan assessment dilakukan di sini |

## Langkah-Langkah yang Dilakukan

### 1. Automation Assessment

```bash
cat > ~/mod4/reports/automation_assessment.md <<'EOF'
# Automation Assessment — Modul 04

## Ringkasan Eksekutif
Python environment siap dan konsisten. Seluruh komponen automation yang ditargetkan
telah dibangun, divalidasi, dan menghasilkan output yang terstruktur. Kali digunakan
secara konsisten sebagai workstation validasi untuk memverifikasi hasil dari perspektif
assessment yang independen.

## Evaluasi per Komponen

### 1. Environment Setup
- Virtual environment aktif dan dikonfirmasi dengan baseline dokumentasi
- Path interpreter dan pip tercatat sebagai referensi
- Seluruh symlink dataset tersedia dan dapat diakses

### 2. Python Foundations
- File I/O menggunakan `pathlib` — portabel dan aman
- Structured output dengan JSON — dapat diproses oleh tool lain
- Argument handling dengan `argparse` — menghilangkan hardcoded values

### 3. Endpoint Automation
- Endpoint checker dengan timeout dan error handling spesifik
- Output tersimpan dalam format yang dapat diproses ulang
- Divalidasi dari Kali — hasil konsisten dengan observasi manual

### 4. Secure Coding
- Input validation mencegah URL dengan skema berbahaya
- Parameterized query diterapkan untuk semua operasi database
- Error messages informatif tanpa membocorkan detail implementasi

### 5. Threat-Aware Scripting
- IOC parser mengekstrak email dan keyword dari phishing sample
- HA256 hash digunakan untuk verifikasi integritas evidence
- ARA rule berhasil mendeteksi keyword phishing pada artefak

### 6. Mini Tool
- Mengintegrasikan seluruh komponen menjadi tool yang kohesif
- Output mencakup timestamp, status, dan header yang relevan
- Divalidasi dari Kali — konsisten dengan curl headers

## Area yang Dapat Dikembangkan Lebih Lanjut

| Area | Pengembangan yang Direkomendasikan |
|------|-----------------------------------|
| IOC parser | Tambahkan ekstraksi IP, URL, dan domain — tidak hanya email |
| YARA rule | Kembangkan rule yang lebih spesifik dengan hex signatures |
| Mini tool | Tambahkan output JSON untuk integrasi dengan tool lain |
| Automation | Jadwalkan dengan cron untuk eksekusi berkala |

## Kesimpulan
Seluruh komponen automation berjalan sesuai ekspektasi dan menghasilkan output yang
dapat divalidasi. Fondasi yang dibangun di modul ini siap dikembangkan menjadi workflow
yang lebih kompleks pada modul-modul selanjutnya.
EOF
cat ~/mod4/reports/automation_assessment.md
```

### 2. Bridge Notes — Jembatan ke Modul Berikutnya

```bash
cat > ~/mod4/reports/module04_bridge_notes.md <<'EOF'
# Module 04 Bridge Notes

## Prinsip Operasional yang Harus Dibawa ke Modul Berikutnya

1. **Script harus menghasilkan output yang tersimpan** — bukan hanya tampilan terminal sesaat
2. **Validasi dari Kali tetap diperlukan** — output Ubuntu tidak cukup tanpa perspektif eksternal
3. **Secure coding adalah kebiasaan, bukan fitur** — diterapkan dari awal, bukan ditambahkan belakangan
4. **Automation harus dapat dijelaskan** — jika tidak bisa menjelaskan cara kerja script, script belum siap pakai
5. **Tool yang sederhana namun terstruktur** lebih bernilai daripada tool yang kompleks namun sulit dipahami

## Koneksi ke Modul Berikutnya

| Komponen Modul 04 | Penggunaan di Modul Berikutnya |
|-------------------|-------------------------------|
| IOC parser dengan regex | Threat hunting — mencari pola IOC dalam log dan traffic |
| YARA rule writing | Blue team analysis — deteksi malware dan artefak berbahaya |
| Endpoint checker | Red team validation — memverifikasi hasil recon terkontrol |
| Secure coding mindset | Secure development — menulis tool yang tidak memperkenalkan kerentanan baru |
| Structured output (JSON) | SIEM integration — output yang dapat diingest oleh platform monitoring |
| Hash verification | Forensic workflow — chain of custody untuk evidence digital |

## Fondasi yang Siap untuk Modul 05, 06, dan 07

- Python digunakan sebagai alat kerja, bukan hanya bahasa yang dipelajari
- Automation mindset terbentuk: input jelas, output terstruktur, error handled
- Secure coding bukan konsep abstrak — sudah dipraktikkan dalam konteks nyata
- Threat-aware thinking sudah terbentuk melalui IOC parsing dan YARA
- Tool building mindset: dari script yang bekerja sekali menuju tool yang dapat dipakai ulang
EOF
cat ~/mod4/reports/module04_bridge_notes.md
```

## Anatomi Assessment yang Efektif

Assessment yang dihasilkan dalam proyek ini mengikuti struktur yang memungkinkan berbagai pembaca mengekstrak nilai yang mereka butuhkan:

| Bagian | Fungsi | Pembaca Utama |
|--------|--------|---------------|
| **Ringkasan Eksekutif** | Status keseluruhan dalam 3-4 kalimat | Semua pembaca |
| **Evaluasi per Komponen** | Detail status tiap area dengan ✅/❌ | Analis teknis, supervisor |
| **Area Pengembangan** | Gap dan rekomendasi yang dapat ditindaklanjuti | Tim pengembangan, mentor |
| **Bridge Notes** | Koneksi eksplisit ke modul berikutnya | Peserta program, instruktur |

**Mengapa evaluasi harus jujur tentang keterbatasan:**

Assessment yang hanya mencatat pencapaian tanpa mengidentifikasi keterbatasan adalah assessment yang tidak berguna untuk perbaikan. Bagian "Area yang Dapat Dikembangkan" bukan pengakuan kegagalan — ini adalah peta jalan untuk iterasi berikutnya, yang merupakan karakteristik dari pengembangan profesional yang matang.

## Temuan Utama

Assessment mengkonfirmasi bahwa seluruh komponen yang ditargetkan telah dibangun dan divalidasi. Lebih penting, pola kerja yang konsisten — membangun di Ubuntu, memvalidasi dari Kali, menyimpan output ke evidence, menutup dengan assessment — telah terbentuk sebagai kebiasaan kerja yang siap dibawa ke modul-modul selanjutnya.

## Apa yang Dipelajari

Kemampuan untuk mengevaluasi hasil kerja sendiri secara objektif — mengakui apa yang berjalan dengan baik dan apa yang perlu dikembangkan — adalah kompetensi yang membedakan praktisi yang terus berkembang dari mereka yang stagnan. Assessment bukan formalitas; ini adalah mekanisme refleksi yang mengubah pengalaman menjadi pembelajaran yang dapat diterapkan.

## Referensi File Bukti

| File | Lokasi | Isi |
|------|--------|-----|
| `automation_assessment.md` | `~/mod4/reports/` | Evaluasi komprehensif seluruh automation yang dibangun |
| `module04_bridge_notes.md` | `~/mod4/reports/` | Prinsip operasional dan koneksi ke modul berikutnya |
