# Core VPS Services

> The always-on cloud workloads hosted on the Core VPS (IONOS, 6 vCPU / 8 GB /
> 240 GB).

---

## Running Services

| Service | Type | Access |
|---------|------|--------|
| **Vikunja** | Task management app | VPN-only |
| **Memos** | Note-taking app | VPN-only |
| **SeaFile** | Self-hosted file sync | VPN-only |
| **Grafana** | Monitoring dashboards | VPN-only |
| **Prometheus** | Metrics collection | VPN-only (scrapes remote targets via WireGuard) |
| **Blackbox Exporter** | HTTP/HTTPS probing | WIP |
| **Uptime Kuma** | Uptime monitoring | WIP |
| **Alertmanager** | Alert routing | WIP |
| **Docker** | Container runtime | Managed via Portainer |

---

## Service Placement Rationale

These services share a common profile:

- **Need to be always-on** — migrating them to the cloud eliminates dependency
  on home power and residential uptime
- **Light compute** — the Edge VPS (2 vCPU / 2 GB) would be too constrained;
  the Core VPS handles them comfortably
- **Not public-facing** — all are accessed through the WireGuard VPN, never
  exposed to the internet. This keeps the security boundary clean.

## Storage

SeaFile is the heaviest storage user. The 240 GB disk on the Core VPS is
sufficient for now, with off-site backup to TrueNAS via the WireGuard tunnel.

## Docker Management

Containers are defined as Docker Compose stacks and managed through Portainer.
Compose files are version-controlled in the on-prem Gitea instance and pulled
to the Core VPS for deployment.

## Monitoring

The full monitoring stack (Prometheus, Grafana, Blackbox, Uptime Kuma,
Alertmanager) runs on the Core VPS. See
[18-Monitoring.md](18-Monitoring.md) for details.

---

For full service placement across the entire architecture, see
[20-Production-architecture.md](20-Production-architecture.md).
