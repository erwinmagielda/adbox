# Environment Setup

ADBox runs across multiple physical laptops using Oracle VirtualBox virtual machines (VMs) connected through the same home Wi-Fi network.

The environment uses Bridged Adapter networking so each VM appears as a separate device on the home network. `AD-SRV01` uses a static Internet Protocol version 4 (IPv4) address, while the Windows 10 clients receive their IPv4 addresses from the EE router through Dynamic Host Configuration Protocol (DHCP).

Domain Name System (DNS) is pointed to `AD-SRV01` so the Windows 10 clients can resolve the `adbox.local` domain before domain join and workstation validation.

## Purpose

The purpose of this section is to document the base environment used before the domain build and client administration work.

It records the physical layout, VirtualBox network mode, addressing plan, DNS configuration, and adapter settings used to make the lab reachable across multiple machines.

This provides the working foundation for Domain Controller (DC) promotion, domain join, Group Policy (GP), Remote Desktop Protocol (RDP), printer mapping, account recovery, and PowerShell administration.

## Host Layout

The lab runs across multiple physical laptops instead of a single host machine. Each VM runs locally on its assigned physical device, while all machines communicate through the same home Wi-Fi network.

| Host Role | Virtual Machine | Notes |
|---|---|---|
| Server host | `AD-SRV01` | Runs the Windows Server 2022 server used for domain services. |
| Client host 1 | `AD-WIN10-01` | Windows 10 Pro domain client used for domain join and workstation validation. |
| Client host 2 | `AD-WIN10-02` | Windows 10 Pro domain client on a separate physical laptop. |

This layout reflects the available hardware and keeps the lab practical. The VMs are not displayed together in one VirtualBox console because they are distributed across different physical laptops.

## VirtualBox Network Mode

Each VM uses VirtualBox Bridged Adapter mode. This allows every VM to appear as a separate device on the same home network, making `AD-SRV01` and the Windows 10 clients reachable across the shared Wi-Fi connection.

`AD-SRV01` uses Bridged Adapter mode through the host Wi-Fi adapter.

![Server Bridged Adapter](/screenshots/lab/02-environment-setup/01-server-bridged-adapter.png)

The Windows 10 clients use the same Bridged Adapter pattern. One client configuration is shown below as evidence, with the same approach applied to both `AD-WIN10-01` and `AD-WIN10-02`.

![Client Bridged Adapter](/screenshots/lab/02-environment-setup/02-client-bridged-adapter.png)

## IP Addressing

The lab uses the home router for normal network access while reserving a stable IPv4 address for the server.

| Device | Addressing | IPv4 Address |
|---|---|---|
| `AD-SRV01` | Static | `192.168.1.50` |
| `AD-WIN10-01` | Router DHCP | `192.168.1.204` |
| `AD-WIN10-02` | Router DHCP | `192.168.1.102` |
| EE Router | Gateway and DHCP provider | `192.168.1.254` |

`AD-SRV01` uses a static IPv4 address so the clients have a consistent target for domain and DNS services. The Windows 10 clients receive their IPv4 addresses from the EE router through DHCP.

## DNS Configuration

`AD-SRV01` is configured to use itself for DNS. This allows it to provide name resolution for the `adbox.local` domain.

![Server IP Configuration](/screenshots/lab/02-environment-setup/03-server-ipconfig-dns.png)

The Windows 10 clients receive their IPv4 addresses from the router, but their DNS server is manually set to `192.168.1.50`. This ensures that domain lookups for `adbox.local` go to the DC instead of the home router.

![Client IP Configuration](/screenshots/lab/02-environment-setup/04-client-ipconfig-dns.png)

This split keeps the lab simple. The router handles IP address assignment, while `AD-SRV01` handles Active Directory DNS for the lab domain.

## IPv6 Lab Adapter Setting

Internet Protocol version 6 (IPv6) was disabled on the Windows 10 lab adapters after DNS testing showed that the clients were receiving router-provided IPv6 DNS information.

![Client IPv6 Disabled](/screenshots/lab/02-environment-setup/05-client-ipv6-disabled.png)

This keeps the lab on a controlled IPv4 path so the clients use `AD-SRV01` for domain resolution. The full troubleshooting record for this issue is documented in [DNS IPv6 Conflict](../troubleshooting/dns-ipv6-conflict.md).

## Validation Checks

The environment is considered ready for the next stage when the following checks pass from each Windows 10 client.

Check connectivity to the server:

```cmd
ping 192.168.1.50
```

Check domain resolution:

```cmd
nslookup adbox.local
```

Check the server Fully Qualified Domain Name (FQDN):

```cmd
nslookup AD-SRV01.adbox.local
```

These checks confirm that the clients can reach the server, resolve the domain, and locate the server by its FQDN.

## Summary

The ADBox environment uses VirtualBox VMs across multiple physical laptops, with Bridged Adapter networking over the same home Wi-Fi network. `AD-SRV01` provides the static server endpoint and DNS path for `adbox.local`, while the EE router continues to provide DHCP addressing for the Windows 10 clients.

This setup provides the network foundation required for the next stage: building and validating the DC.