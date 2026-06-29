# Node.js Examples

No dependencies required for basic usage (Node 18+ built-in fetch).

## Get public IP

```javascript
const res = await fetch('https://ipv4.ippubblico.org/', {
  signal: AbortSignal.timeout(5000)
});
const ip = (await res.text()).trim();
console.log(ip); // 203.0.113.42
```

## Full geolocation

```javascript
const res  = await fetch('https://ippubblico.org/?api=1');
const data = await res.json();
console.log(data.ip);             // 203.0.113.42
console.log(data.geo.country);    // Japan
console.log(data.geo.city);       // Tokyo
console.log(data.isp);            // NTT Communications
```

## Multilingual response

```javascript
const res  = await fetch('https://ippubblico.org/?api=1&lang=ja');
const data = await res.json();
console.log(data.geo.country);    // 日本
console.log(data.geo.city);       // 東京
```

## DDNS updater

```javascript
import fs from 'fs/promises';

const CACHE = '/tmp/ddns_last_ip.txt';

async function getPublicIP() {
  const res = await fetch('https://ipv4.ippubblico.org/', {
    signal: AbortSignal.timeout(5000)
  });
  return (await res.text()).trim();
}

async function check() {
  const current = await getPublicIP();
  let last = null;
  try { last = (await fs.readFile(CACHE, 'utf-8')).trim(); } catch {}

  if (current !== last) {
    console.log(`IP changed: ${last} → ${current}`);
    // Add your DDNS update logic here
    await fs.writeFile(CACHE, current);
  }
}

check();
setInterval(check, 5 * 60 * 1000); // every 5 minutes
```

## Notes

- No external dependencies for basic use (Node 18+)
- Full examples with Express middleware: [ippubblico.org/docs.html](https://ippubblico.org/docs.html)
