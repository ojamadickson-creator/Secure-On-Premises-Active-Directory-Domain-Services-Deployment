# Project Validation Checklist

Final validation checklist for the `ad.beginningdimension.com` deployment, compiled from the
implementation procedures documented in [Phases 1–13](phase-01-configure-dc01.md). Use it to
confirm the environment at the end of the build — items are intentionally left unchecked for use
as a working checklist.

> 

## Core Infrastructure

- [ ] DC01 has hostname `DC01` and IP address `192.168.10.10`.
- [ ] DC02 has hostname `DC02` and IP address `192.168.10.11`.
- [ ] CLIENT01 has hostname `CLIENT01` and IP address `192.168.10.20`.
- [ ] Domain name is `ad.beginningdimension.com`.
- [ ] NetBIOS name is `BEGINNINGDIM`.
- [ ] DC01 and DC02 run AD DS and DNS.
- [ ] Both DCs are Global Catalog servers.
- [ ] Both DCs use only internal DNS addresses.
- [ ] CLIENT01 uses DC01 and DC02 as DNS servers.

## DNS and Replication

- [ ] Forward lookup zone for `ad.beginningdimension.com` exists.
- [ ] Reverse lookup zone for `192.168.10.0/24` exists.
- [ ] Secure dynamic updates are enabled.
- [ ] DNS forwarders are configured.
- [ ] `dcdiag /test:dns` completes successfully.
- [ ] `repadmin /replsummary` shows no replication failures.
- [ ] `_ldap._tcp.dc._msdcs.ad.beginningdimension.com` resolves successfully.

## Site Topology

- [ ] The HQ site exists.
- [ ] The `192.168.10.0/24` subnet is mapped to HQ.
- [ ] DC01 and DC02 are located in the HQ site.
- [ ] `nltest /dsgetsite` returns HQ on all systems.

## Security

- [ ] Separate standard and privileged accounts exist.
- [ ] Domain Admins membership is restricted.
- [ ] Security groups are role-based.
- [ ] Workstation baseline GPO is linked to the Workstations OU.
- [ ] Password and lockout policy is configured.
- [ ] Windows LAPS is configured and tested.
- [ ] LAPS authorization test passes: members of `GG_LAPS_Password_Readers` can retrieve passwords;
  standard non-privileged accounts cannot.
- [ ] Advanced auditing is enabled on domain controllers.
- [ ] Important security event IDs are monitored.

## Resilience

- [ ] Windows Server Backup is installed on both DCs.
- [ ] System State backup completes successfully.
- [ ] DSRM passwords are securely stored.
- [ ] A restoration test is planned and documented.
- [ ] Daily health-check commands are documented.

---

Related: [Phase 13 — Health Checks](phase-13-health-checks.md) ·
[Troubleshooting Log](troubleshooting-log.md) ·
[Evidence Index](evidence-index.md)

Back to: [README](../README.md)
