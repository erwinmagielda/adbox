# ADBox

ADBox is a Windows Server and Active Directory support lab focused on practical domain administration, client management, Group Policy, Remote Desktop access, troubleshooting, and support documentation.

The lab is built across Oracle VirtualBox (VBox) virtual machines (VMs) running on multiple physical laptops, connected through the same home network. It uses Windows Server 2022 as the Domain Controller (DC) for `adbox.local` and Windows 10 Pro clients as domain-joined workstations.

The purpose of ADBox is to demonstrate practical IT support skills in a controlled lab: preparing a server, building a domain, joining clients, organising users and computers, applying policy, testing remote access, documenting faults, and producing clear evidence through Markdown reports and screenshots.

## Skills Demonstrated

The lab is structured around practical IT support tasks rather than isolated theory. Each skill is demonstrated through a working configuration, validation step, screenshot, or troubleshooting record.

The skills below are not listed as generic keywords. Each one is tied to a specific part of the lab so the work can be reviewed through the linked reports and supporting screenshots.

| Skill | Demonstration |
|---|---|
| Virtualisation | Built the lab with VBox VMs across multiple physical laptops. |
| Windows Server Administration | Prepared and configured `AD-SRV01` as the main Windows Server system for the lab. |
| Active Directory Domain Services (AD DS) | Installed AD DS and promoted `AD-SRV01` as the first DC for `adbox.local`. |
| Domain Name System (DNS) | Configured clients to use `AD-SRV01` for domain resolution and validated lookups with `nslookup`. |
| Network Configuration | Used bridged networking, static server addressing, router DHCP, and IPv4 validation across the lab. |
| Windows Client Administration | Joined Windows 10 clients to the domain and confirmed their computer objects in Active Directory. |
| Directory Administration | Created an Organisational Unit (OU) structure for users, computers, groups, and future service accounts. |
| User Administration | Created department users with User Principal Name (UPN) logon formats and password-change requirements. |
| Security Group Administration | Created Global Security groups and tested group membership changes for access-control preparation. |
| Group Policy (GP) | Linked a workstation GPO, applied a logon notice, refreshed policy, and validated the result with `gpresult`. |
| Remote Desktop Protocol (RDP) | Enabled RDP on a domain-joined workstation and validated a remote session with Command Prompt (CMD). |
| Troubleshooting | Documented DNS and firewall issues using discovery, actions, and resolution records. |
| Technical Documentation | Recorded each stage through Markdown reports, screenshots, command output, and clear evidence chains. |

## Lab Environment

ADBox uses a small domain environment built on a home network. The server has a static Internet Protocol version 4 (IPv4) address so clients have a stable target for domain services, while the home router continues to provide normal client addressing through Dynamic Host Configuration Protocol (DHCP).

The key design choice is that DNS for the Windows clients points to `AD-SRV01`, not the router. This allows the clients to resolve `adbox.local`, locate the DC, join the domain, and receive domain-based configuration.

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

The repository separates lab reports, screenshots, notes, troubleshooting records, and future automation scripts. This keeps the evidence easy to follow and makes each stage reviewable without mixing screenshots, notes, and build documentation into one folder.

The `lab/` folder contains the main staged reports. The `troubleshooting/` folder keeps fault records separate so problems can be documented without interrupting the main build flow.

| Folder             | Purpose                                                 |
| ------------------ | ------------------------------------------------------- |
| `lab/`             | Main build and validation reports.                      |
| `troubleshooting/` | Fault records with discovery, actions, and resolution.  |
| `scripts/`         | PowerShell scripts for repeatable administration tasks. |
| `screenshots/`     | Evidence used by the lab and troubleshooting reports.   |
| `notes/`           | CMD, PowerShell, and terminology references.            |

## Lab Contents

The lab is written as a staged build. Each report introduces one practical capability, validates it, and leaves evidence that can be reviewed later.

The stages should be read in order because each one depends on the previous work. For example, GP only becomes meaningful after the clients have joined the domain and been moved into the correct OU.

| Stage | Report | Focus |
|---|---|---|
| 1 | [Lab Overview](lab/01-lab-overview.md) | Technical design, topology, domain plan, network layout, and evidence model. |
| 2 | [Environment Setup](lab/02-environment-setup.md) | VBox networking, static server addressing, client DHCP, and DNS path. |
| 3 | [Domain Controller](lab/03-domain-controller.md) | AD DS installation, forest creation, DC promotion, DNS role validation, and ADUC checks. |
| 4 | [Domain Join](lab/04-domain-join.md) | Windows 10 client domain join, domain sign-in, and AD computer object validation. |
| 5 | [Directory Structure](lab/05-directory-structure.md) | OU design, user accounts, security groups, computer placement, and group membership. |
| 6 | [Group Policy](lab/06-group-policy.md) | Workstation GPO creation, logon notice configuration, `gpupdate`, `gpresult`, and visible client validation. |
| 7 | [Remote Desktop Access](lab/07-remote-desktop.md) | RDP enablement, security group access, remote connection, and session identity validation. |

## Evidence Chain

ADBox follows a practical support administration sequence. Each stage builds on the previous one and creates evidence that can be reviewed through the matching report and screenshot folder.

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

The troubleshooting records document faults that appeared during the lab build. They are written separately from the main reports so the build path stays readable while the problems remain visible.

Each record follows the same structure: discovery, actions, and resolution. This makes the troubleshooting evidence closer to a support workflow, where the issue is observed, tested, fixed, and confirmed.

| Record | Focus |
|---|---|
| [DNS IPv6 Conflict](troubleshooting/01-dns-ipv6-conflict.md) | Client DNS lookups bypassing AD DNS because of router-provided Internet Protocol version 6 (IPv6) DNS information. |
| [Client Firewall Ping](troubleshooting/02-client-firewall-ping.md) | Server-to-client ping blocked by Windows Firewall inbound ICMP rules. |
| [Troubleshooting Records Index](troubleshooting/troubleshooting-records.md) | Index of recorded lab faults and fixes. |                                                                          |

## Current Status

ADBox is a functional lab implementation.

The current build includes a working Windows Server DC, two Windows 10 domain-joined clients, a structured Active Directory OU layout, department users, security groups, a tested workstation GP configuration, and a validated RDP access workflow.

The lab is documented through Markdown reports, screenshots, command output, and troubleshooting records.

## Limitations

ADBox is a lab environment, not a production deployment.

The domain runs on a home network, the DHCP service remains on the EE router, and the systems are built for controlled learning and portfolio evidence. The configuration choices are intentionally simple so each Windows support concept can be tested, documented, and explained clearly.

## Start Here

Begin with [Stage 1: Lab Overview](lab/01-lab-overview.md), which explains the ADBox technical design, domain plan, network layout, and evidence model.

## Licence

This project is provided for learning, documentation, and portfolio demonstration purposes.
