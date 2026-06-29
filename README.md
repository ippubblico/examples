# IPPubblico.org — Code Examples

Free public IP detection API. No API key. No registration. No rate limit for normal use.

**Base URL:** `https://ippubblico.org`

## Endpoints

| Endpoint | Response | Use case |
|----------|----------|----------|
| `https://ipv4.ippubblico.org/` | `203.0.113.42` | IPv4 only, plain text |
| `https://ipv6.ippubblico.org/` | `2001:db8::1` or `NONE` | IPv6 only, plain text |
| `https://ippubblico.org/?text=1` | `IPv4: x.x.x.x\nIPv6: x` | Both protocols |
| `https://ippubblico.org/?api=1` | JSON | Full geolocation |

## JSON Response (`?api=1`)

```json
{
  "status": "ok",
  "ip": "203.0.113.42",
  "ipv4": "203.0.113.42",
  "ipv6": null,
  "isp": "NTT Communications",
  "asn": "AS2914",
  "timezone": "Asia/Tokyo",
  "geo": {
    "city": "Tokyo",
    "region": "Tokyo",
    "country": "Japan",
    "country_code": "JP",
    "lat": 35.6893,
    "lon": 139.6899
  }
}
```

## Multilingual responses

Add `?lang=XX` to get city, region and country in the user's language:

```bash
curl "https://ippubblico.org/?api=1&lang=ja"
# "city": "東京", "country": "日本"

curl "https://ippubblico.org/?api=1&lang=zh"
# "city": "东京", "country": "日本"

curl "https://ippubblico.org/?api=1&lang=de"
# "city": "Tokio", "country": "Japan"
```

Supported localized languages: `de`, `es`, `fr`, `ja`, `pt`, `ru`, `zh`

## Rate limiting

The API allows one request per **10 seconds** per IP. If you exceed this, you receive:

```
HTTP 429 Too Many Requests
Retry-After: 8
```

For embedded devices and DDNS, polling every 30-60 seconds is recommended.

## Examples by platform

| Platform | Language | Folder |
|----------|----------|--------|
| ESP32 / ESP8266 | C++ (Arduino) | [esp32/](esp32/) |
| Arduino | C++ | [arduino/](arduino/) |
| Raspberry Pi | Python | [raspberry-pi/](raspberry-pi/) |
| MicroPython | MicroPython | [micropython/](micropython/) |
| OpenWRT / Router | Shell | [openwrt/](openwrt/) |
| Backend | Python | [python/](python/) |
| Backend | Node.js | [nodejs/](nodejs/) |
| Backend | Go | [go/](go/) |

## Full documentation

[ippubblico.org/docs.html](https://ippubblico.org/docs.html)
