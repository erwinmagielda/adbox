# DNS IPv6 Conflict

## Discovery

During client validation, the Windows 10 lab clients could reach `AD-SRV01` by Internet Protocol version 4 (IPv4) address. This confirmed that basic network connectivity between the client and the Domain Controller (DC) was working.

![Client Ping Success](/screenshots/troubleshooting/01-dns-ipv6-conflict/01-client-ping-success.png)

A direct Domain Name System (DNS) query to `192.168.1.50` also resolved `adbox.local`. This confirmed that the DNS service on `AD-SRV01` was working when the client was told to query the DC directly.

![Direct Nslookup Success](/screenshots/troubleshooting/01-dns-ipv6-conflict/02-direct-nslookup-success.png)

However, a standard `nslookup adbox.local` query timed out. This showed that the issue was not basic connectivity or the DNS service itself. The problem was the default DNS path being used by the client.

![Default Nslookup Timeout](/screenshots/troubleshooting/01-dns-ipv6-conflict/03-default-nslookup-timeout.png)

## Actions

The client adapter configuration was reviewed after the timeout. The IPv4 DNS setting was already pointing to `192.168.1.50`, which meant the intended lab DNS server had been configured correctly.

The issue was isolated by comparing direct and default DNS lookup behaviour:

```text
nslookup adbox.local 192.168.1.50
```

This direct query worked because it forced the client to use `AD-SRV01`.

```text
nslookup adbox.local
```

This default query timed out because the client was still receiving Internet Protocol version 6 (IPv6) DNS information from the EE router.

IPv6 was disabled on the Windows 10 lab adapter to keep DNS resolution controlled through the IPv4-based Active Directory lab path.

![Client IPv6 Disabled](/screenshots/troubleshooting/01-dns-ipv6-conflict/04-client-ipv6-disabled.png)

## Resolution

After IPv6 was disabled on the lab adapter, the standard `nslookup adbox.local` query resolved successfully through `AD-SRV01`.

![Default Nslookup Success](/screenshots/troubleshooting/01-dns-ipv6-conflict/05-default-nslookup-success.png)

The issue was resolved by forcing the lab client to use the intended IPv4 DNS path for the `adbox.local` domain. This confirmed that the DC and DNS service were working correctly, and that the failure was caused by the client preferring the router-provided IPv6 DNS path during default lookup.

This change was applied only to the Windows 10 lab adapters to keep the home lab controlled and predictable. It was not treated as a general recommendation to disable IPv6 outside this lab environment.