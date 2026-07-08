# HomeLab Infrastructure

This repository documents my personal HomeLab infrastructure, serving as a hub for development, system administration, and machine learning experiments.

## Hardware Specifications
* **CPU**: Intel Core i3-10100F (4 cores, 8 threads) @ 4.30 GHz
* **GPU**: NVIDIA GeForce GTX 1050 Ti (4GB VRAM)
* **RAM**: 8 GB DDR4
* **Storage**: 512 GB SSD

## Architecture
The lab is built on **Proxmox VE**, utilizing **LXC** containers and **Docker** for service management. Secure access is achieved via **Tailscale** (Mesh VPN), ensuring privacy without exposing services directly to the internet.

```mermaid
graph TD
    User((User)) --> Nginx[Nginx Reverse Proxy]
    
    subgraph "Core Services"
        Nginx --> CrowdSec[CrowdSec Security]
        Nginx --> Vaultwarden[Vaultwarden]
        Nginx --> Immich[Immich]
        Nginx --> N8N[N8N Automations]
    end

    subgraph "Monitoring & Ops"
        Nginx --> Portainer[Portainer]
        Nginx --> Kuma[Uptime Kuma]
        Nginx --> Grafana[Grafana/Prometheus]
    end
    
    CrowdSec -.-> Nginx
