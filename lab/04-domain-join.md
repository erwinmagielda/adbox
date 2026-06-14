# Domain Join

This section documents the process of joining the Windows 10 client machines to the `adbox.local` domain.

The client join confirms that the Active Directory domain is reachable from the workstation side, that Domain Name System (DNS) resolution is working, and that Windows clients can authenticate against the Domain Controller (DC).

The detailed join process is shown using `AD-WIN10-01`. The same process was then repeated for `AD-WIN10-02`, with both computer objects confirmed in Active Directory Users and Computers.

## Purpose

The purpose of this section is to document the client-side domain join process for the ADBox lab.

It records the pre-join validation checks, domain join configuration, successful join confirmation, domain sign-in, and final computer object validation in Active Directory.

This stage confirms that the lab is ready for Organisational Unit (OU) structure, Group Policy (GP), Remote Desktop Protocol (RDP), printer mapping, account recovery, and workstation administration.

## Pre-Join Validation

Before joining the domain, the client was checked to confirm that it could reach `AD-SRV01` and resolve the `adbox.local` domain.

The pre-join checks confirmed:

| Check | Expected Result |
|---|---|
| Client hostname | `AD-WIN10-01` |
| Client DNS server | `192.168.1.50` |
| Server connectivity | `ping 192.168.1.50` succeeds |
| Domain lookup | `nslookup adbox.local` resolves |
| Server Fully Qualified Domain Name (FQDN) lookup | `nslookup AD-SRV01.adbox.local` resolves |

![Client DNS Precheck](/screenshots/lab/04-domain-join/01-client-dns-precheck.png)

These checks confirmed that the client could reach the DC and use the lab DNS path before the domain join was attempted.

## Domain Join

The client was joined to the domain through the Windows System Properties dialog.

The domain entered was:

```text
adbox.local
```

![Domain Join Dialog](/screenshots/lab/04-domain-join/02-domain-join-dialog.png)

When prompted for credentials, the User Principal Name (UPN) format was used:

```text
Administrator@adbox.local
```

The domain join completed successfully and Windows confirmed that the client had joined the `adbox.local` domain.

![Domain Join Success](/screenshots/lab/04-domain-join/03-domain-join-success.png)

After this confirmation, the client was restarted to complete the domain membership change.

## Domain Sign-In

After restart, the client allowed domain sign-in.

The domain administrator account was used to confirm that the client could authenticate against the domain:

```text
Administrator@adbox.local
```

![Client Domain Login](/screenshots/lab/04-domain-join/04-client-domain-login.png)

This confirmed that the client was no longer only operating as a standalone workgroup machine and could now use domain authentication.

## Active Directory Confirmation

After the join process, Active Directory Users and Computers was checked on `AD-SRV01`.

Both Windows 10 clients were visible as computer objects in the default `Computers` container:

| Client | Status |
|---|---|
| `AD-WIN10-01` | Joined to `adbox.local` |
| `AD-WIN10-02` | Joined to `adbox.local` |

![ADUC Computer Objects](/screenshots/lab/04-domain-join/05-aduc-computer-objects.png)

This confirmed that both clients had joined the domain successfully and were registered in Active Directory.

## Summary

Both Windows 10 clients were successfully joined to the `adbox.local` domain.

The process confirmed that client DNS configuration, domain lookup, domain authentication, and Active Directory computer object creation were working correctly.

The next stage is to organise the joined clients, users, groups, and service accounts into a structured OU layout.