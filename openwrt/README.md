# OpenWRT / Router / Embedded Linux Examples

Shell scripts for routers, NAS devices, and embedded Linux boards.

## Get public IP (one-liner)

```bash
curl -s https://ipv4.ippubblico.org/
```

## DDNS updater script

Save as `/usr/local/bin/ddns_update.sh`:

```bash
#!/bin/sh
# DDNS updater using IPPubblico.org — no API key required
# Compatible with OpenWRT, DD-WRT, and any busybox-based system

CACHE_FILE="/tmp/ddns_last_ip.txt"
LOG_FILE="/var/log/ddns.log"

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" >> "$LOG_FILE"
}

get_public_ip() {
    curl -s --max-time 10 https://ipv4.ippubblico.org/
}

update_ddns() {
    local NEW_IP="$1"
    # Replace with your DDNS provider API call
    # Example for Cloudflare:
    # curl -s -X PUT "https://api.cloudflare.com/client/v4/zones/ZONE_ID/dns_records/RECORD_ID" \
    #   -H "Authorization: Bearer YOUR_TOKEN" \
    #   -H "Content-Type: application/json" \
    #   --data "{\"type\":\"A\",\"name\":\"home.example.com\",\"content\":\"$NEW_IP\",\"ttl\":60}"
    log "TODO: update DNS record to $NEW_IP"
}

CURRENT_IP=$(get_public_ip)

if [ -z "$CURRENT_IP" ]; then
    log "ERROR: Failed to get public IP"
    exit 1
fi

LAST_IP=""
[ -f "$CACHE_FILE" ] && LAST_IP=$(cat "$CACHE_FILE")

if [ "$CURRENT_IP" = "$LAST_IP" ]; then
    log "IP unchanged: $CURRENT_IP"
    exit 0
fi

log "IP changed: $LAST_IP -> $CURRENT_IP"
update_ddns "$CURRENT_IP"
echo "$CURRENT_IP" > "$CACHE_FILE"
```

Add to cron (every 5 minutes):

```
*/5 * * * * /usr/local/bin/ddns_update.sh
```

## Get both IPv4 and IPv6

```bash
#!/bin/sh
RESULT=$(curl -s https://ippubblico.org/?text=1)
IPV4=$(echo "$RESULT" | grep "IPv4:" | cut -d' ' -f2)
IPV6=$(echo "$RESULT" | grep "IPv6:" | cut -d' ' -f2)

echo "IPv4: $IPV4"
echo "IPv6: $IPV6"
```

## Full geolocation

```bash
curl -s "https://ippubblico.org/?api=1" | \
  python3 -c "import sys,json; d=json.load(sys.stdin); \
  print(f'IP: {d[\"ip\"]}'); \
  print(f'Country: {d[\"geo\"][\"country\"]}'); \
  print(f'City: {d[\"geo\"][\"city\"]}')"
```

## Notes

- Compatible with OpenWRT, DD-WRT, Tomato, and any router with `curl`
- If `curl` is not available, use `wget -qO- https://ipv4.ippubblico.org/`
- Poll every **5-10 minutes** for DDNS — IP changes at most once per day on home connections
