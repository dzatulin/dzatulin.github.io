---
title: "Self-Hosted LLMs on GPUs in Kubernetes for Small Teams"
date: 2025-08-09
toc: true
toc_sticky: true
classes: wide
author_profile: true
categories: [Blog]
tags: [MLOps, LLM, GPU]
---

Need a chat bot without external APIs, with predictable cost and full data control? You can run an LLM _inside your cluster_ with a small, production-minded stack: `llama.cpp` + `FastAPI` on a virtual GPU(vGPU).

GitHub: [dzatulin/llama-fastapi-k8s-gpu](https://github.com/dzatulin/llama-fastapi-k8s-gpu).

## Architecture

```
User → Website → (Ingress/ALB) → FastAPI (Gunicorn/Uvicorn)
                                    ↓
                               llama.cpp (CUDA)
                                    ↓
                       Model file (GGUF) from S3 via initContainer
                                    ↓
                              GPU node in K8s
```

- CUDA base image + `llama-cpp-python` (CUBLAS).

**Dockerfile.base**

```
FROM nvidia/cuda:12.2.2-cudnn8-devel-ubuntu22.04

ENV DEBIAN_FRONTEND=noninteractive
ENV PATH="/usr/local/cuda/bin:$PATH"
ENV CUDAToolkit_ROOT="/usr/local/cuda"
...

RUN CMAKE_ARGS="-DLLAMA_CUBLAS=on" pip install llama-cpp-python==0.2.77 \
    --extra-index-url https://github.com/abetlen/llama-cpp-python/releases/download/v0.2.77-cu122/llama_cpp_python-0.2.77-cp310-cp310-linux_x86_64.whl \
    --verbose
...
```

- FastAPI (Gunicorn worker) with **queue + semaphore + timeouts + context trimming** → stable latency, no GPU thrash.
- **initContainer** pulls a quantized `.gguf` from S3 into an `emptyDir` volume → simple startup without persistent PVs.

```
        - name: init-s3-copy
          image: amazon/aws-cli:2.13.17
          command: ["sh", "-c"]
          args:
            - |
              aws s3 cp s3://ai-models/Llama-3-8B-Instruct-Q4_K_M.gguf /app/models/ &&
              echo "Model downloaded successfully"
```

- Deployment uses `tolerations` for `nvidia.com/gpu` and a GPU node pool selector.

## Why self-host an LLM?

- **Cost control & privacy.** Keep sensitive prompts/responses inside your infra and scale GPUs only when needed.
- **Predictable latency.** Place inference close to your app, avoid external quotas.
- **Ops levers.** You own the probes, dashboards, rollout strategy, and model lifecycle.

## Observability

This stack proved reliable in practice: Prometheus + Grafana for numbers, Loki for logs. The goal is to see real model performance (latency, tokens/sec) and catch regressions fast.

**Metrics — what to track**

- **User-facing:** p50/p95/p99 latency, tokens/sec, request rate, success/error rate. These show the model’s real throughput and perceived speed.
- **GPU health:** GPU utilization, **VRAM usage/pressure**, power, throttling reasons. Early signal for capacity issues.
- **System & queueing:** CPU/RAM, pod restarts, network egress.

**Logs**

Use structured request logs (prompt size, latency, response length). Exclude any raw PII. Loki works well for aggregation and search.

**Alerts**

- **Availability/SLO:** higt 5xx error rate, p99 latency or TTFT breaching SLO, readiness probe flapping.
- **GPU Health:** VRAM >95% for N minutes, higt GPU load.
- **Stability:** OOMKilled/evictions, crash loops, model load failures.

## Common problems

- **Autoscaling pods is non-trivial.** CPU isn’t a good signal for LLMs; you need a custom metric (e.g., `inflight_requests`,GPU load, queue depth, tokens/sec).
- **Long model load time.** Big GGUFs slow down rollout and readiness; first-token latency spikes after scale-out.
- **Low RPS on budget GPUs.** Inference is compute/VRAM-bound; cheap cards = low concurrency.
- **CUDA/NVIDIA driver, llama.cpp/llama-cpp-python**, and the device plugin must align. Version drift = crashes or silent slowdowns.
