# Monitoring Stack

> All monitoring is hosted on the Core VPS. Foundation is up and running;
> alerting is a work in progress.

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

## What Prometheus Monitors

Prometheus scrapes metrics from targets across the entire hybrid architecture
via the WireGuard tunnel:

| Target | What's scraped |
|--------|---------------|
| **Edge VPS** | Caddy metrics (request rate, latency, errors) |
| **Core VPS** | Node metrics (CPU, RAM, disk, network) |
| **Proxmox node** | PVE metrics via PVE-exporter (on-prem CT) |
| **Both VPS** | Node-exporter metrics |

All targets are reachable through the WireGuard daisy-chain — Prometheus on
the Core VPS scrapes on-prem targets via the site-to-site tunnel, and Edge
VPS targets via the Core → Edge tunnel.

## Dashboards

All metrics are displayed in Grafana, accessed VPN-only on the Core VPS.
*(Specific dashboard details to be documented as they are built out.)*

## Alerting — Work In Progress

No alerts are configured yet. The alerting policy is still being defined.
Planned alerts include:

- Service downtime (via Blackbox / Uptime Kuma)
- Disk usage thresholds on all nodes
- High CPU / memory usage
- WireGuard tunnel connectivity loss
- Caddy error rate spikes
- Proxmox node health

### Pending Decisions

- **Notification target:** ntfy (already running on Edge VPS), and/or email
  via the Edge Postfix → ProtonMail relay
- **Severity levels:** what's a page vs. a log vs. a quiet notification
- **Thresholds:** specific values for disk, CPU, memory alerts per node

---

*Last updated: June 2026.*
