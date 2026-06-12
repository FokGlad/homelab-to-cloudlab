# Core VPS Services

> The always-on cloud workloads hosted on the Core VPS (IONOS, 6 vCPU / 8 GB /
> 240 GB).

---

## What Runs Here

The Core VPS hosts **containerized services** that benefit from always-on
availability without the cost of running them on home hardware 24/7. All
services run in **Docker Compose** stacks managed by Portainer.

Access is **VPN-only** — nothing on the Core VPS is public-facing.

## Storage

The 240 GB disk handles the container workloads comfortably. The heaviest
storage user is SeaFile (self-hosted file sync), with off-site backup to
TrueNAS via the WireGuard tunnel.

## Docker Management

Containers are defined as Docker Compose stacks and managed through Portainer.
Compose files are version-controlled in the on-prem Gitea instance and pulled
to the Core VPS for deployment.

---

For full service placement across the entire architecture, see
[20-Production-architecture.md](20-Production-architecture.md).
