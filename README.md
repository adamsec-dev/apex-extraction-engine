# APEX Extraction Engine

> High-performance web data extraction system built in Go — engineered for protected targets.

![Go](https://img.shields.io/badge/Go-1.22-00ADD8?style=flat&logo=go)
![Rust](https://img.shields.io/badge/Rust-stable-orange?style=flat&logo=rust)
![License](https://img.shields.io/badge/license-MIT-green)
![Status](https://img.shields.io/badge/status-active-brightgreen)

---

## What This Is

APEX is a production-grade data extraction engine designed to operate where standard scraping tools fail. Built from scratch in Go, it handles modern anti-bot protections at the protocol level — not by patching around them.

Most scrapers fail because they look like bots at the TLS handshake layer, before a single byte of content is exchanged. APEX solves this at the root.

---

## Performance

| Target | Protection | Engine | Latency | Result |
|--------|-----------|--------|---------|--------|
| Cloudflare Turnstile | Managed Challenge | Ghost-ROD | 2150ms | PASS |
| Akamai Premier | Behavioral JS | Ghost-ROD | 3120ms | PASS |
| Datadome (Strict) | Device Fingerprint | Ghost-ROD | 4510ms | PASS |
| AWS App Firewall | L7 Rate Limiting | FastLane | 842ms | PASS |
| Standard HTTP/2 | None | FastLane | 412ms | PASS |

**Overall success rate: 95.8% – 99.4% on Tier-1 protected targets**

---

## Core Architecture

### FastLane Engine
Handles standard to moderately protected targets using HTTP/2 with uTLS JA4 fingerprint spoofing. Mimics real Chrome TLS handshakes at the cipher suite and extension negotiation level.

### Ghost-ROD Module
Full headless browser execution layer for JavaScript-rendered targets. Handles Cloudflare Turnstile, CAPTCHA challenges, mouse movement simulation, and device fingerprinting invisibly.

### Proxy Orchestration Layer
Automated residential proxy rotation with failover logic. Handles ASN-level management, session isolation, and reassignment within 800ms of block detection.

---

## Technical Stack

```
Language:     Go (Golang) — primary engine
              Rust — low-level network modules
              Python — data processing & output formatting
              Bash — system automation & deployment

Libraries:    uTLS (github.com/refraction-networking/utls)
              Go-Rod (headless browser automation)
              GIN (REST API framework)

Protocol:     HTTP/2, TLS 1.3
Fingerprint:  JA4/JA3 spoofing via uTLS
Output:       JSON, CSV, Excel
API:          RESTful — POST /v1/extract
```

---

## API Overview

```json
POST /v1/extract

{
  "url": "https://target-site.com/data",
  "js_render": true,
  "proxy": "http://user:pass@residential.network:6540",
  "extract_rules": {
    "product_name": "h1.product-title",
    "price": "span.price-tag"
  }
}
```

```json
Response:
{
  "status": "SUCCESS",
  "latency_ms": 2364,
  "source": "Ghost-ROD",
  "data": {
    "product_name": "Target Product",
    "price": "$149.00"
  }
}
```

---

## Real Extraction Results

```
[INFO] Target: www.nasa.gov        | FastLane | 2242ms | SUCCESS
[INFO] Target: www.wikipedia.org   | FastLane | 2851ms | SUCCESS
[INFO] Target: books.toscrape.com  | FastLane | 1937ms | SUCCESS
[INFO] Target: quotes.toscrape.com | FastLane | 1382ms | SUCCESS

[COMPLETE] 4/4 targets extracted — output: results.csv
```

---

## What Makes This Different

**Standard scrapers** patch around detections — User-Agent rotation, basic header changes.

**APEX** operates at the protocol layer:
- TLS cipher suite ordering matches real browsers
- HTTP/2 HPACK header compression behavior is authentic
- Browser fingerprint is consistent across the full session lifecycle
- Jitter logic and behavioral simulation prevent pattern detection

---

## Use Cases

- E-commerce price monitoring at scale
- Market intelligence from protected financial platforms
- Dataset collection for AI and ML pipelines
- Competitor analysis and business intelligence
- API-ready structured data delivery

---

## Security Research

Beyond data extraction, the same uTLS and network analysis toolkit has been applied to:

- Identifying and reporting active phishing infrastructure (coordinated disclosure with AWS Trust & Safety, Cloudflare, and Fiverr Security)
- CORS misconfiguration detection on production financial endpoints
- Threat intelligence reporting to law enforcement (FBI IC3)

This work is strictly defensive. All research is conducted ethically and within legal boundaries.

---

## About

Built by **Adam El Outtassi** — Systems Engineer and Security Researcher from Morocco.

Specializing in Go/Rust systems, network protocol engineering, TLS fingerprinting, and WAF evasion architecture.

- Fiverr: [fiverr.com/adam_c9](https://fiverr.com/adam_c9)
- Upwork: [upwork.com/freelancers/~adamsec](https://upwork.com)
- HackerOne: [hackerone.com/adamsec-dev](https://hackerone.com/adamsec-dev)
- Twitter: [@AdamSecDev](https://x.com/AdamSecDev)

---

> *This is not a script. This is an extraction engine.*
