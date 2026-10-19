# PiCloud Architecture Overview

PiCloud is a self-hosted personal cloud running on a Raspberry Pi 4. It is designed to run continuously with minimal maintenance, relying on Docker for service isolation and Tailscale for secure, zero-trust remote access.

This document provides a high-level overview. For specific details, refer to the sub-pages:

- **[Hardware & Storage](hardware.md):** Physical specs, disk layout, MergerFS, and power management.
- **[Networking & Remote Access](networking.md):** Tailscale `serve`, ACLs, port mapping, and domains.
- **[Security Model](security.md):** Firewall rules (default deny), Docker isolation, and known risks.
- **[Services](services.md):** Stack breakdown (Immich, Outline, TaskView, Beszel, etc.).

## Core Principles
- **Zero Public Exposure:** No port forwarding on the router. All access is via Tailscale VPN.
- **Identity-Centric Security:** Single Sign-On (SSO) managed via `tsidp` entirely within the tailnet.
- **Service Isolation:** Each application runs its own stack (database, cache, workers) with no shared state.
- **Simplicity:** Debian 13 base, managed via OpenMediaVault and Portainer.