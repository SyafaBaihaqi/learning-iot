# 🟢 Day 06 — Sinyal Digital vs Analog, ADC 12-Bit, & PWM

Dunia fisik di sekitar kita bersifat **Analog**: suhu udara di ruangan berubah secara halus dan kontinu dari $27.0^\circ\text{C}$ ke $27.1^\circ\text{C}$, intensitas cahaya matahari terbenam bergradasi secara mulus, dan kelembapan tanah mengering setetes demi setetes.

Namun, komputer dan chip ESP32 adalah mesin **Digital**: prosesornya hanya mengenal angka biner (0 dan 1, LOW dan HIGH).

Hari ini kita mempelajari bagaimana mikrokontroler menjembatani dua dunia ini:
1. Mengubah tegangan kontinu analog dari sensor menjadi angka menggunakan **ADC (Analog-to-Digital Converter)**.
2. Menghasilkan efek kontrol analog semu menggunakan **PWM (Pulse Width Modulation)** untuk mengatur kecepatan motor dan meredupkan cahaya.

---

## 1. Analisis Masalah

### Pertanyaan Utama Hari Ini:
> **Mengapa sensor kelembapan tanah (Capacitive Soil Moisture) yang dipasang ke pin GPIO2 (bagian dari ADC2) membaca data dengan sempurna saat offline, tetapi begitu ESP32 menyambung ke Wi-Fi (`WiFi.begin()`), hasil pembacaan sensor langsung kacau atau bernilai konstan 4095?**

Banyak developer pemula menghabiskan waktu berhari-hari mengira sensor mereka rusak atau kodenya salah. Padahal, masalahnya adalah **arsitektur perangkat keras ADC bawaan ESP32**. Memahami perbedaan sinyal digital vs analog dan seluk-beluk ADC internal akan membuat Anda merancang arsitektur pin (*pinout mapping*) dengan benar sejak awal.

---

## 2. Teori Dasar: Digital vs Analog

```text
SINYAL DIGITAL (Diskrit):
     3.3V ───┐       ┌───────┐       ┌───
             │       │       │       │
             │       │       │       │
       0V ───┴───────┘       └───────┘
          Hanya ada 2 nilai pasti: 0 (LOW / 0V) atau 1 (HIGH / 3.3V)

SINYAL ANALOG (Kontinu):
     3.3V ───┐     /\        /\
             │    /  \      /  \  ~ Kontinu
             │   /    \    /    \
       0V ───┴──/──────\──/──────\───────
          Memiliki nilai tak terhingga di antara rentang 0.00V hingga 3.30V
```

### Perbandingan Karakteristik:
| Parameter | Sinyal Digital | Sinyal Analog |
|---|---|---|
| **Nilai Tegangan** | Diskrit: 0V atau 3.3V | Kontinu: rentang apapun antara 0V hingga 3.3V |
| **Kekebalan Derau (Noise)** | Sangat tahan noise (sedikit fluktuasi voltase tidak mengubah arti 0 atau 1) | Sangat rentan noise (fluktuasi sedikit langsung mengubah nilai pembacaan) |
| **Transmisi Jarak Jauh** | Sangat handal lewat protokol serial | Cenderung mengalami penurunan tegangan akibat resistansi kabel |
| **Contoh Sensor Anda** | Sakelar tombol, TCRT5000 (pin D0), DHT22 (protokol bit) | Capacitive Soil Moisture, Potensiometer, TCRT5000 (pin A0) |

---

## 3. ADC (Analog-to-Digital Converter) pada ESP32

Untuk memahami sinyal analog, ESP32 memiliki modul sirkuit internal bernama **ADC**. 
ADC membagi rentang tegangan $0\text{V} - 3.3\text{V}$ menjadi potongan-potongan tangga digital (*kuantisasi*).

### Resolusi ADC ESP32: 12-Bit ($0 - 4095$)
- Resolusi dinyatakan dalam jumlah bit biner:
  $$2^{12} = 4096\text{ tingkatan nilai (dari } 0 \text{ hingga } 4095)$$
- Sebagai perbandingan, Arduino Uno klasik hanya memiliki ADC **10-bit ($0 - 1023$)**. Artinya, ESP32 memiliki resolusi **4 kali lebih presisi**!

```text
Tegangan Input Analog (V)              Nilai Digital ADC (12-bit)
       3.30 V               ────────►           4095
       2.475 V (3/4 VCC)    ────────►           3071
       1.65 V  (1/2 VCC)    ────────►           2047
       0.825 V (1/4 VCC)    ────────►           1023
       0.00 V  (GND)        ────────►              0
```

### Rumus Konversi Nilai ADC ke Voltase:
$$V_{ukur} = \frac{\text{Nilai ADC}}{4095} \times 3.3\text{ Volt}$$

*Contoh:* Jika pin analog membaca angka `2730`:
$$V_{ukur} = \frac{2730}{4095} \times 3.3\text{V} \approx \mathbf{2.20\text{ Volt}}$$

---

## 4. Jebakan Arsitektur ADC ESP32: ADC1 vs ADC2

ESP32 memiliki 18 kanal ADC yang dibagi menjadi dua modul terpisah:

```text
┌────────────────────────────────────────────────────────────────────────┐
│                        MODUL ADC PADA ESP32                            │
├────────────────────────────────────────────────────────────────────────┤
│ 🟢 ADC1 (SANGAT AMAN DIGUNAKAN KAPAN SAJA):                            │
│    • Terhubung ke: GPIO32, GPIO33, GPIO34, GPIO35, GPIO36, GPIO39      │
│    • Bebas digunakan bersamaan dengan Wi-Fi dan Bluetooth.             │
│                                                                        │
│ 🔴 ADC2 (BERMASALAH JIKA WI-FI AKTIF!):                                │
│    • Terhubung ke: GPIO0, GPIO2, GPIO4, GPIO12, GPIO13, GPIO14,        │
│                    GPIO15, GPIO25, GPIO26, GPIO27                      │
│    • Modul ADC2 berbagi sirkuit silikon dengan DRIVER RADIO WI-FI.     │
│    • Begitu fungsi WiFi.begin() dipanggil, sirkuit ADC2 DIBLOKIR oleh   │
│      driver Wi-Fi sehingga analogRead() di pin ini AKAN GAGAL!         │
└────────────────────────────────────────────────────────────────────────┘
```

> **Aturan Emas IoT ESP32:**
> **SELALU sambungkan sensor analog Anda (Soil Moisture, LDR, Potensio) ke pin ADC1 (GPIO32 hingga GPIO39)! Jangan pernah memasang sensor analog ke pin ADC2 jika proyek Anda menggunakan Wi-Fi.**

### Catatan Non-Linearitas ADC ESP32:
ADC bawaan ESP32 terkenal memiliki karakteristik kurva non-linear pada ujung bawah ($<0.15\text{V}$) dan ujung atas ($>3.15\text{V}$). Di zona buta ini (*dead zone*), perubahan tegangan kecil tidak terbaca. Untuk sebagian besar aplikasi sensor lingkungan (seperti kadar air tanah $1.2\text{V}-2.5\text{V}$), rentang tengah ADC ESP32 sangat andal dan bekerja dengan baik.

---

## 5. PWM (Pulse Width Modulation) — Mengatur Output Analog Semu

Mikrokontroler tidak bisa mengubah pin output digitalnya menjadi tegangan murni 1.5V. Pin digital hanya bisa bernilai 0V atau 3.3V.

Lalu bagaimana cara kita:
- Meredupkan kecerahan LED?
- Mengatur kecepatan putar motor DC pada robot mobil L298N?
- Mengontrol sudut posisi servo SG90?

Jawabannya: **PWM (Pulse Width Modulation)**!
PWM adalah teknik menyalakan dan mematikan sinyal digital dengan sangat cepat (ribuan kali per detik / kHz) sehingga beban merasakan **tegangan rata-rata (Average Voltage)**:

```text
1. DUTY CYCLE 25% (LED Redup / Motor Pelan):
   3.3V ─┐   ┌─┐   ┌─┐   ┌─┐
     0V ─┴───┘ └───┘ └───┘ └───
   Tegangan Rata-rata = 25% × 3.3V = 0.825V

2. DUTY CYCLE 50% (LED Sedang / Motor Setengah Kecepatan):
   3.3V ─┌───┐   ┌───┐   ┌───┐
     0V ─┘   └───┘   └───┘   └───
   Tegangan Rata-rata = 50% × 3.3V = 1.65V

3. DUTY CYCLE 100% (LED Terang Penuh / Motor Full Speed):
   3.3V ────────────────────────
     0V ─
   Tegangan Rata-rata = 100% × 3.3V = 3.3V
```

### Rumus Duty Cycle:
$$\text{Duty Cycle (\%)} = \frac{T_{ON}}{T_{ON} + T_{OFF}} \times 100\%$$
$$V_{average} = \text{Duty Cycle} \times V_{CC}$$

Pada ESP32, kita menggunakan modul periferal internal bernama **LEDC (LED Control)** yang dapat menghasilkan sinyal PWM di hampir semua pin GPIO dengan resolusi hingga 16-bit.

---

## 6. Pemetaan Komponen Sensor & Aktuator Anda

Mari kelompokkan semua komponen hardware yang Anda miliki berdasarkan tipe sinyalnya:

```text
┌──────────────────────────────────────┬──────────────────────────────────────┐
│           KOMPONEN DIGITAL           │           KOMPONEN ANALOG            │
├──────────────────────────────────────┼──────────────────────────────────────┤
│ • Sensor DHT22                       │ • Capacitive Soil Moisture Sensor    │
│   (Protokol digital 1-wire bit timing│   (Output tegangan analog ~1.2V-2.8V)│
│ • Sensor HC-SR04                     │ • Water Level Sensor (Resistif)      │
│   (Trigger & Echo pulsa durasi waktu)│   (Output tegangan analog 0V - 2.5V) │
│ • TCRT5000 (Pin DO)                  │ • TCRT5000 (Pin AO)                  │
│   (Output logika 0 atau 1 via op-amp)│   (Output intensitas pantulan infra) │
│ • Modul Relay 5V                     │ • Potensiometer                      │
│   (Hanya status ON / OFF)            │   (Pembagi tegangan analog murni)    │
│ • Layar LCD I2C 16x2 / OLED          │ • Motor DC (Via kontrol PWM Duty)    │
│   (Protokol paket data bus serial)   │ • Servo SG90 (Via kontrol PWM pulsa) │
└──────────────────────────────────────┴──────────────────────────────────────┘
```

---

## 7. Praktikum Day-06: Analisis Konversi ADC & PWM

### Latihan Konversi ADC 12-Bit ke Tegangan
Lengkapi tabel konversi teoritis berikut untuk memahami skala pembacaan sensor analog ESP32:

| Kondisi Sensor Tanah | Nilai Mentah ADC (12-bit) | Perhitungan Tegangan ($V$) | Status Kelembapan |
|---|---|---|---|
| Sensor di Udara Kering | 3200 | $\frac{3200}{4095} \times 3.3\text{V} = \mathbf{2.58\text{ V}}$ | Tanah Kering (Perlu Disiram) |
| Sensor di Tanah Lembap | 2100 | $\frac{2100}{4095} \times 3.3\text{V} = \mathbf{1.69\text{ V}}$ | Kelembapan Ideal |
| Sensor Dicelupkan ke Air | 1350 | $\frac{1350}{4095} \times 3.3\text{V} = \mathbf{1.09\text{ V}}$ | Sangat Basah (Jenuh Air) |

---

## 8. 🧠 Tantangan Berpikir Analitis

1. **Konflik Hardware ADC2:** Anda sedang membangun sistem monitoring tanaman pintar berbasis ESP32. Anda menghubungkan Capacitive Soil Moisture ke pin GPIO4 dan menyalakan Wi-Fi untuk mengirim data ke server. Mengapa pembacaan sensor menjadi rusak saat modul Wi-Fi mulai memancar? Ke pin manakah sensor tanah tersebut harus Anda pindahkan?
2. **Hakikat PWM:** Apakah sinyal PWM dengan Duty Cycle 50% pada pin 3.3V benar-benar menghasilkan tegangan listrik analog 1.65V DC murni? Jika kita mengukurnya dengan osiloskop, bentuk gelombang seperti apa yang terlihat?
3. **Imunitas Derau Sensor:** Mengapa sensor digital DHT22 dapat dipasang dengan kabel sepanjang 10 meter menuju ESP32 tanpa kehilangan akurasi suhu, sementara sensor analog soil moisture akan mengalami pembacaan yang melenceng jauh jika disambung dengan kabel panjang yang sama?

---

## 9. 📚 Panduan Dokumentasi GitHub

Buat file dokumentasi di:
`month-01/week-01/day-06/README.md`

Format standar `README.md`:
```markdown
# Day 06 — Digital vs Analog, ADC 12-Bit & PWM

## 🎯 Objective
Memahami perbedaan mendasar antara sinyal digital dan analog, karakteristik periferal ADC 12-bit ESP32, batasan arsitektur ADC1 vs ADC2 saat Wi-Fi aktif, serta prinsip modulasi lebar pulsa (PWM).

## 📚 What I Learned
- Karakteristik sinyal digital (diskrit 0/1) vs sinyal analog (kontinu).
- Resolusi ADC 12-bit ESP32 ($0 - 4095$) dan rumus konversi ke tegangan: $V = \frac{\text{ADC}}{4095} \times 3.3\text{V}$.
- Batasan vital arsitektur ADC: ADC2 tidak dapat digunakan bersamaan dengan modul Wi-Fi, sensor analog wajib memakai ADC1 (GPIO32 - GPIO39).
- Prinsip kerja PWM untuk mengontrol kecerahan LED, kecepatan motor L298N, dan sudut servo.
- Pemetaan jenis sinyal seluruh komponen hardware yang dimiliki.

## 📊 ADC & PWM Analysis
[Sertakan tabel konversi nilai ADC sensor analog ke voltase riil]

## 🧠 Questions & Answers
[Jawab ketiga pertanyaan tantangan berpikir secara mandiri]

## 📝 Conclusion
[Tuliskan kesimpulan pemahaman Anda hari ini]
```

---

## 10. Git Workflow Day-06

```bash
git status
git add month-01/week-01/day-06/
git commit -m "docs(day-06): complete digital vs analog signals, adc1 vs adc2 architecture, and pwm theory"
git push origin main
```

---

## ✅ Checklist Kelulusan Day-06
- [ ] Memahami perbedaan sifat fisik sinyal digital dan analog.
- [ ] Mampu menghitung konversi nilai mentah ADC 12-bit ($0-4095$) ke satuan Volt.
- [ ] Mengetahui daftar pin ADC1 dan alasan mutlak menghindari ADC2 saat Wi-Fi aktif.
- [ ] Memahami konsep Duty Cycle PWM dalam menghasilkan tegangan rata-rata semu.
- [ ] Mampu mengklasifikasikan komponen yang dimiliki ke dalam kategori digital vs analog.
- [ ] Menyelesaikan tantangan berpikir dan mendokumentasikan ke GitHub.

---

## 🎯 Prinsip Esensial Hari Ini
> **"Dunia nyata berbicara dalam bahasa analog, sementara mikrokontroler berpikir dalam bahasa digital. Memilih pin ADC yang tepat dan memahami PWM adalah cara kita menerjemahkan realitas ke dalam logika program."**
