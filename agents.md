# Infrastructure Lab – Codex Agent Instructions

## Project Purpose

This repository is a **public infrastructure portfolio project**.

Its purpose is to demonstrate real hands-on experience with:

- Proxmox VE
- Linux administration
- Ansible
- Podman
- Prometheus
- Grafana
- Tailscale
- UniFi networking
- VLANs and firewalling
- VM and LXC architecture
- Infrastructure troubleshooting
- Monitoring
- Backup and recovery
- Infrastructure documentation

The target audience is primarily:

- Recruiters
- Hiring managers
- Infrastructure engineers
- System administrators
- DevOps / platform engineers

The repository should demonstrate not only which technologies are used, but also:

- why architectural decisions were made
- how systems are deployed
- how infrastructure is operated
- how failures are troubleshot
- how services are monitored
- how systems can be recovered

The repository should look like a real infrastructure engineering project rather than a collection of tutorials.

---

# Important Rule: Reflect the Real Environment

This repository represents a **sanitized public version of an actual homelab**.

Do not invent infrastructure and present it as implemented.

Clearly distinguish between:

- **Implemented**
- **In progress**
- **Planned**

If something has not actually been implemented, label it accordingly.

For example:

Terraform is planned for Proxmox provisioning but should not currently be presented as fully implemented infrastructure-as-code.

Ansible, Prometheus, Grafana, Podman and Node Exporter have been used in the actual lab and may be presented as implemented where the repository contains the corresponding configuration.

---

# Private vs Public Repository

There is a separate private operational repository named:

```text
homelab-infrastructure
```

This public repository is:

```text
infrastructure-lab
```

The public repository must never become a direct copy of the operational repository.

Instead it should contain:

- sanitized configuration
- architecture documentation
- representative automation
- generic examples based on the real implementation
- design decisions
- troubleshooting methodology

---

# Security Requirements

Never expose real sensitive infrastructure information.

Do not commit:

- passwords
- API keys
- tokens
- SSH private keys
- certificates containing private keys
- Tailscale auth keys
- application credentials
- real secrets
- backup archives
- `.env` files containing secrets
- Terraform state
- secret variable files

Avoid publishing unnecessary:

- internal IP addresses
- MAC addresses
- serial numbers
- public IP addresses
- private DNS names
- usernames tied to real infrastructure
- device identifiers

Use documentation/test networks for sanitized examples where IP addresses are required.

Preferred examples:

```text
192.0.2.0/24
198.51.100.0/24
203.0.113.0/24
```

Use placeholders such as:

```text
<api-token>
<password>
<internal-host>
```

when appropriate.

Always review files for sensitive information before adding them.

---

# Current Real Architecture

The environment is based around an always-on **Proxmox VE** host.

Important workloads include:

```text
Proxmox VE
├── Home Assistant VM
├── tailscale01 LXC
├── monitoring01 VM
└── additional Linux workloads
```

The physical/network environment includes:

```text
Internet / ISP
        |
        v
UniFi Cloud Gateway Ultra
        |
        v
UniFi Flex 2.5G
        |
        | Cat6A uplink between rooms
        v
Netgear GS724Tv4
        |
        +--> Proxmox
        +--> Infrastructure
        +--> Clients
        +--> IoT
```

Additional managed Netgear switching is available for expansion.

Routing and firewall policy should remain conceptually centralized at the UniFi gateway unless the real environment changes.

---

# Network Direction

Network work includes:

- managed switching
- VLAN design
- IoT isolation
- firewall rules
- trusted vs untrusted network separation
- remote access
- DNS/routing troubleshooting

IoT devices include consumer devices such as smart-home equipment, televisions and robotic vacuum devices.

The intended security model is generally:

```text
IoT
 |
 +----> Internet
 |
 X----> Trusted clients
 |
 X----> Infrastructure
```

Specific exceptions should be allowed where Home Assistant or another trusted service needs communication with an IoT device.

Do not claim that every planned VLAN or firewall rule is already implemented unless repository evidence supports it.

---

# Remote Access

Tailscale is used for secure remote access.

A dedicated LXC called:

```text
tailscale01
```

has been used for:

- Tailscale connectivity
- subnet routing
- exit-node functionality
- access to internal networks

The preferred architecture is to access internal administrative services through trusted networks or Tailscale rather than exposing them directly to the Internet.

---

# Administration Environment

Infrastructure administration has been performed from a Bazzite Linux workstation.

A Distrobox environment named:

```text
infra-control
```

has been used as an infrastructure administration environment.

Tools used include:

- Ansible
- SSH
- Git
- infrastructure tooling

The workstation must not be treated as an always-on production dependency.

Always-on services should reside on the always-on Proxmox environment.

---

# Existing Linux Lab

Development/testing has included Ubuntu lab systems such as:

```text
lab-ubuntu01
lab-ubuntu02
monitoring01
```

Historically, local lab VMs were accessed through forwarded SSH ports similar to:

```text
lab-ubuntu01 -> localhost:2222
lab-ubuntu02 -> localhost:2223
```

The public repository should use sanitized identities and addresses where needed.

---

# Monitoring Implementation

The monitoring project uses:

- Prometheus
- Grafana
- Prometheus Node Exporter
- Podman
- Ansible

A monitoring stack was implemented using rootless Podman.

Prometheus collects metrics from Linux systems running Node Exporter.

Conceptually:

```text
Linux Server
     |
     | :9100
     v
Node Exporter
     |
     v
Prometheus
     |
     v
Grafana
```

Monitoring was initially implemented in a lab environment and the architecture is moving toward an always-on dedicated VM:

```text
monitoring01
```

Do not incorrectly state that every monitoring component has already been completely migrated unless the repository or current user instruction confirms it.

---

# Monitoring Philosophy

Monitoring should answer operational questions.

Examples:

- Is the host available?
- Is a service running?
- Is storage approaching capacity?
- Is memory pressure increasing?
- Has resource usage changed?
- Is remote access operational?

Avoid building dashboards simply to maximize the number of displayed metrics.

Prefer actionable dashboards.

---

# Backup and Recovery

Backup and recovery are important parts of this project.

Monitoring-related work has included concepts such as:

- backing up persistent monitoring data
- checksum verification
- retention
- restore testing
- scheduled backup execution

The portfolio should show that backup is not considered complete until restoration has been tested.

A future document should be created at:

```text
docs/backup-recovery.md
```

It should describe:

- what is backed up
- what is intentionally not backed up
- retention
- validation
- restoration
- recovery priorities

Do not invent backup architecture that has not been implemented.

---

# Ansible

Ansible is currently the strongest real automation component of this repository.

The public repository should demonstrate real configuration-management concepts including:

```text
ansible/
├── inventory/
├── group_vars/
├── host_vars/
├── playbooks/
└── roles/
```

Relevant automation includes:

- Linux server management
- monitoring deployment
- Node Exporter deployment
- host-specific configuration
- shared configuration
- service management

Configuration should be idempotent where possible.

---

# Current Ansible Structure

The repository currently contains approximately:

```text
ansible/
├── inventory/
│   └── hosts.example.yml
│
├── playbooks/
│   ├── monitoring.yml
│   └── node-exporter.yml
│
└── roles/
    └── monitoring-stack/
        ├── defaults/
        │   └── main.yml
        └── node_exporter/
            └── tasks/
                └── main.yml
```

This structure needs cleanup.

The intended structure is:

```text
ansible/
├── inventory/
│   └── hosts.example.yml
│
├── playbooks/
│   ├── monitoring.yml
│   └── node-exporter.yml
│
└── roles/
    ├── monitoring_stack/
    │   ├── defaults/
    │   │   └── main.yml
    │   ├── tasks/
    │   │   └── main.yml
    │   └── templates/
    │       └── prometheus.yml.j2
    │
    └── node_exporter/
        └── tasks/
            └── main.yml
```

Fix these issues before expanding the Ansible implementation:

1. Rename:

```text
monitoring-stack
```

to:

```text
monitoring_stack
```

because the playbook currently references:

```yaml
roles:
  - monitoring_stack
```

2. Move:

```text
monitoring-stack/node_exporter/
```

to:

```text
roles/node_exporter/
```

because `node-exporter.yml` expects a standalone role:

```yaml
roles:
  - node_exporter
```

3. Build a real `monitoring_stack` role based on the existing Podman/Prometheus/Grafana implementation.

---

# Monitoring Ansible Role

The monitoring role should represent the actual approach used in the lab.

Expected responsibilities may include:

```text
monitoring_stack
├── Install Podman where required
├── Create monitoring directories
├── Create Prometheus configuration
├── Create persistent data locations
├── Deploy Prometheus container
├── Deploy Grafana container
├── Configure restart/start behavior
└── Validate services
```

Prefer Ansible modules over arbitrary shell commands when a suitable module exists.

Tasks should have meaningful names.

Example:

```yaml
- name: Create monitoring data directory
  ansible.builtin.file:
    path: "{{ monitoring_data_dir }}"
    state: directory
```

Avoid unnecessary abstraction.

---

# Node Exporter Role

The existing implementation is based around installing:

```text
prometheus-node-exporter
```

on Ubuntu systems.

Expected responsibilities include:

- install package
- enable service
- start service
- allow Prometheus connectivity if host firewall rules require it

The public version currently uses sanitized networking examples.

Do not replace working simple package deployment with unnecessary container complexity.

---

# Prometheus Configuration

Prometheus target configuration should be generated from variables where practical.

Example variable:

```yaml
prometheus_node_targets:
  - "192.0.2.11:9100"
  - "192.0.2.12:9100"
```

A template such as:

```text
ansible/roles/monitoring_stack/templates/prometheus.yml.j2
```

should turn this into valid Prometheus configuration.

Prefer readable Jinja.

---

# Podman

The real monitoring lab uses Podman.

Do not replace Podman with Docker simply because Docker examples are more common.

One goal of this project is to accurately demonstrate technologies that were actually used.

Where practical, preserve the rootless-container design.

---

# Terraform

Terraform is part of the intended architecture but is not currently the strongest implemented part of the lab.

Terraform is intended to eventually manage infrastructure provisioning such as:

- Proxmox VMs
- LXC containers
- CPU allocation
- memory
- storage
- virtual networking

Ansible should then configure the operating system and services.

Conceptually:

```text
Terraform
    |
    v
Provision infrastructure
    |
    v
Ansible
    |
    v
Configure operating system and services
```

Until real Terraform configuration has been implemented, label Terraform as:

```text
Planned
```

or:

```text
In progress
```

Do not create large fake Terraform examples just to make the repository look more complete.

---

# Documentation

Current documentation structure:

```text
docs/
├── architecture.md
├── networking.md
├── automation.md
├── monitoring.md
└── security.md
```

There is also:

```text
diagrams/
└── architecture.md
```

The documentation should be useful but not repetitive.

README should remain the high-level entry point.

Detailed reasoning belongs under `docs/`.

---

# Current Documentation Issues

At the time these instructions were written:

```text
docs/architecture.md
```

is effectively empty.

```text
docs/monitoring.md
```

is empty.

These should be completed.

Existing documentation such as:

```text
networking.md
security.md
automation.md
```

already contains substantial material.

When updating them, reduce duplication rather than continuously adding more prose.

---

# README Goals

The main `README.md` should allow someone to understand the project in approximately one minute.

It should contain:

1. What the lab is
2. Key technologies
3. Current architecture
4. What is actually implemented
5. Links to deeper documentation
6. Links to representative automation
7. Project status

Avoid making README excessively long.

Detailed explanations should link into `docs/`.

---

# Architecture Diagram

An architecture diagram exists in:

```text
diagrams/architecture.md
```

It uses Mermaid.

The diagram should clearly distinguish between:

- physical/network infrastructure
- always-on Proxmox infrastructure
- administration/development systems
- implemented monitoring
- planned migrations

Do not present planned architecture as already completed.

---

# Portfolio Quality Standard

Every addition should answer at least one of these questions:

```text
What did I build?
Why did I build it this way?
How did I automate it?
How is it secured?
How is it monitored?
How is it recovered?
How would I troubleshoot it?
```

Avoid adding files that merely increase repository size.

Prefer a smaller amount of strong material over many superficial examples.

---

# Code Quality

Infrastructure code should be:

- readable
- consistently formatted
- idempotent where practical
- logically structured
- documented where intent is not obvious
- easy to troubleshoot

Use descriptive names.

Bad:

```yaml
- name: Do stuff
```

Preferred:

```yaml
- name: Ensure Prometheus configuration directory exists
```

---

# Validation

When modifying Ansible:

Use appropriate validation where possible.

Examples:

```bash
ansible-inventory -i ansible/inventory/hosts.example.yml --graph
```

```bash
ansible-playbook \
  -i ansible/inventory/hosts.example.yml \
  ansible/playbooks/node-exporter.yml \
  --syntax-check
```

Use `ansible-lint` if available.

For YAML:

```bash
yamllint .
```

Do not claim configuration has been successfully deployed unless it has actually been tested against the real lab.

Syntax validation and real deployment validation are different things.

---

# Git Practices

Use meaningful commits.

Good examples:

```text
Fix Ansible monitoring role structure
```

```text
Add Prometheus configuration template
```

```text
Document monitoring architecture
```

```text
Add sanitized monitoring backup workflow
```

Avoid meaningless commit messages such as:

```text
update
fix
stuff
changes
```

---

# Recommended Next Tasks

Work in approximately this order.

## 1. Fix Ansible role structure

Convert:

```text
monitoring-stack
```

to:

```text
monitoring_stack
```

and make `node_exporter` a separate role.

Verify playbook role references.

---

## 2. Complete monitoring_stack

Implement the sanitized equivalent of the real:

- Podman
- Prometheus
- Grafana

deployment.

Add:

```text
roles/monitoring_stack/tasks/main.yml
```

and:

```text
roles/monitoring_stack/templates/prometheus.yml.j2
```

---

## 3. Validate Node Exporter Role

Ensure:

```text
playbooks/node-exporter.yml
```

correctly runs:

```text
roles/node_exporter
```

Validate YAML and Ansible syntax.

---

## 4. Complete Monitoring Documentation

Populate:

```text
docs/monitoring.md
```

with the real monitoring architecture.

Clearly distinguish the lab implementation from the planned/ongoing move to the always-on `monitoring01` server.

---

## 5. Complete Architecture Documentation

Populate:

```text
docs/architecture.md
```

with:

- physical architecture
- Proxmox architecture
- network responsibilities
- workload placement
- management model
- automation model

Avoid duplicating the full contents of networking/security documentation.

---

## 6. Add Backup and Recovery

Create:

```text
docs/backup-recovery.md
```

Document the actual monitoring backup and restore work.

Where suitable, add sanitized automation examples.

Focus on:

- backup
- integrity verification
- retention
- restore testing
- recovery process

---

## 7. Improve README Navigation

Add a clear section similar to:

```markdown
## Documentation

- [Architecture](docs/architecture.md)
- [Networking](docs/networking.md)
- [Security](docs/security.md)
- [Automation](docs/automation.md)
- [Monitoring](docs/monitoring.md)
- [Backup & Recovery](docs/backup-recovery.md)
```

Also link to the most representative Ansible code.

---

## 8. Terraform Later

Only introduce Terraform code after a real Terraform implementation exists or is being actively built.

When that happens, add:

```text
terraform/
```

and document its relationship with Ansible.

---

# Things Not to Do

Do not:

- expose secrets
- expose unnecessary real network addressing
- fabricate implemented infrastructure
- create fake enterprise complexity
- introduce Kubernetes without a real requirement
- introduce multiple monitoring stacks for portfolio appearance
- replace existing technologies merely because another tool is more fashionable
- create huge abstractions before they are needed
- claim Terraform is complete before implementation
- turn the repository into a generic tutorial

---

# Overall Objective

The final repository should tell a clear story:

```text
I designed a homelab
        |
        v
I built the network and virtualization platform
        |
        v
I deployed Linux infrastructure
        |
        v
I automated configuration with Ansible
        |
        v
I deployed monitoring with Prometheus and Grafana
        |
        v
I implemented secure remote access
        |
        v
I segmented untrusted devices
        |
        v
I added monitoring, backup and recovery
        |
        v
I documented how the entire environment works
```

The project should demonstrate practical infrastructure engineering capability, including architecture, implementation, operations and troubleshooting.

Optimize for **credibility, clarity and real technical depth**, not for the largest possible number of technologies.