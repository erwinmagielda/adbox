# Environment Setup

This stage prepares the base network environment before building the Domain Controller.

The aim is simple: `AD-SRV01` needs a stable address, the Windows 10 clients need to reach it, and domain-related DNS queries need to go to the server instead of the home router.

## Host Layout

The lab uses three VirtualBox virtual machines split across separate physical laptops on the same home Wi-Fi network.

Host Role | Virtual Machine | Purpose
--- | --- | ---
Server Host | `AD-SRV01` | Windows Server 2022 system prepared for domain services.
Client Host 1 | `AD-WIN10-01` | Windows 10 Pro client used for domain join and workstation validation.
Client Host 2 | `AD-WIN10-02` | Second Windows 10 Pro client used to confirm the setup works across more than one workstation.

The machines are not shown together in one VirtualBox console because they run on different physical laptops. They still behave as one lab environment because they share the same local network.

## VirtualBox Network Mode

Each virtual machine uses VirtualBox Bridged Adapter mode.

Bridged networking makes each VM appear as its own device on the home network. This allows the server and clients to communicate through the same router while still running as virtual machines.

`AD-SRV01` uses Bridged Adapter mode through the host Wi-Fi adapter.

![Server Bridged Adapter](/screenshots/lab/02-environment-setup/01-server-bridged-adapter.png)

The Windows 10 clients use the same bridged networking pattern. One client example is shown below, with the same approach used for both clients.

![Client Bridged Adapter](/screenshots/lab/02-environment-setup/02-client-bridged-adapter.png)

## IP Addressing

`AD-SRV01` uses a static IPv4 address so the clients have a reliable target for DNS and upcoming Active Directory Domain Services.

The Windows 10 clients receive their IPv4 addresses from the EE router through DHCP.

Device | Addressing | IPv4 Address
--- | --- | ---
`AD-SRV01` | Static | `192.168.1.50`
`AD-WIN10-01` | Router DHCP | `192.168.1.204`
`AD-WIN10-02` | Router DHCP | `192.168.1.102`
EE Router | Gateway and DHCP Provider | `192.168.1.254`

This keeps the setup simple. The router handles normal client addressing, while the server keeps a fixed address for Active Directory and DNS.

## DNS Path

`AD-SRV01` is configured to use itself for DNS.

![Server IP Configuration](/screenshots/lab/02-environment-setup/03-server-ipconfig-dns.png)

The Windows 10 clients keep router-provided IP addressing, but their DNS server is manually set to:

```text
192.168.1.50
````

That sends `adbox.local` lookups to `AD-SRV01`.

![Client IP Configuration](/screenshots/lab/02-environment-setup/04-client-ipconfig-dns.png)

This DNS path matters because Windows clients need to find the Domain Controller before they can join the domain or authenticate against it.

## IPv6 Adapter Setting

IPv6 was disabled on the Windows 10 lab adapters after testing showed that the clients were receiving router-provided IPv6 DNS information.

![Client IPv6 Disabled](/screenshots/lab/02-environment-setup/05-client-ipv6-disabled.png)

With IPv6 enabled, the clients were not consistently using the intended DNS path for the lab domain. Disabling IPv6 kept the environment on the controlled IPv4 path through `AD-SRV01`.

The full issue is documented in [DNS IPv6 Conflict](../troubleshooting/01-dns-ipv6-conflict.md).

## Validation Checks

Before moving into the Domain Controller build, each client needed to prove three things:

| Check                  | Command                         | Expected Result                                   |
| ---------------------- | ------------------------------- | ------------------------------------------------- |
| Server Reach           | `ping 192.168.1.50`             | Client can communicate with `AD-SRV01`.           |
| Lab Domain Resolution  | `nslookup adbox.local`          | Client receives a response through lab DNS.       |
| Server FQDN Resolution | `nslookup AD-SRV01.adbox.local` | Client can locate the server by full domain name. |

These checks confirm that the base network and DNS path are ready before installing and promoting Active Directory Domain Services.

> `AD-SRV01.adbox.local` is the server Fully Qualified Domain Name (FQDN) that combines the hostname and domain name into one complete DNS name.

## Navigation

| Previous                              | Current           | Next                                            |
| ------------------------------------- | ----------------- | ----------------------------------------------- |
| [01 Lab Overview](01-lab-overview.md) | Environment Setup | [03 Domain Controller](03-domain-controller.md) |