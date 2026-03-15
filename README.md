<div align="center">

<img
  src="docs/images/banner.png"
  alt="Self-Hosted AIOps Agent on AWS"
  width="900"
  loading="lazy"
/>

# 🤖 Self-Hosted AIOps Agent on AWS Free Tier

**A lightweight, production-ready AIOps micro-agent running entirely on AWS Free Tier infrastructure**

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue?logo=python)](https://python.org)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.110%2B-green?logo=fastapi)](https://fastapi.tiangolo.com)
[![AWS EC2](https://img.shields.io/badge/AWS-EC2%20t2.micro-orange?logo=amazon-aws)](https://aws.amazon.com/ec2/)
[![llama.cpp](https://img.shields.io/badge/LLM-llama.cpp-blueviolet)](https://github.com/ggerganov/llama.cpp)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

</div>

---

## 📖 Overview

I designed and deployed a **lightweight AIOps micro-agent** running entirely on **AWS Free Tier** infrastructure. This project demonstrates how to build a fully self-hosted, AI-powered operations assistant — no external paid APIs required.

The agent monitors infrastructure health, executes secure shell commands, and routes natural-language requests through a local LLM — all on a single `t2.micro` instance with **1 vCPU and 1 GB RAM**.

---

## 🏗 Architecture

<div align="center">

<img
  src="docs/images/architecture.png"
  alt="AIOps Agent Architecture: Client → FastAPI Agent → Tool Layer → Local LLM Server → Response"
  width="800"
  loading="lazy"
/>

</div>

```
Client
  │
  ▼
FastAPI Agent  ──► Tool Layer ──► [CPU Monitor]
  │                              [Memory Monitor]
  │                              [Shell Executor]
  │
  ▼
Local LLM Server (llama.cpp + TinyLlama 1.1B Q4)
  │
  ▼
Response
```

---

## 🔧 Tech Stack

| Component | Technology |
|-----------|-----------|
| **Cloud** | AWS EC2 t2.micro (Free Tier) |
| **LLM Runtime** | llama.cpp (CPU-only inference) |
| **Model** | TinyLlama 1.1B (Q4 quantized) |
| **Backend** | FastAPI |
| **Routing** | Custom tool-based agent layer |
| **OS** | Ubuntu 22.04 LTS |

---

## ⚙️ What It Can Do

- ✅ **Monitor live CPU & memory usage** in real time
- ✅ **Execute shell commands** securely via the REST API
- ✅ **Route requests** intelligently between tools and the LLM
- ✅ **Serve AI responses** through a clean REST API
- ✅ **Run fully self-hosted** — zero external paid APIs

---

## 🖥 Screenshots

<div align="center">

<img
  src="docs/images/api-health-check.png"
  alt="FastAPI health check endpoint returning system status"
  width="760"
  loading="lazy"
/>

<p><em>FastAPI health-check endpoint returning live CPU and memory status</em></p>

</div>

<div align="center">

<img
  src="docs/images/llm-inference.png"
  alt="TinyLlama LLM inference response in the terminal"
  width="760"
  loading="lazy"
/>

<p><em>TinyLlama 1.1B running CPU inference at ~10-12 tokens/sec</em></p>

</div>

<div align="center">

<img
  src="docs/images/agent-routing.png"
  alt="Agent routing a natural-language request to the correct tool"
  width="760"
  loading="lazy"
/>

<p><em>Agent routing layer dispatching a natural-language query to the right tool</em></p>

</div>

---

## 🚀 Getting Started

### Prerequisites

- AWS account (Free Tier eligible)
- EC2 `t2.micro` instance running Ubuntu 22.04 LTS
- Python 3.10 or later
- At least 2 GB swap space configured (for LLM inference on 1 GB RAM)

### 1. Launch EC2 Instance

```bash
# Configure at least 2 GB swap to handle LLM inference pressure
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

### 2. Install llama.cpp

```bash
git clone https://github.com/ggerganov/llama.cpp.git
cd llama.cpp
make -j$(nproc)
```

### 3. Download TinyLlama (Q4 quantized)

```bash
wget https://huggingface.co/TheBloke/TinyLlama-1.1B-Chat-v1.0-GGUF/resolve/main/tinyllama-1.1b-chat-v1.0.Q4_K_M.gguf \
  -O models/tinyllama.gguf
```

### 4. Clone & Install the Agent

```bash
git clone https://github.com/ketanayatti/Self-Hosted-AIOps-Agent-on-AWS.git
cd Self-Hosted-AIOps-Agent-on-AWS
pip install -r requirements.txt
```

### 5. Start the LLM Server

```bash
./llama.cpp/server \
  -m models/tinyllama.gguf \
  --ctx-size 2048 \
  --threads 1 \
  --host 127.0.0.1 \
  --port 8080
```

### 6. Launch the FastAPI Agent

```bash
uvicorn agent:app --host 0.0.0.0 --port 8000
```

---

## 📡 API Reference

### Health Check

```http
GET /health
```

Returns live CPU and memory metrics.

**Response:**

```json
{
  "status": "ok",
  "cpu_percent": 12.4,
  "memory_percent": 74.1,
  "uptime_seconds": 86400
}
```

### Run a Query

```http
POST /query
Content-Type: application/json

{
  "prompt": "What is the current CPU usage?"
}
```

**Response:**

```json
{
  "tool": "cpu_monitor",
  "result": "Current CPU usage is 12.4%.",
  "tokens_per_sec": 11.3
}
```

### Execute Shell Command

```http
POST /exec
Content-Type: application/json

{
  "command": "df -h /"
}
```

---

## 🔥 Key Challenges & Solutions

| Challenge | Solution |
|-----------|----------|
| 1 GB RAM for LLM inference | 2 GB swap + Q4 quantization |
| CPU-only token generation | Single-threaded llama.cpp server, 10-12 tok/s |
| Secure shell execution | Allowlist-based command validation |
| Low latency routing | Rule-based pre-filter before LLM fallback |

---

## 🔭 Next Steps

- [ ] **RAG-based knowledge memory** — attach a vector store for log history retrieval
- [ ] **Multi-step automation workflows** — chain tool calls for complex remediation
- [ ] **Secure authentication layer** — API key / JWT middleware for the FastAPI app
- [ ] **Grafana dashboard** — visualize CPU, memory and token-throughput over time

---

## 💡 What This Demonstrates

| Skill | Evidence |
|-------|---------|
| **AIOps system design** | End-to-end agent with tool routing and LLM fallback |
| **Self-hosted LLM deployment** | llama.cpp + quantized model on constrained hardware |
| **Infrastructure-aware AI** | Real-time CPU/memory telemetry piped into the agent |
| **Backend + DevOps integration** | FastAPI, systemd, swap management on EC2 |
| **Resource-constrained optimisation** | Stable inference on 1 vCPU / 1 GB RAM |

---

## 📂 Project Structure

```
Self-Hosted-AIOps-Agent-on-AWS/
├── agent.py            # FastAPI app & request router
├── tools/
│   ├── cpu_monitor.py  # CPU usage tool
│   ├── mem_monitor.py  # Memory usage tool
│   └── shell_exec.py   # Secure shell execution tool
├── llm_client.py       # llama.cpp HTTP client wrapper
├── requirements.txt    # Python dependencies
└── docs/
    └── images/         # Architecture diagrams & screenshots
```

---

## 🤝 Contributing

Pull requests are welcome! Please open an issue first to discuss what you would like to change.

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).

---

<div align="center">

*Built with ❤️ on AWS Free Tier · No paid APIs · Fully self-hosted*

</div>
