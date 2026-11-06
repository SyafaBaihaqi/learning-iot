# Day 06 — Digital vs Analog, ADC 12-Bit & PWM

## 🎯 Objective
Memahami perbedaan sifat sinyal digital dan analog, menguasai konversi periferal ADC 12-bit ESP32, memahami batasan kanal ADC1 vs ADC2 saat modul Wi-Fi aktif, serta menerapkan konsep PWM (*Pulse Width Modulation*).

## 📚 What I Learned
- **Sinyal Digital vs Analog:**
  - **Digital:** Bernilai diskrit dua kondisi: Logika 0 ($0\text{V}$) dan Logika 1 ($3.3\text{V}$). Memiliki kekebalan sangat tinggi terhadap derau elektromagnetik.
  - **Analog:** Bernilai kontinu dalam rentang tegangan $0\text{V}$ hingga $3.3\text{V}$, mewakili besaran fisik nyata seperti kelembapan tanah, intensitas cahaya, dan level air.
- **Periferal ADC 12-Bit ESP32:**
  - Menghasilkan rentang angka **0 hingga 4095** ($2^{12} = 4096\text{ tingkatan}$).
  - Rumus konversi tegangan input:
    $$V_{in} = \frac{\text{Nilai ADC}}{4095} \times 3.3\text{ Volt}$$
- **Aturan Kritis Arsitektur Hardware ESP32 (ADC1 vs ADC2):**
  - **ADC1 (GPIO32 - GPIO39):** Sirkuit mandiri, **bebas dan aman digunakan** kapan saja bersamaan dengan koneksi Wi-Fi.
  - **ADC2 (GPIO0, 2, 4, 12-15, 25-27):** Berbagi hardware dengan driver RF Wi-Fi. Begitu fungsi `WiFi.begin()` dipanggil, ADC2 diblokir oleh sistem operasi ESP-IDF sehingga pembacaan `analogRead()` pada pin ADC2 akan rusak/gagal.
- **Konsep PWM (Pulse Width Modulation):**
  - Mengubah output digital menjadi tegangan rata-rata semu ($V_{avg} = \text{Duty Cycle} \times V_{CC}$) dengan mengatur lebar pulsa on/off frekuensi tinggi.
  - Digunakan untuk mengatur tingkat kecerahan LED, kecepatan putar motor DC via L298N, dan sinyal posisi sudut servo SG90.
- **Klasifikasi Perangkat:**
  - **Digital:** DHT22, HC-SR04, Relay 5V, LCD I2C, TCRT5000 (DO).
  - **Analog / PWM:** Capacitive Soil Moisture, Water Level, Potensiometer, Servo SG90, Motor DC (PWM pin EN).

## 📊 ADC & PWM Analysis

### 1. Uji Konversi ADC Sensor Kelembapan Tanah (ADC1 - GPIO34)
| Kondisi Pengujian | Nilai ADC Mentah (12-bit) | Tegangan Konversi ($V$) | Interpretasi Sistem |
|---|---|---|---|
| Sensor di Udara Bebas (Kering) | 3180 | 2.56 V | Kadar air 0% (Tanah Kering Kritis) |
| Sensor di Tanah Agak Kering | 2450 | 1.97 V | Kadar air ~35% (Waktunya Menyiram) |
| Sensor di Tanah Lembap Optimal | 1850 | 1.49 V | Kadar air ~65% (Kondisi Ideal) |
| Sensor Dicelup ke Gelas Air | 1250 | 1.01 V | Kadar air 100% (Tanah Jenuh Air) |

### 2. Efek Duty Cycle PWM pada Tegangan Rata-Rata (VCC = 3.3V)
| Duty Cycle (%) | Nilai Register 8-bit ($0-255$) | Tegangan Rata-rata Output | Efek Beban (LED / Motor) |
|---|---|---|---|
| 0% | 0 | 0.00 V | Mati Total (OFF) |
| 25% | 64 | 0.825 V | LED Redup / Motor Berputar Lambat |
| 50% | 128 | 1.650 V | LED Terang Sedang / Motor Kecepatan Menengah |
| 100% | 255 | 3.300 V | LED Terang Maksimal / Motor Kecepatan Penuh |

## 🧠 Questions & Answers
1. **Mengapa sensor analog di GPIO4 gagal membaca saat Wi-Fi aktif, dan harus dipindah ke mana?**
   GPIO4 terhubung ke modul **ADC2**. Pada ESP32, kanal ADC2 digunakan bersama oleh mikrokontroler dan sirkuit baseband Wi-Fi. Saat Wi-Fi aktif, radio mengambil alih ADC2 secara eksklusif. Solusinya: sensor tanah analog wajib dipindahkan ke pin kanal **ADC1**, seperti **GPIO32, GPIO33, GPIO34, atau GPIO35**, yang memiliki sirkuit independen dari modul radio Wi-Fi.
2. **Apakah PWM 50% menghasilkan tegangan analog DC 1.65V murni?**
   *Tidak.* Secara fisis, sinyal PWM tetap berupa gelombang kotak digital yang berpindah cepat antara 3.3V dan 0V dengan rasio waktu ON dan OFF yang sama ($50:50$). Tegangan 1.65V hanyalah nilai tegangan rata-rata secara matematis (*integral tegangan terhadap waktu*). Komponen mekanis (motor) atau mata manusia (LED) tidak mampu merespons frekuensi tinggi tersebut sehingga menganggapnya sebagai tegangan analog menengah.
3. **Mengapa sensor digital lebih tahan derau (noise) daripada sensor analog pada kabel panjang?**
   Sinyal analog merepresentasikan data melalui level voltase presisi. Jika kabel panjang memiliki resistansi dan terkena interferensi elektromagnetik (menjatuhkan voltase sebesar $0.2\text{V}$), nilai suhu/kelembapan analog akan melenceng total. Sebaliknya, sinyal digital hanya peduli apakah voltase berada di atas batas ambang HIGH ($>2.64\text{V}$) atau di bawah LOW ($<0.8\text{V}$). Penurunan atau derau sebesar $0.2\text{V}$ sama sekali tidak mengubah interpretasi bit 0 dan 1.

## 📝 Conclusion
Sinyal analog memberi kita jendela untuk mengamati dunia nyata, sedangkan periferal ADC dan PWM memungkinkan ESP32 berinteraksi dua arah secara presisi. Mengetahui perbedaan sirkuit ADC1 dan ADC2 adalah pengetahuan arsitektur penting yang mencegah kegagalan sistem saat mengintegrasikan sensor ke jaringan IoT.
