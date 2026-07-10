# Program ESP32

Upload kode berikut.

```cpp
unsigned long lastTime = 0;

void setup() {
  Serial.begin(115200);
}

void loop() {

  if (millis() - lastTime >= 1000) {

    lastTime = millis();

    int value1 = random(0, 100);
    int value2 = random(20, 40);
    float value3 = random(200, 350) / 10.0;

    Serial.print(value1);
    Serial.print(",");
    Serial.print(value2);
    Serial.print(",");
    Serial.println(value3);

  }

}
```

Output Serial Monitor:

```text
56,31,27.8
72,28,25.4
14,35,30.6
89,22,24.7
```

---

# Mengaktifkan Data Streamer di Excel

1. Buka **Microsoft Excel**.
2. Klik **File → Options → Add-ins**.
3. Pilih **COM Add-ins**.
4. Centang **Data Streamer**.
5. Restart Excel jika diminta.

Setelah aktif akan muncul tab:

```text
Data Streamer
```

---

# Menghubungkan ke ESP32

1. Tutup **Serial Monitor** di Arduino IDE (port serial tidak boleh dipakai dua aplikasi sekaligus).
2. Buka Excel.
3. Buka tab **Data Streamer**.
4. Klik **Connect Device**.
5. Pilih COM port ESP32.
6. Klik **Start Data**.

---

# Format Data

Data Streamer membaca data yang dipisahkan koma.

Misalnya:

```text
10,20,30
```

akan menjadi:

| A  | B  | C  |
| -- | -- | -- |
| 10 | 20 | 30 |

Kalau mengirim:

```text
10,20,30
11,21,31
12,22,32
```

maka Excel akan terus menambahkan baris baru.

---

# Contoh Simulasi Sensor

Misalnya ingin mensimulasikan tiga sensor:

```cpp
void setup() {

  Serial.begin(115200);

  randomSeed(analogRead(0));

}

void loop() {

  float suhu = random(250,350)/10.0;
  float kelembaban = random(500,900)/10.0;
  float gas = random(0,100);

  Serial.print(suhu);
  Serial.print(",");
  Serial.print(kelembaban);
  Serial.print(",");
  Serial.println(gas);

  delay(1000);

}
```

Output:

```text
28.4,72.3,14
29.1,70.8,16
30.0,69.5,18
```

Di Excel:

| Suhu | Kelembaban | Gas |
| ---: | ---------: | --: |
| 28.4 |       72.3 |  14 |
| 29.1 |       70.8 |  16 |
| 30.0 |       69.5 |  18 |

---

# Membuat Grafik Otomatis

Setelah data masuk ke Excel:

1. Blok kolom yang berisi data.
2. Pilih **Insert → Line Chart**.
3. Saat Data Streamer menambahkan baris baru, grafik akan ikut diperbarui jika sumber datanya berupa **Excel Table** (Ctrl + T) atau rentang dinamis.

---

## Saran untuk pembelajaran

Kalau tujuan akhirnya adalah proyek **ESP32 → Excel → Analisis Data**, saya biasanya menyusun latihan bertahap:

1. **ESP32 → Serial Monitor** (memastikan data terkirim dengan benar).
2. **ESP32 → Excel Data Streamer** (mencatat data langsung ke spreadsheet).
3. **ESP32 + Sensor → Excel** (mengirim data sensor nyata seperti DHT22 atau MQ).
4. **ESP32 → Excel Dashboard** (grafik dan indikator real-time).
5. **ESP32 → Excel + Blynk** (monitoring lokal di Excel dan jarak jauh di Blynk secara bersamaan).

Urutan ini memudahkan proses belajar dan debugging karena setiap tahap bisa diuji secara terpisah sebelum sistem menjadi lebih kompleks.
