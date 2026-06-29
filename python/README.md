# Python Examples

## Get public IP

```python
import requests

def get_public_ip():
    try:
        r = requests.get('https://ipv4.ippubblico.org/', timeout=5)
        return r.text.strip()
    except requests.RequestException as e:
        print(f'Error: {e}')
        return None

print(get_public_ip())  # 203.0.113.42
```

## Full geolocation

```python
import requests

def get_ip_info(lang='en'):
    r = requests.get(f'https://ippubblico.org/?api=1&lang={lang}', timeout=5)
    data = r.json()
    return data if data['status'] == 'ok' else None

info = get_ip_info(lang='ja')  # Japanese response
if info:
    print(f"IP: {info['ip']}")
    print(f"Country: {info['geo']['country']}")  # 日本
    print(f"City: {info['geo']['city']}")         # 東京
```

## DDNS updater (runs as service)

```python
import requests
import time
import logging
from pathlib import Path

logging.basicConfig(level=logging.INFO,
    format='%(asctime)s %(message)s')

CACHE_FILE  = Path('/tmp/ddns_last_ip.txt')
CHECK_EVERY = 300  # seconds

def get_public_ip():
    try:
        r = requests.get('https://ipv4.ippubblico.org/', timeout=10)
        return r.text.strip()
    except Exception as e:
        logging.error(f'Failed to get IP: {e}')
        return None

def get_cached_ip():
    try: return CACHE_FILE.read_text().strip()
    except: return None

def update_ddns(ip):
    logging.info(f'Updating DNS to {ip}')
    # Add your DDNS provider API call here

while True:
    current = get_public_ip()
    if current:
        last = get_cached_ip()
        if current != last:
            logging.info(f'IP changed: {last} -> {current}')
            update_ddns(current)
            CACHE_FILE.write_text(current)
        else:
            logging.info(f'IP unchanged: {current}')
    time.sleep(CHECK_EVERY)
```

## Notes

- Requires `requests` library: `pip install requests`
- For async use, replace with `httpx` and `asyncio`
- Full examples with Django and FastAPI: [ippubblico.org/docs.html](https://ippubblico.org/docs.html)
