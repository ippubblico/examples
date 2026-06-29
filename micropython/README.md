# MicroPython Examples

For Raspberry Pi Pico W, ESP32 with MicroPython, M5Stack, and similar boards.

## Get public IP

```python
import urequests
import time

def get_public_ip():
    try:
        response = urequests.get('https://ipv4.ippubblico.org/')
        ip = response.text.strip()
        response.close()
        return ip
    except Exception as e:
        print(f'Error: {e}')
        return None

ip = get_public_ip()
print(f'Public IP: {ip}')
```

## DDNS updater

```python
import urequests
import time

last_ip = None

def get_public_ip():
    try:
        r = urequests.get('https://ipv4.ippubblico.org/')
        ip = r.text.strip()
        r.close()
        return ip
    except:
        return None

def update_ddns(new_ip):
    # Add your DDNS provider API call here
    print(f'Updating DNS to {new_ip}')

while True:
    current_ip = get_public_ip()
    if current_ip and current_ip != last_ip:
        print(f'IP changed: {last_ip} -> {current_ip}')
        update_ddns(current_ip)
        last_ip = current_ip
    time.sleep(60)  # check every 60 seconds
```

## Full geolocation (JSON)

```python
import urequests
import ujson

def get_ip_info():
    try:
        r = urequests.get('https://ippubblico.org/?api=1')
        data = ujson.loads(r.text)
        r.close()
        return data
    except Exception as e:
        print(f'Error: {e}')
        return None

info = get_ip_info()
if info and info['status'] == 'ok':
    print(f"IP: {info['ip']}")
    print(f"Country: {info['geo']['country']}")
    print(f"City: {info['geo']['city']}")
```

## Notes

- Tested on Raspberry Pi Pico W and ESP32 with MicroPython
- `urequests` is included in standard MicroPython builds
- For M5Stack, use the same code — MicroPython compatible
- Poll every **60 seconds** minimum for DDNS use cases
