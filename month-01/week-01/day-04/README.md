# Day 04 — Resistance, Ohm's Law & Circuit Protection

## 🎯 Objective
Memahami konsep hambatan listrik (Resistance / $R$), menerapkan Hukum Ohm ($V = I \times R$) untuk memproteksi pin GPIO ESP32, membaca kode warna resistor, serta memahami peran resistor Pull-Up pada sistem IoT.

## 📚 What I Learned
- **Hukum Ohm:** Hubungan fundamental antara tegangan ($V$), arus ($I$), dan hambatan ($R$):
  $$V = I \times R \quad \Longleftrightarrow \quad I = \frac{V}{R} \quad \Longleftrightarrow \quad R = \frac{V}{I}$$
- **Perhitungan Resistor Pembatas Arus LED:**
  Untuk menyalakan LED Merah ($V_f \approx 1.8\text{V}$) pada pin GPIO ESP32 ($3.3\text{V}$) dengan arus aman $6.8\text{ mA}$:
  $$R = \frac{V_{GPIO} - V_{f}}{I} = \frac{3.3\text{V} - 1.8\text{V}}{0.0068\text{A}} \approx 220\ \Omega$$
- **Pembacaan Kode Warna Resistor (4 Gelang):**
  - **Gelang 1 & 2:** Digit nilai signifikan.
  - **Gelang 3:** Faktor pengali ($10^n$).
  - **Gelang 4:** Toleransi (Emas = $\pm 5\%$, Cokelat = $\pm 1\%$).
- **Perilaku Rangkaian Resistor:**
  - **Seri:** Hambatan total bertambah besar ($R_{tot} = R_1 + R_2$), arus sama di semua titik.
  - **Paralel:** Hambatan total mengecil ($\frac{1}{R_{tot}} = \frac{1}{R_1} + \frac{1}{R_2}$), tegangan sama di kedua cabang.
- **Aplikasi Kritis Resistor di Perangkat IoT:**
  - **Pull-Up I2C ($4.7\text{k}\Omega - 10\text{k}\Omega$):** Menjaga jalur komunikasi SDA dan SCL pada level HIGH saat bus dalam kondisi *idle*.
  - **Pull-Up DHT22 ($4.7\text{k}\Omega - 10\text{k}\Omega$):** Menghindari pin data *floating* saat transfer data bit serial tunggal.
- **Prosedur Pengukuran Ohmmeter:** Selalu mengukur resistansi dalam kondisi sirkuit **mati total (unpowered)** untuk melindungi multimeter dan mendapatkan nilai akurat.

## 🧮 Calculations & Lab Results

### 1. Pembuktian Perhitungan Resistor LED
| Parameter | Nilai Teori | Keterangan |
|---|---|---|
| Tegangan Pin GPIO ESP32 ($V_{in}$) | 3.30 V | Logika HIGH |
| Tegangan Jatuh LED Merah ($V_f$) | 1.80 V | Forward drop |
| Tegangan pada Resistor ($V_R$) | 1.50 V | $3.3\text{V} - 1.8\text{V}$ |
| Nilai Resistor Standar ($R$) | $220\ \Omega$ | Nilai terpasang |
| Arus Hasil ($I$) | **6.81 mA** | Sangat aman untuk batas GPIO (maks 40mA) |

### 2. Pengukuran Resistor Riil dengan Multimeter
| Resistor | Kode Warna | Nilai Teori | Hasil Ukur Aktual | Toleransi Nyata |
|---|---|---|---|---|
| $220\ \Omega$ | Merah - Merah - Cokelat - Emas | $220\ \Omega \pm 5\%$ | $218.4\ \Omega$ | $-0.7\%$ (Sangat baik) |
| $1\text{ k}\Omega$ | Cokelat - Hitam - Merah - Emas | $1000\ \Omega \pm 5\%$ | $991\ \Omega$ | $-0.9\%$ |
| $10\text{ k}\Omega$ | Cokelat - Hitam - Oranye - Emas | $10000\ \Omega \pm 5\%$ | $9.88\text{ k}\Omega$ | $-1.2\%$ |

## 🧠 Questions & Answers
1. **Berapa nilai resistor dengan gelang: Cokelat - Hitam - Merah - Emas beserta rentang toleransinya?**
   - Cokelat (1), Hitam (0), Merah ($\times 100$), Emas ($\pm 5\%$).
   - Nilai nominal: $10 \times 100 = \mathbf{1.000\ \Omega} = \mathbf{1\text{ k}\Omega}$.
   - Toleransi $5\%$ dari $1.000\Omega$ adalah $\pm 50\Omega$.
   - Rentang nilai wajar: **$950\ \Omega$ hingga $1.050\ \Omega$**.
2. **Bagaimana menghasilkan nilai $500\ \Omega$ jika hanya memiliki resistor $1\text{ k}\Omega$?**
   Hubungkan **dua buah resistor $1\text{ k}\Omega$ secara PARALEL**:
   $$R_{total} = \frac{R_1 \times R_2}{R_1 + R_2} = \frac{1000 \times 1000}{1000 + 1000} = \frac{1.000.000}{2.000} = \mathbf{500\ \Omega}$$
3. **Mengapa pin input tombol yang tidak diberi resistor pull-up/pull-down (floating pin) bisa berosilasi acak?**
   Pin input CMOS mikrokontroler memiliki impedansi sangat tinggi ($>10\text{ M}\Omega$). Jika pin tidak diikat ke referensi VCC maupun GND, pin tersebut akan bertindak sebagai antena kecil yang menangkap interferensi elektromagnetik (EMI) dari udara atau listrik statis tubuh manusia, menyebabkan logika berfluktuasi acak antara 0 dan 1.

## 📝 Conclusion
Resistansi adalah mekanisme kontrol utama yang memungkinkan kita mendikte seberapa besar arus listrik yang boleh mengalir di setiap percabangan sistem IoT. Menguasai Hukum Ohm adalah syarat mutlak sebelum melangkah ke perancangan skema rangkaian sensor dan aktuator yang lebih kompleks.
