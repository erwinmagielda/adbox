# Domain Controller

This stage turns `AD-SRV01` from a prepared Windows Server machine into the first Domain Controller for the `adbox.local` domain.

The server already has bridged networking, a static IPv4 address, and DNS pointed to itself from the environment setup stage. This page records the Active Directory Domain Services installation, new forest creation, promotion options, expected wizard warnings, and post-promotion checks.

## Build Steps

The Domain Controller build followed the Server Manager promotion workflow from role installation to post-promotion validation.

Step | Action | Covers
--- | --- | ---
01 | Confirm Server Base | Static IP, bridged networking, and DNS pointed to `AD-SRV01`.
02 | Install AD DS | Active Directory Domain Services role added through Server Manager.
03 | Start Promotion | Post-deployment task used to promote the server.
04 | Create New Forest | `adbox.local` created as the root domain.
05 | Set DC Options | DNS and Global Catalog enabled; writable DC selected.
06 | Review DNS Warning | Expected delegation warning checked and accepted.
07 | Confirm NetBIOS Name | `ADBOX` detected as the short domain name.
08 | Keep Default Paths | AD database, logs, and SYSVOL left on default paths.
09 | Run Prerequisites | Promotion checks passed without blocking errors.
10 | Restart Into Domain | Server restarted under the `ADBOX` domain context.
11 | Validate Roles | Server Manager confirmed AD DS and DNS roles.
12 | Validate Domain | ADUC, DNS, and `nslookup` confirmed the domain was working.

## Starting Point

`AD-SRV01` was prepared before promotion so the domain had a stable network base.

Setting | Value
--- | ---
Server Name | `AD-SRV01`
Operating System | Windows Server 2022 Standard Evaluation
Static IPv4 Address | `192.168.1.50`
Preferred DNS | `192.168.1.50`
Planned Domain | `adbox.local`
Planned NetBIOS Name | `ADBOX`

The static IP matters because Windows clients need a reliable DNS and domain target. The server DNS points to itself because the Domain Controller also provides DNS for the lab domain.

These settings are covered in [02 Environment Setup](02-environment-setup.md).

## AD DS Role Installation

The Active Directory Domain Services role was selected through the Add Roles and Features Wizard.

![Selected Server Roles](/screenshots/lab/03-domain-controller/01-selected-server-roles.png)

Installing the role adds the components needed for the server to provide directory services. At this point, the AD DS role is installed, but the server still needs promotion before it can act as a Domain Controller.

After installation, Server Manager showed the post-deployment task to promote the server.

![Server Promotion Notification](/screenshots/lab/03-domain-controller/02-server-promotion-notification.png)

This promotion task is what creates the domain and changes the server into a Domain Controller.

## New Forest Creation

`AD-SRV01` was promoted using the **Add a new forest** option.

The root domain name used was:

```text
adbox.local
```

![New Local Forest](/screenshots/lab/03-domain-controller/03-new-local-forest.png)

This created the first domain in a new Active Directory forest for the lab.

## Domain Controller Options

The promotion wizard configured `AD-SRV01` as the first Domain Controller for the lab domain.

Option | Setting
--- | ---
DNS Server | Enabled
Global Catalog | Enabled
Read-Only Domain Controller | Disabled
Domain Controller Type | Writable Domain Controller

![Domain Controller Options](/screenshots/lab/03-domain-controller/04-domain-controller-options.png)

DNS was enabled because the clients need to resolve `adbox.local` and locate domain services. Global Catalog was enabled because this is the first Domain Controller in the domain.

The Directory Services Restore Mode password was set during promotion. The password is not stored in the repository.

## DNS Delegation Warning

The wizard displayed a DNS delegation warning.

![DNS Delegation Warning](/screenshots/lab/03-domain-controller/05-dns-delegation-warning.png)

> DNS Delegation is used when a parent DNS zone points a child domain to the DNS servers responsible for that child domain. The warning appeared because `adbox.local` is an internal lab domain and there is no parent DNS zone configured to delegate it to `AD-SRV01`.

The warning did not block promotion because the Windows clients are configured to use `AD-SRV01` directly for `adbox.local` DNS lookups.

## NetBIOS Name

The wizard detected the NetBIOS domain name as:

```text
ADBOX
```

![NetBIOS Domain Detection](/screenshots/lab/03-domain-controller/06-netbios-domain-detection.png)

This allows domain accounts to sign in using the legacy-compatible format:

```text
ADBOX\username
```

The domain also supports the User Principal Name format:

```text
username@adbox.local
```

Both formats are used later in the lab when testing domain sign-in, account recovery, Remote Desktop access, and file-share access.

## Directory Paths

The default AD DS database, log, and SYSVOL paths were kept.

Path Type | Location
--- | ---
Database Folder | `C:\Windows\NTDS`
Log Files Folder | `C:\Windows\NTDS`
SYSVOL Folder | `C:\Windows\SYSVOL`

![Controller Default Paths](/screenshots/lab/03-domain-controller/07-controller-default-paths.png)

> `NTDS` is the folder used by Active Directory Domain Services for the directory database and related log files. The main database file is `NTDS.dit`, which stores domain objects such as users, computers, groups, OUs, and directory configuration.

> `SYSVOL` is a shared folder used by Domain Controllers to store domain-wide files such as Group Policy content and logon scripts. Clients need access to SYSVOL so they can read policy files and domain scripts during normal domain operation.

The default paths are suitable for this single-server lab because `AD-SRV01` is the only Domain Controller and the environment does not need separate storage volumes.

## Prerequisites Check

The prerequisites check passed before promotion.

![Prerequisites Check Success](/screenshots/lab/03-domain-controller/08-prerequisites-check-success.png)

The wizard showed warnings, but there were no blocking errors. After the check passed, the promotion completed and the server restarted.

## Post-Promotion Sign-In

After restart, the sign-in context showed the `ADBOX` domain.

![Post Promotion Login](/screenshots/lab/03-domain-controller/09-post-promotion-login.png)

This confirmed that the server was operating inside the new `adbox.local` domain.

## Server Manager Validation

Server Manager confirmed the domain membership and installed roles.

![Manager Domain Confirmed](/screenshots/lab/03-domain-controller/10-manager-domain-confirmed.png)

Check | Result
--- | ---
Computer Name | `AD-SRV01`
Domain | `adbox.local`
Server IP | `192.168.1.50`
Roles Visible | AD DS and DNS

This confirmed that the server identity, domain membership, and core roles were visible after promotion.

## DNS Validation

The DNS role was visible in Server Manager, with `AD-SRV01` listed as the DNS server for the lab address.

![DNS ADBox Zone](/screenshots/lab/03-domain-controller/11-dns-adbox-zone.png)

A server-side lookup for the lab domain also returned the expected result.

```text
nslookup adbox.local
```

![Server Nslookup Success](/screenshots/lab/03-domain-controller/13-server-nslookup-success.png)

This confirmed that `AD-SRV01` could resolve the lab domain after promotion.

## Active Directory Validation

Active Directory Users and Computers showed the `adbox.local` domain.

![ADUC Domain View](/screenshots/lab/03-domain-controller/12-aduc-domain-view.png)

`AD-SRV01` was listed under Domain Controllers, confirming that the server object exists in the expected domain location.

## Result

`AD-SRV01` was promoted as the first Domain Controller for `adbox.local`.

The server now provides:

Service | Purpose
--- | ---
Active Directory Domain Services | Stores and manages users, computers, groups, OUs, and domain objects.
Lab Domain DNS | Resolves `adbox.local` and helps clients locate domain services.
Global Catalog | Supports directory lookups inside the domain.
Domain Authentication | Allows future Windows 10 clients to authenticate domain users and computer accounts.

The next stage tests whether the Windows 10 clients can use this domain and DNS setup to join `adbox.local`.

## Navigation

Previous | Current | Next
--- | --- | ---
[02 Environment Setup](02-environment-setup.md) | Domain Controller | [04 Domain Join](04-domain-join.md)
