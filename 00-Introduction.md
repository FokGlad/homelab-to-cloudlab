# From DevOops to DevOps

> Architecting a hybrid on-prem + cloud infrastructure from scratch — and
> documenting every decision, failure, and lesson learned along the way.

---

## The Journey in Three Acts

### Act 1 — The Homelab (On-Premise)

A Proxmox hypervisor, TrueNAS storage, and an OPNsense firewall running on
dedicated hardware. A Cisco SG-300 managed switch handling VLAN segmentation.
A Raspberry Pi in the DMZ running Cloudflare Tunnels for public access.
Everything accessible via WireGuard VPN only. A solid foundation, but
single-homed, power-hungry, and constrained by residential infrastructure.

### Act 2 — The Hybrid Bridge (VPS + On-Prem)

Two VPS instances provisioned and hardened. A WireGuard **daisy-chain**
connects all three tiers:

- **Core VPS** connects to Edge and on-prem, but denies all public
  connections. SSH is only accessible through the VPN tunnel — no public
  SSH access at all.
- **Edge VPS** connects to on-prem through Core (never directly). It is
  public-facing for web services, but SSH is also tunnel-only — no public
  SSH access.

Reverse proxy, TLS, and DNS services run on the Core VPS.
Docker workloads are managed through version-controlled stacks. FreeIPA on-prem
provides centralized SSH auth, access control, and DNS zones. The homelab
is now extended, not replaced.

### Act 3 — The Cloudlab (Hybrid Production)

Edge VPS instances maintain public-facing availability through Cloudflare
proxy. Core VPS handles heavy workloads and orchestration. On-prem services
are gradually decommissioned as cloud equivalents prove reliable.
Infrastructure is fully codified, reproducible, and documented.

---

## Who Am I

A self-taught infrastructure enthusiast and career-changer transitioning into
DevOps and Cloud Architecture. Holds an **AWS Cloud Practitioner** certification
and pursuing the **AWS Solutions Architect Associate**. Rather than studying
from slides alone, I learn by building. This repository is the living record of
that build.

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
