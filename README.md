# Hybrid Cloud Infrastructure Migration

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

**The solution:** a 3-tier hybrid split. Heavy compute and storage stay on-prem
and can be shut down at night. A cheap, lightweight Core VPS handles always-on
services. An Edge VPS handles public-facing traffic. The home network is never
exposed to the internet.

**The result:** the monthly cost stayed roughly the same, but the infrastructure
gained flexibility, better uptime for always-on services, stronger security
boundaries, and the ability to rebuild VPS instances from an Ansible playbook
in minutes — not hours. The real value is in capability, not savings.

This is both a **portfolio piece** (demonstrating real-world DevOps competency)
and a **reference** for anyone walking a similar path.

## Goals

- Transition from homelab-only to a **hybrid-cloud infrastructure** (VPS + on-prem)
- Apply **real-world DevOps practices** — everything is version-controlled and reproducible
- **Learn by breaking things** (on purpose), then documenting the recovery
- Build a **reference architecture** others can learn from

## Technology Stack

### Infrastructure & Networking
| Technology | Role |
|------------|------|
| **Proxmox VE** | On-prem hypervisor (VMs + LXC containers) |
| **OPNsense** | Firewall, gateway, IDS/IPS, DNS (AdGuard Home + Unbound) |
| **TrueNAS CORE** | On-prem NFS storage appliance |
| **Cisco SG-300** | Managed switch with VLAN segmentation |
| **IONOS VPS** | Two cloud instances — Core (6 vCPU/8 GB) + Edge (2 vCPU/2 GB) |
| **WireGuard** | Site-to-site + remote access VPN (daisy-chain topology) |
| **Cloudflare** | Public DNS, reverse proxy, WAF, DDoS protection |

### Services & Platforms
| Technology | Role |
|------------|------|
| **Docker + Compose** | Containerized workloads on Core VPS |
| **Portainer** | Docker management UI/API |
| **Gitea** | Self-hosted Git for all Compose files and configs |
| **Caddy** | Reverse proxy + TLS termination (Edge + on-prem split) |
| **FreeIPA** | Centralized SSH auth, HBAC, DNS zones for machine hostnames |
| **Postfix** | Two-tier mail relay (on-prem → Edge → ProtonMail SMTP) |
| **Prometheus + Grafana** | Metrics collection and dashboards |
| **ntfy** | Push notification server |

### Automation & Operations
| Technology | Role |
|------------|------|
| **Ansible** | Configuration management, VPS bootstrap, service deployment |
| **Semaphore** | Ansible web UI for scheduling, execution, and audit logging |
| **Proxmox Backup Server (PBS)** | Daily VM/CT backup and restore |
| **UFW** | Host-based firewall on all VPS and on-prem hosts |
| **AdGuard Home** | DNS-level ad/tracker blocking + zone rewrites |
| **Unbound** | Recursive DNS resolver with DNS over TLS |

## Architecture Diagram

*[Architecture diagram placeholder — to be added]*

## Documentation

### 0x — Introduction & Context
| File | Description |
|------|-------------|
| [01-Motivation.md](01-Motivation.md) | Energy cost, constraints, what migrated |

### 1x — Setup & Migration
| File | Description |
|------|-------------|
| [10-Homelab-hardware.md](10-Homelab-hardware.md) | Original on-prem hardware, network, DMZ Pi |
| [11-VPS-provisioning.md](11-VPS-provisioning.md) | IONOS VPS specs, hardening |
| [12-WireGuard-daisy-chain.md](12-WireGuard-daisy-chain.md) | WireGuard topology, SSH-only access, security model |
| [13-Core-VPS-services.md](13-Core-VPS-services.md) | Core VPS workloads: containers, monitoring, SeaFile |
| [14-Service-migration.md](14-Service-migration.md) | What moved where, FreeIPA, Postfix relay, Cloudflare proxy |
| [15-DNS-architecture.md](15-DNS-architecture.md) | Four DNS zones, Caddy split, resolution flow |
| [16-Networking.md](16-Networking.md) | VLAN layout, firewall rules, switch config |
| [17-Docker-management.md](17-Docker-management.md) | Compose files, Portainer, Gitea, planned CI/CD |
| [18-Proxmox-inventory.md](18-Proxmox-inventory.md) | Full VM/CT inventory, VLAN distribution |
| [19-Monitoring.md](19-Monitoring.md) | Metrics, dashboards, planned alerting |
| [20-Backup-strategy.md](20-Backup-strategy.md) | Current state and future plan |
| [21-Automation.md](21-Automation.md) | Ansible + Semaphore, replacing cronjobs |

### 2x — Production & Operations
| File | Description |
|------|-------------|
| [22-Production-architecture.md](22-Production-architecture.md) | Final hybrid topology, service placement, security boundaries |
| [23-Roadmap.md](23-Roadmap.md) | Phase checklist, what's next, decommissioning plan |
| [24-Security.md](24-Security.md) | SSH hardening, fail2ban strategy, access model |
| [25-Disaster-Recovery.md](25-Disaster-Recovery.md) | Recovery procedures, RTO, Ansible rebuild workflow |

### 3x — Deep Dives
| File | Description |
|------|-------------|
| [30-Lessons-learned.md](30-Lessons-learned.md) | Key decisions, trade-offs, and what I'd do differently |
| [31-Access-model.md](31-Access-model.md) | Day-to-day and remote access, VPS source IP restriction, WireGuard tunnels |
| [32-Cost-analysis.md](32-Cost-analysis.md) | Honest cost breakdown: electricity, VPS, and what would actually save money |

## License

Feel free to fork, adapt, or steal any of this. Attribution appreciated but
not required.
