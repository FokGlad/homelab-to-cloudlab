# From DevOops to DevOps

> Architecting a hybrid on-prem + cloud infrastructure from scratch — and
> documenting every decision, failure, and lesson learned along the way.

---

## Who Am I?

I'm a self-taught infrastructure enthusiast and career-changer transitioning into
DevOps and Cloud Architecture. I hold an **AWS Cloud Practitioner** certification
and I'm pursuing the **AWS Solutions Architect Associate** along the way.

Rather than studying from slides alone, I learn by building. This repository is
the living record of that build.

## Why This Exists

Most homelab write-ups focus on the "how" — step-by-step tutorials for
deploying specific services. This project is about the **"why"**:

- Why **this** architecture over that one?
- Why split workloads across VPS and on-prem?
- Why a daisy-chain topology instead of a flat network?
- Why decommission services instead of running everything forever?

The answers come from constraints: cost, reliability, security, and the
practical reality that running a data center out of your house has limits.

This repo is both a **portfolio piece** (demonstrating real-world DevOps
competency) and a **reference** for anyone walking a similar path.

## The Journey in Three Acts

### Act 1 — The Homelab (On-Premise)

A Proxmox hypervisor, TrueNAS storage, and an OPNsense firewall running on
dedicated hardware. Everything accessible via WireGuard VPN only. No public
services. A solid foundation, but single-homed and constrained by residential
infrastructure.

### Act 2 — The Hybrid Bridge (VPS + On-Prem)

Cloud VPS provisioned and hardened. WireGuard mesh connecting cloud back to
on-prem. Reverse proxy, TLS, DNS, and identity services stood up in the cloud.
Docker workloads managed through version-controlled stacks. The homelab is now
extended, not replaced.

### Act 3 — The Cloudlab (Hybrid Production)

Edge VPS instances maintain public-facing availability. Core VPS handles heavy
workloads and orchestration. On-prem services are gradually sunset as cloud
equivalents prove reliable. Infrastructure is fully codified, reproducible, and
documented.

## Guiding Principles

- **Everything is version-controlled.** Infrastructure, configs, docs — all
  in Git. No clickOps.
- **Learn by breaking.** Deliberate failure drives deeper understanding than
  smooth sailing.
- **Document decisions, not just steps.** Anyone can follow a tutorial.
  Explaining *why* shows engineering judgment.
- **Pragmatism over perfection.** Build what works. Refactor later. Ship
  constantly.

> **Status:** Act 2 in progress. This repository is actively maintained and
> will evolve as the architecture evolves.

## License

Feel free to fork, adapt, or steal any of this. Attribution appreciated but
not required.
