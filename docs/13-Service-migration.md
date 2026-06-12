# Service Migration

> What moved, where it landed, and what was learned along the way.

---

## Migration Candidates

Six services were identified for migration from the DMZ Raspberry Pi and
Proxmox node:

| Service | Type | Original Host | Destination |
|---------|------|--------------|-------------|
| **Postfix** | Infra (mail relay) | Proxmox | Edge VPS |
| **ntfy** | Infra (push notifications) | Proxmox | Edge VPS |
| **Vikunja** | App (task management) | Proxmox | Core VPS |
| **Memos** | App (note-taking) | Proxmox | Core VPS |
| **Grafana** | Monitoring | Proxmox | Core VPS |
| **Prometheus** | Monitoring | Proxmox | Core VPS |

### Selection Criteria

Services were split between Core and Edge based on two questions:

1. **Does it need to be public-facing?** → Edge VPS
2. **Does it manage sensitive data or auth?** → Core VPS (behind VPN)

Anything that didn't need to stay on-prem (and wasn't heavy compute) was a
candidate. Heavy storage and compute workloads stayed home.

---

## Mail: Postfix to ProtonMail Relay

Instead of running a full mail server (with all the deliverability headaches),
Postfix on the Edge VPS acts as a **smart relay**:

- Outbound mail is relayed through ProtonMail's SMTP servers
- No need to manage DNS records for mail reputation (SPF, DKIM, DMARC)
  beyond the basics
- Inbound mail is handled natively by ProtonMail
- The Edge VPS only needs to know how to authenticate to ProtonMail's SMTP

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

- **SeaFile** — self-hosted file sync (replacing Google Drive / Dropbox for
  personal use)
- **FreeIPA** — described above
- Monitoring stack (Grafana + Prometheus) — migrated from Proxmox to Core VPS
  for better uptime
