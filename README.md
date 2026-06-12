# From DevOops to DevOps

> A DevOps portfolio project: architecting a hybrid on-prem + cloud infrastructure
> from scratch, applying IaC, containerization, and zero-trust networking.

This repository documents the journey from a self-hosted homelab to a hybrid
VPS + on-prem architecture. It tracks every architecture and networking decision,
every failure, every fix, and every lesson learned along the way.

## Goals

- Transition from homelab-only to a **hybrid-cloud infrastructure** (VPS + on-prem)
- Apply **real-world DevOps practices** — everything is version-controlled and reproducible
- **Learn by breaking things** (on purpose), then documenting the reduction

## Documentation

### 0x — Introduction & Motivation
| File | Description |
|------|-------------|
| [00-Introduction.md](00-Introduction.md) | Project overview, three acts, guiding principles |
| [01-Motivation.md](docs/01-Motivation.md) | Energy cost, constraints, why hybrid, migration candidates |

### 1x — Setup & Migration
| File | Description |
|------|-------------|
| [10-Homelab-hardware.md](docs/10-Homelab-hardware.md) | Original on-prem hardware, network, DMZ Pi |
| [11-VPS-provisioning.md](docs/11-VPS-provisioning.md) | IONOS VPS specs, hardening |
| [12-WireGuard-daisy-chain.md](docs/12-WireGuard-daisy-chain.md) | WireGuard topology, SSH-only access, security model |
| [12b-Core-VPS-services.md](docs/12b-Core-VPS-services.md) | Core VPS workloads: containers, monitoring, SeaFile |
| [13-Service-migration.md](docs/13-Service-migration.md) | What moved where, FreeIPA, Postfix relay, Cloudflare proxy |
| [14-DNS-architecture.md](docs/14-DNS-architecture.md) | Four DNS zones, Caddy split, resolution flow |
| [15-Networking.md](docs/15-Networking.md) | VLAN layout, firewall rules, switch config |
| [16-Docker-management.md](docs/16-Docker-management.md) | Compose files, Portainer, Gitea, planned CI/CD |
| [17-Proxmox-inventory.md](docs/17-Proxmox-inventory.md) | Full VM/CT inventory, VLAN distribution |
| [18-Monitoring.md](docs/18-Monitoring.md) | Prometheus, Grafana, Blackbox, Uptime Kuma, Alertmanager |
| [19-Backup-strategy.md](docs/19-Backup-strategy.md) | Current state and future plan |
| [22-Automation.md](docs/22-Automation.md) | Ansible + Semaphore, replacing cronjobs |

### 2x — Production & Roadmap
| File | Description |
|------|-------------|
| [20-Production-architecture.md](docs/20-Production-architecture.md) | Final hybrid topology, service placement, security boundaries |
| [21-Roadmap.md](docs/21-Roadmap.md) | Phase checklist, what's next, decommissioning plan |

## License

Feel free to fork, adapt, or steal any of this. Attribution appreciated but
not required.
