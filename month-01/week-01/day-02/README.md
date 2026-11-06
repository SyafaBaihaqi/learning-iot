# Day 02 — Voltage & ESP32 Logic Levels

## 🎯 Objective
Memahami konsep beda potensial listrik (Voltage), karakteristik tegangan pada ESP32 DevKit, batas toleransi pin GPIO 3.3V, serta menguasai teknik pengukuran tegangan DC dengan Multimeter Digital.

## 📚 What I Learned
- **Konsep Beda Potensial:** Tegangan ($V$) adalah gaya dorong atau tekanan yang menyebabkan elektron mengalir. Tegangan selalu diukur di antara dua titik referensi (titik ukur relatif terhadap Ground/GND).
- **Arsitektur Daya ESP32:**
  - **Pin VIN (5V):** Jalur masukan daya dari USB atau adaptor eksternal, yang kemudian diturunkan oleh IC regulator onboard (*LDO AMS1117 / ME6211*) menjadi 3.3V.
  - **Pin 3V3:** Jalur tegangan output ter-regulasi khusus untuk chip inti ESP32 dan sensor berdaya rendah.
  - **Pin GND:** Titik referensi $0\text{V}$.
- **Logic Level 3.3V vs 5V:**
  - Pin GPIO ESP32 bekerja pada **3.3V Logic Level** dan **TIDAK 5V Tolerant** (maksimal mutlak adalah 3.6V).
  - Memberikan sinyal 5V langsung ke pin GPIO (misalnya dari pin Echo sensor HC-SR04) dapat merusak sirkuit transistor internal secara permanen.
- **Teknik Proteksi Voltage Divider:** Menggunakan pasangan resistor pembagi tegangan ($R_1 = 1k\Omega$ dan $R_2 = 2k\Omega$) untuk menurunkan sinyal sensor 5V menjadi 3.33V sebelum masuk ke pin input ESP32.
- **Penyebab Brownout Reset:** Penurunan tegangan mendadak (di bawah ~2.8V) akibat lonjakan konsumsi arus saat Wi-Fi menyala, yang memicu sistem proteksi internal CPU me-restart perangkat.

## 📊 Measurement Log
Hasil pengukuran langsung menggunakan Multimeter Digital terdokumentasi lengkap di:
📄 [`measurement.md`](./measurement.md)

| Pengukuran | Nominal | Hasil Pengukuran Aktual |
|---|---|---|
| Pin VIN ESP32 (Power via USB) | 5.0 V | 4.98 V |
| Pin 3V3 ESP32 (Regulator Out) | 3.3 V | 3.30 V |
| Output Baterai Li-ion 18650 | 3.7 V | 4.12 V |

## 🧠 Questions & Answers
1. **Mengapa kabel GND antara ESP32 dan modul eksternal (seperti L298N) wajib digabung (Common Ground)?**
   Tegangan adalah *beda potensial relatif terhadap titik nol*. Jika ground kedua sistem terpisah, ESP32 dan driver motor memiliki titik referensi 0V yang berbeda (mengambang / *floating*). Akibatnya, sinyal logika HIGH (3.3V) dari GPIO ESP32 tidak akan terbaca sebagai 3.3V oleh driver motor, sehingga komunikasi dan kendali PWM gagal total.
2. **Apa yang terjadi jika polaritas tegangan VIN dan GND terbalik?**
   Sebagian besar board development ESP32 tidak memiliki dioda proteksi polaritas terbalik (*reverse polarity protection diode*) pada pin header-nya. Membalik kabel positif dan negatif akan langsung menghantarkan tegangan negatif ke regulator atau chip, yang berakibat fatal (kerusakan permanen / *fried chip*).
3. **Berapa tegangan yang keluar pada pin GPIO saat diset `HIGH` pada ESP32 dibandingkan Arduino Uno?**
   - Pada **ESP32**, output sinyal `HIGH` adalah **~3.3 Volt**.
   - Pada **Arduino Uno**, output sinyal `HIGH` adalah **~5.0 Volt**.

## 📝 Conclusion
Tegangan adalah dasar dari setiap keputusan perancangan sistem elektronika dan IoT. Memastikan bahwa sumber daya stabil dan level tegangan logika antarkomponen cocok (*level matched*) adalah langkah preventif paling krusial untuk mencegah kerusakan hardware dan masalah ketidakstabilan sistem (*brownout/freezing*) di masa depan.
