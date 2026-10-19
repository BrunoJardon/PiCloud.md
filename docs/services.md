# Services & Container Stacks

PiCloud relies on Docker for service isolation. Each application runs its own isolated stack (database, cache, workers). No service shares state or databases with another.

## User-Facing Services
| Service | Purpose | Authentication |
|---------|---------|----------------|
| **Immich** | Photo management and backup | OIDC via `tsidp` |
| **Outline** | Collaborative knowledge base | OIDC via `tsidp` |
| **TaskView** | Task and project management | OIDC via `tsidp` |
| **SMB (Samba)** | File sharing (LAN & tailnet) | Local OS accounts |

## Infrastructure Services
| Service | Purpose |
|---------|---------|
| **OpenMediaVault** | NAS and system administration |
| **Portainer** | Docker stack and container management |
| **Tailscale** | Remote access and network routing |
| **`tsidp`** | OAuth Identity Provider (SSO) |
| **Beszel** | Real-time system & Docker containers monitoring (CPU, RAM, I/O, network) |

## Volume & Data Management
 - Each stack maintains its own Docker volumes to ensure data isolation.
 - Portainer manages the Compose files and configurations for all these stacks from a centralized interface.Q