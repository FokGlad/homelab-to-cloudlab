# Service Migration

> What moved, where it landed, and what was learned along the way.

---

## Migration Rationale

Services previously running on the DMZ Raspberry Pi and the Proxmox node were
evaluated for migration. The criterion was simple: **would this service benefit
from better uptime and always-on availability?** If yes, it was a candidate for
the cloud.

Services that didn't need to be public-facing and weren't heavy compute or
storage were migrated to the Core VPS. Public-facing lightweight services went
to the Edge VPS. Everything else stayed on-prem.

---

## Mail: Two-Tier Postfix Relay

Instead of running a full mail server (with all the deliverability headaches),
the mail setup uses a **two-tier Postfix relay**:

```
On-prem devices & VMs
        │
        ▼
┌───────────────────┐
│  Local Postfix    │  ← On-prem (Infra VLAN)
│  (store & forward)│     Queues mail if upstream is down
└───────┬───────────┘
        │  Forward over WireGuard
        ▼
┌───────────────────┐
│  Edge Postfix     │  ← Edge VPS (systemd)
│  (smart relay)    │     Authenticates to ProtonMail SMTP
└───────┬───────────┘
        │  SMTP relay
        ▼
┌───────────────────┐
│  ProtonMail SMTP  │  ← Final delivery
└───────────────────┘
```

### Local Postfix (On-Prem)

- Runs as an LXC container on the **Infra VLAN**
- Accepts mail from on-prem devices and VMs
- **Store-and-forward:** if the upstream (Edge VPS) is unreachable, mail
  queues locally and retries automatically
- Forwards to Edge Postfix over the WireGuard tunnel

### Edge Postfix (Edge VPS)

- Runs as a **systemd service** (not Docker — see [30-Lessons-learned.md](30-Lessons-learned.md))
- Acts as a **smart relay** to ProtonMail's SMTP servers
- Authenticates to ProtonMail using stored credentials
- No need to manage DNS records for mail reputation (SPF, DKIM, DMARC)
  beyond the basics — ProtonMail handles deliverability
- Inbound mail is handled natively by ProtonMail (not self-hosted)

### Why Two Tiers?

The local Postfix provides **decoupling**. If the Edge VPS is down, the
WireGuard tunnel is broken, or ProtonMail's SMTP is temporarily unreachable,
mail from on-prem services queues locally. Once connectivity is restored,
delivery resumes automatically — no mail lost, no manual intervention.

This was one of the clearest wins of the VPS transition — **mail without the
pain**.

---

## DNS: Cloudflare Proxy (Replacing Cloudflare Tunnels)

The original setup used Cloudflare Tunnels (cloudflared) on the DMZ Raspberry
Pi to expose services without opening ports.

In the hybrid setup, this was replaced by **Cloudflare's reverse proxy**:

- Hostnames point to the Edge VPS IP via Cloudflare DNS (proxy mode, orange cloud)
- Cloudflare handles DDoS protection and TLS termination at the edge
- Edge VPS runs a reverse proxy (Caddy/Traefik) that routes to backend services
- No `cloudflared` daemon needed — standard HTTP/HTTPS, proxied by Cloudflare

**Advantage over tunnels:** simpler stack on the VPS, no vendor-specific tunnel
daemon, and Cloudflare still provides the same WAF + DDoS benefits.

---

## Identity: FreeIPA

FreeIPA runs as a VM on the **Internal VLAN** (on-prem Proxmox). It provides:

| Feature | Purpose |
|---------|---------|
| **DNS zone** | `idm.domain.ltd` — forward and reverse records for all machine hostnames |
| **SSH by hostname** | SSH to any machine using `ssh user@hostname.idm.domain.ltd` — no need to remember IPs |
| **SSH key management** | Centralized `authorized_keys` via LDAP — no more per-machine key files |
| **Authentication** | VM access is authenticated against FreeIPA (PAM/LDAP integration) |
| **HBAC** | Host-Based Access Control rules — who can SSH where |
| **Certificate authority** | Internal PKI for service certificates |

All machines in the fleet (Core VPS, Edge VPS, and on-prem hosts) join the
FreeIPA realm. No more managing `/etc/ssh/authorized_keys` on individual
machines.

---

## New Services Enabled

The hybrid architecture also made room for services that weren't practical
before:

- **SeaFile** — self-hosted file sync, replacing commercial cloud storage for
  personal use. This is a concrete example of what the hybrid architecture
  makes possible: a storage-heavy service that needs always-on availability
  without running 24/7 on expensive home hardware.
- **FreeIPA** — described above
- **Monitoring stack** — migrated from Proxmox to Core VPS for better uptime
