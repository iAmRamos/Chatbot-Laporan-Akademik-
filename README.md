# 🤖 Chatbot Akademik Telegram — SDN 04 Rawang Kota Pariaman

Sistem chatbot berbasis Telegram untuk menampilkan laporan akademik siswa secara otomatis dan real-time. Orang tua/wali murid dapat mengakses nilai, absensi, jadwal, dan analisis perkembangan akademik siswa langsung melalui aplikasi Telegram.

---

## 📋 Deskripsi

Sebelumnya, penyampaian informasi akademik di SDN 04 Rawang Kota Pariaman dilakukan secara manual — orang tua harus menghubungi guru satu per satu atau menunggu pembagian rapor di akhir semester. Sistem ini hadir sebagai solusi otomasi yang memungkinkan akses informasi akademik kapan saja dan di mana saja.

---

## ✨ Fitur

| Fitur | Deskripsi |
|-------|-----------|
| 🔐 **Login / Autentikasi** | Verifikasi identitas pengguna via Telegram ID sebelum akses data |
| 📊 **Nilai Formatif** | Rekap nilai tugas dan ulangan harian |
| 📝 **Nilai Sumatif Akhir Materi** | Nilai ujian per materi/bab |
| 📋 **Nilai Sumatif Akhir Semester** | Rekap nilai akhir semester |
| 🗓️ **Jadwal Pelajaran** | Jadwal pelajaran harian per kelas |
| 📌 **Rekap Absensi** | Data kehadiran siswa |
| 🤖 **Analisis AI** | Ringkasan naratif perkembangan akademik siswa berbasis Google Gemini |
| 💬 **AI Chat** | Obrolan bebas dengan "Pak Ceria" untuk motivasi dan panduan belajar |
| ❓ **Bantuan (Help)** | Panduan penggunaan chatbot |
| 🚪 **Logout** | Keluar dari sesi aktif |

---

## 🏗️ Arsitektur Sistem

```
Wali Murid (Telegram)
        │
        ▼
  Telegram Bot API
        │
        ▼
    n8n Workflow  ──────────► Google Gemini API (Analisis AI)
        │
        ▼
  Google Sheets (Database)
```

**Alur kerja:**
1. Wali murid kirim perintah via Telegram
2. Telegram Bot API meneruskan ke n8n webhook
3. n8n memproses request, query data ke Google Sheets
4. Untuk fitur analisis: n8n memanggil Google Gemini API
5. Respons dikirim balik ke pengguna via Telegram

---

## 🛠️ Tech Stack

| Komponen | Teknologi |
|----------|-----------|
| Chatbot Platform | [Telegram Bot API](https://core.telegram.org/bots/api) |
| Workflow Automation | [n8n](https://n8n.io/) |
| Database | [Google Sheets](https://sheets.google.com) + Google Sheets API |
| AI Analysis | [Google Gemini API](https://ai.google.dev/) |
| Containerization | [Docker](https://docker.com) |
| Tunneling (dev) | [Ngrok](https://ngrok.com) |
| Scripting | JavaScript (n8n Function Node) |

---

## ⚙️ Instalasi & Konfigurasi

### Prasyarat
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) terinstall
- Akun [Ngrok](https://ngrok.com) (untuk tunneling lokal ke internet)
- Akun Telegram & Bot Token (dari [@BotFather](https://t.me/BotFather))
- Google Account + Google Sheets API aktif
- Google Gemini API Key

### 1. Clone Repository
```bash
git clone https://github.com/username/chatbot-akademik-telegram.git
cd chatbot-akademik-telegram
```

### 2. Konfigurasi Environment
Buat file `.env` dari template:
```bash
cp .env.example .env
```

Isi nilai berikut di `.env`:
```env
TELEGRAM_BOT_TOKEN=your_telegram_bot_token
GOOGLE_GEMINI_API_KEY=your_google_gemini_api_key
GOOGLE_SHEET_ID=your_google_sheet_id
NGROK_URL=https://xxxx.ngrok-free.app/
POSTGRES_USER=chatbot
POSTGRES_PASSWORD=your_db_password
POSTGRES_DB=db_sekolah
```

### 3. Jalankan Ngrok
Jalankan ngrok untuk expose port n8n ke internet:
```bash
ngrok http 5678
```
Salin URL HTTPS yang muncul (contoh: `https://xxxx.ngrok-free.app`) → masukkan ke `NGROK_URL` di file `.env`

> ⚠️ URL ngrok berubah setiap kali dijalankan ulang. Update `.env` dan restart Docker setiap kali URL berubah.

### 4. Jalankan Docker
```bash
docker compose up -d
```

Layanan yang berjalan:
| Layanan | URL |
|---------|-----|
| n8n | `http://localhost:5678` |
| PostgreSQL | `localhost:5432` |

### 5. Import Workflow n8n
1. Buka `http://localhost:5678`
2. Login ke dashboard n8n
3. Klik **Import** → pilih file `workflow/chatbot-akademik.json`
4. Aktifkan workflow (toggle di pojok kanan atas)

### 6. Setup Google Sheets
1. Buat Google Sheet baru
2. Gunakan struktur dari folder `database/`
3. Aktifkan Google Sheets API di [Google Cloud Console](https://console.cloud.google.com)
4. Hubungkan credentials di n8n → **Credentials** → **Google Sheets OAuth2**

### 7. Daftarkan Webhook Telegram
```bash
curl -X POST "https://api.telegram.org/bot<YOUR_BOT_TOKEN>/setWebhook" \
     -d "url=https://xxxx.ngrok-free.app/webhook/telegram"
```

---

## 📁 Struktur Project

```
chatbot-akademik-telegram/
├── workflow/
│   └── chatbot-akademik.json    # Export workflow n8n
├── database/
│   └── template-sheets.md       # Struktur Google Sheets (template)
├── docs/
│   ├── arsitektur.md            # Dokumentasi arsitektur sistem
│   ├── use-case-diagram.png     # Diagram UML
│   └── screenshot/              # Screenshot tampilan chatbot
├── docker-compose.yml           # Konfigurasi Docker n8n
├── .env.example                 # Template environment variables
├── .gitignore
└── README.md
```

---

## 📸 Screenshot

| Tampilan Awal | Login | Menu Nilai |
|---------------|-------|------------|
| ![Awal](docs/screenshot/awal.png) | ![Login](docs/screenshot/login.png) | ![Nilai](docs/screenshot/nilai.png) |

| Absensi | Jadwal | Analisis AI |
|---------|--------|-------------|
| ![Absensi](docs/screenshot/absensi.png) | ![Jadwal](docs/screenshot/jadwal.png) | ![AI](docs/screenshot/ai.png) |

---

## 🗄️ Struktur Database (Google Sheets)

| Sheet | Kolom Utama |
|-------|-------------|
| `DataPengguna` | Telegramid, Nama, Nisn, Password, Kelas |
| `NilaiAkhir_DB` | Nisn, Mapel, Nilai |
| `Absensi_DB` | Nisn, Sakit, Izin, Alpha, catatan |
| `Jadwal_DB` | Kelas, Hari, Mapel |
| `NilaiFormatif_DB` | Nisn, Mapel, Tp, Nilai |
| `NilaiSumatifMateri_DB` | Nisn, Mapel, Tp, Nilai |
| `NilaiSumatifSemester_DB` | Nisn, Mapel, Tp, Nilai |
| `TujuanPembelajaran_DB` | Kelas, Mapel, Tp, Deskripsi |
| `RataRata_DB` | Kelas, Mapel, RataRata |

> ⚠️ **Catatan:** Template sheet berisi data dummy. Data siswa asli tidak disertakan dalam repository ini demi menjaga privasi.

---

## 🔒 Keamanan

- Autentikasi berbasis Telegram ID — setiap pengguna hanya bisa akses data anaknya sendiri
- Semua kredensial disimpan di `.env` (tidak di-commit ke repository)
- Data siswa asli tidak tersimpan di repository

---

## 🧪 Pengujian

Pengujian dilakukan menggunakan metode **Black Box Testing** — menguji fungsionalitas sistem tanpa melihat kode internal. Seluruh fitur diuji dari sisi input perintah dan output respon chatbot.

Hasil pengujian: **Semua fitur berjalan sesuai spesifikasi** ✅

---

## 📖 Referensi

Penelitian ini merupakan Tugas Akhir (Skripsi) Program Studi Sistem Informasi, Fakultas Ilmu Komputer, Universitas Putra Indonesia "YPTK" Padang, 2026.

- **Penulis:** Ramos Raymond Dasril
---

## 📄 Lisensi

Project ini dibuat untuk keperluan akademik. Silakan gunakan sebagai referensi dengan mencantumkan sumber.
