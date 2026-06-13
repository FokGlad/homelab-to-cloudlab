# Network Architecture

> VLAN-separated network with dedicated segments for each trust level.

---

## VLAN Layout

| VLAN | Name | Purpose |
|------|------|---------|
| **Internal** | `int` | Internal services + internal Caddy instance (`int.domain.ltd`) |
| **External** | `ext` | External services + external Caddy instance (`ext.domain.ltd`) |
| **Infra** | `infra` | Structurally important hosts/VMs (Gitea, Postfix relay, Vault, Portainer) |
| **VoIP** | `voip` | FreePBX and VoIP devices |
| **IoT** | `iot` | Wireless IoT devices (isolated) |
| **Local** | `local` | PCs, media center, game consoles |
| **Management** | `mgmt` | Desktop PC — unrestricted access to all VLANs |
| **WireGuard VPN** | `wg-vpn` | VPS daisy-chain termination subnet |
| **WireGuard Admin** | `wg-admin` | Remote admin access tunnel |
| **WireGuest** | `wg-guest` | Guest remote access (limited: certain external services only) |

---

## VLAN Details

### Internal VLAN
- Hosts internal-only services (dashboards, internal APIs, admin panels)
- Runs the **internal Caddy instance** for `int.domain.ltd`
- Not routable from the External VLAN or the internet
- Accessible remotely via **WireGuard Admin tunnel** only

### External VLAN
- Hosts on-prem services that need external access (`ext.domain.ltd`)
- Runs the **external Caddy instance**
- Remotely accessible via a **dedicated VPN tunnel** (separate from admin)
- Isolated from Internal VLAN — no cross-VLAN routing by default

### Infra VLAN
- Dedicated to **structurally critical** infrastructure:
  - **Portainer** — Docker management
  - **Postfix relay** — local mail relay to Edge VPS
  - **PVE-exporter** — Proxmox metrics for Prometheus
  - **Vault** — secrets management (WIP)
  - **Gitea Runner** — CI/CD pipeline runner
- Highest security segment — only admin access

### VoIP VLAN
- Dedicated to **FreePBX** and VoIP devices
- Isolated to prevent crosstalk with data traffic
- QoS prioritized for voice traffic at the switch/firewall level

### IoT VLAN
- Wireless-only IoT devices (smart home, sensors, etc.)
- Fully isolated — can only reach the internet, not other VLANs
- No access to internal services

### Local VLAN
- General-purpose: PCs, media center, game consoles
- Standard internet access
- Can reach Internal and External VLAN services as needed

### Management VLAN
- Dedicated to the **desktop PC** used for day-to-day management
- Unrestricted access to all VLANs — can reach Infra, Internal, External,
  everything
- This is the "everything is allowed" subnet
- **Future improvement:** move the desktop to a regular subnet and restrict
  the Management VLAN to specific admin hosts only

### WireGuard VPN VLAN
- Dedicated subnet for the **VPS daisy-chain termination**
- Core VPS and Edge VPS terminate their WireGuard interfaces here
- Routes traffic between VPS tiers and on-prem

### WireGuard Admin VLAN
- Remote admin access tunnel
- Full access to Internal VLAN and Infra VLAN
- Used for management when away from home

### WireGuard Guest VLAN
- Guest remote access tunnel
- **Restricted**: can only reach select services on the External VLAN
- No access to Internal, Infra, or Local VLANs

---

## Firewall Rules (OPNsense)

OPNsense (bare metal) handles all inter-VLAN routing and filtering:

```
Internet
    │
    ▼
┌──────────────────────────────────────────┐
│              OPNsense                    │
│         (Firewall + Gateway)             │
│                                          │
│  VLANs:                                  │
│  ├── Internal  ←──→ Admin WG (allow)     │
│  ├── External  ←──→ Ext VPN tunnel       │
│  ├── Infra     ←──→ Admin WG only        │
│  ├── VoIP      ───→ Internet only        │
│  ├── IoT       ───→ Internet only        │
│  ├── Local     ←──→ Internal, External   │
│  ├── Mgmt      ←──→ Everything           │
│  ├── WG-VPN    ←──→ Core/Edge VPS        │
│  ├── WG-Admin  ←──→ Internal, Infra      │
│  └── WG-Guest  ───→ External (limited)   │
└──────────────────────────────────────────┘
```

Key principles:
- **Default deny** between VLANs
- Only explicitly allowed cross-VLAN traffic flows
- IoT can only reach the internet (no lateral movement)
- Guest VPN gets the most restricted access profile

---

## Switch — Cisco SG-300

The Cisco SG-300 managed switch handles VLAN tagging and trunking:

- **Trunk port** to OPNsense (carries all VLANs)
- **Access ports** assigned per VLAN to physical devices
- **Trunk port** to Proxmox (for VM/CT VLAN assignment)
- VLAN-aware DHCP relay to OPNsense (each VLAN gets its own subnet via
  OPNsense DHCP)
