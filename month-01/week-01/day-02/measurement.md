# 📊 Measurement Log — Day 02 (Voltage)

Catatan hasil pengukuran tegangan DC menggunakan Multimeter Digital.

## 🛠️ Alat & Instrumen
- **Multimeter:** Digital Multimeter
- **Mode Multimeter:** DC Voltage (DCV ⎓) Range 20V
- **Board:** ESP32 DevKit V1 (30-pin)
- **Sumber Daya:** Port USB PC / Laptop (5V DC)

---

## 📋 Tabel Hasil Pengukuran

| Titik Ukur (Probe Merah) | Titik Referensi (Probe Hitam) | Tegangan Tertera (Nominal) | Hasil Ukur Nyata (Aktual) | Keterangan & Status |
|---|---|---|---|---|
| Pin VIN (5V) | Pin GND | 5.00 V | 4.96 V - 5.08 V | Normal (tegangan output dari port USB PC) |
| Pin 3V3 | Pin GND | 3.30 V | 3.28 V - 3.31 V | Sangat stabil (output IC regulator onboard LDO) |
| Output Baterai AA (Baru) | Kutub Negatif Baterai | 1.50 V | ~1.58 V - 1.60 V | Kondisi penuh |
| Baterai 18650 Li-ion | Kutub Negatif Baterai | 3.70 V | ~4.15 V - 4.20 V | Kondisi terisi penuh (Full Charge) |
| Pin Echo HC-SR04 (Direct) | Pin GND | 5.00 V (Pulsa) | ~5.00 V | ⚠️ Berbahaya jika masuk langsung ke GPIO ESP32 |
| Output Voltage Divider | Pin GND | 3.33 V | ~3.31 V | ✅ Aman masuk ke pin GPIO ESP32 |

---

## 🔍 Catatan & Temuan
1. Output pin 3V3 ESP32 sangat presisi karena dikontrol oleh IC linear regulator.
2. Tegangan port USB laptop bisa sedikit berfluktuasi tergantung kualitas kabel data USB dan beban sistem komputer.
3. Selalu pastikan probe hitam berada di GND sebelum menempelkan probe merah ke titik yang diuji untuk mencegah kesalahan pembacaan beda potensial.
