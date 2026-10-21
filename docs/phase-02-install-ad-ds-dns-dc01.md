# Phase 2 — Install AD DS and DNS on DC01

> Phase series: [Phase 1](phase-01-configure-dc01.md) · [Phase 2](phase-02-install-ad-ds-dns-dc01.md) · [Phase 3](phase-03-configure-ad-dns.md) · [Phase 4](phase-04-configure-dc02.md) · [Phase 5](phase-05-ad-site-topology.md) · [Phase 6](phase-06-organizational-units.md) · [Phase 7](phase-07-users-groups-delegation.md) · [Phase 8](phase-08-join-client01.md) · [Phase 9](phase-09-gpo-security-baseline.md) · [Phase 10](phase-10-windows-laps.md) · [Phase 11](phase-11-security-auditing.md) · [Phase 12](phase-12-backup-recovery.md) · [Phase 13](phase-13-health-checks.md)
> 

## Objective & Scope

Install the Active Directory Domain Services (AD DS) and DNS Server roles on DC01 and promote it
as the first domain controller of a new forest, creating the `ad.beginningdimension.com` domain.

**Scope:** role installation, forest creation, DSRM password configuration, and post-promotion
DNS client settings on DC01.

## Prerequisites

- [Phase 1](phase-01-configure-dc01.md) completed: DC01 renamed, static IP configured,
  connectivity validated, updates installed, time verified.
- A strong DSRM password prepared and securely stored (password vault).

## Step-by-Step Execution

### 1. GUI Installation

1. Open **Server Manager**.
2. Select **Manage > Add Roles and Features**.
3. Select **Role-based or feature-based installation**.
4. Select **DC01**.
5. Under **Server Roles**, select:
   - **Active Directory Domain Services**
   - **DNS Server**
6. Select **Add Features** when prompted.
7. Select **Next** until reaching the confirmation page.
8. Select **Install**.

Microsoft documents Server Manager and the AD DS Configuration Wizard as supported methods for
installing AD DS on Windows Server
([Microsoft Learn — Install AD DS (Level 100)](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/deploy/install-active-directory-domain-services--level-100-)).

### 2. Promote DC01

1. In Server Manager, select the **yellow notification flag**.
2. Select **Promote this server to a domain controller**.
3. Choose **Add a new forest**.
4. Enter: `ad.beginningdimension.com`
5. Select **Next**.
6. On **Domain Controller Options**:
   - Keep **DNS Server** selected.
   - Keep **Global Catalog** selected.
   - Enter and securely store a strong **DSRM password**.
7. Continue through the wizard.
8. Confirm the NetBIOS name: `BEGINNINGDIM`
9. Keep the default AD database, log, and SYSVOL locations unless a dedicated storage design exists.
10. Complete prerequisites validation.
11. Select **Install**.

DC01 restarts automatically.

> **Note — NetBIOS 15-character limit:** NetBIOS computer and domain names can only be 15
> characters long, so the forest's NetBIOS name is `BEGINNINGDIM` rather than the full
> `BEGINNINGDIMENSION`. The DNS domain name `ad.beginningdimension.com` is unaffected by this
> limit ([Microsoft Learn — AD DS Maximum Limits](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/active-directory-domain-services-maximum-limits)).

### 3. Configure DC01 DNS After Promotion

After restart, sign in as: `BEGINNINGDIM\Administrator`

Update the IPv4 DNS settings:

| Setting | Value |
|---|---|
| Preferred DNS | `192.168.10.10` |
| Alternate DNS | `127.0.0.1` |

## Validation

- DC01 restarts automatically after promotion and signs in with domain credentials.
- The `ad.beginningdimension.com` zone is created automatically during promotion.
- `BEGINNINGDIM` appears as the NetBIOS domain name at logon.

DNS zone configuration and validation are covered in [Phase 3](phase-03-configure-ad-dns.md).

## Visual Evidence

| # | Caption | Asset path |
|---|---|---|
See the screenshots in the screenshot folder. Path. docs/screenshots/Phase 2
Full mapping: [Evidence Index](evidence-index.md#phase-2--install-ad-ds-and-dns-on-dc01).

## References

- [Microsoft Learn — Install Active Directory Domain Services (Level 100)](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/deploy/install-active-directory-domain-services--level-100-)
- [Microsoft Learn — What's new in AD DS installation and removal](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/deploy/what-s-new-in-active-directory-domain-services-installation-and-removal)
- [Microsoft Learn — AD DS deployment](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/deploy/ad-ds-deployment)
- [Microsoft Learn — AD DS Maximum Limits](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/active-directory-domain-services-maximum-limits)
- [Microsoft Learn — Deploy and manage AD DS domain controllers](https://learn.microsoft.com/en-us/training/modules/deploy-manage-active-directory-domain-services-domain-controllers/)

Next: [Phase 3 — Configure AD DNS](phase-03-configure-ad-dns.md)
