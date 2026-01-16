# Artillery-Load-Test---local-web-app
Load testing results for a local web app running at `http://localhost:3000` using **Artillery** with **two user journeys**: - **Homepage only** (browse `/`) - **Login journey** (visit `/`, open `/login`, think time, then POST `/login`)

This README documents the **test plan**, **how to reproduce**, and the **key performance results** captured from Artillery CLI output and Artillery Cloud export.

---

## 🧪 Test Objective

Validate application performance under increasing load by measuring:
- Throughput (req/s)
- Latency (p50/p95/p99 + max)
- Success rate (HTTP codes, errors, failed VUs)
- Endpoint behavior (top URLs hit)

---

## ⚙️ Test Configuration

**Target**
- `http://localhost:3000`

**Load Profile (Phases)**
| Phase | Duration | Arrival Rate | Ramp |
|------|----------|--------------|------|
| Warm up | 60s | 5/sec | ramp to 10/sec |
| Ramp up to peak load | 60s | 10/sec | ramp to 50/sec |
| Sustained peak load | 120s | 50/sec | steady |

**Traffic Mix**
| Scenario | Weight | What it does |
|---------|--------|--------------|
| Homepage only | ~70% | `GET /` |
| Login journey | ~30% | `GET /` → `GET /login` → `think 2s` → `POST /login` |

---

## 📄 Artillery Script

Save as `script.yaml`:

```yaml
config:
  target: "http://localhost:3000"
  phases:
    - duration: 60
      arrivalRate: 5
      rampTo: 10
      name: "Warm up phase"
    - duration: 60
      arrivalRate: 10
      rampTo: 50
      name: "Ramp up to peak load"
    - duration: 120
      arrivalRate: 50
      name: "Sustained peak load"
  defaults:
    headers:
      Content-Type: "application/json"

scenarios:
  - name: "Homepage only"
    weight: 70
    flow:
      - get:
          url: "/"

  - name: "Login journey"
    weight: 30
    flow:
      - get:
          url: "/"
      - get:
          url: "/login"
      - think: 2
      - post:
          url: "/login"
          json:
            email: "test@example.com"
            password: "password123"
