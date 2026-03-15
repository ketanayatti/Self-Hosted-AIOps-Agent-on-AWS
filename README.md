<div align="center">

# Self-Hosted AIOps Agent on AWS

**A production-grade, fully self-hosted AIOps micro-agent deployed and managed entirely on AWS (EC2 Free Tier) — with zero external AI API costs.**

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.100+-009688?style=for-the-badge&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![AWS](https://img.shields.io/badge/AWS-EC2%20Free%20Tier-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/free/)
[![llama.cpp](https://img.shields.io/badge/llama.cpp-CPU%20Inference-8A2BE2?style=for-the-badge)](https://github.com/ggerganov/llama.cpp)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](https://opensource.org/license/mit/)

</div>

---

## Overview

This project deploys a **self-hosted AIOps micro-agent on AWS** (EC2 t2.micro Free Tier) that provides:
- **Live infrastructure monitoring** (CPU / memory)
- **Controlled command execution** (whitelisted shell commands)
- **On-instance LLM inference** via `llama.cpp` (no OpenAI/Anthropic/external AI APIs)

Everything runs **inside your AWS environment** and is operated like a cloud workload (deploy, run, monitor, manage).

---

## Key Features

- **Cloud deployment on AWS EC2 (Free Tier friendly)**
- **FastAPI REST endpoints** for metrics, queries, and command execution
- **Local-on-instance LLM** served by `llama.cpp` (CPU only)
- **Tool-first routing**: tools for deterministic ops tasks; LLM for natural language
- **No external paid APIs** (fully self-hosted)

---

## High-Level Architecture

Client → FastAPI → Router → (System Tools / LLM Server) → Response  
All services run on the same EC2 instance (or can be separated later).

---

## Getting Started (AWS EC2)

### Prerequisites
- EC2 instance: **t2.micro** (Ubuntu 22.04 LTS recommended)
- Python 3.10+
- 1 GB RAM instances should use **swap** (recommended)
- Security group:
  - Allow **TCP 8000** (FastAPI)
  - Allow **TCP 8080** (llama.cpp server) *only if you need external access* (otherwise keep internal)

### 1) Clone the repo
```bash
git clone https://github.com/ketanayatti/Self-Hosted-AIOps-Agent-on-AWS.git
cd Self-Hosted-AIOps-Agent-on-AWS
```

### 2) Configure swap (recommended for t2.micro)
```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

### 3) Install and build llama.cpp
```bash
sudo apt update && sudo apt install -y build-essential cmake git
git clone https://github.com/ggerganov/llama.cpp /opt/llama.cpp
cd /opt/llama.cpp
cmake -B build -DLLAMA_NATIVE=OFF
cmake --build build --config Release -j$(nproc)
```

### 4) Download the model onto the EC2 instance
```bash
mkdir -p /opt/models
wget -O /opt/models/tinyllama-1.1b-chat-v1.0.Q4_K_M.gguf \
  https://huggingface.co/TheBloke/TinyLlama-1.1B-Chat-v1.0-GGUF/resolve/main/tinyllama-1.1b-chat-v1.0.Q4_K_M.gguf
```

### 5) Start the LLM server (on EC2)
```bash
/opt/llama.cpp/build/bin/llama-server \
  --model /opt/models/tinyllama-1.1b-chat-v1.0.Q4_K_M.gguf \
  --host 0.0.0.0 --port 8080 \
  --ctx-size 2048 --threads 1 --n-predict 256 &
```

### 6) Install Python dependencies
```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### 7) Start the AIOps Agent API (on EC2)
```bash
uvicorn agent:app --host 0.0.0.0 --port 8000 --reload
```

---

## API

### `GET /health`
```bash
curl http://<ec2-public-ip>:8000/health
```

### `GET /metrics`
```bash
curl http://<ec2-public-ip>:8000/metrics
```

### `POST /query`
```bash
curl -X POST http://<ec2-public-ip>:8000/query \
  -H "Content-Type: application/json" \
  -d '{"prompt": "What is the current memory usage?"}'
```

### `POST /exec`
```bash
curl -X POST http://<ec2-public-ip>:8000/exec \
  -H "Content-Type: application/json" \
  -d '{"command": "df -h"}'
```

---

## Project Structure

```
Self-Hosted-AIOps-Agent-on-AWS/
├── agent.py
├── tools/
│   ├── metrics.py
│   └── shell.py
├── llm/
│   └── client.py
├── requirements.txt
└── docs/
```

---

## License

MIT
