# Homelab 

This repository contains my declarative infrastructure setup for managing homelab services using Ansible. The project is structured to ensure repeatable, stable, and automated deployments across LXC containers while keeping configuration under version control.
## Project Structure

    inventory.ini: Inventory file containing the list of target LXC containers.

    group_vars/all.yml: Global configurations and environment variables.

    deploy_services.yml: Main playbook for service deployment and management.

    services/: Directory containing docker-compose.yml files for each service.

## Prerequisites

    Ansible installed on your control node.

    SSH access configured for target LXC containers.

    Python 3 installed on target nodes.

    Docker and Docker Compose V2 installed on target containers.

## Usage

    Clone the repository:
    git clone https://github.com/moanast/homelab.git

## Configure Inventory:
Edit inventory.ini and add the IP addresses of your LXC containers.

## Deploy Services:
Run the main playbook to synchronize configurations and start the containers:
Bash

    ansible-playbook -i inventory.ini deploy_services.yml

## Security Considerations

    No Secrets: This repository does not contain passwords or API keys in plain text.

    Infrastructure Isolation: The network architecture separates the firewall (pfSense/OPNsense) from the virtualization server (Proxmox), ensuring network continuity during server maintenance.

## Notes
### Infrastructure Overview (pve1)

    CPU: Intel(R) Core(TM) i3-10100F (8 threads) @ 4.30 GHz

    RAM: 8.00 GiB

    GPU: NVIDIA GeForce GTX 1050 Ti

    OS: Proxmox VE 9.2.4

    Storage: 512 GiB SATA SSD

## Managed Services

### This Ansible repository currently automates the deployment and lifecycle management of the following containerized services:  

Crowdsec: Security and intrusion prevention.  

Homepage: Dashboard for homelab management.  

Immich: Self-hosted photo and video backup solution.  

Nginx: Reverse proxy and web server.  

Uptime-Kuma: Monitoring for service availability.  

Vaultwarden: Password management.  

Note: The infrastructure is optimized for resource-constrained environments, ensuring high signal-to-compute efficiency.
