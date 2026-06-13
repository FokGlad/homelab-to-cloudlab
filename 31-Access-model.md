# Access Model

> How day-to-day and remote access actually works across the hybrid
> architecture.

---

## Day-to-Day: Desktop PC on Management VLAN

The primary way to interact with the infrastructure is from a **desktop PC**
on the **Management VLAN**. This subnet has unrestricted access to all other
VLANs — Infra, Internal, External, everything. From here you can:

- SSH into any on-prem host (authenticated via FreeIPA)
- Reach the Core VPS (through the WireGuard tunnel, SSH by hostname)
- Reach the Edge VPS (through the WireGuard tunnel — see below)
- Access any web service, dashboard, or management interface

**This is the "everything is allowed" subnet.** The design assumption is that
if someone has physical access to the wired Management VLAN, they're trusted.
Future improvement: move the desktop to a regular subnet and restrict
Management VLAN to specific admin hosts only.

---

## VPS Access — Restricted by Source IP

The VPS instances have an additional layer of access control: SSH is **only
accepted from the desktop PC's IP**, and only over the WireGuard tunnel. Even
if someone has the WireGuard keys, they can't SSH to a VPS unless their
traffic appears to come from the desktop's IP.

This was set up as an ultimate security measure. In practice, it's
**overkill and counterproductive**: if the desktop is offline (power outage,
hardware failure, or you're simply not home), you lose SSH access to the VPS
entiretely. You'd connect from your phone over the admin WireGuard tunnel but
your phone's IP won't match the allowlist.

> **Status:** this restriction may be removed. The WireGuard tunnel alone
> already provides strong authentication. If you can reach the VPS over the
> tunnel, you're already trusted.

---

## Remote Access: WireGuard Admin Tunnel

For remote access (away from home), the **WireGuard Admin tunnel** is the
primary path:

- Designed for a **single host** — a phone
- Terminates on the **WireGuard Admin VLAN**
- Gives access to everything

From the phone, you SSH to machines using FreeIPA hostnames:
```bash
ssh user@proxmox.idm.domain.ltd
ssh user@core-vps.idm.domain.ltd
```
---

## Guest Access: WireGuard Guest Tunnel

For non-admin remote access (e.g., giving someone limited access):

- Terminates on the **WireGuard Guest VLAN**
- **Restricted**: can only reach select services on the **External VLAN**
- No access to Internal, Infra, Local, or Management VLANs

---

## Privilege Summary

| Access Method | Reach | Use Case |
|---------------|-------|----------|
| **Desktop (Mgmt VLAN)** | Everything | Day-to-day management |
| **WireGuard Admin** | Internal + Infra (+ Core VPS) | Remote admin (phone) |
| **WireGuard Guest** | External VLAN (limited) | Guest access |

---

## What Doesn't Work (By Design)

- **Public SSH**: neither VPS accepts SSH from the internet. VPN only.
- **Edge VPS from Admin tunnel**: Edge is only reachable through the Core VPS
  daisy-chain, not directly from the admin tunnel.
- **IoT → anything**: IoT devices can only reach the internet. No lateral
  movement to any other VLAN.
- **Guest → internal**: Guest tunnel is strictly limited to external
  services.
