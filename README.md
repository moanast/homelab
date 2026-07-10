# My Homelab

Welcome to my personal Homelab documentation. This repository tracks the configuration, services, and architecture of my home server setup, currently hosted on a Proxmox VE (PVE) environment.

## Architecture Overview

The following diagram illustrates the current service architecture and how they interact within the Proxmox host.

```mermaid
graph TD
    subgraph Proxmox_Host [Proxmox PVE Host]
        Nginx[Nginx Reverse Proxy]
        Jellyfin[Jellyfin Media Server]
        Immich[Immich Photo Storage]
        Nextcloud[Nextcloud]
        Vaultwarden[Vaultwarden]
        Kuma[Uptime Kuma]
        Homepage[Homepage Dashboard]
    end

    User((User)) -->|HTTPS| Nginx
    Nginx -->|Proxy| Jellyfin
    Nginx -->|Proxy| Immich
    Nginx -->|Proxy| Nextcloud
    Nginx -->|Proxy| Vaultwarden
    Nginx -->|Proxy| Uptime Kuma
    Nginx -->|Proxy| Homepage

    Uptime Kuma -.->|Monitor| Jellyfin
    Uptime Kuma -.->|Monitor| Immich
    Uptime Kuma -.->|Monitor| Nextcloud
    Uptime Kuma -.->|Monitor| Vaultwarden
