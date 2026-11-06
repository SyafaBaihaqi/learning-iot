# Day 01 — Introduction to IoT

## 🎯 Objective
Memahami konsep dasar Internet of Things (IoT), arsitektur dasar, dan perbedaannya dengan embedded system murni.

## 📚 What I Learned
- Pengertian dan esensi dasar Internet of Things (IoT).
- 4 Pilar utama IoT:
  1. **Sense (Input):** Mengambil data fisik lingkungan menggunakan sensor.
  2. **Process (Otak):** Mengolah data pada mikrokontroler (ESP32).
  3. **Communicate (Jaringan):** Mentransmisikan data/perintah melalui protokol jaringan (Wi-Fi, HTTP, MQTT).
  4. **Act (Output):** Menghasilkan tindakan fisik nyata (Relay, Motor, Servo) atau informasi visual.
- Perbedaan esensial antara Standalone Embedded System dengan Sistem Terkoneksi IoT.
- Pemetaan peran komponen hardware yang dimiliki (ESP32, DHT22, HC-SR04, Servo, Relay, L298N).

## 🔄 Basic IoT Architecture
```text
Sensor (DHT22 / HC-SR04)
        │
        ▼
Microcontroller (ESP32)
        │
        ▼
Network Layer (Wi-Fi / Router)
        │
        ▼
Server / Cloud Broker
        │
        ▼
Application Layer (Web Dashboard / Mobile App)
        │
        ▼
Actuator / Decision Maker
```

## 🔧 Hardware Components
- ESP32 DevKit V1
- DHT22 (Temperature & Humidity Sensor)
- HC-SR04 (Ultrasonic Distance Sensor)
- SG90 Servo Motor
- L298N Motor Driver + DC Gear Motor
- Relay Module 5V
- LCD I2C 16x2
- Multimeter Digital & Breadboard

## 🧪 Experiment & System Analysis

### Analisis Kasus 1: ESP32 + DHT22 + LCD I2C
- **Sensor:** DHT22 (mengukur suhu dan kelembapan udara).
- **Processing:** ESP32 (membaca sinyal digital dari sensor, konversi ke derajat Celsius dan persen).
- **Communication:** Hanya komunikasi bus lokal I2C ke LCD (tidak ada transmisi data jaringan).
- **Actuator/Output:** LCD I2C 16x2 menampilkan angka suhu.
- **Status:** **Embedded System Murni**, belum termasuk IoT karena tidak memiliki transmisi data jaringan/eksternal.

### Analisis Kasus 2: Smart Trash Can (ESP32 + HC-SR04 + Servo + Cloud Dashboard)
- **Sensor:** HC-SR04 (mendeteksi jarak objek/tangan pengguna mendekat).
- **Processing:** ESP32 (mengecek jika jarak < 20 cm, picu servo buka; kirim status kapasitas tempat sampah).
- **Communication:** Modul Wi-Fi internal ESP32 ke Web Server / Blynk Cloud via MQTT/REST API.
- **Actuator/Output:** Servo SG90 (membuka tutup fisik) + Visualisasi kapasitas di dashboard web.
- **Status:** **Sistem IoT Penuh**, karena terjadi integrasi antara penginderaan lokal, aktuasi fisik, dan pelaporan status nirkabel jarak jauh.

## 🧠 Questions & Answers
1. **Apakah ESP32 + DHT22 + LCD termasuk IoT?**
   *Tidak.* Sistem ini adalah *standalone embedded system*. Walaupun ESP32 memiliki chip Wi-Fi di dalamnya, jika fitur komunikasinya tidak digunakan untuk bertukar data dengan node/server lain di jaringan, maka ia hanya berfungsi sebagai mikrokontroler konvensional.
2. **Apakah perangkat IoT harus selalu menggunakan internet publik?**
   *Tidak harus selalu internet publik (World Wide Web).* Perangkat yang berkomunikasi di dalam Local Area Network (LAN) atau jaringan privat (intranet / local MQTT broker / edge server) tetap merupakan bagian dari IoT (*Local/Industrial IoT*), karena perangkat tersebut tetap terhubung ke layer jaringan dan bertukar data antar-sistem.
3. **Apa perbedaan mendasar antara IoT dan Embedded System?**
   *Embedded system* adalah sistem komputasi terdedikasi untuk fungsi tertentu yang sering kali bekerja mandiri dan tertutup. Sedangkan *IoT* adalah evolusi embedded system yang dilengkapi dengan kemampuan konektivitas jaringan untuk membentuk ekosistem pertukaran data, monitoring jarak jauh, dan kendali terdistribusi.

## 📝 Conclusion
1. Menggunakan ESP32 tidak otomatis membuat proyek kita menjadi IoT; arsitektur aliran data dan konektivitas jaringannyalah yang menentukan identitas sistem tersebut.
2. Fondasi kokoh sebuah sistem IoT berawal dari pemahaman yang baik terhadap sisi *embedded hardware* (kelistrikan, sensor, mikrokontroler) sebelum melangkah ke sisi *software & networking* (Wi-Fi, cloud, web).
