# Core VPS Services

> The always-on cloud workloads hosted on the Core VPS (IONOS, 6 vCPU / 8 GB /
> 240 GB).

---

## What Runs Here

The Core VPS hosts **containerized services** that benefit from always-on
availability without the cost of running them on home hardware 24/7. Access is
**VPN-only** — nothing on the Core VPS is public-facing.

### Service Categories

| Category | Examples | Why Cloud |
|----------|----------|-----------|
| **Productivity** | Vikunja, Memos, BookStack | Lightweight but need to be always accessible |
| **Communication** | Synapse | Needs 24/7 uptime for messaging |
| **Monitoring** | Prometheus, Grafana | Centralized metrics across the entire hybrid fleet |
| **File sync** | SeaFile | Storage-heavy; runs 24/7 without home power draw |

---

## Docker Management

Each service runs as an **independent Docker Compose stack** — one stack per
service, plus one shared monitoring stack. This keeps deployments isolated:
updating one service never affects another.

All Compose files are version-controlled in the on-prem Gitea instance and
managed through Portainer. No standalone `docker run` containers in production.

---

## Storage

The 240 GB disk handles the container workloads comfortably. The heaviest
storage user is SeaFile. Backups are handled by a script on the Core VPS that
pushes data to the on-prem TrueNAS. Snapshot-based backup is planned for the
future.

---

## Why Docker on Core but systemd on Edge?

The Core VPS has the resources to run Docker comfortably (6 vCPU / 8 GB) and
runs enough services to benefit from container isolation. The Edge VPS
(2 vCPU / 2 GB) runs only a few lightweight services as systemd units to
minimize overhead. See [30-Lessons-learned.md](30-Lessons-learned.md) for the
full rationale.

---

For full service placement across the entire architecture, see
[22-Production-architecture.md](22-Production-architecture.md).
