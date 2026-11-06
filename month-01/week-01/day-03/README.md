# Day 03 — Current, Load & ESP32 Current Limits

## 🎯 Objective
Memahami konsep arus listrik (Current / $I$), interaksi antara rating kapasitas catu daya dengan kebutuhan arus beban, batasan aman pin GPIO ESP32, bahaya korsleting (*short circuit*), serta teknik penyusunan *Power Budgeting* untuk perangkat IoT.

## 📚 What I Learned
- **Konsep Arus Listrik ($I$):** Arus adalah laju perpindahan muatan elektron per satuan waktu ($I = \frac{Q}{t}$) dengan satuan Ampere ($A$) atau miliampere ($mA$).
- **Aturan Emas Kelistrikan Beban:**
  - Tegangan disuplai secara konstan oleh sumber daya (*Power Supply*).
  - Arus ditarik oleh beban (*Load*) sesuai resistansi dan kebutuhan operasionalnya.
  - Adaptor dengan kapasitas arus lebih besar (misal: 5V 3A) tidak akan merusak ESP32 yang hanya butuh 100mA, karena ESP32 hanya akan menarik arus yang diperlukannya saja.
- **Batasan Arus ESP32:**
  - Arus maksimum per pin GPIO: **40 mA** (batas aman operasional: **$\le 12 - 20\text{ mA}$**).
  - Total batas arus seluruh chip: **$\sim 120\text{ mA}$**.
  - Mengapa LED wajib dipasangi resistor pembatas arus ($220\Omega - 330\Omega$): tanpa resistor, LED menarik arus liar melampaui batas aman GPIO dan merusak pin internal ESP32.
- **Konsumsi Daya Dinamis ESP32:**
  - **Deep Sleep:** $10 - 150\ \mu\text{A}$ (sangat krusial untuk proyek IoT mandiri berbasis baterai).
  - **Active CPU:** $\sim 50 - 68\text{ mA}$.
  - **Wi-Fi Transmit Burst:** $\sim 160 - 260\text{ mA}$ (pemicu utama *brownout* jika adaptor/kabel USB jelek).
- **Alasan Pemisahan Catu Daya Aktuator (External Power Supply):**
  - Aktuator seperti Servo SG90 (stall hingga 800mA) dan Motor DC L298N (hingga 2A) menarik arus lonjakan (*inrush current*) yang jauh melampaui kapasitas pin regulator 3.3V ESP32.
  - Aktuator wajib ditenagai sumber eksternal terpisah dengan **Common Ground (GND disatukan)** ke ESP32.
- **Pengukuran Arus:** Pengukuran arus dengan Multimeter Digital **WAJIB DILAKUKAN SECARA SERI** (memutus rangkaian), berbeda dengan pengukuran tegangan yang dilakukan secara paralel.

## 🧮 Power Budget Calculation

Perancangan kebutuhan daya untuk sistem IoT mandiri (Stasiun Monitoring Cuaca & Kendali):

| Komponen | Tegangan Operasi | Konsumsi Arus Tipikal / Puncak | Keterangan |
|---|---|---|---|
| ESP32 DevKit V1 | 5.0 V (VIN) | 200 mA (Peak TX Wi-Fi) | Otak pemrosesan & pengirim data |
| Sensor DHT22 | 3.3 V | 1.5 mA | Sensor suhu & kelembapan |
| Layar LCD I2C 16x2 | 5.0 V | 25 mA | Tampilan lokal (backlight aktif) |
| Modul Relay 5V | 5.0 V | 70 mA | Koil elektromagnetik aktif |
| **Total Konsumsi Arus** | — | **296.5 mA** | Kondisi seluruh komponen aktif bersamaan |
| **Safety Margin (+30%)** | — | **~385 mA** | Toleransi lonjakan sesaat |

> **Rekomendasi Catu Daya:** Adaptor DC 5V dengan rating minimal **1 Ampere (1000 mA)** untuk memastikan tegangan tidak drop saat transmisi Wi-Fi dan aktuator relay aktif bersamaan.

## 🧠 Questions & Answers
1. **Apakah Adaptor 5V 1A dan Adaptor 5V 3A sama-sama aman untuk ESP32? Apa dampaknya?**
   *Keduanya sama-sama aman.* Karena kedua adaptor menyediakan tegangan yang sama (5V), ESP32 hanya akan menarik arus yang dibutuhkannya saja (misal ~100-250mA). Perbedaannya, adaptor 5V 3A memiliki cadangan daya (*headroom*) yang jauh lebih besar sehingga bekerja lebih dingin, tidak mudah panas, dan lebih tahan terhadap lonjakan beban tiba-tiba saat Wi-Fi memancar.
2. **Mengapa korsleting (short circuit) antara pin 5V dan GND menyebabkan percikan api dan kerusakan port USB laptop?**
   Berdasarkan Hukum Ohm:
   $$I = \frac{V}{R}$$
   Ketika kabel menghubungkan 5V langsung ke GND tanpa beban, nilai resistansi sirkuit mendekati nol ($R \approx 0\Omega$). Akibatnya, arus yang berusaha ditarik melonjak mendekati tak hingga ($I \to \infty$). Lonjakan arus masif sesaat ini menghasilkan panas ekstrem secara instan ($P = I^2 \times R$), memicu percikan api, dan mengaktifkan proteksi *overcurrent trip* pada port USB laptop.
3. **Berapa lama baterai 18650 kapasitas 2600 mAh dapat menghidupi ESP32 jika konsumsi arus rata-rata 0.5 mA (Deep Sleep)?**
   Menggunakan rumus ketahanan baterai teoritis:
   $$T = \frac{\text{Kapasitas}}{\text{Konsumsi Arus}} = \frac{2600\text{ mAh}}{0.5\text{ mA}} = 5.200\text{ Jam}$$
   Jika dikonversi ke hari:
   $$\frac{5.200\text{ Jam}}{24\text{ Jam/Hari}} \approx 216.6\text{ Hari (lebih dari 7 Bulan!)}$$
   Hal ini membuktikan betapa krusialnya pemahaman profil arus dan fitur *Deep Sleep* dalam perancangan produk IoT bertenaga baterai di dunia nyata.

## 📝 Conclusion
Arus listrik adalah representasi konsumsi energi riil sebuah sistem. Memahami limitasi arus GPIO dan karakteristik beban aktuator adalah pembeda antara rangkaian yang rapuh dan mudah rusak dengan sistem IoT standar industri yang handal bekerja 24 jam sehari tanpa gangguan.
