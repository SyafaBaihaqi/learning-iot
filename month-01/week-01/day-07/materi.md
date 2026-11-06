# 🟢 Day 07 — Evaluasi Mingguan (Weekly Review) & Blueprint Fondasi IoT

Selamat! Anda telah menyelesaikan **Minggu 01 (Week 01 — IoT Fundamentals)**. 

Dalam 6 hari terakhir, Anda tidak sekadar menghafal rumus atau menyalin kode; Anda telah membangun fondasi pola pikir seorang **IoT Systems Engineer**:
- Memahami beda sistem tertutup (*Embedded*) vs sistem terhubung (*IoT*).
- Mengetahui batas toleransi voltase ESP32 agar chip tidak hangus.
- Menghitung batasan arus GPIO dan alasan aktuator wajib dipasok sumber terpisah.
- Menguasai Hukum Ohm untuk memproteksi sirkuit dan komunikasi bus.
- Mengaudit disipasi daya panas dan strategi *Deep Sleep* baterai.
- Membedakan sinyal digital vs analog serta menavigasi batasan arsitektur ADC1/ADC2.

Hari ini adalah **Milestone Synthesis Day**. Kita tidak menambah teori baru, melainkan menghubungkan seluruh simpul pengetahuan ke dalam satu cetak biru arsitektur terintegrasi (*System Blueprint*).

---

## 1. Peta Sintesis 6 Hari: Menghubungkan Benang Merah

Lihat bagaimana seluruh materi Minggu 1 saling bertaut membentuk piramida pemahaman:

```text
                        ┌────────────────────────┐
                        │      DAY 01: IoT       │
                        │ (Sense-Process-Comm-Act)│
                        └───────────┬────────────┘
                                    │
                        ┌───────────┴────────────┐
                        │  DAY 06: SIGNALS & ADC │
                        │ (Analog 12-bit vs PWM) │
                        └───────────┬────────────┘
                                    │
                        ┌───────────┴────────────┐
                        │     DAY 05: POWER      │
                        │(P = V × I & Deep Sleep)│
                        └───────────┬────────────┘
                                    │
                        ┌───────────┴────────────┐
                        │   DAY 04: RESISTANCE   │
                        │(Ohm's Law: V = I × R)  │
                        └───────────┬────────────┘
                                    │
            ┌───────────────────────┴───────────────────────┐
            │                                               │
┌───────────┴────────────┐                     ┌────────────┴───────────┐
│     DAY 02: VOLTAGE    │                     │     DAY 03: CURRENT    │
│ (3.3V Logic & Limits)  │                     │(Load Demand & GPIO max)│
└────────────────────────┘                     └────────────────────────┘
```

Setiap kali Anda menancapkan satu kabel jumper pada proyek IoT, keenam hukum fisika di atas bekerja secara simultan!

---

## 2. Peta Keselamatan Pinout ESP32 (The ESP32 Pinout Safety Matrix)

Salah satu kesalahan paling sering yang membuat board ESP32 tidak bisa di-flash atau gagal booting adalah salah memilih pin GPIO. Simpan tabel ini sebagai referensi utama Anda:

```text
┌──────────────┬──────────────────┬────────────────────────────────────────────────────────┐
│ Kategori Pin │ Daftar GPIO      │ Aturan Pemakaian & Catatan Keamanan                     │
├──────────────┼──────────────────┼────────────────────────────────────────────────────────┤
│ 🟢 SANGAT    │ GPIO 16, 17, 18, │ Pin GPIO digital standar terbaik. Bebas digunakan     │
│    AMAN      │ GPIO 19, 21, 22, │ untuk output LED, Relay, Trigger ultrasonik, dll.      │
│              │ GPIO 23          │ Pin 21 (SDA) dan 22 (SCL) adalah pin default bus I2C.  │
├──────────────┼──────────────────┼────────────────────────────────────────────────────────┤
│ 🟡 INPUT-ONLY│ GPIO 34, 35,     │ Pin ini TIDAK memiliki sirkuit output atau pull-up     │
│    (ADC1)    │ GPIO 36 (VP),    │ internal! Hanya bisa digunakan sebagai INPUT analog    │
│              │ GPIO 39 (VN)     │ atau digital. SANGAT COCOK untuk sensor analog/tanah. │
├──────────────┼──────────────────┼────────────────────────────────────────────────────────┤
│ 🔴 STRAPPING │ GPIO 0, GPIO 2,  │ HATI-HATI! Pin ini menentukan mode booting chip saat   │
│    PINS      │ GPIO 12, GPIO 15 │ menyala (Flash mode vs Run mode). Jika pin 0 atau 12   │
│              │                  │ ditarik ke tegangan tertentu saat booting, ESP32 GAGAL │
│              │                  │ MENYALA! Hindari memasang sakelar/aktuator di pin ini. │
├──────────────┼──────────────────┼────────────────────────────────────────────────────────┤
│ ⚠️ ADC2      │ GPIO 0, 2, 4, 12,│ Jangan gunakan untuk sensor analog jika menyalakan     │
│    CONFLICT  │ 13, 14, 15, 25-27│ modul Wi-Fi! Kanal ADC2 dimatikan oleh driver Wi-Fi.   │
├──────────────┼──────────────────┼────────────────────────────────────────────────────────┤
│ ⛔ TERLARANG │ GPIO 6 s/d 11    │ JANGAN PERNAH DISENTUH! Pin ini terhubung langsung ke  │
│    (FLASH)   │ (CLK, SD0-SD3)   │ chip memori SPI Flash internal. Menghubungkannya akan   │
│              │                  │ menyebabkan ESP32 crash instan (*Kernel Panic*).       │
└──────────────┴──────────────────┴────────────────────────────────────────────────────────┘
```

---

## 3. Matriks Diagnosa Masalah IoT (Master Troubleshooting Matrix)

Ketika proyek IoT Anda mengalami masalah di masa depan, gunakan alur diagnosa sistematis ini dari lapisan terbawah (fisik) ke atas (software):

| Gejala Masalah | Kemungkinan Akar Masalah | Solusi Berdasarkan Fondasi Week 01 |
|---|---|---|
| ESP32 me-restart sendiri berulang-ulang (*Boot Loop / Brownout*) | Tegangan input drop saat pemancar Wi-Fi aktif karena kabel USB jelek atau arus kurang | Gunakan kabel data tebal, pastikan adaptor minimal 5V 1A, pasang kapasitor decoupling $100\mu\text{F}$ di dekat pin VIN dan GND |
| Chip ESP32 atau regulator AMS1117 sangat panas | Diberi tegangan input terlalu tinggi (misal 12V langsung ke VIN) | Ganti linear supply dengan Switching Buck Converter (Step-Down DC-DC) untuk menurunkan 12V ke 5V dingin |
| Motor/Servo bergerak tersendat-sendat atau membuat ESP32 freeze | Beban motor ditenagai langsung dari pin 3.3V ESP32 | Pisahkan catu daya aktuator menggunakan adaptor/baterai eksternal, dan gabungkan kabel GND-nya (*Common Ground*) |
| Sensor analog membaca nilai 4095 konstan setelah `WiFi.begin()` | Sensor dipasang pada kanal ADC2 yang bentrok dengan Wi-Fi | Pindahkan kabel sinyal sensor analog ke salah satu pin ADC1 (GPIO32, 33, 34, 35) |
| Layar LCD I2C macet atau tidak terdeteksi saat `i2c_scanner` | Jalur SDA/SCL kehilangan tegangan referensi HIGH | Periksa pemasangan resistor Pull-Up ($4.7\text{k}\Omega$) pada jalur SDA dan SCL ke 3.3V |
| Pin GPIO mati total tidak merespons input/output lagi | Pin pernah terkena tegangan $>3.6\text{V}$ atau arus korsleting $>40\text{mA}$ | Selalu gunakan resistor pembatas arus ($220\Omega$) pada LED dan gunakan Voltage Divider untuk sensor 5V |

---

## 4. Proyek Blueprint Integrasi Minggu 1: Smart Greenhouse Node

Sebagai evaluasi mandiri akhir pekan, rancanglah cetak biru pengkabelan dan alokasi pin untuk sistem **Smart Greenhouse (Stasiun Kebun Cerdas)** yang menggabungkan seluruh komponen yang Anda miliki:

### Spesifikasi Kebutuhan Proyek:
1. **Sensor Suhu & Kelembapan Udara (DHT22):** Membaca data cuaca lingkungan.
2. **Sensor Kelembapan Tanah (Capacitive Soil Moisture):** Membaca tingkat kekeringan media tanam secara analog.
3. **Aktuator Pompa Air (Modul Relay 5V):** Mengaktifkan pompa penyiraman jika tanah kering.
4. **Tampilan Informasi (LCD I2C 16x2):** Menampilkan status kebun secara langsung di alat.
5. **Indikator Status (LED Merah & Hijau):** LED Hijau = Normal/Wi-Fi connected; LED Merah = Error/Penyiraman aktif.
6. **Konektivitas:** Wi-Fi aktif mengirimkan data ke web dashboard setiap menit.

### 📋 Tugas Perancangan Anda:
Petakan alokasi pin, proteksi listrik, dan kebutuhan dayanya pada dokumen `system-blueprint.md`:
- Alokasi pin GPIO yang aman dan bebas konflik Wi-Fi.
- Nilai resistor pembatas arus yang diperlukan untuk masing-masing LED.
- Skema catu daya dan kalkulasi Power Budget.

---

## 5. 🧠 Tantangan Evaluasi Akhir Minggu (Exam Questions)

Jawablah 3 pertanyaan komprehensif ini untuk membuktikan penguasaan penuh Anda terhadap materi Minggu 1:
1. **Sintesis Hukum Listrik:** Sebuah modul sensor gas membutuhkan tegangan operasi 5.0V dan menarik arus stabil sebesar $150\text{ mA}$. Jika Anda menyuplai modul ini dari aki motor 12V menggunakan linear regulator LM7805, hitunglah:
   - Berapa disipasi daya panas yang dibuang oleh regulator? ($P = V_{drop} \times I$)
   - Apakah regulator tersebut aman tanpa heatsink (batas aman tanpa heatsink adalah $\le 0.5\text{ W}$)?
   - Apa rekomendasi rekayasa Anda untuk sistem ini?
2. **Pencegahan Brownout & Strapping Pin:** Mengapa menyambungkan relay 5V ke pin GPIO12 atau GPIO0 pada ESP32 sering kali menyebabkan ESP32 menolak booting atau masuk ke *download mode* secara acak saat dinyalakan?
3. **Arsitektur Sinyal:** Jelaskan perbedaan mendasar dalam pengolahan firmware antara membaca nilai dari pin DO (Digital Output) sensor TCRT5000 vs membaca pin AO (Analog Output)-nya pada ESP32! Kapan kita menggunakan DO, dan kapan kita wajib menggunakan AO?

---

## 6. 📚 Panduan Dokumentasi GitHub

Buat dua file dokumentasi di:
- `month-01/week-01/day-07/system-blueprint.md` (Cetak biru arsitektur hardware lengkap)
- `month-01/week-01/day-07/README.md` (Dokumentasi evaluasi mingguan)

Format standar `README.md`:
```markdown
# Day 07 — Week 01 Comprehensive Review & System Blueprint

## 🎯 Objective
Melakukan evaluasi menyeluruh terhadap fondasi kelistrikan, logika level ESP32, batasan arus, disipasi daya, sinyal digital/analog, serta menghasilkan cetak biru sistem IoT pertama yang teruji secara engineering.

## 📚 What I Synthesized in Week 01
- Hubungan terpadu antara Voltage, Current, Resistance, dan Power ($V=IR$ dan $P=VI$).
- Pemetaan zona aman Pinout ESP32 (menghindari strapping pins, memprioritaskan ADC1 untuk sensor analog).
- Matriks diagnosa masalah (*Troubleshooting Matrix*) untuk mendeteksi akar penyebab kegagalan sistem IoT.
- Prinsip integrasi multi-komponen dengan Common Ground dan pemisahan catu daya aktuator.

## 📐 System Blueprint — Smart Greenhouse Node
[Rangkuman cetak biru dari system-blueprint.md]

## 🧠 Comprehensive Exam Answers
[Jawaban analitis ketiga soal evaluasi akhir minggu]

## 📝 Milestone Retrospective
[Refleksi pembelajaran selama 7 hari: konsep apa yang paling membuka wawasan Anda?]
```

---

## 7. Git Workflow Week-01 Milestone

Setelah seluruh dokumentasi selesai, lakukan update pada root `README.md` dan push commit milestone:
```bash
git status
git add month-01/week-01/ README.md
git commit -m "feat(week-01): complete all 7 days of IoT fundamentals with master blueprint"
git push origin main
```

---

## ✅ Checklist Kelulusan Week 01 — IoT Fundamentals
- [ ] Lulus Day 01: Memahami esensi IoT vs Embedded System.
- [ ] Lulus Day 02: Memahami Beda Potensial, Logic 3.3V, dan Voltage Divider.
- [ ] Lulus Day 03: Memahami Batasan Arus GPIO dan Pemisahan Daya Beban Besar.
- [ ] Lulus Day 04: Menguasai Hukum Ohm dan Resistor Pull-Up bus komunikasi.
- [ ] Lulus Day 05: Menguasai Audit Daya Panas dan Manajemen Deep Sleep Baterai.
- [ ] Lulus Day 06: Menguasai Periferal ADC 12-bit (ADC1) dan Kontrol PWM.
- [ ] Lulus Day 07: Menyelesaikan Cetak Biru Sistem Terpadu (*system-blueprint.md*).
- [ ] Berhasil melakukan push progress lengkap 1 minggu ke GitHub!

---

## 🎯 Prinsip Penutup Minggu 01
> **"Seorang programmer IoT yang hebat bukan hanya mereka yang mahir mengetik kode di editor, melainkan mereka yang memahami dengan intim hukum fisika di mana kode tersebut dieksekusi."**
