# MODUL PRAKTIKUM

# PIPELINE AUTOMATION AND ORCHESTRATION DENGAN APACHE AIRFLOW

**Target capaian:** Mahasiswa mampu mengotomatisasi proses pipeline dan
memahami orkestrasi workflow.

**Materi:** (1) Konsep pipeline automation, (2) scheduling dan task
execution, (3) task dependency, (4) workflow orchestration, (5) Apache
Airflow, (6) DAG, task, operator, dan dependency.

  Komponen      Keterangan
  ------------- -----------------------------------------
  Tools utama   Apache Airflow 3.4.0, Docker Compose v2
  Bahasa        Python
  Dataset       CSV transaksi
  Output        CSV valid/reject, SQLite, laporan teks
  Waktu         2--3 pertemuan (6--9 jam)

------------------------------------------------------------------------

## 1. Deskripsi dan Skenario Praktikum

Mahasiswa membangun pipeline transaksi harian: membaca CSV, membersihkan
data, memvalidasi aturan, memuat record valid ke SQLite, dan membuat
laporan. Apache Airflow mengatur urutan task, jadwal, eksekusi, retry,
dan pemantauan.

**Alur workflow**

``` text
Extract CSV → Transform → Validate → Load SQLite → Report
```

### Capaian praktikum

Setelah praktikum, mahasiswa mampu: - Menjelaskan automation dan
orchestration. - Menjelaskan scheduler, task execution, DAG, task,
operator, dan dependency. - Menginstal dan menjalankan Airflow. -
Membuat DAG dan mengatur jadwal. - Menjalankan workflow
manual/terjadwal. - Memeriksa status dan log task. - Membuktikan
kualitas data hasil pipeline.

## 2. Kebutuhan Tools

### Perangkat keras

-   CPU minimal 2 core (disarankan 4 core).
-   RAM minimal 8 GB; disarankan 12--16 GB.
-   Ruang kosong minimal 10 GB.
-   Koneksi internet untuk mengunduh Docker image.

### Perangkat lunak

-   Docker Desktop (Windows/macOS) atau Docker Engine (Linux).
-   Docker Compose v2.
-   Browser.
-   Editor kode (VS Code disarankan).
-   Terminal.

**Sistem operasi:** Windows gunakan Docker Desktop dengan WSL2 dan
jalankan perintah pada terminal Ubuntu/WSL2; macOS gunakan Docker
Desktop; Linux gunakan Docker Engine dan Compose plugin. Airflow
dijalankan di container, sehingga tidak perlu menginstal Airflow
langsung pada Python host.

### Verifikasi Docker

``` bash
docker --version
docker compose version
docker run --rm hello-world
```

Pastikan perintah terakhir menampilkan `Hello from Docker!`.

## 3. Instalasi Apache Airflow dengan Docker Compose

### 3.1 Buat direktori proyek

``` bash
mkdir -p ~/praktikum-airflow
cd ~/praktikum-airflow
mkdir -p dags/data dags/output logs plugins config
```

### 3.2 Unduh Docker Compose resmi Airflow 3.4.0

``` bash
curl -LfO 'https://airflow.apache.org/docs/apache-airflow/3.4.0/docker-compose.yaml'
```

Periksa:

``` bash
ls -l docker-compose.yaml
```

### 3.3 Buat file `.env`

Di Linux/macOS/WSL2:

``` bash
echo "AIRFLOW_UID=$(id -u)" > .env
```

Jika tidak dapat menggunakan UID, buat `.env` berisi:

``` text
AIRFLOW_UID=50000
```

### 3.4 Inisialisasi metadata database

``` bash
docker compose up airflow-init
```

Tunggu sampai proses selesai dengan exit code 0.

### 3.5 Jalankan layanan

``` bash
docker compose up -d
docker compose ps
```

Buka browser ke:

``` text
http://localhost:8080
```

Pada quick-start resmi, kredensial awal biasanya `airflow` / `airflow`.
Gunakan kredensial yang ditampilkan atau dikonfigurasi pada lingkungan
Anda.

**Catatan keamanan:** Docker Compose quick-start ditujukan untuk
pembelajaran dan eksplorasi, bukan deployment produksi. Jangan
mengekspos UI ke internet atau memakai kredensial contoh untuk sistem
nyata.

### 3.6 Perintah operasional

  Kebutuhan                 Perintah
  ------------------------- --------------------------------------------------
  Status container          `docker compose ps`
  Log semua layanan         `docker compose logs -f`
  Log scheduler             `docker compose logs -f airflow-scheduler`
  Stop                      `docker compose stop`
  Start                     `docker compose start`
  Hapus container/network   `docker compose down`
  Hapus juga volume         `docker compose down --volumes --remove-orphans`

Jangan menghapus volume jika masih membutuhkan metadata dan riwayat
Airflow.

## 4. Konsep Dasar

### 4.1 Pipeline automation

Pipeline automation adalah otomatisasi rangkaian pekerjaan data sehingga
proses seperti ekstraksi, transformasi, validasi, dan pemuatan dapat
berjalan tanpa intervensi manual pada setiap tahap.

### 4.2 Automation vs orchestration

  -----------------------------------------------------------------------
  Aspek                   Automation              Orchestration
  ----------------------- ----------------------- -----------------------
  Fokus                   Mengotomatisasi         Mengatur kumpulan
                          pekerjaan               pekerjaan sebagai
                                                  workflow

  Contoh                  Script membersihkan CSV Airflow mengatur
                                                  extract hingga load

  Urutan                  Dapat ditulis di script Dinyatakan melalui
                                                  dependency

  Jadwal                  Cron/pemicu             Scheduler workflow

  Pemantauan              Sering dibuat sendiri   UI dan log Airflow
  -----------------------------------------------------------------------

### 4.3 Scheduling dan task execution

Scheduling menentukan kapan DAG dijalankan. Task execution adalah
pelaksanaan satu unit pekerjaan.

  Cron           Arti
  -------------- --------------------------
  `0 7 * * *`    Setiap hari pukul 07.00
  `0 * * * *`    Setiap jam
  `30 8 * * 1`   Setiap Senin pukul 08.30
  `0 0 1 * *`    Tanggal 1 setiap bulan

Scheduler membuat Dag Run sesuai jadwal; task mulai setelah dependency
dan kapasitas eksekusi terpenuhi. Zona waktu harus ditentukan secara
eksplisit.

### 4.4 Task dependency

Dependency mengatur urutan eksekusi. Contoh:

``` python
extract >> transform >> validate >> load >> report
```

Pola paralel:

``` python
extract >> [clean_customer, clean_product]
[clean_customer, clean_product] >> merge_data
```

### 4.5 Workflow orchestration

Orchestration mengelola urutan task, jadwal, dependency, retry,
kegagalan, dan pemantauan sehingga workflow berjalan terkoordinasi.

## 5. Apache Airflow: DAG, Task, Operator, Dependency

Apache Airflow adalah platform untuk membuat, menjadwalkan, dan memantau
workflow secara programatis.

  Komponen          Fungsi
  ----------------- ----------------------------------------
  DAG               Definisi workflow dan dependency
  Task              Satu unit pekerjaan
  Operator          Menentukan jenis pekerjaan task
  Scheduler         Menentukan kapan task/DAG dijalankan
  DAG Processor     Mem-parsing file DAG
  Executor/Worker   Menjalankan task
  Web UI/API        Mengelola dan memantau workflow
  Metadata DB       Menyimpan status dan metadata eksekusi

Contoh operator: - `BashOperator`: menjalankan perintah shell. -
`PythonOperator`: menjalankan fungsi Python. - `EmptyOperator`: task
kosong untuk struktur workflow. - Operator provider: integrasi layanan
tertentu.

Praktikum utama menggunakan TaskFlow API (`@dag` dan `@task`) agar
fungsi Python mudah disusun menjadi task. Contoh `BashOperator`
diberikan pada bagian 11.

## 6. Menyiapkan Dataset

Buat file `dags/data/transaksi.csv` dengan isi berikut. Data sengaja
memiliki satu ID transaksi kosong dan satu nilai amount negatif untuk
menguji validasi.

``` csv
transaction_id,transaction_date,customer_id,product,quantity,amount,category
TRX001,2026-11-01,C001,Keyboard,2,350000,Peripherals
TRX002,2026-11-01,C002,Mouse,3,150000,Peripherals
TRX003,2026-11-01,C003,Monitor,1,1850000,Electronics
TRX004,2026-11-01,C004,USB Hub,2,240000,Accessories
TRX005,2026-11-01,C005,Webcam,1,625000,Accessories
TRX006,2026-11-01,C006,Headset,2,780000,Audio
TRX007,2026-11-01,C007,SSD,1,950000,Storage
,2026-11-01,C008,Mouse Pad,2,80000,Accessories
TRX009,2026-11-01,C009,Speaker,1,-250000,Audio
TRX010,2026-11-01,C010,Keyboard,1,175000,Peripherals
```

Alternatif membuat file dari terminal:

``` bash
cat > dags/data/transaksi.csv <<'CSV'
transaction_id,transaction_date,customer_id,product,quantity,amount,category
TRX001,2026-11-01,C001,Keyboard,2,350000,Peripherals
TRX002,2026-11-01,C002,Mouse,3,150000,Peripherals
TRX003,2026-11-01,C003,Monitor,1,1850000,Electronics
TRX004,2026-11-01,C004,USB Hub,2,240000,Accessories
TRX005,2026-11-01,C005,Webcam,1,625000,Accessories
TRX006,2026-11-01,C006,Headset,2,780000,Audio
TRX007,2026-11-01,C007,SSD,1,950000,Storage
,2026-11-01,C008,Mouse Pad,2,80000,Accessories
TRX009,2026-11-01,C009,Speaker,1,-250000,Audio
TRX010,2026-11-01,C010,Keyboard,1,175000,Peripherals
CSV
```

Hasil validasi yang diharapkan: 10 record input, 8 valid, 2 ditolak.

## 7. Membangun DAG Pipeline Transaksi

Buat file `dags/pipeline_transaksi.py` dengan kode berikut:

``` python
from __future__ import annotations

import csv
import sqlite3
from pathlib import Path
from datetime import timedelta

import pendulum
from airflow.sdk import dag, task

BASE_DIR = Path("/opt/airflow/dags")
DATA_DIR = BASE_DIR / "data"
OUTPUT_DIR = BASE_DIR / "output"

INPUT_FILE = DATA_DIR / "transaksi.csv"
CLEAN_FILE = OUTPUT_DIR / "transaksi_clean.csv"
VALID_FILE = OUTPUT_DIR / "transaksi_valid.csv"
REJECT_FILE = OUTPUT_DIR / "transaksi_reject.csv"
DB_FILE = OUTPUT_DIR / "warehouse.db"
REPORT_FILE = OUTPUT_DIR / "ringkasan.txt"


@dag(
    dag_id="pipeline_transaksi_harian",
    description="Pipeline otomatis data transaksi CSV",
    schedule="0 7 * * *",
    start_date=pendulum.datetime(2026, 1, 1, tz="Asia/Jakarta"),
    catchup=False,
    default_args={
        "owner": "praktikum",
        "retries": 2,
        "retry_delay": timedelta(minutes=1),
    },
    tags=["praktikum", "pipeline", "transaksi"],
)
def pipeline_transaksi_harian():

    @task
    def extract():
        if not INPUT_FILE.exists():
            raise FileNotFoundError(f"File tidak ditemukan: {INPUT_FILE}")

        OUTPUT_DIR.mkdir(parents=True, exist_ok=True)
        with INPUT_FILE.open("r", newline="", encoding="utf-8") as f:
            rows = list(csv.DictReader(f))

        if not rows:
            raise ValueError("File input tidak memiliki data.")

        print(f"Extract selesai: {len(rows)} record")
        return rows

    @task
    def transform(rows: list[dict]):
        cleaned = []

        for row in rows:
            item = {k: (v or "").strip() for k, v in row.items()}
            item["transaction_id"] = item["transaction_id"].upper()
            item["customer_id"] = item["customer_id"].upper()
            item["category"] = item["category"].title()
            item["quantity"] = int(item["quantity"])
            item["amount"] = float(item["amount"])
            cleaned.append(item)

        with CLEAN_FILE.open("w", newline="", encoding="utf-8") as f:
            writer = csv.DictWriter(f, fieldnames=cleaned[0].keys())
            writer.writeheader()
            writer.writerows(cleaned)

        print(f"Transform selesai: {len(cleaned)} record")
        return cleaned

    @task
    def validate(rows: list[dict]):
        valid, rejected = [], []

        for row in rows:
            reasons = []

            if not row["transaction_id"]:
                reasons.append("transaction_id kosong")
            if not row["customer_id"]:
                reasons.append("customer_id kosong")
            if row["quantity"] <= 0:
                reasons.append("quantity harus lebih dari 0")
            if row["amount"] < 0:
                reasons.append("amount tidak boleh negatif")

            try:
                pendulum.parse(row["transaction_date"])
            except Exception:
                reasons.append("transaction_date tidak valid")

            if reasons:
                rejected.append({**row, "reject_reason": "; ".join(reasons)})
            else:
                valid.append(row)

        if not valid:
            raise ValueError("Tidak ada record valid untuk dimuat.")

        with VALID_FILE.open("w", newline="", encoding="utf-8") as f:
            writer = csv.DictWriter(f, fieldnames=valid[0].keys())
            writer.writeheader()
            writer.writerows(valid)

        reject_fields = (
            list(rejected[0].keys())
            if rejected else list(rows[0].keys()) + ["reject_reason"]
        )
        with REJECT_FILE.open("w", newline="", encoding="utf-8") as f:
            writer = csv.DictWriter(f, fieldnames=reject_fields)
            writer.writeheader()
            writer.writerows(rejected)

        print(f"Valid: {len(valid)}; ditolak: {len(rejected)}")
        return {
            "valid_file": str(VALID_FILE),
            "valid_count": len(valid),
            "rejected_count": len(rejected),
        }

    @task
    def load(validation_result: dict):
        valid_path = Path(validation_result["valid_file"])
        if not valid_path.exists():
            raise FileNotFoundError(valid_path)

        with valid_path.open("r", newline="", encoding="utf-8") as f:
            rows = list(csv.DictReader(f))

        with sqlite3.connect(DB_FILE) as conn:
            conn.execute("""
                CREATE TABLE IF NOT EXISTS transaksi (
                    transaction_id TEXT PRIMARY KEY,
                    transaction_date TEXT NOT NULL,
                    customer_id TEXT NOT NULL,
                    product TEXT NOT NULL,
                    quantity INTEGER NOT NULL,
                    amount REAL NOT NULL,
                    category TEXT NOT NULL
                )
            """)
            conn.executemany("""
                INSERT OR REPLACE INTO transaksi
                (transaction_id, transaction_date, customer_id,
                 product, quantity, amount, category)
                VALUES (?, ?, ?, ?, ?, ?, ?)
            """, [
                (
                    r["transaction_id"], r["transaction_date"],
                    r["customer_id"], r["product"], int(r["quantity"]),
                    float(r["amount"]), r["category"]
                )
                for r in rows
            ])
            conn.commit()

        print(f"Load selesai: {len(rows)} record")
        return len(rows)

    @task
    def report(load_count: int, validation_result: dict):
        with sqlite3.connect(DB_FILE) as conn:
            total_rows = conn.execute(
                "SELECT COUNT(*) FROM transaksi"
            ).fetchone()[0]
            total_amount = conn.execute(
                "SELECT COALESCE(SUM(amount), 0) FROM transaksi"
            ).fetchone()[0]
            categories = conn.execute("""
                SELECT category, COUNT(*), SUM(amount)
                FROM transaksi GROUP BY category ORDER BY category
            """).fetchall()

        lines = [
            "LAPORAN PIPELINE TRANSAKSI",
            "==========================",
            f"Record pada task load: {load_count}",
            f"Record valid batch: {validation_result['valid_count']}",
            f"Record ditolak batch: {validation_result['rejected_count']}",
            f"Total record database: {total_rows}",
            f"Total nilai transaksi: {total_amount:,.2f}",
            "",
            "RINGKASAN PER KATEGORI",
        ]
        for category, count, amount in categories:
            lines.append(f"{category}: {count} transaksi, nilai {amount:,.2f}")

        REPORT_FILE.write_text("\\n".join(lines), encoding="utf-8")
        print("\\n".join(lines))
        return str(REPORT_FILE)

    # Dependency terbentuk melalui aliran output antartask.
    extracted = extract()
    transformed = transform(extracted)
    validation_result = validate(transformed)
    loaded = load(validation_result)
    report(loaded, validation_result)


pipeline_transaksi_harian()
```

### Penjelasan kode

  Kode/parameter    Fungsi
  ----------------- -----------------------------------------------------
  `@dag`            Mendefinisikan workflow
  `dag_id`          Nama unik DAG
  `schedule`        Jadwal eksekusi
  `start_date`      Awal jadwal
  `catchup=False`   Tidak membuat seluruh run historis yang terlewat
  `@task`           Membuat task Python
  `retries=2`       Maksimal dua percobaan ulang setelah percobaan awal
  `retry_delay`     Jeda antarpercobaan
  `extract`         Membaca data sumber
  `transform`       Membersihkan dan menormalisasi data
  `validate`        Memisahkan data valid dan ditolak
  `load`            Menulis data valid ke SQLite
  `report`          Menghasilkan laporan

**Catatan penting:** TaskFlow API mengirim output antartask menggunakan
XCom. Contoh ini mengirim list/dictionary kecil agar mudah dipelajari.
Untuk dataset besar, simpan data pada file/object storage/database dan
kirimkan hanya path atau identifier melalui XCom. Direktori `dags` pada
Docker Compose di-mount ke container Airflow; pastikan worker yang
mengeksekusi task dapat mengakses output tersebut.

## 8. Menjalankan dan Memantau DAG

1.  Simpan `pipeline_transaksi.py` di folder `dags`.
2.  Pastikan file CSV ada di `dags/data`.
3.  Tunggu Airflow memindai DAG.
4.  Buka `http://localhost:8080`.
5.  Cari `pipeline_transaksi_harian`.
6.  Unpause/aktifkan DAG.
7.  Klik **Trigger DAG** untuk menjalankan secara manual.
8.  Buka Grid View atau Graph View.
9.  Tunggu seluruh task selesai.
10. Klik task untuk melihat log.

Status yang diharapkan:

  Task        Status
  ----------- ---------
  extract     Success
  transform   Success
  validate    Success
  load        Success
  report      Success

Pada log `validate`, hasil yang diharapkan:

``` text
Valid: 8; ditolak: 2
```

## 9. Memeriksa Output

Jalankan dari direktori proyek:

``` bash
ls -l dags/output
cat dags/output/transaksi_valid.csv
cat dags/output/transaksi_reject.csv
cat dags/output/ringkasan.txt
```

File yang dihasilkan: - `transaksi_clean.csv` - `transaksi_valid.csv` -
`transaksi_reject.csv` - `warehouse.db` - `ringkasan.txt`

Periksa database dengan Python (tidak memerlukan SQLite CLI):

``` bash
python3 - <<'PY'
import sqlite3
conn = sqlite3.connect("dags/output/warehouse.db")
print("Jumlah:", conn.execute("SELECT COUNT(*) FROM transaksi").fetchone())
for row in conn.execute("""
    SELECT category, COUNT(*), SUM(amount)
    FROM transaksi GROUP BY category
"""):
    print(row)
conn.close()
PY
```

Jumlah record database yang diharapkan setelah satu run adalah 8.

## 10. Praktik Scheduling

Pada DAG utama:

``` python
schedule="0 7 * * *"
start_date=pendulum.datetime(2026, 1, 1, tz="Asia/Jakarta")
catchup=False
```

Artinya DAG dijadwalkan setiap hari pukul 07.00 zona waktu Jakarta.
Untuk eksperimen, ubah sementara:

``` python
schedule="*/5 * * * *"
```

Artinya setiap lima menit. Simpan file, tunggu DAG diproses ulang, lalu
amati Dag Run pada UI. Setelah eksperimen, kembalikan jadwal ke
`0 7 * * *`.

**Manual trigger** dimulai oleh pengguna. **Scheduled run** dibuat
berdasarkan jadwal oleh scheduler. Jadwal tidak selalu berarti task
mulai tepat pada detik jadwal; scheduler tetap memproses run dan
dependency.

## 11. Praktik Operator dengan BashOperator

Buat file `dags/dag_operator_demo.py`:

``` python
import pendulum
from airflow.sdk import DAG
from airflow.providers.standard.operators.bash import BashOperator
from airflow.providers.standard.operators.empty import EmptyOperator

with DAG(
    dag_id="dag_operator_demo",
    description="Contoh operator dan dependency",
    start_date=pendulum.datetime(2026, 1, 1, tz="Asia/Jakarta"),
    schedule=None,
    catchup=False,
    tags=["praktikum", "operator"],
) as dag:

    start = EmptyOperator(task_id="start")

    tampilkan_pesan = BashOperator(
        task_id="tampilkan_pesan",
        bash_command="echo 'Praktikum Apache Airflow dimulai'",
    )

    cek_waktu = BashOperator(
        task_id="cek_waktu",
        bash_command="date",
    )

    selesai = EmptyOperator(task_id="selesai")

    start >> tampilkan_pesan >> cek_waktu >> selesai
```

Langkah pengujian: 1. Simpan file dan tunggu DAG muncul. 2. Jalankan
`dag_operator_demo` secara manual. 3. Buka Graph View. 4. Periksa log
task `tampilkan_pesan` dan `cek_waktu`.

Analisis: - Apa fungsi `BashOperator`? - Apa fungsi `EmptyOperator`? -
Mengapa `cek_waktu` menunggu `tampilkan_pesan`? - Apa yang berubah jika
dependency diubah?

## 12. Praktik Retry dan Simulasi Kegagalan

DAG utama memiliki `retries=2` dan jeda satu menit.

Simulasi file tidak ditemukan:

1.  Pause DAG agar tidak ada run terjadwal baru.
2.  Pindahkan file sumber:

``` bash
mv dags/data/transaksi.csv dags/data/transaksi_backup.csv
```

3.  Trigger DAG secara manual.
4.  Amati task `extract`.
5.  Buka log dan temukan `FileNotFoundError`.
6.  Amati status retry.
7.  Kembalikan file:

``` bash
mv dags/data/transaksi_backup.csv dags/data/transaksi.csv
```

8.  Jika seluruh retry sudah habis, trigger Dag Run baru.

Pertanyaan: - Task mana yang gagal? - Apa pesan error? - Mengapa retry
tidak selalu menyelesaikan masalah? - Kapan sumber masalah harus
diperbaiki terlebih dahulu?

## 13. Praktik Dependency Paralel

Jika dua pekerjaan tidak saling bergantung, keduanya dapat dijalankan
paralel.

``` text
                  +--> hitung_total_transaksi --+
extract_and_clean |                              |--> buat_laporan
                  +--> hitung_per_kategori ------+
```

Contoh pola TaskFlow:

``` python
cleaned = extract_and_clean()
total = hitung_total_transaksi(cleaned)
kategori = hitung_per_kategori(cleaned)
buat_laporan(total, kategori)
```

Task `hitung_total_transaksi` dan `hitung_per_kategori` dapat berjalan
setelah task ekstraksi/pembersihan selesai. `buat_laporan` menunggu
keduanya.

**Tugas pengembangan:** Buat DAG yang membaca data, membersihkan data,
lalu menjalankan dua analisis secara paralel (total penjualan dan
penjualan per kategori), kemudian menggabungkan hasil menjadi laporan.

## 14. Troubleshooting

  -----------------------------------------------------------------------------
  Masalah                       Kemungkinan penyebab    Tindakan
  ----------------------------- ----------------------- -----------------------
  `docker: command not found`   Docker belum            Instal Docker dan buka
                                terpasang/PATH          terminal baru

  Docker daemon tidak aktif     Engine belum berjalan   Jalankan Docker
                                                        Desktop/Engine

  UI tidak dapat diakses        Container belum         Periksa
                                sehat/port bentrok      `docker compose ps` dan
                                                        log

  DAG tidak muncul              Lokasi salah atau       Pastikan file ada di
                                parsing error           `dags`, periksa log

  Import error                  Sintaks/package tidak   Periksa log DAG
                                sesuai                  processor dan provider

  File tidak ditemukan          Path atau mount salah   Periksa `dags/data` dan
                                                        lokasi file

  Permission denied             Izin direktori/UID      Periksa `.env` dan izin
                                                        folder

  Container restart             RAM tidak cukup         Tambah alokasi RAM dan
                                                        periksa log

  Task tertahan                 Dependency belum        Periksa Graph/Grid dan
                                terpenuhi/worker        status worker
  -----------------------------------------------------------------------------

Perintah pemeriksaan:

``` bash
docker compose ps
docker compose logs airflow-scheduler
docker compose logs airflow-dag-processor
docker compose logs airflow-api-server
docker compose exec airflow-scheduler ls -l /opt/airflow/dags
```

## 15. Tugas Praktikum Individu

Bangun workflow **Pipeline Pengolahan Data Penjualan** dengan ketentuan:

1.  Minimal lima task.
2.  Gunakan TaskFlow API atau minimal satu operator.
3.  Terapkan dependency yang benar.
4.  Atur jadwal DAG.
5.  Terapkan validasi data.
6.  Terapkan retry pada task yang sesuai.
7.  Hasilkan laporan atau tabel database.
8.  Jalankan secara manual.
9.  Buktikan hasil melalui UI dan log.

### Pertanyaan analisis

1.  Apa perbedaan pipeline automation dan orchestration?
2.  Mengapa dependency diperlukan?
3.  Apa perbedaan DAG, task, dan operator?
4.  Apa fungsi scheduler?
5.  Apa perbedaan manual trigger dan scheduled run?
6.  Apa yang terjadi jika task prasyarat gagal?
7.  Kapan retry tepat digunakan?
8.  Mengapa XCom tidak cocok untuk dataset besar?
9.  Apa manfaat Graph View?
10. Mengapa quick-start Compose bukan konfigurasi produksi?

### Bukti yang dikumpulkan

-   Source code DAG utama dan DAG operator demo.
-   Dataset CSV.
-   Screenshot daftar DAG.
-   Screenshot Graph View.
-   Screenshot Grid View/status task.
-   Screenshot log task `validate`.
-   File valid dan reject.
-   Database SQLite atau hasil query.
-   File laporan.
-   Jawaban analisis.

### Rubrik penilaian

  Aspek                            Bobot
  --------------------------- ----------
  Instalasi dan konfigurasi          15%
  Pemahaman konsep                   10%
  Desain DAG                         20%
  Dependency                         15%
  Implementasi pipeline              20%
  Pengujian dan monitoring           10%
  Dokumentasi dan analisis           10%
  **Total**                     **100%**

## 16. Checklist Ketercapaian

-   [ ] Docker dan Compose berjalan.
-   [ ] Airflow berhasil diinisialisasi.
-   [ ] Web UI dapat diakses.
-   [ ] Dataset tersedia di direktori yang benar.
-   [ ] DAG berhasil dimuat.
-   [ ] DAG dapat dijalankan manual.
-   [ ] Scheduling dikonfigurasi dan dijelaskan.
-   [ ] Dependency terlihat di Graph View.
-   [ ] Task utama berhasil dieksekusi.
-   [ ] Data valid dan reject terpisah.
-   [ ] Data valid dimuat ke SQLite.
-   [ ] Log dan status task dapat dibaca.
-   [ ] Mahasiswa mampu menjelaskan DAG, task, operator, scheduler, dan
    dependency.

## 17. Menghentikan dan Membersihkan Lingkungan

Stop tanpa menghapus data:

``` bash
docker compose stop
```

Jalankan kembali:

``` bash
docker compose start
```

Hapus container/network:

``` bash
docker compose down
```

Hapus juga volume metadata Airflow:

``` bash
docker compose down --volumes --remove-orphans
```

Perintah terakhir menghapus volume yang dikelola Compose. File proyek
pada direktori host seperti `dags/data` dan `dags/output` tetap ada
kecuali dihapus secara terpisah.

## 18. Kesimpulan

Mahasiswa membangun pipeline yang dapat dijalankan otomatis dengan
Apache Airflow. Workflow didefinisikan sebagai DAG, pekerjaan dibagi
menjadi task, urutan diatur melalui dependency, jadwal ditentukan
melalui scheduler, dan hasil dipantau melalui UI serta log.

**Target akhir:** Mahasiswa mampu mengotomatisasi proses pipeline dan
memahami orkestrasi workflow menggunakan Apache Airflow.

## Referensi Resmi

1.  Apache Airflow Documentation: https://airflow.apache.org/docs/
2.  Running Airflow in Docker 3.4.0:
    https://airflow.apache.org/docs/apache-airflow/3.4.0/howto/docker-compose.html
3.  Core Concepts:
    https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/index.html
4.  DAGs:
    https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/dags.html
5.  TaskFlow API:
    https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/taskflow.html
6.  Docker Compose: https://docs.docker.com/compose/

**Catatan versi:** Modul mematok Airflow 3.4.0 sebagai versi contoh.
Sebelum praktikum dilaksanakan, periksa dokumentasi resmi untuk
memastikan versi Airflow, Python, provider, dan Docker Compose tetap
kompatibel.
