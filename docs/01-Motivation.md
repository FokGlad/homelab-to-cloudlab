# Motivation

> Why move a working homelab to a hybrid VPS + on-prem architecture?

---

## The Problem: Energy Cost

Running a full data center at home 24/7 is expensive. The Proxmox node, TrueNAS
storage, switch, firewall, and a Raspberry Pi in the DMZ — all powered on
continuously, all adding to the electricity bill.

The obvious fix: don't run everything at home, all the time.

---

## The Goal: 3-Tier Hybrid Architecture

Split the infrastructure into three tiers, each optimized for its role:

| Tier | Location | Role | Uptime |
|------|----------|------|--------|
| **On-prem** | Home | Heavy compute + storage | Can be shut down at night |
| **Core VPS** | Cloud (IONOS) | Always-on services + light workloads | 24/7 |
| **Edge VPS** | Cloud (IONOS) | Public-facing reverse proxy + lightweight services (ntfy, postfix) | 24/7 |

This cuts power consumption dramatically: the most expensive hardware sleeps
when it isn't needed, while the lightweight VPS instances (cheap to run)
handle always-on duties.

---

## Secondary Benefits

Moving to hybrid wasn't **only** about energy:

- **Security:** No public-facing services on the homelab anymore. All public
  traffic terminates on the Edge VPS. The home network is never exposed.
- **Flexibility:** VPS instances can be rebuilt, resized, or purged in minutes.
  No more flashing SD cards for a DMZ Pi.
- **Mail without the pain:** Postfix on the Edge VPS relays to ProtonMail's SMTP.
  No need to run a full mail stack and fight deliverability alone.
- **Identity federation:** Centralized FreeIPA realm for SSH access control,
  permission management, and DNS zones dedicated to machine hostnames.

---

## What Got Migrated

Services previously running on the DMZ Raspberry Pi and the Proxmox node were
the migration candidates:

| Service | Type | Origin | Destination |
|---------|------|--------|-------------|
| Postfix | Infra (mail) | DMZ Pi / Proxmox | Edge VPS |
| ntfy | Infra (notifications) | Proxmox | Edge VPS |
| Vikunja | App (tasks) | Proxmox | Core VPS |
| Memos | App (notes) | Proxmox | Core VPS |
| Grafana | Monitoring | Proxmox | Core VPS |
| Prometheus | Monitoring | Proxmox | Core VPS |

New services were also added after the transition (SeaFile, FreeIPA), taking
advantage of the hybrid flexibility.
