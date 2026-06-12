# From DevOops to DevOps

> A DevOps portfolio project: architecting a hybrid on-prem + cloud infrastructure
> from scratch — and documenting every decision, failure, and lesson learned along
> the way.

## The Story

This repository documents my journey from a fully self-hosted homelab to a
**hybrid VPS + on-prem architecture**. It tracks every architecture and
networking decision, every failure, every fix, and every lesson learned along
the way.

**The problem:** running a full data center at home 24/7 is expensive. The
Proxmox node, TrueNAS storage, switch, firewall, and a Raspberry Pi in the DMZ
— all powered on continuously, all adding to the electricity bill.

**the solution:** a 3-tier hybrid split. Heavy compute and storage stay on-prem
and can be shut down at night. A cheap, lightweight Core VPS handles always-on
services. An Edge VPS handles public-facing traffic. The home network is never
exposed to the internet.

**the result:** lower energy costs, better uptime for critical services, stronger
security boundaries, and a flexible infrastructure that can be rebuilt in
minutes — not hours.

This is both a **portfolio piece** (demonstrating real-world DevOps competency)
and a **reference** for anyone walking a similar path.

## Goals

- Transition from homelab-only to a **hybrid-cloud infrastructure** (VPS + on-prem)
- Apply **real-world DevOps practices** — everything is version-controlled and reproducible
- **Learn by breaking things** (on purpose), then documenting the recovery
- Build a **reference architecture** others can learn from

## Documentation

### 0x — Introduction & Context
| File | Description |
|------|-------------|
| [00-Introduction.md](00-Introduction.md) | Three acts, guiding principles |
| [01-Motivation.md](docs/01-Motivation.md) | Energy cost, constraints, what migrated |

### 1x — Setup & Migration
| File | Description |
|------|-------------|
| [10-Homelab-hardware.md](docs/10-Homelab-hardware.md) | Original on-prem hardware, network, DMZ Pi |
| [11-VPS-provisioning.md](docs/11-VPS-provisioning.md) | IONOS VPS specs, hardening |
| [12-WireGuard-daisy-chain.md](docs/12-WireGuard-daisy-chain.md) | WireGuard topology, SSH-only access, security model |
| [13-Core-VPS-services.md](docs/13-Core-VPS-services.md) | Core VPS workloads: containers, monitoring, SeaFile |
| [14-Service-migration.md](docs/14-Service-migration.md) | What moved where, FreeIPA, Postfix relay, Cloudflare proxy |
| [15-DNS-architecture.md](docs/15-DNS-architecture.md) | Four DNS zones, Caddy split, resolution flow |
| [16-Networking.md](docs/16-Networking.md) | VLAN layout, firewall rules, switch config |
| [17-Docker-management.md](docs/17-Docker-management.md) | Compose files, Portainer, Gitea, planned CI/CD |
| [18-Proxmox-inventory.md](docs/18-Proxmox-inventory.md) | Full VM/CT inventory, VLAN distribution |
| [19-Monitoring.md](docs/19-Monitoring.md) | Metrics, dashboards, planned alerting |
| [20-Backup-strategy.md](docs/20-Backup-strategy.md) | Current state and future plan |
| [21-Automation.md](docs/21-Automation.md) | Ansible + Semaphore, replacing cronjobs |

### 2x — Production & Roadmap
| File | Description |
|------|-------------|
| [22-Production-architecture.md](docs/22-Production-architecture.md) | Final hybrid topology, service placement, security boundaries |
| [23-Roadmap.md](docs/23-Roadmap.md) | Phase checklist, what's next, decommissioning plan |

### 3x — Deep Dives
| File | Description |
|------|-------------|
| [30-Lessons-learned.md](docs/30-Lessons-learned.md) | Key decisions, trade-offs, and what I'd do differently |

## License

Feel free to fork, adapt, or steal any of this. Attribution appreciated but
not required.
