# Lab Overview

ADBox is a Windows identity and support administration lab for practising Active Directory-based user, device, policy, and remote support workflows.

This overview explains the technical design behind the lab. It covers the domain layout, network design, core services, evidence model, and troubleshooting approach used before the individual build stages begin.

The lab runs on Oracle VirtualBox (VBox), Windows Server, and Windows 10 clients connected through a shared home network. It uses `AD-SRV01` as the Domain Controller (DC) for `adbox.local`, with Windows 10 clients used to validate domain join, Group Policy (GP), Remote Desktop Protocol (RDP), and workstation administration from the client side.

## Lab Topology

The lab runs across multiple physical laptops using VBox virtual machines (VMs). All machines connect through the same home Wi-Fi network, which allows the server and clients to communicate like separate devices on one local network.

`AD-SRV01` provides domain and Domain Name System (DNS) services for `adbox.local`. The Windows 10 clients are used to test domain join, domain logon, GP, RDP, and workstation administration.

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

`AD-SRV01` uses a static Internet Protocol version 4 (IPv4) address so clients can reliably reach DNS and domain services. The Windows 10 clients receive their IPv4 addresses from the home router, but their DNS settings point to `192.168.1.50` so they can resolve `adbox.local` through the DC.

## Domain Design

The lab uses `adbox.local` as the internal Active Directory domain. This provides the identity boundary where users, computers, groups, Organisational Units (OUs), and GP settings are managed.

| Item                | Value         |
| ------------------- | ------------- |
| Full Domain Name    | `adbox.local` |
| NetBIOS Domain Name | `ADBOX`       |
| DC                  | `AD-SRV01`    |

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

The lab uses VBox Bridged Adapter mode so each VM appears as a separate device on the same home network. This keeps the setup close to a normal small network while still running the systems as isolated VMs.

The server and clients are split across different physical laptops, but they communicate through the same router and wireless network.

```text
Home Wi-Fi / EE Router
├── Physical Laptop: AD-SRV01
├── Physical Laptop: AD-WIN10-01
└── Physical Laptop: AD-WIN10-02
```

| Device        | Addressing                | DNS                                  |
| ------------- | ------------------------- | ------------------------------------ |
| `AD-SRV01`    | Static IP: `192.168.1.50` | Points to itself: `192.168.1.50`     |
| `AD-WIN10-01` | Router DHCP               | Points to `AD-SRV01`: `192.168.1.50` |
| `AD-WIN10-02` | Router DHCP               | Points to `AD-SRV01`: `192.168.1.50` |

The router continues to provide Dynamic Host Configuration Protocol (DHCP) addressing for the clients. DHCP was not enabled on `AD-SRV01` because the lab runs on a shared home network where the router already provides IP address assignment.

Internet Protocol version 6 (IPv6) was disabled on the Windows 10 lab adapters after testing showed that the clients were receiving IPv6 DNS information from the EE router. With IPv6 enabled, `nslookup adbox.local` timed out because the clients were using the wrong DNS path for the lab domain. The full issue is documented in [DNS IPv6 Conflict](../troubleshooting/01-dns-ipv6-conflict.md).

## Core Services

The lab separates responsibilities between the server, router, and Windows clients. This keeps the design simple enough to troubleshoot while still showing how core domain services fit together.

`AD-SRV01` handles identity, authentication, policy, and domain name resolution. The router handles normal home-network address assignment. The Windows 10 clients validate the environment from the workstation side.

| Service | Provider | Role In The Lab |
|---|---|---|
| Active Directory Domain Services (AD DS) | `AD-SRV01` | Stores and manages users, computers, groups, OUs, and domain objects. |
| DC | `AD-SRV01` | Authenticates domain users and computers. |
| DNS | `AD-SRV01` | Resolves `adbox.local` and allows clients to locate domain services. |
| Global Catalog | `AD-SRV01` | Supports directory lookups for users, groups, computers, and AD objects. |
| DHCP | EE Router | Provides IP addresses to the Windows 10 clients on the home network. |
| RDP | Windows clients and server | Supports remote access testing and support workflows. |
| GP | `AD-SRV01` | Applies configuration to users and computers through linked policies. |

## Evidence Model

Each lab stage records the configuration, validation checks, screenshots, issues found, and confirmed working state. The goal is not just to show that something was configured, but to show how it was checked from the server or client side.

The build is dependency-led. The server must be reachable before DNS can be tested, DNS must work before clients can join the domain, and domain membership must exist before users, groups, policies, and RDP can be properly validated.

```text
Environment Setup → Domain Controller → Domain Join → Directory Structure → Group Policy → Remote Desktop Access
```

Screenshots are stored under matching folders so each report can be reviewed alongside its evidence.

```text
screenshots/lab/
├── 02-environment-setup/
├── 03-domain-controller/
├── 04-domain-join/
├── 05-directory-structure/
├── 06-group-policy/
└── 07-remote-desktop/
```

## Troubleshooting Approach

Troubleshooting records are stored separately from the main lab reports so the build path stays readable while real faults remain documented.

Each record uses the same structure:

| Section    | Purpose                                                                       |
| ---------- | ----------------------------------------------------------------------------- |
| Discovery  | What was observed, what failed, and what evidence showed the issue existed.   |
| Actions    | Checks, commands, configuration changes, and tests used to isolate the cause. |
| Resolution | What fixed the issue and how the working state was confirmed.                 |

Current records are listed in [Troubleshooting Records](../troubleshooting/troubleshooting-records.md).

| Record | Focus |
|---|---|
| [DNS IPv6 Conflict](../troubleshooting/01-dns-ipv6-conflict.md) | DNS resolution, IPv4, IPv6, and router-provided DNS behaviour. |
| [Client Firewall Ping](../troubleshooting/02-client-firewall-ping.md) | Windows Firewall, Internet Control Message Protocol (ICMP), and server-to-client ping testing. |

ADBox stays focused on technical administration: connectivity, DNS, domain join, policies, access, recovery, and validation. Ticket intake, user-facing support forms, and service desk workflow are handled separately in the [N3 ticketing lab](https://github.com/erwinmagielda/n3).

## Next Stage

[Stage 2: Environment Setup](02-environment-setup.md), which documents the physical host layout, VBox network mode, IP addressing, DNS configuration, and adapter settings used before building the DC.
