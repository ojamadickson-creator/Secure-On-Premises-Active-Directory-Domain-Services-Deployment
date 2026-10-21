# Phase 13 — Health Checks and Troubleshooting

> Phase series: [Phase 1](phase-01-configure-dc01.md) · [Phase 2](phase-02-install-ad-ds-dns-dc01.md) · [Phase 3](phase-03-configure-ad-dns.md) · [Phase 4](phase-04-configure-dc02.md) · [Phase 5](phase-05-ad-site-topology.md) · [Phase 6](phase-06-organizational-units.md) · [Phase 7](phase-07-users-groups-delegation.md) · [Phase 8](phase-08-join-client01.md) · [Phase 9](phase-09-gpo-security-baseline.md) · [Phase 10](phase-10-windows-laps.md) · [Phase 11](phase-11-security-auditing.md) · [Phase 12](phase-12-backup-recovery.md) · [Phase 13](phase-13-health-checks.md)
>

## Objective & Scope

Establish the routine operational procedures that keep the environment healthy: daily domain
controller checks, FSMO role verification, DNS and replication troubleshooting workflows, domain
discovery validation, secure-channel repair, and account lockout investigation.

**Scope:** standing health-check commands and troubleshooting runbooks for the domain.

## Daily Domain Controller Checks

```bat
dcdiag /q
repadmin /replsummary
repadmin /showrepl
w32tm /query /status
```

## Check FSMO Roles

```bat
netdom query fsmo
```

For this project, retain FSMO roles on **DC01** unless there is an approved maintenance or
recovery requirement.

## DNS Troubleshooting

```bat
ipconfig /all
ipconfig /flushdns
ipconfig /registerdns
dcdiag /test:dns
nslookup -type=SRV _ldap._tcp.dc._msdcs.ad.beginningdimension.com
```

For the full DNS post-promotion incident set (forwarder timeouts, stale multi-homed records,
binding conflicts), see the [troubleshooting log](troubleshooting-log.md).

## Domain Discovery

```bat
nltest /dsgetdc:ad.beginningdimension.com
nltest /dsgetsite
```

## Replication Troubleshooting

```bat
repadmin /showrepl
repadmin /replsummary
repadmin /syncall /AdeP
```

## Repair a Broken Computer Trust

Sign in to CLIENT01 with a local administrator account, then run PowerShell as Administrator:

```powershell
Test-ComputerSecureChannel `
  -Repair `
  -Credential (Get-Credential)
```

Restart CLIENT01 after a successful repair.

> **Operational note:** `-Credential (Get-Credential)` opens an interactive credential prompt —
> this is expected; supply domain administrative credentials when prompted. See the
> [PowerShell parameter prompt notes](troubleshooting-log.md#supplementary-operational-notes).

## Investigate Account Lockouts

1. On DC01, open **Event Viewer**.
2. Open **Windows Logs > Security**.
3. Filter for **Event ID: 4740**.
4. Identify the source computer.
5. Check Credential Manager, mapped drives, services, scheduled tasks, mobile devices, and saved
   passwords on the source machine — stale credentials on these are the most common cause of
   repeated lockouts.

## Validation

- Daily checks run clean: `dcdiag /q` silent (no failures), `repadmin /replsummary` with no
  failures, `w32tm /query /status` showing stratum and last successful sync time.
- FSMO roles all held by DC01.
- SRV record queries resolve; `dcdiag /test:dns` passes.
- `nltest /dsgetdc:` returns a DC; `nltest /dsgetsite` returns HQ on all systems.
- Secure-channel repair can be validated by testing `Test-ComputerSecureChannel -Repair` on a
  workstation whose trust requires repair (e.g., after a DC restore), confirming the cmdlet
  returns `True` and the domain logon succeeds afterward.





## References

- [Microsoft Learn — Active Directory Replication Concepts](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/replication/active-directory-replication-concepts)
- [Microsoft Learn — AD DS deployment](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/deploy/ad-ds-deployment)
- [RAFF Technologies — AD replication topology](https://rafftechnologies.com/windows-server/ad-replication-topology)

Next: [Troubleshooting & Issues Encountered](troubleshooting-log.md)
