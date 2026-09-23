# MODUL PRAKTIKUM

# Automated Data Contracts & Assertions

## Data Cleaning & Transformation

------------------------------------------------------------------------

## A. Identitas Praktikum

  -----------------------------------------------------------------------
  Komponen                            Keterangan
  ----------------------------------- -----------------------------------
  Pertemuan                           4

  Topik                               Automated Data Contracts &
                                      Assertions

  Sub-CPMK                            Mahasiswa mampu merancang dan
                                      menerapkan kontrak data
                                      terotomatisasi untuk mencegah
                                      masuknya data cacat (*fail-fast*)
                                      ke dalam arsitektur data

  Tools utama                         Python, Pydantic, Great
                                      Expectations

  Dataset                             Data transaksi/order

  Pendekatan                          Data Contract → Validation →
                                      Quality Gate → Fail-Fast → Data
                                      Warehouse

  Bentuk praktikum                    Hands-on / individual atau kelompok
                                      kecil
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# B. Deskripsi Praktikum

Pada praktikum ini mahasiswa menerjemahkan konsep **Data Contract**,
**Data Quality as Code**, **Assertions**, **Schema Validation**, dan
**Fail-Fast** menjadi implementasi Python.

Skenario yang digunakan adalah sebuah **Order Service** sebagai *data
producer* yang menghasilkan data transaksi. Data tersebut harus memenuhi
kontrak sebelum diteruskan ke tahap downstream yang disimulasikan
sebagai **Data Warehouse**.

Alur utama praktikum:

``` text
Data Producer
      |
      v
Incoming Batch
      |
      v
Data Contract
      |
      +--------------------+
      |                    |
      v                    v
   Pydantic          Great Expectations
      |                    |
      +---------+----------+
                |
                v
         Automated Validation
                |
          +-----+-----+
          |           |
        PASS         FAIL
          |           |
          v           v
   Data Warehouse    Reject
```

Praktikum tidak hanya meminta mahasiswa menghasilkan output `PASS` atau
`FAIL`, tetapi juga menjelaskan **mengapa data diterima/ditolak**,
aturan apa yang dilanggar, dan bagaimana mekanisme tersebut mencegah
data cacat masuk ke tahap downstream.

------------------------------------------------------------------------

# C. Tujuan Praktikum

Setelah menyelesaikan praktikum, mahasiswa diharapkan mampu:

1.  menyiapkan environment Python untuk data validation;
2.  membuat virtual environment;
3.  menginstal dan memverifikasi Pydantic;
4.  menginstal dan memverifikasi Great Expectations;
5.  membuat dataset valid dan invalid;
6.  merancang data contract;
7.  mengimplementasikan contract menggunakan Pydantic;
8.  membuat validation rule dan assertion;
9.  menerapkan strict schema validation;
10. membuat custom business rule;
11. menangani `ValidationError`;
12. melakukan validasi terhadap batch data;
13. menerapkan mekanisme **fail-fast**;
14. membuat Expectation menggunakan Great Expectations;
15. membuat Expectation Suite;
16. menjalankan Validation Definition;
17. menganalisis Validation Result;
18. membuat quality gate sebelum data diteruskan;
19. mensimulasikan Data Warehouse sebagai target data yang hanya
    menerima batch valid;
20. memahami perbedaan peran Pydantic dan Great Expectations.

------------------------------------------------------------------------


# D. Tools yang Digunakan

## 1. Python

Digunakan sebagai bahasa implementasi seluruh eksperimen.

## 2. Pydantic

Digunakan untuk:

-   membuat model data;
-   mendefinisikan schema;
-   melakukan validasi tipe;
-   membuat constraint;
-   menghasilkan validation error;
-   menerapkan business rule pada model.

Praktikum menggunakan pendekatan Pydantic v2.

## 3. Great Expectations

Digunakan untuk:

-   mendefinisikan Expectations;
-   membuat Expectation Suite;
-   menghubungkan data dengan Data Source/Data Asset/Batch;
-   menjalankan validasi;
-   menghasilkan Validation Result.

Dokumentasi GX saat penyusunan modul ini menggunakan GX Core 1.23.1 dan
Python 3.10--3.13. API GX berubah antarversi, sehingga praktikum ini
menggunakan API GX Core 1.x, bukan API tutorial GX 0.17 lama.

## 4. Text Editor / IDE

Disarankan menggunakan salah satu:

-   Visual Studio Code;
-   PyCharm;
-   Jupyter Notebook;
-   editor Python lain.

------------------------------------------------------------------------

# E. Persiapan Environment

## E.1. Memeriksa Python

Buka terminal.

### macOS/Linux

``` bash
python3 --version
```

Jika `python3` tidak tersedia, coba:

``` bash
python --version
```

### Windows

``` powershell
py --version
```

Target praktikum:

``` text
Python 3.10 – 3.13
```

Untuk GX Core versi yang digunakan dalam modul ini, dokumentasi resmi GX
mensyaratkan Python 3.10--3.13.

------------------------------------------------------------------------

# E.2. Membuat Folder Project

Buat folder:

``` text
praktikum-data-contract/
```

Kemudian masuk ke folder tersebut:

``` bash
cd praktikum-data-contract
```

Struktur awal:

``` text
praktikum-data-contract/
```

------------------------------------------------------------------------

# E.3. Membuat Virtual Environment

### macOS/Linux

``` bash
python3 -m venv .venv
```

Aktivasi:

``` bash
source .venv/bin/activate
```

### Windows PowerShell

``` powershell
py -m venv .venv
```

Aktivasi:

``` powershell
.venv\Scripts\Activate.ps1
```

Jika berhasil, terminal biasanya menampilkan:

``` text
(.venv)
```

Virtual environment direkomendasikan agar dependency praktikum
terisolasi dari project Python lain.

------------------------------------------------------------------------

# E.4. Upgrade pip

``` bash
python -m pip install --upgrade pip
```

Periksa:

``` bash
python -m pip --version
```

------------------------------------------------------------------------

# F. Instalasi Tools

## F.1. Instal Pydantic

``` bash
python -m pip install "pydantic>=2,<3"
```

Verifikasi:

``` bash
python -c "import pydantic; print(pydantic.__version__)"
```

Output seharusnya menunjukkan versi Pydantic 2.x.

------------------------------------------------------------------------

# F.2. Instal Pandas

Pandas digunakan untuk membaca dan mengelola dataset CSV.

``` bash
python -m pip install pandas
```

Verifikasi:

``` bash
python -c "import pandas; print(pandas.__version__)"
```

------------------------------------------------------------------------

# F.3. Instal Great Expectations

``` bash
python -m pip install great_expectations
```

Verifikasi:

``` bash
python -c "import great_expectations as gx; print(gx.__version__)"
```

Pastikan import berhasil.

------------------------------------------------------------------------

# F.4. Membuat requirements.txt

Buat file:

``` text
requirements.txt
```

Isi:

``` text
pydantic>=2,<3
pandas
great_expectations
```

Instalasi berikutnya cukup dengan:

``` bash
python -m pip install -r requirements.txt
```

------------------------------------------------------------------------

# G. Struktur Project

Buat struktur:

``` text
praktikum-data-contract/
│
├── .venv/
│
├── data/
│   ├── orders_valid.csv
│   ├── orders_invalid.csv
│   ├── orders_schema_broken.csv
│   └── warehouse_orders.csv
│
├── src/
│   ├── contract.py
│   ├── validate_pydantic.py
│   ├── validate_batch.py
│   └── validate_gx.py
│
├── requirements.txt
│
└── README.md
```

Folder `data/`:

``` bash
mkdir data
```

Folder `src/`:

``` bash
mkdir src
```

Pada Windows jika `mkdir` tidak bekerja sesuai shell, buat folder
tersebut melalui file explorer.

------------------------------------------------------------------------

# H. Studi Kasus

## H.1. Skenario

Sebuah sistem **Order Service** menghasilkan data transaksi.

Producer menghasilkan data:

``` text
order_id
customer_id
amount
status
transaction_date
payment_date
```

Consumer utama adalah:

``` text
Data Warehouse
```

Contract yang disepakati:

  Field              Tipe            Aturan
  ------------------ --------------- ------------------------
  order_id           string          wajib
  customer_id        integer         \> 0
  amount             float           \>= 0
  status             string          PAID/PENDING/CANCELLED
  transaction_date   datetime        wajib
  payment_date       datetime/null   wajib jika PAID

------------------------------------------------------------------------

# I. Percobaan 1 --- Membuat Dataset

## I.1. Membuat Dataset Valid

Buat:

``` text
data/orders_valid.csv
```

Isi:

``` csv
order_id,customer_id,amount,status,transaction_date,payment_date
ORD-001,101,250000.0,PAID,2026-09-20T08:00:00,2026-09-20T08:15:00
ORD-002,102,125000.0,PENDING,2026-09-20T08:05:00,
ORD-003,103,75000.0,CANCELLED,2026-09-20T08:10:00,
ORD-004,104,500000.0,PAID,2026-09-20T08:20:00,2026-09-20T08:30:00
```

------------------------------------------------------------------------

## I.2. Membuat Dataset Invalid

Buat:

``` text
data/orders_invalid.csv
```

Isi:

``` csv
order_id,customer_id,amount,status,transaction_date,payment_date
ORD-101,201,250000.0,PAID,2026-09-20T09:00:00,2026-09-20T09:15:00
ORD-102,-202,125000.0,PENDING,2026-09-20T09:05:00,
ORD-103,203,-75000.0,PAID,2026-09-20T09:10:00,
ORD-104,204,50000.0,UNKNOWN,2026-09-20T09:15:00,
ORD-105,205,100000.0,PAID,2026-09-20T09:20:00,
```

Identifikasi masalah:

``` text
ORD-102 → customer_id negatif
ORD-103 → amount negatif
ORD-104 → status tidak dikenal
ORD-105 → PAID tetapi payment_date kosong
```

------------------------------------------------------------------------

# J. Percobaan 2 --- Memahami Data Contract

Sebelum menulis code, tuliskan contract dalam bentuk tabel.

  Field              Required   Type          Constraint
  ------------------ ---------- ------------- ------------------------
  order_id           Ya         string        tidak kosong
  customer_id        Ya         integer       \> 0
  amount             Ya         float         \>= 0
  status             Ya         enum/string   PAID/PENDING/CANCELLED
  transaction_date   Ya         datetime      valid datetime
  payment_date       Tidak      datetime      wajib jika status PAID

Kemudian jawab:

1.  Field mana yang merupakan identifier?
2.  Field mana yang harus memiliki nilai?
3.  Field mana yang memiliki range?
4.  Field mana yang memiliki allowed values?
5.  Aturan mana yang merupakan business rule?
6.  Apa yang harus dilakukan ketika satu record gagal?
7.  Apa yang harus dilakukan ketika satu batch gagal?

------------------------------------------------------------------------

# K. Percobaan 3 --- Membuat Data Contract dengan Pydantic

Buat:

``` text
src/contract.py
```

Isi:

``` python
from datetime import datetime
from typing import Literal

from pydantic import BaseModel, ConfigDict, Field, model_validator


class OrderContract(BaseModel):
    model_config = ConfigDict(
        strict=True,
        extra="forbid",
    )

    order_id: str = Field(min_length=1)
    customer_id: int = Field(gt=0)
    amount: float = Field(ge=0)
    status: Literal["PAID", "PENDING", "CANCELLED"]
    transaction_date: datetime
    payment_date: datetime | None = None

    @model_validator(mode="after")
    def paid_requires_payment_date(self):
        if self.status == "PAID" and self.payment_date is None:
            raise ValueError(
                "payment_date wajib diisi jika status=PAID"
            )
        return self
```

------------------------------------------------------------------------

# L. Penjelasan Contract

## L.1. Strict Mode

``` python
model_config = ConfigDict(strict=True)
```

Strict mode digunakan agar mahasiswa dapat melihat perbedaan antara:

``` text
tipe benar
```

dan:

``` text
nilai yang mungkin dapat dikonversi otomatis
```

Dalam data contract yang ketat, coercion yang tidak diinginkan dapat
ditolak.

------------------------------------------------------------------------

## L.2. Extra Fields

``` python
extra="forbid"
```

Artinya field yang tidak didefinisikan dalam contract ditolak.

Misalnya:

``` json
{
  "order_id": "ORD-001",
  "customer_id": 101,
  "amount": 250000,
  "status": "PAID",
  "transaction_date": "...",
  "payment_date": "...",
  "hacker_field": "unexpected"
}
```

Field `hacker_field` tidak ada dalam contract sehingga data ditolak.

------------------------------------------------------------------------

## L.3. Constraint

``` python
customer_id: int = Field(gt=0)
```

Artinya:

``` text
customer_id > 0
```

Sedangkan:

``` python
amount: float = Field(ge=0)
```

berarti:

``` text
amount >= 0
```

------------------------------------------------------------------------

## L.4. Allowed Values

``` python
status: Literal["PAID", "PENDING", "CANCELLED"]
```

Maka:

``` text
PAID       → valid
PENDING    → valid
CANCELLED  → valid
UNKNOWN    → invalid
```

------------------------------------------------------------------------

## L.5. Business Rule

``` python
@model_validator(mode="after")
def paid_requires_payment_date(self):
```

Aturan:

``` text
IF status == PAID
THEN payment_date wajib tersedia
```

Ini berbeda dari schema sederhana karena aturan bergantung pada hubungan
antarfield.

------------------------------------------------------------------------

# M. Percobaan 4 --- Validasi Satu Record

Buat:

``` text
src/validate_pydantic.py
```

Isi:

``` python
from datetime import datetime

from pydantic import ValidationError

from contract import OrderContract


valid_order = {
    "order_id": "ORD-001",
    "customer_id": 101,
    "amount": 250000.0,
    "status": "PAID",
    "transaction_date": datetime(2026, 9, 20, 8, 0),
    "payment_date": datetime(2026, 9, 20, 8, 15),
}


try:
    order = OrderContract.model_validate(valid_order)

    print("VALID")
    print(order)

except ValidationError as exc:
    print("INVALID")
    print(exc)
```

Jalankan:

``` bash
cd src
python validate_pydantic.py
```

Jika berhasil:

``` text
VALID
```

------------------------------------------------------------------------

# N. Percobaan 5 --- Menguji Data Invalid

Ubah:

``` python
"customer_id": 101
```

menjadi:

``` python
"customer_id": -101
```

Jalankan kembali.

Hasil yang diharapkan:

``` text
INVALID
```

Amati informasi error.

------------------------------------------------------------------------

# O. Percobaan 6 --- Menguji Business Rule

Gunakan:

``` python
invalid_order = {
    "order_id": "ORD-999",
    "customer_id": 999,
    "amount": 100000.0,
    "status": "PAID",
    "transaction_date": datetime(2026, 9, 20, 10, 0),
    "payment_date": None,
}
```

Validasi:

``` python
try:
    OrderContract.model_validate(invalid_order)

    print("VALID")

except ValidationError as exc:
    print("INVALID")
    print(exc)
```

Hasil:

``` text
INVALID
```

Alasan:

``` text
status = PAID
payment_date = None
```

Melanggar:

``` text
PAID → payment_date wajib tersedia
```

------------------------------------------------------------------------

# P. Percobaan 7 --- Menguji Extra Field

Tambahkan:

``` python
"unexpected_field": "ABC"
```

Contoh:

``` python
invalid_order = {
    "order_id": "ORD-888",
    "customer_id": 888,
    "amount": 100000.0,
    "status": "PENDING",
    "transaction_date": datetime(2026, 9, 20, 10, 0),
    "payment_date": None,
    "unexpected_field": "ABC",
}
```

Hasil:

``` text
INVALID
```

Diskusikan:

> Mengapa `extra="forbid"` dapat dianggap sebagai bagian dari data
> contract?

------------------------------------------------------------------------

# Q. Percobaan 8 --- Validasi Batch

Sekarang validasi tidak lagi satu record, tetapi seluruh batch.

Buat:

``` text
src/validate_batch.py
```

Isi:

``` python
import csv
from datetime import datetime

from pydantic import ValidationError

from contract import OrderContract


def parse_datetime(value):
    if value is None or value == "":
        return None

    return datetime.fromisoformat(value)


def load_orders(path):
    with open(path, newline="", encoding="utf-8") as file:
        reader = csv.DictReader(file)

        rows = []

        for row in reader:
            rows.append(
                {
                    "order_id": row["order_id"],
                    "customer_id": int(row["customer_id"]),
                    "amount": float(row["amount"]),
                    "status": row["status"],
                    "transaction_date": parse_datetime(
                        row["transaction_date"]
                    ),
                    "payment_date": parse_datetime(
                        row["payment_date"]
                    ),
                }
            )

        return rows


def validate_batch(rows):
    errors = []
    valid_orders = []

    for row_number, row in enumerate(rows, start=2):
        try:
            order = OrderContract.model_validate(row)
            valid_orders.append(order)

        except ValidationError as exc:
            errors.append(
                {
                    "row": row_number,
                    "order_id": row.get("order_id"),
                    "errors": exc.errors(),
                }
            )

    return valid_orders, errors


rows = load_orders("../data/orders_valid.csv")

valid_orders, errors = validate_batch(rows)

print(f"Jumlah record : {len(rows)}")
print(f"Valid         : {len(valid_orders)}")
print(f"Invalid       : {len(errors)}")
```

Jalankan dari folder `src`:

``` bash
python validate_batch.py
```

------------------------------------------------------------------------

# R. Percobaan 9 --- Menguji Batch Invalid

Ubah:

``` python
rows = load_orders("../data/orders_valid.csv")
```

menjadi:

``` python
rows = load_orders("../data/orders_invalid.csv")
```

Jalankan:

``` bash
python validate_batch.py
```

Amati:

``` text
Jumlah record
Valid
Invalid
```

Kemudian tampilkan detail error:

``` python
for error in errors:
    print(error)
```

------------------------------------------------------------------------

# S. Percobaan 10 --- Menerapkan Fail-Fast

Konsep yang akan diterapkan:

``` text
Batch
  |
  v
Validate seluruh batch
  |
  +---- semua valid ----> ACCEPT
  |
  +---- ada invalid ----> REJECT
```

Tambahkan:

``` python
if errors:
    print("FAIL-FAST: BATCH DITOLAK")

    for error in errors:
        print(error)

    raise SystemExit(1)

print("BATCH VALID")
print("Batch dapat diteruskan ke Data Warehouse.")
```

------------------------------------------------------------------------

# T. Percobaan 11 --- Simulasi Data Warehouse

Kita akan membuat Data Warehouse sederhana dengan file:

``` text
data/warehouse_orders.csv
```

Aturan:

> File warehouse hanya boleh diisi apabila seluruh batch lolos validasi.

Tambahkan fungsi:

``` python
def save_to_warehouse(orders, output_path):
    with open(
        output_path,
        "w",
        newline="",
        encoding="utf-8",
    ) as file:

        writer = csv.writer(file)

        writer.writerow(
            [
                "order_id",
                "customer_id",
                "amount",
                "status",
                "transaction_date",
                "payment_date",
            ]
        )

        for order in orders:
            writer.writerow(
                [
                    order.order_id,
                    order.customer_id,
                    order.amount,
                    order.status,
                    order.transaction_date.isoformat(),
                    (
                        order.payment_date.isoformat()
                        if order.payment_date
                        else ""
                    ),
                ]
            )
```

Kemudian:

``` python
if errors:
    print("FAIL-FAST: batch ditolak.")
    raise SystemExit(1)

save_to_warehouse(
    valid_orders,
    "../data/warehouse_orders.csv",
)

print("Batch berhasil dimuat ke Data Warehouse.")
```

------------------------------------------------------------------------

# U. Percobaan 12 --- Membuktikan Fail-Fast

## U.1. Gunakan Data Valid

Gunakan:

``` text
orders_valid.csv
```

Jalankan pipeline.

Hasil:

``` text
BATCH VALID
Batch berhasil dimuat ke Data Warehouse.
```

Periksa:

``` text
data/warehouse_orders.csv
```

------------------------------------------------------------------------

## U.2. Gunakan Data Invalid

Gunakan:

``` text
orders_invalid.csv
```

Jalankan pipeline.

Hasil:

``` text
FAIL-FAST: batch ditolak.
```

Periksa:

``` text
data/warehouse_orders.csv
```

Pastikan batch invalid **tidak ditulis sebagai batch baru**.

------------------------------------------------------------------------

