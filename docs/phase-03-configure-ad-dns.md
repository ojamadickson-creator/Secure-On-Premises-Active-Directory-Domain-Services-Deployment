# Phase 3 — Configure AD DNS

> Phase series: [Phase 1](phase-01-configure-dc01.md) · [Phase 2](phase-02-install-ad-ds-dns-dc01.md) · [Phase 3](phase-03-configure-ad-dns.md) · [Phase 4](phase-04-configure-dc02.md) · [Phase 5](phase-05-ad-site-topology.md) · [Phase 6](phase-06-organizational-units.md) · [Phase 7](phase-07-users-groups-delegation.md) · [Phase 8](phase-08-join-client01.md) · [Phase 9](phase-09-gpo-security-baseline.md) · [Phase 10](phase-10-windows-laps.md) · [Phase 11](phase-11-security-auditing.md) · [Phase 12](phase-12-backup-recovery.md) · [Phase 13](phase-13-health-checks.md)
>

## Objective & Scope

Configure the AD-integrated DNS infrastructure on DC01: upstream forwarders for internet name
resolution, a reverse lookup zone for the lab subnet, and full validation of service-locator
records.

**Scope:** DNS forwarders, reverse lookup zone creation, SRV record validation on DC01.

## Prerequisites

- [Phase 2](phase-02-install-ad-ds-dns-dc01.md) completed — DC01 promoted and signing in as
  `BEGINNINGDIM\Administrator`.

## Step-by-Step Execution

### 1. Configure DNS Forwarders

1. Open **Server Manager > Tools > DNS**.
2. Right-click **DC01**.
3. Select **Properties**.
4. Select the **Forwarders** tab.
5. Select **Edit**.
6. Add approved upstream DNS resolvers, such as:
   - `1.1.1.1`
   - `8.8.8.8`
7. Select **OK**, then **Apply**.

> **Lab note:** Do not add the lab gateway (`192.168.10.1`) as a forwarder. A router
> interface that is not acting as an active recursive DNS listener will time out and show a red
> cross in DNS Manager — this exact issue was diagnosed and resolved during the build
> ([Incident 02](troubleshooting-log.md#incident-02--dns-forwarder-gateway-timeout-192168101)).

### 2. Create Reverse Lookup Zone

1. Open **DNS Manager**.
2. Expand **DC01**.
3. Right-click **Reverse Lookup Zones**.
4. Select **New Zone**.
5. Select **Primary zone**.
6. Select **Store the zone in Active Directory**.
7. Select replication **to all DNS servers in the forest**.
8. Select **IPv4 Reverse Lookup Zone**.
9. Enter: `192.168.10`
10. Select **Allow only secure dynamic updates**.
11. Complete the wizard.

### 3. Validate DNS

```bat
nslookup ad.beginningdimension.com
nslookup -type=SRV _ldap._tcp.dc._msdcs.ad.beginningdimension.com
dcdiag /test:dns
```

The SRV record lookup should locate DC01. AD DS depends on DNS service records for
domain-controller discovery, authentication, and related directory functions.

## Validation

- `nslookup ad.beginningdimension.com` resolves to `192.168.10.10`.
- The `_ldap._tcp.dc._msdcs` SRV query returns DC01 as the domain controller.
- `dcdiag /test:dns` completes with `ad.beginningdimension.com passed test DNS`.

> **Expected diagnostic noise:** warnings about public forwarders failing to resolve the local
> loopback reverse record (`1.0.0.127.in-addr.arpa`) are standard and non-impacting — see
> [Incident 04](troubleshooting-log.md#incident-04--multi-homed-interface-binding-conflicts) verification notes.

## Visual Evidence

| # | Caption | Asset path |
|---|---|---|
| See Docs/screenshots/Phase 3 for screenshots.

Full mapping: [Evidence Index](evidence-index.md#phase-3--configure-ad-dns).

## References

- [Microsoft Learn — Install Active Directory Domain Services (Level 100)](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/deploy/install-active-directory-domain-services--level-100-)
- [Microsoft Learn — AD DS deployment](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/deploy/ad-ds-deployment)

Next: [Phase 4 — Configure DC02](phase-04-configure-dc02.md)
