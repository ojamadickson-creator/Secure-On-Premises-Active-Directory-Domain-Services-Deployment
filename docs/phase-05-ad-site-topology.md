# Phase 5 — Configure AD Site Topology

> Phase series: [Phase 1](phase-01-configure-dc01.md) · [Phase 2](phase-02-install-ad-ds-dns-dc01.md) · [Phase 3](phase-03-configure-ad-dns.md) · [Phase 4](phase-04-configure-dc02.md) · [Phase 5](phase-05-ad-site-topology.md) · [Phase 6](phase-06-organizational-units.md) · [Phase 7](phase-07-users-groups-delegation.md) · [Phase 8](phase-08-join-client01.md) · [Phase 9](phase-09-gpo-security-baseline.md) · [Phase 10](phase-10-windows-laps.md) · [Phase 11](phase-11-security-auditing.md) · [Phase 12](phase-12-backup-recovery.md) · [Phase 13](phase-13-health-checks.md)
> 
## Objective & Scope

Model the physical network in Active Directory by creating the HQ site, associating the
`192.168.10.0/24` subnet, moving both domain controllers into the site, and validating site
awareness on all systems.

**Scope:** AD Sites and Services configuration — site, subnet, DC placement, and site-awareness
validation. 

## Topology Plan

This project has one fast, reliable LAN, so it uses one AD site.

| Configuration item | Value |
|---|---|
| Site name | `HQ` |
| Subnet | `192.168.10.0/24` |
| Domain controllers | DC01 and DC02 |
| Workstation | CLIENT01 |
| Intersite link | Not required for one-site design |
| Default site link | Leave unchanged |

AD sites are used to represent physical-network connectivity, map IP subnets to locations,
optimize DC discovery, and guide replication traffic
([Microsoft Learn — Designing the Site Topology](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/designing-the-site-topology)).

## Step-by-Step Execution

### 1. Create the HQ Site

1. On DC01, open **Server Manager > Tools > Active Directory Sites and Services**.
2. Expand **Sites**.
3. Right-click **Sites > New Site**.
4. Enter: `HQ`
5. Select: `DEFAULTIPSITELINK`
6. Select **OK**.

### 2. Create and Associate the Subnet

1. Right-click **Subnets**.
2. Select **New Subnet**.
3. Enter: `192.168.10.0/24`
4. Associate it with: `HQ`
5. Select **OK**.

### 3. Move Domain Controllers

1. Expand:

```text
Sites
└── Default-First-Site-Name
    └── Servers
```

2. Right-click **DC01 > Move > select HQ**.
3. Repeat for **DC02**.
4. Confirm both domain controllers appear here:

```text
Sites
└── HQ
    └── Servers
        ├── DC01
        └── DC02
```

> **Do not delete `Default-First-Site-Name`** until confirming that no domain controllers or
> subnets remain associated with it.

### 4. Validate Site Awareness

Run this on DC01, DC02, and CLIENT01:

```bat
nltest /dsgetsite
```

**Expected result:** `HQ`

The Knowledge Consistency Checker (KCC) normally creates and maintains domain-controller
replication connection objects; administrators should define sites, subnets, and site-link policy
rather than manually creating replication links in a simple deployment
([Microsoft Learn — Active Directory Site Replication](https://learn.microsoft.com/en-us/training/modules/active-directory-site-replication/)).


## Validation

- The `HQ` site exists in Active Directory Sites and Services.
- The `192.168.10.0/24` subnet is mapped to HQ.
- DC01 and DC02 are located in the HQ site.
- `nltest /dsgetsite` returns `HQ` on DC01, DC02, and CLIENT01.

## Visual Evidence

| # | Caption | Asset path |
|---|---|---|
See the screenshots | | docs/screenshots/Phase 5 |

Full mapping: [Evidence Index](evidence-index.md#phase-5--configure-ad-site-topology).

## References

- [Microsoft Learn — Designing the Site Topology](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/designing-the-site-topology)
- [Microsoft Learn — Understanding Active Directory Site Topology](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/understanding-active-directory-site-topology)
- [Microsoft Learn — Creating a Site Design](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/creating-a-site-design)
- [Microsoft Learn — Active Directory Site Replication (training)](https://learn.microsoft.com/en-us/training/modules/active-directory-site-replication/)

Next: [Phase 6 — Create Organizational Units](phase-06-organizational-units.md)
