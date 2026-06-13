# Monitoring Stack

> All monitoring is hosted on the Core VPS. Foundation is running; alerting is
> a work in progress.

---

## Components

| Component | Role | Status |
|-----------|------|--------|
| **Prometheus** | Metrics collection and storage | ✅ Running |
| **Grafana** | Dashboards and visualization | ✅ Running |
| **Blackbox Exporter** | HTTP/HTTPS endpoint probing | WIP |
| **Uptime Kuma** | Uptime monitoring and status page | WIP |
| **Alertmanager** | Alert routing and notification | WIP |

---

## What Prometheus Scrapes

Prometheus scrapes metrics from targets across the entire hybrid architecture
via the WireGuard daisy-chain:

| Target | What's Scraped |
|--------|----------------|
| **Edge VPS** | Caddy metrics (request rate, latency, errors) via Node Exporter |
| **Core VPS** | Node metrics (CPU, RAM, disk, network) via Node Exporter |
| **Proxmox node** | PVE metrics via PVE-exporter (on-prem LXC container) |

All targets are reachable through the WireGuard tunnel — Prometheus on the
Core VPS scrapes on-prem targets via the site-to-site tunnel, and Edge VPS
targets via the Core → Edge leg.

---

## Dashboards

Grafana renders all metrics from Prometheus. The dashboards cover:
infrastructure health (CPU, memory, disk, network) for every node, and
application-level metrics (Caddy request rates, error rates).

Grafana is accessed **VPN-only** on the Core VPS — no public exposure.

*[Architecture diagram placeholder — to be added]*

---

## Alerting — Planned

No alerts are configured yet. The alerting stack (Alertmanager + notifiers)
still needs to be stood up and tuned. The planned alert categories are:

- **Service downtime** — detected via Blackbox Exporter or Uptime Kuma
- **Resource thresholds** — disk, CPU, memory on all nodes
- **Tunnel health** — WireGuard connectivity loss between any pair of sites
- **Application errors** — Caddy error rate spikes, container restarts

### Open Questions

- **Notification target:** ntfy (already running on Edge VPS) and/or email via
  the Edge Postfix → ProtonMail relay
- **Severity levels:** what pages vs. logs vs. silently records
- **Thresholds:** per-node values for disk, CPU, memory warnings and criticals
