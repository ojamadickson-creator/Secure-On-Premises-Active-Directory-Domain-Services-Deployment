# Phase 10 — Deploy Windows LAPS

> Phase series: [Phase 1](phase-01-configure-dc01.md) · [Phase 2](phase-02-install-ad-ds-dns-dc01.md) · [Phase 3](phase-03-configure-ad-dns.md) · [Phase 4](phase-04-configure-dc02.md) · [Phase 5](phase-05-ad-site-topology.md) · [Phase 6](phase-06-organizational-units.md) · [Phase 7](phase-07-users-groups-delegation.md) · [Phase 8](phase-08-join-client01.md) · [Phase 9](phase-09-gpo-security-baseline.md) · [Phase 10](phase-10-windows-laps.md) · [Phase 11](phase-11-security-auditing.md) · [Phase 12](phase-12-backup-recovery.md) · [Phase 13](phase-13-health-checks.md)
> Source: Active Directory Project Guide (compiled 25-page guide), pages 16–17, plus the Phase 10
> Pre-Deployment Execution Guide (LAPS forest-level change record & backup procedure).

## Objective & Scope

Deploy Windows LAPS to manage and rotate unique local administrator passwords on domain-joined
systems: extend the AD schema, grant computer self-update permissions, delegate password-read
access, configure the LAPS GPO, and validate password backup and retrieval.

**Scope:** forest schema extension, OU permissions, `Cfg-WindowsLAPS` GPO, and CLIENT01
validation.

## Prerequisites

- [Phase 9](phase-09-gpo-security-baseline.md) completed — the `Cfg-WindowsLAPS` GPO design
  exists and the Workstations OU is in place.
- The pre-deployment requirements below completed: a formal change management record and a
  point-in-time backup of DC01.
- Credentials with Schema Admins membership for the schema update.

## Pre-Deployment: Change Record & Point-in-Time Backup

Two mandatory pre-deployment requirements are completed prior to executing the schema update:
registering a formal change management record and securing a point-in-time backup.

### 1. Forest-Level Change Documentation Record

Modifying the Active Directory schema (via `Update-LapsADSchema`) permanently adds the new
attribute definitions across every domain controller in the forest. The change is recorded as
follows:

| Change log entry | Value |
|---|---|
| Change title | Active Directory Schema Extension for Windows LAPS |
| Change scope | Forest-wide (`ad.beginningdimension.com`) |
| Target domain controller | DC01 (Schema Master / PDC Emulator) |
| Execution date/time | October 20, 2026 |
| Executed by | `BEGINNINGDIM\Administrator` |
| Command / tool | PowerShell cmdlet: `Update-LapsADSchema` |
| Impact & purpose | Adds schema attributes necessary for managing and encrypting unique local administrator passwords across domain-joined endpoints |
| Rollback strategy | System State restore from dedicated backup / VM appliance import. AD schema modifications cannot be undone; recovery requires restoring Active Directory to a pre-change state |

### 2. VirtualBox Export Appliance Backup

Fastest hypervisor-level backup method to secure a complete point-in-time image of DC01 directly
to an external host drive:

1. **Shut down DC01:** power off the DC01 virtual machine completely.
2. **Open the Appliance Export wizard:** in VirtualBox Manager, navigate to
   **File > Export Appliance...**
3. **Select the target VM:** select DC01 from the virtual machine list and click Next.
4. **Specify the backup destination:** on the Format Settings page, click the folder icon next to
   the file path, navigate to the external hard drive, and assign a filename
   (e.g., `DC01_Pre_LAPS_Backup.ova`).
5. **Complete the export:** click through the remaining wizard options and click Finish. Wait
   for the export progress bar to reach 100%.

> **Recovery note:** an exported appliance (`.ova`) acts as an exact VM clone. If an issue occurs
> during LAPS implementation, re-importing the appliance restores DC01 to its pre-change state.

Windows LAPS is a key security control because it manages and rotates unique local administrator
passwords on domain-joined systems. Microsoft requires a one-time AD schema update, permission
for managed devices to update password data, permission for authorized users to read passwords,
and a policy targeting managed devices
([Microsoft Learn — Get started with Windows LAPS and Windows Server AD](https://learn.microsoft.com/en-us/windows-server/identity/laps/laps-scenarios-windows-server-active-directory)).
The schema update permanently adds attribute definitions (`msLAPS-Password`,
`msLAPS-PasswordExpirationTime`) across every domain controller in the forest.

**Encryption prerequisite:** LAPS password encryption requires a domain functional level of
Windows Server 2016 or later — verify the DFL before relying on encrypted LAPS passwords. Also
confirm the intended managed local account (the built-in `Administrator` account by default)
exists on the target systems.

## Step-by-Step Execution

### 1. Extend the AD Schema

On DC01, open **PowerShell as Administrator**:

```powershell
Update-LapsADSchema
```

This is the forest-level change documented in the pre-deployment record above — the schema
attributes cannot be removed once added, which is why the point-in-time backup is mandatory
before running it.

The schema update is a one-time operation for the entire forest. It may prompt for confirmation —
review each prompt and confirm to proceed. See the
[supplementary operational notes](troubleshooting-log.md#supplementary-operational-notes) for
prompt-handling guidance.

### 2. Grant Computer Self-Permission

```powershell
Set-LapsADComputerSelfPermission `
  -Identity "OU=Workstations,OU=Computers,OU=Beginningdimension,DC=ad,DC=beginningdimension,DC=com"
```

This sets inheritable permissions on the Workstations OU so each computer object can update its
own LAPS password attributes.

### 3. Grant LAPS Read Permission

```powershell
Set-LapsADReadPasswordPermission `
  -Identity "OU=Workstations,OU=Computers,OU=Beginningdimension,DC=ad,DC=beginningdimension,DC=com" `
  -AllowedPrincipals "BEGINNINGDIM\GG_LAPS_Password_Readers"
```

This cmdlet delegates permission for specified users or groups to query LAPS passwords for
computers within the target OU
([Microsoft Learn — Set-LapsADReadPasswordPermission](https://learn.microsoft.com/en-us/powershell/module/laps/set-lapsadreadpasswordpermission?view=windowsserver2025-ps)).

### 4. Create the LAPS GPO

1. Open **Group Policy Management**.
2. Right-click the **Workstations** OU.
3. Select **Create a GPO in this domain, and Link it here**.
4. Name it: `Cfg-WindowsLAPS`
5. Edit the GPO.
6. Browse to:

```text
Computer Configuration
└── Policies
    └── Administrative Templates
        └── System
            └── LAPS
```

7. Configure:

| LAPS policy | Value |
|---|---|
| Configure password backup directory | Enabled: Active Directory |
| Password length | 16 characters or more |
| Password complexity | Uppercase, lowercase, numbers, special characters |
| Password age | 30 days |
| Post-authentication action | Reset password after approved local-admin use, if suitable |
| Password encryption | Enabled (default); configure authorized password decryptors as `GG_LAPS_Password_Readers` |

Windows LAPS Group Policy settings are located under Computer Configuration > Policies >
Administrative Templates > System > LAPS
([Microsoft Learn — Windows LAPS management policy settings](https://learn.microsoft.com/en-us/windows-server/identity/laps/laps-management-policy-settings)).

> **Encryption note:** Passwords stored in Active Directory are encrypted by default, and only
> the configured authorized password decryptors — by default, just Domain Admins — can decrypt
> them. Configuring `GG_LAPS_Password_Readers` as the authorized password decryptor is what
> allows its members to retrieve passwords with `Get-LapsADPassword`; the read permission granted
> with `Set-LapsADReadPasswordPermission` alone is not sufficient for encrypted passwords
> ([Microsoft Learn — Windows LAPS policy settings](https://learn.microsoft.com/en-us/windows-server/identity/laps/laps-management-policy-settings)).

### 5. Test LAPS

On CLIENT01:

```bat
gpupdate /force
```

Restart CLIENT01.

On DC01, using an authorized LAPS reader account:

```powershell
Get-LapsADPassword -Identity "CLIENT01" -AsPlainText
```

> **Security:** Do not place the recovered password in project documentation, tickets, or
> screenshots.

## Validation

- `Update-LapsADSchema` completes without errors (run with `-Verbose` to review each step if
  needed).
- Both permission cmdlets return the target OU distinguished names.
- `Get-LapsADPassword -Identity "CLIENT01"` returns a password with `PasswordUpdateTime` and
  `ExpirationTimestamp`, and `Source: EncryptedPassword` with a successful decryption status.
- Authorization tests: a member of `GG_LAPS_Password_Readers` successfully retrieves the
  password (positive test), and a standard non-privileged account cannot (negative test) —
  this confirms the read-permission and decryptor configuration behaves as intended.

## Visual Evidence

| # | Caption | Asset path |
|---|---|---|
| [54](../assets/screenshots/screenshot-54.png) | `Update-LapsADSchema` execution and confirmation output | `assets/screenshots/screenshot-54.png` |
| [55](../assets/screenshots/screenshot-55.png) | `Set-LapsADComputerSelfPermission` output for the Workstations OU | `assets/screenshots/screenshot-55.png` |
| [56](../assets/screenshots/screenshot-56.png) | `Set-LapsADReadPasswordPermission` output for GG_LAPS_Password_Readers | `assets/screenshots/screenshot-56.png` |
| [57](../assets/screenshots/screenshot-57.png) | `Cfg-WindowsLAPS` GPO created and linked to the Workstations OU | `assets/screenshots/screenshot-57.png` |
| [58](../assets/screenshots/screenshot-58.png) | LAPS policy settings — backup directory, length, complexity, age | `assets/screenshots/screenshot-58.png` |
| [59](../assets/screenshots/screenshot-59.png) | `Get-LapsADPassword` output on CLIENT01 (sanitized, no password shown) | `assets/screenshots/screenshot-59.png` |
| [60](../assets/screenshots/screenshot-60.png) | Authorized password decryptors configured as GG_LAPS_Password_Readers | `assets/screenshots/screenshot-60.png` |

Full mapping: [Evidence Index](evidence-index.md#phase-10--deploy-windows-laps).

## References

- [Microsoft Learn — Get started with Windows LAPS and Windows Server AD](https://learn.microsoft.com/en-us/windows-server/identity/laps/laps-scenarios-windows-server-active-directory)
- [Microsoft Learn — Windows LAPS overview](https://learn.microsoft.com/en-us/windows-server/identity/laps/laps-overview)
- [Microsoft Learn — Windows LAPS management policy settings](https://learn.microsoft.com/en-us/windows-server/identity/laps/laps-management-policy-settings)
- [Microsoft Learn — Set-LapsADReadPasswordPermission (PowerShell)](https://learn.microsoft.com/en-us/powershell/module/laps/set-lapsadreadpasswordpermission?view=windowsserver2025-ps)
- [Microsoft Learn — Windows LAPS PowerShell cmdlets overview](https://learn.microsoft.com/en-us/windows-server/identity/laps/laps-management-powershell)
- [Deploying LAPS with Group Policy and PowerShell automation (Abdalla Elmorssi)](https://www.abdallaelmorsi.net/active-directory-deploying-laps-with-group-policy-and-powershell-automation/)
- [Windows LAPS Configuration Guide (Veeam Community)](https://community.veeam.com/cyber-security-space-95/windows-laps-configuration-guide-4752)

Next: [Phase 11 — Security Auditing](phase-11-security-auditing.md)
