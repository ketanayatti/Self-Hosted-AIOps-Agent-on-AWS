<div align="center">

# 🤖 Self-Hosted AIOps Agent on AWS

**A production-grade, fully self-hosted AI Operations micro-agent — running on AWS Free Tier with zero external API costs.**

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.100+-009688?style=for-the-badge&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![AWS](https://img.shields.io/badge/AWS-EC2%20Free%20Tier-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/free/)
[![llama.cpp](https://img.shields.io/badge/llama.cpp-CPU%20Inference-8A2BE2?style=for-the-badge)](https://github.com/ggerganov/llama.cpp)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](LICENSE)

> *Practical AIOps. No cloud AI subscriptions. No GPU. Just pure engineering.*

</div>

---

## 📸 Screenshots

> Project screenshots are located in the [`docs/`](docs/) folder.

| Agent API Response | System Metrics Dashboard | Architecture Overview |
|---|---|---|
| ![API Response](docs/api-response.png) | ![Metrics](docs/metrics-dashboard.png) | ![Architecture](docs/architecture-diagram.png) |

---

## 📖 Overview

This project demonstrates a **lightweight, self-hosted AIOps micro-agent** designed to run entirely on **AWS Free Tier** infrastructure (EC2 t2.micro — 1 vCPU, 1 GB RAM). It combines a local Large Language Model (LLM) running via `llama.cpp` with a **FastAPI** backend and a custom **tool-based routing layer** to deliver real operational intelligence without any external paid APIs.

The agent can monitor live system metrics, execute shell commands, and answer natural-language operations queries — all from a single, resource-constrained server.

---

## ✨ Features

| Capability | Description |
|---|---|
| 📊 **Live System Monitoring** | Real-time CPU and memory usage tracking |
| 🖥️ **Secure Shell Execution** | Execute whitelisted shell commands via REST API |
| 🧠 **Local LLM Inference** | TinyLlama 1.1B (Q4 quantized) running fully on CPU |
| 🔀 **Tool-Based Routing** | Intelligent routing between tools and LLM based on query intent |
| 🌐 **REST API Interface** | Clean FastAPI endpoints for easy integration |
| 🔒 **Zero External Dependencies** | Fully self-hosted — no OpenAI, no Anthropic, no paid APIs |
| ⚡ **Optimized for Constraints** | Tuned for ~10–12 tokens/sec on 1 GB RAM with swap |

---

## 🏗️ Architecture

```
┌────────────────────────────────────────────────────────────┐
│                      AWS EC2 t2.micro                      │
│                  (1 vCPU · 1 GB RAM · Swap)                │
│                                                            │
│  ┌──────────┐    ┌───────────────┐    ┌─────────────────┐  │
│  │  Client  │───▶│  FastAPI App  │───▶│   Tool Router   │  │
│  │ (HTTP)   │    │  (Port 8000)  │    │                 │  │
│  └──────────┘    └───────────────┘    └────────┬────────┘  │
│                                                │           │
│                          ┌─────────────────────┤           │
│                          │                     │           │
│                   ┌──────▼──────┐    ┌─────────▼───────┐  │
│                   │  LLM Server │    │   System Tools   │  │
│                   │ (llama.cpp) │    │ CPU · MEM · Shell│  │
│                   │ TinyLlama1B │    │                 │  │
│                   └─────────────┘    └─────────────────┘  │
└────────────────────────────────────────────────────────────┘
```

**Request Flow:**
```
Client Request
     │
     ▼
FastAPI Agent  ──▶  Intent Classifier
                          │
              ┌───────────┴───────────┐
              ▼                       ▼
        Tool Layer              Local LLM
    (CPU/MEM/Shell)         (TinyLlama 1.1B)
              │                       │
              └───────────┬───────────┘
                          ▼
                   Unified Response
```

---

## 🔧 Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| **Compute** | AWS EC2 t2.micro | Free Tier cloud host |
| **OS** | Ubuntu 22.04 LTS | Base operating system |
| **LLM Runtime** | [llama.cpp](https://github.com/ggerganov/llama.cpp) | CPU-optimised LLM inference |
| **Model** | TinyLlama 1.1B (Q4_K_M) | Lightweight, quantized chat model |
| **Backend** | FastAPI + Uvicorn | High-performance async API server |
| **Monitoring** | psutil | Cross-platform system metrics |
| **Language** | Python 3.10+ | Core application language |

---

## 🚀 Getting Started

### Prerequisites

- AWS EC2 instance (**t2.micro**, Ubuntu 22.04 LTS) or any Linux server
- Python 3.10+
- At least **2 GB swap** configured (critical for 1 GB RAM instances)
- `cmake`, `gcc`, `g++` for building llama.cpp

---

### 1. Clone the Repository

```bash
git clone https://github.com/ketanayatti/Self-Hosted-AIOps-Agent-on-AWS.git
cd Self-Hosted-AIOps-Agent-on-AWS
```

### 2. Configure Swap (for 1 GB RAM instances)

```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

### 3. Install llama.cpp

```bash
sudo apt update && sudo apt install -y build-essential cmake git
git clone https://github.com/ggerganov/llama.cpp /opt/llama.cpp
cd /opt/llama.cpp
cmake -B build -DLLAMA_NATIVE=OFF
cmake --build build --config Release -j$(nproc)
```

### 4. Download TinyLlama Model

```bash
mkdir -p /opt/models
# Download TinyLlama 1.1B Q4_K_M GGUF
wget -O /opt/models/tinyllama-1.1b-chat-v1.0.Q4_K_M.gguf \
  https://huggingface.co/TheBloke/TinyLlama-1.1B-Chat-v1.0-GGUF/resolve/main/tinyllama-1.1b-chat-v1.0.Q4_K_M.gguf
```

### 5. Start the LLM Server

```bash
/opt/llama.cpp/build/bin/llama-server \
  --model /opt/models/tinyllama-1.1b-chat-v1.0.Q4_K_M.gguf \
  --host 0.0.0.0 --port 8080 \
  --ctx-size 2048 --threads 1 --n-predict 256 &
```

### 6. Install Python Dependencies

```bash
cd /path/to/Self-Hosted-AIOps-Agent-on-AWS
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### 7. Start the AIOps Agent

```bash
uvicorn agent:app --host 0.0.0.0 --port 8000 --reload
```

---

## 📡 API Reference

### `GET /health`
Health check endpoint.

```bash
curl http://<your-ec2-ip>:8000/health
```
```json
{ "status": "ok", "llm": "online", "tools": "ready" }
```

---

### `GET /metrics`
Returns live CPU and memory stats.

```bash
curl http://<your-ec2-ip>:8000/metrics
```
```json
{
  "cpu_percent": 12.4,
  "memory": {
    "total_mb": 983,
    "used_mb": 712,
    "percent": 72.4
  }
}
```

---

### `POST /query`
Send a natural-language or structured query to the agent.

```bash
curl -X POST http://<your-ec2-ip>:8000/query \
  -H "Content-Type: application/json" \
  -d '{"prompt": "What is the current memory usage?"}'
```
```json
{
  "route": "tool",
  "tool": "memory_check",
  "result": "Memory usage is at 72.4% (712 MB / 983 MB).",
  "latency_ms": 18
}
```

---

### `POST /exec`
Execute a whitelisted shell command.

```bash
curl -X POST http://<your-ec2-ip>:8000/exec \
  -H "Content-Type: application/json" \
  -d '{"command": "df -h"}'
```
```json
{
  "stdout": "Filesystem      Size  Used Avail Use% Mounted on\n/dev/xvda1       20G  6.1G   14G  31% /",
  "returncode": 0
}
```

---

## ⚙️ Performance & Optimization

Running a quantized LLM on a 1 GB RAM EC2 instance is non-trivial. Here are the key optimizations applied:

| Technique | Impact |
|---|---|
| **Q4_K_M quantization** | ~60% model size reduction vs. FP16 |
| **2 GB swap file** | Prevents OOM kills during inference |
| **Context window = 2048** | Limits peak memory footprint |
| **Single thread (--threads 1)** | Prevents CPU throttling on t2.micro |
| **Tool-first routing** | Avoids LLM calls for deterministic queries |

**Observed throughput:** ~10–12 tokens/sec on t2.micro (CPU-only)

---

## 🗺️ Roadmap

- [ ] **RAG-based Knowledge Memory** — Attach a vector store (e.g. ChromaDB) for log-aware context retrieval
- [ ] **Multi-Step Automation Workflows** — Chain tools into automated runbooks (restart service → verify → alert)
- [ ] **Secure Authentication Layer** — API key / JWT authentication for production hardening
- [ ] **Alert & Notification System** — Threshold-based alerts via webhooks (Slack, PagerDuty)
- [ ] **Metrics Dashboard** — Lightweight web UI showing live charts and agent activity
- [ ] **Docker Deployment** — Single `docker compose up` setup for reproducible environments

---

## 📁 Project Structure

```
Self-Hosted-AIOps-Agent-on-AWS/
├── agent.py                # FastAPI application & tool routing logic
├── tools/
│   ├── metrics.py          # CPU & memory monitoring (psutil)
│   └── shell.py            # Secure shell command execution
├── llm/
│   └── client.py           # llama.cpp HTTP client wrapper
├── requirements.txt        # Python dependencies
├── docs/                   # Project screenshots & diagrams
│   └── README.md           # Guide for adding screenshots
└── README.md               # This file
```

---

## 💡 What This Project Demonstrates

- **Practical AIOps system design** — A working agent architecture, not just theory
- **Self-hosted LLM deployment** — Proving LLMs don't need expensive cloud APIs
- **Infrastructure-aware AI** — Agent that understands and interacts with its own host
- **Backend + DevOps integration** — Combining software engineering with cloud operations
- **Resource-constrained optimization** — Production thinking under extreme hardware limits

---

## 🤝 Contributing

Contributions, issues and feature requests are welcome!

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Commit your changes: `git commit -m 'feat: add your feature'`
4. Push to the branch: `git push origin feature/your-feature`
5. Open a Pull Request

---

## 📜 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

---

## 👤 Author

**Ketan Ayatti**

[![GitHub](https://img.shields.io/badge/GitHub-ketanayatti-181717?style=flat-square&logo=github)](https://github.com/ketanayatti)

---

<div align="center">

*Built with 💻 on AWS Free Tier — proving that great engineering doesn't require deep pockets.*

⭐ If you found this useful, please **star the repo** — it means a lot!

</div>
