# 📐 System Blueprint — Smart Greenhouse IoT Node

Dokumen perancangan cetak biru sistem perangkat keras (*Hardware Architecture Blueprint*) terpadu sebagai tugas evaluasi akhir Minggu 01 (Week 01).

---

## 1. Ikhtisar Sistem
Sistem ini berfungsi memantau kondisi iklim mikro tanaman secara real-time dan melakukan penyiraman otomatis cerdas saat tanah kering, serta mengirim data telemetri ke Web Dashboard melalui Wi-Fi.

---

## 2. Pemetaan Pinout ESP32 (Pin Allocation Table)

Semua pin dipilih secara cermat berdasarkan matriks keselamatan hardware agar tidak bentrok dengan Strapping Pins maupun modul radio Wi-Fi:

| Perangkat / Komponen | Tipe Sinyal | Pin Perangkat | Terhubung ke Pin ESP32 | Alasan Pemilihan & Keamanan |
|---|---|---|---|---|
| **Sensor Tanah (Capacitive)** | Analog Input | Sinyal (AOUT) | **GPIO 34 (ADC1_CH6)** | Pin input-only ADC1, 100% bebas konflik dengan Wi-Fi |
| **Sensor Suhu (DHT22)** | Digital Bi-directional | DATA | **GPIO 4** | Pin digital standar; ditambah Pull-Up resistor $4.7\text{k}\Omega$ ke 3.3V |
| **Modul Relay 5V (Pompa)** | Digital Output | IN (Trigger) | **GPIO 18** | Pin digital output murni bebas strapping pin |
| **LCD Display I2C 16x2** | Bus Serial (I2C) | SDA | **GPIO 21 (SDA default)** | Jalur data I2C standar ESP32 |
| **LCD Display I2C 16x2** | Bus Serial (I2C) | SCL | **GPIO 22 (SCL default)** | Jalur clock I2C standar ESP32 |
| **LED Hijau (Status Wi-Fi)** | Digital Output | Anoda (+) | **GPIO 19** | Dilengkapi resistor pembatas arus $220\ \Omega$ seri |
| **LED Merah (Penyiraman)** | Digital Output | Anoda (+) | **GPIO 23** | Dilengkapi resistor pembatas arus $220\ \Omega$ seri |

---

## 3. Skema Catu Daya & Common Ground Architecture

```text
              [ Adaptor Utama 5V 2A DC ]
                          │
             ┌────────────┴────────────┐
             │                         │
             ▼ (5V DC)                 ▼ (5V DC)
      [ Pin VIN ESP32 ]          [ VCC Modul Relay ]
             │                   [ VCC LCD I2C ]
             ▼
    (Regulator LDO 3.3V)
             │
             ├─────────────────► VCC Sensor DHT22 (3.3V)
             ├─────────────────► VCC Sensor Tanah (3.3V)
             │
             ▼
       [ PIN GND ESP32 ] ◄═══════ COMMON GROUND ═══════► [ GND SEMUA PERANGKAT ]
```

---

## 4. Power & Current Budget Calculation

| Komponen | Tegangan Kerja ($V$) | Arus Tipikal / Puncak ($I$) | Daya Diserap ($P$) |
|---|---|---|---|
| ESP32 DevKit (Wi-Fi TX Peak) | 5.0 V (VIN) | 240 mA | 1.200 W |
| Sensor DHT22 | 3.3 V | 1.5 mA | 0.005 W |
| Sensor Tanah Capacitive | 3.3 V | 5.0 mA | 0.016 W |
| Modul LCD I2C 16x2 | 5.0 V | 25 mA | 0.125 W |
| Modul Relay 5V (Aktif) | 5.0 V | 70 mA | 0.350 W |
| 2x LED Indikator ($2 \times 7\text{mA}$) | 3.3 V | 14 mA | 0.046 W |
| **Total Arus Sistem Maksimal** | — | **~355.5 mA** | Kondisi seluruh komponen menyala |
| **Batas Arus Adaptor (5V 2A)** | — | **2.000 mA** | Cadangan daya: $\approx 82\%$ (Sangat Aman & Dingin) |

---

## 5. Ringkasan Fitur Proteksi Listrik Terpasang
1. **Proteksi Arus GPIO:** Masing-masing LED menggunakan resistor $220\ \Omega$ membatasi arus di angka $6.8\text{ mA}$ (jauh di bawah batas 40mA).
2. **Proteksi Bus Komunikasi:** Jalur DHT22 diberi resistor Pull-Up $4.7\text{k}\Omega$ menjaga integritas data tanpa noise.
3. **Pemisahan Jalur Beban:** Relay koil mengambil arus dari jalur 5V VIN, bukan dari pin 3V3 ESP32, untuk mencegah *voltage dip*.
4. **Common Ground:** Seluruh kutub negatif komponen disatukan ke pin GND ESP32 sebagai referensi 0V mutlak.
