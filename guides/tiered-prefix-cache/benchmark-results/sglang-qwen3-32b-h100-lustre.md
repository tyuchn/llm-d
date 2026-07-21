# Qwen/Qwen3-32B SGLang Lustre Filesystem Offloading Benchmark (16×H100)

This benchmark evaluates 16 × H100 GPUs, distributed across 8 model servers (2 H100s per server with TP=2) using Qwen3-32B and SGLang HiCache native filesystem offloading (`--hicache-storage-backend file`).

All results demonstrate the empirical impact of enabling multi-tier KV cache offloading (`L1 HBM <-> L2 Host CPU RAM <-> L3 Managed Lustre`) with in-memory client metadata caching enabled (`SGLANG_HICACHE_FILE_BACKEND_ENABLE_METADATA_CACHE=true`) relative to CPU-only and HBM-only serving configurations under high cache pressure where the working set exceeds host memory.

* **Workload**: 250 prefix groups, 5 prompts per group, system prompt length of 16,000 tokens, question length of 256 tokens, output length of 256 tokens.
* **GPU Cache Size (Total)**: 321,856 tokens per replica (~78.6 GiB allocated to KV cache across 2 H100 GPUs per replica).
* **CPU Cache Size (Total)**: 819,200 tokens per replica (200 GiB / replica host RAM via `--hicache-size 200`).
* **Lustre Storage (L3)**: GCP Managed Lustre PVC (`preprov-pvc` / `llm-d-kv-cache-storage`) mounted at `/mnt/files-storage`.
* **Workload Unique Cache (Working Set)**: 4,640,000 tokens (~760 GB / 708 GiB).

---

## Comparative Performance Table (160 GiB HBM + 200 GiB CPU RAM + Lustre PVC)

| Target Rate | Configuration | Mean TTFT (s) | P90 TTFT (s) | Mean E2E Latency (s) | P90 E2E Latency (s) | Throughput (tok/s) |
| :---: | :--- | :---: | :---: | :---: | :---: | :---: |
| **5.0 QPS** | HBM-only | 2.12 | 4.35 | 9.54 | 13.67 | 68,517.5 |
| | HBM + CPU RAM | 0.95 (-55.2%) | 1.39 (-68.0%) | 10.26 (+7.5%) | 18.55 (+35.7%) | 73,860.3 (+7.8%) |
| | **HBM + CPU RAM + Lustre** | **1.08 (-49.1%)** | **1.50 (-65.5%)** | **11.06 (+15.9%)** | **16.20 (+18.5%)** | **76,758.9 (+12.0%)** |
| **10.0 QPS** | HBM-only | 9.76 | 19.66 | 25.62 | 34.78 | 111,155.9 |
| | HBM + CPU RAM | 0.43 (-95.6%) | 1.27 (-93.5%) | 7.74 (-69.8%) | 11.08 (-68.1%) | 152,045.3 (+36.8%) |
| | **HBM + CPU RAM + Lustre** | **0.40 (-95.9%)** | **1.26 (-93.6%)** | **7.28 (-71.6%)** | **10.56 (-69.6%)** | **154,565.1 (+39.1%)** |
| **20.0 QPS** | HBM-only | 40.55 | 76.70 | 53.77 | 87.84 | 117,451.7 |
| | HBM + CPU RAM | 1.56 (-96.2%) | 3.99 (-94.8%) | 9.61 (-82.1%) | 12.21 (-86.1%) | 280,447.9 (+138.8%) |
| | **HBM + CPU RAM + Lustre** | **0.53 (-98.7%)** | **1.43 (-98.1%)** | **8.82 (-83.6%)** | **10.29 (-88.3%)** | **283,849.2 (+141.7%)** |
| **40.0 QPS** | HBM-only | 107.88 | 185.77 | 121.07 | 199.02 | 115,459.0 |
| | HBM + CPU RAM | 25.56 (-76.3%) | 47.25 (-74.6%) | 33.61 (-72.2%) | 54.66 (-72.5%) | 308,779.8 (+167.4%) |
| | **HBM + CPU RAM + Lustre** | **19.35 (-82.1%)** | **37.03 (-80.1%)** | **28.29 (-76.6%)** | **45.63 (-77.1%)** | **332,944.9 (+188.4%)** |
