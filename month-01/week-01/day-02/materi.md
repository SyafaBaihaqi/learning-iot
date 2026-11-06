# 🟢 Day 02 — Tegangan Listrik (Voltage) & Logika Level ESP32

Sebelum kita menghubungkan sensor atau aktuator apa pun ke ESP32, kita wajib memahami **Tegangan (Voltage)**. Banyak pemula merusak board mikrokontroler (*chip terbakar / mengeluarkan asap tipis*) atau frustrasi karena ESP32 mengalami *restart* berulang kali tanpa tahu sebabnya. Akar dari sebagian besar masalah tersebut terletak pada pemahaman tegangan yang keliru.

---

## 1. Analisis Masalah

### Pertanyaan Utama Hari Ini:
> **Mengapa menghubungkan kabel power 5V langsung ke pin GPIO ESP32 dapat merusak chip secara permanen, sementara menghubungkannya ke pin VIN aman-aman saja?**

Banyak orang mengira:
> *"Listrik ya listrik, asal ada colokannya pasti nyala."*

Dalam elektronika mikrokontroler, **level tegangan (voltage level) adalah batas hidup dan mati sebuah komponen**. ESP32 bekerja pada level tegangan yang berbeda dengan Arduino Uno konvensional. Memahami tegangan bukan hanya soal menghafal angka "3.3V" atau "5V", melainkan memahami konsep **Beda Potensial** dan **Toleransi Tegangan**.

---

## 2. Teori Dasar: Apa Itu Tegangan?

### Definisi Fisika & Intuitif
**Tegangan (Voltage / Simbol: $V$)** adalah **beda potensial listrik** antara dua titik dalam suatu rangkaian. 
Satuan tegangan adalah **Volt (V)**.

Tegangan adalah "gaya dorong" atau tekanan yang menyebabkan muatan listrik (elektron) bergerak melalui konduktor.

### 🌊 Analogi Tangki Air & Pipa:
Bayangkan sebuah tangki air di atas menara yang dihubungkan dengan pipa ke tanah:
```text
      ┌───────────┐
      │  TANGKI   │  ◄── Potensial Tinggi (VCC / 5V / 3.3V)
      │    AIR    │
      └─────┬─────┘
            │
            │  ◄──────── Beda Ketinggian = TEGANGAN (VOLTAGE)
            │            Semakin tinggi tangki, semakin besar tekanan dorongnya.
            ▼
      ┌───────────┐
      │   KRAN    │
      └─────┬─────┘
            │
            ▼ ◄───────── Saluran Pembuangan ke Tanah (GND / 0V)
     ═══════════════
```
- **Ketinggian tangki terhadap tanah** = **Tegangan (Voltage)**.
- **Tanah (Ground / GND)** = Titik referensi $0\text{ Volt}$.
- Tanpa adanya beda ketinggian (beda potensial), air tidak akan pernah mengalir. Sama halnya dengan listrik: **arus listrik tidak akan mengalir jika tidak ada beda tegangan antara dua titik.**

### Listrik DC (Direct Current) vs AC (Alternating Current)
- **DC (Arus Searah):** Polaritas tegangan tetap stabil (kutub positif selalu positif, kutub negatif selalu negatif). Contoh: Baterai, output charger HP, port USB, dan catu daya mikrokontroler.
- **AC (Arus Bolak-Balik):** Polaritas tegangan berubah-ubah secara periodik bolak-balik (membentuk gelombang sinus). Contoh: Listrik PLN rumah tangga (220V AC).
> **Peringatan Keras:** Semua rangkaian dasar mikrokontroler dan sensor kita menggunakan **DC murni**. Jangan pernah menghubungkan komponen DC langsung ke listrik stopkontak AC!

---

## 3. Karakteristik Tegangan pada ESP32

Board ESP32 DevKit memiliki beberapa pin tegangan penting yang wajib dibedakan dengan sangat teliti:

```text
               ┌──────────┐
         3V3 ──┤ 1     30 ├── CLK / GPIO6
          EN ──┤ 2     29 ├── SD0 / GPIO7
        GPIO36 ──┤ 3     28 ├── SD1 / GPIO8
        GPIO39 ──┤ 4     27 ├── SD2 / GPIO9
        GPIO34 ──┤ 5     26 ├── SD3 / GPIO10
        GPIO35 ──┤ 6     25 ├── CMD / GPIO11
        GPIO32 ──┤ 7     24 ├── GPIO5
        GPIO33 ──┤ 8     23 ├── GPIO18
        GPIO25 ──┤ 9     22 ├── GPIO19
        GPIO26 ──┤ 10    21 ├── GPIO21
        GPIO27 ──┤ 11    20 ├── RXD0 / GPIO3
        GPIO14 ──┤ 12    19 ├── TXD0 / GPIO1
        GPIO12 ──┤ 13    18 ├── GPIO22
        GPIO13 ──┤ 14    17 ├── GPIO23
         GND ──┤ 15    16 ├── VIN / 5V  ◄── Input Daya USB / Adaptor
               └──────────┘
```

### 1. Pin VIN (atau tertulis 5V / VBUS)
- Terhubung langsung ke jalur daya 5V dari kabel USB.
- Jalur ini masuk ke sebuah komponen bernama **Onboard Voltage Regulator (LDO)** (biasanya IC AMS1117-3.3 atau ME6211).
- Regulator ini bertugas menurunkan tegangan dari **5.0V menjadi 3.3V stabil** untuk memberi makan chip ESP32.
- **Batas aman input VIN:** $4.8\text{V} - 6.0\text{V}$ (maksimal mutlak $9\text{V}-12\text{V}$ jika terpaksa, tapi regulator akan sangat panas).

### 2. Pin 3V3 (3.3V Output)
- Merupakan jalur output dari regulator onboard.
- Inilah tegangan operasi inti dari chip prosesor ESP32.
- Dapat digunakan untuk menyuplai sensor berdaya rendah (DHT22, TCRT5000, potensioneter).
- **PERINGATAN:** Arus dari pin 3V3 terbatas (maksimal sekitar $500\text{mA} - 800\text{mA}$). Jangan gunakan pin ini untuk menyuplai motor atau servo besar!

### 3. Pin GND (Ground / 0V Reference)
- Titik referensi dasar bertegangan $0\text{ Volt}$.
- Semua pengukuran tegangan dilakukan dengan probe hitam multimeter dihubungkan ke GND.
- **Konsep Common Ground:** Jika menggunakan dua sumber daya berbeda (misal: ESP32 dicolok USB, motor ditenagai baterai 7.4V terpisah), **kabel GND kedua sistem HARUS digabung**. Tanpa common ground, ESP32 tidak bisa membaca level tegangan sinyal dengan benar.

---

## 4. Logika Level: 3.3V Logic vs 5V Logic

Mikrokontroler mengenali dunia digital hanya melalui dua kondisi: **HIGH (Logika 1)** dan **LOW (Logika 0)**. Yang membedakan adalah berapa nilai voltasenya:

```text
Logika 5V (Arduino Uno / Sensor Klasik)       Logika 3.3V (ESP32 / Raspberry Pi / ARM)
┌───────────────────────────────── 5.0V       ┌───────────────────────────────── 3.3V
│                                             │
│        HIGH (3.5V - 5.0V)                   │        HIGH (2.64V - 3.3V)
│                                             │
├───────────────────────────────── 3.5V       ├───────────────────────────────── 2.64V
░░░░░░ Ambang Ragu / Terlarang ░░░░░░         ░░░░░░ Ambang Ragu / Terlarang ░░░░░░
├───────────────────────────────── 1.5V       ├───────────────────────────────── 0.8V
│                                             │
│        LOW (0.0V - 1.5V)                    │        LOW (0.0V - 0.8V)
│                                             │
└───────────────────────────────── 0.0V (GND) └───────────────────────────────── 0.0V (GND)
```

> **⚠️ BAHAYA FATAL: Pin GPIO ESP32 TIDAK 5V Tolerant!**
> Batas maksimum mutlak tegangan pada pin GPIO ESP32 adalah **$3.6\text{V}$**.
> Jika Anda menghubungkan pin sinyal dari sensor 5V (misal: Pin *Echo* dari HC-SR04 versi 5V) langsung ke pin input ESP32, tegangan 5V akan merusak sirkuit internal transistor ESP32.

---

## 5. Studi Kasus Nyata: Mengatasi Sensor HC-SR04 (5V) ke ESP32

Sensor jarak ultrasonik HC-SR04 standar membutuhkan catu daya 5V untuk pemancar ultrasoniknya dan menghasilkan sinyal balik pada pin **Echo** sebesar **5V**.

Bagaimana cara menghubungkannya ke ESP32 yang hanya aman menerima 3.3V?
Solusinya: Menggunakan **Voltage Divider (Pembagi Tegangan Resistor)**!

### Rumus Pembagi Tegangan:
$$V_{out} = V_{in} \times \frac{R_2}{R_1 + R_2}$$

```text
Pin ECHO HC-SR04 (5V) ───────┬──────── R1 (1 kΩ)
                             │
                             ├──────────────► Ke GPIO ESP32 (3.3V)
                             │
                            [R2] (2 kΩ)
                             │
                            GND (0V)
```

**Perhitungan:**
$$V_{out} = 5\text{V} \times \frac{2000}{1000 + 2000} = 5\text{V} \times \frac{2}{3} = 3.33\text{ Volt}$$
Tegangan $3.33\text{V}$ ini sangat aman dan sempurna untuk dibaca oleh pin GPIO ESP32!

---

## 6. Fenomena Nyata IoT: "Brownout Reset"

Pernahkah Anda menemui ESP32 yang me-restart sendiri terus menerus saat kode mulai menjalankan perintah `WiFi.begin()`?
Pesan error di Serial Monitor:
```text
rst:0x10 (RTCWDT_RTC_RESET),boot:0x13 (SPI_FAST_FLASH_BOOT)
Brownout detector was triggered
```

### Mengapa ini terjadi?
Modul radio Wi-Fi ESP32 membutuhkan lonjakan arus sesaat (*burst current*) hingga $250\text{mA}$ saat mulai memancarkan sinyal radio. 
Jika kabel USB tipis/jelek, atau port USB komputer memiliki hambatan tinggi, tegangan input 5V akan **merosot (voltage drop)** ke bawah $4.5\text{V}$, yang membuat output 3.3V anjlok ke bawah $2.8\text{V}$. 
Sensor internal ESP32 mendeteksi tegangan drop tersebut dan langsung memicu *Brownout Detector* untuk mereset CPU demi mencegah korupsi memori.

---

## 7. Praktikum Day-02: Mengukur Tegangan dengan Multimeter

Siapkan Multimeter Digital Anda. Hari ini kita melatih keterampilan dasar paling krusial: **mengukur tegangan DC secara aman.**

```text
Langkah Pemasangan Probe Multimeter:
1. Pasang kabel HITAM ke colokan bertuliskan "COM" (Common / Ground).
2. Pasang kabel MERAH ke colokan bertuliskan "VΩmA" (Tegangan/Resistansi).
3. Putar sakelar pemilih (selector switch) ke posisi "V⎓" (DC Voltage / garis lurus dengan garis putus-putus di bawahnya).
4. Jika multimeter manual memiliki range: pilih range "20V" (karena kita mengukur tegangan di bawah 20V).
```

### Prosedur Pengukuran:
1. **Uji Port USB:**
   - Hubungkan kabel USB dari PC/Laptop ke ESP32.
   - Tempelkan probe HITAM ke pin **GND** ESP32.
   - Tempelkan probe MERAH ke pin **VIN** (atau 5V) ESP32.
   - *Catat hasil pengukuran pada tabel (seharusnya di kisaran 4.75V - 5.25V).*
2. **Uji Output Regulator 3.3V:**
   - Probe HITAM tetap di **GND**.
   - Tempelkan probe MERAH ke pin **3V3** ESP32.
   - *Catat hasil pengukuran (seharusnya sangat stabil di kisaran 3.28V - 3.32V).*
3. **Uji Baterai (jika memiliki baterai AA / 18650 / 9V):**
   - Probe HITAM ke kutub negatif (-).
   - Probe MERAH ke kutub positif (+).
   - *Catat nilai voltase aktualnya.*

---

## 8. 🧠 Tantangan Berpikir Analitis

Jawablah 3 pertanyaan praktikum ini:
1. **Kasus Common Ground:** Anda membuat robot mobil menggunakan ESP32 yang ditenagai powerbank (5V) dan motor DC yang ditenagai baterai 2x 18650 (7.4V) melalui driver L298N. Apa yang terjadi jika kabel Ground (GND) dari ESP32 **tidak dihubungkan** ke kabel Ground (GND) driver L298N? Mengapa sinyal kontrol PWM tidak bisa bekerja?
2. **Bahaya Polaritas Terbalik:** Apa yang terjadi jika kabel daya positif (+) dan negatif (GND) terbalik saat dicolokkan ke pin VIN dan GND ESP32? Apakah ada proteksi dioda di sana?
3. **Analisis Tegangan Pin GPIO:** Jika Anda mengukur pin GPIO yang sedang diset `digitalWrite(pin, HIGH);` menggunakan multimeter ke GND, berapa voltase yang terbaca pada ESP32 dibandingkan pada Arduino Uno?

---

## 9. 📚 Panduan Dokumentasi GitHub

Buat folder dan file berikut:
- `month-01/week-01/day-02/measurement.md` (Tabel catatan pengukuran multimeter)
- `month-01/week-01/day-02/README.md` (Dokumentasi harian)

Format standar `README.md`:
```markdown
# Day 02 — Voltage & ESP32 Logic Levels

## 🎯 Objective
Memahami konsep beda potensial (Voltage), perbedaan logika 3.3V vs 5V, bahaya over-voltage pada pin GPIO ESP32, serta menguasai pengukuran tegangan DC dengan multimeter.

## 📚 What I Learned
- Definisi tegangan listrik sebagai beda potensial ($V$).
- Perbedaan pin VIN (5V input via USB) dan pin 3V3 (output regulated) pada ESP32 DevKit.
- Batas toleransi pin GPIO ESP32 (maksimal 3.6V, TIDAK 5V tolerant).
- Prinsip rangkaian pembagi tegangan (Voltage Divider) untuk menurunkan sinyal sensor 5V ke 3.3V.
- Penyebab dan solusi fenomena *Brownout Reset* pada ESP32.

## 📊 Measurement Log
[Sertakan data pengukuran multimeter dari measurement.md]

## 🧠 Analysis & Critical Questions
[Jawab ketiga pertanyaan tantangan berpikir dengan logika kelistrikan]

## 📝 Conclusion
[Tuliskan kesimpulan pemahaman Anda hari ini]
```

---

## 10. Git Workflow Day-02

```bash
git status
git add month-01/week-01/day-02/
git commit -m "docs(day-02): complete voltage theory, measurements, and esp32 logic levels"
git push origin main
```

---

## ✅ Checklist Kelulusan Day-02
- [ ] Memahami analogi tegangan sebagai tekanan/beda potensial.
- [ ] Mengetahui perbedaan fungsi pin VIN, 3V3, dan GND pada ESP32.
- [ ] Memahami mengapa pin GPIO ESP32 rusak jika diberi tegangan di atas 3.6V.
- [ ] Menguasai cara penggunaan multimeter untuk mengukur tegangan DC.
- [ ] Melakukan pengukuran tegangan pada pin VIN dan 3V3 ESP32 serta mencatat hasilnya.
- [ ] Memahami rumus dan fungsi Voltage Divider.
- [ ] Menyelesaikan tantangan berpikir dan mendokumentasikan ke GitHub.

---

## 🎯 Prinsip Esensial Hari Ini
> **"Tegangan adalah beda potensial. Jangan pernah berasumsi semua mikrokontroler menggunakan tegangan yang sama — selalu periksa logic level sebelum mencolokkan kabel sinyal ke GPIO ESP32."**
