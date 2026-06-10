# Domain Controller

This section documents the build of `AD-SRV01` as the first Domain Controller (DC) for the ADBox lab.

The server starts as a prepared Windows Server 2022 virtual machine with a static Internet Protocol version 4 (IPv4) address and working network connectivity. This stage installs Active Directory Domain Services (AD DS), promotes the server into a new forest, creates the `adbox.local` domain, and validates the server after promotion.

The completed build provides the identity, authentication, Domain Name System (DNS), and directory foundation required before Windows 10 clients can join the domain.

## Purpose

The purpose of this section is to document how `AD-SRV01` was promoted from a prepared Windows Server system into the first DC for `adbox.local`.

It records the AD DS role installation, domain promotion choices, DNS configuration, NetBIOS domain name, default directory paths, prerequisite checks, post-promotion sign-in, and final validation.

This stage confirms that the lab has a working Active Directory foundation before moving on to client domain join, Organisational Units (OUs), Group Policy (GP), Remote Desktop Protocol (RDP), printer mapping, account recovery, and PowerShell administration.

## Server Preparation

Before promotion, `AD-SRV01` had already been prepared with Bridged Adapter networking, a static IPv4 address, and DNS pointed to itself.

| Setting | Value |
|---|---|
| Server Name | `AD-SRV01` |
| Operating System | Windows Server 2022 Standard Evaluation |
| Static IPv4 Address | `192.168.1.50` |
| Preferred DNS | `192.168.1.50` |
| Planned Domain | `adbox.local` |
| Planned NetBIOS Name | `ADBOX` |

These settings were documented in [Environment Setup](02-environment-setup.md).

## AD DS Role Installation

The AD DS role was selected through the Add Roles and Features Wizard. This installed the role required for the server to provide Active Directory domain services.

![Selected Server Roles](/screenshots/lab/03-domain-controller/01-selected-server-roles.png)

After the role installation completed, Server Manager showed that additional post-deployment configuration was required. The server then needed to be promoted to a DC.

![Server Promotion Notification](/screenshots/lab/03-domain-controller/02-server-promotion-notification.png)

## Domain Promotion

`AD-SRV01` was promoted by selecting **Add a new forest** and using `adbox.local` as the root domain name.

![New Local Forest](/screenshots/lab/03-domain-controller/03-new-local-forest.png)

This created a new Active Directory forest and domain for the lab instead of joining an existing domain or adding a new domain to an existing forest.

## Domain Controller Options

The promotion wizard selected the Windows Server 2016 forest and domain functional levels.

The following DC capabilities were selected:

| Option | Setting |
|---|---|
| Domain Name System (DNS) Server | Enabled |
| Global Catalog (GC) | Enabled |
| Read-Only Domain Controller (RODC) | Not selected |

![Domain Controller Options](/screenshots/lab/03-domain-controller/04-domain-controller-options.png)

The Directory Services Restore Mode (DSRM) password was also set during this stage. The password itself is not documented in the repository.

## DNS Delegation Warning

The wizard displayed a DNS delegation warning because there was no authoritative parent DNS zone for `adbox.local`.

![DNS Delegation Warning](/screenshots/lab/03-domain-controller/05-dns-delegation-warning.png)

This was expected in the lab because `adbox.local` is an internal Active Directory domain. No external parent DNS zone is being used for delegation.

## NetBIOS Domain Name

The wizard detected the NetBIOS domain name as `ADBOX`.

![NetBIOS Domain Detection](/screenshots/lab/03-domain-controller/06-netbios-domain-detection.png)

This allows the domain to support the legacy-compatible logon format:

```text
ADBOX\username
```

The domain also supports the User Principal Name (UPN) format:

```text
username@adbox.local
```

## Directory Paths

The default AD DS database, log, and SYSVOL paths were kept.

| Path Type | Location |
|---|---|
| Database Folder | `C:\Windows\NTDS` |
| Log Files Folder | `C:\Windows\NTDS` |
| SYSVOL Folder | `C:\Windows\SYSVOL` |

![Controller Default Paths](/screenshots/lab/03-domain-controller/07-controller-default-paths.png)

The default paths were suitable for this lab because the environment uses a single DC and does not require separate storage volumes.

## Prerequisites Check

The prerequisites check passed successfully before installation.

![Prerequisites Check Success](/screenshots/lab/03-domain-controller/08-prerequisites-check-success.png)

The wizard displayed warnings about DNS delegation and older cryptography compatibility. These warnings did not block promotion. The DNS delegation warning was expected for the internal lab domain, and the prerequisite check confirmed that installation could continue.

## Post-Promotion Sign-In

After promotion, the server restarted and the sign-in context changed to the `ADBOX` domain.

![Post Promotion Login](/screenshots/lab/03-domain-controller/09-post-promotion-login.png)

This confirmed that `AD-SRV01` was no longer operating only as a standalone server. It had become part of the newly created `adbox.local` domain.

## Server Manager Validation

Server Manager confirmed that the computer name was `AD-SRV01` and the domain was `adbox.local`.

![Manager Domain Confirmed](/screenshots/lab/03-domain-controller/10-manager-domain-confirmed.png)

Server Manager also showed the AD DS and DNS roles in the left navigation, confirming that the server roles were present after promotion.

## DNS Role Validation

The DNS role was visible in Server Manager, with `AD-SRV01` listed as the DNS server using the lab IPv4 address `192.168.1.50`.

![DNS ADBox Zone](/screenshots/lab/03-domain-controller/11-dns-adbox-zone.png)

This confirmed that DNS was available on the server after promotion.

## Active Directory Validation

Active Directory Users and Computers confirmed that the `adbox.local` domain was available and that `AD-SRV01` was listed under Domain Controllers.

![ADUC Domain View](/screenshots/lab/03-domain-controller/12-aduc-domain-view.png)

The view also showed `AD-SRV01` as a GC, confirming that the server was providing Global Catalog functionality.

## Name Resolution Validation

A server-side DNS lookup for `adbox.local` returned records for the domain, including the lab IPv4 address `192.168.1.50`.

![Server Nslookup Success](/screenshots/lab/03-domain-controller/13-server-nslookup-success.png)

This confirmed that the domain name could be resolved from `AD-SRV01` after promotion.

## Summary

`AD-SRV01` was successfully promoted as the first DC for the `adbox.local` forest and domain. The server provides AD DS, DNS, and GC services, with the NetBIOS domain name `ADBOX`.

The completed build confirms that the Active Directory foundation is working and ready for the next stage: joining Windows 10 clients to the domain.
