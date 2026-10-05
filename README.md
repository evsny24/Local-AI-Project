# Local AI Project Setup

An overview of the setup and configuration process for deploying a private, hardware-accelerated local AI environment on Windows. The goal of this project was to establish a self-hosted LLM interface accessible securely from anywhere, optimized specifically on an AMD graphics card.

## System Baseline & Core Tooling

The initial phase involved installing the core version control, runtime environments, and containerization layers required to support the local stack:

* **Git & Docker Desktop:** Installed to manage repositories and isolate the web interface layer.
* **Ollama:** Installed directly on the host Windows environment rather than inside a container. 

Because Docker on Windows does not support native GPU passthrough for AMD hardware, running Ollama as a bare-metal host service was necessary to ensure the system could utilize the 16GB VRAM on the graphics card.

## Hardware-Specific Model Selection

With 16GB of available VRAM, the model selection strategy focused on maximizing intelligence while maintaining local execution speeds. 

1. **General Text Performance:** Evaluated and selected the Qwen 2.5 model family as the optimal balance for general-purpose tasks, fitting comfortably within the hardware envelope.
2. **Vision Capabilities:** Pulled a dedicated vision model (such as Qwen2.5-VL or Llama 3.2 Vision) to enable multi-modal capabilities. This allows the backend infrastructure to automatically trigger and switch to a vision model the moment an image is uploaded.

The models were retrieved locally via the host terminal:

```cmd
ollama run qwen2.5
ollama pull qwen2.5-vl
```

## Application Containerization & Interconnect

With the AI backend running on the host, the next phase involved pulling and configuring the Odysseus application layer via Docker to serve as the primary user interface. 

To bridge the gap between the containerized UI and the host-bound Ollama instance, the container environment variables were mapped to point to the host machine's local IP network rather than `localhost`, allowing Odysseus to send inference requests directly to the AMD-accelerated host backend.

```cmd
docker pull pewdiepie/odysseus
docker run -d -p 3000:3000 --name odysseus -v odysseus_data:/app/data pewdiepie/odysseus
```

## Secure Remote Access

To make the entire setup securely accessible outside the home network without exposing open ports to the public internet, Tailscale was integrated into the machine. 

By initializing a private mesh VPN, the local AI interface can be securely reached from any authorized external device (such as a phone or remote laptop) by routing traffic directly through the assigned Tailscale IP address.

```cmd
tailscale up
```
