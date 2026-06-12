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
- [x] FreeIPA realm established on-prem (Internal VPS)
- [x] All machines joined to FreeIPA (SSH auth, DNS zones, HBAC)

## Phase 3 — Infrastructure as Code (In Progress)

- [ ] Terraform/Pulumi definitions for VPS provisioning
- [ ] Ansible playbooks for OS hardening and service deployment
- [ ] Docker Compose stacks fully in CI/CD pipeline
- [ ] Automated backup verification

## Phase 4 — On-Prem Decommissioning (Planned)

- [ ] Identify candidate services for full cloud migration
- [ ] Migrate remaining Proxmox workloads to VPS
- [ ] Reduce on-prem footprint (keep only TrueNAS + PBS for storage)
- [ ] Document lessons from each migration
- [ ] Evaluate whether Proxmox PVE node can be retired or repurposed

---

*Last updated: June 2026.*
