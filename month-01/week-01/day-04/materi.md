# 🟢 Day 04 — Resistansi (Resistance), Hukum Ohm, & Resistor

Setelah menguasai **Tegangan ($V$)** dan **Arus ($I$)**, hari ini kita mempelajari jembatan matematika dan fisik yang menghubungkan keduanya: **Resistansi (Hambatan Listrik / $R$)** dan **Hukum Ohm**.

Komponen paling murah, paling sederhana, namun paling sering menyelamatkan mikrokontroler dari kehancuran adalah **Resistor**. Tanpa resistor, LED akan meledak, pin ESP32 akan terbakar, sensor tidak bisa dibaca stabil, dan bus komunikasi digital seperti I2C akan macet total.

---

## 1. Analisis Masalah

### Pertanyaan Utama Hari Ini:
> **Mengapa sebuah lampu LED kecil membutuhkan resistor pembatas arus saat dipasang ke pin GPIO 3.3V ESP32, dan apa yang terjadi jika nilai resistor yang dipasang terlalu kecil (misal $10\Omega$) atau terlalu besar (misal $1\text{ M}\Omega$)?**

Banyak pemula mengira:
> *"Tegangan ESP32 kan cuma 3.3V, kecil kok, masa bisa merusak LED atau pin?"*

Jawabannya: **Bisa, dan sangat cepat!** Tegangan hanyalah dorongan. Jika tidak ada hambatan (resistansi) yang menahan dorongan tersebut, arus akan mengalir tanpa batas hingga komponen terlemah dalam sirkuit meleleh.

---

## 2. Teori Dasar: Apa Itu Resistansi & Hukum Ohm?

### Definisi Resistansi
**Resistansi (Resistance / Simbol: $R$)** adalah kemampuan suatu material untuk **menahan atau membatasi aliran muatan listrik (elektron)**.
Satuan resistansi adalah **Ohm (dilambangkan dengan simbol Yunani Omega: $\Omega$)**.

Dalam elektronika kita sering menggunakan kelipatan:
- **$1\text{ kilo-Ohm (k}\Omega\text{)}$** = $1.000\ \Omega$
- **$1\text{ Mega-Ohm (M}\Omega\text{)}$** = $1.000.000\ \Omega$

### 🌊 Analogi Saluran Air:
```text
      Tekanan Air (Tegangan / V)
            ════════════►
     ┌───────────┐         ┌───────────┐
     │   Pipa    │ Penyem- │   Pipa    │
     │   Lebar   ├───pitan─┤   Lebar   │
     │           │ (Resis- │           │
     │           │  tansi) │           │
     └───────────┘    ▲    └───────────┘
                      │
           RESISTANSI / HAMBATAN (R)
   (Mengatur berapa liter air per detik yang boleh lolos = Arus / I)
```

---

### 📐 Hukum Ohm (Ohm's Law)
Ditemukan oleh fisikawan Jerman Georg Simon Ohm pada tahun 1827. Hukum ini menyatakan bahwa **arus listrik ($I$) yang mengalir melalui suatu konduktor berbanding lurus dengan tegangan beda potensial ($V$) dan berbanding terbalik dengan resistansi ($R$)**:

$$V = I \times R$$

Dari satu rumus dasar ini, kita mendapatkan segitiga ajaib:

```text
         /  V  \          V = I × R
        /───────\         I = V / R
       /  I │ R  \        R = V / I
      ─────────────
```

- **Mencari Tegangan:** $V = I \times R$ (Tegangan = Arus dikali Hambatan)
- **Mencari Arus:** $I = \frac{V}{R}$ (Arus = Tegangan dibagi Hambatan)
- **Mencari Nilai Hambatan:** $R = \frac{V}{I}$ (Hambatan = Tegangan dibagi Arus)

---

## 3. Menghitung Resistor Pembatas Arus untuk LED

LED adalah dioda pemancar cahaya semikonduktor. Setiap LED memiliki karakteristik tegangan jatuh maju (**Forward Voltage Drop / $V_f$**) yang berbeda tergantung warnanya:
- **LED Merah:** $V_f \approx 1.8\text{V} - 2.0\text{V}$
- **LED Kuning / Hijau:** $V_f \approx 2.0\text{V} - 2.2\text{V}$
- **LED Biru / Putih:** $V_f \approx 3.0\text{V} - 3.2\text{V}$

Arus kerja yang aman dan terang untuk LED standar $5\text{mm}$ adalah **$10\text{ mA} - 15\text{ mA}$** ($0.010\text{A} - 0.015\text{A}$).

```text
  [ GPIO ESP32 (3.3V) ]
           │
           ▼
    [ Resistor R ]  ◄── Menahan selisih tegangan (VR = V_GPIO - V_LED)
           │
           ▼
     [ LED Merah ]  ◄── Menjatuhkan tegangan maju (Vf = 1.8V)
           │
           ▼
      [ GND (0V) ]
```

### Langkah Perhitungan Hukum Ohm:
1. Hitung tegangan yang harus diserap oleh resistor ($V_R$):
   $$V_R = V_{GPIO} - V_f = 3.3\text{V} - 1.8\text{V} = 1.5\text{ Volt}$$
2. Tentukan target arus aman ($I = 10\text{ mA} = 0.01\text{ A}$):
3. Hitung nilai hambatan resistor ($R$) dengan rumus $R = \frac{V_R}{I}$:
   $$R = \frac{1.5\text{V}}{0.01\text{A}} = 150\ \Omega$$

Karena nilai standar resistor yang paling umum di pasaran adalah **$220\ \Omega$** atau **$330\ \Omega$**:
- Jika kita memakai resistor **$220\ \Omega$**:
  $$I = \frac{1.5\text{V}}{220\ \Omega} \approx 6.8\text{ mA}$$
  *(Arus ini sangat aman untuk GPIO ESP32 dan LED tetap menyala terang).*
- Jika resistor diganti **$10\ \Omega$**:
  $$I = \frac{1.5\text{V}}{10\ \Omega} = 150\text{ mA}!$$
  *(Pin GPIO ESP32 akan langsung overload dan berisiko rusak karena melebihi batas 40mA!)*
- Jika resistor diganti **$1\text{ M}\Omega$ ($1.000.000\ \Omega$)**:
  $$I = \frac{1.5\text{V}}{1.000.000\ \Omega} = 0.0000015\text{ A} = 1.5\ \mu\text{A}$$
  *(Arus terlalu kecil, LED tidak akan menyala sama sekali).*

---

## 4. Membaca Kode Warna Resistor

Resistor jenis *carbon composition* atau *metal film* menggunakan cincin gelang warna untuk mengindikasikan nilai hambatannya:

```text
Tabel Hafalan Cincin Warna:
┌─────────┬─────────┬──────────────┬────────────────┐
│ Warna   │ Angka   │ Pengali (×)  │ Toleransi      │
├─────────┼─────────┼──────────────┼────────────────┤
│ Hitam   │ 0       │ 10⁰ (×1)     │ —              │
│ Cokelat │ 1       │ 10¹ (×10)    │ ±1% (F)        │
│ Merah   │ 2       │ 10² (×100)   │ ±2% (G)        │
│ Oranye  │ 3       │ 10³ (×1k)    │ —              │
│ Kuning  │ 4       │ 10⁴ (×10k)   │ —              │
│ Hijau   │ 5       │ 10⁵ (×100k)  │ ±0.5%          │
│ Biru    │ 6       │ 10⁶ (×1M)    │ ±0.25%         │
│ Ungu    │ 7       │ 10⁷ (×10M)   │ ±0.1%          │
│ Abu-abu │ 8       │ 10⁸          │ ±0.05%         │
│ Putih   │ 9       │ 10⁹          │ —              │
│ Emas    │ —       │ ×0.1         │ ±5% (J)        │
│ Perak   │ —       │ ×0.01        │ ±10% (K)       │
└─────────┴─────────┴──────────────┴────────────────┘
Jembatan Keledai: "HI-CO-ME-O-KU-HI-BI-U-A-P"
```

### Contoh Praktis Resistor 4 Gelang:
1. **Resistor $220\ \Omega$:**
   - Gelang 1: **Merah** (2)
   - Gelang 2: **Merah** (2)
   - Gelang 3: **Cokelat** ($\times 10$)
   - Gelang 4: **Emas** ($\pm 5\%$)
   - Hasil: $22 \times 10 = 220\ \Omega \pm 5\%$
2. **Resistor $10\text{ k}\Omega$ ($10.000\ \Omega$):**
   - Gelang 1: **Cokelat** (1)
   - Gelang 2: **Hitam** (0)
   - Gelang 3: **Oranye** ($\times 1.000$)
   - Gelang 4: **Emas** ($\pm 5\%$)
   - Hasil: $10 \times 1.000 = 10.000\ \Omega = 10\text{ k}\Omega$

---

## 5. Rangkaian Resistor: Seri vs Paralel

```text
RANGKAIAN SERI (Berderet):
   ───[ R1 ]───[ R2 ]───
   • Hambatan total bertambah besar:
     R_total = R1 + R2
   • Arus yang mengalir di setiap resistor sama besar.
   • Beda potensial (tegangan) terbagi di masing-masing resistor.

RANGKAIAN PARALEL (Bercabang):
         ┌───[ R1 ]───┐
      ───┤            ├───
         └───[ R2 ]───┘
   • Hambatan total menjadi lebih kecil daripada resistor terkecil:
     1 / R_total = (1 / R1) + (1 / R2)  atau  R_total = (R1 × R2) / (R1 + R2)
   • Tegangan di kedua kaki masing-masing resistor sama besar.
   • Arus total terbagi ke masing-masing cabang.
```

---

## 6. Aplikasi Kritis Resistor di Dunia IoT

Resistor dalam sistem ESP32 tidak hanya untuk menyalakan LED. Dua fungsi terbesarnya adalah:

### 1. Resistor Pull-Up pada Bus I2C ($4.7\text{k}\Omega - 10\text{k}\Omega$)
Layar LCD I2C 16x2 atau sensor OLED menggunakan jalur data SDA dan SCL bertipe *Open-Drain*. 
Mikrokontroler dan sensor hanya bisa menarik sinyal ke GND (LOW), tetapi tidak bisa mendorong sinyal ke 3.3V (HIGH).
**Resistor Pull-Up** menghubungkan jalur SDA dan SCL ke 3.3V sehingga saat tidak ada yang mengirim data, sinyal otomatis ditarik ke level HIGH. Tanpa pull-up resistor, komunikasi I2C akan macet (*bus hang*).

### 2. Resistor Pull-Up pada Sensor Suhu DHT22 ($4.7\text{k}\Omega - 10\text{k}\Omega$)
Pin sinyal DATA pada sensor DHT22 memerlukan resistor pull-up eksternal ke 3.3V agar kabel komunikasi tidak melayang (*floating*) saat jeda antar-transmisi paket data pulsa waktu.

---

## 7. Praktikum Day-04: Mengukur Resistansi dengan Multimeter

> **🚨 PERINGATAN KESELAMATAN WAJIB:**
> **JANGAN PERNAH MENGUKUR RESISTANSI PADA SIRKUIT YANG SEDANG DIALIRI LISTRIK!**
> Pastikan rangkaian mati total atau cabut komponen yang ingin diukur dari breadboard. Mengukur resistansi saat ada tegangan aktif akan merusak multimeter dan menghasilkan pembacaan yang salah total.

### Langkah Pengukuran Ohmmeter:
1. Hubungkan probe HITAM ke lubang **COM**.
2. Hubungkan probe MERAH ke lubang **V$\Omega$mA**.
3. Putar selector switch multimeter ke simbol **$\Omega$ (Ohm)**.
4. Sentuhkan kedua ujung probe logam merah dan hitam: multimeter harus membaca mendekati **$0.0\ \Omega - 0.2\ \Omega$** (resistansi kabel probe sendiri).
5. Jepitkan kedua probe ke masing-masing kaki resistor (tidak ada polaritas plus-minus pada resistor, boleh terbalik).
6. Bandingkan hasil pembacaan angka pada layar multimeter dengan gelang warna resistor:
   - Resistor $220\ \Omega$ dengan toleransi $5\%$ emas normalnya terbaca di kisaran $209\ \Omega - 231\ \Omega$.
   - Resistor $10\text{ k}\Omega$ normalnya terbaca di kisaran $9.5\text{ k}\Omega - 10.5\text{ k}\Omega$.

---

## 8. 🧠 Tantangan Berpikir Analitis

Jawablah 3 pertanyaan praktikum ini:
1. **Analisis Toleransi:** Sebuah resistor memiliki gelang warna: **Cokelat - Hitam - Merah - Emas**. Berapakah nilai nominalnya? Berapakah rentang nilai minimum dan maksimum yang masih dianggap wajar menurut toleransinya?
2. **Kombinasi Paralel:** Anda membutuhkan resistor $500\ \Omega$ untuk sebuah sirkuit, tetapi di kotak komponen Anda hanya ada resistor $1\text{ k}\Omega$. Bagaimana cara Anda merangkai resistor yang ada agar bernilai tepat $500\ \Omega$? Buktikan dengan rumus!
3. **Floating Pin:** Apa yang terjadi jika pin tombol push-button dihubungkan langsung ke pin GPIO input ESP32 tanpa resistor *Pull-Up* atau *Pull-Down*? Mengapa nilai `digitalRead()` bisa berosilasi acak antara 0 dan 1 saat tangan kita didekatkan tanpa menyentuh tombol?

---

## 9. 📚 Panduan Dokumentasi GitHub

Buat file dokumentasi di:
`month-01/week-01/day-04/README.md`

Format standar `README.md`:
```markdown
# Day 04 — Resistance, Ohm's Law & Circuit Protection

## 🎯 Objective
Memahami konsep hambatan listrik (Resistance / $R$), menguasai perhitungan Hukum Ohm ($V = I \times R$), membaca kode warna resistor, serta memahami peran vital resistor dalam proteksi GPIO dan bus komunikasi IoT.

## 📚 What I Learned
- Definisi resistansi dan Hukum Ohm.
- Cara menghitung nilai resistor pembatas arus untuk LED pada ESP32 (3.3V).
- Cara membaca tabel cincin kode warna 4 gelang.
- Perbedaan perilaku resistor pada rangkaian seri ($R_{total} = R_1 + R_2$) dan paralel.
- Penerapan resistor Pull-Up pada bus komunikasi I2C (LCD/OLED) dan single-bus DHT22.
- Pengukuran nilai resistansi aktual menggunakan Ohmmeter multimeter.

## 🧮 Calculations & Experiments
[Tuliskan perhitungan LED dan hasil perbandingan gelang warna vs multimeter]

## 🧠 Questions & Answers
[Jawab ketiga pertanyaan tantangan berpikir secara mandiri]

## 📝 Conclusion
[Tuliskan kesimpulan pemahaman Anda hari ini]
```

---

## 10. Git Workflow Day-04

```bash
git status
git add month-01/week-01/day-04/
git commit -m "docs(day-04): complete resistance theory, ohm's law calculations, and color code mastery"
git push origin main
```

---

## ✅ Checklist Kelulusan Day-04
- [ ] Memahami hubungan matematis dan fisis dari Hukum Ohm ($V = I \times R$).
- [ ] Mampu menghitung nilai resistor pembatas arus untuk LED pada tegangan 3.3V.
- [ ] Mampu membaca kode warna resistor tanpa bantuan tabel.
- [ ] Mengetahui perbedaan hambatan total pada rangkaian seri vs paralel.
- [ ] Memahami fungsi resistor Pull-Up pada sensor DHT22 dan bus I2C.
- [ ] Mengukur nilai resistor secara aman menggunakan multimeter (kondisi unpowered).
- [ ] Menyelesaikan tantangan berpikir dan mendokumentasikan ke GitHub.

---

## 🎯 Prinsip Esensial Hari Ini
> **"Hukum Ohm adalah hukum gravitasi dalam elektronika. Setiap volt yang Anda berikan dan setiap ampere yang mengalir tunduk sepenuhnya pada hambatan yang Anda pasang."**
