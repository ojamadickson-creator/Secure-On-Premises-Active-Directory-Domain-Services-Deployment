# Phase 9 — Group Policy Security Baseline

> Phase series: [Phase 1](phase-01-configure-dc01.md) · [Phase 2](phase-02-install-ad-ds-dns-dc01.md) · [Phase 3](phase-03-configure-ad-dns.md) · [Phase 4](phase-04-configure-dc02.md) · [Phase 5](phase-05-ad-site-topology.md) · [Phase 6](phase-06-organizational-units.md) · [Phase 7](phase-07-users-groups-delegation.md) · [Phase 8](phase-08-join-client01.md) · [Phase 9](phase-09-gpo-security-baseline.md) · [Phase 10](phase-10-windows-laps.md) · [Phase 11](phase-11-security-auditing.md) · [Phase 12](phase-12-backup-recovery.md) · [Phase 13](phase-13-health-checks.md)
> Source: Active Directory Project Guide (compiled 25-page guide), pages 15–16, plus the
> Workstation Baseline Group Policy Paths reference (GPME configuration paths).

## Objective & Scope

Design and deploy the Group Policy security baseline: domain-wide password and lockout policy,
domain-controller hardening and auditing controls, and a workstation security baseline, with the
Windows LAPS and update-management GPOs created here and detailed in later phases.

**Scope:** GPO design, Default Domain Policy password/lockout configuration, workstation baseline
settings, and pilot-first deployment practice.

## Prerequisites

- [Phase 8](phase-08-join-client01.md) completed — CLIENT01 is domain-joined for validation.
- Group Policy Management console available on a domain controller.

## Group Policy Design

| GPO | Linked to | Objective |
|---|---|---|
| `Sec-Domain-PasswordPolicy` | Domain root | Password, lockout, and Kerberos policy |
| `Sec-DomainControllers-Baseline` | Domain Controllers OU | Security and DC controls |
| `Sec-DomainControllers-Auditing` | Domain Controllers OU | Advanced auditing |
| `Sec-Workstation-Baseline` | Workstations OU | Firewall, Defender, workstation controls |
| `Cfg-WindowsLAPS` | Workstations OU | LAPS configuration |
| `Cfg-WindowsUpdate` | Workstations and servers | Update-management configuration |

> **Execution note:** the documented procedure applies the domain password and lockout policy by
> editing the **Default Domain Policy** (the domain's effective password policy), while
> `Sec-Domain-PasswordPolicy` in the table above represents the design intent of a dedicated
> named GPO. Both approaches are valid; the source guide's documented execution edits Default
> Domain Policy.

> **Test new policies on the Pilot OU before linking them to production OUs.**

## Step-by-Step Execution

### 1. Domain Password and Lockout Policy

1. Open **Group Policy Management**.
2. Right-click **Default Domain Policy > Edit**.
3. Browse to:

```text
Computer Configuration
└── Policies
    └── Windows Settings
        └── Security Settings
            └── Account Policies
```

4. Configure:

| Policy | Recommended value |
|---|---|
| Enforce password history | 24 passwords |
| Maximum password age | 90 days |
| Minimum password age | 1 day |
| Minimum password length | 14 characters |
| Password complexity | Enabled |
| Account lockout threshold | 10 failed attempts |
| Account lockout duration | 15 minutes |
| Reset account lockout counter | 15 minutes |

### 2. Workstation Security Baseline

Create and link `Sec-Workstation-Baseline` to the **Workstations** OU. Initially configure:

- Windows Defender Antivirus enabled.
- Windows Firewall enabled for Domain, Private, and Public profiles.
- Guest account disabled.
- Automatic screen lock required.
- Windows Update configuration enabled.
- Local administrator use restricted.
- Baseline audit policy enabled.

GPME configuration paths for these controls (from the lab's Workstation Baseline Group Policy
Paths reference):

| Control | GPME path | What to configure |
|---|---|---|
| Defender Antivirus | Computer Configuration > Policies > Administrative Templates > Windows Components > Microsoft Defender Antivirus | Ensure "Turn off Microsoft Defender Antivirus" is Disabled (or left unconfigured if enabled by default); configure real-time protection options as needed |
| Firewall profiles | Computer Configuration > Policies > Windows Settings > Security Settings > Windows Defender Firewall with Advanced Security > Windows Defender Firewall with Advanced Security | Right-click the root node or properties; set firewall state to On for Domain, Private, and Public profiles |
| Guest account | Computer Configuration > Policies > Windows Settings > Security Settings > Local Policies > Security Options | "Accounts: Guest account status" = Disabled |
| Inactivity lock | Computer Configuration > Policies > Windows Settings > Security Settings > Local Policies > Security Options | "Interactive logon: Machine inactivity limit" = 900 seconds (15 minutes) |
| Screen saver protection | Computer Configuration > Policies > Administrative Templates > Control Panel > Personalization | Enable "Password protect the screen saver" and configure the timeout |
| Windows Update | Computer Configuration > Policies > Administrative Templates > Windows Components > Windows Update > Manage end user experience | "Configure Automatic Updates" — manage how and when updates are downloaded and installed (recommended state: auto-download & notify) |
| Built-in Administrator | Computer Configuration > Policies > Windows Settings > Security Settings > Local Policies > Security Options | "Accounts: Administrator account status" = Disabled (LAPS manages local credentials) |
| User Account Control | Computer Configuration > Policies > Windows Settings > Security Settings > Local Policies > Security Options | "User Account Control: Run all administrators in Admin Approval Mode" = Enabled |
| Baseline audit policy | Computer Configuration > Policies > Windows Settings > Security Settings > Advanced Audit Policy Configuration > Audit Policies | Drill into subcategories (Logon/Logoff, Account Management, Privilege Use) and configure Success and Failure auditing |

**Apply to CLIENT01 first, validate, then expand deployment.**

## Validation

- `gpresult /r` on CLIENT01 shows `Sec-Workstation-Baseline` applied.
- A test account locks out after 10 failed attempts and unlocks after 15 minutes.
- `net accounts` on a domain member reflects the configured password and lockout policy.
- Windows Defender and Firewall settings on CLIENT01 match the baseline.

> **Incident note:** the first application of `Sec-Workstation-Baseline` on CLIENT01 failed with
> `gpupdate /force` fatal errors caused by Kerberos clock skew — see
> [Incident 01](troubleshooting-log.md#incident-01--kerberos-time-sync-failure-breaking-gpo-application).

## Visual Evidence

| # | Caption | Asset path |
|---|---|---|
| [48](../assets/screenshots/screenshot-48.png) | Group Policy Management showing the six-GPO baseline design | `assets/screenshots/screenshot-48.png` |
| [49](../assets/screenshots/screenshot-49.png) | Default Domain Policy — password policy settings | `assets/screenshots/screenshot-49.png` |
| [50](../assets/screenshots/screenshot-50.png) | Account lockout policy settings in Default Domain Policy | `assets/screenshots/screenshot-50.png` |
| [51](../assets/screenshots/screenshot-51.png) | Sec-Workstation-Baseline — Defender and Firewall settings | `assets/screenshots/screenshot-51.png` |
| [52](../assets/screenshots/screenshot-52.png) | Sec-DomainControllers-Baseline linked to the Domain Controllers OU | `assets/screenshots/screenshot-52.png` |
| [53](../assets/screenshots/screenshot-53.png) | `gpresult /r` on CLIENT01 showing applied baseline GPOs | `assets/screenshots/screenshot-53.png` |

Full mapping: [Evidence Index](evidence-index.md#phase-9--group-policy-security-baseline).

## References

- [Microsoft Learn — Deploy and manage AD DS domain controllers](https://learn.microsoft.com/en-us/training/modules/deploy-manage-active-directory-domain-services-domain-controllers/)
- [Microsoft Learn — AD DS deployment](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/deploy/ad-ds-deployment)

Next: [Phase 10 — Deploy Windows LAPS](phase-10-windows-laps.md)
