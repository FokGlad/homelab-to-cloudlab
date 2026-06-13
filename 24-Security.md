# Security & SSH Hardening

> How SSH access is locked down across the entire fleet, and why fail2ban
> isn't everywhere.

---

## Principles

- **No public SSH anywhere.** Not on the Edge VPS, not on the Core VPS, not on
  any on-prem host.
- **WireGuard is the only gateway.** If you can't reach the WireGuard tunnel,
  you can't SSH to anything.
- **Key-based authentication only.** Password authentication is disabled fleet-wide.
- **Root login is disabled.** All access goes through a standard user, then
  privilege escalation via `sudo`.

---

## SSH Hardening (All Hosts)

Applied to every machine in the fleet — both VPS instances and on-prem hosts:

| Setting | Value | Why |
|---------|-------|-----|
| **Port** | Non-default (not 22) | Reduces noise from automated scanners |
| **Authentication** | Key-only | Passwords are not secrets keys can't be |
| **Root login** | `PermitRootLogin no` | Forces named-user accountability |
| **Password auth** | `PasswordAuthentication no` | Eliminates brute-force vector |
| **Access** | WireGuard tunnel only | No public SSH port on any host |

The non-default SSH port is a **noise reduction** measure, not a security
boundary. The real boundary is WireGuard: even if you guess the port, you can't
reach it without being on the VPN.

---

## Fail2Ban Strategy

### Core VPS — No Fail2Ban

The Core VPS has **no public-facing interfaces at all**. SSH is only reachable
through the WireGuard tunnel, which already requires key-based authentication.
There is nothing for fail2ban to protect — the attack surface doesn't exist.

### Edge VPS — Fail2Ban as Rate Limiter

The Edge VPS is public-facing for web services (HTTP/HTTPS), but SSH is still
WireGuard-only. Fail2Ban here serves a different purpose:

- **Rate limiting** on public HTTP/HTTPS endpoints (Caddy)
- Protecting against abuse of exposed web services
- Not protecting SSH (which isn't public)

This is a deliberate design choice: the Edge VPS doesn't need fail2ban for SSH
because SSH was never exposed in the first place.

---

## Ansible Automation

The SSH hardening steps above are **not manual**. They're applied via the
shared Ansible bootstrap playbook that runs on both VPS instances (and can be
applied to any new host):

1. Create admin user
2. Deploy SSH public key
3. Apply SSH configuration (port, key-only, no root)
4. Install and configure WireGuard
5. Configure UFW (deny all inbound except WireGuard + required public ports)

See [21-Automation.md](21-Automation.md) for the full automation workflow, and
[25-Disaster-Recovery.md](25-Disaster-Recovery.md) for rebuild procedures.
