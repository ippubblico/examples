# Arduino Examples

For Arduino boards with WiFi (Arduino Uno R4 WiFi, Arduino Nano 33 IoT, MKR WiFi 1010).

## Get public IP

```cpp
#include <WiFiNINA.h>  // for Nano 33 IoT, MKR WiFi
// #include <WiFi.h>   // for Uno R4 WiFi

#include <ArduinoHttpClient.h>

const char* ssid     = "your-ssid";
const char* password = "your-password";

WiFiSSLClient wifiClient;
HttpClient    httpClient(wifiClient, "ipv4.ippubblico.org", 443);

String getPublicIP() {
  httpClient.get("/");
  int statusCode = httpClient.responseStatusCode();
  String response = httpClient.responseBody();

  if (statusCode == 200) {
    response.trim();
    return response;
  }
  return "";
}

void setup() {
  Serial.begin(9600);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) delay(500);

  String ip = getPublicIP();
  Serial.println("Public IP: " + ip);
}

void loop() {
  delay(60000);
  Serial.println("Public IP: " + getPublicIP());
}
```

## Required libraries

- `WiFiNINA` (for Nano 33 IoT, MKR WiFi 1010)
- `ArduinoHttpClient`

Install via Arduino Library Manager.

## Notes

- For ESP32/ESP8266, use the [esp32/](../esp32/) examples instead — simpler HTTP client
- Poll every **60 seconds** minimum for DDNS use cases
