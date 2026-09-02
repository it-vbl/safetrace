# SafeTrace

SafeTrace adalah platform Sistem Informasi Geografis (Geographic Information System/GIS) sumber terbuka dan full-stack untuk ketertelusuran rantai pasok perkebunan. Platform ini terdiri atas dua repositori independen—backend REST API berbasis Django dan dashboard frontend berbasis Next.js—yang secara bersama-sama memungkinkan organisasi mengelola data petani, bidang kebun, dokumen STDB (Surat Tanda Daftar Budidaya), kepatuhan GAP (Good Agricultural Practice), dan layer peta interaktif.

Proyek ini mencakup implementasi studi kasus untuk koperasi petani di Kabupaten Sekadau, Kalimantan Barat, Indonesia.

Sistem ini dikembangkan dengan dukungan pendanaan dari Uni Eropa, Kementerian Federal Jerman untuk Kerja Sama Ekonomi dan Pembangunan (BMZ), serta Kementerian Luar Negeri Belanda sebagai bagian dari Proyek SAFE yang dilaksanakan oleh GIZ. Isi dokumen ini sepenuhnya menjadi tanggung jawab PT VBL dan tidak serta-merta mencerminkan pandangan GIZ, Uni Eropa, Kementerian Federal Jerman untuk Kerja Sama Ekonomi dan Pembangunan (BMZ), maupun Kementerian Luar Negeri Belanda.

Repositori ini menyediakan satu referensi terpadu untuk menyiapkan dan menggunakan kedua aplikasi.

## Repositori

- [Backend SafeTrace](https://github.com/it-vbl/safetrace-backend) - REST API Django, pemrosesan geospasial, penyimpanan data, dan logika bisnis.
- [Frontend SafeTrace](https://github.com/it-vbl/safetrace-frontend) - Dashboard Next.js, peta, tabel, dan alur kerja berbasis peran.

## Gambaran Umum Proyek

SafeTrace memberikan koperasi pertanian, instansi pemerintah, dan mitra pengolahan pandangan bersama yang transparan mengenai asal produk pertanian dan cara produk tersebut dihasilkan. Backend menyediakan REST API yang menyimpan dan memproses data geospasial serta kepatuhan, sedangkan frontend menampilkan data tersebut dalam dashboard interaktif yang dilengkapi peta, tabel, dan alur kerja berbasis peran.

## Backend (Django)

Backend menangani seluruh penyimpanan data, pemrosesan geospasial, dan logika bisnis untuk sistem ketertelusuran, termasuk peringatan deforestasi, pengelolaan data spasial, dan sinkronisasi berkala dengan sumber data eksternal seperti Google Earth Engine (GEE).

### Dependensi Backend

- Python 3.12
- Library GDAL untuk pemrosesan shapefile
- Poppler untuk `pdf2image`
- PostgreSQL dengan ekstensi PostGIS
- Redis sebagai backend antrean tugas
- Ubuntu 24.04 sebagai sistem operasi referensi

### Dependensi Sistem (Ubuntu 24.04)

```bash
sudo apt update
sudo apt install -y python3-dev python3.12-dev
sudo apt install -y gdal-bin libgdal-dev
sudo apt install -y poppler-utils
sudo apt install -y postgresql postgresql-contrib postgis
sudo apt install -y redis-server
```

### Penyiapan Backend

1. Clone repositori:

   ```bash
   git clone https://github.com/it-vbl/safetrace-backend.git
   cd safetrace-backend
   ```

2. Buat dan aktifkan virtual environment:

   ```bash
   python3.12 -m venv venv
   source venv/bin/activate
   ```

3. Instal Python binding GDAL yang sesuai dengan library sistem yang terpasang:

   ```bash
   gdal-config --version
   pip install gdal==$(gdal-config --version)
   ```

4. Instal dependensi Python lainnya:

   ```bash
   pip install -r requirements.txt
   ```

5. Impor data dasar wilayah administratif Indonesia. Karena proyek menggunakan `django-wilayah-indonesia`, data referensi administratif untuk provinsi, kabupaten/kota, kecamatan, dan desa/kelurahan harus diimpor sebelum penggunaan pertama:

   ```bash
   ./manage.py import_base_csv
   ```

6. Jalankan migrasi basis data:

   ```bash
   python manage.py migrate
   ```

7. Jalankan development server:

   ```bash
   python manage.py runserver
   ```

8. Buka [http://localhost:8000](http://localhost:8000) pada peramban.

### Lingkungan Produksi

Untuk menjalankan backend menggunakan pengaturan produksi, ekspor environment variable berikut sebelum memulai aplikasi:

```bash
export DJANGO_ENV=prod
```

Perintah tersebut menginstruksikan Django untuk memuat konfigurasi dari `settings/prod.py`, bukan pengaturan development default. Pastikan file `.env` produksi yang valid tersedia di direktori utama proyek.

### Penyiapan Produksi

1. Clone proyek ke server. Lokasi yang direkomendasikan adalah `/var/www/html`:

   ```bash
   cd /var/www/html
   sudo git clone https://github.com/it-vbl/safetrace-backend.git safetrace-backend
   cd safetrace-backend
   ```

2. Pastikan file `.env` produksi tersedia di direktori utama proyek.

3. Jalankan skrip deployment awal:

   ```bash
   sudo bash scripts/deploy_init.sh
   ```

4. Pastikan seluruh layanan berhasil dijalankan:

   ```bash
   sudo systemctl status safetrace
   sudo systemctl status nginx
   ```

5. Konfigurasikan cron job untuk sinkronisasi data deforestasi Google Earth Engine:

   ```bash
   sudo crontab -e
   ```

   Tambahkan job seperti berikut untuk menjalankan sinkronisasi setiap hari pukul 02.00:

   ```cron
   0 2 * * * cd /var/www/html/safetrace-backend && DJANGO_ENV=prod /var/www/html/safetrace-backend/venv/bin/python manage.py get_data_gee_deforestation >> /var/log/safetrace_gee_deforestation.log 2>&1
   ```

6. Untuk rilis dan pembaruan berikutnya, jalankan:

   ```bash
   sudo bash scripts/deploy_update.sh
   ```

### Pemecahan Masalah Backend

#### Ketidaksesuaian versi GDAL

Jika muncul pesan kesalahan seperti berikut:

```text
Python bindings of GDAL X.X.X require at least libgdal X.X.X
```

Python binding tidak sesuai dengan library sistem GDAL yang terpasang. Periksa versi yang terpasang dan instal binding yang sesuai:

```bash
gdal-config --version
pip install gdal==$(gdal-config --version)
```

Server Ubuntu 24.04 umumnya menggunakan GDAL 3.8.4 atau versi yang lebih baru.

#### Header Python tidak tersedia

Jika muncul pesan `Python.h: No such file or directory`, instal header development Python:

```bash
sudo apt install python3-dev python3.12-dev
```

#### Skrip deployment

Helper deployment secara otomatis menginstal versi GDAL yang sesuai dengan versi library sistem yang terdeteksi:

```bash
# Deployment awal
sudo bash scripts/deploy_init.sh

# Memperbarui deployment yang sudah ada
sudo bash scripts/deploy_update.sh
```

## Frontend (Next.js)

Frontend merupakan dashboard GIS sumber terbuka yang menggunakan REST API backend. Frontend dapat dihubungkan dengan backend apa pun yang kompatibel untuk mengelola data petani, data perkebunan, dokumen STDB, catatan kepatuhan GAP, dan layer peta interaktif.

### Fitur Utama

- **Dashboard peta GIS interaktif** - memvisualisasikan poligon bidang kebun, peringatan deforestasi, filter spasial, dan layer peta khusus menggunakan Leaflet.
- **Pengelolaan ketertelusuran** - menangani data petani, bidang kebun, data penjualan, serta kepatuhan GAP yang mencakup praktik produksi, penggunaan pestisida, penggunaan pupuk, dan pengelolaan limbah berbahaya dan beracun (B3).
- **Pengaturan dan kontrol akses berbasis peran** - mendukung pengelolaan pengguna, profil pengguna, peran berbasis izin (Administrator, Ketua Kelompok Tani, Instansi Pemerintah, dan Mitra Pabrik), serta konfigurasi layer peta.

### Teknologi Frontend

- **Framework:** Next.js 15 dengan App Router dan Turbopack
- **UI:** React 19 dan Tailwind CSS
- **Pengelolaan state:** Redux Toolkit
- **Pemetaan:** Leaflet, React Leaflet, dan Turf.js
- **Tabel dan grafik:** AG Grid, Chart.js, dan D3
- **Formulir:** Formik dan Yup
- **Klien API:** Axios
- **Komponen UI:** Prinsip Atomic Design dengan Radix UI

### Prasyarat Frontend

- Node.js 18 atau versi yang lebih baru; versi 20 atau yang lebih baru direkomendasikan
- npm; pnpm dan yarn juga didukung

### Penyiapan Frontend

1. Clone repositori:

   ```bash
   git clone https://github.com/it-vbl/safetrace-frontend.git
   cd safetrace-frontend
   ```

2. Instal dependensi:

   ```bash
   npm install
   ```

3. Salin file environment contoh:

   ```bash
   cp .env.local.example .env
   ```

   Buka `.env`, kemudian isi `NEXT_PUBLIC_BASE_URL` dengan URL dasar REST API backend. Nilai ini wajib diisi.

4. Jalankan aplikasi:

   ```bash
   npm run dev
   ```

Aplikasi akan tersedia di [http://localhost:3000](http://localhost:3000).

### Skrip yang Tersedia

- `npm run dev` - menjalankan development server menggunakan Turbopack.
- `npm run build` - membuat build produksi yang telah dioptimalkan.
- `npm run start` - menjalankan build produksi.
- `npm run lint` - menjalankan ESLint untuk memeriksa potensi kesalahan dalam codebase.

### Struktur Proyek Frontend

Frontend menggunakan arsitektur Atomic Design agar library komponen tetap mudah dikembangkan dan dipelihara.

```text
src/
├── app/             # Direktori utama Next.js App Router (halaman dan layout)
├── assets/          # Ikon SVG dan gambar khusus proyek
├── components/      # Komponen UI yang dapat digunakan kembali (Atomic Design)
│   ├── atoms/       # Komponen terkecil (Button, Input, Icon)
│   ├── molecules/   # Kombinasi beberapa atom (Form Field, Card Header)
│   ├── organisms/   # Kelompok molecule (Modal, Complex Form, Header)
│   ├── providers/   # Context provider (Redux, Toast, Theme)
│   └── ui/          # Komponen yang bersumber dari Radix UI atau Shadcn
├── config/          # Konfigurasi tema dan aset dasar
├── constants/       # Variabel dan enum statis global
├── hooks/           # React hook khusus
├── i18n/            # Konfigurasi lokalisasi dan data terjemahan
├── libs/            # Library utilitas dan helper perizinan
├── services/        # Integrasi REST API menggunakan Axios
├── store/           # Konfigurasi dan slice pengelolaan state Redux
├── styles/          # CSS global dan dasar Tailwind
├── types/           # Type dan interface TypeScript
└── utils/           # Formatter, helper environment, dan utilitas umum

public/              # Logo, latar belakang, dan ikon yang disajikan secara publik
```

### Penyesuaian Tema dan Logo

1. **Warna merek:** Perbarui warna tema di `src/config/brand.ts`. Class Tailwind yang sesuai dibuat secara otomatis dari file ini.
2. **Logo dan aset:** Perbarui path logo institusi, latar belakang login, dan aset lainnya di `src/config/assets.ts`, kemudian tempatkan file fisiknya di direktori `/public`.

## Dokumentasi

- [README SafeTrace (PDF)](./Safetrace%20-%20Readme.pdf)
- [Panduan Pengguna SafeTrace (PDF)](./Safetrace%20-%20User%20Guide.pdf)
- [Penilaian Risiko WHISP - Bahasa Inggris (PDF)](./Whisp_Risk_Assessment_English.pdf)
- [Baseline Hutan dan Peringatan Kehilangan Tutupan Hutan - Bahasa Inggris (PDF)](./CUKK_Forest_Baseline_and_Loss_Alert_English.pdf)

## Lisensi

Frontend dan backend dirilis menggunakan Lisensi MIT. Lihat file `LICENSE` di masing-masing repositori untuk informasi selengkapnya.
