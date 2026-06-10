# Client Firewall Ping

## Discovery

During connectivity testing, the Windows 10 lab client could reach `AD-SRV01` by Internet Protocol version 4 (IPv4) address. This confirmed that client-to-server connectivity was working.

![Client Ping Success](/screenshots/troubleshooting/02-client-firewall-ping/01-client-ping-success.png)

However, `AD-SRV01` could not ping the Windows 10 client. The ping requests timed out, showing that the server could not receive an Internet Control Message Protocol (ICMP) echo reply from the client.

![Server Ping Failed](/screenshots/troubleshooting/02-client-firewall-ping/02-server-ping-failed.png)

The client firewall profile was checked using `netsh advfirewall show currentprofile`. The active profile was the Public Profile, with inbound traffic blocked by default.

![Client Firewall Profile](/screenshots/troubleshooting/02-client-firewall-ping/03-client-firewall-profile.png)

This showed that the issue was not a full network failure. The client could reach the Domain Controller (DC), but inbound ping requests to the client were being blocked by Windows Firewall.

## Actions

The issue was isolated by comparing traffic direction:

```text
ping 192.168.1.50
```

This worked from the Windows 10 client because outbound traffic from the client to `AD-SRV01` was allowed.

```text
ping 192.168.1.204
```

This failed from `AD-SRV01` because inbound ICMP echo requests to the Windows 10 client were blocked.

To allow ping testing in the lab, Command Prompt was opened as Administrator on the Windows 10 client. The inbound ICMPv4 echo rule was then enabled:

```text
netsh advfirewall firewall set rule name="File and Printer Sharing (Echo Request - ICMPv4-In)" new enable=yes
```

The command updated the matching firewall rules successfully.

![ICMP Rule Enabled](/screenshots/troubleshooting/02-client-firewall-ping/04-icmp-rule-enabled.png)

## Resolution

After the ICMPv4 echo rule was enabled, `AD-SRV01` could successfully ping the Windows 10 client.

![Server Ping Success](/screenshots/troubleshooting/02-client-firewall-ping/05-server-ping-success.png)

The issue was resolved by enabling the Windows Firewall rule that allows inbound ICMPv4 echo requests on the lab client. This confirmed that the earlier timeout was caused by client-side firewall behaviour, not by a broken network path.

This issue affected server-to-client ping testing only. It did not prevent domain join, because the client could already reach the DC and use the lab DNS path required for Active Directory domain communication.