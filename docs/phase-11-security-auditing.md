# Phase 11 — Security Auditing

> Phase series: [Phase 1](phase-01-configure-dc01.md) · [Phase 2](phase-02-install-ad-ds-dns-dc01.md) · [Phase 3](phase-03-configure-ad-dns.md) · [Phase 4](phase-04-configure-dc02.md) · [Phase 5](phase-05-ad-site-topology.md) · [Phase 6](phase-06-organizational-units.md) · [Phase 7](phase-07-users-groups-delegation.md) · [Phase 8](phase-08-join-client01.md) · [Phase 9](phase-09-gpo-security-baseline.md) · [Phase 10](phase-10-windows-laps.md) · [Phase 11](phase-11-security-auditing.md) · [Phase 12](phase-12-backup-recovery.md) · [Phase 13](phase-13-health-checks.md)
> 
## Objective & Scope

Enable advanced security auditing on the domain controllers: an auditing GPO with success and
failure auditing across key categories, forced subcategory precedence, and defined key security
event IDs to monitor.

**Scope:** `Sec-DomainControllers-Auditing` GPO, Advanced Audit Policy Configuration, and event
monitoring workflow (Event Viewer in the lab; SIEM forwarding in production).

## Prerequisites

- [Phase 9](phase-09-gpo-security-baseline.md) completed — the GPO design includes
  `Sec-DomainControllers-Auditing`.

## Step-by-Step Execution

### 1. Create Auditing GPO

1. Open **Group Policy Management**.
2. Right-click the **Domain Controllers** OU.
3. Create and link: `Sec-DomainControllers-Auditing`
4. Edit the GPO.
5. Browse to:

```text
Computer Configuration
└── Policies
    └── Windows Settings
        └── Security Settings
            └── Advanced Audit Policy Configuration
```

Enable **Success and Failure** auditing for:

- Account Logon.
- Account Management.
- Directory Service Access.
- Logon/Logoff.
- Policy Change.
- Privilege Use.
- System.

### 2. Force Advanced Audit Policy

Enable:

```text
Computer Configuration
└── Policies
    └── Windows Settings
        └── Security Settings
            └── Local Policies
                └── Security Options
```

Set:

> **Audit: Force audit policy subcategory settings to override audit policy category settings = Enabled**

This setting is required so that the granular subcategory settings in the Advanced Audit Policy
Configuration take precedence over the legacy nine-category audit policy — otherwise category
settings can mask the fine-grained subcategory configuration.

The source guide specifies auditing at the category level (the seven categories listed above).
When configuring subcategories directly, the following matrix maps the categories to their key
subcategories and the events they generate (recommended configuration — verify against your
requirements and Microsoft's per-event documentation):

| Category | Key subcategories | Success/Failure | Representative events |
|---|---|---|---|
| Account Logon | Credential Validation; Kerberos Authentication Service; Kerberos Service Ticket Operations | Success and Failure | 4776, 4771, 4769 |
| Account Management | User Account Management; Security Group Management; Computer Account Management | Success and Failure | 4720, 4740, 4728/4732/4756 |
| Directory Service Access | Directory Service Access; Directory Service Changes | Success and Failure | 4662, 5136/5137 |
| Logon/Logoff | Logon; Logoff; Special Logon | Success and Failure | 4624/4625, 4634, 4672 |
| Policy Change | Audit Policy Change; Authentication Policy Change | Success and Failure | 4719, 4713 |
| Privilege Use | Sensitive Privilege Use | Success and Failure | 4673 |
| System | Security State Change; Security System Extension | Success and Failure | 4608/4609, 4697 |

> **Note on Event 1102 (Security log cleared):** this event is always written when the Security
> log is cleared, regardless of the audit policy configuration — it does not depend on the
> System subcategories above and should be monitored unconditionally.

> **Note on Event 5136 (Directory Service Changes):** the Directory Service Changes subcategory
> additionally requires SACL configuration on the objects to be audited — enabling the
> subcategory alone does not generate 5136 events for objects without a matching SACL.

Validate the resulting policy from an elevated command prompt:

```bat
auditpol /get /category:*
```

Confirm each subcategory reports the intended Success/Failure settings on the domain
controllers.

### 3. Key Event IDs

| Event ID | Security relevance |
|---|---|
| 4625 | Failed logon attempt |
| 4740 | Account lockout |
| 4720 | User account created |
| 4728 / 4732 / 4756 | Member added to a security group |
| 4769 | Kerberos service ticket request |
| 5136 | Active Directory object changed |
| 1102 | Security audit log cleared |

For a lab, investigate logs in **Event Viewer**. In a production environment, send DC security
logs to a centralized SIEM such as Splunk or Microsoft Sentinel.

## Validation

- `gpresult /r` on DC01/DC02 shows `Sec-DomainControllers-Auditing` applied.
- `auditpol /get /category:*` on the DCs reports the intended Success/Failure settings.
- Failed logon attempts generate Event ID 4625 in the DC Security log.
- Creating a test user generates Event ID 4720.
- The "Force audit policy subcategory settings" Security Option shows **Enabled**.

## Visual Evidence

| # | Caption | Asset path |
|---|---|---|
| See screenshots | | docs/screenshots/Phase 11 |

Full mapping: [Evidence Index](evidence-index.md#phase-11--security-auditing).

## References

- [Microsoft Learn — AD DS deployment](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/deploy/ad-ds-deployment)
- [Microsoft Learn — Deploy and manage AD DS domain controllers](https://learn.microsoft.com/en-us/training/modules/deploy-manage-active-directory-domain-services-domain-controllers/)

Next: [Phase 12 — Backup and Recovery](phase-12-backup-recovery.md)
