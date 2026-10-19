# Security Model & Identity

## Firewall (Default Deny)
The host firewall follows a **default deny** policy, managed via iptables. 

**Allowed inbound rules (from the local LAN):**
- **ICMP** (ping)
- **SSH** — Sysadmin only
- **OMV Panel** — Sysadmin only
- **SMB** — Users
- **All other traffic is DROPped.**

Access to user applications and SMB is exclusively routed through the Tailscale tailnet. Docker manages its own internal `DOCKER` chain to isolate containers. Each stack (`immich_default`, `outline_default`, etc.) runs on its own isolated bridge network with strict `iptables` rules preventing cross-container communication unless explicitly routed.

## Identity & Single Sign-On (SSO)
SSO is provided by `tsidp` (Tailscale's OAuth identity provider) running standalone in Docker with `TAILSCALE_USE_WIP_CODE=1`. It operates entirely within the tailnet as `idp.<tailnet>.ts.net`, avoiding the need for a public callback URL required by Keycloak or Authentik.

- **Outline:** Uses `OIDC_ISSUER_URL=https://idp.<tailnet>.ts.net`
- **Immich & TaskView:** Integrated natively via OIDC.

## Known Limitations & Risks
- **`tsidp` is experimental (WIP).** Tailscale marks it as under active development and subject to breaking changes. If it fails, services fall back to local authentication (this fallback is not automated).
- **Single HDD.** No redundancy. If the 1TB disk fails before SnapRAID is implemented, data is lost.
- **No automated backups yet.** SnapRAID (planned) provides parity, but not a true 3-2-1 backup.
- **No UPS.** Sudden power loss can interrupt writes.