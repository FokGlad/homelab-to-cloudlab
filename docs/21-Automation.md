# Automation — Ansible & Semaphore

> Replacing ad-hoc scripts and cronjobs with a structured automation platform.

---

## The Problem

Maintenance tasks across the infrastructure were handled by local scripts
cronjobs on individual machines. This works but doesn't scale:

- No central visibility into what ran and when
- Scripts are tied to the machine they live on
- No audit trail or approval flow
- Easy to forget what exists where

---

## The Solution: Ansible + Semaphore

**Semaphore** (the Ansible web UI) runs on-prem and serves as the central
automation platform:

- **Ansible playbooks** define the tasks (updates, deployments, config changes)
- **Semaphore** provides the scheduling, execution, and audit log
- Replaces scattered cronjobs with version-controlled, visible automation

---

## What's Automated

### VPS Updates

Both VPS instances are updated through Ansible playbooks triggered via
Semaphore, replacing per-machine `unattended-upgrades` cronjobs with a
controlled, logged process.

## VPS Rebuild Playbooks

Both VPS instances share a **common Ansible bootstrap playbook** that handles
initial provisioning:

- Admin user creation
- SSH hardening (non-default port, key-only, no root)
- WireGuard tunnel setup
- UFW configuration

What comes after the bootstrap differs per instance.

### Core VPS Rebuild

After the bootstrap playbook:

1. Install Docker Engine
2. Deploy Portainer agent
3. Portainer pulls Compose files from the on-prem Gitea instance
4. All services come up from Git-controlled definitions

### Edge VPS Rebuild

After the bootstrap playbook, a second playbook handles service deployment:

- Caddy reverse proxy + TLS
- Postfix relay to ProtonMail
- ntfy push notifications
- UFW rules
- WireGuard tunnel to Core VPS
- Node Exporter for Prometheus

See [24-Disaster-Recovery.md](24-Disaster-Recovery.md) for the full recovery
walkthrough.

### On-Prem Tasks

*(Being migrated from cronjobs to Ansible/Semaphore over time.)*

---

## Architecture

```
┌─────────────────────────────┐
│  Semaphore (on-prem)        │
│  - Web UI for Ansible       │
│  - Scheduling               │
│  - Audit log                │
└──────────┬──────────────────┘
           │ SSH (via WireGuard for VPS)
    ┌──────┼──────┐
    │      │      │
  Core  Edge   On-prem
  VPS   VPS    hosts
```

Semaphore reaches the VPS instances through the WireGuard tunnel. No public
access required.

---

## Future Plans

- Migrate remaining on-prem cronjobs to Semaphore
- Add Gitea Runner as a Semaphore-triggered task (CI/CD pipeline)
- Use Semaphore for rolling updates across the fleet
