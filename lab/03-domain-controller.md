# Domain Controller

`AD-SRV01` is built as the first Domain Controller (DC) for the ADBox lab.

The server starts as a prepared Windows Server 2022 virtual machine with a static Internet Protocol version 4 (IPv4) address, bridged networking, and Domain Name System (DNS) pointed to itself. This stage installs Active Directory Domain Services (AD DS), promotes the server into a new forest, creates the `adbox.local` domain, and validates the domain services after promotion.

The completed build provides the identity, authentication, DNS, and directory foundation required before Windows 10 clients can join the domain.

## Server Preparation

Before promotion, `AD-SRV01` had already been prepared with a stable network configuration.

| Setting             | Value                                   |
| ------------------- | --------------------------------------- |
| Server Name         | `AD-SRV01`                              |
| Operating System    | Windows Server 2022 Standard Evaluation |
| Static IPv4 Address | `192.168.1.50`                          |
| Preferred DNS       | `192.168.1.50`                          |
| Planned Domain      | `adbox.local`                           |
| Planned NetBIOS     | `ADBOX`                                 |

These settings are documented in [02-environment-setup.md](02-environment-setup.md).

## AD DS Role Installation

The AD DS role was selected through the Add Roles and Features Wizard. This installs the role required for the server to provide Active Directory domain services.

![Selected Server Roles](/screenshots/lab/03-domain-controller/01-selected-server-roles.png)

After the role installation completed, Server Manager displayed a post-deployment task to promote the server to a DC.

![Server Promotion Notification](/screenshots/lab/03-domain-controller/02-server-promotion-notification.png)

## Domain Promotion

`AD-SRV01` was promoted by selecting **Add a new forest** and using `adbox.local` as the root domain name.

![New Local Forest](/screenshots/lab/03-domain-controller/03-new-local-forest.png)

This created a new Active Directory forest and domain for the lab.

## Domain Controller Options

The promotion wizard used the Windows Server 2016 forest and domain functional levels.

| Option                             | Setting      |
| ---------------------------------- | ------------ |
| DNS Server                         | Enabled      |
| Global Catalog (GC)                | Enabled      |
| Read-Only Domain Controller (RODC) | Not selected |

![Domain Controller Options](/screenshots/lab/03-domain-controller/04-domain-controller-options.png)

`AD-SRV01` was configured as a writable DC with DNS and GC enabled. The Directory Services Restore Mode (DSRM) password was also set during this stage, but the password is not documented in the repository.

## DNS Delegation Warning

The wizard displayed a DNS delegation warning because there was no authoritative parent DNS zone for `adbox.local`.

![DNS Delegation Warning](/screenshots/lab/03-domain-controller/05-dns-delegation-warning.png)

This warning was expected because `adbox.local` is an internal lab domain. No external parent DNS zone is being used for delegation.

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

| Path Type        | Location            |
| ---------------- | ------------------- |
| Database Folder  | `C:\Windows\NTDS`   |
| Log Files Folder | `C:\Windows\NTDS`   |
| SYSVOL Folder    | `C:\Windows\SYSVOL` |

![Controller Default Paths](/screenshots/lab/03-domain-controller/07-controller-default-paths.png)

The default paths are suitable for this lab because it uses a single DC and does not require separate storage volumes.

## Prerequisites Check

The prerequisites check passed before installation.

![Prerequisites Check Success](/screenshots/lab/03-domain-controller/08-prerequisites-check-success.png)

The wizard displayed warnings about DNS delegation and older cryptography compatibility, but neither warning blocked promotion.

## Post-Promotion Sign-In

After promotion, the server restarted and the sign-in context changed to the `ADBOX` domain.

![Post Promotion Login](/screenshots/lab/03-domain-controller/09-post-promotion-login.png)

This shows that `AD-SRV01` was no longer operating only as a standalone server. It had become part of the newly created `adbox.local` domain.

## Server Manager Validation

Server Manager confirmed the server identity and domain membership.

![Manager Domain Confirmed](/screenshots/lab/03-domain-controller/10-manager-domain-confirmed.png)

The view confirms:

| Item          | Value          |
| ------------- | -------------- |
| Computer Name | `AD-SRV01`     |
| Domain        | `adbox.local`  |
| Server IP     | `192.168.1.50` |
| Roles Visible | AD DS and DNS  |

## DNS Role Validation

The DNS role was visible in Server Manager, with `AD-SRV01` listed as the DNS server using the lab IPv4 address `192.168.1.50`.

![DNS ADBox Zone](/screenshots/lab/03-domain-controller/11-dns-adbox-zone.png)

This confirms that the DNS role was available after promotion.

## Active Directory Validation

Active Directory Users and Computers (ADUC) confirmed that the `adbox.local` domain was available and that `AD-SRV01` was listed under Domain Controllers.

![ADUC Domain View](/screenshots/lab/03-domain-controller/12-aduc-domain-view.png)

The view also shows `AD-SRV01` as a GC, confirming that the server provides Global Catalog functionality.

## Name Resolution Validation

A server-side DNS lookup for `adbox.local` returned records for the domain, including the lab IPv4 address `192.168.1.50`.

![Server Nslookup Success](/screenshots/lab/03-domain-controller/13-server-nslookup-success.png)

This confirms that `AD-SRV01` can resolve the lab domain after promotion.

## Summary

`AD-SRV01` was successfully promoted as the first DC for the `adbox.local` forest and domain. The server now provides AD DS, DNS, and GC services, with the NetBIOS domain name `ADBOX`.

The Active Directory foundation is ready for the next stage: joining Windows 10 clients to the domain.

## Next Stage

[Stage 4: Domain Join](04-domain-join.md), which documents joining the Windows 10 clients to `adbox.local` and confirming the computer objects in ADUC.
