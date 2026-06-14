# Lab Overview

ADBox is a Windows identity and support administration lab for practising Active Directory-based user, device, policy, and remote support workflows.

The lab runs on Oracle VirtualBox (VBox), Windows Server, and Windows 10 clients connected through a shared home network. It covers Domain Controller (DC) promotion, Domain Name System (DNS), Dynamic Host Configuration Protocol (DHCP), Internet Protocol version 4 (IPv4), Transmission Control Protocol/Internet Protocol (TCP/IP), Remote Desktop Protocol (RDP), and the supporting services used by Windows domain environments.

The aim is to build defendable evidence of practical Windows support work: preparing a domain, joining clients, organising users and computers, applying Group Policy (GP), testing remote access, mapping resources, recovering accounts, documenting faults, and using PowerShell for repeatable administration.

Each stage records the configuration, validation checks, issues found, screenshots captured, and confirmed working state.

## Lab Topology

The lab runs across multiple physical laptops using VBox virtual machines (VMs) connected through the same home Wi-Fi network. `AD-SRV01` provides domain and DNS services for `adbox.local`, while the Windows 10 clients are used to test domain join, user logon, GP, remote access, and workstation administration.

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

`AD-SRV01` uses a static IP address so clients can reliably reach DNS and domain services. The Windows 10 clients receive their IP addresses from the home router, but their DNS settings point to `192.168.1.50` so they can resolve `adbox.local` through the DC.

The network printer remains part of the lab as a practical support scenario for printer discovery, client mapping, access validation, and planned GP deployment.

## Domain Design

The lab uses `adbox.local` as the internal Active Directory domain. This provides the identity boundary where users, computers, groups, Organisational Units (OUs), and GP settings are managed.

| Item                | Value         |
|                     |               |
| Full Domain Name    | `adbox.local` |
| NetBIOS Domain Name | `ADBOX`       |
| Domain Controller   | `AD-SRV01`    |

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

The lab uses VBox Bridged Adapter mode so each VM appears as a separate device on the same home network. This allows the server and clients to communicate across the shared Wi-Fi network while still running as isolated VMs on different physical laptops.

```text
Home Wi-Fi / EE Router
├── Physical Laptop: AD-SRV01
├── Physical Laptop: AD-WIN10-01
└── Physical Laptop: AD-WIN10-02
```

| Device        | Addressing                | DNS                                  |
|---------------|---------------------------|--------------------------------------|
| `AD-SRV01`    | Static IP: `192.168.1.50` | Points to itself: `192.168.1.50`     |
| `AD-WIN10-01` | Router DHCP               | Points to `AD-SRV01`: `192.168.1.50` |
| `AD-WIN10-02` | Router DHCP               | Points to `AD-SRV01`: `192.168.1.50` |

The router continues to provide DHCP addressing for the clients. DHCP was not enabled on `AD-SRV01` because the lab runs on a shared home network where the router already provides IP address assignment.

Internet Protocol version 6 (IPv6) was disabled on the Windows 10 lab adapters after testing showed that the clients were receiving IPv6 DNS information from the EE router. With IPv6 enabled, `nslookup adbox.local` timed out because the clients were using the wrong DNS path for the lab domain. The full issue is documented in [`dns-ipv6-conflict.md`](../troubleshooting/01-dns-ipv6-conflict.md).

## Core Services

The lab separates responsibilities between the server, router, and Windows clients.

| Service | Provider | Role In The Lab |
|---|---|---|
| Active Directory Domain Services (AD DS) | `AD-SRV01` | Stores and manages users, computers, groups, OUs, and domain objects. |
| DC | `AD-SRV01` | Authenticates domain users and computers. |
| DNS | `AD-SRV01` | Resolves `adbox.local` and allows clients to locate domain services. |
| Global Catalog | `AD-SRV01` | Supports directory lookups for users, groups, computers, and AD objects. |
| DHCP | EE Router | Provides IP addresses to the Windows 10 clients on the home network. |
| RDP | Windows clients and server | Supports remote access testing and support workflows. |
| GP | `AD-SRV01` | Applies configuration to users and computers through linked policies. |

`AD-SRV01` handles identity, authentication, policy, and domain name resolution. The router handles normal home-network address assignment. The Windows 10 clients validate the environment from the workstation side.

## Evidence Chain

The lab is documented as a staged support workflow.

| Step | Stage | Evidence |
|---|
| 1 | Server Installation        | Windows Server installed in VBox.                                                | 
| 2 | Environment Setup          | Static server address, client DHCP, DNS path, and bridged networking documented. |
| 3 | DC Build                   | `AD-SRV01` promoted as the first DC for `adbox.local`.                           |
| 4 | Domain Join                | Windows 10 clients joined to `adbox.local`.                                      |
| 5 | Directory Structure        | Users, computers, OUs, and groups organised.                                     |
| 6 | GP Validation              | Policies applied and validated against users or computers.                       |
| 7 | RDP Support                | Remote access tested through controlled access rules.                            |
| 8 | Printer Mapping            | Printer discovery, mapping, and access validation documented.                    |
| 9 | Account Recovery           | Password reset, unlock, disable, and recovery tasks recorded.                    |
| 10 | PowerShell Administration | Repeatable administration tasks scripted and validated.                          |
| 11 | Troubleshooting           | Issues documented with discovery, actions, and resolution.                       |

The chain is dependency-led. The server must be reachable before DNS can be tested, DNS must work before clients can join the domain, and domain membership must exist before users, groups, policies, RDP, printer mapping, and account recovery scenarios can be properly validated.

## Repository Structure

The repository separates build records, troubleshooting notes, screenshots, scripts, and reference notes.

| Folder             | Purpose                                                       |
|--------------------|---------------------------------------------------------------|
| `lab/`             | Main build and validation reports.                            |
| `troubleshooting/` | Fault records with discovery, actions, and resolution.        |
| `scripts/`         | PowerShell scripts for repeatable administration tasks.       |
| `screenshots/`     | Evidence used by the lab and troubleshooting reports.         |
| `notes/`           | Command Prompt (CMD), PowerShell, and terminology references. |

## Troubleshooting Records

Troubleshooting records are stored separately from the main lab reports so the build path stays readable while real faults remain documented.

Current records are listed in [`troubleshooting-records.md`](../troubleshooting/troubleshooting-records.md).

| Record | Focus |
|---|---|
| [`01-dns-ipv6-conflict.md`](../troubleshooting/01-dns-ipv6-conflict.md) | DNS resolution, IPv4, IPv6, and router-provided DNS behaviour. |
| [`02-client-firewall-ping.md`](../troubleshooting/02-client-firewall-ping.md) | Windows Firewall, ICMP, and server-to-client ping testing. |

ADBox stays focused on technical administration: connectivity, DNS, domain join, policies, access, recovery, and validation. Ticket intake, user-facing support forms, and service desk workflow is handled separately in the [N3 ticketing lab](https://github.com/erwinmagielda/n3).

## Next Stage

[Stage 2: Environment Setup](02-environment-setup.md), which documents the physical host layout, VBox network mode, IP addressing, DNS configuration, and adapter settings used before building the DC.
