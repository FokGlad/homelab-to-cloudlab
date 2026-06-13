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

**Outcomes:**
- ✅ No inbound connections to the homelab network — all public traffic terminates on Edge VPS
- ✅ SSH accessible only through WireGuard — zero public SSH exposure across the fleet
- ✅ VPS instances reproducible from scratch via Ansible

---

## Phase 2 — Service Migration ✅

- [x] Services migrated from DMZ Pi and Proxmox to VPS (always-on availability)
- [x] Postfix migrated to Edge VPS (ProtonMail relay)
- [x] FreeIPA realm established on-prem (Internal VLAN)
- [x] All machines joined to FreeIPA (SSH auth, DNS zones, HBAC)

**Outcomes:**
- ✅ Always-on services (SeaFile, Synapse, Vikunja, etc.) run independently of home power and ISP
- ✅ Centralized SSH authentication — no more per-machine `authorized_keys` management
- ✅ DMZ Raspberry Pi decommissioned — one less device to maintain and secure

---

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

**Target outcomes:**
- ⬜ Alert on any node going down within 5 minutes
- ⬜ Alert on disk usage above 85% on any VPS or on-prem host
- ⬜ WireGuard tunnel connectivity loss detected and notified

---

## Phase 4 — Backup Strategy (In Progress)

- [x] VPS data pushed to TrueNAS NFS share → Backup relay VM → PBS
- [x] Proxmox VMs/CTs backed up daily to PBS
- [ ] Off-site TrueNAS replication (ZFS send/receive)
- [ ] Blu-ray archival for important documents
- [ ] Define retention policy and replication schedule

**Target outcomes:**
- ⬜ Recovery time for any VPS: under 1 hour from fresh Debian install
- ⬜ Recovery time for Proxmox node: under 12 hours (PBS restore)
- ⬜ Off-site copy of TrueNAS data at a separate physical location
- ⬜ Blu-ray archival of important documents on a semi-annual schedule

---

## Phase 5 — Automation (In Progress)

- [x] Semaphore running on-prem as Ansible web UI
- [x] Shared VPS bootstrap playbook (user creation, SSH hardening, WireGuard) — written but **untested**
- [x] Edge VPS rebuild playbook (full reprovisioning from scratch)
- [x] VPS update playbooks via Semaphore (replacing ad-hoc cronjobs)
- [x] On-prem VMs/servers use Ansible for automated updates
- [ ] Test shared VPS bootstrap playbook against a fresh instance
- [ ] Migrate remaining on-prem cronjobs to Semaphore
- [ ] Gitea Runner → Portainer CI/CD as Semaphore task

**Target outcomes:**
- ⬜ Full VPS rebuild (bootstrap + services) in under 30 minutes, zero manual steps
- ⬜ All infrastructure changes version-controlled and auditable via Semaphore
- ⬜ GitOps workflow: push to Gitea → automatic redeploy via Portainer

---

## Phase 6 — Decommissioning & Optimization (Planned)

- [ ] Identify candidate services for full cloud migration
- [ ] Reduce on-prem power draw by shutting down Proxmox outside daytime hours (target: 8–10 hours/day shutdown)
- [ ] Document lessons from each migration
- [ ] Evaluate whether Proxmox PVE node can be retired or repurposed

**Target outcomes:**
- ⬜ On-prem power consumption reduced by ~40% (night shutdown schedule)
- ⬜ On-prem footprint reduced to TrueNAS + PBS only (storage and backup)
- ⬜ Every migration documented with rationale, process, and lessons learned
