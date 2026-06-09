# From DevOops to DevOps

> A DevOps portfolio project: architecting a hybrid on-prem + cloud infrastructure 
> from scratch, applying IaC, containerization, and zero-trust networking.

This repository documents my journey from a self-hosted homelab to a hybrid 
VPS + on-prem architecture. It tracks every architecture and networking decision, 
every failure, every fix, and every lesson learned along the way.

## Goals

- Transition from homelab-only to a **hybrid-cloud infrastructure** (VPS + on-prem)
- Apply **real-world DevOps practices** — everything is version-controlled and reproducible
- **Learn by breaking things** (on purpose), then documenting the recovery

## Scope

| Area | What's covered |
|------|---------------|
| Provisioning | VPS setup, OS hardening, baseline security |
| Networking | VPN mesh, reverse proxies, DNS, TLS |
| Services | Mail, auth, exposed services with proper access controls |
| IaC | Terraform / Pulumi for infrastructure definition |
| Orchestration | Docker Compose → Kubernetes migration |
| Sunset | Gradual decommissioning of on-prem services |

> **Note:** This is a living document. Things break, get rebuilt, and sometimes 
> abandoned. That's the point.
