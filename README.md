# Enterprise Active Directory Deployment & Security Hardening Lab


# Enterprise Active Directory Deployment & Security Hardening Lab

[![OS - Windows Server 2025](https://img.shields.io/badge/OS-Windows_Server_2025-0078D6?style=flat&logo=windows&logoColor=white)](https://www.microsoft.com/evalcenter/evaluate-windows-server-2025)
[![Domain - ad.beginningdimension.com](https://img.shields.io/badge/Domain-ad.beginningdimension.com-002050?style=flat&logo=active-directory&logoColor=white)](#)
[![Security - Hardened Baseline](https://img.shields.io/badge/Security-Hardened_Baseline-2EA44F?style=flat&logo=shield&logoColor=white)](#)
[![Focus - IT_Administrator_Portfolio](https://img.shields.io/badge/Focus-IT_Administrator_Portfolio-8A2BE2?style=flat)](#)
<p align="center">
  <strong>Windows Server 2025 On-Premises Deployment</strong><br>
  Secure AD DS · DNS · Group Policy · Windows LAPS · Auditing · Backup · High Availability<br>
  Portfolio Project for IT Administrator.
</p>

---

## Executive Summary

This project documents the design, deployment, and administration of a secure, resilient on-premises
Microsoft Active Directory Domain Services (AD DS) environment built and validated in a hands-on lab.
The environment uses two Windows Server 2025 domain controllers with AD-integrated DNS, a Windows 11
domain-joined workstation, Active Directory Sites and Services, structured Organizational Units,
Group Policy security baselines, Windows LAPS, advanced security auditing, and System State backup
procedures — with routine validation built around command-line diagnostics
(`dcdiag`, `repadmin`, `nltest`, `w32tm`).

The design models a small organization with one headquarters site and a single routed subnet.
**DC01** hosts the forest root domain controller with the initial FSMO roles; **DC02** provides a
replica domain controller, DNS, and authentication redundancy; **CLIENT01** is a Windows 11
workstation joined to the domain. A **router** at `192.168.10.1` provides default
gateway and Ethernet 2 provided NAT/internet access, while DC02 and CLIENT01 operate on the isolated internal segment.

The build was not a straight-line run: real incidents — Kerberos clock-skew GPO failures, DNS
forwarder timeouts, stale multi-homed adapter records, and interface binding conflicts — were
diagnosed and resolved methodically. Every incident, root cause, and fix is documented in the
[Troubleshooting & Issues Encountered](docs/troubleshooting-log.md) log.

## Architecture & Topology

| Component | Detail |
|---|---|
| Forest / Domain | `ad.beginningdimension.com` |
| NetBIOS name | `BEGINNINGDIM` |
| AD site | `HQ` |
| Subnet | `192.168.10.0/24` |
| Reverse DNS zone | `10.168.192.in-addr.arpa` |
| Domain controllers | DC01 (192.168.10.10), DC02 (192.168.10.11) — both Global Catalog |
| Client | CLIENT01 (192.168.10.20), Windows 11 Enterprise |
| Gateway | router at 192.168.10.1 (NAT, default gateway) |
| Hypervisor | VirtualBox — VM appliance exports (`.ova`) used for point-in-time DC backups |
| Client DNS | 192.168.10.10 (preferred), 192.168.10.11 (alternate) |

```text
Internet
   |
Router
192.168.10.1
   |
------------------------------------------------
|                HQ AD Site                     |
|              192.168.10.0/24                  |
|                                               |
| DC01                          DC02            |
| Windows Server 2025           Windows Server 2025
| 192.168.10.10                 192.168.10.11    |
| AD DS / DNS / GC              AD DS / DNS / GC |
| Initial FSMO roles            Replica DC      |
|       <--- AD Replication --->                |
|                                               |
| CLIENT01                                      |
| Windows 11                                    |
| 192.168.10.20                                 |
| DNS: DC01 and DC02                            |
------------------------------------------------
```

The HQ site represents the organization's main on-premises LAN. AD sites are logical
representations of physical network topology; they use subnets, sites, and site links to guide
client service location and replication traffic
([Microsoft Learn — Designing the Site Topology](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/designing-the-site-topology)).

## Key Skills & Technologies Demonstrated

| Technical area | Skills demonstrated |
|---|---|
| Windows Server | Server installation, static addressing, role installation, patching, administration |
| Active Directory | Forest and domain deployment, domain controllers, OUs, users, groups, FSMO roles |
| DNS | AD-integrated DNS, forwarders, reverse lookup zones, SRV records, secure dynamic updates |
| High availability | Replica DC deployment, DNS redundancy, replication monitoring |
| Group Policy | Password policy, account lockout, baseline policies, OU targeting |
| Security | Least privilege, admin account separation, Windows LAPS, Protected Users testing, audit policy |
| Networking | IPv4 addressing, DNS client configuration, AD site/subnet mapping, routing |
| Troubleshooting | dcdiag, repadmin, nltest, w32tm, secure-channel repair, lockout investigation |
| Backup & recovery | System State backups, DSRM awareness, restore planning |
| Disaster recovery | Daily System State backups with `wbadmin`, documented DSRM credential handling, restore testing in isolated recovery environments, and VirtualBox appliance (`.ova`) exports before forest-level changes |
| SIEM integration (planned) | Advanced audit policy on domain controllers and key security event IDs, with a documented production path for forwarding DC security logs to a centralized SIEM (Splunk / Microsoft Sentinel) — the lab itself uses Event Viewer |
| Documentation | Network diagram, implementation steps, validation evidence, security checklist |



## Implementation Phases

The build is documented as 13 numbered phases:

| Phase | Document | Focus |
|---|---|---|
| 1 | [Configure DC01](docs/phase-01-configure-dc01.md) | Host identity, static addressing, connectivity |
| 2 | [Install AD DS & DNS on DC01](docs/phase-02-install-ad-ds-dns-dc01.md) | Role installation, forest promotion |
| 3 | [Configure AD DNS](docs/phase-03-configure-ad-dns.md) | Forwarders, reverse zone, validation |
| 4 | [Configure DC02](docs/phase-04-configure-dc02.md) | Replica DC, cross-pointed DNS, replication health |
| 5 | [Configure AD Site Topology](docs/phase-05-ad-site-topology.md) | HQ site, subnet mapping, site awareness |
| 6 | [Create Organizational Units](docs/phase-06-organizational-units.md) | OU tree, default location redirection |
| 7 | [Users, Groups & Delegation](docs/phase-07-users-groups-delegation.md) | Standard/privileged accounts, role-based groups, least privilege |
| 8 | [Join CLIENT01 to the Domain](docs/phase-08-join-client01.md) | Client domain join and validation |
| 9 | [Group Policy Security Baseline](docs/phase-09-gpo-security-baseline.md) | Password/lockout policy, workstation baseline |
| 10 | [Deploy Windows LAPS](docs/phase-10-windows-laps.md) | Schema extension, permissions, LAPS GPO, validation |
| 11 | [Security Auditing](docs/phase-11-security-auditing.md) | Advanced audit policy, key event IDs |
| 12 | [Backup & Recovery](docs/phase-12-backup-recovery.md) | Windows Server Backup, System State backups |
| 13 | [Health Checks & Troubleshooting](docs/phase-13-health-checks.md) | Daily checks, DNS/replication diagnostics, trust repair, lockouts |

Additional documentation:

| Document | Purpose |
|---|---|
| [Troubleshooting & Issues Encountered](docs/troubleshooting-log.md) | Real incidents, root causes, and resolutions from the build |
| [Validation Checklist](docs/validation-checklist.md) | Final project validation evidence |
| [Evidence Index (Screenshots 1–72)](docs/evidence-index.md) | Mapping of all screenshots to phases and steps |
| [References](docs/references.md) | Official Microsoft documentation, benchmarks, and tool references |

## Repository Structure

```text
enterprise-ad-hardening-lab/
├── README.md                          # This file — project overview and navigation
├── docs/
│   ├── phase-01-configure-dc01.md
│   ├── phase-02-install-ad-ds-dns-dc01.md
│   ├── phase-03-configure-ad-dns.md
│   ├── phase-04-configure-dc02.md
│   ├── phase-05-ad-site-topology.md
│   ├── phase-06-organizational-units.md
│   ├── phase-07-users-groups-delegation.md
│   ├── phase-08-join-client01.md
│   ├── phase-09-gpo-security-baseline.md
│   ├── phase-10-windows-laps.md
│   ├── phase-11-security-auditing.md
│   ├── phase-12-backup-recovery.md
│   ├── phase-13-health-checks.md
│   ├── troubleshooting-log.md          # Incident log from the build
│   ├── validation-checklist.md        # Final validation evidence
│   ├── evidence-index.md              # Screenshots mapping
│   └── references.md                  # Formal bibliography
└── screenshots/
    └── phases 1-12 /                   # Screenshots (see evidence-index.md)
        └── README.md                 # Naming convention and contribution notes
```

## About the Screenshots

The build is documented by 12-phase numbered screenshots captured during execution. Because the
screenshot files are added to the repository as a batch, each phase document references its
screenshots by phases. The full mapping — screenshot number, phase, step, and caption — lives in the
[Evidence Index](docs/evidence-index.md), 

## Security & Sanitization Notice

No real passwords, DSRM passwords, LAPS secrets, recovery keys, or confidential information are
published in this repository. 

---

Portfolio contact and full reference list: see [References](docs/references.md).
