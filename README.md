# Infrastructure Lab

A hands-on infrastructure lab built to develop and demonstrate practical skills in virtualization, Linux administration, networking, automation, monitoring and infrastructure-as-code.

The environment is centered around **Proxmox VE** and combines virtual machines, containers, managed networking and self-hosted services. The goal is to build infrastructure that is reproducible, understandable and realistic to operate in a small environment.

> This repository is a sanitized public portfolio version of the lab. Internal addresses, credentials and other sensitive configuration are intentionally excluded.

## Architecture Overview

```
Internet
   |
   v
UniFi Gateway
   |
   v
Managed Switching
   |
   +-- Proxmox VE
   |    |
   |    +-- Home Assistant VM
   |    +-- Linux VMs / LXCs
   |    +-- Tailscale
   |    +-- Monitoring services
   |
   +-- Infrastructure devices
   +-- Client devices
   +-- IoT devices
```

See the full architecture overview here:

[View architecture diagram](diagrams/architecture.md)

The design is evolving towards clearer network segmentation, centralized monitoring and repeatable server configuration.

## Design Goals

- Keep the environment simple enough to operate and troubleshoot
- Separate infrastructure, clients and IoT where it provides useful security boundaries
- Prefer automation over repetitive manual configuration
- Keep infrastructure configuration documented and version controlled
- Build monitoring into the environment instead of treating it as an afterthought
- Use virtualization to isolate services and make recovery easier
- Understand the complete path from client and DNS to network, host and application

## Technology Stack

| Area | Technologies |
| --- | --- |
| Virtualization | Proxmox VE, KVM/QEMU, LXC |
| Linux | Ubuntu Server, Rocky Linux |
| Automation | Ansible |
| Infrastructure as Code | Terraform |
| Networking | UniFi, managed switching, VLANs, DNS, routing, firewalling |
| Remote Access | Tailscale |
| Monitoring | Grafana, Prometheus |
| Smart Home | Home Assistant |
| Version Control | Git, GitHub |

## Virtualization

**Proxmox VE** is used as the primary hypervisor.

Services are placed in VMs, containers or LXC depending on their requirements. The aim is not to containerize everything, but to choose the simplest isolation model that fits each workload.

Examples:

- **Home Assistant** runs as a dedicated VM
- Lightweight infrastructure services can run in **LXC containers**
- General-purpose Linux workloads can run as separate **VMs**
- Long-running infrastructure services are prioritized for hosts that remain available 24/7

## Networking

The lab uses a **UniFi gateway** together with managed switching.

Current networking work includes:

- VLAN design and network segmentation
- Isolation of IoT devices
- Firewall policy between network segments
- Internal service access
- DNS and routing troubleshooting
- Secure remote access through Tailscale

The network design is intentionally being developed incrementally so each security boundary can be tested before additional complexity is introduced.

## Automation

### Ansible

Ansible is used to move Linux server configuration away from one-off manual changes.

The structure includes concepts such as:

```
ansible/
├── inventory/
├── group_vars/
├── host_vars/
├── roles/
└── playbooks/
```

This allows common configuration to be shared while keeping host-specific settings separate.

### Terraform

Terraform is being introduced for infrastructure provisioning and repeatability.

The goal is to separate:

- **Terraform** — infrastructure provisioning
- **Ansible** — operating system and service configuration

This keeps responsibilities clear and makes the environment easier to reproduce.

## Monitoring

The monitoring stack is built around **Prometheus and Grafana**.

The objective is to monitor infrastructure rather than only checking services when something breaks.

Areas being developed include:

- Host availability
- CPU and memory usage
- Storage utilization
- Service health
- Dashboards
- Alerting and notifications

## Security

Security improvements are introduced with a focus on practical risk reduction rather than unnecessary enterprise complexity.

Current areas include:

- Network segmentation
- IoT isolation
- Restricting unnecessary inter-network access
- VPN-based remote access
- Minimizing externally exposed services
- Separating administrative services from general client access
- Avoiding secrets and sensitive infrastructure data in public repositories

## Documentation

This repository will contain documentation covering both the final architecture and the reasoning behind key decisions.

Planned documentation:

```
docs/
├── architecture.md
├── networking.md
├── automation.md
├── monitoring.md
└── security.md
```

The focus is not only on **what** was configured, but also **why** a particular design was chosen and how it was validated.

## What This Project Demonstrates

This lab is intended to demonstrate practical experience with:

- Linux server administration
- Proxmox virtualization
- VM and container architecture
- Network design and troubleshooting
- Infrastructure automation
- Infrastructure-as-code
- Monitoring and observability
- Secure remote access
- Documentation and change management
- End-to-end infrastructure troubleshooting

## Project Status

**Active / evolving**

The environment is continuously being improved as new services, automation and network segmentation are introduced.

Future updates to this repository will include architecture diagrams, sanitized configuration examples and deeper documentation of individual infrastructure decisions.