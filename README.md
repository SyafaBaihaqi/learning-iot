# 🚀 IoT Learning Journey

Repository ini berisi dokumentasi perjalanan belajar **Internet of Things (IoT)** secara bertahap, mulai dari dasar elektronika, mikrokontroler, sensor, aktuator, komunikasi, hingga pembuatan sistem IoT yang terintegrasi.

## 🎯 Tujuan Repository

Repository ini dibuat sebagai **learning repository**, bukan sekadar kumpulan source code.

Tujuan utama:

- Memahami konsep dasar IoT.
- Memahami dasar elektronika yang diperlukan untuk IoT.
- Menguasai penggunaan ESP32 dan mikrokontroler.
- Mempelajari sensor dan aktuator.
- Memahami komunikasi antarperangkat.
- Mempelajari konektivitas Wi-Fi dan Internet.
- Membuat sistem monitoring dan kontrol berbasis IoT.
- Membiasakan diri melakukan dokumentasi proyek secara terstruktur.
- Membangun portofolio pembelajaran IoT secara bertahap.

> **Prinsip belajar:** memahami konsep → mencoba → menguji → mendokumentasikan → mengevaluasi.

---

## 🧠 Learning Path

Materi dipelajari secara bertahap:

```text
Electronics
    ↓
Microcontroller
    ↓
GPIO
    ↓
Sensors
    ↓
Actuators
    ↓
Communication
    ↓
Wi-Fi
    ↓
Web / API
    ↓
IoT Architecture
    ↓
IoT Project
```

Konsep dasar IoT yang menjadi fondasi:

```text
        ┌──────────┐
        │  Sensor  │
        └────┬─────┘
             │
             ▼
        ┌──────────┐
        │ ESP32 /  │
        │ MCU      │
        └────┬─────┘
             │
       ┌─────┴─────┐
       ▼           ▼
   Processing   Communication
                   │
                   ▼
             Wi-Fi / Network
                   │
                   ▼
            Server / Web
                   │
                   ▼
             User / System
```

---

# 📁 Repository Structure

Struktur repository menggunakan pola:

```text
Month
└── Week
    └── Day
        └── Files
```

Contoh:

```text
iot-learning/
│
├── README.md
│
├── month-01/
│   ├── week-01/
│   │   ├── day-01/
│   │   │   ├── README.md
│   │   │   ├── notes.md
│   │   │   └── images/
│   │   │
│   │   ├── day-02/
│   │   │   ├── README.md
│   │   │   └── measurement.md
│   │   │
│   │   └── day-03/
│   │       ├── README.md
│   │       └── code/
│   │
│   ├── week-02/
│   │   ├── day-08/
│   │   └── ...
│   │
│   └── ...
│
├── month-02/
│   ├── week-01/
│   └── ...
│
├── month-03/
│   ├── week-01/
│   └── ...
│
└── assets/
    ├── images/
    ├── diagrams/
    └── references/
```

Struktur ini sengaja dibuat fleksibel sehingga jumlah hari tidak harus selalu sama.

---

# 📚 Learning Progress

## Month 01 — IoT & Electronics Fundamentals

Fokus:

- Pengenalan IoT
- Dasar listrik dan elektronika
- Voltage
- Current
- Resistance
- Power
- Digital dan analog
- VCC dan GND
- Multimeter
- Breadboard
- Resistor
- LED
- Dasar rangkaian elektronik

### Week 01 — IoT Fundamentals

- [x] [Day 01 — Introduction to IoT](month-01/week-01/day-01/)
- [x] [Day 02 — Voltage](month-01/week-01/day-02/)
- [x] [Day 03 — Current](month-01/week-01/day-03/)
- [x] [Day 04 — Resistance](month-01/week-01/day-04/)
- [x] [Day 05 — Power](month-01/week-01/day-05/)
- [x] [Day 06 — Digital vs Analog](month-01/week-01/day-06/)
- [x] [Day 07 — Weekly Review](month-01/week-01/day-07/)

### Week 02 — Basic Electronics

- [ ] Day 08 — Breadboard
- [ ] Day 09 — Resistor
- [ ] Day 10 — LED
- [ ] Day 11 — Pull-up & Pull-down
- [ ] Day 12 — Switch & Button
- [ ] Day 13 — Multimeter
- [ ] Day 14 — Weekly Project

### Week 03 — ESP32 Fundamentals

- [ ] Day 15 — Introduction to ESP32
- [ ] Day 16 — GPIO
- [ ] Day 17 — Digital Input
- [ ] Day 18 — Digital Output
- [ ] Day 19 — PWM
- [ ] Day 20 — ADC
- [ ] Day 21 — Weekly Project

### Week 04 — Basic ESP32 Project

- [ ] Day 22 — LED Control
- [ ] Day 23 — Button Control
- [ ] Day 24 — LED + Button
- [ ] Day 25 — Buzzer
- [ ] Day 26 — LCD
- [ ] Day 27 — Sensor Reading
- [ ] Day 28 — Mini Project

---

# 🔌 Month 02 — Sensors, Actuators & Communication

Fokus:

- Sensor
- Actuator
- Data acquisition
- Motor
- Servo
- Relay
- UART
- I2C
- SPI
- Bluetooth
- Wi-Fi

### Week 05 — Sensors

- [ ] Day 29 — Sensor Fundamentals
- [ ] Day 30 — HC-SR04
- [ ] Day 31 — DHT22
- [ ] Day 32 — Soil Moisture Sensor
- [ ] Day 33 — Water Level Sensor
- [ ] Day 34 — TCRT5000
- [ ] Day 35 — Sensor Mini Project

### Week 06 — Actuators

- [ ] Day 36 — Actuator Fundamentals
- [ ] Day 37 — Servo
- [ ] Day 38 — DC Motor
- [ ] Day 39 — Motor Driver
- [ ] Day 40 — Relay
- [ ] Day 41 — Buzzer
- [ ] Day 42 — Actuator Project

### Week 07 — Communication

- [ ] Day 43 — UART
- [ ] Day 44 — I2C
- [ ] Day 45 — SPI
- [ ] Day 46 — Bluetooth
- [ ] Day 47 — Wi-Fi
- [ ] Day 48 — Communication Comparison
- [ ] Day 49 — Communication Project

### Week 08 — ESP32 Integration

- [ ] Day 50 — Multiple Sensors
- [ ] Day 51 — Sensor + LCD
- [ ] Day 52 — Sensor + Actuator
- [ ] Day 53 — State Machine
- [ ] Day 54 — Error Handling
- [ ] Day 55 — Data Logging
- [ ] Day 56 — Mini Project

---

# 🌐 Month 03 — Networking, Web & IoT Architecture

Fokus:

- TCP/IP dasar
- HTTP
- REST API
- Web Server
- MQTT
- Database
- Dashboard
- IoT architecture
- Security dasar
- Final project

### Week 09 — Networking

- [ ] Day 57 — Network Fundamentals
- [ ] Day 58 — IP Address
- [ ] Day 59 — Router & Gateway
- [ ] Day 60 — DNS
- [ ] Day 61 — TCP vs UDP
- [ ] Day 62 — HTTP
- [ ] Day 63 — Network Project

### Week 10 — ESP32 Web & API

- [ ] Day 64 — ESP32 Wi-Fi
- [ ] Day 65 — Web Server
- [ ] Day 66 — HTTP Request
- [ ] Day 67 — REST API
- [ ] Day 68 — JSON
- [ ] Day 69 — API Integration
- [ ] Day 70 — Web Monitoring Project

### Week 11 — MQTT & IoT Architecture

- [ ] Day 71 — MQTT Fundamentals
- [ ] Day 72 — MQTT Broker
- [ ] Day 73 — Publish & Subscribe
- [ ] Day 74 — ESP32 MQTT
- [ ] Day 75 — IoT Architecture
- [ ] Day 76 — Data Flow
- [ ] Day 77 — IoT Security Basics

### Week 12 — Final Project

- [ ] Day 78 — Project Planning
- [ ] Day 79 — System Architecture
- [ ] Day 80 — Hardware Integration
- [ ] Day 81 — Firmware
- [ ] Day 82 — Network
- [ ] Day 83 — Web Dashboard
- [ ] Day 84 — Testing
- [ ] Day 85 — Debugging
- [ ] Day 86 — Documentation
- [ ] Day 87 — Optimization
- [ ] Day 88 — Final Testing
- [ ] Day 89 — Final Documentation
- [ ] Day 90 — Project Review

---

# 📄 Standard Day Structure

Setiap pembelajaran harian menggunakan struktur file yang konsisten.

```text
day-XX/
├── README.md
├── notes.md
├── code/
├── images/
└── result.md
```

Tidak semua folder harus digunakan setiap hari.

Untuk materi teori:

```text
day-XX/
└── README.md
```

Untuk materi praktik:

```text
day-XX/
├── README.md
├── code/
│   └── main.ino
├── images/
│   └── prototype.jpg
└── result.md
```

---

# 📝 Daily Documentation

Setiap `README.md` harian menggunakan format:

```markdown
# Day XX — [Judul Materi]

## 🎯 Objective

Apa yang ingin dipahami hari ini?

## 📚 Theory

Penjelasan teori yang dipelajari.

## 🔧 Components

Komponen yang digunakan.

## 🔌 Wiring

Hubungan antar-komponen.

## 💻 Code

Program yang digunakan.

## 🧪 Experiment

Percobaan yang dilakukan.

## 📊 Result

Hasil percobaan.

## 🐛 Troubleshooting

Masalah yang ditemukan dan solusinya.

## 🧠 Conclusion

Kesimpulan pembelajaran.

## 📌 What I Learned

Poin penting yang dipahami.

## 📸 Documentation

Foto atau diagram percobaan.
```

---

# 🔄 Learning Method

Setiap materi dipelajari menggunakan siklus:

```text
        ┌─────────────┐
        │    Learn    │
        └──────┬──────┘
               ↓
        ┌─────────────┐
        │   Practice  │
        └──────┬──────┘
               ↓
        ┌─────────────┐
        │    Test     │
        └──────┬──────┘
               ↓
        ┌─────────────┐
        │   Document  │
        └──────┬──────┘
               ↓
        ┌─────────────┐
        │   Review    │
        └──────┬──────┘
               │
               └──────────────→ Next Topic
```

Tujuannya bukan hanya **"kode berhasil dijalankan"**, tetapi memahami:

> **Mengapa rangkaian tersebut bekerja, bagaimana data mengalir, apa yang terjadi ketika terjadi kesalahan, dan bagaimana sistem dapat dikembangkan.**

---

# 🧰 Main Hardware

Perangkat utama yang digunakan dalam pembelajaran:

- ESP32 DevKit
- Arduino Uno
- HC-SR04
- DHT22
- TCRT5000
- Capacitive Soil Moisture Sensor
- Water Level Sensor
- SG90 Servo
- MG996R Servo
- DC Gear Motor
- L298N
- L9110S
- Relay
- LCD I2C 16x2
- LCD I2C 20x4
- OLED
- LED
- Buzzer
- MAX7219 LED Matrix
- Joystick
- Breadboard
- Multimeter

> Komponen yang digunakan disesuaikan dengan perangkat yang tersedia. Pembelian komponen tambahan dilakukan hanya jika memang diperlukan untuk materi tertentu.

---

# 💻 Development Tools

Tools utama:

- Arduino IDE
- Wokwi
- Git
- GitHub
- VS Code
- Serial Monitor

---

# 🌱 Learning Philosophy

Repository ini menggunakan pendekatan **progressive learning**.

Materi baru harus dibangun dari pemahaman sebelumnya.

Contohnya:

```text
Voltage
   ↓
Current
   ↓
Resistance
   ↓
Power
   ↓
Electronics
   ↓
ESP32
   ↓
Sensor
   ↓
Actuator
   ↓
Communication
   ↓
Network
   ↓
IoT
```

Dengan pendekatan ini, ketika sebuah proyek mengalami masalah, debugging tidak hanya dilakukan dengan mencoba-coba kode, tetapi dengan menganalisis:

```text
Hardware
   ↓
Power
   ↓
Wiring
   ↓
Signal
   ↓
Code
   ↓
Communication
   ↓
Application
```

---

# 📊 Progress

| Month    | Topic                              | Status         |
| -------- | ---------------------------------- | -------------- |
| Month 01 | Electronics & ESP32 Fundamentals   | 🟡 In Progress |
| Month 02 | Sensors, Actuators & Communication | ⚪ Not Started  |
| Month 03 | Networking, Web & IoT Architecture | ⚪ Not Started  |

---

# 🏆 Final Goal

Pada akhir learning journey, diharapkan mampu membuat sistem IoT yang memiliki:

```text
Sensor
   ↓
ESP32
   ↓
Data Processing
   ↓
Wi-Fi
   ↓
Server / API / MQTT
   ↓
Database
   ↓
Web Dashboard
```

serta sistem kontrol:

```text
Web / Mobile
      ↓
Network
      ↓
ESP32
      ↓
Actuator
      ↓
Physical Device
```

Target akhirnya bukan hanya mampu membuat satu proyek, tetapi memahami **fundamental IoT sehingga dapat merancang dan mengembangkan berbagai jenis sistem IoT secara mandiri.**

---

# 📌 Repository Status

**Learning Type:** IoT
**Duration:** Progressive Learning
**Main Platform:** ESP32
**Documentation:** Markdown
**Version Control:** Git & GitHub
**Project Status:** 🟡 Active Learning

---

## 🚀 Start Learning

Mulai dari:

```text
month-01/
└── week-01/
    └── day-01/
        └── README.md
```

**Learn → Build → Test → Document → Review → Improve**
