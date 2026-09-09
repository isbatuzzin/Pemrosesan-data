# MODUL PRAKTIKUM
# Data Ingestion – Batch & API
## Pagination, Rate Limiting, Retry/Backoff, Incremental Loading (Watermark), Clean Code, Logging, Validasi, dan Apache Parquet

> **Basis:** Dikembangkan berdasarkan materi ajar *Data Ingestion – Batch & API* yang diunggah.
>
> **Bentuk:** Praktikum langkah demi langkah, siap diimplementasikan.
>
> **Target:** Mahasiswa yang telah memahami dasar Python, fungsi, perulangan, JSON, CSV, dan HTTP dasar.
>
> **Estimasi:** Dapat digunakan untuk 1–3 sesi praktikum. Untuk kelas 2 SKS, dosen dapat memilih Praktikum 0–10 sebagai praktik inti dan menjadikan proyek mini sebagai tugas lanjutan.

---

# DAFTAR ISI

1. Tujuan Praktikum
2. Peta Kompetensi dan Keterkaitan dengan Materi Ajar
3. Hasil Akhir yang Harus Dicapai
4. Prasyarat
5. Persiapan Lingkungan
6. Struktur Project
7. Praktikum 0 — Menyiapkan Local Mock API
8. Praktikum 1 — Dasar Data Ingestion dan HTTP
9. Praktikum 2 — Pagination
10. Praktikum 3 — Rate Limiting dan HTTP 429
11. Praktikum 4 — Retry, Exponential Backoff, dan Jitter
12. Praktikum 5 — Incremental Loading dan Watermark
13. Praktikum 6 — Clean Code dan Modular Python
14. Praktikum 7 — Logging dan Observability
15. Praktikum 8 — Validasi Data
16. Praktikum 9 — Apache Parquet
17. Praktikum 10 — Pipeline End-to-End
18. Praktikum 11 — Strategi Penyimpanan Batch dan Partitioning
19. Praktik Terbimbing
20. Mini Case Study
21. Pengujian dan Verifikasi
22. Anti-Pattern dan Perbaikan
23. Tugas Mini / Proyek Praktikum
24. Rubrik Penilaian
25. Troubleshooting
26. Checklist Pengumpulan
27. Lampiran Source Code Lengkap

---

# 1. TUJUAN PRAKTIKUM

Setelah menyelesaikan modul ini, mahasiswa mampu membangun pipeline ingestion berbasis API yang:

1. Mengambil data dari REST API menggunakan HTTP `GET`.
2. Memahami struktur request, response, status code, header, dan body.
3. Mengambil seluruh data menggunakan **pagination**.
4. Menghindari masalah memory dengan memproses data per halaman menggunakan **generator**.
5. Menangani **rate limiting** dan HTTP `429 Too Many Requests`.
6. Menerapkan **timeout** pada request.
7. Menerapkan **retry** secara terbatas.
8. Menerapkan **exponential backoff**.
9. Menggunakan **jitter** untuk menghindari retry serempak.
10. Membedakan error sementara dan error yang tidak boleh di-retry secara buta.
11. Menerapkan **incremental loading**.
12. Menyimpan dan menggunakan **watermark** sebagai checkpoint.
13. Memastikan watermark hanya diperbarui setelah data berhasil disimpan.
14. Menulis kode Python yang modular.
15. Menggunakan logging untuk observability.
16. Memvalidasi hasil ingestion.
17. Menyimpan data ke format **Apache Parquet**.
18. Membangun pipeline ingestion end-to-end.
19. Mengorganisasi hasil batch berdasarkan partisi.
20. Mengidentifikasi dan memperbaiki anti-pattern umum.

---

# 2. PETA KOMPETENSI

```text
                        DATA INGESTION
                               │
                               ▼
                         External API
                               │
                               ▼
                           HTTP GET
                               │
              ┌────────────────┼────────────────┐
              │                │                │
              ▼                ▼                ▼
          Pagination       Timeout        Rate Limiting
              │                │                │
              └────────────────┼────────────────┘
                               ▼
                     Retry / Backoff / Jitter
                               │
                               ▼
                     Incremental Loading
                          Watermark
                               │
                               ▼
                        Data Validation
                               │
                               ▼
                    Logging / Observability
                               │
                               ▼
                      Parquet Storage
                               │
                               ▼
                  Reliable Data Pipeline
```

## Keterkaitan dengan materi ajar

| Bagian Materi Ajar | Implementasi dalam Praktikum |
|---|---|
| Mengapa ingestion kompleks | Praktikum 1 dan diskusi masalah |
| Batch API ingestion | Praktikum 0 dan 10 |
| Dasar HTTP | Praktikum 1 |
| Pagination | Praktikum 2 |
| Rate limiting | Praktikum 3 |
| Retry/backoff/jitter | Praktikum 4 |
| Incremental loading/watermark | Praktikum 5 |
| Clean code/modular Python | Praktikum 6 |
| Logging/observability | Praktikum 7 |
| Apache Parquet | Praktikum 9 |
| Pipeline end-to-end | Praktikum 10 |
| Strategi penyimpanan batch | Praktikum 11 |
| Mini case study | Bagian 20 |
| Evaluasi | Bagian 21 |
| Tugas mini | Bagian 23 |
| Rubrik | Bagian 24 |
| Anti-pattern | Bagian 22 |

---

# 3. HASIL AKHIR PRAKTIKUM

Pada akhir praktikum, mahasiswa akan memiliki project seperti berikut:

```text
api_ingestion/
│
├── requirements.txt
├── mock_api.py
│
├── config.py
├── api_client.py
├── pagination.py
├── watermark.py
├── validator.py
├── storage.py
├── main.py
│
├── data/
│   ├── transactions/
│   │   └── transaction_date=YYYY-MM-DD/
│   │       └── transactions.parquet
│   └── ...
│
├── state/
│   └── watermark.json
│
└── logs/
    └── ingestion.log
```

Pipeline akhir:

```text
START
  │
  ▼
Load Configuration
  │
  ▼
Read Watermark
  │
  ▼
Request API
  │
  ├─────────────── Error sementara ───────────────┐
  │                                               │
  ▼                                               │
Response Success? ◄──── Retry / Backoff / Jitter ─┘
  │
  ▼
Pagination
  │
  ▼
Validate Data
  │
  ▼
Write Parquet
  │
  ▼
Verify Output
  │
  ▼
Update Watermark
  │
  ▼
END
```

---

# 4. PRASYARAT

Mahasiswa sebaiknya telah memahami:

- Python dasar.
- Variabel dan tipe data.
- Fungsi.
- `if`, `for`, `while`.
- List dan dictionary.
- File JSON.
- Dasar `pandas`.
- Konsep HTTP dasar.

Software:

- Python 3.10 atau lebih baru.
- VS Code, PyCharm, atau IDE lain.
- Terminal / Command Prompt.
- Internet tidak wajib untuk praktikum inti karena API simulasi dijalankan secara lokal.

---

# 5. PERSIAPAN LINGKUNGAN

## Langkah 1 — Buat folder project

Linux/macOS:

```bash
mkdir api_ingestion
cd api_ingestion
```

Windows PowerShell:

```powershell
mkdir api_ingestion
cd api_ingestion
```

## Langkah 2 — Buat virtual environment

```bash
python -m venv .venv
```

Aktivasi Linux/macOS:

```bash
source .venv/bin/activate
```

Aktivasi Windows PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

Aktivasi Windows CMD:

```cmd
.venv\Scripts\activate
```

## Langkah 3 — Buat `requirements.txt`

```text
requests
pandas
pyarrow
```

Install:

```bash
pip install -r requirements.txt
```

## Langkah 4 — Verifikasi instalasi

```bash
python -c "import requests, pandas, pyarrow; print('Environment siap')"
```

Jika berhasil:

```text
Environment siap
```

---

# 6. STRUKTUR PROJECT AWAL

Buat struktur:

```text
api_ingestion/
│
├── requirements.txt
├── mock_api.py
├── config.py
├── api_client.py
├── pagination.py
├── watermark.py
├── validator.py
├── storage.py
├── main.py
│
├── data/
├── state/
└── logs/
```

Perintah cepat Linux/macOS:

```bash
mkdir -p data state logs
touch mock_api.py config.py api_client.py pagination.py
touch watermark.py validator.py storage.py main.py
```

Pada Windows, file dapat dibuat melalui IDE.

---

# 7. PRAKTIKUM 0 — MENYIAPKAN LOCAL MOCK API

## Tujuan

Sebelum mengakses API publik, kita membuat API lokal agar:

- Praktikum dapat dilakukan tanpa ketergantungan pada API pihak ketiga.
- Pagination dapat dikontrol.
- HTTP 429 dapat disimulasikan.
- Error 500 dapat disimulasikan.
- Watermark dapat diuji.
- Semua mahasiswa memperoleh perilaku API yang konsisten.

## Langkah 1 — Buat `mock_api.py`

Salin kode berikut.

```python
import json
from datetime import datetime, timedelta, timezone
from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib.parse import parse_qs, urlparse


HOST = "127.0.0.1"
PORT = 8000


def create_transactions():
    base_time = datetime(
        2026,
        9,
        1,
        8,
        0,
        0,
        tzinfo=timezone.utc,
    )

    records = []

    for i in range(1, 61):
        updated_at = base_time + timedelta(minutes=i * 10)

        records.append(
            {
                "id": i,
                "customer": f"customer_{i:03d}",
                "amount": i * 10000,
                "updated_at": updated_at.isoformat(),
            }
        )

    return records


TRANSACTIONS = create_transactions()


class DemoHandler(BaseHTTPRequestHandler):

    rate_limit_counter = 0
    unstable_counter = 0

    def send_json(
        self,
        status_code,
        payload,
        headers=None,
    ):
        body = json.dumps(payload).encode("utf-8")

        self.send_response(status_code)
        self.send_header(
            "Content-Type",
            "application/json",
        )
        self.send_header(
            "Content-Length",
            str(len(body)),
        )

        if headers:
            for key, value in headers.items():
                self.send_header(key, str(value))

        self.end_headers()
        self.wfile.write(body)

    def do_GET(self):

        parsed = urlparse(self.path)
        query = parse_qs(parsed.query)

        if parsed.path == "/transactions":
            self.handle_transactions(query)

        elif parsed.path == "/cursor-transactions":
            self.handle_cursor_transactions(query)

        elif parsed.path == "/rate-limited":
            self.handle_rate_limited()

        elif parsed.path == "/unstable":
            self.handle_unstable()

        elif parsed.path == "/reset":
            DemoHandler.rate_limit_counter = 0
            DemoHandler.unstable_counter = 0

            self.send_json(
                200,
                {
                    "message": "counter reset"
                },
            )

        else:
            self.send_json(
                404,
                {
                    "error": "endpoint not found"
                },
            )

    def handle_transactions(self, query):

        page = int(
            query.get("page", ["1"])[0]
        )

        limit = int(
            query.get("limit", ["10"])[0]
        )

        updated_since = query.get(
            "updated_since",
            [None],
        )[0]

        records = TRANSACTIONS.copy()

        if updated_since:
            records = [
                record
                for record in records
                if record["updated_at"] > updated_since
            ]

        start = (page - 1) * limit
        end = start + limit

        page_data = records[start:end]

        next_url = None

        if end < len(records):
            next_page = page + 1

            next_url = (
                f"http://{HOST}:{PORT}/transactions"
                f"?page={next_page}"
                f"&limit={limit}"
            )

            if updated_since:
                next_url += (
                    f"&updated_since={updated_since}"
                )

        payload = {
            "data": page_data,
            "page": page,
            "limit": limit,
            "total": len(records),
            "next": next_url,
        }

        self.send_json(200, payload)

    def handle_cursor_transactions(self, query):

        limit = int(
            query.get("limit", ["10"])[0]
        )

        cursor_value = query.get(
            "cursor",
            [None],
        )[0]

        start = int(cursor_value) if cursor_value else 0
        end = start + limit

        page_data = TRANSACTIONS[start:end]

        next_cursor = (
            str(end)
            if end < len(TRANSACTIONS)
            else None
        )

        payload = {
            "data": page_data,
            "next_cursor": next_cursor,
        }

        self.send_json(200, payload)

    def handle_rate_limited(self):

        DemoHandler.rate_limit_counter += 1

        if DemoHandler.rate_limit_counter <= 2:
            self.send_json(
                429,
                {
                    "error": "Too Many Requests"
                },
                headers={
                    "Retry-After": "1"
                },
            )
            return

        self.send_json(
            200,
            {
                "message": "request accepted"
            },
        )

    def handle_unstable(self):

        DemoHandler.unstable_counter += 1

        if DemoHandler.unstable_counter <= 2:
            self.send_json(
                500,
                {
                    "error": "temporary server error"
                },
            )
            return

        self.send_json(
            200,
            {
                "message": "server recovered"
            },
        )

    def log_message(self, format, *args):
        print(
            f"[MOCK API] "
            f"{self.address_string()} - "
            f"{format % args}"
        )


if __name__ == "__main__":

    server = HTTPServer(
        (HOST, PORT),
        DemoHandler,
    )

    print(
        f"Mock API running at "
        f"http://{HOST}:{PORT}"
    )

    print(
        "Endpoints:"
    )

    print(
        f"  /transactions"
    )

    print(
        f"  /cursor-transactions"
    )

    print(
        f"  /rate-limited"
    )

    print(
        f"  /unstable"
    )

    print(
        f"  /reset"
    )

    server.serve_forever()
```

## Langkah 2 — Jalankan API

Buka terminal pertama:

```bash
python mock_api.py
```

Output:

```text
Mock API running at http://127.0.0.1:8000
```

Terminal ini harus tetap berjalan.

## Langkah 3 — Uji endpoint

Buka terminal kedua:

```bash
python -c "import requests; print(requests.get('http://127.0.0.1:8000/transactions').json())"
```

Atau gunakan browser:

```text
http://127.0.0.1:8000/transactions?page=1&limit=5
```

---

# 8. PRAKTIKUM 1 — DASAR DATA INGESTION DAN HTTP

## Tujuan

Mahasiswa memahami:

- HTTP request.
- HTTP response.
- Status code.
- Headers.
- JSON body.
- Timeout.

## Langkah 1 — Buat `http_basic.py`

```python
import requests


url = (
    "http://127.0.0.1:8000/"
    "transactions?page=1&limit=5"
)

response = requests.get(
    url,
    timeout=(3, 10),
)

print("Status Code:")
print(response.status_code)

print("\nHeaders:")
print(dict(response.headers))

print("\nJSON Body:")
print(response.json())
```

Jalankan:

```bash
python http_basic.py
```

## Langkah 2 — Analisis hasil

Mahasiswa harus mengidentifikasi:

```text
Status Code
    ↓
200
```

Header:

```text
Content-Type
```

Body:

```json
{
  "data": [],
  "page": 1,
  "limit": 5,
  "total": 60,
  "next": "..."
}
```

## Langkah 3 — Tambahkan `raise_for_status()`

```python
import requests


url = (
    "http://127.0.0.1:8000/"
    "transactions?page=1&limit=5"
)

response = requests.get(
    url,
    timeout=(3, 10),
)

response.raise_for_status()

data = response.json()

print(data)
```

## Eksperimen

Ubah endpoint menjadi:

```text
http://127.0.0.1:8000/not-found
```

Apa yang terjadi setelah:

```python
response.raise_for_status()
```

### Pertanyaan

1. Apa fungsi HTTP status code?
2. Mengapa `raise_for_status()` berguna?
3. Apa perbedaan connect timeout dan read timeout?
4. Mengapa ingestion sebaiknya tidak menggunakan request tanpa timeout?

---

# 9. PRAKTIKUM 2 — PAGINATION

# 9.1 Page Number Pagination

## Tujuan

Mengambil seluruh data dari API yang membagi data menjadi beberapa halaman.

API:

```text
GET /transactions?page=1&limit=10
```

## Langkah 1 — Ambil satu halaman

Buat `pagination_basic.py`.

```python
import requests


page = 1
limit = 10

url = (
    "http://127.0.0.1:8000/"
    "transactions"
)

response = requests.get(
    url,
    params={
        "page": page,
        "limit": limit,
    },
    timeout=(3, 10),
)

response.raise_for_status()

payload = response.json()

print(
    "Jumlah record:",
    len(payload["data"]),
)

for record in payload["data"]:
    print(record)
```

## Langkah 2 — Ambil semua halaman

```python
import requests


url = (
    "http://127.0.0.1:8000/"
    "transactions"
)

page = 1
limit = 10

all_data = []

while True:

    response = requests.get(
        url,
        params={
            "page": page,
            "limit": limit,
        },
        timeout=(3, 10),
    )

    response.raise_for_status()

    payload = response.json()

    data = payload.get(
        "data",
        [],
    )

    if not data:
        break

    all_data.extend(data)

    print(
        f"Page {page}: "
        f"{len(data)} record"
    )

    page += 1


print(
    "Total record:",
    len(all_data),
)
```

Jalankan:

```bash
python pagination_basic.py
```

Target:

```text
Total record: 60
```

## Penjelasan

```python
while True:
```

melakukan perulangan sampai:

```python
if not data:
    break
```

Artinya:

> Ketika API tidak lagi mengirim data, pagination berhenti.

---

# 9.2 Kesalahan Umum: Lupa Menaikkan Nomor Halaman

Kode salah:

```python
page = 1

while True:

    response = fetch_page(page)

    data = response["data"]

    if not data:
        break

    # page += 1 tidak ada
```

Akibat:

```text
Page 1
Page 1
Page 1
Page 1
...
```

Ini adalah bentuk **infinite loop**.

## Tugas

Tambahkan batas pengaman:

```python
MAX_PAGES = 1000
```

Kemudian:

```python
if page > MAX_PAGES:
    raise RuntimeError(
        "Maximum page limit exceeded"
    )
```

---

# 9.3 Pagination dengan Generator

## Mengapa?

Pendekatan berikut menyimpan semua data:

```python
all_data = []

all_data.extend(page_data)
```

Untuk data sangat besar, memory dapat meningkat.

Gunakan generator:

```python
import requests


def fetch_paginated_data(
    url,
    limit=10,
):

    page = 1

    while True:

        response = requests.get(
            url,
            params={
                "page": page,
                "limit": limit,
            },
            timeout=(3, 10),
        )

        response.raise_for_status()

        payload = response.json()

        data = payload.get(
            "data",
            [],
        )

        if not data:
            break

        yield data

        page += 1


url = (
    "http://127.0.0.1:8000/"
    "transactions"
)

for page_data in fetch_paginated_data(
    url,
):
    print(
        "Processing:",
        len(page_data),
        "records",
    )
```

## Konsep

```text
Fetch Page 1
    │
    ▼
Process Page 1
    │
    ▼
Fetch Page 2
    │
    ▼
Process Page 2
```

Mahasiswa tidak harus menyimpan seluruh dataset sekaligus.

---

# 9.4 Next Link Pagination

API mock mengembalikan:

```json
{
  "data": [],
  "next": "http://127.0.0.1:8000/transactions?page=2&limit=10"
}
```

Implementasi:

```python
import requests


url = (
    "http://127.0.0.1:8000/"
    "transactions?page=1&limit=10"
)

total = 0

while url:

    response = requests.get(
        url,
        timeout=(3, 10),
    )

    response.raise_for_status()

    payload = response.json()

    data = payload.get(
        "data",
        [],
    )

    total += len(data)

    print(
        "Fetched:",
        len(data),
    )

    url = payload.get("next")


print(
    "Total:",
    total,
)
```

Prinsip:

> Jika API menyediakan `next`, gunakan URL tersebut. Jangan mengasumsikan struktur parameter yang tidak dijamin oleh kontrak API.

---

# 9.5 Cursor Pagination

Endpoint:

```text
/cursor-transactions?limit=10
```

Implementasi:

```python
import requests


url = (
    "http://127.0.0.1:8000/"
    "cursor-transactions"
)

cursor = None
total = 0

while True:

    params = {
        "limit": 10,
    }

    if cursor:
        params["cursor"] = cursor

    response = requests.get(
        url,
        params=params,
        timeout=(3, 10),
    )

    response.raise_for_status()

    payload = response.json()

    data = payload.get(
        "data",
        [],
    )

    total += len(data)

    cursor = payload.get(
        "next_cursor"
    )

    if not cursor:
        break


print(
    "Total:",
    total,
)
```

## Prinsip penting

Cursor adalah **opaque token**.

Artinya:

```text
Client menerima token
        ↓
Client menggunakan token
        ↓
Client tidak perlu mengetahui isi token
```

Jangan membuat asumsi seperti:

```python
cursor = last_id + 10
```

kecuali API secara eksplisit mendefinisikan kontrak tersebut.

---

# 9.6 Tantangan Pagination

Buat fungsi:

```python
def fetch_pages(
    url,
    limit=10,
):
    ...
```

Syarat:

- Menggunakan generator.
- Menggunakan timeout.
- Memanggil `raise_for_status()`.
- Berhenti jika data kosong.
- Memiliki `MAX_PAGES`.

---

# 10. PRAKTIKUM 3 — RATE LIMITING DAN HTTP 429

## Tujuan

Memahami:

```text
HTTP 429 Too Many Requests
```

Endpoint:

```text
http://127.0.0.1:8000/rate-limited
```

Endpoint akan:

```text
Request 1 → 429
Request 2 → 429
Request 3 → 200
```

dan memberikan:

```text
Retry-After: 1
```

## Langkah 1 — Uji tanpa penanganan

```python
import requests


url = (
    "http://127.0.0.1:8000/"
    "rate-limited"
)

response = requests.get(
    url,
    timeout=(3, 10),
)

print(
    "Status:",
    response.status_code,
)

print(
    "Headers:",
    dict(response.headers),
)

print(
    "Body:",
    response.text,
)
```

Jalankan beberapa kali.

---

# 10.1 Pendekatan Kurang Baik

```python
while True:

    response = requests.get(url)

    if response.status_code == 200:
        break
```

Masalah:

- Tidak ada timeout.
- Tidak ada batas retry.
- Dapat melakukan request terlalu cepat.
- Dapat memperparah beban server.
- Dapat menjadi infinite retry.

---

# 10.2 Membaca Header `Retry-After`

Buat:

```python
import requests


url = (
    "http://127.0.0.1:8000/"
    "rate-limited"
)

response = requests.get(
    url,
    timeout=(3, 10),
)

if response.status_code == 429:

    retry_after = response.headers.get(
        "Retry-After"
    )

    print(
        "Retry after:",
        retry_after,
    )
```

---

# 10.3 Penanganan 429

Sebelum menguji ulang, reset API:

```text
http://127.0.0.1:8000/reset
```

Atau:

```python
import requests

requests.get(
    "http://127.0.0.1:8000/reset"
)
```

Kemudian:

```python
import time
import requests


url = (
    "http://127.0.0.1:8000/"
    "rate-limited"
)

MAX_RETRIES = 5


for attempt in range(MAX_RETRIES):

    response = requests.get(
        url,
        timeout=(3, 10),
    )

    if response.status_code == 200:

        print(
            "Success:",
            response.json(),
        )

        break

    if response.status_code == 429:

        retry_after = response.headers.get(
            "Retry-After",
            "1",
        )

        wait_time = float(
            retry_after
        )

        print(
            f"Rate limited. "
            f"Wait {wait_time} seconds"
        )

        time.sleep(wait_time)

        continue

    response.raise_for_status()

else:

    raise RuntimeError(
        "Maximum retries exceeded"
    )
```

## Diskusi

Mengapa:

```python
time.sleep(1)
```

tidak selalu cukup?

Jawaban:

- Setiap API dapat memiliki kebijakan berbeda.
- API dapat menyediakan `Retry-After`.
- Rate limit dapat berbasis detik, menit, quota, user, token, atau algoritme lain.
- Client sebaiknya mengikuti dokumentasi dan sinyal yang diberikan server.

---

# 10.4 Tugas

Modifikasi kode agar:

1. Menampilkan nomor percobaan.
2. Menampilkan status code.
3. Membaca `Retry-After`.
4. Berhenti setelah maksimal 5 percobaan.
5. Menulis log setiap kali terjadi 429.

---

# 11. PRAKTIKUM 4 — RETRY, EXPONENTIAL BACKOFF, DAN JITTER

## Tujuan

Membangun retry yang:

- Terbatas.
- Tidak dilakukan untuk semua error.
- Memberi waktu server untuk pulih.
- Tidak membuat banyak client melakukan retry secara serempak.

---

# 11.1 Simulasi Error Sementara

Endpoint:

```text
http://127.0.0.1:8000/unstable
```

Perilaku:

```text
Request 1 → 500
Request 2 → 500
Request 3 → 200
```

Reset:

```python
import requests

requests.get(
    "http://127.0.0.1:8000/reset"
)
```

---

# 11.2 Retry Manual

```python
import time
import requests


url = (
    "http://127.0.0.1:8000/"
    "unstable"
)

MAX_RETRIES = 5


for attempt in range(
    1,
    MAX_RETRIES + 1,
):

    response = requests.get(
        url,
        timeout=(3, 10),
    )

    print(
        f"Attempt {attempt}: "
        f"status={response.status_code}"
    )

    if response.status_code == 200:

        print(
            "Success:",
            response.json(),
        )

        break

    if response.status_code in (
        500,
        502,
        503,
        504,
    ):

        if attempt == MAX_RETRIES:
            break

        time.sleep(1)

        continue

    response.raise_for_status()

else:

    raise RuntimeError(
        "Request failed"
    )
```

---

# 11.3 Mengapa Retry Tidak Boleh Tanpa Batas?

Kode salah:

```python
while True:

    try:
        response = requests.get(url)
        response.raise_for_status()
        break

    except Exception:
        continue
```

Masalah:

```text
ERROR
  ↓
Retry
  ↓
ERROR
  ↓
Retry
  ↓
ERROR
  ↓
...
```

Akibat:

- Server terus dibebani.
- Program tidak selesai.
- Error permanen tidak ditangani dengan benar.
- Biaya komputasi meningkat.
- Sulit melakukan observasi.

---

# 11.4 Exponential Backoff

Rumus sederhana:

```text
wait = base_delay × 2^(attempt - 1)
```

Jika:

```text
base_delay = 1
```

Maka:

| Attempt | Wait |
|---|---:|
| 1 | 1 detik |
| 2 | 2 detik |
| 3 | 4 detik |
| 4 | 8 detik |

Implementasi:

```python
base_delay = 1

for attempt in range(1, 5):

    wait_time = (
        base_delay
        * (2 ** (attempt - 1))
    )

    print(
        f"Attempt {attempt}: "
        f"wait {wait_time}"
    )
```

---

# 11.5 Retry + Exponential Backoff

```python
import time
import requests


url = (
    "http://127.0.0.1:8000/"
    "unstable"
)

MAX_RETRIES = 5
BASE_DELAY = 1


for attempt in range(
    1,
    MAX_RETRIES + 1,
):

    response = requests.get(
        url,
        timeout=(3, 10),
    )

    if response.status_code == 200:

        print(
            "Success:",
            response.json(),
        )

        break

    if response.status_code in (
        500,
        502,
        503,
        504,
    ):

        if attempt == MAX_RETRIES:
            break

        wait_time = (
            BASE_DELAY
            * (2 ** (attempt - 1))
        )

        print(
            f"Temporary error. "
            f"Wait {wait_time} seconds"
        )

        time.sleep(wait_time)

        continue

    response.raise_for_status()

else:

    raise RuntimeError(
        "Maximum retries exceeded"
    )
```

---

# 11.6 Jitter

## Masalah tanpa jitter

Bayangkan 100 client:

```text
10:00:00 → ERROR
10:00:01 → RETRY bersama
```

Semua melakukan retry bersamaan.

## Jitter

Tambahkan variasi:

```python
import random


wait_time = (
    base_delay
    * (2 ** (attempt - 1))
)

jitter = random.uniform(
    0,
    1,
)

sleep_time = (
    wait_time + jitter
)
```

Contoh:

```text
Client A → 1.12 detik
Client B → 1.74 detik
Client C → 1.35 detik
```

---

# 11.7 Retry + Backoff + Jitter

```python
import random
import time
import requests


RETRYABLE_STATUS_CODES = {
    429,
    500,
    502,
    503,
    504,
}


def request_with_retry(
    url,
    max_retries=5,
    base_delay=1,
):

    for attempt in range(
        1,
        max_retries + 1,
    ):

        try:

            response = requests.get(
                url,
                timeout=(3, 10),
            )

            if response.status_code == 200:

                return response

            if (
                response.status_code
                not in RETRYABLE_STATUS_CODES
            ):
                response.raise_for_status()

            if attempt == max_retries:
                break

            if response.status_code == 429:

                retry_after = response.headers.get(
                    "Retry-After"
                )

                if retry_after:
                    wait_time = float(
                        retry_after
                    )
                else:
                    wait_time = (
                        base_delay
                        * (
                            2
                            ** (attempt - 1)
                        )
                    )

            else:

                wait_time = (
                    base_delay
                    * (
                        2
                        ** (attempt - 1)
                    )
                )

            jitter = random.uniform(
                0,
                1,
            )

            sleep_time = (
                wait_time + jitter
            )

            print(
                f"Attempt {attempt} failed "
                f"with {response.status_code}. "
                f"Wait {sleep_time:.2f}s"
            )

            time.sleep(sleep_time)

        except requests.exceptions.RequestException as error:

            if attempt == max_retries:
                raise

            wait_time = (
                base_delay
                * (
                    2
                    ** (attempt - 1)
                )
            )

            jitter = random.uniform(
                0,
                1,
            )

            sleep_time = (
                wait_time + jitter
            )

            print(
                f"Network error: {error}. "
                f"Wait {sleep_time:.2f}s"
            )

            time.sleep(sleep_time)

    raise RuntimeError(
        "Maximum retries exceeded"
    )
```

Penggunaan:

```python
import requests

requests.get(
    "http://127.0.0.1:8000/reset"
)

response = request_with_retry(
    "http://127.0.0.1:8000/unstable"
)

print(
    response.json()
)
```

---

# 11.8 Error Sementara vs Error yang Perlu Pemeriksaan

Contoh yang dapat dipertimbangkan untuk retry:

```text
Timeout
Connection error
429
500
502
503
504
```

Contoh yang biasanya tidak boleh di-retry secara buta:

```text
400
401
403
404
```

Alasan:

```text
401 → kemungkinan kredensial salah
403 → kemungkinan permission salah
404 → kemungkinan endpoint salah
400 → kemungkinan request salah
```

Retry tidak akan memperbaiki request yang salah.

---

# 11.9 Tugas

Tambahkan:

- `MAX_RETRIES = 3`
- `BASE_DELAY = 0.5`
- Jitter maksimum 0.5 detik.
- Logging untuk setiap retry.
- Penanganan 429 yang memprioritaskan `Retry-After`.

---

# 12. PRAKTIKUM 5 — INCREMENTAL LOADING DAN WATERMARK

## Tujuan

Menghindari:

```text
Full Load
↓
Mengambil seluruh data
↓
Setiap kali program berjalan
```

Kita ingin:

```text
Run 1
↓
Ambil data baru

Run 2
↓
Ambil data setelah checkpoint sebelumnya
```

---

# 12.1 Full Load

```text
API
 │
 ▼
Record 1–1.000.000
 │
 ▼
Pipeline
```

Dilakukan setiap run.

Kelemahan:

- Banyak request.
- Lambat.
- Mahal.
- Menghasilkan pekerjaan berulang.

---

# 12.2 Incremental Loading

Gunakan watermark:

```text
Last successful watermark
2026-09-01T12:00:00+00:00
```

Run berikutnya:

```text
GET /transactions?
updated_since=2026-09-01T12:00:00+00:00
```

---

# 12.3 Buat `watermark.py`

```python
import json
from pathlib import Path


DEFAULT_WATERMARK = (
    "1970-01-01T00:00:00+00:00"
)


def load_watermark(
    file_path,
):

    path = Path(file_path)

    if not path.exists():
        return DEFAULT_WATERMARK

    with path.open(
        "r",
        encoding="utf-8",
    ) as file:

        data = json.load(file)

    return data.get(
        "watermark",
        DEFAULT_WATERMARK,
    )


def save_watermark(
    file_path,
    watermark,
):

    path = Path(file_path)

    path.parent.mkdir(
        parents=True,
        exist_ok=True,
    )

    with path.open(
        "w",
        encoding="utf-8",
    ) as file:

        json.dump(
            {
                "watermark": watermark
            },
            file,
            indent=2,
        )
```

---

# 12.4 Uji Watermark

Buat `test_watermark.py`.

```python
from watermark import (
    load_watermark,
    save_watermark,
)


FILE_PATH = (
    "state/watermark.json"
)


current = load_watermark(
    FILE_PATH
)

print(
    "Current watermark:",
    current,
)

new_value = (
    "2026-09-01T12:00:00+00:00"
)

save_watermark(
    FILE_PATH,
    new_value,
)

print(
    "Watermark saved"
)

print(
    "Reload:",
    load_watermark(FILE_PATH),
)
```

Hasil file:

```json
{
  "watermark": "2026-09-01T12:00:00+00:00"
}
```

---

# 12.5 Menggunakan Watermark pada API

```python
import requests

from watermark import load_watermark


watermark = load_watermark(
    "state/watermark.json"
)

response = requests.get(
    (
        "http://127.0.0.1:8000/"
        "transactions"
    ),
    params={
        "page": 1,
        "limit": 10,
        "updated_since": watermark,
    },
    timeout=(3, 10),
)

response.raise_for_status()

payload = response.json()

print(
    payload["data"]
)
```

---

# 12.6 Siklus Watermark yang Benar

```text
Read Watermark
      │
      ▼
Request API
      │
      ▼
Process Data
      │
      ▼
Validate Data
      │
      ▼
Write Storage
      │
      ▼
Verify Storage
      │
      ▼
Update Watermark
```

---

# 12.7 Pola Salah

```text
Read Watermark
      │
      ▼
Request API
      │
      ▼
Update Watermark   ← TERLALU AWAL
      │
      ▼
Write Storage
      │
      X
    ERROR
```

Masalah:

Watermark sudah maju, tetapi data belum berhasil disimpan.

Run berikutnya dapat:

```text
Melewati data
```

---

# 12.8 Pola Aman

```text
Read Watermark
      │
      ▼
Request API
      │
      ▼
Validate
      │
      ▼
Write Data
      │
      ▼
Verify Data
      │
      ▼
Update Watermark
```

---

# 12.9 Menentukan Watermark Baru

Jika semua record berhasil:

```python
new_watermark = max(
    record["updated_at"]
    for record in records
)
```

Namun perhatikan:

- Timestamp bisa sama.
- Data dapat terlambat datang.
- API dapat memiliki urutan berbeda.
- Operator pembanding `>` dan `>=` dapat memengaruhi duplikasi atau kehilangan data.

Untuk sistem produksi, strategi watermark harus mengikuti kontrak API dan karakteristik sumber data.

---

# 12.10 Tugas

Buat program:

```text
Run 1
↓
Ambil seluruh data
↓
Simpan Parquet
↓
Update watermark

Run 2
↓
Gunakan watermark
↓
Ambil hanya data yang lebih baru
```

Verifikasi dengan menampilkan:

```text
Watermark before:
Watermark after:
Records fetched:
```

---

# 13. PRAKTIKUM 6 — CLEAN CODE DAN MODULAR PYTHON

## Tujuan

Mengubah:

```text
main.py
└── 500 baris
```

menjadi:

```text
api_ingestion/
├── config.py
├── api_client.py
├── pagination.py
├── watermark.py
├── validator.py
├── storage.py
└── main.py
```

---

# 13.1 Single Responsibility

Setiap file memiliki satu tanggung jawab utama.

| File | Tanggung Jawab |
|---|---|
| `config.py` | Konfigurasi |
| `api_client.py` | Komunikasi API |
| `pagination.py` | Iterasi halaman |
| `watermark.py` | State/checkpoint |
| `validator.py` | Validasi |
| `storage.py` | Penyimpanan |
| `main.py` | Orkestrasi pipeline |

---

# 13.2 Buat `config.py`

```python
API_BASE_URL = (
    "http://127.0.0.1:8000"
)

TRANSACTIONS_URL = (
    f"{API_BASE_URL}/transactions"
)

PAGE_SIZE = 10

CONNECT_TIMEOUT = 3
READ_TIMEOUT = 10

MAX_RETRIES = 5
BASE_DELAY = 1

WATERMARK_FILE = (
    "state/watermark.json"
)

DATA_DIRECTORY = "data"

LOG_FILE = (
    "logs/ingestion.log"
)
```

## Hindari Magic Number

Kurang baik:

```python
requests.get(
    url,
    timeout=30,
)
```

Di banyak tempat.

Lebih baik:

```python
REQUEST_TIMEOUT = 30
```

Kemudian:

```python
requests.get(
    url,
    timeout=REQUEST_TIMEOUT,
)
```

---

# 13.3 Buat `api_client.py`

```python
import logging
import random
import time

import requests


LOGGER = logging.getLogger(
    __name__
)


RETRYABLE_STATUS_CODES = {
    429,
    500,
    502,
    503,
    504,
}


def get_json(
    session,
    url,
    params=None,
    timeout=(3, 10),
    max_retries=5,
    base_delay=1,
):

    for attempt in range(
        1,
        max_retries + 1,
    ):

        try:

            response = session.get(
                url,
                params=params,
                timeout=timeout,
            )

            status_code = response.status_code

            LOGGER.info(
                "HTTP request: "
                "attempt=%s status=%s",
                attempt,
                status_code,
            )

            if status_code == 200:
                return response.json()

            if (
                status_code
                not in RETRYABLE_STATUS_CODES
            ):
                response.raise_for_status()

            if attempt == max_retries:
                response.raise_for_status()

            if status_code == 429:

                retry_after = response.headers.get(
                    "Retry-After"
                )

                if retry_after:
                    wait_time = float(
                        retry_after
                    )
                else:
                    wait_time = (
                        base_delay
                        * (
                            2
                            ** (attempt - 1)
                        )
                    )

            else:

                wait_time = (
                    base_delay
                    * (
                        2
                        ** (attempt - 1)
                    )
                )

            jitter = random.uniform(
                0,
                0.5,
            )

            sleep_time = (
                wait_time + jitter
            )

            LOGGER.warning(
                "Retrying request: "
                "attempt=%s status=%s "
                "sleep=%.2f",
                attempt,
                status_code,
                sleep_time,
            )

            time.sleep(sleep_time)

        except (
            requests.exceptions.Timeout,
            requests.exceptions.ConnectionError,
        ) as error:

            if attempt == max_retries:
                raise

            wait_time = (
                base_delay
                * (
                    2
                    ** (attempt - 1)
                )
            )

            jitter = random.uniform(
                0,
                0.5,
            )

            sleep_time = (
                wait_time + jitter
            )

            LOGGER.warning(
                "Network error: %s. "
                "attempt=%s sleep=%.2f",
                error,
                attempt,
                sleep_time,
            )

            time.sleep(sleep_time)

    raise RuntimeError(
        "Maximum retries exceeded"
    )
```

---

# 13.4 Buat `pagination.py`

```python
import logging


LOGGER = logging.getLogger(
    __name__
)


def fetch_pages(
    fetch_function,
    watermark,
    page_size,
    max_pages=10000,
):

    page = 1

    while True:

        if page > max_pages:
            raise RuntimeError(
                "Maximum page limit exceeded"
            )

        payload = fetch_function(
            page=page,
            limit=page_size,
            watermark=watermark,
        )

        data = payload.get(
            "data",
            [],
        )

        LOGGER.info(
            "Fetched page=%s records=%s",
            page,
            len(data),
        )

        if not data:
            break

        yield data

        page += 1
```

---

# 13.5 Prinsip Penamaan

Kurang baik:

```python
x = get()
```

Lebih baik:

```python
response_data = get_transactions()
```

Kurang baik:

```python
a
b
c
```

Lebih baik:

```python
watermark
page_size
retry_count
output_path
```

---

# 13.6 Latihan Refactoring

Berikan mahasiswa kode berikut:

```python
def main():
    # request
    # retry
    # pagination
    # validation
    # save
    # watermark
    # logging
    pass
```

Tugas:

Pisahkan menjadi minimal:

```text
api_client.py
pagination.py
validator.py
storage.py
watermark.py
main.py
```

---

# 14. PRAKTIKUM 7 — LOGGING DAN OBSERVABILITY

## Tujuan

Menggantikan:

```python
print("start")
print("page 1")
print("error")
```

dengan:

```python
logging.info(...)
logging.warning(...)
logging.error(...)
```

---

# 14.1 Buat konfigurasi logging

Buat `logging_setup.py` atau letakkan fungsi berikut di `main.py`.

```python
import logging
from pathlib import Path


def configure_logging(
    log_file,
):

    path = Path(log_file)

    path.parent.mkdir(
        parents=True,
        exist_ok=True,
    )

    logging.basicConfig(
        level=logging.INFO,
        format=(
            "%(asctime)s "
            "%(levelname)s "
            "%(name)s "
            "%(message)s"
        ),
        handlers=[
            logging.FileHandler(
                path,
                encoding="utf-8",
            ),
            logging.StreamHandler(),
        ],
        force=True,
    )
```

---

# 14.2 Level Logging

| Level | Kegunaan |
|---|---|
| DEBUG | Detail diagnosis |
| INFO | Aktivitas normal |
| WARNING | Kondisi tidak ideal |
| ERROR | Kegagalan |
| CRITICAL | Kegagalan serius |

---

# 14.3 Contoh

```python
import logging


logging.info(
    "Pipeline started"
)

logging.info(
    "Fetching page=%s",
    1,
)

logging.warning(
    "Rate limit reached"
)

logging.error(
    "Failed to save data"
)
```

---

# 14.4 Informasi yang Perlu Dicatat

Minimal:

```text
Pipeline started
Watermark loaded
Request page number
Number of records
HTTP status
Retry attempt
Wait time
Output file
Validation result
Watermark updated
Pipeline finished
```

---

# 14.5 Observability

Pertanyaan yang harus dapat dijawab dari log:

```text
Kapan pipeline berjalan?
Berapa halaman yang diproses?
Berapa record yang diambil?
Apakah terjadi retry?
Berapa lama backoff?
File apa yang dihasilkan?
Apakah penyimpanan berhasil?
Watermark berubah menjadi apa?
```

---

# 14.6 Latihan

Tambahkan logging pada:

```python
fetch_pages()
get_json()
save_records()
load_watermark()
main()
```

Kemudian buka:

```text
logs/ingestion.log
```

---

# 15. PRAKTIKUM 8 — VALIDASI DATA

## Tujuan

Jangan langsung menyimpan data tanpa memeriksa struktur.

---

# 15.1 Buat `validator.py`

```python
REQUIRED_FIELDS = {
    "id",
    "customer",
    "amount",
    "updated_at",
}


def validate_record(
    record,
):

    missing_fields = (
        REQUIRED_FIELDS
        - set(record.keys())
    )

    if missing_fields:

        raise ValueError(
            "Missing fields: "
            f"{sorted(missing_fields)}"
        )

    if not isinstance(
        record["id"],
        int,
    ):
        raise ValueError(
            "id must be int"
        )

    if not isinstance(
        record["customer"],
        str,
    ):
        raise ValueError(
            "customer must be str"
        )

    if not isinstance(
        record["amount"],
        (int, float),
    ):
        raise ValueError(
            "amount must be numeric"
        )


def validate_records(
    records,
):

    if not isinstance(
        records,
        list,
    ):
        raise ValueError(
            "records must be a list"
        )

    for record in records:
        validate_record(record)

    return True
```

---

# 15.2 Uji Validasi

```python
from validator import validate_records


records = [
    {
        "id": 1,
        "customer": "customer_001",
        "amount": 10000,
        "updated_at": (
            "2026-09-01T08:10:00+00:00"
        ),
    }
]


validate_records(records)

print(
    "Validation passed"
)
```

---

# 15.3 Uji Error

Hapus field:

```python
"amount"
```

Contoh:

```python
records = [
    {
        "id": 1,
        "customer": "customer_001",
        "updated_at": "2026-09-01",
    }
]
```

Hasil yang diharapkan:

```text
ValueError: Missing fields: ['amount']
```

---

# 15.4 Validasi Duplikasi

Tambahkan fungsi:

```python
def validate_unique_ids(
    records,
):

    ids = [
        record["id"]
        for record in records
    ]

    if len(ids) != len(set(ids)):
        raise ValueError(
            "Duplicate id detected"
        )
```

Gunakan:

```python
validate_unique_ids(
    records
)
```

## Catatan

Untuk dataset besar, strategi deduplikasi harus mempertimbangkan:

- Unique key.
- Window waktu.
- Batch boundaries.
- Upsert/merge.
- Dukungan storage.

---

# 16. PRAKTIKUM 9 — APACHE PARQUET

## Tujuan

Menyimpan hasil ingestion dalam format kolumnar.

---

# 16.1 Mengapa Tidak Selalu CSV?

CSV:

```text
id,customer,amount
1,A,100
2,B,200
```

Keunggulan:

- Sederhana.
- Mudah dibaca.

Namun untuk analitik skala lebih besar, Parquet menawarkan:

- Penyimpanan kolumnar.
- Compression.
- Metadata.
- Integrasi dengan ekosistem analitik.

---

# 16.2 Buat `storage.py`

```python
from pathlib import Path

import pandas as pd


def save_records(
    records,
    output_file,
):

    path = Path(output_file)

    path.parent.mkdir(
        parents=True,
        exist_ok=True,
    )

    df = pd.DataFrame(
        records
    )

    df.to_parquet(
        path,
        index=False,
        compression="snappy",
    )

    return path


def read_records(
    input_file,
):

    return pd.read_parquet(
        input_file
    )
```

---

# 16.3 Menulis Parquet

```python
from storage import save_records


records = [
    {
        "id": 1,
        "customer": "Andi",
        "amount": 10000,
        "updated_at": (
            "2026-09-01T08:00:00+00:00"
        ),
    },
    {
        "id": 2,
        "customer": "Budi",
        "amount": 20000,
        "updated_at": (
            "2026-09-01T09:00:00+00:00"
        ),
    },
]


output_file = (
    "data/transactions.parquet"
)

save_records(
    records,
    output_file,
)

print(
    "Parquet saved"
)
```

---

# 16.4 Membaca Kembali

```python
from storage import read_records


df = read_records(
    "data/transactions.parquet"
)

print(df)
```

---

# 16.5 Validasi Output

```python
import pandas as pd


input_records = [
    {
        "id": 1,
        "customer": "Andi",
        "amount": 10000,
        "updated_at": "2026-09-01",
    }
]


df = pd.DataFrame(
    input_records
)

output_file = (
    "data/validation.parquet"
)

df.to_parquet(
    output_file,
    index=False,
)

result = pd.read_parquet(
    output_file
)

assert len(result) == len(df)

assert list(result.columns) == list(
    df.columns
)

print(
    "Parquet validation passed"
)
```

---

# 16.6 Eksperimen Compression

Coba:

```python
compression="snappy"
```

Kemudian:

```python
compression="gzip"
```

Bandingkan ukuran file jika dataset cukup besar.

> Ketersediaan codec bergantung pada environment dan engine yang digunakan.

---

# 17. PRAKTIKUM 10 — MEMBANGUN PIPELINE END-TO-END

Ini adalah inti praktikum.

Pipeline:

```text
Configuration
      │
      ▼
Logging
      │
      ▼
Read Watermark
      │
      ▼
Fetch API
      │
      ▼
Retry / Backoff / Jitter
      │
      ▼
Pagination
      │
      ▼
Validation
      │
      ▼
Collect Batch
      │
      ▼
Write Parquet
      │
      ▼
Verify Output
      │
      ▼
Update Watermark
```

---

# 17.1 Buat `main.py`

```python
import logging
from datetime import datetime
from pathlib import Path

import pandas as pd
import requests

from api_client import get_json
from config import (
    BASE_DELAY,
    CONNECT_TIMEOUT,
    DATA_DIRECTORY,
    LOG_FILE,
    MAX_RETRIES,
    PAGE_SIZE,
    READ_TIMEOUT,
    TRANSACTIONS_URL,
    WATERMARK_FILE,
)
from pagination import fetch_pages
from storage import (
    read_records,
    save_records,
)
from validator import (
    validate_records,
    validate_unique_ids,
)
from watermark import (
    load_watermark,
    save_watermark,
)


def configure_logging(
    log_file,
):

    path = Path(log_file)

    path.parent.mkdir(
        parents=True,
        exist_ok=True,
    )

    logging.basicConfig(
        level=logging.INFO,
        format=(
            "%(asctime)s "
            "%(levelname)s "
            "%(name)s "
            "%(message)s"
        ),
        handlers=[
            logging.FileHandler(
                path,
                encoding="utf-8",
            ),
            logging.StreamHandler(),
        ],
        force=True,
    )


def fetch_transaction_page(
    session,
    page,
    limit,
    watermark,
):

    params = {
        "page": page,
        "limit": limit,
    }

    if watermark:
        params[
            "updated_since"
        ] = watermark

    return get_json(
        session=session,
        url=TRANSACTIONS_URL,
        params=params,
        timeout=(
            CONNECT_TIMEOUT,
            READ_TIMEOUT,
        ),
        max_retries=MAX_RETRIES,
        base_delay=BASE_DELAY,
    )


def get_max_watermark(
    records,
):

    if not records:
        return None

    return max(
        record["updated_at"]
        for record in records
    )


def build_output_file():

    now = datetime.now()

    partition = (
        f"transaction_date="
        f"{now:%Y-%m-%d}"
    )

    return (
        Path(DATA_DIRECTORY)
        / "transactions"
        / partition
        / "transactions.parquet"
    )


def run_pipeline():

    configure_logging(
        LOG_FILE
    )

    logger = logging.getLogger(
        "pipeline"
    )

    logger.info(
        "Pipeline started"
    )

    watermark = load_watermark(
        WATERMARK_FILE
    )

    logger.info(
        "Watermark loaded: %s",
        watermark,
    )

    session = requests.Session()

    all_records = []

    def fetch_function(
        page,
        limit,
        watermark,
    ):

        return fetch_transaction_page(
            session=session,
            page=page,
            limit=limit,
            watermark=watermark,
        )

    for page_records in fetch_pages(
        fetch_function=fetch_function,
        watermark=watermark,
        page_size=PAGE_SIZE,
    ):

        validate_records(
            page_records
        )

        all_records.extend(
            page_records
        )

        logger.info(
            "Records collected=%s",
            len(all_records),
        )

    if not all_records:

        logger.info(
            "No new records"
        )

        return

    validate_unique_ids(
        all_records
    )

    new_watermark = (
        get_max_watermark(
            all_records
        )
    )

    output_file = (
        build_output_file()
    )

    save_records(
        all_records,
        output_file,
    )

    logger.info(
        "Parquet saved: %s",
        output_file,
    )

    result = read_records(
        output_file
    )

    if len(result) != len(
        all_records
    ):

        raise RuntimeError(
            "Stored record count mismatch"
        )

    if not isinstance(
        result,
        pd.DataFrame,
    ):

        raise RuntimeError(
            "Output is not a DataFrame"
        )

    # PENTING:
    # Update watermark hanya setelah:
    # 1. validasi berhasil
    # 2. data berhasil disimpan
    # 3. output berhasil dibaca kembali

    save_watermark(
        WATERMARK_FILE,
        new_watermark,
    )

    logger.info(
        "Watermark updated: %s",
        new_watermark,
    )

    logger.info(
        "Pipeline finished successfully"
    )


if __name__ == "__main__":

    run_pipeline()
```

---

# 17.2 Perbaiki `config.py`

Pastikan:

```python
API_BASE_URL = (
    "http://127.0.0.1:8000"
)

TRANSACTIONS_URL = (
    f"{API_BASE_URL}/transactions"
)

PAGE_SIZE = 10

CONNECT_TIMEOUT = 3
READ_TIMEOUT = 10

MAX_RETRIES = 5
BASE_DELAY = 1

WATERMARK_FILE = (
    "state/watermark.json"
)

DATA_DIRECTORY = "data"

LOG_FILE = (
    "logs/ingestion.log"
)
```

---

# 17.3 Jalankan Pipeline

Pastikan mock API berjalan:

Terminal 1:

```bash
python mock_api.py
```

Terminal 2:

```bash
python main.py
```

Target:

```text
Pipeline started
Watermark loaded
Fetched page=1 records=10
Fetched page=2 records=10
...
Parquet saved
Watermark updated
Pipeline finished successfully
```

---

# 17.4 Periksa Output

Periksa:

```text
data/
```

Contoh:

```text
data/
└── transactions/
    └── transaction_date=2026-09-09/
        └── transactions.parquet
```

Periksa watermark:

```text
state/watermark.json
```

Periksa log:

```text
logs/ingestion.log
```

---

# 17.5 Uji Run Kedua

Jalankan lagi:

```bash
python main.py
```

Karena watermark sudah berada pada timestamp terakhir, target hasil:

```text
No new records
```

Ini menunjukkan konsep:

```text
Incremental Loading
```

---

# 18. PRAKTIKUM 11 — STRATEGI PENYIMPANAN BATCH DAN PARTITIONING

## Masalah

Jangan selalu:

```text
data/
└── transactions.parquet
```

untuk seluruh data.

Alternatif:

```text
data/
└── transactions/
    ├── transaction_date=2026-09-01/
    │   └── transactions.parquet
    │
    ├── transaction_date=2026-09-02/
    │   └── transactions.parquet
    │
    └── transaction_date=2026-09-03/
        └── transactions.parquet
```

---

# 18.1 Partition Berdasarkan Tanggal

Buat:

```python
from pathlib import Path
from datetime import datetime


def build_partition_path(
    base_directory,
):

    current_date = (
        datetime.now()
        .strftime("%Y-%m-%d")
    )

    return (
        Path(base_directory)
        / "transactions"
        / (
            f"transaction_date="
            f"{current_date}"
        )
        / "transactions.parquet"
    )
```

Penggunaan:

```python
output_file = (
    build_partition_path(
        "data"
    )
)
```

---

# 18.2 Mengapa Partitioning Berguna?

Jika analisis membutuhkan:

```text
Data tanggal 2026-09-01
```

sistem tidak harus selalu membaca seluruh koleksi.

Namun strategi partisi harus mempertimbangkan:

- Query pattern.
- Jumlah file.
- Volume data.
- Ukuran batch.
- Kebutuhan downstream.

---

# 19. PRAKTIK TERBIMBING

# Praktik A — Pagination

## Tugas

Lengkapi:

```python
def fetch_all():

    page = 1

    while True:

        response = fetch_page(
            page
        )

        data = response[
            "data"
        ]

        # TODO:
        # Tentukan kondisi berhenti

        # TODO:
        # Kembalikan data per halaman

        # TODO:
        # Naikkan nomor halaman
```

## Jawaban referensi

```python
def fetch_all():

    page = 1

    while True:

        response = fetch_page(
            page
        )

        data = response.get(
            "data",
            [],
        )

        if not data:
            break

        yield data

        page += 1
```

---

# Praktik B — Retry

## Tugas

Buat fungsi:

```python
def get_with_retry(
    url,
):
    pass
```

Syarat:

- Maksimal 3 retry.
- Timeout.
- Retry untuk 500, 502, 503, 504.
- Exponential backoff.
- Jitter.
- Error non-retryable harus dilempar.

---

# Praktik C — Watermark

## Tugas

Buat:

```python
load_watermark()
```

dan:

```python
save_watermark()
```

Format:

```json
{
  "watermark": "2026-09-01T10:00:00+00:00"
}
```

Syarat:

- Jika file belum ada, gunakan default.
- Folder dibuat otomatis.
- Watermark hanya diperbarui setelah output berhasil.

---

# Praktik D — Parquet

Buat DataFrame:

```python
import pandas as pd


data = [
    {
        "id": 1,
        "name": "Andi",
        "amount": 10000,
    },
    {
        "id": 2,
        "name": "Budi",
        "amount": 20000,
    },
]

df = pd.DataFrame(
    data
)
```

Simpan:

```python
df.to_parquet(
    "data.parquet",
    index=False,
)
```

Baca:

```python
result = pd.read_parquet(
    "data.parquet"
)

print(result)
```

---

# 20. MINI CASE STUDY — TRANSACTION API

## Kasus

Sebuah perusahaan memiliki API:

```text
GET /transactions
```

Response:

```json
{
  "data": [
    {
      "id": 1001,
      "amount": 25000,
      "updated_at": "2026-09-01T10:00:00+00:00"
    }
  ],
  "next": "..."
}
```

Kondisi:

1. API memiliki jutaan transaksi.
2. API menggunakan pagination.
3. API memiliki rate limit.
4. Server kadang mengalami 500.
5. Data dapat berubah.
6. Pipeline dijalankan berkala.
7. Data harus disimpan sebagai Parquet.

---

# 20.1 Pertanyaan Analisis

1. Mengapa tidak cukup menggunakan satu `requests.get()`?
2. Pagination apa yang digunakan?
3. Apa kondisi berhentinya?
4. Error mana yang dapat di-retry?
5. Bagaimana menangani 429?
6. Bagaimana menentukan watermark?
7. Kapan watermark diperbarui?
8. Bagaimana mencegah data duplikat?
9. Bagaimana menyimpan output?
10. Informasi apa yang perlu dicatat pada log?

---

# 20.2 Solusi Konseptual

```text
Read Watermark
      │
      ▼
GET API
      │
      ▼
429? ───── Yes ───► Read Retry-After
      │                    │
      No                   ▼
      │                  Wait
      ▼                    │
500? ───── Yes ───► Retry / Backoff
      │                    │
      No                   └────► GET API
      │
      ▼
Process Page
      │
      ▼
More Page?
  │        │
 Yes       No
  │         │
  └──► GET  ▼
        API Validate
              │
              ▼
         Save Parquet
              │
              ▼
          Verify
              │
              ▼
       Update Watermark
```

---

# 21. PENGUJIAN DAN VERIFIKASI

Praktikum bukan hanya:

```text
Program berjalan
```

Tetapi:

```text
Program benar
```

---

# 21.1 Uji Pagination

Target:

```text
60 records
```

Buat:

```python
def test_pagination():

    total = 0

    for page_data in fetch_paginated_data(
        url=(
            "http://127.0.0.1:8000/"
            "transactions"
        ),
        limit=10,
    ):
        total += len(page_data)

    assert total == 60
```

---

# 21.2 Uji Rate Limiting

1. Reset server.
2. Request endpoint `/rate-limited`.
3. Pastikan terjadi 429.
4. Pastikan client menunggu.
5. Pastikan request berikutnya berhasil.

---

# 21.3 Uji Retry

1. Reset server.
2. Request `/unstable`.
3. Pastikan terjadi 500.
4. Pastikan retry dilakukan.
5. Pastikan akhirnya 200.

---

# 21.4 Uji Watermark

Kasus:

```text
Watermark lama:
2026-09-01T08:00:00+00:00
```

Setelah pipeline:

```text
Watermark baru:
timestamp maksimum data yang berhasil diproses
```

---

# 21.5 Uji Kegagalan Storage

Simulasi:

```python
raise RuntimeError(
    "Storage failed"
)
```

Pastikan:

```text
Watermark TIDAK berubah
```

Ini adalah pengujian penting.

---

# 21.6 Uji Parquet

```python
result = pd.read_parquet(
    output_file
)

assert len(result) == expected_count
```

Tambahkan:

```python
assert "id" in result.columns
assert "updated_at" in result.columns
```

---

# 22. ANTI-PATTERN DAN PERBAIKAN

# 22.1 Infinite Retry

Salah:

```python
while True:
    try:
        request()
    except:
        continue
```

Perbaikan:

```python
for attempt in range(
    MAX_RETRIES
):
    ...
```

---

# 22.2 Tidak Menggunakan Timeout

Salah:

```python
requests.get(url)
```

Perbaikan:

```python
requests.get(
    url,
    timeout=(3, 10),
)
```

---

# 22.3 Retry Semua Error

Salah:

```python
if response.status_code != 200:
    retry()
```

Masalah:

```text
404
↓
Retry
↓
404
↓
Retry
```

Perbaikan:

```python
RETRYABLE = {
    429,
    500,
    502,
    503,
    504,
}
```

Tetap sesuaikan dengan kontrak API.

---

# 22.4 Update Watermark Terlalu Awal

Salah:

```python
save_watermark(
    new_watermark
)

save_data()
```

Perbaikan:

```python
save_data()

verify_data()

save_watermark(
    new_watermark
)
```

---

# 22.5 Satu Fungsi Sangat Besar

Salah:

```text
main()
 ├── request
 ├── retry
 ├── pagination
 ├── validation
 ├── save
 ├── watermark
 └── logging
```

Perbaikan:

```text
main()
 ├── api_client
 ├── pagination
 ├── validator
 ├── storage
 └── watermark
```

---

# 22.6 Tidak Ada Logging

Salah:

```python
print("error")
```

Perbaikan:

```python
logging.error(
    "Failed to save output"
)
```

---

# 22.7 Menyimpan Semua Data di Memory

Salah untuk data sangat besar:

```python
all_data = []

for page in pages:
    all_data.extend(page)
```

Alternatif:

```text
Fetch Page
    ↓
Validate
    ↓
Write Batch
    ↓
Release / continue
```

Implementasi produksi dapat menggunakan:

- Per-page write.
- Chunk.
- Temporary staging.
- Partitioned files.
- Database staging.

---

# 23. TUGAS MINI / PROYEK PRAKTIKUM

## Judul

**Membangun Reliable Batch API Ingestion Pipeline**

## Skenario

Gunakan Local Mock API atau API lain yang sesuai.

Program harus memiliki:

### 1. Timeout

```python
timeout=(3, 10)
```

### 2. Pagination

Harus mengambil seluruh data.

### 3. Retry

Minimal:

```text
3–5 attempts
```

### 4. Exponential Backoff

Contoh:

```text
1
2
4
8
```

### 5. Rate Limit Handling

Menangani:

```text
429 Too Many Requests
```

Memprioritaskan:

```text
Retry-After
```

jika tersedia.

### 6. Watermark

Harus:

- Dibaca sebelum ingestion.
- Disimpan sebagai state.
- Diperbarui setelah output berhasil.

### 7. Logging

Minimal mencatat:

```text
Start
Watermark
Page
Record count
Retry
Error
Output
Finish
```

### 8. Parquet

Output:

```text
*.parquet
```

### 9. Validation

Minimal:

- Required fields.
- Record count.
- Unique ID.

### 10. Clean Code

Minimal:

```text
config.py
api_client.py
pagination.py
watermark.py
validator.py
storage.py
main.py
```

---

# 24. RUBRIK PENILAIAN

| Komponen | Bobot | Indikator |
|---|---:|---|
| HTTP dan API | 10% | Request dan response benar |
| Pagination | 15% | Semua data diambil dan berhenti benar |
| Timeout | 5% | Timeout digunakan |
| Rate limiting | 10% | 429 ditangani |
| Retry/backoff/jitter | 15% | Retry terbatas dan terkontrol |
| Incremental loading | 10% | Watermark digunakan |
| Watermark safety | 10% | Update setelah storage berhasil |
| Clean code | 10% | Modular dan mudah dibaca |
| Logging | 5% | Proses dapat diamati |
| Validation | 5% | Data diperiksa |
| Parquet | 5% | Output benar |

Total:

```text
100%
```

---

# 25. TROUBLESHOOTING

## Error: Connection Refused

Contoh:

```text
Connection refused
```

Periksa:

1. Apakah `mock_api.py` sudah berjalan?
2. Apakah port `8000` benar?
3. Apakah URL benar?

Jalankan:

```bash
python mock_api.py
```

---

## Error: ModuleNotFoundError

Contoh:

```text
ModuleNotFoundError: No module named 'pandas'
```

Install:

```bash
pip install -r requirements.txt
```

Pastikan virtual environment aktif.

---

## Error: Tidak Bisa Menulis Parquet

Periksa:

```bash
pip install pyarrow
```

Kemudian:

```bash
python -c "import pyarrow; print(pyarrow.__version__)"
```

---

## Error: Address Already in Use

Port 8000 sedang digunakan.

Pilihan:

1. Hentikan program lain.
2. Ubah:

```python
PORT = 8000
```

menjadi:

```python
PORT = 8001
```

Kemudian sesuaikan:

```python
API_BASE_URL = (
    "http://127.0.0.1:8001"
)
```

---

## Pipeline Mengambil Data yang Sama

Periksa:

- Watermark.
- Operator pembanding.
- Format timestamp.
- Apakah watermark benar-benar tersimpan.
- Apakah API mendukung filter `updated_since`.

---

## Watermark Tidak Berubah

Periksa:

1. Apakah ada data?
2. Apakah storage berhasil?
3. Apakah program berhenti sebelum `save_watermark()`?
4. Apakah path file benar?

---

# 26. CHECKLIST PENGUMPULAN

## API

- [ ] URL dapat dikonfigurasi.
- [ ] Request menggunakan `requests`.
- [ ] Timeout digunakan.
- [ ] Status error ditangani.

## Pagination

- [ ] Semua halaman diambil.
- [ ] Kondisi berhenti benar.
- [ ] Nomor halaman/cursor diperbarui.
- [ ] Tidak terjadi infinite loop.

## Reliability

- [ ] Retry terbatas.
- [ ] Error sementara dipertimbangkan.
- [ ] Backoff diterapkan.
- [ ] Jitter diterapkan.
- [ ] Network error ditangani.

## Rate Limiting

- [ ] HTTP 429 dikenali.
- [ ] `Retry-After` dibaca jika tersedia.
- [ ] Client tidak melakukan retry agresif.

## Incremental Loading

- [ ] Watermark dibaca.
- [ ] Watermark dikirim ke request.
- [ ] Watermark baru dihitung.
- [ ] Update dilakukan setelah output berhasil.

## Clean Code

- [ ] Modul dipisahkan.
- [ ] Nama fungsi jelas.
- [ ] Konfigurasi tidak tersebar.
- [ ] Tidak banyak magic number.

## Logging

- [ ] Start dicatat.
- [ ] Page dicatat.
- [ ] Retry dicatat.
- [ ] Output dicatat.
- [ ] Finish dicatat.

## Storage

- [ ] Data tersimpan sebagai Parquet.
- [ ] Output dapat dibaca kembali.
- [ ] Jumlah record divalidasi.
- [ ] Kolom divalidasi.

---

# 27. EVALUASI KONSEPTUAL

## Soal 1

Mengapa API besar biasanya menggunakan pagination?

**Jawaban inti:** Karena mengirim seluruh data sekaligus dapat tidak efisien atau tidak praktis. Pagination membagi data menjadi bagian yang lebih kecil.

---

## Soal 2

Apa fungsi HTTP 429?

**Jawaban inti:** Menunjukkan bahwa client telah mengirim terlalu banyak request dalam periode tertentu.

---

## Soal 3

Mengapa retry harus dibatasi?

**Jawaban inti:** Agar program tidak melakukan retry tanpa akhir dan tidak terus membebani sistem.

---

## Soal 4

Apa fungsi exponential backoff?

**Jawaban inti:** Menambah waktu tunggu secara bertahap sebelum retry sehingga sistem memiliki waktu untuk pulih.

---

## Soal 5

Apa fungsi jitter?

**Jawaban inti:** Menambahkan variasi waktu agar banyak client tidak melakukan retry secara bersamaan.

---

## Soal 6

Apa itu watermark?

**Jawaban inti:** Penanda posisi atau status terakhir yang telah berhasil diproses.

---

## Soal 7

Mengapa watermark diperbarui setelah data berhasil disimpan?

**Jawaban inti:** Agar pipeline tidak melewati data yang sebenarnya belum berhasil disimpan.

---

## Soal 8

Mengapa modular code lebih baik?

**Jawaban inti:** Karena tanggung jawab dipisahkan sehingga kode lebih mudah dibaca, diuji, dan dipelihara.

---

## Soal 9

Mengapa Parquet cocok untuk data analitik?

**Jawaban inti:** Karena merupakan format kolumnar dengan dukungan metadata dan kompresi yang cocok untuk banyak beban kerja analitik.

---

# 28. CHECKPOINT DOSEN SELAMA PRAKTIKUM

## Checkpoint 1 — HTTP

Mahasiswa harus dapat menunjukkan:

```text
200
headers
JSON body
```

## Checkpoint 2 — Pagination

Mahasiswa harus menunjukkan:

```text
Total record = 60
```

## Checkpoint 3 — Rate Limit

Mahasiswa harus menunjukkan:

```text
429
↓
Retry-After
↓
Wait
↓
200
```

## Checkpoint 4 — Retry

Mahasiswa harus menunjukkan:

```text
500
↓
Retry
↓
Backoff
↓
200
```

## Checkpoint 5 — Watermark

Mahasiswa harus menunjukkan:

```text
state/watermark.json
```

## Checkpoint 6 — Logging

Mahasiswa harus menunjukkan:

```text
logs/ingestion.log
```

## Checkpoint 7 — Storage

Mahasiswa harus menunjukkan:

```text
transactions.parquet
```

dan:

```python
pd.read_parquet(...)
```

---

# 29. PENGEMBANGAN LANJUTAN

Setelah modul dasar selesai, mahasiswa dapat mengembangkan:

## A. Menggunakan `requests.Session()`

Keuntungan:

- Reuse koneksi.
- Konfigurasi bersama.
- Lebih rapi untuk banyak request.

## B. Menggunakan `urllib3.Retry`

Dapat digunakan melalui adapter `requests`.

Namun mahasiswa tetap perlu memahami retry manual terlebih dahulu.

## C. Unit Testing

Gunakan:

```text
pytest
```

Untuk menguji:

```text
watermark
validator
pagination
retry policy
storage
```

## D. Environment Variable

Jangan menyimpan secret di source code.

Contoh:

```python
import os

API_TOKEN = os.getenv(
    "API_TOKEN"
)
```

## E. Configuration File

Gunakan:

```text
.env
YAML
TOML
JSON
```

sesuai kebutuhan.

## F. Data Partitioning Lebih Lanjut

Contoh:

```text
data/
└── transactions/
    └── year=2026/
        └── month=09/
            └── day=09/
```

---

# 30. MENTAL MODEL UNTUK MAHASISWA

Jangan melihat ingestion sebagai:

```python
requests.get(url)
```

Lihat sebagai sistem:

```text
API
 │
 ▼
Can I connect?
 │
 ▼
Will it respond?
 │
 ▼
Is response successful?
 │
 ▼
Do I need pagination?
 │
 ▼
Am I rate limited?
 │
 ▼
Should I retry?
 │
 ▼
How long should I wait?
 │
 ▼
Where is my last checkpoint?
 │
 ▼
Is data valid?
 │
 ▼
Can I store it safely?
 │
 ▼
Can I verify the output?
 │
 ▼
Can I resume tomorrow?
```

---

# 31. SOURCE CODE RINGKAS — `requirements.txt`

```text
requests
pandas
pyarrow
```

---

# 32. SOURCE CODE RINGKAS — PERINTAH MENJALANKAN

## Terminal 1

```bash
python mock_api.py
```

## Terminal 2

```bash
python main.py
```

## Reset simulator

```python
import requests

requests.get(
    "http://127.0.0.1:8000/reset"
)
```

---

# 33. URUTAN IMPLEMENTASI YANG DIREKOMENDASIKAN

Mahasiswa sebaiknya **tidak langsung menyalin `main.py` lengkap**.

Gunakan urutan berikut:

```text
LANGKAH 1
Environment
    ↓
LANGKAH 2
Mock API
    ↓
LANGKAH 3
HTTP Basic
    ↓
LANGKAH 4
Pagination
    ↓
LANGKAH 5
429
    ↓
LANGKAH 6
Retry
    ↓
LANGKAH 7
Backoff
    ↓
LANGKAH 8
Jitter
    ↓
LANGKAH 9
Watermark
    ↓
LANGKAH 10
Validation
    ↓
LANGKAH 11
Parquet
    ↓
LANGKAH 12
Logging
    ↓
LANGKAH 13
Modularization
    ↓
LANGKAH 14
End-to-End Pipeline
```

---

# 34. DELIVERABLE MAHASISWA

Mahasiswa mengumpulkan:

```text
api_ingestion/
```

berisi:

```text
requirements.txt
mock_api.py

config.py
api_client.py
pagination.py
watermark.py
validator.py
storage.py
main.py

README.md

data/
state/
logs/
```

Selain itu:

1. Screenshot pipeline berhasil.
2. Screenshot log.
3. Screenshot watermark.
4. Bukti file Parquet.
5. Penjelasan singkat desain.
6. Jawaban evaluasi.

---

# 35. FORMAT LAPORAN PRAKTIKUM

## 1. Identitas

```text
Nama:
NIM:
Kelas:
Tanggal:
```

## 2. Tujuan

Jelaskan tujuan praktikum.

## 3. Lingkungan

```text
Python version:
OS:
Library:
```

## 4. Implementasi

Jelaskan:

- HTTP.
- Pagination.
- Retry.
- Rate limiting.
- Watermark.
- Validation.
- Parquet.

## 5. Hasil

Sertakan:

- Jumlah record.
- Jumlah halaman.
- Watermark sebelum.
- Watermark sesudah.
- Output Parquet.

## 6. Analisis

Jawab:

- Apa kesulitan utama?
- Error apa yang ditemukan?
- Bagaimana error ditangani?
- Mengapa watermark penting?

## 7. Kesimpulan

Ringkas pembelajaran utama.

---

# 36. PENUTUP

Setelah menyelesaikan modul ini, mahasiswa seharusnya memahami bahwa:

> **Data ingestion bukan hanya proses memanggil API.**

Pipeline yang baik harus:

```text
Reliable
Scalable
Recoverable
Maintainable
Efficient
Observable
```

Dengan kata lain:

```text
Ambil data dengan benar
        +
Tangani kegagalan
        +
Hormati API
        +
Simpan checkpoint
        +
Validasi data
        +
Simpan dengan aman
        +
Buat proses dapat diamati
```

menjadi fondasi untuk membangun pipeline data yang lebih matang.

---

# 37. REFERENSI UNTUK PENDALAMAN

Modul ini mengikuti topik dan prinsip yang dibahas dalam materi ajar yang diunggah. Untuk pendalaman implementasi, mahasiswa disarankan menggunakan dokumentasi resmi:

- HTTP Semantics dan status code dari RFC Editor.
- Dokumentasi resmi Python `requests`.
- Dokumentasi resmi `urllib3`.
- Dokumentasi resmi Python Logging.
- PEP 8 untuk gaya penulisan Python.
- Dokumentasi Apache Parquet.
- Dokumentasi Apache Arrow / PyArrow.
- Dokumentasi pandas untuk `to_parquet()` dan `read_parquet()`.

> **Catatan untuk dosen:** Saat menggunakan API eksternal pada praktikum lanjutan, selalu sesuaikan implementasi pagination, retry, rate limit, autentikasi, dan incremental loading dengan dokumentasi API yang digunakan. Tidak semua API memiliki kontrak atau perilaku yang sama.

---

# 38. CHECKLIST FINAL

Sebelum praktikum dinyatakan selesai:

- [ ] Mock API berjalan.
- [ ] HTTP request berhasil.
- [ ] Timeout digunakan.
- [ ] Semua halaman berhasil diambil.
- [ ] Infinite loop dicegah.
- [ ] Cursor/next link dipahami.
- [ ] HTTP 429 diuji.
- [ ] `Retry-After` dibaca.
- [ ] Retry dibatasi.
- [ ] Backoff diterapkan.
- [ ] Jitter diterapkan.
- [ ] Watermark disimpan.
- [ ] Watermark tidak diperbarui sebelum storage sukses.
- [ ] Kode dipisahkan ke modul.
- [ ] Logging berjalan.
- [ ] Data divalidasi.
- [ ] Output Parquet berhasil dibuat.
- [ ] Parquet dapat dibaca kembali.
- [ ] Jumlah record diverifikasi.
- [ ] Pipeline end-to-end berhasil dijalankan.

---

**SELESAI — MODUL PRAKTIKUM DATA INGESTION – BATCH & API**
