# ADBox

ADBox is my Windows Server and Active Directory support lab.

I built it to practise the kind of support work I expect to meet in IT roles: preparing a Windows Server environment, building a domain, joining Windows clients, organising users and computers, applying Group Policy, testing Remote Desktop access, setting up shared folder permissions, recovering user accounts, and checking the environment with PowerShell.

The lab runs on Oracle VirtualBox virtual machines across multiple physical laptops connected through the same home network. `AD-SRV01` provides the Domain Controller and DNS role for `adbox.local`. Two Windows 10 clients are used to test domain join, policy application, remote access, file sharing, and user sign-in behaviour.

## Lab Coverage

The reports below follow the lab from the first network checks through to practical support tasks such as file access, account recovery, and PowerShell review.

Section | Report | Coverage
--- | --- | ---
01 | [Lab Overview](lab/01-lab-overview.md) | Overall lab design, topology, domain plan, network layout, core services, and evidence approach.
02 | [Environment Setup](lab/02-environment-setup.md) | VirtualBox bridged networking, home-network layout, static server IP, client DHCP, DNS path, and IPv6 adapter notes.
03 | [Domain Controller](lab/03-domain-controller.md) | AD DS role installation, new forest creation, `adbox.local` promotion, DNS role checks, ADUC validation, and server-side name resolution.
04 | [Domain Join](lab/04-domain-join.md) | Windows 10 domain join, client DNS checks, domain sign-in, restart behaviour, and computer object confirmation in Active Directory.
05 | [Directory Structure](lab/05-directory-structure.md) | OU layout, workstation placement, department users, Global Security groups, group membership, and correction of an incorrect group member.
06 | [Group Policy](lab/06-group-policy.md) | Workstation GPO targeting, logon notice configuration, `gpupdate /force`, `gpresult /r`, and visible policy confirmation on the client.
07 | [Remote Desktop](lab/07-remote-desktop.md) | Enabling RDP on a domain-joined workstation, adding an AD security group to local RDP access, connecting remotely, and validating the session.
08 | [File Sharing](lab/08-file-sharing.md) | Creating a department share, configuring share permissions, applying NTFS permissions, testing group-based access, and creating a file from the client.
09 | [Account Recovery](lab/09-account-recovery.md) | Resetting a user password, forcing password change at next sign-in, disabling the account, confirming blocked access, enabling the account again, and testing restored sign-in.
10 | [PowerShell Administration](lab/10-powershell-administration.md) | Checking AD users, groups, computer objects, disabled users, DNS resolution, client connectivity, share access, and exporting a basic user report.

## Setup Troubleshooting

These records cover setup issues found while getting the ADBox network and client communication working reliably.

Section | Report | Coverage
--- | --- | ---
01 | [DNS IPv6 Conflict](troubleshooting/01-dns-ipv6-conflict.md) | Client lookups for `adbox.local` failed because the Windows 10 clients were receiving router-provided IPv6 DNS information. The fix kept domain lookups on the intended IPv4 DNS path through `AD-SRV01`.
02 | [Client Firewall Ping](troubleshooting/02-client-firewall-ping.md) | Server-to-client ping failed because Windows Firewall blocked inbound ICMP on the client. The fix confirmed that both clients could respond to network checks from `AD-SRV01`.                   

## Repository Layout

The repository is split so the main reports, screenshots, troubleshooting notes, and supporting files stay easy to review.

| Folder             | Purpose                                                                |
| ------------------ | ---------------------------------------------------------------------- |
| `lab/`             | Main staged reports for the ADBox build.                               |
| `troubleshooting/` | Fault records written around discovery, checks, fix, and confirmation. |
| `screenshots/`     | Screenshot evidence used by the lab reports.                           |
| `scripts/`         | PowerShell scripts and repeatable administration work.                 |
| `notes/`           | Supporting command notes, terminology, and reference material.         |

## Start Here

Start with [01 Lab Overview](lab/01-lab-overview.md), then follow the reports in order from Environment Setup through PowerShell Administration.

## Licence

This project is provided for learning, documentation, and portfolio demonstration purposes.
