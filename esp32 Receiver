
```
ESP32 Sensor
(MQ6 + MQ7 + DHT22)
        │
     ESP-NOW
        │
        ▼
ESP32 Gateway
        │
   ├── Relay Fan
   ├── Relay Alarm
   ├── WiFi
   └── Blynk
```

Receiver ini akan:

* menerima data ESP-NOW
* mengirim ke Blynk
* mengontrol relay
* mendeteksi sensor offline
* reconnect otomatis ke WiFi/Blynk

---

# Wiring Receiver

| Perangkat   | GPIO   |
| ----------- | ------ |
| Relay Fan   | GPIO18 |
| Relay Alarm | GPIO19 |

Tidak ada sensor lagi di receiver.

---

# Program Lengkap ESP32 Receiver

```cpp
#define BLYNK_TEMPLATE_ID "TMPL64q0Aj8z4"
#define BLYNK_TEMPLATE_NAME "ChickAlert"
#define BLYNK_AUTH_TOKEN "RTbT15XMvSepKBhk_UvLYrNebSht-giX"

#include <WiFi.h>
#include <esp_now.h>
#include <BlynkSimpleEsp32.h>

char ssid[] = "celsiii_tasiaaa";
char pass[] = "ftui1234";

#define RELAY_FAN    18
#define RELAY_ALARM  19

typedef struct
{
    float lpg;
    float co;
    float temp;
    bool danger;

} SensorData;

SensorData incomingData;

unsigned long lastReceive = 0;

BlynkTimer timer;
```

---

## Callback Receive

```cpp
void OnDataRecv(
        const esp_now_recv_info_t *recv_info,
        const uint8_t *incomingDataBytes,
        int len)
{

    memcpy(
        &incomingData,
        incomingDataBytes,
        sizeof(incomingData)
    );

    lastReceive = millis();

    Serial.println("--------------------");

    Serial.print("LPG : ");
    Serial.println(incomingData.lpg);

    Serial.print("CO : ");
    Serial.println(incomingData.co);

    Serial.print("TEMP : ");
    Serial.println(incomingData.temp);

    Serial.print("Danger : ");
    Serial.println(incomingData.danger);

}
```

---

## Kirim ke Blynk

```cpp
void sendBlynk()
{

    if(Blynk.connected())
    {

        Blynk.virtualWrite(V0,incomingData.lpg);

        Blynk.virtualWrite(V1,incomingData.co);

        Blynk.virtualWrite(V2,incomingData.temp);

        Blynk.virtualWrite(V3,incomingData.danger);

    }

}
```

---

## Kontrol Relay

```cpp
void controlRelay()
{

    if(incomingData.danger)
    {

        digitalWrite(RELAY_FAN,HIGH);

        digitalWrite(RELAY_ALARM,HIGH);

    }

    else
    {

        digitalWrite(RELAY_FAN,LOW);

        digitalWrite(RELAY_ALARM,LOW);

    }

}
```

---

## Cek Sensor Offline

Kalau lebih dari 5 detik tidak menerima paket.

```cpp
void checkOffline()
{

    if(millis()-lastReceive>5000)
    {

        Serial.println("Sensor Offline");

        digitalWrite(RELAY_FAN,LOW);

        digitalWrite(RELAY_ALARM,LOW);

        if(Blynk.connected())
        {
            Blynk.virtualWrite(V3,0);
        }

    }

}
```

---

## Reconnect WiFi

```cpp
void reconnectWiFi()
{

    if(WiFi.status()!=WL_CONNECTED)
    {

        Serial.println("Reconnect WiFi");

        WiFi.begin(ssid,pass);

    }

}
```

---

## Setup

```cpp
void setup()
{

    Serial.begin(115200);

    pinMode(RELAY_FAN,OUTPUT);
    pinMode(RELAY_ALARM,OUTPUT);

    digitalWrite(RELAY_FAN,LOW);
    digitalWrite(RELAY_ALARM,LOW);

    WiFi.mode(WIFI_STA);

    WiFi.begin(ssid,pass);

    Blynk.config(BLYNK_AUTH_TOKEN);

    while(WiFi.status()!=WL_CONNECTED)
    {
        delay(500);
        Serial.print(".");
    }

    Blynk.connect();

    if(esp_now_init()!=ESP_OK)
    {
        Serial.println("ESP NOW ERROR");
        return;
    }

    esp_now_register_recv_cb(OnDataRecv);

    timer.setInterval(2000L,sendBlynk);

}
```

---

## Loop

```cpp
void loop()
{

    Blynk.run();

    timer.run();

    reconnectWiFi();

    controlRelay();

    checkOffline();

}
```

---

# Cara Kerja Keseluruhan

```
ESP32 SENSOR

MQ6
MQ7
DHT22
     │
     ▼
Hitung ppm
     │
     ▼
Struct SensorData
     │
     ▼
ESP-NOW
     │
     │
──────────────
     │
     ▼
ESP32 RECEIVER
     │
     ├── Terima Struct
     │
     ├── Relay Fan
     │
     ├── Relay Alarm
     │
     └── Kirim ke Blynk
```

---

# Flow Program

```
Boot

↓

ESP-NOW aktif

↓

Menunggu paket

↓

Paket datang

↓

Copy ke Struct

↓

Update LCD/Blynk (opsional)

↓

Relay ON/OFF

↓

Menunggu paket berikutnya
```

---

# Need to be improve

Kalau ini akan dipakai sebagai **proyek tugas akhir atau penelitian**, saya menyarankan beberapa peningkatan agar sistem lebih andal:

1. **Tambahkan ID node** di dalam `struct`, sehingga satu receiver bisa menerima data dari beberapa ESP32 sensor.

   ```cpp
   typedef struct {
       uint8_t nodeID;
       float lpg;
       float co;
       float temp;
       bool danger;
   } SensorData;
   ```

2. **Tambahkan nomor urut paket (sequence number)** untuk mengetahui apakah ada paket yang hilang.

3. **Tambahkan checksum/CRC** untuk memverifikasi integritas data (meskipun ESP-NOW sudah memiliki pemeriksaan kesalahan bawaan di level protokol).

4. **Gunakan `millis()` pada transmitter** daripada `delay(2000)`, sehingga nanti Anda bisa menambahkan fitur lain tanpa membuat ESP32 berhenti selama 2 detik.

Dengan empat perubahan itu, kualitas proyek Anda akan jauh lebih baik dan lebih mudah dikembangkan di masa depan.
