# Domain Join

The Windows 10 clients are joined to the `adbox.local` domain after the Domain Controller (DC) and Domain Name System (DNS) services are confirmed working.

This stage proves that the domain is reachable from the workstation side, that client DNS points to `AD-SRV01`, and that Windows clients can authenticate against Active Directory. The detailed join process is shown with `AD-WIN10-01`, then repeated on `AD-WIN10-02`.

Both joined computer objects are confirmed in Active Directory Users and Computers (ADUC) before moving into Organisational Unit (OU) structure, Group Policy (GP), Remote Desktop Protocol (RDP), printer mapping, account recovery, and workstation administration.

## Pre-Join Validation

Before attempting the domain join, `AD-WIN10-01` was checked from the client side.

| Check                                     | Expected Result                          |
| ----------------------------------------- | ---------------------------------------- |
| Client hostname                           | `AD-WIN10-01`                            |
| Client DNS server                         | `192.168.1.50`                           |
| Server connectivity                       | `ping 192.168.1.50` succeeds             |
| Domain lookup                             | `nslookup adbox.local` resolves          |
| Server Fully Qualified Domain Name (FQDN) | `nslookup AD-SRV01.adbox.local` resolves |

![Client DNS Precheck](/screenshots/lab/04-domain-join/01-client-dns-precheck.png)

These checks confirmed that the client could reach the DC and use the lab DNS path before the join was attempted.

## Domain Join

The client was joined through the Windows System Properties dialog.

The domain entered was:

```text
adbox.local
```

![Domain Join Dialog](/screenshots/lab/04-domain-join/02-domain-join-dialog.png)

When Windows requested credentials, the User Principal Name (UPN) format was accepted:

```text
Administrator@adbox.local
```

The join completed successfully and Windows confirmed that the client had joined the `adbox.local` domain.

![Domain Join Success](/screenshots/lab/04-domain-join/03-domain-join-success.png)

After the success message, the client was restarted to complete the membership change.

## Domain Sign-In

After restart, the client allowed sign-in with a domain account.

```text
Administrator@adbox.local
```

![Client Domain Login](/screenshots/lab/04-domain-join/04-client-domain-login.png)

This confirmed that the workstation was no longer only operating as a standalone workgroup machine. It could now use domain authentication.

## Active Directory Confirmation

After both clients were joined, ADUC was checked on `AD-SRV01`.

| Client        | Status                  |
| ------------- | ----------------------- |
| `AD-WIN10-01` | Joined to `adbox.local` |
| `AD-WIN10-02` | Joined to `adbox.local` |

![ADUC Computer Objects](/screenshots/lab/04-domain-join/05-aduc-computer-objects.png)

Both Windows 10 clients appeared in the default `Computers` container. This confirmed that the join process created computer objects in Active Directory.

The objects are moved into the planned workstation OU in the next stage, where the directory structure is organised for policy and administration.

## Summary

Both Windows 10 clients joined the `adbox.local` domain successfully.

The join process confirmed client DNS configuration, domain lookup, domain authentication, and computer object creation in Active Directory.

## Next Stage

[Stage 5: Directory Structure](05-directory-structure.md), which organises users, computers, OUs, and groups into a controlled Active Directory structure.
