# Phase 8 — Join CLIENT01 to the Domain

> Phase series: [Phase 1](phase-01-configure-dc01.md) · [Phase 2](phase-02-install-ad-ds-dns-dc01.md) · [Phase 3](phase-03-configure-ad-dns.md) · [Phase 4](phase-04-configure-dc02.md) · [Phase 5](phase-05-ad-site-topology.md) · [Phase 6](phase-06-organizational-units.md) · [Phase 7](phase-07-users-groups-delegation.md) · [Phase 8](phase-08-join-client01.md) · [Phase 9](phase-09-gpo-security-baseline.md) · [Phase 10](phase-10-windows-laps.md) · [Phase 11](phase-11-security-auditing.md) · [Phase 12](phase-12-backup-recovery.md) · [Phase 13](phase-13-health-checks.md)
> 

## Objective & Scope

Configure the Windows 11 workstation (CLIENT01), join it to the `ad.beginningdimension.com` domain,
and validate authentication, policy application, and computer object placement.

**Scope:** CLIENT01 network configuration, GUI domain join, and post-join validation.

## Prerequisites

- [Phase 6](phase-06-organizational-units.md) completed — OU redirection is in place so the
  computer object lands in the Workstations OU.
- Windows 11 Pro or Enterprise edition (domain-join capable).
- Domain credentials for the join operation.

## Step-by-Step Execution

### 1. Configure CLIENT01 Network

Set static IPv4 configuration:

| Setting | Value |
|---|---|
| Computer name | `CLIENT01` |
| IP address | `192.168.10.20` |
| Subnet mask | `255.255.255.0` |
| Default gateway | `192.168.10.1` |
| Preferred DNS | `192.168.10.10` |
| Alternate DNS | `192.168.10.11` |

Domain clients must use **only the internal AD DNS servers** — never public DNS servers directly.

### 2. Join Domain Through GUI

1. On CLIENT01, open **Settings > System > About**.
2. Select **Domain or workgroup** or **Advanced system settings**.
3. Select **Change**.
4. Choose **Domain**.
5. Enter: `ad.beginningdimension.com`
6. Authenticate as: `BEGINNINGDIM\Administrator`
7. Restart CLIENT01.

### 3. Validate the Domain Join

Sign in as: `BEGINNINGDIM\jdoe`

Run:

```bat
whoami
echo %logonserver%
nltest /dsgetdc:ad.beginningdimension.com
gpupdate /force
gpresult /r
```

Move CLIENT01 from the default location, if needed, into:
`Beginningdimension > Computers > Workstations`

> **Operational note:** if `gpresult /r` does not reflect a newly linked GPO or new group
> membership, the Kerberos ticket cache may still hold the old token — see
> [token-cache refresh](troubleshooting-log.md#supplementary-operational-notes) in the
> troubleshooting log.

## Validation

- `whoami` returns `beginningdim\jdoe`.
- `%logonserver%` points at DC01 or DC02.
- `nltest /dsgetdc:` returns a domain controller in the HQ site.
- `gpupdate /force` completes without errors.
- `gpresult /r` shows applied Group Policy objects.
- CLIENT01's computer object resides in the Workstations OU.

## Visual Evidence

| # | Caption | Asset path |
|---|---|---|
See Screenshots | | docs/screenshots/Phase 8 |

Full mapping: [Evidence Index](evidence-index.md#phase-8--join-client01-to-the-domain).

## References

- [Microsoft Learn — Deploy and manage AD DS domain controllers](https://learn.microsoft.com/en-us/training/modules/deploy-manage-active-directory-domain-services-domain-controllers/)
- [Microsoft Learn — AD DS deployment](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/deploy/ad-ds-deployment)

Next: [Phase 9 — Group Policy Security Baseline](phase-09-gpo-security-baseline.md)
