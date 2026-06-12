# Docker & Container Management

> All containers run via Docker Compose, managed by Portainer, version-controlled
> in Gitea.

---

## Principles

- **Every container runs via Docker Compose** — no standalone `docker run`
- **All Compose files are version-controlled** in a self-hosted Gitea instance
- **Portainer** is the management interface (no manual CLI management in production)
- **CI/CD is planned** (see [Roadmap](21-Roadmap.md) Phase 3)

---

## Stack: Gitea + Portainer

| Component | Role | Placement |
|-----------|------|-----------|
| **Gitea** | Self-hosted Git repository for all Compose files and configs | Infra VLAN |
| **Portainer** | Docker management UI/API | Infra VLAN |
| **Docker Engine** | Container runtime | Proxmox LXC/VMs (on-prem) + Core VPS |

Both Gitea and Portainer live on the **Infra VLAN** — the most restricted
segment — because they are structurally critical infrastructure.

---

## Compose File Workflow

```
Developer (or Hermes)
    │
    ▼
Edit docker-compose.yml locally
    │
    ▼
git push → Gitea (self-hosted)
    │
    ▼
[Gitea Runner] → triggers Portainer API (planned)
    │
    ▼
Portainer recreates containers on the target host
```

### Current State
- Compose files are **already in Gitea** and version-controlled
- Portainer is **already managing** all on-prem Docker workloads
- The CI/CD pipeline (Gitea Runner → Portainer API) is **not yet
  implemented** — currently, updates are applied manually through Portainer
  after pushing to Gitea

### Planned CI/CD
- A **Gitea Runner** will watch for pushes to the main branch
- On push, it calls the **Portainer API** to redeploy the affected stack
- This gives a full GitOps workflow: push to Gitea → automatic redeploy

---

## Service Categories

Docker workloads fall into three categories:

| Category | Examples | Placement |
|----------|----------|-----------|
| **Infra services** | Gitea, Portainer, Vault, FreeIPA | Infra VLAN (on-prem) |
| **App services** | Vikunja, Memos, SeaFile, ntfy | Core VPS / External VLAN |
| **Monitoring** | Grafana, Prometheus | Core VPS |

---

## Portainer Management

Portainer runs as a container itself and manages:

- **On-prem Docker hosts** (Proxmox LXC containers running Docker)
- **Core VPS Docker** (remote agent)

Stacks are defined in Portainer using the Compose files from Gitea. This means
the Gitea repo is the **source of truth** — Portainer is just the execution
layer.
