# Networking & Remote Access

## Tailscale `serve` (Zero Public Exposure)
PiCloud does not use port forwarding. All external access is handled by Tailscale `serve`, which proxies local container ports to `*.<tailnet>.ts.net` domains over HTTPS (port 443) exclusively within the tailnet.

| Service | Tailnet URL | Local Proxy Target |
|---------|-------------|--------------------|
| Immich | `https://photos.<tailnet>.ts.net` | `127.0.0.1:8400` |
| Outline | `https://notes.<tailnet>.ts.net` | `localhost:8401` |
| TaskView | `https://todo.<tailnet>.ts.net` | `localhost:8402` |
| Beszel | `https://beszel.<tailnet>.ts.net` | `localhost:8100` |
| Portainer | `https://portainer.<tailnet>.ts.net` | `127.0.0.1:8000` |

## Tailscale ACLs (Access Control Lists)
Access is governed by Tailscale ACLs, which enforce a strict zero-trust model. 

- **Service Access:** ACLs are configured to allow all authenticated tailnet users to access the ports of the server's services and the `tsidp` identity provider.
- **Client Isolation:** The ACLs explicitly prevent tailnet clients from accessing other clients. Users can reach PiCloud, but they cannot see or communicate with each other's devices on the tailnet.

## Port Mapping
To prevent conflicts, services follow a strict port mapping strategy:

| Range | Purpose | Examples |
|-------|---------|----------|
| 8000-8099 | Administration | Portainer |
| 8100-8199 | Monitoring | Beszel |
| 8200-8299 | Infrastructure | Beszel |
| 8300-8399 | App APIs & Services | TaskView MCP |
| 8400-8499 | User Applications | Immich, Outline, TaskView |