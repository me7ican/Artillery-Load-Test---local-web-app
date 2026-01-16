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

How To Run (Local)
1) Install Artillery
npm i -g artillery

2) Run the test
artillery run script.yaml

3) Run + upload to Artillery Cloud (optional)
artillery run script.yaml --record --key <YOUR_KEY>


Note: If your login requires CSRF tokens/cookies/session handling, the login POST may need extra headers or cookie capture.

📊 Results Summary (Key Numbers)
High-Level

Total virtual users created: 8,250

Completed: 8,250

Failed VUs: 0 ✅

Total HTTP requests/responses: 18,066 / 18,066

Avg request rate: 74 req/s

Peak request rate: 114 req/s

Latency (Overall)

From CLI summary:

Mean: 22.4 ms

Median: 16.9 ms

p95: 62.2 ms

p99: 92.8 ms

Max: 158 ms

Interpretation: under peak load, 95% of requests completed within ~62ms and 99% within ~93ms. This indicates stable latency for localhost conditions.

Scenario Distribution (VUs)

Homepage only: 5,796 VUs

Login journey: 2,454 VUs

This aligns closely with a 70/30 weighted traffic split.

Endpoint Coverage (Requests by URL)

Artillery Cloud breakdown shows:

/ → 8,250 total

/login → 4,908 total

✅ This matches the script logic:

Every VU hits / once → 8,250

Login journey hits /login twice (GET + POST) → 2 × 2,454 = 4,908

HTTP Status Codes

Observed mostly:

200 OK

303 See Other (redirect)

The presence of 303 is typical for authentication flows (redirect-after-login). If redirects are enabled in the HTTP client, extra follow-up requests may increase total request count and affect “requests per URL” interpretation.

🧠 Discussion (What This Means)
✅ What went well

0 failed users and no errors during peak load suggests the app is stable for the tested profile.

Latency stayed low with a strong p95/p99, indicating good responsiveness.

⚠️ What to watch

303 redirects are expected, but for a performance report:

confirm whether redirects are intended behavior for /login

confirm whether the test client follows redirects automatically (can change total request counts and endpoint stats)

Login flow realism depends on your backend:

CSRF protections, sessions, rate limiting, captcha, etc. may require additional scripting.
