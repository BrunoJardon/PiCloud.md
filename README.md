# PiCloud

> A self-hosted personal cloud built on a Raspberry Pi 4, providing private file storage and photo management for family and friends.

PiCloud is designed as a private alternative to Google Drive and Google Photos while maintaining complete ownership of data, low power consumption, and a modular architecture that is easy to maintain and expand.

---

# Table of Contents

- [Overview](#overview)
- [Hardware](#hardware)
- [Software Stack](#software-stack)
- [Infrastructure](#infrastructure)
- [Storage Architecture](#storage-architecture)
- [Filesystem Layout](#filesystem-layout)
- [Immich Deployment](#immich-deployment)
- [Network Architecture](#network-architecture)
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
- Secure remote access using Tailscale
- Centralized administration with OpenMediaVault
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
| OpenMediaVault | NAS administration |
| Docker Engine | Container runtime |
| Immich | Photo management |
| PostgreSQL | Immich database |
| Valkey | Cache & queues |
| Samba | File sharing |
| Tailscale | Remote access |
| MergerFS | Storage abstraction |
| Monit | Service monitoring |
| SMART | Disk health monitoring |

---

# Infrastructure

```text
              Family & Friends
                     │
          ┌──────────┴──────────┐
          │                     │
        SMB                 Immich
          │                     │
          └──────────┬──────────┘
                     │
                Raspberry Pi 4
                     │
         ┌───────────┴───────────┐
         │                       │
 OpenMediaVault             Docker Engine
         │                       │
         │                 Immich Stack
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
 Docker AppData           Compose Files


                 HDD (1 TB)
                    │
                  ext4
                    │
                 MergerFS
                    │
           /srv/mergerfs/storage
            ┌───────┴────────┐
            │                │
          drive/          immich/
```

## User Storage

```
drive/
├── user1/
├── user2/
├── user3/
└── ...
```

Each user has an independent home directory exported through SMB.

## Immich Storage

```
immich/
├── backups/
├── encoded-video/
├── library/
├── profile/
├── thumbs/
└── upload/
```

The HDD is configured with:

- Advanced Power Management: **127**
- Spindown timeout: **20 minutes**

This reduces unnecessary wear while avoiding excessive spin-up/spin-down cycles.

---

# Filesystem Layout

```
/
├── docker/
│   ├── appdata/
│   ├── compose/
│   └── storage/
│
├── srv/
│   └── mergerfs/
│       └── storage/
│
└── run/
    └── OverlayFS write cache
```

## Docker

```
/docker/
├── appdata/
│   └── immich/
│
├── compose/
│   └── immich
│
└── storage/
```

| Directory | Purpose |
|----------|---------|
| appdata | Persistent application data |
| compose | Docker Compose projects |
| storage | Docker Engine data |

---

# Immich Deployment

Immich containers are executed using a dedicated **docker** user.

Current stack:

```
immich_server
immich_machine_learning
immich_postgres
immich_redis
```

Characteristics:

- Hardware transcoding via VAAPI
- PostgreSQL stored on NVMe
- User assets stored on HDD
- Machine learning cache stored in Docker volumes

---

# Network Architecture

## Local Network

Available services:

- SMB
- Immich
- ICMP

Administrative services remain restricted.

## Remote Access

```
 Remote User
      │
  Tailscale
      │
   Firewall
      │
 ┌────┴────┐
 │         │
SMB     Immich
```

SSH and OpenMediaVault are intentionally inaccessible through Tailscale.

No service is exposed directly to the public Internet.

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

## Administrative Access

Restricted to a trusted workstation.

| Service | Port |
|---------|------|
| SSH | 22 |
| HTTP | 80 |
| HTTPS | 443 |

Security principles:

- Dedicated Docker user
- Tailscale ACLs
- Firewall segmentation
- Service isolation
- No public port forwarding

---

# Monitoring

PiCloud continuously monitors system health using:

- SMART disk monitoring
- Monit
- Docker health checks
- OpenMediaVault email notifications

Notifications include:

- SMART warnings
- Service failures
- Storage events
- System alerts

---

# Reliability

Current reliability measures:

- Operating system isolated on NVMe
- User data isolated on HDD
- Docker AppData stored separately
- OverlayFS write cache for system directories
- SMART monitoring
- Email alerts
- HDD power management
- Container isolation

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

## Why Immich?

- Multi-user
- Active development
- Excellent mobile apps
- Google Photos replacement

---

# Known Limitations

Current limitations include:

- Single HDD (no redundancy)
- No automated backups
- No UPS
- Disaster recovery procedures not yet implemented

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

- Prometheus
- Grafana
- Historical metrics

Power

- UPS
- Automatic safe shutdown

---

# Philosophy

PiCloud is built around a simple principle:

> Personal cloud infrastructure should be private, understandable, maintainable and fully controlled by its owner.

Rather than maximizing features, the project prioritizes simplicity, reliability, long-term maintainability and complete ownership of personal data.
