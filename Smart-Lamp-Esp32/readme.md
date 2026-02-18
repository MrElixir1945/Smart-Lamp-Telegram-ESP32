# 💡 Smart-Lamp-Telegram-ESP32

> Sistem kendali saklar lampu fisik dari jarak jauh via Telegram — menggunakan ESP32 + Servo MG90s yang menekan saklar dinding secara mekanik, tanpa modifikasi instalasi listrik.

![MicroPython](https://img.shields.io/badge/Firmware-MicroPython-green?logo=python) ![Python](https://img.shields.io/badge/Backend-Python-blue?logo=python) ![ESP32](https://img.shields.io/badge/Hardware-ESP32-red) ![Platform](https://img.shields.io/badge/Server-Proxmox%20VE-orange) ![License](https://img.shields.io/badge/License-MIT-green)

---

## 🧠 Ide & Konsep

Project ini lahir dari kebutuhan sederhana: **mematikan lampu kamar dari kasur tanpa harus berdiri.**

Pendekatannya unik — alih-alih menggunakan relay yang membutuhkan modifikasi kabel listrik, sistem ini menggunakan **servo motor yang menekan saklar dinding secara fisik dan mekanik**. Lebih aman, lebih simpel, dan bisa dipasang/dilepas kapan saja.

---

## 🏗️ Arsitektur Sistem

```
[Telegram User]
      │
      │ Inline Button
      ▼
[Bot Server - Python]  ←── Baca jadwal dari jadwal.json
      │
      │ HTTP GET Request (Local Network)
      ▼
[ESP32 Web Server - MicroPython]
      │
      │ PWM Signal
      ▼
[Servo MG90s]
      │
      │ Tekan fisik
      ▼
[Saklar Dinding 💡]
```

---

## 🛠️ Tech Stack

### Hardware
| Komponen | Detail |
|---|---|
| Mikrokontroler | ESP32 DevKit V1 |
| Aktuator | Servo MG90s |
| Mekanisme | Servo menekan saklar dinding fisik |

### Software
| Komponen | Detail |
|---|---|
| Firmware (ESP32) | MicroPython |
| Backend Bot | Python 3 |
| Bot Framework | `python-telegram-bot` |
| HTTP Client | `requests` |
| Scheduling | `APScheduler` via `python-telegram-bot` Job Queue |
| Timezone | `pytz` — Asia/Makassar (WITA) |
| Config | `python-dotenv` |
| Server | Proxmox VE (LXC Container Ubuntu) |

---

## ✨ Fitur

### Kontrol Manual
- 💡 **Toggle Saklar** — Nyala/Mati via tombol Inline Keyboard Telegram
- 📊 **Status Real-time** — Cek kondisi lampu saat ini langsung dari menu

### Penjadwalan Otomatis
- 🔔 **Jadwal Sekali** — Set alarm sekali pakai (auto-hapus setelah jalan)
- 🔁 **Jadwal Rutin** — Set alarm harian yang berulang
- 📋 **List Jadwal** — Lihat semua jadwal aktif
- 🗑️ **Hapus Jadwal** — Hapus jadwal tertentu via nomor urut
- 💾 **Persistent** — Jadwal tersimpan di `jadwal.json` dan tetap aktif setelah bot restart

### Keamanan & Stabilitas (ESP32)
- 🔑 **API Key Authentication** — Setiap request dicek API key-nya, koneksi ditolak jika salah
- 🐕 **Watchdog Timer** — ESP32 auto-restart jika hang lebih dari 30 detik
- 📶 **Auto-reconnect WiFi** — Otomatis reconnect jika koneksi WiFi putus
- 🧹 **Memory Management** — `gc.collect()` rutin untuk cegah memory leak di MicroPython

---

## ⚙️ Cara Kerja Servo

Servo SG90 dipasang secara mekanik di atas saklar dinding:

| State | Sudut Servo | Aksi |
|---|---|---|
| Lampu HIDUP | 110° | Servo menekan saklar ke bawah |
| Lampu MATI | 60° | Servo kembali ke posisi standby |

Sinyal PWM dimatikan setelah servo bergerak (`servo.deinit()`) untuk mencegah dengung dan panas berlebih.

---

## 🚀 Cara Setup

### Bagian 1: Firmware ESP32

**1. Install MicroPython di ESP32**

Download firmware MicroPython untuk ESP32 di [micropython.org](https://micropython.org/download/esp32/), lalu flash:
```bash
pip install esptool
esptool.py --chip esp32 erase_flash
esptool.py --chip esp32 write_flash -z 0x1000 firmware.bin
```

**2. Upload `firmware.py` ke ESP32**

Gunakan tools seperti **Thonny IDE** atau **ampy**:
```bash
pip install adafruit-ampy
ampy --port /dev/ttyUSB0 put firmware.py main.py
```

**3. Konfigurasi di `firmware.py`**
```python
SSID = "NAMA_WIFI_KAMU"
PASSWORD = "PASSWORD_WIFI_KAMU"
API_KEY = "API_KEY_RAHASIA_KAMU"  # Samakan dengan .env bot
SERVO_PIN = 13
```

---

### Bagian 2: Bot Server

**1. Clone repo**
```bash
git clone https://github.com/MrElixir1945/Smart-Room-Telegram-ESP32.git
cd Smart-Room-Telegram-ESP32
```

**2. Buat virtual environment**
```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

**3. Konfigurasi `.env`**

Copy file `.env.example` dan isi dengan data kamu:
```bash
cp .env.example .env
nano .env
```

```env
TELEGRAM_TOKEN=token_bot_telegram_kamu
ALLOWED_USER_ID=user_id_telegram_kamu
ESP_IP=http://192.168.x.x        # IP lokal ESP32 di jaringan WiFi
ESP_API_KEY=api_key_rahasia_kamu  # Harus sama dengan di firmware.py
```

**4. Jalankan bot**
```bash
python bot.py
```

---

## 📁 Struktur Project

```
Smart-Room-Telegram-ESP32/
├── firmware.py         # MicroPython firmware untuk ESP32 (Web Server)
├── bot.py              # Backend bot Telegram (Python)
├── .env.example        # Template konfigurasi (isi & rename jadi .env)
├── requirements.txt    # Python dependencies
└── jadwal.json         # Auto-generated: penyimpanan data jadwal
```

---

## 📋 Contoh Penggunaan

```
User:   /start
Bot:    🤖 PANEL KONTROL
        Status Lampu: MATI 🌑
        [💡 SAKLAR (TOGGLE)] [⏰ ATUR JADWAL]

User:   klik [💡 SAKLAR]
Bot:    ✅ Berhasil!
        Status Lampu: HIDUP 💡
        → Servo bergerak ke 110° dan menekan saklar fisik

User:   klik [⏰ ATUR JADWAL] → [🔔 SEKALI]
Bot:    📝 INPUT SEKALI
        Ketik jam: 22.00

User:   22.00
Bot:    ✅ Diset: 22.00
        → Jam 22:00 servo otomatis menyalakan lampu
```

---

## ⚠️ Catatan Penting

- ESP32 dan Bot Server harus berada di **jaringan WiFi yang sama** (lokal)
- IP ESP32 sebaiknya di-**set static** di router agar tidak berubah
- `jadwal.json` ter-generate otomatis, tidak perlu dibuat manual
- Servo perlu dikalibrasi sudutnya sesuai posisi pemasangan fisik di saklar kamu

---

## 👤 Author

**Mr. Elixir** — [@MrElixir1945](https://github.com/MrElixir1945)

*Self-hosted on Proxmox VE Home Server*
