# PRISM Benchmark Evidence v1.0

**Reproducible Performance Evidence for PRISM GPU-Native Portfolio Optimization Engine**

Published by [Asymmetry Computing](https://asymmetrycomputing.com)

---

## What Is This?

PRISM is a **hosted portfolio optimization service** — it solves large-scale investment portfolio problems on GPUs in milliseconds. You send data over HTTPS, you get optimized portfolio weights back. **There is nothing to download or install.**

This repository contains the **benchmark evidence** proving PRISM's speed and quality claims. Every CSV and JSON file here is a reproducible artifact you can independently verify.

---

## Quick Start: Try the API in 60 Seconds

### 1. Check that the API is alive

Open your browser or terminal and visit:

```
https://prism.asymmetrycomputing.com/v1/health
```

You should see:
```json
{"status": "ok", "version": "1.0.0"}
```

### 2. Get an API key

Email **info@asymmetrycomputing.com** and request a free Developer key. You'll receive a key that looks like `pk_...`.

### 3. Check system status (requires key)

**Using curl (Mac / Linux / Windows terminal):**
```bash
curl -H "X-PRISM-Key: YOUR_KEY_HERE" \
  https://prism.asymmetrycomputing.com/v1/status
```

**Using Python (any version 3.6+, only needs the `requests` library):**
```python
import requests

r = requests.get(
    "https://prism.asymmetrycomputing.com/v1/status",
    headers={"X-PRISM-Key": "YOUR_KEY_HERE"}
)
print(r.json())
```

You'll see available engines, GPU info, and supported modes.

### 4. Run your first optimization

**Using curl:**
```bash
curl -X POST https://prism.asymmetrycomputing.com/v1/solve \
  -H "X-PRISM-Key: YOUR_KEY_HERE" \
  -H "Content-Type: application/json" \
  -d '{
    "n_assets": 100,
    "mode": "balanced",
    "gamma": 0.0005,
    "position_max": 0.10,
    "deadline_ms": 5000
  }'
```

**Using Python:**
```python
import requests

response = requests.post(
    "https://prism.asymmetrycomputing.com/v1/solve",
    headers={
        "X-PRISM-Key": "YOUR_KEY_HERE",
        "Content-Type": "application/json"
    },
    json={
        "n_assets": 100,
        "mode": "balanced",
        "gamma": 0.0005,
        "position_max": 0.10,
        "deadline_ms": 5000
    }
)

result = response.json()
print(f"Status:     {result['status']}")
print(f"Solve time: {result['wall_ms']:.1f} ms")
print(f"Feasible:   {result['feasible']}")
print(f"Solve ID:   {result['solve_id']}")
```

### 5. Retrieve the audit certificate

Every solve produces a unique, tamper-evident audit record:

```bash
curl -H "X-PRISM-Key: YOUR_KEY_HERE" \
  https://prism.asymmetrycomputing.com/v1/audit/SOLVE_ID_HERE
```

### 6. Run a batch of optimizations

```bash
curl -X POST https://prism.asymmetrycomputing.com/v1/batch \
  -H "X-PRISM-Key: YOUR_KEY_HERE" \
  -H "Content-Type: application/json" \
  -d '{
    "problems": [
      {"n_assets": 100, "mode": "fast"},
      {"n_assets": 500, "mode": "balanced"},
      {"n_assets": 1000, "mode": "precision"}
    ]
  }'
```

---

## API Reference

| Endpoint | Method | Auth Required? | Description |
|----------|--------|----------------|-------------|
| `/v1/health` | GET | No | Is the service alive? |
| `/v1/status` | GET | Yes | Engine list, GPU info, queue depth |
| `/v1/solve` | POST | Yes | Optimize a portfolio |
| `/v1/batch` | POST | Yes | Optimize multiple portfolios at once |
| `/v1/audit/{solve_id}` | GET | Yes | Retrieve audit certificate for a solve |

### Authentication

Add this header to every authenticated request:
```
X-PRISM-Key: YOUR_KEY_HERE
```

### Solve Request Fields

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `n_assets` | integer | **Yes** | — | Number of assets in your portfolio (2 to 100,000) |
| `mode` | string | No | `balanced` | Quality/speed tradeoff: `realtime`, `fast`, `balanced`, `precision`, or `research` |
| `engine` | string | No | auto | Compute engine: `auto`, `cpu`, `gpu`, `extreme`, `factor-cpu`, `factor-gpu` |
| `gamma` | number | No | 0.0005 | Risk aversion parameter (0 to 1) |
| `position_max` | number | No | 0.10 | Maximum weight per asset (0 to 1) |
| `deadline_ms` | integer | No | 50 | Time budget in milliseconds (1 to 60,000) |
| `crisis_mode` | boolean | No | false | Enable crisis-mode constraints |
| `mu_b64` | string | No | — | Expected returns vector — base64-encoded float64 array |
| `B_b64` | string | No | — | Factor loadings matrix (N×K) — base64-encoded float64 |
| `D_b64` | string | No | — | Idiosyncratic variance — base64-encoded float64 array |
| `cov_b64` | string | No | — | Full covariance matrix — base64-encoded float64 |

### Solve Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `solve_id` | string | Unique identifier for this solve |
| `status` | string | `optimal`, `feasible`, or error status |
| `n_assets` | integer | Universe size used |
| `engine_used` | string | Which engine handled the solve |
| `wall_ms` | number | Total wall-clock time in milliseconds |
| `objective` | number | Optimal objective value |
| `feasible` | boolean | Whether the solution satisfies all constraints |
| `weights_b64` | string | Optimized portfolio weights — base64-encoded float64 |
| `audit_hash` | string | SHA-256 hash for tamper-evident verification |

---

## How to Replicate Our Benchmark (Step by Step)

You can independently verify PRISM's performance claims using only the API. **No software to install, no packages to download.**

### Step 1: Get a key
Email **info@asymmetrycomputing.com** for a free Developer key.

### Step 2: Run the reproducibility test
Copy this Python script, paste your key, and run it. It performs 40 consecutive solves at N=5,000 assets:

```python
import requests, time

API = "https://prism.asymmetrycomputing.com"
KEY = "YOUR_KEY_HERE"                     # <-- paste your key here
HEADERS = {"X-PRISM-Key": KEY, "Content-Type": "application/json"}

results = []
for i in range(40):
    r = requests.post(
        f"{API}/v1/solve",
        headers=HEADERS,
        json={"n_assets": 5000, "mode": "balanced"}
    )
    d = r.json()
    results.append(d)
    print(f"Run {i+1:2d}: {d['wall_ms']:8.1f} ms  feasible={d['feasible']}  status={d['status']}")

# Print summary
wall_times = sorted([r["wall_ms"] for r in results])
feasible_count = sum(1 for r in results if r["feasible"])
unique_hashes = len(set(r.get("audit_hash", "") for r in results))

print(f"\n--- Summary ---")
print(f"Median solve time: {wall_times[20]:.1f} ms")
print(f"Feasible solves:   {feasible_count}/40")
print(f"Unique audit IDs:  {unique_hashes}")
```

### Step 3: Compare with the evidence
Open `PRISM_EVIDENCE_API_REPRO_5000.csv` in this repo and compare your wall_ms distribution against the published results.

---

## Benchmark Results

| Metric | Value | Evidence File |
|--------|-------|---------------|
| **Median Solve Time (5,000 assets)** | 14,000 ms (GPU) | `PRISM_EVIDENCE_CANONICAL_5000_REAL.csv` |
| **Speedup vs Gurobi** | 13.0–20.6× | `PRISM_EVIDENCE_SUPERIORITY_GATE_5000_REAL.csv` |
| **Speedup vs OSQP** | 4.3–10.4× | `PRISM_EVIDENCE_SUPERIORITY_GATE_5000_REAL.csv` |
| **Reproducibility** | 40/40 optimal & feasible | `PRISM_EVIDENCE_API_REPRO_5000.csv` |
| **p99 Latency** | 258 ms | `PRISM_EVIDENCE_CANONICAL_5000_REAL.csv` |
| **Throughput** | ~14,000 optimizations/hour | Computed from median solve time |
| **Scale Smoke** | N=20K, 50K, 100K | `PRISM_EVIDENCE_SCALE_SMOKE_20000_50000_2026-02-15.json` |

## Hardware & Environment

- **GPU**: NVIDIA RTX 4000 Ada (20 GB VRAM)
- **CUDA**: 12.x
- **CPU Baselines**: Gurobi 13.0.1 (barrier method), OSQP (default settings)
- **Data**: Real market data from Yahoo Finance / public exchanges
- **Warm start**: None (cold-start benchmarks throughout)

---

## Evidence Artifacts

### Primary Claims

| ID | File | Description |
|----|------|-------------|
| A1 | `PRISM_EVIDENCE_SUPERIORITY_GATE_5000_REAL.csv` | PRISM vs Gurobi vs OSQP — 4 scenario families |
| A2 | `PRISM_EVIDENCE_CANONICAL_5000_REAL.csv` | Canonical 5,000-asset benchmark |
| A3 | `PRISM_EVIDENCE_API_REPRO_5000_summary.json` | API reproducibility summary |
| A4 | `PRISM_EVIDENCE_API_REPRO_5000.csv` | 40-run API reproducibility detail |
| A5 | `PRISM_EVIDENCE_REAL_DATASET_SUMMARY_2026-02-15.json` | Real dataset coverage summary |
| A6 | `PRISM_EVIDENCE_REAL_DATASET_COVERAGE_2026-02-15.csv` | Per-ticker coverage detail |
| A7 | `PRISM_EVIDENCE_REAL_INPUT_SOLVES_2026-02-15.json` | Real-input solve verification |
| A8 | `PRISM_EVIDENCE_SCALE_SMOKE_20000_50000_2026-02-15.json` | Large-N scale smoke test |
| A9 | `PRISM_EVIDENCE_GPU_MOAT_REALCACHE_LARGE_N_2026-02-15.csv` | GPU moat across asset counts |
| A10 | `PRISM_EVIDENCE_REAL_CACHE_EXPANDED_SUMMARY_2026-02-15.json` | Expanded real-cache summary |
| S1 | `PRISM_EVIDENCE_SCALING_SUPPORT_SYNTHETIC.csv` | Synthetic scaling support data |

### Supporting Evidence

| File | Description |
|------|-------------|
| `PRISM_EVIDENCE_INDUSTRY_GRADE_SUMMARY_2026-02-16.json` | Industry-grade campaign summary |
| `PRISM_EVIDENCE_INDUSTRY_GRADE_CAMPAIGN_2026-02-16.csv` | Per-run campaign detail |
| `PRISM_EVIDENCE_UNIVERSE_US_COMMON_2026-02-16.csv` | US common stock universe used |
| `PRISM_EVIDENCE_SUPERIORITY_GATE_5000_REAL_summary.json` | Superiority gate summary |
| `PRISM_EVIDENCE_REAL_DATASET_SAMPLE50_2026-02-15.csv` | Sample dataset extract |

### Datasets

| File | Description |
|------|-------------|
| `datasets/us_common_universe_20260216_summary.json` | Universe metadata summary |
| `datasets/us_common_universe_20260216.csv` | Full universe ticker list |
| `datasets/strict_us_common_metadata_20260216.csv` | Strict common stock filter metadata |

### Documentation

| File | Description |
|------|-------------|
| `PRISM_Institutional_Memo.md` | Full institutional memo (Markdown) |
| `PRISM_Institutional_Memo.pdf` | Formatted institutional memo (PDF) |
| `PRISM_Cover_Sheets.md` | Cover sheet appendix |

---

## Important: No Downloadable Software

- **PRISM is a hosted service only** — there is no downloadable software, library, SDK, or package.
- **No solver internals are exposed** — the API returns optimized weights, timing, and audit metadata only.
- **All computation runs on Asymmetry Computing GPU infrastructure** — your requests are processed on dedicated hardware and results are returned over HTTPS.
- **Rate limits apply** — Developer keys are rate-limited. Contact us for production capacity.

## Reproducibility Guarantee

All benchmark results are:
- **Deterministic**: Fixed random seeds, no warm start
- **Hash-verified**: SHA-256 hashes of input datasets recorded in evidence files
- **Single GPU**: No multi-GPU aggregation
- **Compute-only**: Timing excludes data loading and I/O

---

## License

All evidence data in this repository is provided for verification and due diligence purposes. The PRISM engine is proprietary software operated as a hosted service. Evidence files may be shared with prospective clients and investors under standard NDA terms.

Copyright © 2026 Asymmetry Computing. All rights reserved.

## Contact

- **API Keys & Technical**: info@asymmetrycomputing.com
- **Website**: [asymmetrycomputing.com](https://asymmetrycomputing.com)
