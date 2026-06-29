# ESP32 / ESP8266 Examples

Get your device's public IP address using IPPubblico.org — no API key required.

## Basic example — get IPv4 address

```cpp
#include <WiFi.h>
#include <HTTPClient.h>

String getPublicIP() {
  HTTPClient http;
  http.begin("https://ipv4.ippubblico.org/");
  int httpCode = http.GET();

  String ip = "";
  if (httpCode == 200) {
    ip = http.getString();
    ip.trim();
  }
  http.end();
  return ip;
}

void setup() {
  Serial.begin(115200);
  WiFi.begin("your-ssid", "your-password");

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
  }

  String publicIP = getPublicIP();
  Serial.println("Public IP: " + publicIP);
}

void loop() {
  delay(60000); // check every 60 seconds
  Serial.println("Public IP: " + getPublicIP());
}
```

## DDNS updater example

```cpp
#include <WiFi.h>
#include <HTTPClient.h>

String lastKnownIP = "";

String getPublicIP() {
  HTTPClient http;
  http.begin("https://ipv4.ippubblico.org/");
  int httpCode = http.GET();

  if (httpCode == 429) {
    // Rate limited — read Retry-After header
    String retryAfter = http.header("Retry-After");
    int waitSec = retryAfter.toInt();
    http.end();
    delay(waitSec * 1000);
    return "";
  }

  String ip = "";
  if (httpCode == 200) {
    ip = http.getString();
    ip.trim();
  }
  http.end();
  return ip;
}

void checkIPChange() {
  String currentIP = getPublicIP();
  if (currentIP.length() == 0) return;

  if (currentIP != lastKnownIP) {
    Serial.println("IP changed: " + lastKnownIP + " -> " + currentIP);
    lastKnownIP = currentIP;
    // Add your DDNS update logic here
  }
}

void setup() {
  Serial.begin(115200);
  WiFi.begin("your-ssid", "your-password");
  while (WiFi.status() != WL_CONNECTED) delay(500);
  checkIPChange();
}

void loop() {
  delay(60000); // check every 60 seconds
  checkIPChange();
}
```

## Full geolocation (JSON)

```cpp
#include <WiFi.h>
#include <HTTPClient.h>
#include <ArduinoJson.h>

void getIPInfo() {
  HTTPClient http;
  http.begin("https://ippubblico.org/?api=1");
  int httpCode = http.GET();

  if (httpCode == 200) {
    String payload = http.getString();
    JsonDocument doc;
    deserializeJson(doc, payload);

    Serial.println("IP: " + String(doc["ip"].as<const char*>()));
    Serial.println("Country: " + String(doc["geo"]["country"].as<const char*>()));
    Serial.println("City: " + String(doc["geo"]["city"].as<const char*>()));
    Serial.println("ISP: " + String(doc["isp"].as<const char*>()));
  }
  http.end();
}
```

## Notes

- Poll every **60 seconds** minimum for DDNS use cases
- The API allows one request per 10 seconds per IP
- Works on ESP32 and ESP8266 with the same code
- Requires `ArduinoJson` library for the JSON example
