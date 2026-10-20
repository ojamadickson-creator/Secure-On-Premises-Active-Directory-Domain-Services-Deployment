# Troubleshooting & Issues Encountered

This log documents the real-world problems faced during the build, their root causes, and the
methodical steps used to resolve them. It is organized as incident reports: symptom → root cause
→ remediation → verification. Incidents 01–04 are drawn from the lab's incident and
post-promotion troubleshooting briefs for `ad.beginningdimension.com`; the supplementary notes
are supplemental runbook guidance (Kerberos token caching, LAPS cmdlet prompts, advanced audit
policy) — they complement the incident log rather than record specific failures.

> Source: lab incident report & technical resolution brief (time sync / GPO) and the AD & DNS
> post-promotion troubleshooting reference (DC01), both for `ad.beginningdimension.com`.

---

## Incident 01 — Kerberos Time-Sync Failure Breaking GPO Application

**Status: RESOLVED**

### Symptom

Group Policy Objects — specifically `Sec-Workstation-Baseline` — failed to apply on CLIENT01.
Executing `gpupdate /force` returned fatal errors indicating the system clock was not synchronized
with the domain controllers. The PDC (DC01) also reported *"no time data was available"*.

### Root Cause

Excessive clock skew between the domain controllers and CLIENT01. Active Directory relies on
Kerberos v5 authentication, which rejects authentication tokens when time drift exceeds the
tolerance (5 minutes by default — a configurable Kerberos policy, not a fixed protocol
constant). The isolated lab setup lacked an external NTP reference, so the primary DC had no
authoritative time source, starving downstream nodes.

### Resolution Workflow

In this lab's resolution — a single-site domain where DC01 holds the PDC Emulator role — time
distribution was configured strictly top-down. In general enterprise terms, the domain hierarchy
model means member systems synchronize from the domain time service (an authoritative DC)
via NT5DS rather than each node configuring external peers; which DC a member actually syncs
from can be confirmed with `w32tm /query /source` and `w32tm /query /status`:

1. **External NTP source:** DC01 (PDC Emulator) synchronizes directly with public internet NTP
   servers and is marked as a reliable time source.
2. **Domain hierarchy:** DC02 and CLIENT01 synchronize from the domain time hierarchy via
   internal RPC/NTP traffic.

**Step 1 — Establish primary time authority on DC01 (PDC Emulator).** Configure DC01 to pull time
from reliable external NTP pools and mark itself as an authoritative time source:

```bat
w32tm /config /manualpeerlist:"time.windows.com pool.ntp.org" /syncfromflags:manual /reliable:yes /update
net stop w32time & net start w32time
w32tm /resync /force
```

**Step 2 — Point downstream members (DC02 & CLIENT01) to the hierarchy.** Reconfigure internal
member systems to pull time directly from the AD domain hierarchy:

```bat
w32tm /config /syncfromflags:domhier /update
net stop w32time & net start w32time
w32tm /resync /force
```

**Step 3 — Enforce & verify GPO application on CLIENT01.** Once time skew is eliminated, refresh
policy and verify computer-scoped GPO enforcement:

```bat
gpupdate /force
gpresult /scope computer /r
```

### Command Reference Quick-Sheet

| Action / Purpose | Command | Target Node |
|---|---|---|
| Set manual NTP | `w32tm /config /manualpeerlist:...` | DC01 (PDC) |
| Set domain hierarchy | `w32tm /config /syncfromflags:domhier` | DC02 / CLIENT01 |
| Force resync | `w32tm /resync /force` | All nodes |
| Query time status | `w32tm /query /status` | All nodes |
| Force policy refresh | `gpupdate /force` | CLIENT01 |
| Verify applied GPOs | `gpresult /scope computer /r` | CLIENT01 |

### Key Takeaways

- **Kerberos clock tolerance:** always check system time synchronization first when
  troubleshooting generic GPO application or domain logon failures.
- **Network security boundary:** isolated member servers and workstations do not need internet
  access to maintain time sync; the domain hierarchy handles internal distribution.

---

## Incident 02 — DNS Forwarder Gateway Timeout (192.168.10.1)

**Status: RESOLVED**

### Symptom

In DNS Manager under **Forwarders**, the local gateway address (`192.168.10.1`) displayed a red
cross with a query timeout error.

### Root Cause

The default gateway router (OPNsense) was not operating as an active recursive DNS listener on
port 53, causing recursive lookup attempts to hang and time out.

### Remediation

1. Open **DNS Manager > right-click server node > Properties > Forwarders** tab.
2. Select the gateway IP address (`192.168.10.1`) and click **Delete**.
3. Retain reliable public upstream resolvers (e.g., `1.1.1.1`, `8.8.8.8`), which validate with
   green checkmarks.

---

## Incident 03 — DNS Timeouts and "Non-Existent Domain" Errors After Promotion

**Status: RESOLVED**

### Symptom

Executing `nslookup ad.beginningdimension.com` or SRV queries failed with request timeouts or
*"Non-existent domain"* errors.

### Root Cause

An active secondary network interface (DHCP/NAT adapter) was enabled during the AD DS role
promotion. The DNS service bound to the external interface and populated the zone with stale
`10.0.3.x` host records instead of the static lab IP (`192.168.10.10`).

### Remediation

1. **Isolate the network adapter:** open `ncpa.cpl` and disable all extraneous adapters. Ensure
   only the dedicated lab switch interface is active.
2. **Purge stale records:** in DNS Manager, expand `ad.beginningdimension.com` and delete all
   stale A records (including `(same as parent folder)` and `dc01`) pointing to `10.0.3.x`.
3. **Force re-registration:** clear local caches and restart registration services from Command
   Prompt:

```bat
ipconfig /flushdns
ipconfig /registerdns
net stop netlogon && net start netlogon
```

---

## Incident 04 — Multi-Homed Interface Binding Conflicts

**Status: RESOLVED**

### Symptom

Intermittent query failures returning *"No response from server"* despite purging stale DNS
records.

### Root Cause

Having multiple active adapters on a domain controller splits directory traffic and causes the
DNS server service to listen on unroutable interfaces.

### Remediation

1. Verify in **Network Connections** that only one primary static adapter is active.
2. Enforce explicit IP binding: open **DNS Manager > Right-click DC01 > Properties > Interfaces**
   tab.
3. Select **"Only the following IP addresses"**, check `192.168.10.10`, and uncheck all
   loopback/secondary IPs.
4. Restart DNS and Netlogon services:

```bat
net stop dns && net start dns
net stop netlogon && net start netlogon
```

### Post-Remediation Verification Protocol (Passed)

To ensure directory services and service-locator (SRV) records are operating correctly across the
forest, the following commands are executed in an administrative CLI:

```bat
nslookup ad.beginningdimension.com 192.168.10.10
nslookup -type=SRV _ldap._tcp.dc._msdcs.ad.beginningdimension.com
dcdiag /test:dns
```

**Expected diagnostic output note:** warnings regarding public forwarders failing to resolve the
local loopback reverse record (`1.0.0.127.in-addr.arpa`) are standard and non-impacting. Overall
domain validation is confirmed when `dcdiag` outputs:

```text
ad.beginningdimension.com passed test DNS.
```

---

## Supplementary Operational Notes

These notes capture recurring operational practices and prompt-handling techniques relevant to
this build. They are supplemental runbook guidance, not logged incidents.

### Kerberos token-cache group membership delays

Group membership changes made in Active Directory do not affect an existing logon session until
the session's security token is rebuilt. Two distinct mechanisms are involved, and they should
not be conflated:

- **The local access token (who the machine thinks you are):** created once at logon and
  immutable for the life of the session. `klist purge` does **not** refresh it, and
  `whoami /groups` in an existing process keeps the old membership until the user signs out and
  signs back in. The reliable user-session refresh is sign-out/sign-in (or reboot).
- **Kerberos tickets (what remote services see):** service tickets inherit group SIDs from the
  TGT, and ordinary TGT renewal only extends validity without refreshing the group list.
  Purging tickets (`klist purge` as the user, or `klist -lh 0 -li 0x3e7 purge` for the computer
  account, followed by `gpupdate /force`) forces new tickets to be requested, so subsequent
  access to Kerberos-authenticated remote resources reflects current membership.

Practical guidance: for user-facing permissions on the local machine or GPO security-group
filtering, use sign-out/sign-in; for remote resource access and computer-account group changes,
the `klist` purge approach avoids a reboot. NTLM-authenticated sessions hold their membership
until the session ends.

### Windows LAPS schema and permission cmdlet prompts

- `Update-LapsADSchema` is a one-time, forest-level change and prompts for confirmation before
  extending the schema; review each prompt and confirm. Run it with `-Verbose` to watch each
  attribute being added, and always take a System State backup first. The account running it
  needs Schema Admins membership.
- `Set-LapsADComputerSelfPermission` and `Set-LapsADReadPasswordPermission` operate on an OU
  identity; if inheritance is blocked on child OUs, the permission must be applied to each child
  OU separately.
- Password read permission alone is not sufficient for encrypted passwords — the retrieving
  group must also be configured as the authorized password decryptor in the LAPS GPO.

### Advanced audit policy subcategory configuration

The nine legacy audit-policy categories can mask Advanced Audit Policy subcategory settings.
Always enable **Audit: Force audit policy subcategory settings to override audit policy category
settings** when using Advanced Audit Policy Configuration, and verify the resulting audit policy
with `gpresult /h` or by inspecting generated events (e.g., 4625 on failed logon).

### PowerShell parameter prompt resolution

Interactive cmdlets such as `Test-ComputerSecureChannel -Repair -Credential (Get-Credential)`
open a Windows credential prompt by design. Supply domain administrative credentials when
prompted. Note that pre-collecting the credential (`$cred = Get-Credential`) still prompts
interactively — it only moves the prompt earlier; genuinely unattended automation requires a
different technique (for example, a scheduled task running as an appropriate service account).
For interactive troubleshooting, collecting the credential first and passing it as a parameter
keeps the cmdlet itself free of mid-command prompts.

---

## References

- [Microsoft Learn — Windows LAPS overview](https://learn.microsoft.com/en-us/windows-server/identity/laps/laps-overview)
- [Microsoft Learn — Windows LAPS management policy settings](https://learn.microsoft.com/en-us/windows-server/identity/laps/laps-management-policy-settings)
- [Microsoft Learn — Get started with Windows LAPS and Windows Server AD](https://learn.microsoft.com/en-us/windows-server/identity/laps/laps-scenarios-windows-server-active-directory)
- [Microsoft Learn — Active Directory Replication Concepts](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/replication/active-directory-replication-concepts)
- [Microsoft Learn — Designing the Site Topology](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/designing-the-site-topology)

Back to: [README](../README.md) · [Phase 13](phase-13-health-checks.md) · [Validation Checklist](validation-checklist.md)
