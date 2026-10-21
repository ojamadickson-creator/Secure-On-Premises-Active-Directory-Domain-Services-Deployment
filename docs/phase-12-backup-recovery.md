# Phase 12 — Backup and Recovery

> Phase series: [Phase 1](phase-01-configure-dc01.md) · [Phase 2](phase-02-install-ad-ds-dns-dc01.md) · [Phase 3](phase-03-configure-ad-dns.md) · [Phase 4](phase-04-configure-dc02.md) · [Phase 5](phase-05-ad-site-topology.md) · [Phase 6](phase-06-organizational-units.md) · [Phase 7](phase-07-users-groups-delegation.md) · [Phase 8](phase-08-join-client01.md) · [Phase 9](phase-09-gpo-security-baseline.md) · [Phase 10](phase-10-windows-laps.md) · [Phase 11](phase-11-security-auditing.md) · [Phase 12](phase-12-backup-recovery.md) · [Phase 13](phase-13-health-checks.md)
> 

## Objective & Scope

Establish domain-controller recovery capability: install Windows Server Backup on both DCs and
perform a System State backup to a dedicated backup target.

**Scope:** Windows Server Backup feature installation and System State backup via `wbadmin` on
DC01 and DC02.

## Prerequisites

- A dedicated backup target available, such as drive `E:` or approved network storage.
- DSRM passwords documented and stored in an approved password vault.

## Step-by-Step Execution

### 1. Install Windows Server Backup

On both DC01 and DC02:

1. Open **Server Manager**.
2. Select **Manage > Add Roles and Features**.
3. Select **Features**.
4. Select **Windows Server Backup**.
5. Select **Install**.

### 2. Create a System State Backup

Open **Command Prompt as Administrator**:

```bat
wbadmin start systemstatebackup -backupTarget:E: -quiet
```

Replace `E:` with the approved backup disk or storage location.

### 3. Backup Rules

- Run System State backups **daily** on both DCs.
- Use protected and separate backup storage.
- Maintain at least one offline or immutable backup where possible.
- Test restoration in an isolated recovery environment.
- Maintain documented DSRM credentials.
- **Do not rely on replication as backup.**
- **Do not use VM snapshots as the normal AD recovery process** — restoring domain controllers
  from old VM snapshots can cause USN rollback and replication divergence.
- **Hypervisor-level VM appliance exports complement System State backups** in this lab: a
  VirtualBox Export Appliance (`.ova`) image of DC01 was taken before the forest-level LAPS
  schema change (see [Phase 10](phase-10-windows-laps.md#pre-deployment-change-record--point-in-time-backup)),
  providing an exact point-in-time clone for pre-change recovery. Re-importing an appliance
  restores the full machine; for AD-aware recovery, pair this with AD restoration guidance for
  virtualized domain controllers.

## Validation

- Windows Server Backup appears in Server Manager > Tools on both DCs.
- `wbadmin start systemstatebackup` completes with a summary showing the backup completed
  successfully.
- `wbadmin get versions` lists the created System State backup on the target.

## Visual Evidence

| # | Caption | Asset path |
|---|---|---|
| See Screenshots | | docs/screenshots/Phase 12 |
Full mapping: [Evidence Index](evidence-index.md#phase-12--backup-and-recovery).

## References

- [Microsoft Learn — AD forest recovery: backing up system state](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/forest-recovery-guide/ad-forest-recovery-backing-up-system-state)
- [Microsoft Learn — AD forest recovery procedures](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/forest-recovery-guide/ad-forest-recovery-procedures)
- [Backup and restore Active Directory System State (mcsaguru)](https://mcsaguru.com/backup-restore-active-directory-system-state)
- [Active Directory backup & forest recovery guide (Decryption Digest)](https://www.decryptiondigest.com/blog/active-directory-backup-forest-recovery-guide)
- [IBM — Best practices for Active Directory setup and configuration](https://www.ibm.com/support/pages/best-practices-active-directory-setup-and-configuration)

Next: [Phase 13 — Health Checks and Troubleshooting](phase-13-health-checks.md)
