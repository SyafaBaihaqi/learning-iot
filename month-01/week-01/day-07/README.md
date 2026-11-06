# Day 07 — Week 01 Comprehensive Review & System Blueprint

## 🎯 Objective
Melakukan evaluasi menyeluruh terhadap fondasi kelistrikan, logika level ESP32, batasan arus, disipasi daya, sinyal digital/analog, serta menghasilkan cetak biru sistem IoT terpadu (*System Blueprint*) yang teruji secara engineering.

## 📚 What I Synthesized in Week 01

### 1. Fondasi Kelistrikan & Rumus Utama
- **Hukum Ohm:** $V = I \times R$, dasar penentuan arus aman dan proteksi komponen.
- **Hukum Joule (Daya Listrik):** $P = V \times I = I^2 \times R$, dasar perhitungan konsumsi baterai dan pelepasan energi panas.
- **Toleransi Logika:** ESP32 beroperasi pada level logika **3.3V dan TIDAK 5V tolerant** (maksimal 3.6V).
- **Aturan Beban:** Tegangan disuplai konstan oleh sumber, sedangkan arus ditarik oleh beban. Beban besar (motor/servo/relay) wajib menggunakan catu daya eksternal dengan **Common Ground**.

### 2. Peta Keselamatan Pinout ESP32
- **Pin ADC1 (GPIO 32 - 39):** Pilihan wajib untuk sensor analog agar tidak bentrok dengan modul Wi-Fi.
- **Pin Strapping (GPIO 0, 2, 12, 15):** Dihindari untuk aktuator/tombol agar ESP32 tidak gagal boot.
- **Pin Flash (GPIO 6 - 11):** Terlarang disentuh karena terhubung ke memori internal SPI Flash.

### 3. Matriks Diagnosa Masalah (Troubleshooting Matrix)
Mampu mendiagnosa secara mandiri gejala kerusakan khas IoT: *Brownout Reset*, *Thermal Overheating*, *ADC Reading Glitches*, dan *I2C Bus Hang*.

## 📐 System Blueprint — Smart Greenhouse Node
Cetak biru arsitektur hardware lengkap terdokumentasi di:
📄 [`system-blueprint.md`](./system-blueprint.md)

- **Sensor:** DHT22 (Digital @ GPIO4) & Capacitive Soil Moisture (Analog @ GPIO34 ADC1).
- **Aktuator & Display:** Modul Relay 5V (@ GPIO18), LCD I2C 16x2 (@ GPIO21 & GPIO22), 2x LED Indikator (@ GPIO19 & GPIO23).
- **Power Budget:** Total konsumsi puncak sistem adalah **$355.5\text{ mA}$ pada 5V ($1.78\text{ W}$)**. Disuplai menggunakan adaptor 5V 2A dengan cadangan daya bebas hingga $82\%$.

## 🧠 Comprehensive Exam Answers
1. **Disipasi Panas Sensor Gas (LM7805 pada Aki 12V @ 150mA):**
   - $V_{drop} = 12\text{V} - 5\text{V} = 7\text{ Volt}$.
   - Disipasi daya panas:
     $$P_{panas} = V_{drop} \times I = 7\text{V} \times 0.15\text{A} = \mathbf{1.05\text{ Watt}}$$
   - Nilai $1.05\text{W}$ jauh melampaui batas aman pendinginan udara bebas LM7805 ($0.5\text{W}$), sehingga regulator akan mengalami *overheating* dan mati proteksi termal.
   - **Rekomendasi Rekayasa:** Mengganti linear regulator LM7805 dengan modul switching **DC-DC Step-Down Buck Converter (misal LM2596 atau MP1584)** yang memiliki efisiensi $>90\%$ dan bekerja sangat dingin.
2. **Mengapa Relay pada GPIO12 atau GPIO0 dapat Menggagalkan Booting ESP32?**
   GPIO0 dan GPIO12 adalah **Strapping Pins**. Saat ESP32 pertama kali dinyalakan, pin-pin ini dibaca oleh sirkuit bootloader internal:
   - Jika GPIO0 ditarik ke LOW saat power-on, ESP32 masuk ke *UART Download Mode* alih-alih menjalankan program pengguna.
   - Jika GPIO12 ditarik ke level tertentu, voltase chip memori flash berubah dari 3.3V ke 1.8V, membuat chip flash gagal terbaca.
   Resistansi koil modul relay yang terpasang di pin ini dapat secara tidak sengaja menarik tegangan pin ke LOW/HIGH saat booting, menggagalkan proses startup sistem.
3. **Perbedaan Pengolahan Firmware: Pin DO vs Pin AO pada TCRT5000:**
   - **Pin DO (Digital Output):** Menghasilkan logika biner 0 atau 1 berdasarkan pembanding (*comparator op-amp LM393*) dengan ambang batas yang diatur potensiometer manual di modul. Cocok untuk aplikasi sederhana seperti *obstacle avoidance* atau deteksi garis cepat tanpa perlu komputasi kalibrasi di ESP32.
   - **Pin AO (Analog Output):** Menghasilkan voltase analog kontinu ($0\text{V} - 3.3\text{V}$) yang proporsional dengan jumlah pantulan cahaya inframerah. Wajib digunakan jika sistem ingin mengukur gradasi kepekatan warna garis atau mendeteksi variasi jarak secara presisi menggunakan `analogRead()` 12-bit.

## 📝 Milestone Retrospective
Minggu 01 telah mengubah paradigma belajar dari sekadar "merangkai coba-coba" menjadi perancangan rekayasa berbasis data dan hukum fisika kelistrikan. Dengan pemahaman mendalam tentang voltase, arus, hukum ohm, daya, dan sinyal analog/digital, fondasi untuk memasuki materi dasar elektronika praktis di Minggu 02 (Breadboard, Resistor, LED, Pull-up/down, Multimeter, Button) kini telah berdiri kokoh!
