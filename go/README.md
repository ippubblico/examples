# Go Examples

No external dependencies — standard library only.

## Get public IP

```go
package main

import (
    "fmt"
    "io"
    "net/http"
    "strings"
    "time"
)

func getPublicIP() (string, error) {
    client := &http.Client{Timeout: 5 * time.Second}
    resp, err := client.Get("https://ipv4.ippubblico.org/")
    if err != nil {
        return "", err
    }
    defer resp.Body.Close()
    body, _ := io.ReadAll(resp.Body)
    return strings.TrimSpace(string(body)), nil
}

func main() {
    ip, err := getPublicIP()
    if err != nil {
        fmt.Printf("Error: %v\n", err)
        return
    }
    fmt.Printf("Public IP: %s\n", ip)
}
```

## DDNS updater (static binary, no runtime needed)

```go
package main

import (
    "fmt"
    "io"
    "log"
    "net/http"
    "os"
    "strings"
    "time"
)

const (
    cacheFile  = "/tmp/ddns_last_ip.txt"
    checkEvery = 5 * time.Minute
)

var client = &http.Client{Timeout: 10 * time.Second}

func getPublicIP() (string, error) {
    resp, err := client.Get("https://ipv4.ippubblico.org/")
    if err != nil { return "", err }
    defer resp.Body.Close()
    body, _ := io.ReadAll(resp.Body)
    return strings.TrimSpace(string(body)), nil
}

func checkAndUpdate() {
    current, err := getPublicIP()
    if err != nil { log.Printf("Error: %v", err); return }

    last := ""
    if data, err := os.ReadFile(cacheFile); err == nil {
        last = strings.TrimSpace(string(data))
    }

    if current == last {
        log.Printf("IP unchanged: %s", current)
        return
    }

    log.Printf("IP changed: %s → %s", last, current)
    // Add your DDNS update logic here
    os.WriteFile(cacheFile, []byte(current), 0644)
}

func main() {
    log.Println("DDNS updater started")
    checkAndUpdate()
    for range time.Tick(checkEvery) {
        checkAndUpdate()
    }
}
```

Build as static binary:

```bash
go build -o ddns_updater .

# Cross-compile for Raspberry Pi
GOOS=linux GOARCH=arm64 go build -o ddns_updater_arm64 .

# Cross-compile for OpenWRT (MIPS)
GOOS=linux GOARCH=mips go build -o ddns_updater_mips .
```

## Notes

- Zero external dependencies — standard library only
- Compiles to a single static binary
- Cross-compile for any platform: ARM, MIPS, x86, x86_64
- Full examples with Gin middleware: [ippubblico.org/docs.html](https://ippubblico.org/docs.html)
