# Local AI Project Setup

An overview of the setup and configuration process for deploying a private, local AI environment on Windows with an AMD graphics card. The goal of this project was to establish a self-hosted LLM interface accessible securely from anywhere to escape the rate limits of cloud models and to experiment with agent capabilities.

## System Baseline & Core Tooling

The initial phase involved installing the runtime environment and and containerization layers required to support the local stack:

* **Docker Desktop:** Installed to isolate the web interface layer.
* **Ollama:** Installed directly on the host Windows environment. 

Because Docker on Windows does not support native GPU passthrough for AMD hardware, running Ollama as a bare-metal host service was necessary to ensure the system could utilize the 16GB VRAM on the graphics card.

## Application Containerization & Interconnect

With the AI backend going to be running on the host, the next phase involved pulling and configuring the Odysseus application layer via Docker to serve as the primary user interface. 

Why Odysseus and not just Ollama?
- It was a project set on by Felix Kjellberg (Pewdiepie) who used to make YouTube videos about gaming, but has now transitioned to developing projects like these. I used to watch his gaming videos, and now he has inspired me to get into local AI as well.

## Hardware-Specific Model Selection

With 16GB of available VRAM, my model selection strategy focused on maximizing intelligence while maintaining local execution speeds. 

1. **Testing hardware limits:** I wanted to push my hardware to the limit and downloaded a Qwen-3.5:24b model which uses about 17GB of memory. Although, since it couldn't run entirely on the 16GB GPU, it had to offload some to the RAM which tanked my speeds to 9 tokens per second.
2. **General Text Performance:** I liked how the model performed though, so I selected the smaller Qwen-3.5:9b model as the optimal balance for general-purpose tasks, fitting comfortably within limit, running at about 90 tokens per second.
3. **Vision Capabilities:** I also pulled a dedicated vision model, Qwen3-VL:8b to enable multi-modal capabilities. This allows the backend infrastructure to automatically trigger and switch to a vision model to analyze images I upload before passing them back to my main model.

The models were retrieved locally via the host terminal, NOT on odysseus's cookbook:

```cmd
ollama pull qwen3.5-vl
```

## Secure Remote Access

To make the entire setup securely accessible outside my network without being able to access the router I am connected to, I used Tailscale and connected all my devices to a tailnet. 

By initializing this private mesh VPN, my local AI interface can be securely reached from any authorized external device like my phons by routing traffic directly through the assigned Tailscale IP address.
