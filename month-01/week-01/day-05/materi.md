# 🟢 Day 05 — Daya Listrik (Power), Disipasi Panas, & Manajemen Energi

Setelah memahami Tegangan ($V$), Arus ($I$), dan Hambatan ($R$), kini kita menggabungkan ketiganya untuk memahami konsep kerja nyata: **Daya Listrik (Electrical Power / $P$)** dan **Energi Listrik ($E$)**.

Dalam proyek IoT, daya listrik adalah penentu utama:
1. Mengapa regulator atau transistor Anda terasa sangat panas hingga berisiko terbakar.
2. Berapa lama perangkat sensor Anda dapat menyala menggunakan baterai sebelum kehabisan daya.
3. Seberapa efisien konversi energi dari adaptor atau panel surya (*solar panel*) ke ESP32.

---

## 1. Analisis Masalah

### Pertanyaan Utama Hari Ini:
> **Jika Anda memberi makan pin VIN ESP32 dengan tegangan 12V dari aki motor, mengapa chip regulator kecil (AMS1117) di board ESP32 menjadi luar biasa panas dalam hitungan detik, padahal ESP32 hanya butuh daya sekitar 0.5 Watt? Ke mana perginya sisa energi tersebut?**

Banyak orang berpikir:
> *"Kan batas VIN tertulis bisa sampai 12V, berarti aman-aman saja dong."*

Secara teori voltase, regulator linear memang mampu menahan tegangan hingga 12V-15V. Namun secara teori **Daya dan Panas (Thermal Dissipation)**, regulator tersebut dipaksa membakar sisa tegangan menjadi panas murni. Memahami daya listrik akan menyelamatkan proyek IoT Anda dari mati suri akibat panas berlebih (*thermal shutdown*).

---

## 2. Teori Dasar: Apa Itu Daya Listrik?

### Definisi Fisika
**Daya Listrik (Electric Power / Simbol: $P$)** adalah laju energi listrik yang diserap atau dihasilkan dalam suatu rangkaian per satuan waktu:
$$P = \frac{E}{t}$$

Satuan daya listrik adalah **Watt (W)** atau **miliwatt (mW)** ($1\text{ W} = 1.000\text{ mW}$).

### 📐 Rumus Dasar Daya Listrik (Joule's Law)
Daya dihitung dari perkalian tegangan dan arus:
$$P = V \times I$$

Dengan mensubstitusikan Hukum Ohm ($V = I \times R$ dan $I = \frac{V}{R}$), kita mendapatkan rumus turunan yang sangat berguna:
$$P = I^2 \times R$$
$$P = \frac{V^2}{R}$$

```text
              /   P   \          P = V × I
             /─────────\         V = P / I
            /  V  │  I  \        I = P / V
           ───────────────
```

- **Jika Anda tahu Tegangan & Arus:** Gunakan $P = V \times I$
- **Jika Anda tahu Arus & Hambatan:** Gunakan $P = I^2 \times R$ (rumus disipasi panas pada resistor/kabel)
- **Jika Anda tahu Tegangan & Hambatan:** Gunakan $P = \frac{V^2}{R}$

---

## 3. Energi Listrik vs Daya Listrik

Sering kali orang tertukar antara **Daya (Power)** dan **Energi (Energy)**:
- **Daya ($P$):** Kecepatan pemakaian listrik saat ini (Satuan: Watt). Contoh: ESP32 sedang menarik daya $0.66\text{ Watt}$ saat Wi-Fi aktif.
- **Energi ($E$):** Total akumulasi daya yang dihabiskan sepanjang waktu (Satuan: **Watt-hour / Wh** atau **Joule**):
  $$E = P \times t$$

### Memahami Kapasitas Baterai: mAh vs Wh
Baterai smartphone atau baterai Li-ion 18650 biasanya mencantumkan kapasitas dalam **miliampere-hour (mAh)**:
- Sebuah baterai 18650 tertulis: **$2.600\text{ mAh (3.7V)}$**.
- Artinya baterai tersebut mampu mengalirkan arus $2.600\text{ mA}$ selama 1 jam, atau $260\text{ mA}$ selama 10 jam.

Untuk mengetahui **Energi Total Aktual** dalam Watt-hour (Wh):
$$\text{Energi (Wh)} = \frac{\text{Kapasitas (mAh)} \times \text{Tegangan (V)}}{1.000}$$
$$\text{Energi 18650} = \frac{2600\text{ mAh} \times 3.7\text{V}}{1.000} \approx \mathbf{9.62\text{ Watt-hour}}$$

> **💡 Mengapa Powerbank 10.000 mAh cepat habis?**
> Powerbank memiliki baterai internal 3.7V ($10.000\text{mAh} \times 3.7\text{V} = 37\text{Wh}$). Ketika dinaikkan (*step-up*) ke tegangan output USB 5V, kapasitas efektif arusnya berkurang menjadi: $\frac{37\text{Wh}}{5\text{V}} = 7.400\text{ mAh}$ (belum dipotong rugi-rugi efisiensi konverter sekitar 15%).

---

## 4. Bedah Kasus: Mengapa Regulator Panas? (Linear vs Switching)

Mari kita jawab teka-teki regulator ESP32:

```text
Kasus: ESP32 disuplai 12V melalui pin VIN saat Wi-Fi menyala (Arus I = 0.2A / 200mA).
Tegangan inti chip ESP32 yang dibutuhkan = 3.3V.

Regulator Onboard (AMS1117-3.3) adalah LINEAR REGULATOR (LDO).
Prinsip kerja Linear Regulator:
Ia menurunkan tegangan dengan cara "membuang" kelebihan tegangan menjadi PANAS murni!

Tegangan yang harus dibuang (V_drop):
V_drop = V_in - V_out = 12V - 3.3V = 8.7 Volt

Daya Panas yang terbuang sia-sia (P_panas):
P_panas = V_drop × I = 8.7V × 0.2A = 1.74 Watt!
```

Daya panas sebesar $1.74\text{ Watt}$ pada kemasan IC sekecil $6\text{mm}$ tanpa lempengan pendingin (*heatsink*) akan menaikkan suhu IC hingga **$>110^\circ\text{C}$ dalam hitungan detik**! Akibatnya, sensor proteksi termal regulator akan memutus aliran daya dan ESP32 mati suri.

### Solusi Modern IoT: Buck Converter (Switching Regulator)
Untuk menurunkan tegangan dari sumber tinggi (misal aki 12V atau panel surya) ke 5V/3.3V, jangan gunakan linear regulator! Gunakan modul **DC-DC Step Down Buck Converter** (seperti chip MP1584, LM2596, atau TPS54302).

```text
┌─────────────────────────────────┬─────────────────────────────────┐
│     Linear Regulator (LDO)      │   Switching (Buck Converter)    │
├─────────────────────────────────┼─────────────────────────────────┤
│ • Menurunkan tegangan dengan    │ • Menurunkan tegangan dengan    │
│   mengubah selisih jadi PANAS.  │   sakelar pulsa frekuensi tinggi│
│ • Efisiensi sangat rendah jika  │ • Efisiensi sangat tinggi       │
│   selisih voltase besar (30-40%)│   (mencapai 90% - 95%).         │
│ • Sangat panas.                 │ • Sangat dingin dan hemat daya. │
│ • Cocok hanya jika selisih kecil│ • Pilihan wajib untuk baterai   │
│   (misal 5V turun ke 3.3V).     │   dan sistem bertenaga surya.   │
└─────────────────────────────────┴─────────────────────────────────┘
```

---

## 5. Rating Daya Resistor ($1/8\text{W}, 1/4\text{W}, 1/2\text{W}$)

Resistor juga memiliki batas daya maksimum yang boleh diserapnya. Resistor kecil berwarna krem/biru yang sering kita pakai di breadboard umumnya memiliki rating **$1/4\text{ Watt}$ ($0.25\text{ W}$)**.

```text
Uji Keamanan Resistor LED (Day 04):
- Arus yang mengalir: I = 6.8 mA = 0.0068 A
- Nilai resistor: R = 220 Ω
- Tegangan pada resistor: VR = 1.5 V

Hitung daya panas yang dibuang resistor (P):
P = I² × R = (0.0068 A)² × 220 Ω = 0.0000462 × 220 ≈ 0.010 Watt (10 mW)

Bandingkan dengan rating resistor:
Daya disipasi (10 mW) « Batas aman resistor 1/4W (250 mW).
Kesimpulan: Resistor bekerja sangat dingin dan 100% aman!
```

---

## 6. Praktik Manajemen Energi: Perhitungan Umur Baterai IoT

Bayangkan Anda membuat **Node Sensor Pertanian IoT** (ESP32 + Capacitive Soil Moisture Sensor) ditenagai 1 buah baterai 18650 ($2.600\text{ mAh}$).

### Skenario Operasi Siklus Tugas (Duty Cycle):
Perangkat diatur agar bangun setiap **10 menit (600 detik)**:
1. **Fase Bangun (Active):** Membaca sensor tanah, menyalakan Wi-Fi, mengirim paket MQTT ke dashboard web.
   - Durasi: **$5\text{ detik}$**.
   - Konsumsi Arus: **$150\text{ mA}$**.
2. **Fase Tidur Lelap (Deep Sleep):** Mematikan CPU, RAM, dan modul radio Wi-Fi.
   - Durasi: **$595\text{ detik}$**.
   - Konsumsi Arus: **$0.05\text{ mA}$ ($50\ \mu\text{A}$)**.

### Perhitungan Arus Rata-Rata ($I_{avg}$):
$$I_{avg} = \frac{(I_{active} \times t_{active}) + (I_{sleep} \times t_{sleep})}{t_{total}}$$
$$I_{avg} = \frac{(150\text{ mA} \times 5\text{ s}) + (0.05\text{ mA} \times 595\text{ s})}{600\text{ s}} = \frac{750 + 29.75}{600} = \frac{779.75}{600} \approx \mathbf{1.30\text{ mA}}$$

### Estimasi Umur Baterai:
$$\text{Waktu Hidup (Jam)} = \frac{\text{Kapasitas}}{\text{Arus Rata-rata}} = \frac{2600\text{ mAh}}{1.30\text{ mA}} \approx 2.000\text{ Jam}$$
$$\text{Waktu Hidup (Hari)} = \frac{2000\text{ Jam}}{24\text{ Jam/Hari}} \approx \mathbf{83.3\text{ Hari (Hampir 3 Bulan Tanpa Cas!)}}$$

Bandingkan jika ESP32 **dibiarkan menyala terus tanpa Deep Sleep**:
$$\text{Waktu Hidup} = \frac{2600\text{ mAh}}{150\text{ mA}} \approx \mathbf{17.3\text{ Jam (Kurang dari 1 hari langsung habis!)}}$$

---

## 7. Praktikum Day-05: Menghitung Konsumsi Daya Riil Sistem

Lakukan audit daya untuk sistem IoT yang ingin Anda bangun menggunakan rumus $P = V \times I$:

| Komponen | Tegangan Kerja ($V$) | Arus Puncak ($I$) | Daya Maksimal ($P = V \times I$) |
|---|---|---|---|
| ESP32 DevKit (Wi-Fi TX) | 5.0 V | 0.240 A (240 mA) | $5.0 \times 0.24 = \mathbf{1.20\text{ W}}$ |
| Sensor DHT22 | 3.3 V | 0.0015 A (1.5 mA) | $3.3 \times 0.0015 = \mathbf{0.005\text{ W}}$ |
| Layar LCD I2C 16x2 | 5.0 V | 0.025 A (25 mA) | $5.0 \times 0.025 = \mathbf{0.125\text{ W}}$ |
| Modul Relay 5V Aktif | 5.0 V | 0.070 A (70 mA) | $5.0 \times 0.070 = \mathbf{0.350\text{ W}}$ |
| **Total Daya Sistem** | — | — | **$\mathbf{1.68\text{ Watt}}$** |

---

## 8. 🧠 Tantangan Berpikir Analitis

1. **Efisiensi Daya:** Jika sebuah sistem IoT menarik daya $1.68\text{ Watt}$ dari catu daya 5V, berapakah arus total yang ditarik dari sumber ($I = \frac{P}{V}$)?
2. **Kapasitas Nyata Powerbank:** Mengapa powerbank berlabel 20.000 mAh pada kenyataannya tidak bisa mengisi baterai HP 5.000 mAh sebanyak 4 kali penuh? Sebutkan minimal 2 faktor kehilangan energi secara fisik!
3. **Penyebab Kebakaran Komponen:** Mengapa sambungan kabel yang kendur (*loose contact*) pada sirkuit arus tinggi (misal kabel aki motor ke driver L298N) dapat melelehkan plastik soket breadboard atau konektor terminal? (Petunjuk: Pikirkan hubungan resistansi kontak mikro dan rumus $P = I^2 \times R$).

---

## 9. 📚 Panduan Dokumentasi GitHub

Buat file dokumentasi di:
`month-01/week-01/day-05/README.md`

Format standar `README.md`:
```markdown
# Day 05 — Electrical Power, Thermal Dissipation & Energy Management

## 🎯 Objective
Memahami konsep Daya Listrik ($P = V \times I$), disipasi panas pada regulator dan resistor, perbedaan Linear Regulator vs Buck Converter, serta teknik estimasi ketahanan baterai sistem IoT berbasis Deep Sleep.

## 📚 What I Learned
- Definisi Daya Listrik ($P$) dalam Watt dan Energi Listrik ($E$) dalam Watt-hour (Wh).
- Tiga rumus utama daya: $P = V \times I$, $P = I^2 \times R$, dan $P = \frac{V^2}{R}$.
- Penyebab panas berlebih pada regulator linear AMS1117 akibat selisih tegangan tinggi ($V_{drop}$).
- Keunggulan efisiensi Switching Buck Converter untuk sistem bertenaga baterai/aki.
- Strategi memperpanjang umur baterai menggunakan mode *Deep Sleep* dan perhitungan *Duty Cycle*.

## 📊 Energy & Power Calculation
[Sertakan tabel perhitungan daya sistem dan estimasi umur baterai]

## 🧠 Questions & Answers
[Jawab ketiga pertanyaan tantangan berpikir secara mandiri]

## 📝 Conclusion
[Tuliskan kesimpulan pemahaman Anda hari ini]
```

---

## 10. Git Workflow Day-05

```bash
git status
git add month-01/week-01/day-05/
git commit -m "docs(day-05): complete electrical power theory, heat dissipation, and battery life estimation"
git push origin main
```

---

## ✅ Checklist Kelulusan Day-05
- [ ] Memahami rumus dasar daya listrik $P = V \times I$ dan $P = I^2 \times R$.
- [ ] Memahami perbedaan kapasitas mAh dan energi total Wh pada baterai.
- [ ] Mengetahui penyebab regulator linear sangat panas saat diberi tegangan tinggi.
- [ ] Mampu membedakan kapan harus menggunakan LDO vs Buck Converter.
- [ ] Mampu menghitung disipasi daya resistor untuk memastikan tidak melebihi rating 1/4W.
- [ ] Mampu menghitung estimasi masa pakai baterai berdasarkan siklus duty cycle Deep Sleep.
- [ ] Menyelesaikan tantangan berpikir dan mendokumentasikan ke GitHub.

---

## 🎯 Prinsip Esensial Hari Ini
> **"Daya adalah laju energi yang dikonsumsi, dan energi yang terbuang selalu berubah menjadi panas. Desain IoT yang unggul bukan hanya tentang fitur canggih, melainkan efisiensi pemanfaatan setiap miliwatt energi."**
