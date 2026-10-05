# Local AI Project Setup

An overview of the setup and configuration process for deploying a private, local AI environment on Windows with an AMD graphics card. The goal of this project was to establish a self-hosted LLM interface accessible securely from anywhere to escape the rate limits of cloud models and to experiment with agent capabilities.

## System Baseline & Core Tooling

The initial phase involved installing the runtime environment and and containerization layers required to support the local stack:

- **Docker Desktop:** Installed to isolate the web interface layer.
  
<img width="50%" height="50%" alt="image" src="https://github.com/user-attachments/assets/0c2b5ce9-d794-41a5-b8b2-3dc3b6997662" />

- **Ollama:** Installed directly on the host Windows environment.
  
<img width="50%" height="50%" alt="image" src="https://github.com/user-attachments/assets/71234b61-5b7e-49db-a11b-b822d25d5dd2" />


Because Docker on Windows does not support native GPU passthrough for AMD hardware, running Ollama as a bare-metal host service was necessary to ensure the system could use the graphics card.

## Application Containerization & Interconnect

With the AI backend going to be running on the host, the next phase involved pulling and configuring the Odysseus application layer via Docker to serve as the primary user interface. 

Why Odysseus and not just Ollama?
- It was a project set on by Felix Kjellberg (Pewdiepie) who used to make YouTube videos about gaming, but has now transitioned to developing projects like these. I used to watch his gaming videos, and now he has inspired me to get into local AI as well.
- It has a ton of features like deep research, sandboxed agent mode, and it is accessed through a web interface.

<img width="50%" height="50%" alt="image" src="https://github.com/user-attachments/assets/c844d8c1-767a-4556-bd16-d72dbb75554e" />

1. **Setup:** I used the easiest method to install, which is docker compose in the odysseus folder.
```cmd
docker compose up -d --build
```
2. I had to edit the docker-compose.yaml file to increase the default context limit since responses would cut off in the middle of long tasks.
```yaml
environment:
  - OLLAMA_CONTEXT_LENGTH=64000
  - OLLAMA_NUM_PREDICT=8192
``` 
3. I also had to map my external projects folder to the docker container filesystem so changes could be made on my filesystem rather than the docker local filesystem.
```yaml
volumes:
  - C:/path_to_local_projects_folder:/app/projects
```
then reboot
```cmd
docker compose down
docker compose up -d
```


## Hardware-Specific Model Selection

With 16GB of available VRAM, my model selection strategy focused on maximizing intelligence while maintaining local execution speeds. 

1. **Testing hardware limits:** I wanted to push my hardware to the limit and downloaded a Qwen-3.5:24b model which uses about 17GB of memory. Although, since it couldn't run entirely on the 16GB GPU, it had to offload some to the RAM which tanked my speeds to 9 tokens per second.
2. **General Text Performance:** I tried the smaller Qwen-3.5:9b model and found it to be okay for general-purpose tasks, fitting comfortably within limit, running at about 90 tokens per second. However, in agent mode, it had issues parsing the commands it needed to edit and create files.
3. **Model for Agenting:** I eventually settled on gemma4:12b which was able to perform the agent commands and also had decent speed around 60 tokens per second.
4. **Vision Capabilities:** I also pulled a dedicated vision model, Qwen3-VL:8b to enable multi-modal capabilities. This allows the backend infrastructure to automatically trigger and switch to a vision model to analyze images I upload before passing them back to my main model.

The models were retrieved locally via the host terminal, NOT on odysseus's cookbook:

```cmd
ollama pull gemma4:12b
```

## Easy startup of docker desktop, odysseus container, and ollama

I thought it would be too much work to be clicking around my computer to start all these processes each time I wanted to use it. Therefore, I created a simple .bat file that opens everything that it needs to run:

```cmd
@echo off

echo [1/3] Starting Ollama ...
start "" /B "C:\PATH_TO_OLLAMA\Ollama\ollama app.exe"

echo [2/3] Starting Docker Desktop ...
start "" "C:\PATH_TO_DOCKER\DockerDesktop\Docker Desktop.exe"

echo Waiting for Docker Engine to fully boot up ...
:wait_loop
docker ps >nul 2>&1
if %errorlevel% neq 0 {
timeout /t 3 /nobreak >nul
goto wait_loop
}

echo [3/3] Docker is ready! Launching Odysseus ...
cd /d "C:\PATH_TO_ODYSSEUS\Odysseus\odysseus"
docker compose up -d

echo Done! You can close this window now.
pause
```

## Secure Remote Access

To make the entire setup securely accessible outside my network without being able to access the router I am connected to, I used Tailscale and connected all my devices to a tailnet. 

By initializing this private mesh VPN, my local AI interface can be securely reached from any authorized external device like my phons by routing traffic directly through the assigned Tailscale IP address.

<img width="100%" height="100%" alt="image" src="https://github.com/user-attachments/assets/06300f00-4c97-4186-a560-51ca8e8c8859" />
