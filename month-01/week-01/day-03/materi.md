# 🟢 Day 03 — Arus Listrik (Current), Beban, & Batasan GPIO

Kemarin kita sudah memahami bahwa **Tegangan (Voltage)** adalah "tekanan" listrik. Hari ini kita mempelajari pasangannya yang tak terpisahkan: **Arus Listrik (Current)**. 

Banyak kasus proyek IoT gagal—seperti mikrokontroler mendadak hang, servo bergetar liar tanpa tenaga, atau pin GPIO mati total—bukan karena tegangannya salah, melainkan karena perancang tidak memperhitungkan **kebutuhan dan batasan arus listrik**.

---

## 1. Analisis Masalah

### Dua Pertanyaan Paradoks Hari Ini:
> 1. **Jika charger HP bertuliskan "5V 2A (2000mA)", apakah aman dicolokkan ke ESP32 yang hanya butuh 100mA? Apakah ESP32 akan terbakar karena arusnya "terlalu besar"?**
> 2. **Sebaliknya, mengapa jika kita menghubungkan sebuah motor servo kecil SG90 langsung ke pin daya 3.3V ESP32, ESP32 justru langsung mati dan restart saat servo mulai bergerak?**

Dua pertanyaan ini sering membingungkan pemula. Kuncinya adalah memahami aturan emas kelistrikan:
> **"Tegangan didorong oleh sumber (Power Supply), tetapi Arus ditarik oleh beban (Load)."**

---

## 2. Teori Dasar: Apa Itu Arus Listrik?

### Definisi Fisika
**Arus Listrik (Current / Simbol: $I$)** adalah laju aliran muatan listrik (elektron) yang mengalir melalui suatu konduktor per satuan waktu:
$$I = \frac{Q}{t}$$

Satuan arus listrik adalah **Ampere (A)**.
Dalam elektronika mikrokontroler dan sensor, kita lebih sering menggunakan satuan yang lebih kecil:
- **1 Ampere (A)** = $1.000\text{ miliampere (mA)}$
- **1 miliampere (mA)** = $1.000\text{ mikroampere (\mu A)}$

### 🌊 Analogi Aliran Air dalam Pipa:
```text
           Tekanan Air (Tegangan / V)
                 ══════════►
     ┌───────────────────────────────────┐
     │  ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~  │ ◄── Debit Aliran Air per Detik
     │  ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~  │     = ARUS LISTRIK (I)
     └───────────────────────────────────┘
```
- **Tegangan ($V$):** Tekanan air yang mendorong.
- **Arus ($I$):** Banyaknya liter air yang benar-benar mengalir lewat pipa setiap detik.
- **Hambatan/Beban ($R$):** Sempit atau luasnya kran/pipa yang menahan aliran air.

### Rating Power Supply vs Kebutuhan Beban
Ketika sebuah adaptor tertulis: **Output: 5V 2A**:
- **5V** adalah **Tegangan Tetap**: Sumber daya wajib menghasilkan 5V konstan.
- **2A** adalah **Kapasitas Arus Maksimal**: Sumber daya *mampu menyediakan hingga* 2 Ampere jika beban memintanya. Sumber daya **tidak memaksakan** 2A masuk ke dalam perangkat.

Jika ESP32 membutuhkan $100\text{mA}$ pada $5\text{V}$, maka adaptor 2A hanya akan menyalurkan $100\text{mA}$. Sisanya ($1.900\text{mA}$) menjadi cadangan daya yang membuat adaptor bekerja dengan dingin dan santai.

---

## 3. Batasan Kritis Arus pada ESP32

Chip silikon di dalam ESP32 berukuran mikroskopis. Jalur tembaga internalnya sangat halus dan memiliki kapasitas arus yang terbatas.

```text
┌─────────────────────────────────────────────────────────────┐
│             BATASAN ARUS SPESIFIKASI ESP32                  │
├─────────────────────────────────────────────────────────────┤
│ • Arus Maksimum per Pin GPIO         : 40 mA                │
│ • Arus Rekomendasi Aman per Pin GPIO : 12 mA - 20 mA        │
│ • Arus Maksimum Total Seluruh GPIO   : ~120 mA              │
│ • Arus Maksimum Regulator Onboard 3V3: ~500 mA - 800 mA     │
└─────────────────────────────────────────────────────────────┘
```

### Mengapa LED Wajib Memakai Resistor Pembatas Arus?
LED (Light Emitting Diode) memiliki karakteristik semikonduktor dengan resistansi internal yang mendekati nol saat mulai menyala (*forward voltage drop*).

Jika Anda menghubungkan kaki LED langsung antara pin GPIO (3.3V) dan GND:
$$I = \frac{V}{R} \approx \frac{3.3\text{V} - 2.0\text{V}}{0.5\Omega} \approx 2.6\text{ Ampere} = 2.600\text{ mA}!$$
Arus sebesar $2.600\text{mA}$ akan berusaha ditarik seketika. Karena pin GPIO ESP32 hanya mampu menahan maksimal $40\text{mA}$, maka transistor sakelar internal di dalam chip ESP32 akan **terbakar dan mati permanen**, atau LED tersebut akan meledak kecil dan hangus.

Itulah mengapa kita **wajib memasang resistor pembatas arus ($220\Omega - 330\Omega$)** secara seri dengan LED untuk membatasi arus di kisaran aman:
$$I = \frac{3.3\text{V} - 2.0\text{V}}{220\Omega} \approx \frac{1.3\text{V}}{220\Omega} \approx 5.9\text{ mA (Sangat aman untuk GPIO)}$$

---

## 4. Profil Konsumsi Arus ESP32 dalam Berbagai Mode

Salah satu keunggulan terbesar ESP32 untuk IoT bertenaga baterai adalah fitur manajemen dayanya:

| Mode Operasi ESP32 | Konsumsi Arus | Fungsi / Aktivitas |
|---|---|---|
| **Deep Sleep Mode** | **$10\ \mu\text{A} - 150\ \mu\text{A}$** | CPU utama mati, hanya RTC (Real Time Clock) dan ULP coprocessor yang hidup. Daya tahan baterai bisa berbulan-bulan/tahunan. |
| **Light Sleep Mode** | **$\sim 0.8\text{ mA}$** | CPU ditangguhkan, RAM tetap menyimpan data, modul Wi-Fi mati. |
| **Active Mode (CPU 240MHz)** | **$\sim 50\text{ mA} - 68\text{ mA}$** | Menjalankan algoritma pengolahan data, membaca sensor tanpa Wi-Fi. |
| **Wi-Fi RX (Menerima Data)** | **$\sim 95\text{ mA} - 110\text{ mA}$** | Mendengarkan paket data nirkabel dari router. |
| **Wi-Fi TX Peak (Memancar Data)** | **$\sim 160\text{ mA} - 260\text{ mA}$** | Lonjakan arus tinggi sesaat ketika antena memancarkan sinyal radio ke access point. |

---

## 5. Mengapa Aktuator Membutuhkan Catu Daya Terpisah?

Mari kita bedah perangkat aktuator yang Anda miliki:

### 1. SG90 Micro Servo
- Saat diam: menarik $\sim 10\text{ mA}$.
- Saat berputar santai: menarik $\sim 100\text{ mA} - 250\text{ mA}$.
- **Saat stall (terhimpit beban / menahan posisi):** menarik hingga **$500\text{ mA} - 800\text{ mA}$**!

### 2. Motor DC + Driver L298N
- Menarik **$300\text{ mA} - 2.0\text{ A}$** tergantung beban mekanis roda dan torsi.

### ⚠️ Bahaya Fatal Menyuplai Aktuator dari Pin ESP32:
Jika Anda menghubungkan kabel daya servo atau motor ke pin `3V3` atau pin `VIN` ESP32, ketika motor mulai berputar, terjadi tarikan arus mendadak (*inrush current*). Tarikan arus ini:
1. Melampaui batas regulator onboard ESP32.
2. Menyebabkan tegangan suplai anjlok drastis (*voltage dip*).
3. Mengakibatkan ESP32 me-restart sendiri berulang-ulang (*Brownout Reset* atau freeze).

### Solusi Standar Industri IoT:
Gunakan **External Power Supply** khusus untuk motor/servo, dan satukan kabel Ground-nya (**Common Ground**) ke ESP32:
```text
           [ Baterai / Adaptor Motor 7.4V ]
                    │              │
                 (+)│              │(-) GND
                    ▼              │
             [ Driver L298N ]      │
              ▲ (Pin Sinyal IN1-4) │
              │                    │
              │ (3.3V Logic PWM)   │
              │                    │
          [ ESP32 ] ───────────────┴──── COMMON GROUND (0V)
              │
          (Ditenagai kabel USB 5V terpisah)
```

---

## 6. Sourcing vs Sinking Current

Mikrokontroler dapat mengendalikan beban luar melalui dua cara rangkaian:

```text
1. SOURCING CURRENT (GPIO sebagai Sumber Arus)
   [ GPIO (HIGH / 3.3V) ] ───► [ Resistor ] ───► [ LED (+) Anoda ] ───► [ LED (-) Katoda ] ───► [ GND (0V) ]
   (Arus KELUAR dari pin GPIO menuju Ground)

2. SINKING CURRENT (GPIO sebagai Penampung/Penarik Arus)
   [ 3.3V VCC ] ──────────────► [ LED (+) Anoda ] ───► [ LED (-) Katoda ] ───► [ Resistor ] ───► [ GPIO (LOW / 0V) ]
   (Arus MASUK dari VCC ke dalam pin GPIO)
```
- ESP32 mendukung kedua konfigurasi ini dengan aman asalkan arus dibatasi di bawah $12-20\text{mA}$.

---

## 7. Praktikum Day-03: Mengukur Arus dengan Multimeter Digital

> **🚨 PERINGATAN KESELAMATAN SANGAT PENTING:**
> Mengukur tegangan dilakukan secara **PARALEL**, tetapi mengukur arus **WAJIB DILAKUKAN SECARA SERI**.
> **JANGAN PERNAH** menempelkan probe multimeter dalam mode Arus (Amperemeter) langsung ke kutub baterai atau pin 5V dan GND! Multimeter dalam mode amperemeter memiliki resistansi internal mendekati nol ($0\Omega$). Jika dicolokkan paralel, akan terjadi **Short Circuit (Korsleting)** seketika yang dapat meledakkan sekring internal multimeter Anda!

### Diagram Pengukuran Arus secara SERI:
```text
  [ Sumber Tegangan 3.3V ]
             │
             ▼
     (Kabel Terputus)
             │
      ┌──────┴──────┐
      │ Probe MERAH │
      │  [MULTIMETER│
      │   MODE mA]  │
      │ Probe HITAM │
      └──────┬──────┘
             │
             ▼
      [ Resistor 220Ω ]
             │
             ▼
      [ Anoda (+) LED ]
      [ Katoda (-) LED ]
             │
             ▼
         [ GND (0V) ]
```

### Prosedur Pengukuran:
1. Pastikan rangkaian LED + Resistor terhubung di breadboard.
2. Cabut salah satu kabel penghubung (misal kabel dari pin 3V3 ke kaki resistor) sehingga rangkaian terputus.
3. Atur multimeter ke mode **DC Current (mA ⎓)**.
4. Sisipkan multimeter ke dalam sela-sela jalur kabel yang putus tersebut:
   - Probe MERAH ke arah sumber tegangan (3.3V).
   - Probe HITAM ke arah kaki resistor yang menuju LED.
5. Saat LED menyala, multimeter akan menunjukkan nilai arus riil yang mengalir melalui sirkuit (biasanya terbaca sekitar $5\text{ mA} - 15\text{ mA}$).

---

## 8. Praktik Perancangan: Menghitung Power Budget Sistem IoT

Misalkan Anda ingin merancang sebuah stasiun cuaca IoT berbasis ESP32 dengan komponen berikut:
1. **ESP32 DevKit:** Saat transmisi Wi-Fi aktif = $\sim 200\text{ mA}$
2. **Sensor Suhu DHT22:** Mengukur data = $\sim 1.5\text{ mA}$
3. **Layar LCD I2C 16x2:** Backlight menyala = $\sim 25\text{ mA}$
4. **Relay Module 5V:** Koil aktif mengunci = $\sim 70\text{ mA}$

### Perhitungan Total Kebutuhan Arus Maksimal ($I_{total}$):
$$I_{total} = 200\text{ mA} + 1.5\text{ mA} + 25\text{ mA} + 70\text{ mA} = 296.5\text{ mA}$$

Tambahkan faktor keamanan (*safety margin*) sebesar minimal **$30\%$**:
$$I_{rekomendasi} = 296.5\text{ mA} \times 1.3 \approx 385\text{ mA}$$

**Kesimpulan Engineering:**
Power supply adaptor $5\text{V}$ dengan kapasitas minimal **$500\text{ mA} - 1\text{ A}$** sudah sangat ideal dan aman untuk menjamin sistem berjalan 24/7 tanpa mengalami brownout.

---

## 9. 🧠 Tantangan Berpikir Analitis

Jawablah 3 pertanyaan kritis berikut:
1. **Kasus Charger:** Anda memiliki dua buah charger HP: Adaptor A (5V 1A) dan Adaptor B (5V 3A). Apakah kedua charger ini aman dipakai untuk menyuplai ESP32? Apa perbedaan dampak fisiknya pada kinerja rangkaian?
2. **Korsleting (Short Circuit):** Jelaskan secara matematis menggunakan Hukum Ohm ($I = \frac{V}{R}$) mengapa kabel telanjang yang langsung menghubungkan pin 5V ke pin GND dapat mengeluarkan percikan api dan merusak port USB laptop Anda!
3. **Baterai 18650:** Sebuah baterai Li-ion 18650 memiliki kapasitas tertulis $2600\text{ mAh}$. Jika ESP32 diprogram menggunakan mode *Deep Sleep* sehingga rata-rata hanya mengonsumsi arus sebesar $0.5\text{ mA}$, secara teoritis berapa lama sistem IoT tersebut dapat bertahan hidup tanpa diisi ulang? (Gunakan rumus: $T = \frac{\text{Kapasitas}}{\text{Konsumsi Arus}}$).

---

## 10. 📚 Panduan Dokumentasi GitHub

Buat file dokumentasi di:
`month-01/week-01/day-03/README.md`

Format standar `README.md`:
```markdown
# Day 03 — Current, Load & ESP32 Current Limits

## 🎯 Objective
Memahami konsep arus listrik ($I$), perbedaan antara rating sumber daya vs konsumsi beban, batasan arus pin GPIO ESP32, bahaya korsleting, dan teknik menyusun anggaran arus (*power budget*).

## 📚 What I Learned
- Definisi Arus Listrik ($I$) sebagai laju aliran muatan elektron (satuan Ampere / mA).
- Prinsip beban: Tegangan disuplai konstan oleh sumber, sedangkan arus ditarik sesuai kebutuhan beban.
- Batas aman pin GPIO ESP32 (maksimum 40mA, rekomendasi $\le 20mA$).
- Alasan fundamental mengapa resistor pembatas arus wajib dipasang pada LED.
- Konsumsi daya dinamis ESP32 (Deep Sleep vs Wi-Fi TX burst).
- Mengapa aktuator berarus besar (Servo, Motor DC) wajib memakai catu daya terpisah (*external power supply*).

## 🧮 Power Budget Calculation
[Sertakan tabel perhitungan kebutuhan arus sistem yang Anda rancang]

## 🧠 Questions & Answers
[Jawab ketiga pertanyaan tantangan berpikir dengan analisis mendalam]

## 📝 Conclusion
[Tuliskan kesimpulan pemahaman Anda hari ini]
```

---

## 11. Git Workflow Day-03

```bash
git status
git add month-01/week-01/day-03/
git commit -m "docs(day-03): complete electrical current theory, power budget, and gpio limitations"
git push origin main
```

---

## ✅ Checklist Kelulusan Day-03
- [ ] Memahami perbedaan esensial antara Tegangan (dorongan) dan Arus (aliran).
- [ ] Mengetahui bahwa adaptor ber-Ampere besar tidak merusak beban ber-Ampere kecil asalkan tegangannya sama.
- [ ] Mengetahui batasan arus pin GPIO ESP32 ($\le 20\text{mA}$).
- [ ] Mampu menghitung nilai resistor pembatas arus untuk LED.
- [ ] Memahami mengapa servo/motor butuh catu daya eksternal dan common ground.
- [ ] Memahami cara pengukuran arus dengan multimeter secara SERI dan bahaya jika diukur paralel.
- [ ] Mampu menyusun perhitungan Power Budget untuk sistem IoT.
- [ ] Menyelesaikan tantangan berpikir dan mendokumentasikan ke GitHub.

---

## 🎯 Prinsip Esensial Hari Ini
> **"Tegangan yang salah merusak perangkat seketika; Arus yang tak terkontrol merusak komponen perlahan lewat panas berlebih atau meruntuhkan kestabilan sistem lewat brownout."**
