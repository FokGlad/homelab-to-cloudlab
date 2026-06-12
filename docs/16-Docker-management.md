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
| **Gitea** | Self-hosted Git repository for all Compose files and configs | Infra VLAN (CT) |
| **Portainer** | Docker management UI/API | Infra VLAN (CT) |
| **Docker Engine** | Container runtime | Proxmox LXC containers + Core VPS |

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
- A **Gitea Runner** (CT on Infra VLAN) will watch for pushes to the main
  branch
- On push, it calls the **Portainer API** to redeploy the affected stack
- This gives a full GitOps workflow: push to Gitea → automatic redeploy
