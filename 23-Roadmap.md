# Roadmap

> What's done, what's next, and what the endgame looks like.

---

## Phase 1 — Foundation ✅

- [x] Two VPS provisioned from IONOS (Core + Edge)
- [x] Both VPS hardened (key-only SSH, SSH through WireGuard, auto-updates)
- [x] WireGuard daisy-chain established (on-prem ↔ Core ↔ Edge)
- [x] Cloudflare proxy configured (replacing Cloudflare Tunnels)
- [x] Reverse proxy (Caddy/Traefik) on Edge VPS
- [x] DNS, TLS configured
- [x] Docker stacks running on Core VPS, Git-controlled

## Phase 2 — Service Migration ✅

- [x] Services migrated from DMZ Pi and Proxmox to VPS (always-on availability)
- [x] Postfix migrated to Edge VPS (ProtonMail relay)
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

## Phase 5 — Automation (In Progress)

- [x] Semaphore running on-prem as Ansible web UI
- [x] Shared VPS bootstrap playbook (user creation, SSH hardening, WireGuard) — written but **untested**
- [x] Edge VPS rebuild playbook (full reprovisioning from scratch)
- [x] VPS update playbooks via Semaphore (replacing ad-hoc cronjobs)
- [x] On-prem VMs/servers use Ansible for automated updates
- [ ] Test shared VPS bootstrap playbook against a fresh instance
- [ ] Migrate remaining on-prem cronjobs to Semaphore
- [ ] Gitea Runner → Portainer CI/CD as Semaphore task

## Phase 6 — Infrastructure as Code + Decommissioning (Planned)

- [ ] Identify more candidate services for full cloud migration
- [ ] Document lessons from each migration

---

*Last updated: June 2026.*
