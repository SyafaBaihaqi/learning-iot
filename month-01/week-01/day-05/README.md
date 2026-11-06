# Day 05 — Electrical Power, Thermal Dissipation & Energy Management

## 🎯 Objective
Memahami konsep Daya Listrik ($P$), laju disipasi energi panas ($P = I^2 \times R$), perbedaan mendasar Linear LDO vs Switching Buck Converter, serta teknik estimasi umur baterai untuk perangkat IoT.

## 📚 What I Learned
- **Rumus Dasar Daya Listrik (Joule's Law):**
  $$P = V \times I \quad \Longleftrightarrow \quad P = I^2 \times R \quad \Longleftrightarrow \quad P = \frac{V^2}{R}$$
- **Daya vs Energi:**
  - Daya ($P$) adalah laju konsumsi instan (Satuan: Watt).
  - Energi ($E$) adalah total akumulasi daya terhadap waktu ($E = P \times t$, Satuan: Watt-hour / Wh).
  - Kapasitas energi baterai dihitung dengan: $\text{Wh} = \frac{\text{mAh} \times \text{Volt}}{1000}$.
- **Penyebab Panas Linear Regulator (AMS1117-3.3):**
  Linear regulator membuang selisih tegangan ($V_{in} - V_{out}$) menjadi panas murni. Memasok tegangan 12V ke VIN ESP32 dengan konsumsi 200mA menghasilkan disipasi panas sia-sia sebesar:
  $$P_{panas} = (12\text{V} - 3.3\text{V}) \times 0.2\text{A} = \mathbf{1.74\text{ Watt}}$$
  Panas sebesar ini tanpa heatsink akan memicu *thermal shutdown*.
- **Solusi Switching Buck Converter:** Menggunakan teknologi switching frekuensi tinggi untuk memangkas rugi-rugi daya dengan efisiensi mencapai $90\% - 95\%$, sangat krusial untuk proyek IoT bertenaga aki/solar panel.
- **Strategi Duty Cycle & Deep Sleep:**
  Dengan menerapkan siklus bangun 5 detik ($150\text{ mA}$) dan tidur 595 detik ($0.05\text{ mA}$), konsumsi rata-rata sistem turun drastis ke $\mathbf{1.30\text{ mA}}$, memperpanjang usia baterai 18650 dari 17 jam menjadi **hampir 3 bulan**.

## 📊 Energy & Power Calculation

### 1. Power Audit Sistem Cuaca IoT
| Komponen | Tegangan Kerja | Arus Operasi | Daya Diserap ($P$) |
|---|---|---|---|
| ESP32 DevKit (Wi-Fi TX Peak) | 5.0 V | 240 mA | 1.200 W |
| Sensor Suhu & Kelembapan DHT22 | 3.3 V | 1.5 mA | 0.005 W |
| Modul Display LCD I2C 16x2 | 5.0 V | 25 mA | 0.125 W |
| Modul Relay 5V (Koil Terbuka) | 5.0 V | 70 mA | 0.350 W |
| **Total Daya Puncak Sistem** | — | — | **1.680 Watt** |

### 2. Estimasi Ketahanan Baterai Li-ion 18650 ($2600\text{ mAh}$)
- **Mode Always ON (Aktif terus menerus @ 150mA):** $\approx 17.3\text{ Jam}$
- **Mode Deep Sleep (Bangun tiap 10 menit @ rerata 1.3mA):** $\approx 2000\text{ Jam}$ ($\approx \mathbf{83.3\text{ Hari}}$)

## 🧠 Questions & Answers
1. **Berapa arus total yang ditarik jika sistem IoT berdaya 1.68 Watt disuplai adaptor 5V?**
   Menggunakan rumus $I = \frac{P}{V}$:
   $$I = \frac{1.68\text{ W}}{5.0\text{ V}} = \mathbf{0.336\text{ A}} = \mathbf{336\text{ mA}}$$
2. **Mengapa powerbank 20.000 mAh tidak bisa mengisi penuh 4 kali HP baterai 5.000 mAh?**
   - **Perbedaan Tegangan Sel vs Output:** Label 20.000 mAh mengacu pada sel baterai internal 3.7V ($74\text{ Wh}$). Saat dinaikkan ke 5V USB, kapasitas arus efektifnya turun menjadi $\frac{74\text{ Wh}}{5\text{ V}} = 14.800\text{ mAh}$.
   - **Rugi-Rugi Efisiensi (Conversion Loss):** Sirkuit konverter step-up dan panas kabel memakan rugi efisiensi sekitar $10\% - 20\%$.
   - **Konversi Kedua di HP:** Chip charger HP menurunkan kembali 5V USB ke 3.7V - 4.4V baterai HP dengan efisiensi tambahan.
3. **Mengapa sambungan kabel longgar pada sirkuit arus tinggi dapat melelehkan soket plastik?**
   Sambungan longgar menciptakan titik sentuh mikro yang sangat kecil, sehingga menimbulkan resistansi kontak ($R_{kontak}$) yang tidak terduga (misal $1\ \Omega$). Jika arus motor besar ($I = 2\text{ A}$) melewati titik resistansi tersebut:
   $$P = I^2 \times R = (2\text{ A})^2 \times 1\ \Omega = \mathbf{4\text{ Watt panas terlokalisasi!}}$$
   Daya panas 4 Watt yang terkonsentrasi pada luas permukaan milimeter seketika melelehkan plastik konektor.

## 📝 Conclusion
Efisiensi daya dan manajemen termal adalah faktor utama yang membedakan prototipe meja praktikum dari produk IoT yang siap dipasang secara permanen di lapangan. Menggunakan converter yang tepat dan memanfaatkan fitur Deep Sleep adalah kunci sistem IoT mandiri energi.
