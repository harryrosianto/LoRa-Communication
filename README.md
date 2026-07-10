Kalau arsitekturnya seperti ini:

```
ESP32 #1 (Sensor)
MQ6 + MQ7 + DHT22
        │
        ▼
    LoRa SX1278/SX1276
        ))))
~~~~~~~~~ Udara ~~~~~~~~~
        ((((
    LoRa SX1278/SX1276
        │
        ▼
ESP32 #2 (Receiver)
        │
        ├── Relay
        ├── LCD (opsional)
        └── Blynk / PC / MQTT / dll
```

lebih baik **Blynk hanya ada di ESP32 receiver**. Jadi ESP32 sensor hanya membaca sensor lalu mengirim data lewat LoRa. Receiver menerima data lalu mengirim ke Blynk.

---

# STEP 1. Siapkan Hardware

Misalnya menggunakan **LoRa SX1278 RA-02 (433 MHz)**.

Hubungkan ke ESP32.

| LoRa   | ESP32  |
| ------ | ------ |
| VCC    | 3.3V   |
| GND    | GND    |
| SCK    | GPIO18 |
| MISO   | GPIO19 |
| MOSI   | GPIO23 |
| NSS/CS | GPIO5  |
| RST    | GPIO14 |
| DIO0   | GPIO26 |

Kalau pin berbeda tinggal diubah pada program.

---

# STEP 2. Install Library

Arduino IDE

Library Manager

Install

* LoRa by Sandeep Mistry

---

# STEP 3. Tambahkan Library

Pada ESP32 Sensor tambahkan

```cpp
#include <SPI.h>
#include <LoRa.h>
```

---

# STEP 4. Definisikan Pin LoRa

Di atas program

```cpp
#define LORA_SS   5
#define LORA_RST  14
#define LORA_DIO0 26
```

---

# STEP 5. Inisialisasi LoRa

Di dalam setup()

```cpp
SPI.begin(18,19,23,5);

LoRa.setPins(LORA_SS, LORA_RST, LORA_DIO0);

if (!LoRa.begin(433E6)) {
  Serial.println("LoRa gagal!");
  while(true);
}

Serial.println("LoRa Ready");
```

Kalau modul 915 MHz

```
915E6
```

Kalau 868

```
868E6
```

---

# STEP 6. Buat Format Data

Jangan kirim satu-satu.

Lebih enak dibuat seperti

```
LPG,CO,TEMP,DANGER
```

contoh

```
45.8,12.5,27.4,0
```

atau

```
120.5,53.2,31.6,1
```

---

# STEP 7. Kirim Data

Di akhir fungsi

```cpp
sendSensorData()
```

tambahkan

```cpp
String data = String(lpgPPM,1) + "," +
              String(coPPM,1) + "," +
              String(temp,1) + "," +
              String(danger);

LoRa.beginPacket();
LoRa.print(data);
LoRa.endPacket();

Serial.println("SEND : " + data);
```

Selesai.

ESP32 pertama sekarang sudah mengirim data setiap 2 detik.

---

# STEP 8. Program ESP32 Receiver

Program lengkap.

```cpp
#include <SPI.h>
#include <LoRa.h>

#define LORA_SS   5
#define LORA_RST  14
#define LORA_DIO0 26

float lpg;
float co;
float temp;
int danger;

void setup() {

  Serial.begin(115200);

  SPI.begin(18,19,23,5);

  LoRa.setPins(LORA_SS, LORA_RST, LORA_DIO0);

  if(!LoRa.begin(433E6))
  {
    Serial.println("LoRa Failed");
    while(true);
  }

  Serial.println("Receiver Ready");
}

void loop()
{

  int packetSize = LoRa.parsePacket();

  if(packetSize)
  {

    String incoming = "";

    while(LoRa.available())
    {
      incoming += (char)LoRa.read();
    }

    Serial.println(incoming);

    sscanf(incoming.c_str(),
           "%f,%f,%f,%d",
           &lpg,
           &co,
           &temp,
           &danger);

    Serial.println("======");
    Serial.print("LPG : ");
    Serial.println(lpg);

    Serial.print("CO : ");
    Serial.println(co);

    Serial.print("Temp : ");
    Serial.println(temp);

    Serial.print("Danger : ");
    Serial.println(danger);

  }

}
```

---

# STEP 9. Contoh Output Receiver

```
45.2,12.7,27.4,0

======
LPG : 45.2
CO : 12.7
Temp : 27.4
Danger : 0
```

Kalau

```
120.5,55.2,31.6,1
```

maka

```
Danger : 1
```

---

# STEP 10. Menyalakan Relay di Receiver

Misalnya relay di GPIO25.

```cpp
#define RELAY 25
```

setup()

```cpp
pinMode(RELAY,OUTPUT);
digitalWrite(RELAY,LOW);
```

Setelah parsing

```cpp
if(danger==1)
{
    digitalWrite(RELAY,HIGH);
}
else
{
    digitalWrite(RELAY,LOW);
}
```

---

# STEP 11. Kirim ke Blynk dari Receiver

Di receiver tinggal

```cpp
Blynk.virtualWrite(V0,lpg);
Blynk.virtualWrite(V1,co);
Blynk.virtualWrite(V2,temp);
Blynk.virtualWrite(V3,danger);
```

Jadi receiver bertugas:

```
LoRa
↓

Parsing

↓

Relay

↓

Blynk

↓

LCD (opsional)
```

---

# STEP 12. Agar Lebih Andal (Disarankan)

Daripada mengirim string CSV, gunakan format dengan penanda yang jelas:

```
LPG:120.5;CO:51.2;TEMP:30.1;D:1
```

atau lebih baik lagi kirim langsung sebuah `struct` (binary packet), misalnya:

```cpp
struct SensorData {
  float lpg;
  float co;
  float temp;
  bool danger;
};
```

Lalu di pengirim:

```cpp
SensorData data = {lpgPPM, coPPM, temp, danger};

LoRa.beginPacket();
LoRa.write((uint8_t*)&data, sizeof(data));
LoRa.endPacket();
```

Dan di penerima:

```cpp
SensorData data;

int packetSize = LoRa.parsePacket();
if (packetSize == sizeof(SensorData)) {
  LoRa.readBytes((uint8_t*)&data, sizeof(data));

  Serial.println(data.lpg);
  Serial.println(data.co);
  Serial.println(data.temp);
  Serial.println(data.danger);
}
```

Pendekatan `struct` lebih efisien, lebih cepat, dan mengurangi risiko kesalahan parsing dibanding mengirim data dalam bentuk teks CSV, terutama jika sistem akan dikembangkan menjadi proyek yang lebih kompleks.
