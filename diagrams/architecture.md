# Infrastructure Architecture

The diagram below shows the current lab architecture together with the planned migration of monitoring workloads to the always-on Proxmox environment.

```mermaid
flowchart TD

    Internet["Internet / ISP"]
    UCG["UniFi Cloud Gateway Ultra"]
    USW["UniFi Flex 2.5G"]
    NETGEAR["Netgear GS724Tv4"]

    Internet --> UCG
    UCG --> USW
    USW -->|"Cat6A uplink"| NETGEAR

    subgraph PROXMOX["Always-on Proxmox Environment"]
        HA["Home Assistant VM"]
        TS["tailscale01 LXC"]
        MON01["monitoring01 VM"]
    end

    NETGEAR --> PROXMOX

    TS -->|"Subnet routing / Exit Node"| TAILNET["Tailscale"]

    subgraph WORKSTATION["Bazzite Administration Workstation"]
        DISTRO["infra-control Distrobox"]
        LAB01["lab-ubuntu01"]
        LAB02["lab-ubuntu02"]

        DISTRO -->|"Ansible / SSH"| LAB01
        DISTRO -->|"Ansible / SSH"| LAB02
        DISTRO -->|"Ansible / SSH"| MON01
    end

    subgraph CURRENT["Current Monitoring Implementation"]
        PODMAN["Rootless Podman"]
        PROM["Prometheus"]
        GRAF["Grafana"]
    end

    LAB02 --> PODMAN
    PODMAN --> PROM
    PODMAN --> GRAF

    LAB01 -->|"Node Exporter :9100"| PROM
    LAB02 -->|"Node Exporter :9100"| PROM

    LAB02 -.->|"Planned migration"| MON01