# Phase 7 — User Accounts, Groups, and Delegation

> Phase series: [Phase 1](phase-01-configure-dc01.md) · [Phase 2](phase-02-install-ad-ds-dns-dc01.md) · [Phase 3](phase-03-configure-ad-dns.md) · [Phase 4](phase-04-configure-dc02.md) · [Phase 5](phase-05-ad-site-topology.md) · [Phase 6](phase-06-organizational-units.md) · [Phase 7](phase-07-users-groups-delegation.md) · [Phase 8](phase-08-join-client01.md) · [Phase 9](phase-09-gpo-security-baseline.md) · [Phase 10](phase-10-windows-laps.md) · [Phase 11](phase-11-security-auditing.md) · [Phase 12](phase-12-backup-recovery.md) · [Phase 13](phase-13-health-checks.md)
> Source: Active Directory Project Guide (compiled 25-page guide), pages 12–13.

## Objective & Scope

Create standard and privileged user accounts with strict separation, build role-based security
groups, and delegate least-privilege administrative tasks to the helpdesk group.

**Scope:** jdoe (standard), adm_jdoe (privileged), GG_* security groups, delegated password-reset
control for GG_IT_Helpdesk.

## Prerequisites

- [Phase 6](phase-06-organizational-units.md) completed — OU hierarchy exists.
- Domain administrative credentials.

## Step-by-Step Execution

### 1. Create a Standard User Account

In **Active Directory Users and Computers**, create:

| Attribute | Value |
|---|---|
| Name | John Doe |
| User logon name | `jdoe` |
| OU | `Beginningdimension > Users` |

Set a strong temporary password and **require password change at first sign-in**.

### 2. Create a Privileged Account

Create a separate administrator account:

| Attribute | Value |
|---|---|
| Name | John Doe Admin |
| User logon name | `adm_jdoe` |
| OU | `Beginningdimension > Admin > Tier0` |

Usage discipline:

- Use `BEGINNINGDIM\jdoe` for standard daily work.
- Use `BEGINNINGDIM\adm_jdoe` **only** for approved administrative tasks.

### 3. Create Security Groups

Create these Global Security groups inside
`OU=SecurityGroups,OU=Groups,OU=Beginningdimension`:

- `GG_IT_Helpdesk`
- `GG_Tier0_Admins`
- `GG_Tier1_Admins`
- `GG_Tier2_Admins`
- `GG_LAPS_Password_Readers`
- `GG_HR_Users`

Add `adm_jdoe` to `GG_Tier0_Admins`.

### 4. Apply Least Privilege

Avoid adding routine administrator accounts directly to Domain Admins. Use delegation where
possible.

**Example — delegate password reset to helpdesk staff:**

1. Right-click the **Users** OU.
2. Select **Delegate Control**.
3. Add `GG_IT_Helpdesk`.
4. Select:
   - **Reset user passwords.**
   - **Force password change at next logon.**
   - **Read user information.**

## Validation

- `jdoe` and `adm_jdoe` exist in their respective OUs with correct attributes.
- All six `GG_*` security groups exist in the SecurityGroups OU.
- `adm_jdoe` is a member of `GG_Tier0_Admins`.
- A member of `GG_IT_Helpdesk` can reset a user password in the Users OU but holds no other
  administrative rights.

## Visual Evidence

| # | Caption | Asset path |
|---|---|---|
| [37](../assets/screenshots/screenshot-37.png) | jdoe standard user account properties in the Users OU | `assets/screenshots/screenshot-37.png` |
| [38](../assets/screenshots/screenshot-38.png) | adm_jdoe privileged account in the Admin > Tier0 OU | `assets/screenshots/screenshot-38.png` |
| [39](../assets/screenshots/screenshot-39.png) | GG_* security groups created in the SecurityGroups OU | `assets/screenshots/screenshot-39.png` |
| [40](../assets/screenshots/screenshot-40.png) | adm_jdoe membership in GG_Tier0_Admins | `assets/screenshots/screenshot-40.png` |
| [41](../assets/screenshots/screenshot-41.png) | Delegation of Control wizard targeting GG_IT_Helpdesk | `assets/screenshots/screenshot-41.png` |
| [42](../assets/screenshots/screenshot-42.png) | Delegated task summary — password reset permissions | `assets/screenshots/screenshot-42.png` |

Full mapping: [Evidence Index](evidence-index.md#phase-7--user-accounts-groups-and-delegation).

## References

- [Microsoft Learn — AD DS deployment](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/deploy/ad-ds-deployment)
- [Microsoft Learn — Deploy and manage AD DS domain controllers](https://learn.microsoft.com/en-us/training/modules/deploy-manage-active-directory-domain-services-domain-controllers/)

Next: [Phase 8 — Join CLIENT01 to the Domain](phase-08-join-client01.md)
