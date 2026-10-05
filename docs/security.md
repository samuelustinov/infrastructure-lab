# Security Architecture

This document describes the security principles used in my infrastructure lab.

The security model focuses on practical risk reduction, network isolation, controlled administration and recoverability without introducing unnecessary enterprise complexity.

> This is a sanitized public representation of the environment. Credentials, internal addresses, device identifiers and sensitive security configuration are intentionally excluded.

---

## Security Goals

The environment is designed around the following principles:

- Minimize publicly exposed services
- Separate systems based on trust level
- Restrict unnecessary lateral movement
- Use secure remote-access mechanisms
- Protect administrative interfaces
- Keep credentials outside source control
- Maintain clear infrastructure ownership and roles
- Make configuration recoverable
- Apply changes incrementally and verify them

Security is treated as an architectural concern rather than a single product or firewall rule.

---

# Security Layers

The environment uses multiple security layers.

```text
Internet Edge
     ↓
Gateway / Firewall
     ↓
Network Segmentation
     ↓
Host Security
     ↓
Application Security
     ↓
Identity / Authentication
     ↓
Monitoring
     ↓
Backup / Recovery
```

No single layer is treated as sufficient on its own.

---

# Internet Edge

The UniFi gateway provides the primary security boundary between the home network and the Internet.

Responsibilities include:

- Stateful firewalling
- NAT
- Inter-network policy
- Routing
- VPN-related connectivity
- Traffic inspection and visibility

The general principle is to avoid exposing internal administrative services directly to the Internet.

Services such as:

- Proxmox
- Grafana
- SSH
- Home Assistant administration
- switch management interfaces

should normally remain internal.

---

# Remote Access

Remote administration is provided through Tailscale rather than exposing management ports publicly.

Conceptually:

```text
Remote Administrator
        |
        v
   Tailscale
        |
        v
  tailscale01
        |
        v
Selected Internal Services
```

This provides an authenticated private network path into the lab.

The security benefits include:

- no public SSH exposure
- no public Proxmox management exposure
- no public Grafana exposure
- reduced dependency on inbound port forwarding
- centrally controllable remote access

Remote access should still follow least-privilege principles.

VPN connectivity does not automatically imply unrestricted access to every internal system.

---

# Network Segmentation

Devices are separated according to their role and trust level.

The intended architecture includes categories such as:

```text
Management
Infrastructure
Trusted Clients
IoT
Guest / Untrusted
```

These segments do not automatically trust one another.

Firewall rules determine which communication paths are allowed.

---

# IoT Security

IoT devices are treated as lower-trust devices.

Examples include:

- televisions
- robotic vacuum cleaners
- consumer smart-home devices

The intended default policy is:

```text
IoT
 |
 +----> Internet
 |
 X----> Trusted Clients
 |
 X----> Management
 |
 X----> Infrastructure
```

Where Home Assistant or another service requires access to an IoT device, explicit exceptions are preferred over broad unrestricted communication.

Example:

```text
Home Assistant
      |
      | required ports only
      v
IoT Device
```

The goal is to reduce unnecessary lateral access from devices that may have limited security controls or cloud dependencies.

---

# Administrative Access

Administrative interfaces should be reachable only from trusted paths.

Examples include:

- Proxmox web interface
- SSH
- switch administration
- UniFi administration
- Grafana administration
- Linux management services

The desired model is:

```text
Trusted Admin Device
       |
       v
Management / Infrastructure
```

rather than:

```text
Any Device
    |
    v
All Management Interfaces
```

As network segmentation develops, management traffic can be restricted further.

---

# Proxmox Security

The Proxmox host is a high-value infrastructure component because compromise of the hypervisor could affect multiple workloads.

Security considerations therefore include:

- limiting access to the management interface
- using dedicated administrative credentials
- applying updates intentionally
- minimizing unnecessary services
- protecting backup access
- controlling SSH access
- monitoring storage capacity and host health
- avoiding unnecessary Internet exposure

Virtual workloads are not considered a substitute for protecting the hypervisor itself.

---

# VM and Container Isolation

Different workloads use different virtualization models.

Virtual machines provide stronger isolation because each VM operates with its own kernel.

LXC containers provide lower overhead but share the host kernel.

This affects workload placement.

Conceptually:

```text
Higher isolation requirement
        |
        v
       VM

Lightweight trusted service
        |
        v
       LXC
```

LXC is used where its efficiency provides a clear benefit and its isolation model is appropriate.

---

# Linux Server Security

Linux servers should follow a consistent baseline.

Areas include:

- regular package updates
- SSH key authentication where practical
- restricted administrative access
- minimal installed software
- disabling unnecessary services
- clear service ownership
- appropriate firewalling
- monitoring
- predictable configuration through Ansible

Automation can help reduce configuration drift between servers.

---

# SSH

SSH is an important administrative protocol within the environment.

Preferred practices include:

- public-key authentication
- avoiding unnecessary public exposure
- limiting administrative users
- using VPN-based remote access
- keeping private keys outside repositories
- protecting privileged accounts

Root access should be controlled and used intentionally rather than becoming the default administrative workflow.

---

# Secrets Management

Secrets must never be intentionally stored in public Git repositories.

Examples include:

- passwords
- private SSH keys
- API tokens
- Tailscale authentication keys
- application secrets
- database passwords
- certificates containing private keys
- cloud credentials

Public examples should instead use placeholders:

```text
API_TOKEN=<your-token>
DB_PASSWORD=<your-password>
```

or environment variables.

---

# Public Repository Sanitization

The public `infrastructure-lab` repository is intentionally separated from the private operational infrastructure repository.

The public repository may contain:

- architecture
- sanitized examples
- generic configuration
- documentation
- reusable patterns

It should not contain:

- credentials
- production secrets
- sensitive internal hostnames
- unnecessary private addressing
- personal identifiers
- complete operational backups

This separation allows infrastructure work to be demonstrated publicly without turning the portfolio into an information disclosure risk.

---

# Git Security

Before configuration is published, it should be reviewed for secrets and sensitive values.

A secret should be considered compromised if it has already been committed to repository history.

Simply deleting it from the latest version does not necessarily remove it from Git history.

The preferred response to an exposed credential is:

```text
Revoke / rotate credential
        ↓
Remove from repository
        ↓
Clean history where required
        ↓
Prevent recurrence
```

Rotation is more important than merely deleting the string.

---

# Firewall Philosophy

Firewall rules follow a simple principle:

```text
Allow what is required
Deny what is unnecessary
```

Rules should be specific enough to provide a useful security boundary without becoming impossible to maintain.

Preferred:

```text
Home Assistant
    ->
Specific IoT subnet / required ports
```

Instead of:

```text
Trusted network
    ->
Allow everything everywhere
```

Policy changes should be introduced gradually and tested after each change.

---

# Change Safety

Security changes can accidentally cause service outages.

Changes should therefore be made in a controlled sequence.

Example:

```text
1. Document current connectivity
2. Identify required communication
3. Add new rule
4. Validate required traffic
5. Validate blocked traffic
6. Monitor for unexpected failures
7. Continue to next restriction
```

This is particularly important when modifying:

- VLANs
- firewall rules
- routing
- DNS
- remote access
- hypervisor networking

---

# Monitoring and Security

Monitoring supports security by making unexpected behavior easier to identify.

Relevant areas include:

- service availability
- authentication failures
- unexpected resource usage
- storage exhaustion
- host outages
- failed backups
- infrastructure changes

Grafana and Prometheus are primarily used for observability rather than acting as a security information and event management platform.

A heavier SIEM platform would only be introduced if the operational and learning benefits justified the additional complexity.

---

# Backup as a Security Control

Security includes the ability to recover from:

- configuration mistakes
- hardware failure
- software failure
- corrupted data
- compromised systems
- accidental deletion

Backups therefore form part of the security architecture.

A backup should not be considered complete until restoration has been tested.

The long-term objective is to document:

```text
What is backed up?
Where is it stored?
How often?
How long is it retained?
How is it restored?
Has restoration been tested?
```

---

# Home Assistant Security

Home Assistant is treated as a sensitive internal service because it controls physical devices in the home.

Security considerations include:

- keeping the administrative interface private
- restricting unnecessary network access
- protecting credentials
- backing up configuration
- controlling integrations
- limiting IoT network access where practical

Its VM-based deployment also creates a clear backup and recovery boundary.

---

# Monitoring Service Security

Monitoring systems often have visibility across large parts of the infrastructure.

They can therefore contain valuable operational information.

Grafana and Prometheus should remain internal unless there is a clear reason to expose them.

Preferred access:

```text
Trusted LAN
or
Tailscale
```

rather than direct public Internet exposure.

---

# Principle of Least Privilege

Access should be limited to what is required.

This applies to:

- network traffic
- user accounts
- SSH permissions
- application credentials
- API access
- automation accounts
- infrastructure administration

Least privilege is treated as a direction rather than an excuse to create excessive administrative complexity.

Controls should remain maintainable.

---

# Availability vs Security

Security controls must also account for availability.

For example, an extremely restrictive firewall policy that regularly breaks core services can create operational risk.

The objective is therefore to balance:

- confidentiality
- integrity
- availability
- maintainability

For a homelab, maintainability is especially important because a single person is responsible for operating the environment.

---

# Current Security Priorities

Current areas of improvement include:

- increasing network segmentation
- improving IoT isolation
- reducing unnecessary inter-VLAN access
- restricting infrastructure management paths
- improving backup and restore procedures
- expanding monitoring and alerting
- documenting security-sensitive configuration
- keeping public infrastructure examples sanitized

---

# Future Security Improvements

Potential future improvements include:

### Dedicated Management Network

Moving management interfaces into a dedicated network where the additional isolation provides enough benefit.

### Centralized Authentication

Introducing centralized identity where it provides operational value without creating unnecessary dependency.

### Centralized Logging

Aggregating logs if troubleshooting and security visibility requirements justify it.

### Improved Backup Isolation

Ensuring backup systems are sufficiently separated from the systems they protect.

### Automated Configuration Validation

Using infrastructure-as-code and configuration management to reduce drift and improve repeatability.

---

# Security Philosophy

The goal is not to reproduce every enterprise security technology inside a home environment.

The objective is to understand and apply the underlying principles:

- minimize exposure
- separate trust zones
- control access
- automate where useful
- monitor important systems
- maintain recoverability
- document decisions

The result should be a lab that is more secure because it is understandable and maintainable, not simply because it contains more security products.