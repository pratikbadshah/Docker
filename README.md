# Local LLM Stack Setup Guide

Run a local LLM backend (Ollama) with a web UI (Open WebUI), accelerated by your NVIDIA GPU, using Docker.

## Prerequisites

- Ubuntu (or Debian-based) host
- NVIDIA GPU with drivers already installed
- `sudo` access

## Table of Contents

1. [Install Docker Engine](#1-install-docker-engine)
2. [Set Up NVIDIA GPU Acceleration Toolkit](#2-set-up-nvidia-gpu-acceleration-toolkit)
3. [Run the LLM Backend (Ollama)](#3-run-the-llm-backend-ollama)
4. [Run the Web Interface (Open WebUI)](#4-run-the-web-interface-open-webui)
5. [Access and Download Models](#5-access-and-download-models)

---

## 1. Install Docker Engine

```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable --now docker
```

This installs Docker and starts the service on boot.

---

## 2. Set Up NVIDIA GPU Acceleration Toolkit

Add the NVIDIA Container Toolkit repo and install it so Docker containers can access the GPU.

```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

Verify GPU access from inside a container:

```bash
docker run --rm --gpus all nvidia/cuda:12.0.0-base-ubuntu22.04 nvidia-smi
```

You should see your GPU listed in the output. If not, stop here and fix your NVIDIA driver / toolkit installation before continuing.

---

## 3. Run the LLM Backend (Ollama)

```bash
docker run -d --gpus=all -v ollama:/root/.ollama -p 11434:11434 --name ollama --restart unless-stopped ollama/ollama
```

This starts Ollama in the background, with GPU access and persistent storage in the `ollama` volume, listening on port `11434`.

---

## 4. Run the Web Interface (Open WebUI)

```bash
docker run -d -p 3000:8080 --network=host -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:ollama
```

> **Note:** `--network=host` makes the container share the host's network stack, which means the `-p 3000:8080` port mapping is ignored. With `--network=host`, Open WebUI will actually be reachable on **port 8080** (the container's own port), not 3000. If you want it on port 3000 instead, drop `--network=host` and keep the `-p 3000:8080` mapping.

---

## 5. Access and Download Models

1. Open your browser and go to:
   ```
   http://localhost:8080
   ```
2. Sign up for a local account on the Open WebUI welcome screen.
3. Pull a model via the terminal:
   ```bash
   docker exec -it ollama ollama run llama3.1
   ```
4. Refresh the Open WebUI browser tab (`http://localhost:8080`).
5. Select the model from the dropdown menu.
6. Start chatting — you're now running a fully local LLM stack.

---

## Summary

| Component | Port | Purpose |
|---|---|---|
| Ollama | 11434 | LLM backend / model runner |
| Open WebUI | 8080 | Chat interface |

## Troubleshooting

- **`nvidia-smi` fails in Docker** — check that NVIDIA drivers are installed on the host and the toolkit was configured correctly (Step 2).
- **Open WebUI won't load** — confirm which port it's actually bound to; see the note in Step 4 about `--network=host`.
- **Model pull is slow** — large models can take a while depending on your connection; progress is shown in the terminal.
