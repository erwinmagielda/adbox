# Lab Overview

ADBox starts with a simple design problem: one server, two Windows clients, and a home network that needs to behave like a small Windows domain lab.

This page explains the shape of the lab before the build begins. It covers the physical layout, domain naming, network plan, and the role each machine plays.

## Physical Layout

The lab is split across multiple physical laptops. Each virtual machine runs locally on its assigned host, while all machines communicate through the same home Wi-Fi network.

```text
Home Wi-Fi / EE Router
│
├── AD-SRV01
│   ├── Windows Server 2022
│   ├── Domain Controller
│   ├── DNS Server
│   ├── Global Catalog
│   └── Static IP: 192.168.1.50
│
├── AD-WIN10-01
│   ├── Windows 10 Pro
│   ├── Domain Client
│   ├── Router DHCP
│   └── DNS: 192.168.1.50
│
└── AD-WIN10-02
    ├── Windows 10 Pro
    ├── Domain Client
    ├── Router DHCP
    └── DNS: 192.168.1.50
```

This layout keeps the lab close to a small networked Windows environment: separate machines, shared local network, one server providing domain services, and clients validating the setup from the workstation side.

## Domain Plan

The domain plan gives the lab a consistent identity boundary for users, computers, groups, policies, and access testing.

| Area                  | Design                                                 |
| --------------------- | ------------------------------------------------------ |
| Full Domain Name      | `adbox.local`                                          |
| NetBIOS Domain Name   | `ADBOX`                                                |
| Domain Controller     | `AD-SRV01`                                             |
| Server Role           | Writable Domain Controller, DNS Server, Global Catalog |
| Client Machines       | `AD-WIN10-01`, `AD-WIN10-02`                           |

The NetBIOS domain name is the short Windows name for the domain. In this lab, `ADBOX` is the short name used for the full domain `adbox.local`.

Both names can be used during sign-in, but they appear in different formats:

| Format              | Example                  | Meaning                                          |
| ------------------- | ------------------------ | ------------------------------------------------ |
| NetBIOS logon name  | `ADBOX\sam.taylor`       | Uses the short Windows domain name.              |
| User Principal Name | `sam.taylor@adbox.local` | Uses the full domain name in email-style format. |

## Network Plan

The virtual machines use VirtualBox Bridged Adapter mode so each VM appears as its own device on the home network.

| Machine       | Addressing                                 | DNS                                  |
| ------------- | ------------------------------------------ | ------------------------------------ |
| `AD-SRV01`    | Static IPv4: `192.168.1.50`                | Points to itself: `192.168.1.50`     |
| `AD-WIN10-01` | Router DHCP                                | Points to `AD-SRV01`: `192.168.1.50` |
| `AD-WIN10-02` | Router DHCP                                | Points to `AD-SRV01`: `192.168.1.50` |
| EE Router     | Gateway and DHCP provider: `192.168.1.254` | Provides normal home-network access  |

The main design choice is that the router handles client IP addressing, while `AD-SRV01` handles DNS for the lab domain. That gives the clients a stable path to `adbox.local` without turning the home router into part of the Active Directory setup.

## Service Roles

Each part of the lab has a clear job.

| Service                          | Handled By                   | Used For                                                               |
| -------------------------------- | ---------------------------- | ---------------------------------------------------------------------- |
| Active Directory Domain Services | `AD-SRV01`                   | Users, computers, groups, OUs, and domain objects.                     |
| Authentication                   | `AD-SRV01`                   | Domain user and computer sign-in.                                      |
| DNS                              | `AD-SRV01`                   | Resolving `adbox.local` and locating domain services.                  |
| Global Catalog                   | `AD-SRV01`                   | Directory lookups inside the domain.                                   |
| DHCP                             | EE Router                    | Client IP addressing on the home network.                              |
| Client Testing                   | `AD-WIN10-01`, `AD-WIN10-02` | Domain join, sign-in, policy, RDP, file access, and account behaviour. |

## Design Notes

`AD-SRV01` uses a static IPv4 address because the clients need a reliable DNS and domain target.

The Windows 10 clients use router DHCP for their IP addresses, but their DNS points to `AD-SRV01`. This keeps domain lookup traffic on the lab path.

The clients are tested from the workstation side because that is where most support issues become visible: failed sign-in, missing policy, broken name resolution, blocked access, or incorrect permissions.

## Navigation

| Previous                     | Current         | Next                                            |
| ---------------------------- | --------------- | ----------------------------------------------- |
[Project README](../README.md) | 01 Lab Overview | [02 Environment Setup](02-environment-setup.md) |
