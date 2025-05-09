# Environmental Impact Data-Logger – Node-RED Flows  
*A reference implementation for real-time environmental monitoring and automatic data submission to the **Bangkok Metropolitan Administration – Building Control Office (BMA BCO)** platform*  

---

## 1. Overview  

This repository contains the complete **Node-RED** source code and auxiliary scripts used in the *Environmental Impact Data-Logger* prototype.  
The system polls multi-sensor nodes over **RS-485 / Modbus RTU**, validates and normalises the measurements, converts them into the **BMA E-Permit JSON schema**, and pushes the payload to the **BCO API Gateway** through an **HTTPS + JWT** authenticated channel.  

Key design goals  

| Goal | Rationale |
|------|-----------|
| **Standards-Based I/O** | Leverage Modbus RTU for maximum vendor neutrality. |
| **Edge Validation & Normalisation** | Reduce bandwidth; guarantee schema compliance before uplink. |
| **Secure, Resilient Uplink** | TLS 1.3, JWT auto-refresh, retry w/ exponential back-off. |
| **Minimal Footprint** | Runs on EdgeBox RPi 200 (4 GB RAM) with < 5 % CPU @ 10 s polling. |
| **Extensible** | Drag-and-drop new sensors, rules, or cloud endpoints in Node-RED. |

---

## 2. Architecture  

```
┌────────┐  Modbus  ┌────────┐           ┌────────────────────────────┐
│Sensor 1│◀────────▶│ Node-RED │ HTTPS   │   BMA BCO API Gateway      │
│Sensor 2│◀────────▶│  Flows  │─────────▶│ (E-Permit dashboard & DB)  │
└────────┘           └────────┘           └────────────────────────────┘
```

* **Modbus Layer** – Python helper (`/services/modbus-agent`) publishes validated registers to local MQTT topics.  
* **Node-RED Layer** – `/flows` directory defines dashboards, alerts, and the JSON-mapping → HTTPS uplink.  
* **Security Layer** – `/services/jwt-manager` renews a short-lived token; `tls-client-credentials` folder stores PEM files.  

---

## 3. Repository Structure  

```
.
├── flows/                     # Exported Node-RED flow JSON files
│   ├── main.flow.json         # Core polling → mapping → uplink
└── LICENSE
```

---

## 4. Prerequisites  

| Component | Version | Notes |
|-----------|---------|-------|
| **Node-RED** | ≥ 3.1 | Pre-built in Docker image |
| **EdgeBox RPi 200** | CM4 32-bit Debian Bookworm | Kernel 6.1 w/ rs485 overlay |
| **Python** | 3.10+ | For Modbus agent |
| **npm** | 8+ | Only required if rebuilding Docker |

Hardware  

* RS-485 shielded twisted-pair bus, terminated 120 Ω  
* LTE Cat 4 module with public Internet or private APN reachability to BCO gateway  

---

## 5. Quick Start — Importing the Flow

1. **Download the flow JSON**  
   * Grab `flows/main.flow.json` from this repository ( Code ▸ Download ZIP  or the *Raw* button ▸ Save As ).

2. **Open your Node-RED editor**  
   * Default URL is `http://<your-node-red-host>:1880`.

3. **Import the flow**  
   * Menu ▸ *Import* ▸ *Clipboard* ▸ paste the JSON **or** drag-and-drop the saved file.  
   * Review the nodes; Node-RED will highlight any missing dependencies.

4. **Install required nodes** (once, if they aren’t present)  
   ```bash
   # from the Node-RED editor Menu ▸ Manage palette ▸ Install
   node-red-contrib-modbus
   node-red-contrib-jwt
   ```

5. **Configure credentials & serial port**  
   * Double-click the “Environment Config” node and enter  
     * `BCO_URL`, `JWT_CLIENT_ID`, `JWT_SECRET`  
     * `MODBUS_PORT` (e.g. `/dev/ttyS0`) and `MODBUS_BAUD` (9600).

6. **Deploy**  
   * Click **Deploy** – the flow will begin polling sensors and pushing data to BCO.

> **Tip:** keep a serial console or `node-red-log` open to watch for connection errors while you fine-tune the settings.

## 7. Security Considerations  

* **TLS 1.3** enforced; certificate pinned inside container.  
* Tokens are refreshed every 5 h; failure triggers exponential back-off (max 30 min) while buffering data locally.  
* Sensitive files live in a read-only Docker volume; secrets should be injected via Docker secrets or Kubernetes Secrets in production.

---

## 8. Development & Contribution  

1. Fork ➜ create feature branch ➜ commit (conventional-commits)  
2. Run `scripts/lint.sh` (ESLint + black)  
3. Submit a Pull Request with a detailed description  
4. At least one reviewer must approve before merge.

Bug reports via GitHub Issues; please attach logs (`services/modbus-agent/*.log`) and Node-RED flow snippets.

---

## 9. License  

Distributed under the **MIT License** – see [`LICENSE`](LICENSE) for details.  
Sensor vendor SDKs included in `vendor/` may carry their own licenses.

---

## 10. References  

* BMA BCO. *Construction E-Permit Sensor Data JSON Reference*, 2025.  
* International Electrotechnical Commission. *IEC 61131-2: Programmable Controllers – Equipment Requirements*, 2017.  
* Open Modbus Organization. *Modbus Application Protocol Specification v1.1b3*, 2012.  

> **Questions?**  
> Open an issue or contact `awatchar(at)engr.tu.ac.th` (subject “EID-Node-RED”).
