# Phase 6 — Create Organizational Units

> Phase series: [Phase 1](phase-01-configure-dc01.md) · [Phase 2](phase-02-install-ad-ds-dns-dc01.md) · [Phase 3](phase-03-configure-ad-dns.md) · [Phase 4](phase-04-configure-dc02.md) · [Phase 5](phase-05-ad-site-topology.md) · [Phase 6](phase-06-organizational-units.md) · [Phase 7](phase-07-users-groups-delegation.md) · [Phase 8](phase-08-join-client01.md) · [Phase 9](phase-09-gpo-security-baseline.md) · [Phase 10](phase-10-windows-laps.md) · [Phase 11](phase-11-security-auditing.md) · [Phase 12](phase-12-backup-recovery.md) · [Phase 13](phase-13-health-checks.md)
> 

## Objective & Scope

Build the structured Organizational Unit (OU) hierarchy that underpins Group Policy targeting and
delegated administration, then redirect default user and computer object locations into the new
structure.

**Scope:** OU tree design and creation, `redircmp`/`redirusr` default-location redirection.

## Prerequisites

- [Phase 5](phase-05-ad-site-topology.md) completed.
- Domain administrative credentials.

## Recommended OU Design

```text
ad.beginningdimension.com
│
├── OU=Beginningdimension
│   ├── OU=Users
│   ├── OU=Computers
│   │   ├── OU=Workstations
│   │   └── OU=Laptops
│   ├── OU=Servers
│   │   ├── OU=MemberServers
│   │   └── OU=ApplicationServers
│   ├── OU=Groups
│   │   ├── OU=SecurityGroups
│   │   └── OU=DistributionGroups
│   ├── OU=ServiceAccounts
│   ├── OU=Admin
│   │   ├── OU=Tier0
│   │   ├── OU=Tier1
│   │   └── OU=Tier2
│   └── OU=Pilot
```

This tiered structure separates standard users, workstations, servers, groups, service accounts,
privileged administrative accounts (Tier0/1/2), and a Pilot OU for policy testing.

## Step-by-Step Execution

### 1. Create OUs in GUI

1. Open **Active Directory Users and Computers**.
2. Right-click `ad.beginningdimension.com`.
3. Select **New > Organizational Unit**.
4. Create the top-level OU: `Beginningdimension`
5. Create the remaining OUs using the structure above.

### 2. Redirect Default User and Computer Locations

Run on DC01 as Administrator:

```bat
redircmp "OU=Workstations,OU=Computers,OU=Beginningdimension,DC=ad,DC=beginningdimension,DC=com"
redirusr "OU=Users,OU=Beginningdimension,DC=ad,DC=beginningdimension,DC=com"
```

This directs newly joined computers to the **Workstations** OU and newly created users to the
**Users** OU instead of the default `Computers` and `Users` containers — a prerequisite for
cleanly targeting GPOs at new objects.

## Validation

- The OU tree in ADUC matches the recommended design.
- A newly created user lands in `Beginningdimension > Users`.
- A newly joined computer lands in `Beginningdimension > Computers > Workstations`.

## Visual Evidence

| # | Caption | Asset path |
|---|---|---|
See screenshots| | docs/screenshots/Phase 6 |

Full mapping: [Evidence Index](evidence-index.md#phase-6--create-organizational-units).

## References

- [Microsoft Learn — Deploy and manage AD DS domain controllers](https://learn.microsoft.com/en-us/training/modules/deploy-manage-active-directory-domain-services-domain-controllers/)
- [Microsoft Learn — AD DS deployment](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/deploy/ad-ds-deployment)

Next: [Phase 7 — User Accounts, Groups, and Delegation](phase-07-users-groups-delegation.md)
