# PiCloud

> A self-hosted personal cloud built on a Raspberry Pi 4, providing private file storage, photo management, and collaborative tools for family and friends.

PiCloud is a private alternative to Google Drive, Google Photos, and Notion, maintaining complete ownership of data, low power consumption, and a modular architecture that is easy to maintain and expand.

---

# Table of Contents

- [Overview](#overview)
- [Hardware](#hardware)
- [Software Stack](#software-stack)
- [Infrastructure](#infrastructure)
- [Storage Architecture](#storage-architecture)
- [Filesystem Layout](#filesystem-layout)
- [Services](#services)
- [Network Architecture](#network-architecture)
- [Port Management](#port-management)
- [Security Model](#security-model)
- [Monitoring](#monitoring)
- [Reliability](#reliability)
- [Design Decisions](#design-decisions)
- [Known Limitations](#known-limitations)
- [Future Roadmap](#future-roadmap)
- [Philosophy](#philosophy)

---

# Overview

PiCloud provides:

- Personal cloud storage through SMB/Samba
- Self-hosted photo management with Immich
- Collaborative knowledge base with Outline
- Task management with Taskview
- Centralized administration with OpenMediaVault and Portainer
- Secure remote access using Tailscale
- Integrated identity management with Tailscale IDP
- Containerized services using Docker

The system is designed to run continuously with minimal maintenance while remaining simple to understand and expand.

---

# Hardware

| Component | Specification |
|-----------|---------------|
| Platform | Raspberry Pi 4 Model B Rev 1.5 |
| CPU | Broadcom BCM2711 (4×1.8 GHz) |
| RAM | 4 GB LPDDR4 |
| System Storage | 128 GB NVMe SSD |
| Data Storage | 1 TB External HDD |
| Network | Gigabit Ethernet |
| Operating System | Debian GNU/Linux 13 (Trixie) |

---

# Software Stack

| Component | Purpose |
|----------|---------|
| Debian 13 | Base operating system |
| OpenMediaVault | NAS and system administration |
| Portainer | Docker service management |
| Docker Engine | Container runtime |
| Immich | Photo management |
| Outline | Collaborative knowledge base |
| Taskview | Task management |
| PostgreSQL | Immich database |
| Valkey | Cache & queues |
| Samba | File sharing |
| Tailscale | Remote access & identity management |
| Tailscale IDP | OAuth identity provider |
| MergerFS | Storage abstraction |
| SMART | Disk health monitoring |

---

# Infrastructure

```text
              Family & Friends
                     │
          ┌──────────┴──────────┐
          │                     │
        SMB                 Web Services
          │                     │
          │          ┌──────────┴──────────┐
          │          │          |          │
          │      Immich     TaskView    Outline
          │          │          |          │
          └──────────┴─────────────────────┘
                     │
                Raspberry Pi 4
                     │
         ┌───────────┴───────────┐
         │                       │
 OpenMediaVault             Portainer
         │                       │
    System Admin           Docker Management
         │                       │
         └───────────┬───────────┘
                     │
               Docker Engine
                     │
               Immich Stack
              Outline Stack
              Taskview Stack
                     │
                     ▼
                 Storage
```

---

# Storage Architecture

PiCloud separates the operating system from user data.

```text
                 NVMe SSD
                 (128 GB)
                    │
        ┌───────────┴────────────┐
        │                        │
 Debian + OMV             Docker Engine
                    Docker Volumes & Data

                 HDD (1 TB)
                    │
                  ext4
                    │
                 MergerFS
                    │
                 storage
      ┌───────┬─────┴───┬────────┐
      │       |         │        |
    drive/ Outline/ TaskView/ Immich/
```

The HDD is configured with:

- Advanced Power Management: **127**
- Spindown timeout: **20 minutes**

This reduces unnecessary wear while avoiding excessive spin-up/spin-down cycles.

---

## Docker Storage

Docker maintains its default storage locations:

| Directory | Purpose |
|----------|---------|
| `/var/lib/docker` | Docker Engine data, volumes, and container storage |
| `/etc/docker` | Docker daemon configuration |

Portainer manages all Docker Compose files and configurations through its appdata volume, providing centralized management of containerized services.

---

# Services

PiCloud hosts the following user-accessible services:

| Service | Purpose | Access |
|---------|---------|--------|
| Immich | Photo management and backup | All users |
| Outline | Collaborative knowledge base | All users |
| Taskview | Task and project management | All users |

## Service Management

All Docker services are managed through Portainer, providing:

- Container health monitoring
- Resource usage tracking
- Log aggregation
- Service orchestration
- Easy updates and rollbacks

---

# Network Architecture

## Local Network

Available services:

- SMB
- Immich
- Outline
- Taskview
- ICMP

Administrative services remain restricted to trusted workstations.

## Remote Access with Tailscale

```
      Remote User
            │
            │
   Firewall / Tailscale ACLs
            │
       ┌────┴────┐
       │         │
      SMB     Web Services
                 (HTTPS)
```

### Tailscale Features

- **Secure Private Network**: All communication encrypted by default
- **Services** : Use subdomains for different services on the same domain
- **Tailscale IDP**: Built-in OAuth identity provider for single sign-on
- **HTTPS Certificates**: Automatic certificate management
- **No Port Forwarding**: No exposure to public internet

### Service URLs

Services are accessible via Tailscale's `--serve` feature with subdomains:

```
https://photos.tailnet-name.ts.net
https://notes.tailnet-name.ts.net
https://todo.tailnet-name.ts.net
```

## Administrative Access

Restricted to a trusted workstation.

| Service | Port |
|---------|------|
| SSH | 22 |
| OpenMediaVault | 8000 (HTTP) / 9000 (HTTPS) |
| Portainer | 8001 (HTTP) / 9001 (HTTPS) |

---

# Port Management

To prevent port conflicts and maintain organization, services are assigned to specific port ranges:

| Type | HTTP Port Range | HTTPS/Secure Range | Services |
|------|----------------|-------------------|----------|
| Administration | 8000 - 8099 | 9000 - 9099 | OpenMediaVault, Portainer |
| Monitoring | 8100 - 8199 | 9100 - 9199 | Grafana, Prometheus, Beszel |
| Infrastructure | 8200 - 8299 | 9200 - 9299 | Tailscale IDP |
| App Services | 8300 - 8399 | 9300 - 9399 | APIs, MCPs, Databases |
| User Applications | 8400 - 8999 | 9400 - 9999 | Immich, Outline, Taskview |

This structured approach ensures:
- Clear separation of service categories
- No port conflicts between services
- Easier firewall rule management
- Simplified service discovery

---

# Security Model

The firewall follows a **default deny** policy.

## User Access

| Service | Port |
|---------|------|
| SMB | TCP 139 |
| SMB | TCP 445 |
| NetBIOS | UDP 137-138 |
| Immich | TCP 2283 |
| Outline | TCP 3000 |
| Taskview | TCP 8080 |

## Identity Management with Tailscale IDP

PiCloud implements single sign-on using Tailscale IDP:

- **OAuth Integration**: All user-facing services authenticate via Tailscale IDP
- **Automatic Authentication**: Users logged into Tailscale are automatically authenticated
- **Centralized User Management**: User identities managed through Tailscale
- **Seamless Experience**: No additional login required for trusted Tailscale users

## Administrative Access

Restricted to a trusted workstation.

| Service | Port |
|---------|------|
| SSH | 22 |
| OpenMediaVault | 8000 (HTTP) / 9000 (HTTPS) |
| Portainer | 8001 (HTTP) / 9001 (HTTPS) |

Security principles:

- Tailscale ACLs
- Firewall segmentation
- Service isolation
- No public port forwarding
- OAuth single sign-on
- Automatic HTTPS certificates
- Default deny policy

---

# Monitoring

PiCloud monitors system health through multiple layers:

- SMART disk monitoring
- Portainer health checks for all containers
- OpenMediaVault email notifications
- Docker health checks

Notifications include:

- SMART warnings
- Service failures
- Storage events
- System alerts
- Container health status changes

---

# Reliability

Current reliability measures:

- Operating system isolated on NVMe
- User data isolated on HDD
- Docker data managed in standard locations
- OverlayFS write cache for system directories
- SMART monitoring
- Email alerts
- HDD power management
- Container isolation
- Service health monitoring via Portainer

---

# Design Decisions

## Why Raspberry Pi?

- Silent
- Low power consumption
- Reliable
- Affordable

## Why OpenMediaVault?

- Debian ecosystem
- Excellent NAS tooling
- Easy administration

## Why Portainer?

- Centralized Docker management
- Easy service orchestration
- Resource monitoring
- Simplified troubleshooting

## Why Docker?

- Service isolation
- Easy upgrades
- Simple recovery

## Why MergerFS?

Storage can grow by simply adding additional disks without changing user paths.

## Why Tailscale?

- No port forwarding
- Encrypted connections
- Simple onboarding
- Reduced attack surface
- Built-in identity management
- Automatic HTTPS certificates

## Why Tailscale IDP?

- Native integration with Tailscale network
- No additional infrastructure required
- Automatic user authentication
- Single sign-on across all services
- Simplified user management

## Why Immich?

- Multi-user
- Active development
- Excellent mobile apps
- Google Photos replacement

## Why Outline?

- Modern collaborative knowledge base
- Excellent for team documentation
- Markdown support
- Active development

## Why Taskview?

- Lightweight task management
- Kanban-style organization
- Simple user interface

---

# Known Limitations

Current limitations include:

- Single HDD (no redundancy)
- No automated backups
- No UPS
- Disaster recovery procedures not yet implemented
- Limited resource capacity for additional services

---

# Future Roadmap

Storage

- Additional HDDs
- SnapRAID parity
- Storage expansion

Operations

- Automated backups
- Backup verification
- Disaster recovery documentation

Monitoring

- Beszel
- Historical metrics

Power

- UPS
- Automatic safe shutdown

Services

- Additional collaborative tools

---

# Philosophy

PiCloud is built around a simple principle:

> Personal cloud infrastructure should be private, understandable, maintainable and fully controlled by its owner.

Rather than maximizing features, the project prioritizes simplicity, reliability, long-term maintainability and complete ownership of personal data. With the addition of Tailscale IDP, the system demonstrates how modern zero-trust principles can be applied to personal infrastructure, making secure access seamless while maintaining complete ownership and control.