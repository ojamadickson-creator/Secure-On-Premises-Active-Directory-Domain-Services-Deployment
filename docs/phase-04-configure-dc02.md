# Phase 4 — Configure DC02

> Phase series: [Phase 1](phase-01-configure-dc01.md) · [Phase 2](phase-02-install-ad-ds-dns-dc01.md) · [Phase 3](phase-03-configure-ad-dns.md) · [Phase 4](phase-04-configure-dc02.md) · [Phase 5](phase-05-ad-site-topology.md) · [Phase 6](phase-06-organizational-units.md) · [Phase 7](phase-07-users-groups-delegation.md) · [Phase 8](phase-08-join-client01.md) · [Phase 9](phase-09-gpo-security-baseline.md) · [Phase 10](phase-10-windows-laps.md) · [Phase 11](phase-11-security-auditing.md) · [Phase 12](phase-12-backup-recovery.md) · [Phase 13](phase-13-health-checks.md)
> 

## Objective & Scope

Deploy the second domain controller (DC02) for authentication and DNS redundancy: configure its
identity and networking, join it to the domain, install AD DS and DNS, promote it as a replica
domain controller, and configure cross-pointed DNS client settings.

**Scope:** DC02 identity/networking, domain join, role installation, promotion, cross-pointed DNS,
and replication health validation.

## Prerequisites

- [Phase 3](phase-03-configure-ad-dns.md) completed — DC01 healthy, DNS validated.
- Domain administrative credentials available (`BEGINNINGDIM\Administrator`).
- A strong DSRM password prepared and securely stored for DC02.

## Step-by-Step Execution

### 1. Configure Server Identity and Networking

On the second Windows Server 2025 system:

1. Rename the server: `DC02`
2. Restart.
3. Configure IPv4 settings:

| Setting | Value |
|---|---|
| IP address | `192.168.10.11` |
| Subnet mask | `255.255.255.0` |
| Default gateway | `192.168.10.1` |
| Preferred DNS | `192.168.10.10` |
| Alternate DNS | *(leave blank initially)* |

4. Install Windows updates.
5. Test communication:

```bat
ping 192.168.10.10
nslookup ad.beginningdimension.com 192.168.10.10
```

### 2. Join DC02 to the Domain

1. Open **Server Manager > Local Server**.
2. Click **Workgroup**.
3. Select **Change**.
4. Select **Domain**.
5. Enter: `ad.beginningdimension.com`
6. Authenticate as: `BEGINNINGDIM\Administrator`
7. Restart DC02.

### 3. Install AD DS and DNS

1. Sign in to DC02 with domain administrative credentials.
2. Open **Server Manager**.
3. Select **Manage > Add Roles and Features**.
4. Install:
   - **Active Directory Domain Services**
   - **DNS Server**
5. Finish installation.

### 4. Promote DC02

1. Select the Server Manager notification flag.
2. Select **Promote this server to a domain controller**.
3. Choose **Add a domain controller to an existing domain**.
4. Confirm: `ad.beginningdimension.com`
5. On **Domain Controller Options**:
   - Select **DNS Server**.
   - Select **Global Catalog**.
   - Do **not** select Read-only domain controller.
   - Set and securely record a **DSRM password**.
6. Complete the wizard and select **Install**.
7. Allow DC02 to restart.

### 5. Configure Cross-Pointed DNS

After DC02 restarts, update DNS settings so each domain controller points primarily at its peer
for name resolution, avoiding self-referential island scenarios:

**DC01 DNS client settings**

| Setting | Value |
|---|---|
| Preferred DNS | `192.168.10.11` |
| Alternate DNS | `192.168.10.10` |

**DC02 DNS client settings**

| Setting | Value |
|---|---|
| Preferred DNS | `192.168.10.10` |
| Alternate DNS | `192.168.10.11` |

### 6. Validate DC Health

Run on either domain controller:

```bat
dcdiag
repadmin /replsummary
repadmin /showrepl
repadmin /syncall /AdeP
```

**Do not continue until replication is healthy and all critical DNS tests are successful.**

## Validation

- `dcdiag` reports no failures for DC02.
- `repadmin /replsummary` shows no replication failures between DC01 and DC02.
- `repadmin /showrepl` shows successful inbound replication from the partner DC.
- `repadmin /syncall /AdeP` completes without errors.
- Both DCs respond as Global Catalog servers and DNS servers.

## Visual Evidence

| # | Caption | Asset path |
|---|---|---|
Please see the screenshots in | | docs/screenshots/Phase 4|
Full mapping: [Evidence Index](evidence-index.md#phase-4--configure-dc02).

## References

- [Microsoft Learn — Install Active Directory Domain Services (Level 100)](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/deploy/install-active-directory-domain-services--level-100-)
- [Microsoft Learn — Active Directory Replication Concepts](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/replication/active-directory-replication-concepts)
- [Microsoft Learn — Deploy and manage AD DS domain controllers](https://learn.microsoft.com/en-us/training/modules/deploy-manage-active-directory-domain-services-domain-controllers/)

Next: [Phase 5 — Configure AD Site Topology](phase-05-ad-site-topology.md)
