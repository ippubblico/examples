# Raspberry Pi Examples

Python examples for Raspberry Pi (all models).

## Get public IP

```python
import requests

def get_public_ip():
    try:
        r = requests.get('https://ipv4.ippubblico.org/', timeout=5)
        return r.text.strip()
    except Exception as e:
        print(f'Error: {e}')
        return None

print(get_public_ip())
```

## DDNS updater as systemd service

Save as `/usr/local/bin/ddns_updater.py`:

```python
#!/usr/bin/env python3
import requests
import time
import logging
from pathlib import Path

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s %(message)s',
    handlers=[
        logging.FileHandler('/var/log/ddns_updater.log'),
        logging.StreamHandler()
    ]
)

CACHE  = Path('/tmp/ddns_last_ip.txt')
EVERY  = 300  # seconds

def get_ip():
    r = requests.get('https://ipv4.ippubblico.org/', timeout=10)
    return r.text.strip()

def update_dns(ip):
    logging.info(f'Updating DNS to {ip}')
    # Add your DNS provider API call here

logging.info('DDNS updater started')
while True:
    try:
        current = get_ip()
        last    = CACHE.read_text().strip() if CACHE.exists() else None
        if current != last:
            logging.info(f'IP changed: {last} -> {current}')
            update_dns(current)
            CACHE.write_text(current)
        else:
            logging.info(f'IP unchanged: {current}')
    except Exception as e:
        logging.error(f'Error: {e}')
    time.sleep(EVERY)
```

Systemd service file `/etc/systemd/system/ddns-updater.service`:

```ini
[Unit]
Description=DDNS Updater
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=/usr/bin/python3 /usr/local/bin/ddns_updater.py
Restart=on-failure
RestartSec=30

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
sudo systemctl enable ddns-updater
sudo systemctl start ddns-updater
```

## Notes

- Works on all Raspberry Pi models (Zero, 3, 4, 5)
- Requires `requests`: `pip install requests`
- For MicroPython (Pico W), see the [micropython/](../micropython/) folder
