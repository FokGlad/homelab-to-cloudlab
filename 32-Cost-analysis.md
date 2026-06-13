# Cost Analysis

> The honest numbers: what the hybrid transition actually costs, and what I'd do
> differently with unlimited budget.

---

## Electricity

Before the hybrid transition, the full rack ran 24/7:

```
Always-on rack: ~20-25 EUR/month
```

With the homelab on a night schedule (shut down ~8-10 hours per day):

```
Night-shutdown rack: ~15 EUR/month
```

Savings: roughly **5-10 EUR/month** from reduced uptime.

---

## VPS Costs

| Instance | Specs | Monthly Cost |
|----------|-------|-------------|
| **Core VPS** (IONOS) | 6 vCPU / 8 GB / 240 GB | 6 EUR |
| **Edge VPS** (IONOS) | 2 vCPU / 2 GB / 80 GB | 3 EUR |
| **Total** | | **9 EUR/month** |

Contract: 2 years.

---

## The Honest Summary

```
Before:  ~20-25 EUR/month (electricity only)
Now:     ~15 EUR/month (electricity) + 9 EUR/month (VPS) = ~24 EUR/month
```

The total monthly cost is roughly **the same**. The hybrid transition did not
reduce my costs.

---

## What Would Actually Have Saved Money

The energy draw comes from the hardware itself — an Intel i5-10600K, a discrete
GPU in the Proxmox node, an old TrueNAS box with spinning disks. The real fix
would be **modern, efficient hardware**:

- Low-power CPU (Intel T-series, ARM, or similar)
- 80+ Platinum/Titanium PSUs instead of older units
- Fewer spinning disks, more flash storage

I can't afford to replace all that hardware right now. The VPS transition was
the budget-friendly alternative: rent efficient hardware instead of
buying it.

---

## Where the Real Value Is

Even at the same monthly cost, the hybrid approach delivered:

- **Better uptime** for services that need to be always-on (not dependent on
  home power or residential ISP)
- **Stronger security boundaries** (home network is never exposed)
- **Flexibility** (VPS instances rebuilt from an Ansible playbook in minutes)
- **Learning** (FreeIPA, WireGuard daisy-chain, multi-zone DNS, Ansible —
  all things I wouldn't have touched with an all-on-prem setup)

The cost optimization story would look very different with efficient hardware.
For now, the value is in flexibility and capability, not savings.
