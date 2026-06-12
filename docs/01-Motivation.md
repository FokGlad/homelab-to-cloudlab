# Motivation

> Why move a working homelab to a hybrid VPS + on-prem architecture?

---

## The Problem: Energy Cost

Running a full data center at home 24/7 is expensive. The Proxmox node, TrueNAS
storage, switch, firewall, and a Raspberry Pi in the DMZ — all powered on
continuously, all adding to the electricity bill.

The obvious fix: don't run everything at home, all the time.

---

## The Solution: 3-Tier Hybrid Architecture

Split the infrastructure into three tiers, each optimized for its role:

| Tier | Location | Role | Uptime |
|------|----------|------|--------|
| **On-prem** | Home | Heavy compute + storage | Can be shut down at night |
| **Core VPS** | Cloud (IONOS) | Always-on containerized services | 24/7 |
| **Edge VPS** | Cloud (IONOS) | Public-facing reverse proxy + lightweight services | 24/7 |

This cuts power consumption dramatically: the most expensive hardware sleeps
when it isn't needed, while the lightweight VPS instances (cheap to run)
handle always-on duties.

---

## What Made This Worth It

Beyond energy savings, the hybrid approach unlocked things the old homelab
couldn't do:

- **Security:** No public-facing services on the homelab anymore. All public
  traffic terminates on the Edge VPS. The home network is never exposed.
- **Always-on without the cost:** Services that benefit from 24/7 availability
  run on the Core VPS, independent of home power and residential ISP uptime.
- **Mail without the pain:** Postfix on the Edge VPS relays to ProtonMail's SMTP.
  No need to run a full mail stack and fight deliverability alone.
- **Identity federation:** Centralized FreeIPA realm for SSH access control,
  permission management, and DNS zones dedicated to machine hostnames.
- **New possibilities:** SeaFile — self-hosted file sync that needs always-on
  availability without running on expensive home hardware.
- **Disposability:** VPS instances can be rebuilt from an Ansible playbook in
  minutes. No more flashing SD cards for a DMZ Pi.

---

## What Migrated, and Why

Services previously running on the DMZ Raspberry Pi and the Proxmox node were
evaluated against a single criterion: **would this benefit from better uptime
and always-on availability?** If yes, it moved to the cloud. Public-facing
lightweight services went to the Edge VPS. Everything else stayed on-prem.
