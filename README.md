```markdown
# Environmental Impact Data-Logger · Node-RED Flows

Minimal Node-RED flows for polling construction-site sensors over **RS-485 / Modbus RTU**, converting the data to the **BMA E-Permit JSON schema**, and posting each payload to the **Bangkok Metropolitan Administration – Building Control Office (BCO)** API via **HTTPS + JWT**.

---

## Quick Start

```bash
git clone https://github.com/<your-org>/eid-nodered.git
cd eid-nodered
cp config/env.sample .env      # edit BCO_URL, JWT creds, serial port …
docker compose up -d           # Node-RED, Modbus agent, JWT manager
```

Open **`http://<edgebox-ip>:1880`** for the Node-RED editor  
(default `admin / changeme` — change immediately).

---

## Essential Configuration (`.env`)

| Key              | Example                                         |
|------------------|-------------------------------------------------|
| `BCO_URL`        | `https://bco.bangkok.go.th/api/v1/environment`  |
| `JWT_CLIENT_ID`  | `eid-logger-01`                                 |
| `MODBUS_PORT`    | `/dev/ttyS0`                                    |
| `MODBUS_BAUD`    | `9600`                                          |
| `DEVICE_ID`      | `BMA-EID-001`                                   |

Modbus registers ↔ JSON fields live in **`config/modbus-map.yaml`** (hot-reload).

---

## Requirements

* Edge device: EdgeBox RPi 200 (or any ARM64+Docker host)  
* Software: Docker & Compose, Node-RED ≥ 3.1 (pre-built image)  
* Bus: Shielded RS-485, 120 Ω termination, 9600-115200 bps  
* Network: LTE Cat 4 (or wired) with outbound HTTPS to BCO

---

## Security Notes

* TLS 1.3 with certificate pinning  
* JWT auto-refresh every 5 h; data buffered (SQLite) if offline  
* Watch-dog restarts Node-RED on heartbeat timeout (60 s)

---

## License

[MIT](LICENSE) – free to use, modify, and distribute.
```
