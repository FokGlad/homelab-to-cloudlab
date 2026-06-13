# Disaster Recovery

> How fast can you rebuild from scratch, and what the recovery procedures look
> like for each tier.

---

## Recovery Time Objective

| Tier | Backup Method | RTO | Notes |
|------|--------------|-----|-------|
| **On-prem (Proxmox)** | Daily PBS backup | < 12 hours | Assumes no hardware failure; restore from PBS |
| **Core VPS** | Ansible + Gitea | Minutes | Rebuild from fresh Debian, pull compose files |
| **Edge VPS** | Ansible playbooks | Minutes | Rebuild from fresh Debian, redeploy services |

---

## On-Prem Recovery

Proxmox Backup Server (PBS) backs up all VMs and LXC containers daily. In the
event of a Proxmox node failure:

1. Install Proxmox on new or existing hardware
2. Connect to PBS
3. Restore VMs/CTs from the latest backup
4. Rejoin FreeIPA realm (if the FreeIPA VM was affected, restore it first —
   everything else depends on it)

**Recovery time: under 12 hours** if no hardware needs replacing. The longest
step is the PBS restore itself, which depends on data size and network speed.

---

## Core VPS Recovery

The Core VPS is designed to be **fully disposable**. Recovery from a fresh
Debian install:

### Step 1 — Ansible Bootstrap (shared playbook)

The same bootstrap playbook used for initial provisioning:

- Create admin user
- Apply SSH hardening (non-default port, key-only, no root)
- Install and configure WireGuard
- Configure UFW

### Step 2 — Docker + Portainer

- Install Docker Engine
- Deploy Portainer agent
- Portainer connects to the on-prem Gitea instance and pulls the latest
  Compose files

### Step 3 — Services Come Up

All service definitions live in Gitea. Portainer recreates the containers
automatically. No manual service configuration required.

**Recovery time: minutes.** The entire Core VPS can be rebuilt from scratch by
running the Ansible playbook and letting Portainer do the rest.

---

## Edge VPS Recovery

Same two-phase approach as Core, with a second playbook for services:

### Phase 1 — Ansible Bootstrap (shared playbook)

Identical to the Core bootstrap:

- Create admin user
- Apply SSH hardening
- Configure SSH port and `sshd_config`
- Set up WireGuard tunnel to Core VPS

### Phase 2 — Services Playbook

A second Ansible playbook handles service deployment:

| Service | Role |
|---------|------|
| **Caddy** | Reverse proxy + TLS termination |
| **Postfix** | SMTP relay to ProtonMail |
| **ntfy** | Push notification server |
| **UFW** | Firewall rules |
| **WireGuard** | Tunnel to Core VPS (also configured in Phase 1) |
| **Node Exporter** | Metrics for Prometheus |

**Recovery time: minutes.** Run both playbooks against a fresh Debian install
and the Edge VPS is back.

---

## The Shared Bootstrap

The Ansible bootstrap script is **the same for both VPS instances**. The
difference is what comes after:

```
Fresh Debian install
        │
        ▼
┌─────────────────────┐
│  Shared Bootstrap   │
│  - Admin user       │
│  - SSH hardening    │
│  - WireGuard        │
│  - UFW              │
└─────────┬───────────┘
          │
    ┌─────┴─────┐
    │           │
    ▼           ▼
┌────────┐  ┌──────────────┐
│  Core  │  │  Edge        │
│  +Docker│  │  +Services   │
│  +Portainer│ │  playbook   │
│  +Gitea │  │  (Caddy,     │
│  pull   │  │   Postfix,   │
└────────┘  │   ntfy, etc.)│
            └──────────────┘
```

This means spinning up a replacement VPS — or a new one — is a matter of
running the playbooks against a fresh Debian image. No manual steps, no
documentation lookup, no "how did I set this up again?"
