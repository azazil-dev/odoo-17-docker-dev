<div align="center">

# 🐳 Odoo 17 — Docker Compose Template

Template siap pakai untuk menjalankan lingkungan pengembangan **Odoo 17** menggunakan **Docker Compose**,
lengkap dengan **PostgreSQL 15** sebagai database terpisah dan manajemen kredensial aman via **Docker Secrets**.

[![Odoo](https://img.shields.io/badge/Odoo-v17.0-714b67?logo=odoo&logoColor=white&style=flat)](#)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15-336791?logo=postgresql&logoColor=white&style=flat)](#)
[![Docker](https://img.shields.io/badge/Docker_Compose-v2-2496ED?logo=docker&logoColor=white&style=flat)](#)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen?style=flat)](#)
[![Portal](https://img.shields.io/badge/Akses-port_8069-4d4d4d?style=flat)](#)

[![Lisensi](https://img.shields.io/badge/Lisensi-MIT-yellow?style=flat)](#)
[![Versi](https://img.shields.io/badge/Version-v17.0-714b67?style=flat)](#)
[![GitHub](https://img.shields.io/badge/Hosted_on-GitHub-181717?logo=github&logoColor=white&style=flat)](https://github.com/azazil-dev/odoo-17-docker-dev)

</div>

---

## 📖 Daftar Isi

- [Fitur](#-fitur)
- [Prasyarat](#-prasyarat)
- [Struktur Proyek](#-struktur-proyek)
- [Quickstart](#-quickstart)
- [Konfigurasi](#-konfigurasi)
- [Penjelasan docker-compose.yml](#-penjelasan-docker-composeyml)
- [Mengakses Odoo](#-mengakses-odoo)
- [Perintah Harian](#-perintah-harian)
- [Menambahkan Modul (Addons)](#-menambahkan-modul-addons)
- [Troubleshooting](#-troubleshooting)
- [Screenshot](#-screenshot)
- [Referensi](#-referensi)
- [Lisensi](#-lisensi)

---

## ✨ Fitur

- **Odoo 17.0** — Image resmi `odoo:17.0` pada service `web`.
- **PostgreSQL 15** — Image resmi `postgres:15` pada service `postgres-15`, terhubung langsung ke Odoo.
- **Kredensial aman via Docker Secrets** — Password database tidak di-hardcode, melainkan dibaca dari file rahasia (`config/odoo_pg_pass`).
- **Named Volumes persisten** — Data Odoo (`/var/lib/odoo`) dan PostgreSQL (`PGDATA`) tetap tersimpan meski container dihapus.
- **Custom addons** — Folder lokal `./addons` di-mount ke `/mnt/extra-addons`, sehingga modul kustom langsung dikenali tanpa rebuild image.
- **Konfigurasi via volume** — Folder `./config` di-mount ke `/etc/odoo`, tempat Odoo membaca `odoo.conf`.
- **Kontainer terisolasi** — Setiap layanan berjalan di container sendiri dengan `depends_on` untuk urutan startup yang benar.
- **Akses default** — Odoo dapat diakses di `http://localhost:8069` tanpa konfigurasi tambahan.

---

## 🧱 Prasyarat

Sebelum memulai, pastikan sistem Anda sudah memiliki:

| Kebutuhan | Versi Minimal | Cara Verifikasi |
|---|---|---|
| **Docker Engine** | 20.10+ | `docker --version` |
| **Docker Compose** (plugin) | v2.x | `docker compose version` |
| **Docker Compose** (standalone, opsional) | v1.27+ | `docker-compose version` |

> **Catatan:** Compose file masih menyertakan baris `version: '2'`. Pada Docker Compose v2, atribut
> tersebut sudah **obsolete** dan diabaikan (hanya memunculkan warning). Aman dibiarkan, tetapi bisa
> dihapus untuk menghilangkan warning.

---

## 📁 Struktur Proyek

```
odoo-17-development-setup-with-docker/
├── config/
│   ├── odoo.conf        # Konfigurasi utama Odoo (mounted ke /etc/odoo)
│   └── odoo_pg_pass     # 🔒 File secret: password PostgreSQL (tidak di-commit!)
├── addons/              # 📦 Folder modul kustom Odoo (mounted ke /mnt/extra-addons)
├── screenshot/          # 🖼️ Dokumentasi visual hasil setup
├── docker-compose.yml   # ⚙️ Definisi layanan (web + postgres-15)
├── .gitignore           # Aturan file yang dikecualikan dari git
├── LICENSE              # Lisensi MIT
└── README.md            # 📄 Dokumentasi ini
```

> ⚠️ **Penting:** `config/odoo_pg_pass` terdaftar di `.gitignore`, sehingga **tidak ikut** saat
> repository di-clone. Anda harus membuatnya sendiri sebelum menjalankan container
> (lihat [Quickstart](#-quickstart) langkah 2).

---

## 🚀 Quickstart

Ikuti langkah-langkah berikut untuk menjalankan lingkungan development Odoo 17.

### 1. Clone Repository

```bash
git clone https://github.com/azazil-dev/odoo-17-docker-dev.git
cd odoo-17-docker-dev
```

> Cabang `main` sudah berisi kode untuk **Odoo 17**. Tidak perlu checkout branch tambahan.

### 2. Buat File Secret PostgreSQL

File `config/odoo_pg_pass` **tidak ada** setelah clone (lihat `.gitignore`). Buat file tersebut dan isi
dengan password pilihan Anda:

```bash
printf 'ganti_dengan_password_aman' > config/odoo_pg_pass
```

> 💡 Gunakan `printf` (tanpa newline) alih-alih `echo`, karena `echo` menambahkan karakter baris baru
> yang ikut terbaca sebagai bagian dari password.
>
> ⚠️ File ini berisi **rahasia**. Jangan pernah di-commit ke git. Jika pernah ter-commit sebelumnya,
> segera putar ulang password dan hapus dari riwayat git.

### 3. Samakan Konfigurasi Odoo

Buka `config/odoo.conf` dan sesuaikan:

```ini
[options]
admin_passwd = ganti_dengan_password_aman   ; Master password web UI Odoo
db_host = postgres-15
db_port = 5432
db_user = odoo
db_password = ganti_dengan_password_kuat   ; Samakan dengan isi config/odoo_pg_pass
addons_path = /mnt/extra-addons
```

- **`admin_passwd`** — Master password untuk membuat/mengelola database dari web UI.
- **`db_password`** — Nilai ini di-*override* oleh Docker Secret saat runtime, tetapi sebaiknya tetap
  disamakan dengan isi `config/odoo_pg_pass` agar tidak membingungkan.

### 4. Jalankan Container

```bash
docker compose up -d
```

> Gunakan `docker-compose up -d` jika memakai Compose v1 (standalone).
> Mode `-d` (detached) menjalankan container di latar belakang. Tanpa `-d`, log tampil di terminal.

### 5. Verifikasi & Akses

```bash
docker compose ps
```

Pastikan **kedua** service berstatus `Up`:

```
NAME                       IMAGE         STATUS         PORTS
...-postgres-15-1          postgres:15   Up             5432/tcp
...-web-1                  odoo:17.0     Up             0.0.0.0:8069->8069/tcp
```

> Jika container `web` tidak muncul, kemungkinan ia crash dan berhenti. Cek dengan
> `docker compose ps -a` dan `docker compose logs web`.

Buka browser dan akses **http://localhost:8069** 🎉. Pada halaman pertama, buat database baru dengan
mengisi **Master Password** yang telah Anda set pada langkah 3.

---

## ⚙️ Konfigurasi

### 🔒 Kredensial PostgreSQL (Docker Secrets)

Password PostgreSQL disimpan di `config/odoo_pg_pass`. Alur kerjanya:

1. Docker Compose mendaftarkan file tersebut sebagai **secret** bernama `postgresql_password`.
2. Secret ini di-mount ke `/run/secrets/postgresql_password` di **kedua** container (`web` dan `postgres-15`).
3. Service `postgres-15` membaca nilai ini melalui `POSTGRES_PASSWORD_FILE`.
4. Service `web` membaca nilai ini melalui `PASSWORD_FILE` (variabel ini meng-**override** `db_password`
   dari file config).

> ⚠️ **Perhatikan:** PostgreSQL hanya membaca `POSTGRES_PASSWORD_FILE` **sekali saja**, yaitu saat
> volume `db-data-volume` pertama kali di-inisialisasi. Jika Anda mengubah isi `config/odoo_pg_pass`
> **setelah** database dibuat, password di dalam database **tidak** ikut berubah, sehingga Odoo akan
> gagal login (`password authentication failed`). Solusi: samakan kembali passwordnya, atau reset
> volume dengan `docker compose down -v` (⚠️ **semua data hilang**).

**Setiap kali mengubah `config/odoo_pg_pass` pada setup yang masih kosong, restart service:**

```bash
docker compose up -d --force-recreate
```

### 📄 Konfigurasi Odoo (`config/odoo.conf`)

```ini
[options]
admin_passwd = ganti_dengan_password_aman
db_host = postgres-15
db_port = 5432
db_user = odoo
db_password = ganti_dengan_password_aman  ; di-override oleh Docker Secrets saat runtime
addons_path = /mnt/extra-addons
```

| Opsi | Nilai | Keterangan |
|---|---|---|
| `admin_passwd` | *(ubah)* | Master password web UI Odoo |
| `db_host` | `postgres-15` | Nama service database (hostname di dalam jaringan Docker) |
| `db_port` | `5432` | Port default PostgreSQL |
| `db_user` | `odoo` | User database (harus sama dengan `POSTGRES_USER` di compose) |
| `db_password` | *(dari secret)* | Di-override oleh secret `postgresql_password` |
| `addons_path` | `/mnt/extra-addons` | Lokasi modul kustom di dalam container |

> Perubahan pada `odoo.conf` dibaca saat container di-restart:
>
> ```bash
> docker compose restart web
> ```

---

## 🗂️ Penjelasan docker-compose.yml

```yaml
version: '2'                              # obsolete pada Compose v2, boleh dihapus
services:
  web:
    image: odoo:17.0                      # Image resmi Odoo 17
    depends_on:
      - postgres-15                       # Pastikan database jalan duluan
    ports:
      - "8069:8069"                       # Host:Container — akses web via localhost:8069
    volumes:
      - odoo-data-volume:/var/lib/odoo    # Data Odoo persisten
      - ./config:/etc/odoo                # Baca odoo.conf dari host
      - ./addons:/mnt/extra-addons        # Modul kustom dari host
    environment:
      - PASSWORD_FILE=/run/secrets/postgresql_password
    secrets:
      - postgresql_password

  postgres-15:
    image: postgres:15                    # Image resmi PostgreSQL 15
    environment:
      - POSTGRES_DB=postgres
      - POSTGRES_PASSWORD_FILE=/run/secrets/postgresql_password
      - POSTGRES_USER=odoo
      - PGDATA=/var/lib/postgresql/data/pgdata
    volumes:
      - db-data-volume:/var/lib/postgresql/data/pgdata
    secrets:
      - postgresql_password

volumes:
  odoo-data-volume:                       # Volume persisten Odoo
  db-data-volume:                         # Volume persisten PostgreSQL

secrets:
  postgresql_password:
    file: ./config/odoo_pg_pass           # Sumber password (jangan di-commit!)
```

### Ringkasan Layanan

| Layanan | Image | Port | Fungsi |
|---|---|---|---|
| `web` | `odoo:17.0` | `8069 → 8069` | Aplikasi Odoo (UI & API) |
| `postgres-15` | `postgres:15` | *(internal)* | Database PostgreSQL untuk Odoo |

**Mengganti port Odoo (misal mengatasi port bentrok):** ubah bagian `ports` di service `web`.

```yaml
ports:
  - "8070:8069"   # Odoo kini dapat diakses di http://localhost:8070
```

---

## 🖥️ Mengakses Odoo

1. Buka **http://localhost:8069** di browser.
2. Halaman *Database Manager* akan muncul (hanya saat belum ada database).
3. Isi **Master Password** (nilai `admin_passwd` di `config/odoo.conf`).
4. Lengkapi form *Create Database*:
   - **Database Name** — bebas, mis. `odoo17`
   - **Email / Password** — kredensial login admin untuk database tersebut
   - **Language** — mis. *Bahasa Indonesia*
   - **Demo data** — centang untuk menguji dengan data contoh (disarankan untuk development)
5. Klik **Create Database**, lalu Anda masuk ke dashboard Odoo 17.

---

## 🛠️ Perintah Harian

| Aksi | Perintah |
|---|---|
| Jalankan container | `docker compose up -d` |
| Lihat status | `docker compose ps` |
| Lihat status (termasuk yang berhenti) | `docker compose ps -a` |
| Lihat log (semua) | `docker compose logs -f` |
| Lihat log (service tertentu) | `docker compose logs -f web` |
| Restart service | `docker compose restart web` |
| Hentikan container | `docker compose stop` |
| Hentikan & hapus container | `docker compose down` |
| Hapus container + volume (⚠️ data hilang) | `docker compose down -v` |
| Akses shell container | `docker compose exec web bash` |
| Akses psql (PostgreSQL) | `docker compose exec postgres-15 psql -U odoo` |

---

## 📦 Menambahkan Modul (Addons)

1. Salin folder modul kustom Anda ke `./addons/`:

   ```bash
   cp -r /path/ke/modul-kustom ./addons/
   ```

2. **(Manual)** Jalankan *Upgrade* dari UI Odoo:
   - Buka **Aplikasi → Update Apps List** (aktifkan *Developer Mode* bila perlu).
   - Cari nama modul, lalu klik **Install**.

3. **(Otomatis)** Atau install langsung via baris perintah:

   ```bash
   docker compose exec web odoo -i nama_modul -d nama_database --stop-after-init
   ```

> Folder `./addons` ter-mount ke `/mnt/extra-addons`, sehingga modul baru langsung dikenali tanpa rebuild image.

---

## 🔍 Troubleshooting

| Gejala | Kemungkinan Penyebab | Solusi |
|---|---|---|
| **`bind source path does not exist: .../config/odoo_pg_pass`** | File secret belum dibuat (tidak ikut ter-commit karena `.gitignore`) | Buat `config/odoo_pg_pass` (lihat [Quickstart](#-quickstart) langkah 2), lalu `docker compose up -d` |
| **Container `web` hilang / berhenti tak lama setelah start** (`password authentication failed for user "odoo"`) | Isi `config/odoo_pg_pass` berubah **setelah** volume database ter-inisialisasi, sehingga password tersimpan di DB tidak cocok | Samakan password, atau reset total: `docker compose down -v` lalu `up -d` (⚠️ menghapus semua data) |
| **Gagal autentikasi database (`FATAL: password authentication failed`)** | Isi `config/odoo_pg_pass` tidak cocok dengan data volume yang sudah ada | Pastikan file secret berisi password yang benar, lalu `docker compose up -d --force-recreate`. Untuk reset total: `docker compose down -v` lalu `up -d` (⚠️ menghapus semua data) |
| **Container `web` restart terus-menerus (crash loop)** | Konfigurasi salah atau database belum siap | `docker compose logs web` untuk melihat error. Pastikan `postgres-15` berstatus `Up` |
| **Port `8069` sudah dipakai** | Service lain menempati port | Ubah mapping port di `docker-compose.yml` (mis. `8070:8069`), lalu `docker compose up -d` |
| **Modul kustom tidak terdeteksi** | Modul tidak berada di `./addons` atau nama manifest salah | Pastikan folder modul berisi `__manifest__.py` dan berada di `./addons/`, lalu jalankan *Update Apps List* |
| **Perubahan konfigurasi tidak berlaku** | Container belum di-restart | `docker compose restart web` atau `docker compose up -d --force-recreate` |
| **Data hilang setelah `down`** | Volume terhapus | Lakukan `docker compose down` **tanpa** `-v` agar volume persisten tetap tersimpan |

---

## 🖼️ Screenshot

Dokumentasi visual hasil setup lingkungan development Odoo 17:

![Screenshot 1](https://raw.githubusercontent.com/azazil-dev/odoo-17-docker-dev/main/screenshot/1.png)

![Screenshot 2](https://raw.githubusercontent.com/azazil-dev/odoo-17-docker-dev/main/screenshot/2.png)

![Screenshot 3](https://raw.githubusercontent.com/azazil-dev/odoo-17-docker-dev/main/screenshot/3.png)

![Screenshot 4](https://raw.githubusercontent.com/azazil-dev/odoo-17-docker-dev/main/screenshot/4.png)

---

## 📚 Referensi

- [Odoo Documentation](https://www.odoo.com/documentation/17.0/)
- [Odoo Docker Image — Docker Hub](https://hub.docker.com/_/odoo)
- [PostgreSQL — Docker Hub](https://hub.docker.com/_/postgres)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Docker Secrets](https://docs.docker.com/engine/swarm/secrets/)

---

## 📄 Lisensi

Didistribusikan di bawah lisensi **MIT**. Lihat file [`LICENSE`](./LICENSE) untuk detail selengkapnya.
