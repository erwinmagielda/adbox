# Lab Overview

ADBox is a Windows identity and support administration lab for practising Active Directory-based user, device, policy, and remote support workflows.

The lab is built around virtualised Windows infrastructure using Oracle VirtualBox, Windows Server, and Active Directory. Around that foundation, it covers Domain Controller (DC) promotion, Domain Name System (DNS), Dynamic Host Configuration Protocol (DHCP), Internet Protocol version 4 (IPv4), Transmission Control Protocol/Internet Protocol (TCP/IP), Remote Desktop Protocol (RDP), and the supporting services used by Windows domain environments.

ADBox then applies those systems to practical support administration: user accounts, security groups, Organisational Units (OUs), Group Policy (GP), workstation administration, printer mapping, account recovery, troubleshooting, and PowerShell administration.


## Purpose

The purpose of ADBox is to build defendable, hands-on evidence of Active Directory, Windows support, virtualisation, and network connectivity skills.

The lab shows how a Windows domain is prepared, how clients are connected to it, and how common support tasks are carried out through users, groups, policies, remote access, device mapping, and documented troubleshooting.

Each stage records the configuration, validation steps, issues encountered, and confirmed working state.


## Lab Topology

The lab runs across multiple physical laptops using VirtualBox virtual machines connected through the same home Wi-Fi network. `AD-SRV01` provides the domain and DNS services for `adbox.local`, while the Windows 10 clients are used to test domain join, user logon, GP, remote access, and workstation administration.

```text
Home Wi-Fi / EE Router
├── AD-SRV01
│   ├── Windows Server 2022
│   ├── Writable Domain Controller
│   ├── DNS Server
│   ├── Global Catalog
│   └── Static IP: 192.168.1.50
│
├── AD-WIN10-01
│   ├── Windows 10 Pro
│   ├── Domain Client
│   ├── Dynamic IP: 192.168.1.204
│   └── DNS: 192.168.1.50
│
└── AD-WIN10-02
    ├── Windows 10 Pro
    ├── Domain Client
    ├── Dynamic IP: 192.168.1.102
    └── DNS: 192.168.1.50
```

`AD-SRV01` is assigned a static IP address so clients can reliably use it for DNS and domain services. The Windows 10 clients receive their IP addresses from the home router, but their DNS settings point to `192.168.1.50` so they can resolve `adbox.local` through the DC.

The network printer remains part of the lab as a practical support scenario for printer discovery, client mapping, access validation, and planned GP deployment.


## Domain Design

The lab uses `adbox.local` as the internal Active Directory domain. This provides the identity boundary for the environment, where users, computers, groups, OUs, and GP settings are managed.

| Item | Value |
|---|---|
| Full Domain Name | `adbox.local` |
| NetBIOS Domain Name | `ADBOX` |
| Domain Controller | `AD-SRV01` |

`AD-SRV01` is promoted as the first DC for the domain. Its domain role is writable, meaning users, groups, computers, policies, and other directory objects can be created and managed from this server.

`AD-SRV01` also provides Active Directory-integrated DNS. This allows Windows clients to locate the domain, find the DC, authenticate users, and resolve internal records for `adbox.local`.

The domain supports the NetBIOS logon format:

```text
ADBOX\username
```

It also supports the User Principal Name (UPN) format:

```text
username@adbox.local
```


## Network Design

The lab uses VirtualBox Bridged Adapter mode so the virtual machines appear as separate devices on the same home network. This allows the server and clients to communicate across the shared Wi-Fi network while still running as isolated virtual machines on different physical laptops.

```text
Home Wi-Fi / EE Router
├── Physical laptop running AD-SRV01
├── Physical laptop running AD-WIN10-01
└── Physical laptop running AD-WIN10-02
```

`AD-SRV01` uses a static IPv4 address so the clients can reliably reach the DC and DNS server.

| Device | Addressing | DNS |
|---|---|---|
| `AD-SRV01` | Static IP: `192.168.1.50` | Points to itself: `192.168.1.50` |
| `AD-WIN10-01` | Router DHCP | Points to `AD-SRV01`: `192.168.1.50` |
| `AD-WIN10-02` | Router DHCP | Points to `AD-SRV01`: `192.168.1.50` |

The Windows 10 clients receive their IP addresses from the home router, but their DNS settings are manually pointed to `192.168.1.50`. This allows the clients to resolve `adbox.local` through the DC instead of using the router for domain name resolution.

Internet Protocol version 6 (IPv6) was disabled on the Windows 10 lab adapters after testing showed that the clients were receiving IPv6 DNS information from the EE router. With IPv6 enabled, `nslookup adbox.local` timed out because the clients were using the wrong DNS path for the lab domain. Disabling IPv6 kept the lab controlled through IPv4 and allowed `adbox.local` to resolve through `AD-SRV01`.

The home router continues to provide normal DHCP addressing for the clients. DHCP was not enabled on `AD-SRV01` because the lab runs on a shared home network where the router already provides IP address assignment.


## Core Services

This section summarises the main services used by the lab and where each responsibility sits.

| Service | Provider | Role In The Lab |
|---|---|---|
| Active Directory Domain Services (AD DS) | `AD-SRV01` | Stores and manages users, computers, groups, Organisational Units (OUs), and domain objects. |
| Domain Controller (DC) | `AD-SRV01` | Authenticates domain users and computers. |
| Domain Name System (DNS) | `AD-SRV01` | Resolves `adbox.local` and allows clients to locate domain services. |
| Global Catalog | `AD-SRV01` | Supports directory lookups for users, groups, computers, and other Active Directory objects. |
| Dynamic Host Configuration Protocol (DHCP) | EE Router | Provides IP addresses to the Windows 10 clients on the home network. |
| Remote Desktop Protocol (RDP) | Windows clients and server | Supports remote access testing and support workflows. |
| Group Policy (GP) | `AD-SRV01` | Applies configuration to users and computers through linked policies. |

This split keeps the lab responsibilities clear. `AD-SRV01` handles identity, authentication, policy, and domain name resolution. The router remains responsible for normal home-network address assignment, while the Windows 10 clients are used to validate the environment from the workstation side.


## Evidence Chain

The lab is documented as a step-by-step support workflow.

| Step | Stage | Evidence |
|---|---|---|
| 1 | Server installation | Windows Server installed in VirtualBox. |
| 2 | Server preparation | `AD-SRV01` renamed and assigned a static IP address. |
| 3 | Domain Controller promotion | Active Directory Domain Services (AD DS) installed and `AD-SRV01` promoted to a Domain Controller (DC). |
| 4 | Domain validation | `adbox.local` domain and Domain Name System (DNS) records confirmed. |
| 5 | Client DNS configuration | Windows 10 clients configured to use `192.168.1.50` for DNS. |
| 6 | Domain join | Windows 10 clients joined to `adbox.local`. |
| 7 | Directory organisation | Computers, users, groups, and Organisational Units (OUs) organised. |
| 8 | Policy validation | Group Policy (GP) settings applied and validated. |
| 9 | Support scenarios | Remote Desktop Protocol (RDP), printer mapping, account recovery, and workstation support scenarios tested. |
| 10 | PowerShell administration | PowerShell scripts added for repeatable administration. |
| 11 | Troubleshooting records | Issues found during the build documented with discovery, actions, and resolution. |

The chain shows how each part of the environment depends on the previous stage. The server must be reachable before DNS can be tested, DNS must work before clients can join the domain, and domain membership must be in place before users, groups, policies, RDP, printer mapping, and account recovery scenarios can be properly validated.


## Structured Documentation

The repository is organised so the build notes, troubleshooting records, scripts, screenshots, and learning references stay separate.

| Folder | Purpose |
|---|---|
| `lab/` | Structured build notes and support workflow documentation. |
| `troubleshooting/` | Issues discovered during the lab, with discovery, actions, and resolution. |
| `scripts/` | PowerShell scripts used for repeatable administration tasks. |
| `screenshots/` | Visual evidence of configuration, validation, and results. |
| `notes/` | Command Prompt (CMD), PowerShell, and terminology references used while learning. |

This keeps the main lab documentation readable while still preserving the supporting evidence. Build steps stay in `lab/`, fault-finding stays in `troubleshooting/`, reusable administration work goes into `scripts/`, and screenshots provide visual proof of configuration and validation.


## Troubleshooting Notes

Troubleshooting notes use three sections:

- **Discovery**  
  Records what was observed, what failed, and what evidence showed the issue existed.

- **Actions**  
  Records the checks, commands, configuration changes, comparisons, and tests used to narrow down the cause.

- **Resolution**  
  Records what fixed the issue and how the working state was confirmed.

This keeps ADBox focused on the technical administration side of the work: connectivity, DNS, domain join, policies, access, recovery, and validation. Ticket intake, ticket forms, user-facing support records, and service desk workflow will be handled separately in the planned [N3 ticketing lab](https://github.com/erwinmagielda/n3).