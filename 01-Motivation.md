# Motivation

> Why move a working homelab to a hybrid VPS + on-prem architecture?

---

## The Problem: Energy Cost

Running a homelab 24/7 can be expensive. The Proxmox node, TrueNAS
storage, switch, firewall — all powered on continuously, all adding
to the electricity bill. Roughly 20-25 EUR/month for the rack alone.

The instinct was simple: don't run everything at home, all the time.

---

## The Goal: 3-Tier Hybrid Architecture

Split the infrastructure into three tiers, each optimized for its role:

| Tier | Location | Role | Uptime |
|------|----------|------|--------|
| **On-prem** | Home | Heavy compute + storage | Can be shut down at night |
| **Core VPS** | Cloud (IONOS) | Always-on containerized services | 24/7 |
| **Edge VPS** | Cloud (IONOS) | Public-facing reverse proxy + lightweight services | 24/7 |

This reduces power draw at home: the most expensive hardware sleeps when it
isn't needed, while lightweight VPS instances handle always-on duties.

---

## Cost Reality Check

The honest numbers:

```
Before:  ~20-25 EUR/month (electricity only)
Now:     ~15 EUR/month (electricity) + 9 EUR/month (VPS) = ~24 EUR/month
```

The hybrid transition did **not** reduce total monthly costs. For a proper cost
optimization, I'd need to replace the rack hardware with modern low-power
CPUs and efficient PSUs — which I can't afford right now. The VPS approach
was the budget-friendly alternative: rent efficient hardware by the hour.

The real gains are in **flexibility and capability**, not savings. See
[32-Cost-analysis.md](32-Cost-analysis.md) for the full breakdown.

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
