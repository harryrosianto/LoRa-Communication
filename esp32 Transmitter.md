---

# Arsitektur

```text
ESP32 SENSOR
│
├── MQ6
├── MQ7
├── DHT22
│
└── ESP-NOW
        │
        │
        ▼
ESP32 RECEIVER
        │
        ├── Relay Fan
        ├── Relay Alarm
        └── Blynk
```

---

# Wiring ESP32 Sensor

| Sensor     | ESP32  |
| ---------- | ------ |
| MQ6 AO     | GPIO32 |
| MQ7 AO     | GPIO33 |
| DHT22 DATA | GPIO4  |
| DHT22 VCC  | 3.3V   |
| DHT22 GND  | GND    |

---

# Step 1. Ambil MAC Address Receiver

Upload dulu program ini ke ESP32 receiver.

```cpp
#include <WiFi.h>

void setup() {

  Serial.begin(115200);

  WiFi.mode(WIFI_STA);

  Serial.println(WiFi.macAddress());

}

void loop() {}
```

Misalnya keluar

```
3C:E9:0E:15:8A:44
```

Nanti kita isi di transmitter.

---

# Step 2. Program ESP32 Sensor (Transmitter)

```cpp
#include <WiFi.h>
#include <esp_now.h>
#include <DHT.h>

#define MQ6_PIN 32
#define MQ7_PIN 33

#define DHTPIN 4
#define DHTTYPE DHT22

DHT dht(DHTPIN, DHTTYPE);

typedef struct
{
  float lpg;
  float co;
  float temp;
  bool danger;

} SensorData;

SensorData dataSend;
```

---

## MAC Address Receiver

Ganti sesuai hasil Step 1.

```cpp
uint8_t receiverMAC[] =
{
  0x3C,
  0xE9,
  0x0E,
  0x15,
  0x8A,
  0x44
};
```

---

## Threshold

```cpp
#define LPG_LIMIT 100
#define CO_LIMIT 50
```

---

## Fungsi Averaging

```cpp
int readAverage(int pin)
{
  long total = 0;

  for(int i=0;i<20;i++)
  {
    total += analogRead(pin);
    delay(5);
  }

  return total/20;
}
```

---

## MQ6

Sama seperti program Anda.

```cpp
float mq6ToPPM(int adc)
{

  float ppm = ((adc - 500) * 100.0) / 200.0;

  if(ppm<0)
    ppm=0;

  return ppm;

}
```

---

## MQ7

```cpp
float mq7ToPPM(int adc)
{

  float ppm = ((adc - 3000) * 50.0) / 1095.0;

  if(ppm<0)
    ppm=0;

  return ppm;

}
```

---

## Callback Pengiriman

Supaya tahu data berhasil dikirim.

```cpp
void OnDataSent(const uint8_t *mac_addr, esp_now_send_status_t status)
{

  Serial.print("Send Status : ");

  if(status==ESP_NOW_SEND_SUCCESS)
    Serial.println("SUCCESS");
  else
    Serial.println("FAILED");

}
```

---

## Setup

```cpp
void setup()
{

  Serial.begin(115200);

  dht.begin();

  WiFi.mode(WIFI_STA);

  if(esp_now_init()!=ESP_OK)
  {
    Serial.println("ESP NOW ERROR");
    return;
  }

  esp_now_register_send_cb(OnDataSent);

  esp_now_peer_info_t peerInfo = {};

  memcpy(peerInfo.peer_addr, receiverMAC, 6);

  peerInfo.channel = 0;

  peerInfo.encrypt = false;

  if(esp_now_add_peer(&peerInfo)!=ESP_OK)
  {
      Serial.println("Peer Error");
      return;
  }

}
```

---

## Loop

```cpp
void loop()
{

  int mq6ADC = readAverage(MQ6_PIN);

  int mq7ADC = readAverage(MQ7_PIN);

  float temp = dht.readTemperature();

  dataSend.lpg = mq6ToPPM(mq6ADC);

  dataSend.co = mq7ToPPM(mq7ADC);

  dataSend.temp = temp;

  dataSend.danger =
  (
      dataSend.lpg>=LPG_LIMIT ||
      dataSend.co>=CO_LIMIT
  );

  esp_now_send(
      receiverMAC,
      (uint8_t *) &dataSend,
      sizeof(dataSend)
  );

  Serial.println("----------------------");

  Serial.print("LPG : ");

  Serial.println(dataSend.lpg);

  Serial.print("CO : ");

  Serial.println(dataSend.co);

  Serial.print("TEMP : ");

  Serial.println(dataSend.temp);

  Serial.print("DANGER : ");

  Serial.println(dataSend.danger);

  delay(2000);

}
```

---

# Yang dilakukan ESP32 ini

Setiap 2 detik

```
MQ6
↓

MQ7
↓

DHT22
↓

Hitung ppm
↓

Cek danger
↓

Buat struct

{
 LPG
 CO
 TEMP
 Danger
}

↓

ESP-NOW

↓

ESP32 Receiver
```

---

# Kenapa pakai Struct?

Daripada mengirim

```
120.4,52.1,31.5,1
```

kita mengirim langsung isi memorinya.

```
float
float
float
bool
```
