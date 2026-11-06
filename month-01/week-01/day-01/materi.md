# 🟢 Day 01 — Mengenal Internet of Things (IoT)

Hari ini kita belum perlu merangkai komponen dengan listrik. Kita mulai dari fondasi konsep berpikir yang benar, karena pemahaman inilah yang akan memandu pemilihan sensor, mikrokontroler, protokol Wi-Fi, MQTT, hingga integrasi cloud pada hari-hari berikutnya.

---

## 1. Analisis Masalah

### Pertanyaan Utama Hari Ini:
> **Apa sebenarnya yang membedakan sebuah sistem biasa (embedded system) dengan Internet of Things (IoT)?**

Sering kali pemula memiliki anggapan:
> *"Kalau sudah pakai ESP32 dan dipasang sensor, itu otomatis sudah disebut IoT."*

Pernyataan ini **belum tentu tepat**.

ESP32 yang membaca sensor suhu lalu menampilkan angkanya ke layar LCD bisa jadi hanyalah sebuah **Embedded System mandiri (standalone)**, bukan sistem IoT. Kita harus melihat bagaimana data mengalir, di mana data disimpan, dan bagaimana perangkat berinteraksi dengan dunia luar.

---

## 2. Teori Dasar

### Apa Itu IoT?
**Internet of Things (IoT)** adalah infrastruktur jaringan global yang menghubungkan objek fisik melalui kemampuan komputasi dan komunikasi, sehingga perangkat dapat:
1. **Mengumpulkan data** dari lingkungan fisik (*Sensing*).
2. **Memproses data** secara lokal maupun terdistribusi (*Processing*).
3. **Mengirimkan dan menerima data/perintah** melalui jaringan (*Communication*).
4. **Mengambil keputusan atau melakukan tindakan fisik** berdasarkan data (*Actuation*).

```text
┌──────────────┐       ┌──────────────┐       ┌──────────────┐       ┌──────────────┐
│    SENSE     │       │   PROCESS    │       │ COMMUNICATE  │       │     ACT      │
│ (Input Data) │ ───►  │  (Komputasi) │ ───►  │  (Jaringan)  │ ───►  │ (Aksi Fisik) │
│    DHT22     │       │    ESP32     │       │  Wi-Fi/MQTT  │       │  Relay / Fan │
└──────────────┘       └──────────────┘       └──────────────┘       └──────────────┘
```

### Arsitektur Aliran Data IoT:
```text
         [ Sensor: DHT22 ]
                 │ (Membaca Suhu: 31°C, Hum: 75%)
                 ▼
         [ ESP32 (Microcontroller) ]
                 │ (Formatting data ke JSON: {"temp":31, "hum":75})
                 ▼
         [ Wi-Fi / Router Lokal ]
                 │ (Protokol HTTP/MQTT)
                 ▼
         [ Cloud / Web Server / Broker ]
                 │ (Penyimpanan Database & Pengolahan)
                 ▼
         [ Dashboard Web / Mobile App ]
                 │
                 ▼
         [ Pengguna / Otomasi Lanjutan ]
```

---

## 3. Empat Pilar Utama IoT

Dalam setiap sistem IoT, selalu terdapat empat bagian penting:

### 1. Sense (Sensor / Input)
Perangkat menangkap besaran fisik dari dunia nyata dan mengubahnya menjadi sinyal listrik.
- **DHT22**: Suhu dan kelembapan udara.
- **HC-SR04**: Jarak objek menggunakan gelombang ultrasonik.
- **TCRT5000**: Refleksi cahaya inframerah (deteksi garis/jarak dekat).
- **Capacitive Soil Moisture**: Kadar air di dalam tanah.
- **Water Level Sensor**: Ketinggian permukaan air.

### 2. Process (Pemrosesan / Otak)
Mikrokontroler menerima sinyal dari sensor, melakukan konversi data, menerapkan logika komputasi (*thresholding*, filtering), dan menyiapkan paket data.
- **ESP32**: Memiliki prosesor Dual-Core 32-bit (Xtensa LX6), memori SRAM 520KB, clock speed hingga 240MHz, serta modul radio Wi-Fi & Bluetooth bawaan.

### 3. Communicate (Komunikasi / Jaringan)
Perangkat mengirimkan data atau mendengarkan perintah dari luar.
- **Media**: Wi-Fi (802.11 b/g/n), Bluetooth/BLE.
- **Protokol**: HTTP/REST, MQTT, WebSocket, CoAP.
- **Jaringan**: Local Area Network (LAN) atau Wide Area Network (Internet/Cloud).

### 4. Act (Aktuator / Output)
Perangkat mengubah sinyal listrik perintah menjadi aksi fisik nyata atau representasi informasi.
- **Relay**: Sakelar elektrik untuk menyalakan perangkat berdaya besar (lampu 220V, pompa air).
- **Servo Motor (SG90/MG996R)**: Penggerak sudut presisi (buka-tutup palang, kunci pintu).
- **DC Motor (via driver L298N/L9110S)**: Penggerak roda kendaraan/robot.
- **Buzzer & LED**: Indikator alarm dan status sistem.
- **LCD I2C 16x2 / OLED**: Tampilan informasi langsung di dekat perangkat.

---

## 4. IoT vs Embedded System

| Kriteria | Embedded System Murni | Internet of Things (IoT) |
|---|---|---|
| **Fokus Utama** | Menjalankan fungsi spesifik secara mandiri (*standalone*). | Kolaborasi antar-perangkat dan integrasi data jarak jauh. |
| **Konektivitas Jaringan** | Sering kali tidak ada koneksi jaringan (hanya I2C, SPI, UART lokal). | Wajib memiliki koneksi jaringan (Wi-Fi, Ethernet, LoRa, Seluler). |
| **Penyimpanan Data** | Terbatas pada RAM/Flash lokal, sering hilang saat restart. | Disimpan secara terpusat di Database/Cloud (*Historical Logging*). |
| **Akses & Kontrol** | Harus diakses langsung di depan alat (tombol fisik, layar lokal). | Dapat diakses dan dikontrol dari mana saja melalui Web/Aplikasi. |
| **Contoh** | Mesin cuci manual, kalkulator digital, termometer badan digital biasa. | Smart AC yang bisa diatur lewat HP dari luar rumah, Smart Farming dengan monitoring web. |

> **Rumus Pemahaman:**
> *Semua perangkat IoT adalah Embedded System, tetapi tidak semua Embedded System adalah IoT.*

---

## 5. Studi Kasus Kontekstual: Smart RC Car

Mari kita ambil contoh perangkat yang sering dibuat pemula: **Robot Mobil (RC Car)**.

### Skenario 1: Robot Penghindar Rintangan (Obstacle Avoidance)
```text
[ HC-SR04 ] ──► [ ESP32 ] ──► [ Driver L298N ] ──► [ DC Motor ]
```
- Jika jarak < 15 cm, ESP32 membelokkan roda ke kanan.
- **Status:** Ini adalah **Embedded Control System**, BUKAN IoT. Kenapa? Karena sistem ini bekerja sepenuhnya tertutup di dalam mikrokontroler tanpa pertukaran data jaringan ke entitas lain.

### Skenario 2: IoT Telemetry Connected RC Car
```text
   [ HC-SR04 ] ────────┐
                       ▼
                    [ ESP32 ] ◄── Wi-Fi ──► [ Web Dashboard di Laptop/HP ]
                       │                         - Kontrol Setir Real-Time
                       ▼                         - Monitoring Jarak (Grafik Telemetri)
               [ Driver L298N ]                  - Peringatan Tabrakan di Browser
                       │
                  [ DC Motor ]
```
- Data jarak dikirim ke laptop secara nirkabel via WebSocket/MQTT.
- Pengguna dapat mengendalikan mobil dari browser dan melihat riwayat halangan yang ditemui.
- **Status:** Ini telah menjadi **Sistem IoT**, karena ada pertukaran data dua arah melalui layer jaringan.

---

## 6. Praktikum Day-01 (Analisis & Pemetaan Sistem)

Isilah tabel pemetaan berikut untuk menguji pemahaman Anda:

### Studi Kasus 1: Smart Greenhouse (Kebun Pintar)
Sebuah sistem menyiram tanaman otomatis saat tanah kering dan melaporkan kelembapan ke website pemilik kebun.
- **Sensor:** ?
- **Processing:** ?
- **Communication:** ?
- **Actuator/Output:** ?

### Studi Kasus 2: Sistem Deteksi Kebocoran Air Rumah Tangga
Sebuah sistem mematikan keran utama jika mendeteksi genangan air dan mengirim notifikasi Telegram ke HP pemilik.
- **Sensor:** ?
- **Processing:** ?
- **Communication:** ?
- **Actuator/Output:** ?

---

## 7. Praktik Utama: Desain Arsitektur IoT Mandiri

Pilihlah komponen yang Anda miliki saat ini:
- **Pilihan Sensor:** DHT22, HC-SR04, TCRT5000, Soil Moisture, atau Water Level.
- **Otak:** ESP32 DevKit.
- **Pilihan Aktuator/Output:** LED, Buzzer, LCD I2C, Relay, atau Servo.

Tuliskan rancangan arsitektur sistem IoT impian Anda:
1. **Nama Sistem:**
2. **Masalah Nyata yang Ingin Diselesaikan:**
3. **Data yang Dibaca (Sensor):**
4. **Logika yang Diterapkan di ESP32:**
5. **Jalur Komunikasi & Data yang Dikirim:**
6. **Aksi Fisik yang Dijalankan (Aktuator):**

---

## 8. 🧠 Tantangan Berpikir Analitis

Jawablah pertanyaan-pertanyaan ini menggunakan pemikiran kritis Anda:
1. **Kasus:** ESP32 diprogram membaca suhu dari DHT22 dan menampilkan angkanya pada layar LCD I2C 16x2. Di dalam kode program, Wi-Fi dinyalakan dan terkoneksi ke router rumah, tetapi program sama sekali tidak mengirim atau menerima data apa pun melalui Wi-Fi. **Apakah sistem ini sudah bisa diklaim sebagai sistem IoT? Jelaskan alasan logis Anda.**
2. **Konektivitas:** Apakah sistem IoT **harus selalu terhubung ke internet publik (Global Internet)**? Bisakah sebuah sistem yang hanya terhubung ke Local Area Network (LAN) di dalam ruangan tanpa akses internet disebut IoT? Jelaskan bedanya.
3. **Keandalan:** Mengapa sistem kontrol keselamatan kritis (seperti rem darurat mobil atau tombol emergency stop mesin pabrik) tidak boleh 100% bergantung pada koneksi internet/cloud?

---

## 9. 📚 Panduan Dokumentasi GitHub (README.md)

Buat atau perbarui file `README.md` pada folder:
`month-01/week-01/day-01/README.md`

Gunakan format standar dokumentasi berikut:
```markdown
# Day 01 — Introduction to IoT

## 🎯 Objective
Memahami konsep dasar Internet of Things (IoT), arsitektur 4 pilar (Sense, Process, Communicate, Act), dan perbedaannya dengan standalone embedded system.

## 📚 What I Learned
- Pengertian dasar IoT dan bagaimana data mengalir dari sensor ke cloud.
- 4 Bagian utama sistem IoT: Sensing, Processing, Communication, Actuation.
- Perbedaan esensial antara Embedded System mandiri dan IoT.
- Karakteristik perangkat mikrokontroler (ESP32) sebagai node IoT.

## 🔄 IoT Architecture Diagram
[Gambarkan diagram alur data sistem yang Anda rancang]

## 🔧 Components
- ESP32 DevKit
- Sensor: [Sebutkan]
- Aktuator: [Sebutkan]

## 🧪 Experiment & Analysis
[Tuliskan hasil analisis kasus praktikum dan desain sistem mandiri Anda]

## 🧠 Questions & Answers
[Jawab ketiga pertanyaan pada tantangan berpikir dengan analisis sendiri]

## 📝 Conclusion
[Tuliskan 1-2 paragraf kesimpulan pemahaman Anda hari ini]
```

---

## 10. Git Workflow Day-01

Setelah menyelesaikan catatan dan file dokumentasi:
```bash
git status
git add month-01/week-01/day-01/
git commit -m "docs(day-01): complete introduction to IoT and architecture analysis"
git push origin main
```

---

## ✅ Checklist Kelulusan Day-01
- [ ] Memahami definisi konkret IoT di luar sekadar hardware + coding.
- [ ] Mampu mengidentifikasi 4 pilar utama IoT pada berbagai alat di sekitar.
- [ ] Mampu membedakan kapan suatu perangkat disebut embedded system murni vs IoT.
- [ ] Menyelesaikan tabel praktikum analisis sistem.
- [ ] Menjawab 3 pertanyaan tantangan berpikir.
- [ ] Mengisi file `README.md` di folder `day-01` dengan pemikiran sendiri.
- [ ] Melakukan Git commit dan push ke GitHub.

---

## 🎯 Prinsip Esensial Hari Ini
> **"IoT bukan sekadar menambahkan Wi-Fi pada mikrokontroler; IoT adalah tentang bagaimana data bermakna dihantarkan tepat waktu untuk menghasilkan aksi yang bermanfaat."**
