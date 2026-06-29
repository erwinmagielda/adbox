# Domain Join

This stage joins the Windows 10 clients to the `adbox.local` domain.

The Domain Controller and DNS service are already in place from the previous stage. The focus here is the workstation side: checking that the clients can reach `AD-SRV01`, resolving the lab domain through the correct DNS path, joining the domain, signing in with domain credentials, and confirming the new computer objects in Active Directory.

## Join Steps

The domain join followed the same process on both Windows 10 clients.

Step | Action | Covers
--- | --- | ---
01 | Confirm Client DNS | Client hostname, DNS server, server reachability, lab domain lookup, and full server name lookup.
02 | Enter Domain Name | `adbox.local` entered through the Windows System Properties join dialog.
03 | Authenticate Join | Domain administrator credentials used to approve the join.
04 | Restart Client | Windows restarted to apply the domain membership change.
05 | Sign-In Domain Account | Client sign-in tested using domain credentials.
06 | Confirm Computer Object | Joined workstation checked in Active Directory Users and Computers.

## Pre-Join Checks

Before attempting the join, `AD-WIN10-01` was checked from the client side.

Check | Expected Result
--- | ---
Client Hostname | `AD-WIN10-01`
Client DNS Server | `192.168.1.50`
Server Reach | `ping 192.168.1.50`
Lab Domain Resolution | `nslookup adbox.local` 
Server Full Name Resolution | `nslookup AD-SRV01.adbox.local`

![Client DNS Precheck](../screenshots/lab/04-domain-join/01-client-dns-precheck.png)

> Domain join depends heavily on DNS. A client can have working internet access and still fail domain join if it is using the router or public DNS instead of the Domain Controller for `adbox.local` lookups.

These checks confirmed that the client could reach `AD-SRV01` and use the lab DNS path before the domain join was attempted.

## Joining AD-WIN10-01

The client was joined through the Windows System Properties dialog.

The domain entered was:

```text
adbox.local
```

![Domain Join Dialog](../screenshots/lab/04-domain-join/02-domain-join-dialog.png)

Windows then requested credentials with permission to join the computer to the domain.

The join was approved using the User Principal Name format:

```text
Administrator@adbox.local
```

> `Administrator@adbox.local` uses the User Principal Name format. It identifies the account and the domain together, which makes the sign-in target clear during the join.

The join completed successfully and Windows confirmed that the client had joined the `adbox.local` domain.

![Domain Join Success](../screenshots/lab/04-domain-join/03-domain-join-success.png)

After the success message, the client was restarted so Windows could apply the domain membership change.

## Domain Sign-In

After restart, `AD-WIN10-01` allowed sign-in with a domain account.

```text
Administrator@adbox.local
```

![Client Domain Login](../screenshots/lab/04-domain-join/04-client-domain-login.png)

This confirmed that the workstation could authenticate against the domain and operate as a domain-joined client.

## Joining AD-WIN10-02

The same process was repeated on `AD-WIN10-02`.

The client DNS path was checked first, then the machine was joined to:

```text
adbox.local
```

After restart, the client was able to use domain sign-in in the same way as `AD-WIN10-01`.

## Active Directory Confirmation

After both clients were joined, Active Directory Users and Computers was checked on `AD-SRV01`.

Client | Domain Join State
--- | ---
`AD-WIN10-01` | Joined to `adbox.local`
`AD-WIN10-02` | Joined to `adbox.local`

![ADUC Computer Objects](../screenshots/lab/04-domain-join/05-aduc-computer-objects.png)

Both Windows 10 clients appeared in the default `Computers` container.

> New domain-joined Windows computers are placed in the default `Computers` container first. In the next stage, the workstation objects are moved into a dedicated Workstations OU so policies and administration tasks can target them cleanly.

## Result

Both Windows 10 clients joined the `adbox.local` domain successfully.

Check | Result
--- | ---
Client DNS pointed to `AD-SRV01` | Passed
Client resolved `adbox.local` | Passed
`AD-WIN10-01` joined the domain | Passed
`AD-WIN10-02` joined the domain | Passed
Domain sign-in worked after restart | Passed
Computer objects appeared in Active Directory | Passed

The lab is now ready for directory organisation: moving workstation objects, creating users, building groups, and preparing the structure used by later Group Policy and access-control tests.

## Navigation

Previous | Current | Next
--- | --- | ---
[03 Domain Controller](03-domain-controller.md) | Domain Join | [05 Directory Structure](05-directory-structure.md)
