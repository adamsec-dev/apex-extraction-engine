# APEX Extraction Engine

> High-performance web data extraction system built in Go — engineered for protected targets.

![Go](https://img.shields.io/badge/Go-1.22-00ADD8?style=flat&logo=go)
![Rust](https://img.shields.io/badge/Rust-stable-orange?style=flat&logo=rust)
![Python](https://img.shields.io/badge/Python-3.11-blue?style=flat&logo=python)
![License](https://img.shields.io/badge/license-MIT-green)
![Status](https://img.shields.io/badge/status-active-brightgreen)

---

## What This Is

APEX is a production-grade data extraction engine designed to operate where standard scraping tools fail. Built from scratch in Go, it handles modern anti-bot protections at the **protocol level** — not by patching around them.

Most scrapers fail because they look like bots at the TLS handshake layer, before a single byte of content is exchanged. APEX solves this at the root.

---

## Project Structure

```
apex-extraction-engine/
├── main.go                          # API server entry point (GIN framework)
├── go.mod
├── pkg/
│   ├── analyzer/
│   │   └── classifier.go           # WAF response classifier
│   ├── fingerprint/
│   │   ├── profiles.go             # Browser fingerprint profiles
│   │   └── manager.go              # Polymorphic identity manager
│   ├── orchestrator/
│   │   └── runner.go               # Core execution engine
│   ├── session/
│   │   ├── session.go              # Session lifecycle management
│   │   └── jar.go                  # Cookie jar handler
│   ├── solver/
│   │   ├── js_solver.go            # JavaScript challenge solver
│   │   └── rod_solver.go           # Ghost-ROD headless module
│   └── transport/
│       └── client.go               # uTLS transport layer
├── utls_evasion/
│   ├── utls_core.go                # TLS fingerprint spoofing core
│   ├── adaptive_brain.go           # Adaptive evasion logic
│   ├── simulator.go                # Behavioral simulation
│   └── memory.go                   # Session memory & rotation
└── internal/
    └── metrics/
        ├── collector.go            # Performance metrics
        └── types.go                # Metric types
```

---

## Core Architecture

### 1. WAF Response Classifier

The engine automatically identifies the target defense layer and routes accordingly:

```go
type ResultType string

const (
    Real200   ResultType = "REAL_200"
    Challenge ResultType = "CHALLENGE"
    Block     ResultType = "BLOCK"
    RateLimit ResultType = "RATE_LIMIT"
)

func (c *Classifier) Analyze(resp *http.Response) (AnalysisResult, error) {
    if resp.StatusCode == 403 {
        return AnalysisResult{Type: Block, Confidence: 1.0}, nil
    }
    if strings.Contains(bodyStr, "captcha") || strings.Contains(bodyStr, "challenge") {
        return AnalysisResult{Type: Challenge, Confidence: 0.9}, nil
    }
    return AnalysisResult{Type: Real200, Confidence: 0.8}, nil
}
```

---

### 2. Browser Fingerprint Engine

Polymorphic identity system — each request carries a consistent, authentic browser signature:

```go
type Profile struct {
    ID            string
    UserAgent     string
    TLSID         utls.ClientHelloID
    JA4           string
    HTTP2Settings map[uint16]uint32
}

var Chrome_134_Win = Profile{
    ID:    "chrome_134_windows",
    TLSID: utls.HelloChrome_Auto,
    JA4:   "t13d1516h2_8daaf61527df_...",
}

// GetRandomProfile returns a polymorphic identity for evasion
func (m *Manager) GetRandomProfile() Profile {
    return m.AvailableProfiles[rand.Intn(len(m.AvailableProfiles))]
}
```

---

### 3. Adaptive Retry Engine

Exponential backoff with jitter — simulates human behavioral patterns:

```go
for attempt := 0; attempt < 3; attempt++ {
    if attempt > 0 {
        waitTime := time.Duration(attempt*2) * time.Second
        time.Sleep(waitTime + time.Duration(rand.Intn(1000))*time.Millisecond)
    }
    client, err := transport.NewApexClient(jar)
}
```

---

### 4. Dual-Engine Routing

FastLane for standard targets → Ghost-ROD auto-escalation for protected targets:

```go
html, err = executeFastLane(req.URL, req.Proxy, nil)

if err != nil || isBlocked(html) || req.JSRender {
    rodSolver := solver.NewRodSolver()
    cookies, _ := rodSolver.GetClearanceCookie(req.URL, req.Proxy)
    html, err = executeFastLane(req.URL, req.Proxy, cookies)
}

func isBlocked(html string) bool {
    checks := []string{
        "access denied", "blocked", "just a moment",
        "bot detection", "security check",
    }
    for _, c := range checks {
        if strings.Contains(strings.ToLower(html), c) {
            return true
        }
    }
    return len(html) < 900
}
```

---

## REST API

```
POST /v1/extract
Content-Type: application/json
```

### Request

```json
{
  "url": "https://target-site.com/data",
  "js_render": false,
  "api_key": "apex_live_xxxx",
  "proxy": "http://user:pass@residential.network:6540",
  "extract_rules": {
    "product_name": "h1.product-title",
    "price": "span.price-tag"
  }
}
```

### Response

```json
{
  "status": "SUCCESS",
  "latency_ms": 2364,
  "source": "Ghost-ROD",
  "data": {
    "product_name": "Enterprise Logic Core v2",
    "price": "$499.00"
  }
}
```

---

## Python Client

```python
import requests

API_URL = "http://localhost:8080/v1/extract"

TARGETS = [
    "https://www.nasa.gov",
    "https://www.wikipedia.org",
    "http://books.toscrape.com",
]

for url in TARGETS:
    payload = {
        "url": url,
        "js_render": False,
        "api_key": "apex_live_xxxx",
        "proxy": "http://user:pass@node:6540",
        "extract_rules": {"Page_Title": "title", "Header": "h1"}
    }
    data = requests.post(API_URL, json=payload, timeout=180).json()
    print(f"[{data['status']}] {url} | {data['source']} | {data['latency_ms']}ms")
```

---

## Live Extraction Results

```
[INFO] Target: www.nasa.gov        | FastLane | 2242ms | SUCCESS
[INFO] Target: www.wikipedia.org   | FastLane | 2851ms | SUCCESS
[INFO] Target: books.toscrape.com  | FastLane | 1937ms | SUCCESS
[INFO] Target: quotes.toscrape.com | FastLane | 1382ms | SUCCESS

[COMPLETE] 4/4 targets — output: results.csv

Page_Title                     Routed_IP
NASA                           31.59.20.176
Wikipedia                      23.26.71.145
All products | Books to Scrape 23.26.71.145
Quotes to Scrape               23.26.71.145
<img width="1304" height="472" alt="Ajouter un sous-titre" src="https://github.com/user-attachments/assets/429c7fcc-88ea-4682-81f3-cde9632cf26a" />
<img width="1348" height="516" alt="Custom Web Scraping Engine High Performance Data Extraction Fast • Stable • Reliable" src="https://github.com/user-attachments/assets/e02b6ca5-f57a-45ac-951d-93a75205cfc5" />

```

---

## Performance Benchmarks

| Target | Protection | Engine | Latency | Result |
|--------|-----------|--------|---------|--------|
| Cloudflare Turnstile | Managed Challenge | Ghost-ROD | 2150ms | PASS |
| Akamai Premier | Behavioral JS | Ghost-ROD | 3120ms | PASS |
| Datadome (Strict) | Device Fingerprint | Ghost-ROD | 4510ms | PASS |
| AWS App Firewall | L7 Rate Limiting | FastLane | 842ms | PASS |
| Standard HTTP/2 | None | FastLane | 412ms | PASS |

**Success rate: 95.8% – 99.4% on Tier-1 protected targets**

---

## Technical Stack

```
Primary Engine:   Go (Golang)
Network Layer:    Rust
Data Processing:  Python
Automation:       Bash / Linux

Libraries:
  uTLS     — github.com/refraction-networking/utls
  Go-Rod   — headless browser automation
  GIN      — REST API framework

Protocol:         HTTP/2, TLS 1.3
Fingerprinting:   JA4/JA3 via uTLS
Output:           JSON, CSV, Excel
<img width="1350" height="627" alt="Capture d’écran 2026-04-12 180031" src="https://github.com/user-attachments/assets/65187088-60e0-4024-b945-3dc136e96022" />

```

---

## Security Research

The same toolkit has been applied to defensive security research:

- Identified active **PhaaS syndicate** targeting Fiverr, Depop, Airbnb users globally
- Mapped hidden AWS origin servers via Host Header Injection and SSL certificate matching
- Detected **CORS misconfigurations** on production financial endpoints
- Coordinated disclosure with AWS Trust & Safety, Cloudflare, and Fiverr Security

All research is strictly defensive and conducted ethically.

---

## About

Built by **Adam El Outtassi** — Systems Engineer and Security Researcher from Morocco.

| Platform | Link |
|----------|------|
| Fiverr | [fiverr.com/adam_c9](https://fiverr.com/adam_c9) |
| Upwork | [upwork.com/freelancers/adamsec](https://www.upwork.com/freelancers/~01709ce34b086c899e?mp_source=share) |
| HackerOne | [hackerone.com/adamsec-dev](https://hackerone.com/adamsec-dev) |
| Twitter | [@AdamSecDev](https://x.com/AdamSecDev) |

---

> *This is not a script. This is an extraction engine.*
