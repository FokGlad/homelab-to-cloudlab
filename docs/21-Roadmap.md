# Roadmap

> What's done, what's next, and what the endgame looks like.

---

## Phase 1 — Foundation ✅

- [x] Two VPS provisioned from IONOS (Core + Edge)
- [x] Both VPS hardened (key-only SSH, fail2ban, auto-updates)
- [x] WireGuard daisy-chain established (on-prem ↔ Core ↔ Edge)
- [x] Cloudflare proxy configured (replacing Cloudflare Tunnels)
- [x] Reverse proxy (Caddy/Traefik) on Edge VPS
- [x] DNS, TLS configured
- [x] Docker stacks running on Core VPS, Git-controlled

## Phase 2 — Service Migration ✅

- [x] Postfix migrated to Edge VPS (ProtonMail relay)
- [x] ntfy migrated to Edge VPS
- [x] Vikunja, Memos migrated to Core VPS
- [x] Grafana + Prometheus migrated to Core VPS
- [x] SeaFile deployed on Core VPS
- [x] FreeIPA realm established on-prem (Internal VLAN)
- [x] All machines joined to FreeIPA (SSH auth, DNS zones, HBAC)

## Phase 3 — Monitoring & Alerting (In Progress)

- [x] Prometheus + Grafana running on Core VPS
- [x] Edge VPS Caddy metrics being scraped
- [x] Proxmox metrics via PVE-exporter being scraped
- [ ] Blackbox Exporter (WIP)
- [ ] Uptime Kuma (WIP)
- [ ] Alertmanager with defined alerting policy
- [ ] Disk usage alerts on all nodes
- [ ] Service downtime alerts
- [ ] Notification target decision (ntfy vs. email)

## Phase 4 — Backup Strategy (Planned)

- [x] VPS data pushed to TrueNAS NFS share → Backup relay VM → PBS
- [ ] Off-site TrueNAS replication (ZFS send/receive)
- [ ] Blu-ray archival for important documents
- [ ] Define retention policy and replication schedule

## Phase 5 — Infrastructure as Code (Planned)

- [ ] Terraform/Pulumi definitions for VPS provisioning
- [ ] Ansible playbooks for OS hardening and service deployment
- [ ] Gitea Runner → Portainer CI/CD pipeline
- [ ] Automated backup verification

## Phase 6 — On-Prem Decommissioning (Future)

- [ ] Identify candidate services for full cloud migration
- [ ] Migrate remaining Proxmox workloads to VPS
- [ ] Reduce on-prem footprint (keep only TrueNAS + PBS for storage)
- [ ] Document lessons from each migration
- [ ] Evaluate whether Proxmox PVE node can be retired or repurposed

---

*Last updated: June 2026.*
