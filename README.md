# ADBox

ADBox is a Windows Server and Active Directory support lab focused on practical domain administration, client management, Group Policy, Remote Desktop access, troubleshooting, and support documentation.

The lab runs across Oracle VirtualBox (VBox) Virtual Machines (VMs) on multiple physical laptops connected through the same home network. It uses Windows Server 2022 as the Domain Controller (DC) for `adbox.local` and Windows 10 Pro clients as domain-joined workstations.

ADBox demonstrates a practical support workflow: preparing a server, building a domain, joining clients, organising users and computers, applying policy, testing remote access, documenting faults, and recording evidence through Markdown reports and screenshots.

## Skills Demonstrated

Each skill is backed by a working configuration, validation step, screenshot, or troubleshooting record.

| Skill                                    | Demonstration                                                                                                 |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| Virtualisation                           | Built the lab with VBox VMs across multiple physical laptops.                                                 |
| Windows Server Administration            | Prepared and configured `AD-SRV01` as the main Windows Server system for the lab.                             |
| Active Directory Domain Services (AD DS) | Installed AD DS and promoted `AD-SRV01` as the first DC for `adbox.local`.                                    |
| Domain Name System (DNS)                 | Configured clients to use `AD-SRV01` for domain resolution and validated lookups with `nslookup`.             |
| Network Configuration                    | Used bridged networking, static server addressing, router DHCP, and IPv4 validation across the lab.           |
| Windows Client Administration            | Joined Windows 10 clients to the domain and confirmed their computer objects in Active Directory.             |
| Directory Administration                 | Created an Organisational Unit (OU) structure for users, computers, groups, and future service accounts.      |
| User Administration                      | Created department users with User Principal Name (UPN) logon formats and password-change requirements.       |
| Security Group Administration            | Created Global Security groups and tested group membership changes for access-control preparation.            |
| Group Policy (GP)                        | Linked a workstation GPO, applied a logon notice, refreshed policy, and validated the result with `gpresult`. |
| Remote Desktop Protocol (RDP)            | Enabled RDP on a domain-joined workstation and validated a remote session with Command Prompt (CMD).          |
| Troubleshooting                          | Documented DNS and firewall issues using discovery, actions, and resolution records.                          |
| Technical Documentation                  | Recorded each stage through Markdown reports, screenshots, command output, and clear evidence chains.         |

## Lab Environment

ADBox uses a small Windows domain environment built on a home network. `AD-SRV01` uses a static Internet Protocol version 4 (IPv4) address so clients have a stable target for domain services. The EE router provides client addressing through Dynamic Host Configuration Protocol (DHCP).

The Windows clients use `AD-SRV01` for DNS. This allows them to resolve `adbox.local`, locate the DC, join the domain, and receive domain-based configuration.

| Component               | Details                                 |
| ----------------------- | --------------------------------------- |
| Domain                  | `adbox.local`                           |
| NetBIOS Domain          | `ADBOX`                                 |
| DC                      | `AD-SRV01`                              |
| Server Operating System | Windows Server 2022 Standard Evaluation |
| Client Operating System | Windows 10 Pro                          |
| Clients                 | `AD-WIN10-01`, `AD-WIN10-02`            |
| Server IPv4 Address     | `192.168.1.50`                          |
| Gateway                 | `192.168.1.254`                         |
| DHCP Provider           | EE Router                               |
| DNS Provider            | `AD-SRV01`                              |
| Virtualisation          | VBox                                    |
| Network Mode            | Bridged Adapter                         |

## Repository Structure

The repository separates lab reports, screenshots, notes, troubleshooting records, and PowerShell script work. This keeps each stage reviewable and keeps the evidence close to the relevant report.

The `lab/` folder contains the staged build records. The `troubleshooting/` folder contains fault records written around discovery, action, and resolution.

| Folder             | Purpose                                                 |
| ------------------ | ------------------------------------------------------- |
| `lab/`             | Main build and validation reports.                      |
| `troubleshooting/` | Fault records with discovery, actions, and resolution.  |
| `scripts/`         | PowerShell scripts for repeatable administration tasks. |
| `screenshots/`     | Evidence used by the lab and troubleshooting reports.   |
| `notes/`           | CMD, PowerShell, and terminology references.            |

## Lab Contents

The lab is written as a staged build. Each report introduces one practical capability, validates it, and stores evidence that can be reviewed later.

The stages follow the order of a working domain build: prepare the environment, build the DC, join clients, organise Active Directory, apply GP, and validate RDP access.

| Stage | Report                                               | Focus                                                                                                        |
| ----- | ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| 1     | [Lab Overview](lab/01-lab-overview.md)               | Technical design, topology, domain plan, network layout, and evidence model.                                 |
| 2     | [Environment Setup](lab/02-environment-setup.md)     | VBox networking, static server addressing, client DHCP, and DNS path.                                        |
| 3     | [Domain Controller](lab/03-domain-controller.md)     | AD DS installation, forest creation, DC promotion, DNS role validation, and ADUC checks.                     |
| 4     | [Domain Join](lab/04-domain-join.md)                 | Windows 10 client domain join, domain sign-in, and AD computer object validation.                            |
| 5     | [Directory Structure](lab/05-directory-structure.md) | OU design, user accounts, security groups, computer placement, and group membership.                         |
| 6     | [Group Policy](lab/06-group-policy.md)               | Workstation GPO creation, logon notice configuration, `gpupdate`, `gpresult`, and visible client validation. |
| 7     | [Remote Desktop Access](lab/07-remote-desktop.md)    | RDP enablement, security group access, remote connection, and session identity validation.                   |

## Evidence Chain

ADBox follows a practical support administration sequence. Each stage builds on the previous one and creates evidence through configuration screenshots, validation commands, or troubleshooting records.

```text
Environment Setup → Domain Controller → Domain Join → Directory Structure → Group Policy → Remote Desktop Access
```

Screenshots are stored under matching lab folders so the evidence follows the same order as the report.

Example:

```text
screenshots/lab/06-group-policy/
├── 01-logon-policy-configured.png
├── 02-client-gpupdate-success.png
├── 03-gpresult-client-summary.png
├── 04-gpresult-computer-applied.png
├── 05-gpresult-user-settings.png
└── 06-logon-message-shown.png
```

## Troubleshooting Records

Troubleshooting records document faults found during the lab build. Each record captures the issue, the checks performed, the fix applied, and the confirmed working state.

| Record                                                                      | Focus                                                                                                              |
| --------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| [DNS IPv6 Conflict](troubleshooting/01-dns-ipv6-conflict.md)                | Client DNS lookups bypassing AD DNS because of router-provided Internet Protocol version 6 (IPv6) DNS information. |
| [Client Firewall Ping](troubleshooting/02-client-firewall-ping.md)          | Server-to-client ping blocked by Windows Firewall inbound ICMP rules.                                              |
| [Troubleshooting Records Index](troubleshooting/troubleshooting-records.md) | Index of recorded lab faults and fixes.                                                                            |

## Current Status

ADBox is a functional lab implementation.

The current build includes a working Windows Server DC, two Windows 10 domain-joined clients, a structured Active Directory OU layout, department users, security groups, a tested workstation GP configuration, and a validated RDP access workflow.

The lab is documented through Markdown reports, screenshots, command output, and troubleshooting records.

## Lab Scope

ADBox runs on a home-network lab using Windows Server, Windows 10 clients, VBox, router DHCP, and staged Markdown documentation.

The configuration choices keep the lab understandable and reviewable: one DC, two Windows clients, a clear OU structure, controlled security groups, visible GP testing, and RDP validation.

## Start Here

Begin with [Stage 1: Lab Overview](lab/01-lab-overview.md), which explains the ADBox technical design, domain plan, network layout, and evidence model.

## Licence

This project is provided for learning, documentation, and portfolio demonstration purposes.
