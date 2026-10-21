# Phase 1 — Configure DC01

> Phase series: [Phase 1](phase-01-configure-dc01.md) · [Phase 2](phase-02-install-ad-ds-dns-dc01.md) · [Phase 3](phase-03-configure-ad-dns.md) · [Phase 4](phase-04-configure-dc02.md) · [Phase 5](phase-05-ad-site-topology.md) · [Phase 6](phase-06-organizational-units.md) · [Phase 7](phase-07-users-groups-delegation.md) · [Phase 8](phase-08-join-client01.md) · [Phase 9](phase-09-gpo-security-baseline.md) · [Phase 10](phase-10-windows-laps.md) · [Phase 11](phase-11-security-auditing.md) · [Phase 12](phase-12-backup-recovery.md) · [Phase 13](phase-13-health-checks.md)
> 

## Objective & Scope

Prepare the first Windows Server 2025 machine to become the forest root domain controller:
assign a meaningful hostname, configure static IPv4 addressing, and validate basic network
connectivity before any roles are installed.

**Scope:** hostname assignment, static IP configuration, connectivity validation, patching, and
time/date verification on DC01.

## Prerequisites

- Windows Server 2025 installed and accessible as the local `Administrator` account.
- The `192.168.10.0/24` network is available and `192.168.10.1` (gateway) is confirmed.
- Before proceeding, confirm each pre-deployment checklist item:

- [ ] All virtual machines or servers are connected to the same virtual switch or LAN.
- [ ] The `192.168.10.0/24` network is available.
- [ ] `192.168.10.1` is confirmed as the correct default gateway.
- [ ] All systems have current Windows updates.
- [ ] All servers have appropriate local Administrator passwords.
- [ ] Strong DSRM passwords are prepared and securely stored.
- [ ] A backup target is available, such as drive `E:` or approved network storage.
- [ ] Windows 11 edition supports domain joining.

## Step-by-Step Execution

### 1. Rename DC01

1. Sign in to the first Windows Server 2025 machine as local `Administrator`.
2. Open **Server Manager**.
3. Select **Local Server**.
4. Click the current **Computer name**.
5. Select **Change**.
6. Enter: `DC01`
7. Restart the server.

### 2. Configure Static IP Address

1. Open **Server Manager > Local Server**.
2. Click the network adapter link.
3. Select **Change adapter options**.
4. Right-click the active network adapter and select **Properties**.
5. Select **Internet Protocol Version 4 (TCP/IPv4)**.
6. Select **Properties**.
7. Configure the following:

| Setting | Value |
|---|---|
| IP address | `192.168.10.10` |
| Subnet mask | `255.255.255.0` |
| Default gateway | `192.168.10.1` |
| Preferred DNS | `192.168.10.1` (temporarily) |
| Alternate DNS | *(leave blank)* |

8. Select **OK**.

The gateway is used as a temporary DNS resolver only until the server is promoted to a domain
controller and becomes authoritative for its own zone.

### 3. Validate Connectivity

Open **Command Prompt as Administrator**:

```bat
hostname
ipconfig /all
ping 192.168.10.1
```

Then install all available Windows updates and confirm that the server's **date, time, and time
zone are correct** before continuing. Accurate time is a hard prerequisite for Kerberos
authentication — see the [time-sync incident](troubleshooting-log.md#incident-01--kerberos-time-sync-failure-breaking-gpo-application) in the
troubleshooting log for what happens when it drifts.

## Validation

- `hostname` returns `DC01`.
- `ipconfig /all` shows the static address `192.168.10.10` with gateway `192.168.10.1`.
- `ping 192.168.10.1` succeeds.
- Windows Update reports no pending updates; system date, time, and time zone are correct.

## Visual Evidence

| # | Caption | Asset path |
|---|---|---|
Refer to phase 1 in the screenshot folder.
Full mapping: [Evidence Index](evidence-index.md#phase-1--configure-dc01).

## References

- [Microsoft Learn — Install Active Directory Domain Services (Level 100)](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/deploy/install-active-directory-domain-services--level-100-)
- [Microsoft Learn — Deploy and manage AD DS domain controllers](https://learn.microsoft.com/en-us/training/modules/deploy-manage-active-directory-domain-services-domain-controllers/)

Next: [Phase 2 — Install AD DS and DNS on DC01](phase-02-install-ad-ds-dns-dc01.md)
