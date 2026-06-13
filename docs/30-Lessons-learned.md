# Lessons Learned

> Key decisions, trade-offs, and what I'd do differently — so far.

---

## Why systemd on the Edge VPS instead of Docker?

The Edge VPS runs Caddy, Postfix, and ntfy as **systemd service units** rather
than Docker containers. The reasoning:

- **Resource budget:** 2 vCPU / 2 GB RAM. Docker adds overhead that matters at
  this scale.
- **Simplicity:** one less layer to debug. When Postfix isn't relaying, I check
  `systemctl status`, not `docker compose logs`.
- **Rebuild speed:** the Ansible playbook installs and configures systemd units
  directly. No container registry, no image pulls, no compose file management
  for three services.

The Core VPS uses Docker because it has the resources and runs enough services
to benefit from container isolation.

---

## Why a daisy-chain instead of full mesh?

With only three nodes (Edge, Core, On-prem), a full mesh would mean three
tunnels instead of two. The daisy-chain saves one tunnel and adds a security
benefit: the Edge VPS can never reach the homelab directly. If the edge is
compromised, the attacker still has to get through Core.

The trade-off is that Core becomes a single point of failure for cross-site
traffic. Acceptable for a homelab; for production, I'd add a second Core VPS.

---

## Why FreeIPA on-prem instead of the cloud?

FreeIPA is the identity provider — if it goes down, SSH access to everything
breaks. Running it on-prem means:

- It's on the most reliable network segment (Internal VLAN, wired)
- No dependency on cloud provider uptime for authentication
- Latency for on-prem hosts is near-zero

The trade-off is that cloud VPS instances need a working WireGuard tunnel to
reach FreeIPA for authentication. If the tunnel goes down, already-authenticated
sessions survive, but new SSH logins don't work until it's restored.

---

## What the hybrid architecture made possible

The clearest example is **SeaFile** — self-hosted file sync that needs
always-on availability. Running it on-prem meant either keeping the Proxmox
node powered on 24/7 (expensive) or accepting that files wouldn't be available
at night. Running it on the Core VPS gives 24/7 access without the energy cost.

This is the pattern: any service that needs to be always-on but isn't
public-facing belongs on the Core VPS.

---

## What I'd do differently

- **Start with Ansible from day one.** I provisioned the first VPS manually,
  then had to reverse-engineer the steps into a playbook. Starting with
  Ansible would have saved time and made the first rebuild painless.
- **Define the backup strategy before migrating services.** I migrated first
  and figured out backups later. Some data lived on the VPS for weeks without
  a proper backup path.
- **Monitoring before migration.** I set up Prometheus after the migration was
  mostly done. Having metrics from the start would have made it easier to
  catch issues during the transition.
- **Account for total cost from the start.** The hybrid transition didn't
  reduce my monthly costs — it stayed roughly the same. The value is in
  flexibility and capability, not savings. If cost reduction is the goal,
  the answer is efficient hardware, not cloud migration. See
  [32-Cost-analysis.md](docs/32-Cost-analysis.md) for the full breakdown.
