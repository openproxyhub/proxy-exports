
# Open Proxy Export – Telemetry‑Backed, Continuously Verified Proxy Lists

This repository provides **live, automatically validated proxy lists** generated from a dedicated proxy monitoring system.  
Unlike typical “scraped lists,” this project tracks each proxy over time using **protocol validation**, **uptime checks**, **anonymity tests**, and **geo‑verification**.

The result: **higher quality, fewer dead proxies, clearer metadata, and transparent filtering.**

---

# ⭐ Why This Proxy List Is Different

### ✔ Continuously updated  
A backend system checks proxies 24/7 and pushes new results to this repo.

### ✔ Only currently working proxies  
Every proxy in these lists has **recently passed**:
- Protocol validation  
- Uptime availability test  
- Active lifecycle check  

### ✔ Clear structure and multiple formats  
Each protocol has:
- `*.txt` – simple `IP:PORT`
- `*.json` – rich metadata

Additionally:
- Per‑country lists  
- Anonymity‑based lists  
- An all‑in‑one list  

### ✔ Real egress detection  
Country and ASN are based on **actual exit IP**, not lookup of the proxy’s listening IP.

### ✔ Friendly, clean, accessible documentation  
This repo is meant to be useful without requiring deep proxy knowledge.

---

# 📁 Directory Overview

```
/
├── all_proxies.txt
├── all_proxies.json
├── http_absolute.txt
├── http_absolute.json
├── http_connect.txt
├── http_connect.json
├── https_proxy_tls.txt
├── https_proxy_tls.json
├── socks4.txt
├── socks4.json
├── socks5.txt
├── socks5.json
├── proxies_by_country/
│   ├── US.txt
│   ├── DE.txt
│   └── ...
└── anonymity/
    ├── elite.txt
    ├── anonymous.txt
    └── transparent.txt
```

---

# 🔍 How the System Works (Detailed but Accessible)

The backend system has several automated components that work together to ensure proxy quality.

---

## 1. Proxy Discovery & Registry

Proxies enter the system through various sources and are stored in a registry with:
- IP  
- Port  
- Last seen time  
- Lifecycle status (new, active, slow, dead, deactivated)

Lifecycle ensures that failing proxies are gradually deprioritized, and working ones are kept active.

---

## 2. Protocol Validation

A validator determines *what kind* of proxy each entry truly is:

- **HTTP Absolute** HTTP proxy using absolute-form requests.
- **HTTP CONNECT** HTTP proxy that supports the CONNECT tunnel (typically for HTTPS).
- **HTTPS Proxy (TLS-based)** You connect to the proxy over HTTPS (TLS).
- **SOCKS4** Protocol-agnostic TCP (SOCKS4 may support UDP & auth).
- **SOCKS5** Protocol-agnostic TCP (SOCKS5 may support UDP & auth).

Incorrect protocol claims or repeated failures push a proxy toward “dead” status.

**Only proxies with `proto_status = 'ok'` are exported.**

---

## 3. Uptime Monitoring

A dedicated uptime worker checks:
- Does the proxy respond?
- How fast?
- Does it connect consistently?

Each probe updates:
- Last known uptime (`up` / `down`)
- Latency  
- Daily rolling statistics

**Only proxies with a *recent* `up` result appear in these lists.**

---

## 4. Geo / Country Verification

A geo worker sends traffic **through** the proxy to determine:
- Its real exit IP  
- Country code (ISO)  
- ASN  
- ISP/Organization  

This ensures accuracy for:
- `proxies_by_country/` lists  
- JSON metadata (`country`, `asn`, etc.)

---

## 5. Anonymity Testing

A capability probe checks:
- Does the proxy hide your IP?
- Does it add proxy headers?
- Does it leak information?

From this we classify proxies into:
- **elite** – hides your IP + no proxy headers  
- **anonymous** – hides IP but adds headers  
- **transparent** – exposes your IP or proxy identity  

These appear in the `/anonymity` folder.

---

## 6. Performance & Lifecycle Updating

Proxies are periodically promoted, demoted, or deactivated based on:
- Uptime consistency  
- Protocol reliability  
- Heavy/light validation results  
- Geo anomalies  
- Performance score  

This ensures lists contain **stable**, not just “once worked” proxies.

---

## 7. Export & Auto-Publish

A script periodically:
1. Queries all proxies currently meeting quality criteria  
2. Writes them into TXT and JSON files  
3. Organizes them by protocol, country, anonymity  
4. Commits and pushes updates to this GitHub repo

This means the repo stays fresh **without manual intervention**.

---

# 📦 TXT Format

TXT files contain one proxy per line:

```
IP:PORT
IP:PORT
```

Simple and compatible with almost any proxy tool.

---

# 📦 JSON Format (Detailed Metadata)

Each JSON list contains objects with enriched metadata:

```json
{
  "ip": "203.0.113.10",
  "port": 8080,
  "protocol": "http_connect",
  "country": "US",
  "asn": 12345,
  "org": "Example ISP",
  "last_seen": "2025-11-19T12:00:00Z",
  "first_seen": "2025-10-10T08:30:00Z",
  "latency_ms": 850,
  "anonymity": "elite"
}
```

(JSON structure may expand as the system adds more telemetry.)

---

# 🚀 Example Usage

### HTTP (curl)

```bash
curl -x http://IP:PORT https://httpbin.org/ip
```

### SOCKS5 (curl)

```bash
curl --socks5-hostname IP:PORT https://httpbin.org/ip
```

### Programmatic (Python)

```python
import json

data = json.load(open("all_proxies.json"))

elite = [p for p in data if p["anonymity"] == "elite"]
```

---

# ⚠️ Disclaimer

These are **public proxies**.  
They may be:
- Slow  
- Unpredictable  
- Overloaded  
- Unsafe for sensitive use  

Always encrypt your traffic (HTTPS, SOCKS5, etc.) and use good judgment.

---

# 💬 Feedback

Suggestions and documentation improvements are welcome.  
Direct proxy submissions are **not accepted**, as the system automatically discovers and validates proxies.
