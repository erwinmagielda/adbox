# Environment Setup

ADBox runs across multiple physical laptops using Oracle VirtualBox (VBox) virtual machines (VMs) connected through the same home Wi-Fi network.

This stage documents the base environment used before the domain build: physical host layout, bridged networking, Internet Protocol version 4 (IPv4) addressing, Domain Name System (DNS) configuration, and client adapter settings. The aim is to prove that the server and clients can communicate reliably before Domain Controller (DC) promotion, domain join, Group Policy (GP), Remote Desktop Protocol (RDP), printer mapping, account recovery, and PowerShell administration.

`AD-SRV01` is configured as the stable server endpoint for the lab. The Windows 10 clients receive their IPv4 addresses from the EE router through Dynamic Host Configuration Protocol (DHCP), while DNS is pointed to `AD-SRV01` so the clients can resolve the `adbox.local` domain.

## Host Layout

The lab is split across multiple physical laptops rather than one single host. Each VM runs locally on its assigned physical device, while all machines communicate through the same home Wi-Fi network.

| Host Role | Virtual Machine | Notes |
| --- | --- | --- |
| Server Host   | `AD-SRV01`      | Windows Server 2022 server used for domain services.                          |
| Client Host 1 | `AD-WIN10-01`   | Windows 10 Pro domain client used for domain join and workstation validation. |
| Client Host 2 | `AD-WIN10-02`   | Windows 10 Pro domain client running on a separate physical laptop.           |

The VMs are not shown together in one VBox console because they are distributed across different physical laptops. This reflects the available hardware and still provides a useful networked lab environment.

## Network Mode

Each VM uses VBox Bridged Adapter mode. This makes every VM appear as a separate device on the home network, allowing `AD-SRV01` and the Windows 10 clients to communicate across the shared Wi-Fi connection.

`AD-SRV01` uses Bridged Adapter mode through the host Wi-Fi adapter.

![Server Bridged Adapter](/screenshots/lab/02-environment-setup/01-server-bridged-adapter.png)

The Windows 10 clients use the same Bridged Adapter pattern. One client configuration is shown below, with the same approach applied to both `AD-WIN10-01` and `AD-WIN10-02`.

![Client Bridged Adapter](/screenshots/lab/02-environment-setup/02-client-bridged-adapter.png)

## Addressing Plan

The EE router provides normal home-network access, while `AD-SRV01` uses a static IPv4 address so the clients have a consistent target for domain and DNS services.

| Device        | Addressing                | IPv4 Address    |
| ------------- | ------------------------- | --------------- |
| `AD-SRV01`    | Static                    | `192.168.1.50`  |
| `AD-WIN10-01` | Router DHCP               | `192.168.1.204` |
| `AD-WIN10-02` | Router DHCP               | `192.168.1.102` |
| EE Router     | Gateway and DHCP provider | `192.168.1.254` |

This split keeps the environment simple. The router handles DHCP for the client machines, while the server keeps a fixed address for Active Directory and DNS services.

## DNS Configuration

`AD-SRV01` is configured to use itself for DNS. This allows it to provide name resolution for the `adbox.local` domain.

![Server IP Configuration](/screenshots/lab/02-environment-setup/03-server-ipconfig-dns.png)

The Windows 10 clients receive their IPv4 addresses from the router, but their DNS server is manually set to `192.168.1.50`. This ensures that domain lookups for `adbox.local` go to the DC instead of the home router.

![Client IP Configuration](/screenshots/lab/02-environment-setup/04-client-ipconfig-dns.png)

The router remains responsible for address assignment, while `AD-SRV01` handles Active Directory DNS for the lab domain.

## IPv6 Adapter Setting

Internet Protocol version 6 (IPv6) was disabled on the Windows 10 lab adapters after testing showed that the clients were receiving router-provided IPv6 DNS information.

![Client IPv6 Disabled](/screenshots/lab/02-environment-setup/05-client-ipv6-disabled.png)

This keeps the lab on a controlled IPv4 path so the clients use `AD-SRV01` for domain resolution. The full troubleshooting record is documented in [DNS IPv6 Conflict](../troubleshooting/01-dns-ipv6-conflict.md).

## Validation Checks

Before moving into the domain build, each Windows 10 client must be able to reach the server and resolve the lab domain.

Check connectivity to the server:

```text
ping 192.168.1.50
```

Check domain resolution:

```text
nslookup adbox.local
```

Check the server Fully Qualified Domain Name (FQDN):

```text
nslookup AD-SRV01.adbox.local
```

These checks confirm that the clients can reach the server, resolve the domain, and locate `AD-SRV01` by FQDN before the domain join stage.

## Next Stage

[Stage 3: Domain Controller](03-domain-controller.md), which documents the AD DS role installation, `adbox.local` forest creation, DC promotion, DNS role validation, and post-promotion checks.
