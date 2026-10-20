# Evidence Index — Screenshots 1–72

This index maps all 72 numbered screenshots from the lab build to their phase, step, and intended
asset path. Because the image files live in `assets/screenshots/` and are added as a batch, phase
documents reference screenshots by number and path rather than embedding them inline.

> **Status note — proposed mapping:** the number-to-step allocations and captions below are a
> proposed mapping derived from the logical execution sequence of the build. The image files were
> not available for review when this index was written, so every pairing must be confirmed
> against the actual images when they are added to `assets/screenshots/` — correct any mismatch
> before publication.

## Naming Convention

- File name: `screenshot-NN.png` where `NN` is the zero-padded screenshot number (01–72).
- Path: `assets/screenshots/screenshot-NN.png`
- All screenshots must be sanitized: no passwords, DSRM secrets, LAPS secrets, or real public
  addresses. See [assets/screenshots/README.md](../assets/screenshots/README.md).

## Phase 1 — Configure DC01

*Screenshots 01–05*

| # | Phase step | Proposed caption (verify against image) | Asset path |
|---|---|---|---|
| 01 | Rename DC01 | Server Manager > Local Server showing computer rename to DC01 | `assets/screenshots/screenshot-01.png` |
| 02 | Static IP | IPv4 properties with static address 192.168.10.10 and gateway 192.168.10.1 | `assets/screenshots/screenshot-02.png` |
| 03 | Validate connectivity | `ipconfig /all` output confirming static addressing on DC01 | `assets/screenshots/screenshot-03.png` |
| 04 | Validate connectivity | `ping 192.168.10.1` validating gateway connectivity | `assets/screenshots/screenshot-04.png` |
| 05 | Validate connectivity | Windows Update and date/time verification on DC01 | `assets/screenshots/screenshot-05.png` |

## Phase 2 — Install AD DS and DNS on DC01

*Screenshots 06–12*

| # | Phase step | Proposed caption (verify against image) | Asset path |
|---|---|---|---|
| 06 | GUI installation | Add Roles and Features — AD DS role selection on DC01 | `assets/screenshots/screenshot-06.png` |
| 07 | GUI installation | DNS Server role selection with Add Features prompt | `assets/screenshots/screenshot-07.png` |
| 08 | GUI installation | Installation progress and confirmation page | `assets/screenshots/screenshot-08.png` |
| 09 | Promote DC01 | Server Manager notification flag — Promote this server to a domain controller | `assets/screenshots/screenshot-09.png` |
| 10 | Promote DC01 | Deployment Configuration — Add a new forest, ad.beginningdimension.com | `assets/screenshots/screenshot-10.png` |
| 11 | Promote DC01 | Domain Controller Options — DNS Server, Global Catalog, DSRM password | `assets/screenshots/screenshot-11.png` |
| 12 | Promote DC01 | Prerequisites check passed and promotion completed; sign-in as BEGINNINGDIM\Administrator | `assets/screenshots/screenshot-12.png` |

## Phase 3 — Configure AD DNS

*Screenshots 13–18*

| # | Phase step | Proposed caption (verify against image) | Asset path |
|---|---|---|---|
| 13 | DNS forwarders | DNS Manager Forwarders tab with 1.1.1.1 and 8.8.8.8 | `assets/screenshots/screenshot-13.png` |
| 14 | Reverse lookup zone | New Zone wizard — reverse lookup primary zone | `assets/screenshots/screenshot-14.png` |
| 15 | Reverse lookup zone | AD-integrated zone replication scope selection | `assets/screenshots/screenshot-15.png` |
| 16 | Reverse lookup zone | IPv4 Reverse Lookup Zone 192.168.10 with secure dynamic updates | `assets/screenshots/screenshot-16.png` |
| 17 | Validate DNS | `nslookup -type=SRV` query output locating DC01 | `assets/screenshots/screenshot-17.png` |
| 18 | Validate DNS | `dcdiag /test:dns` passing all critical tests | `assets/screenshots/screenshot-18.png` |

## Phase 4 — Configure DC02

*Screenshots 19–27*

| # | Phase step | Proposed caption (verify against image) | Asset path |
|---|---|---|---|
| 19 | Server identity | Server Manager computer rename to DC02 | `assets/screenshots/screenshot-19.png` |
| 20 | Networking | DC02 IPv4 static configuration (192.168.10.11, DNS 192.168.10.10) | `assets/screenshots/screenshot-20.png` |
| 21 | Networking | `ping` and `nslookup` from DC02 to DC01 | `assets/screenshots/screenshot-21.png` |
| 22 | Domain join | System Properties — joining ad.beginningdimension.com | `assets/screenshots/screenshot-22.png` |
| 23 | Domain join | "Welcome to the domain" confirmation prompt | `assets/screenshots/screenshot-23.png` |
| 24 | Install AD DS and DNS | AD DS and DNS Server role installation on DC02 | `assets/screenshots/screenshot-24.png` |
| 25 | Promote DC02 | Promotion — Add a domain controller to an existing domain | `assets/screenshots/screenshot-25.png` |
| 26 | Promote DC02 | DC02 Domain Controller Options — DNS Server, Global Catalog, DSRM | `assets/screenshots/screenshot-26.png` |
| 27 | Validate DC health | `repadmin /replsummary` healthy replication with cross-pointed DNS | `assets/screenshots/screenshot-27.png` |

## Phase 5 — Configure AD Site Topology

*Screenshots 28–32*

| # | Phase step | Proposed caption (verify against image) | Asset path |
|---|---|---|---|
| 28 | Create HQ site | Active Directory Sites and Services — New Site HQ | `assets/screenshots/screenshot-28.png` |
| 29 | Associate subnet | New Subnet 192.168.10.0/24 associated with HQ | `assets/screenshots/screenshot-29.png` |
| 30 | Move DCs | Moving DC01 from Default-First-Site-Name to HQ | `assets/screenshots/screenshot-30.png` |
| 31 | Move DCs | HQ site containing DC01 and DC02 | `assets/screenshots/screenshot-31.png` |
| 32 | Validate site awareness | `nltest /dsgetsite` returning HQ on all systems | `assets/screenshots/screenshot-32.png` |

## Phase 6 — Create Organizational Units

*Screenshots 33–36*

| # | Phase step | Proposed caption (verify against image) | Asset path |
|---|---|---|---|
| 33 | OU design | ADUC showing the completed Beginningdimension OU tree | `assets/screenshots/screenshot-33.png` |
| 34 | Create OUs | Creating the top-level OU via the New Object wizard | `assets/screenshots/screenshot-34.png` |
| 35 | Redirect defaults | `redircmp` / `redirusr` command output on DC01 | `assets/screenshots/screenshot-35.png` |
| 36 | Redirect defaults | New computer object landing in the Workstations OU after redirection | `assets/screenshots/screenshot-36.png` |

## Phase 7 — User Accounts, Groups, and Delegation

*Screenshots 37–42*

| # | Phase step | Proposed caption (verify against image) | Asset path |
|---|---|---|---|
| 37 | Standard account | jdoe standard user account properties in the Users OU | `assets/screenshots/screenshot-37.png` |
| 38 | Privileged account | adm_jdoe privileged account in the Admin > Tier0 OU | `assets/screenshots/screenshot-38.png` |
| 39 | Security groups | GG_* security groups created in the SecurityGroups OU | `assets/screenshots/screenshot-39.png` |
| 40 | Security groups | adm_jdoe membership in GG_Tier0_Admins | `assets/screenshots/screenshot-40.png` |
| 41 | Delegation | Delegation of Control wizard targeting GG_IT_Helpdesk | `assets/screenshots/screenshot-41.png` |
| 42 | Delegation | Delegated task summary — password reset permissions | `assets/screenshots/screenshot-42.png` |

## Phase 8 — Join CLIENT01 to the Domain

*Screenshots 43–47*

| # | Phase step | Proposed caption (verify against image) | Asset path |
|---|---|---|---|
| 43 | Network config | CLIENT01 IPv4 configuration pointing at DC01/DC02 DNS | `assets/screenshots/screenshot-43.png` |
| 44 | Domain join | Domain join dialog with credentials for ad.beginningdimension.com | `assets/screenshots/screenshot-44.png` |
| 45 | Validate join | Sign-in as BEGINNINGDIM\jdoe with `whoami` output | `assets/screenshots/screenshot-45.png` |
| 46 | Validate join | `nltest /dsgetdc:` and `%logonserver%` validation output | `assets/screenshots/screenshot-46.png` |
| 47 | Validate join | CLIENT01 computer object in the Workstations OU with `gpresult /r` | `assets/screenshots/screenshot-47.png` |

## Phase 9 — Group Policy Security Baseline

*Screenshots 48–53*

| # | Phase step | Proposed caption (verify against image) | Asset path |
|---|---|---|---|
| 48 | GPO design | Group Policy Management showing the six-GPO baseline design | `assets/screenshots/screenshot-48.png` |
| 49 | Password policy | Default Domain Policy — password policy settings | `assets/screenshots/screenshot-49.png` |
| 50 | Lockout policy | Account lockout policy settings in Default Domain Policy | `assets/screenshots/screenshot-50.png` |
| 51 | Workstation baseline | Sec-Workstation-Baseline — Defender and Firewall settings | `assets/screenshots/screenshot-51.png` |
| 52 | DC baseline | Sec-DomainControllers-Baseline linked to the Domain Controllers OU | `assets/screenshots/screenshot-52.png` |
| 53 | Validation | `gpresult /r` on CLIENT01 showing applied baseline GPOs | `assets/screenshots/screenshot-53.png` |

## Phase 10 — Deploy Windows LAPS

*Screenshots 54–60*

| # | Phase step | Proposed caption (verify against image) | Asset path |
|---|---|---|---|
| 54 | Extend schema | `Update-LapsADSchema` execution and confirmation output | `assets/screenshots/screenshot-54.png` |
| 55 | Self-permission | `Set-LapsADComputerSelfPermission` output for the Workstations OU | `assets/screenshots/screenshot-55.png` |
| 56 | Read permission | `Set-LapsADReadPasswordPermission` output for GG_LAPS_Password_Readers | `assets/screenshots/screenshot-56.png` |
| 57 | LAPS GPO | `Cfg-WindowsLAPS` GPO created and linked to the Workstations OU | `assets/screenshots/screenshot-57.png` |
| 58 | LAPS GPO | LAPS policy settings — backup directory, length, complexity, age | `assets/screenshots/screenshot-58.png` |
| 59 | Test LAPS | `Get-LapsADPassword` output on CLIENT01 (sanitized, no password shown) | `assets/screenshots/screenshot-59.png` |
| 60 | Test LAPS | Authorized password decryptors configured as GG_LAPS_Password_Readers | `assets/screenshots/screenshot-60.png` |

## Phase 11 — Security Auditing

*Screenshots 61–64*

| # | Phase step | Proposed caption (verify against image) | Asset path |
|---|---|---|---|
| 61 | Auditing GPO | Creating and linking Sec-DomainControllers-Auditing to the DC OU | `assets/screenshots/screenshot-61.png` |
| 62 | Audit categories | Advanced Audit Policy categories enabled for Success and Failure | `assets/screenshots/screenshot-62.png` |
| 63 | Force subcategories | "Force audit policy subcategory settings" override enabled | `assets/screenshots/screenshot-63.png` |
| 64 | Event monitoring | Event Viewer Security log showing key event IDs (4625, 4720, 4769) | `assets/screenshots/screenshot-64.png` |

## Phase 12 — Backup and Recovery

*Screenshots 65–67*

| # | Phase step | Proposed caption (verify against image) | Asset path |
|---|---|---|---|
| 65 | Install WSB | Windows Server Backup feature installation on DC01/DC02 | `assets/screenshots/screenshot-65.png` |
| 66 | System State backup | `wbadmin start systemstatebackup` execution output | `assets/screenshots/screenshot-66.png` |
| 67 | System State backup | System State backup completion confirmation | `assets/screenshots/screenshot-67.png` |

## Phase 13 — Health Checks and Troubleshooting

*Screenshots 68–72*

| # | Phase step | Proposed caption (verify against image) | Asset path |
|---|---|---|---|
| 68 | Daily checks | `dcdiag /q` clean output on the domain | `assets/screenshots/screenshot-68.png` |
| 69 | Daily checks | `repadmin /replsummary` with no replication failures | `assets/screenshots/screenshot-69.png` |
| 70 | FSMO check | `netdom query fsmo` showing all roles on DC01 | `assets/screenshots/screenshot-70.png` |
| 71 | Time status | `w32tm /query /status` on DC01, DC02, and CLIENT01 | `assets/screenshots/screenshot-71.png` |
| 72 | Trust repair | `Test-ComputerSecureChannel -Repair` and final validation on CLIENT01 | `assets/screenshots/screenshot-72.png` |

---

Back to: [README](../README.md)
