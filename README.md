
This repository documents my personal HomeLab infrastructure, serving as a hub for development, system administration practice, and machine learning experimentation.

## Hardware Specifications
* **CPU**: Intel Core i3-10100F (4 cores, 8 threads) @ 4.30 GHz
* **GPU**: NVIDIA GeForce GTX 1050 Ti (4GB VRAM)
* **RAM**: 8 GB DDR4
* **Storage**: 512 GB SSD

## Infrastructure & Architecture
The lab is built on **Proxmox VE**, utilizing **LXC Containers** and **Docker** for service management. Remote access and secure networking are handled via **Tailscale** (Mesh VPN), ensuring privacy without exposing services to the public internet.


## Services

| Service | Category | Purpose |
| :--- | :--- | :--- |
| **Tailscale** | Networking | Secure Mesh VPN & remote access |
| **Portainer** | DevOps | Docker orchestration & management |
| **Jellyfin** | Media | Self-hosted media streaming |
| **Immich** | Photos | High-performance AI photo/video backup |
| **Nextcloud** | Cloud | Personal file synchronization & productivity |
| **Home Assistant** | IoT | Centralized smart home automation |

## Documentation
* [Hardware Specs](./hardware/hardware.md) - Detailed breakdown of the physical machine.
* [Docker Services](./services/) - Infrastructure-as-Code configurations.

## Security Policy
* **No credentials stored**: Sensitive files like `.env`, SSH keys, and configuration secrets are strictly ignored via `.gitignore`.
* **Zero Trust**: Access to the lab is restricted to the private Tailscale network.

---

### Getting Started
To deploy any of the services provided in this repo:
1. Clone this repository to your server.
2. Navigate to the specific service directory: `cd services/<service-name>`.
3. Launch the container: `docker compose up -d`.
