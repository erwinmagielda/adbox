# Remote Desktop

This stage tests Remote Desktop access to a domain-joined Windows 10 workstation.

The goal is to confirm that `AD-WIN10-01` can accept remote connections, that an Active Directory group can be added to the local Remote Desktop Users list, and that the session can be checked from inside the remote workstation.

## RDP Target

The test target was `AD-WIN10-01`, one of the Windows 10 clients joined to `adbox.local`.

Area | Value
--- | ---
Target Computer | `AD-WIN10-01`
Domain | `adbox.local`
NetBIOS Domain | `ADBOX`
Access Group | `GG_RDP_Allowed`
Test Account | `ADBOX\Administrator`

`GG_RDP_Allowed` was created during the directory structure stage as the access group for Remote Desktop testing.

>The final connection used `ADBOX\Administrator`, which already has remote access by default. The group setup is still useful because it shows how Remote Desktop access can be delegated to normal domain users through group membership.

## Remote Desktop Enabled

Remote Desktop was enabled on `AD-WIN10-01` through Windows Settings.

![RDP Settings Enabled](/screenshots/lab/07-remote-desktop/01-rdp-settings-enabled.png)

Windows displayed a confirmation prompt before enabling remote access.

![RDP Enable Confirmation](/screenshots/lab/07-remote-desktop/02-rdp-enable-confirmation.png)

This confirmed that the workstation was configured to accept Remote Desktop connections before the connection test was attempted.

>RDP needs to be enabled on the target machine. Joining a workstation to the domain does not automatically mean it will accept remote desktop sessions.

## RDP Access Group

The `GG_RDP_Allowed` security group was selected as the access-control group for remote access.

![RDP Group Selected](/screenshots/lab/07-remote-desktop/03-rdp-group-selected.png)

The group was then visible in the Remote Desktop Users list on `AD-WIN10-01`.

![RDP Group Added](/screenshots/lab/07-remote-desktop/04-rdp-group-added.png)

This means members of `ADBOX\GG_RDP_Allowed` can be granted Remote Desktop access to the workstation.

>Adding a domain group to the local Remote Desktop Users list connects Active Directory group membership with local workstation access. A user can be managed centrally in AD, while the workstation still controls who is allowed to connect remotely.

## Remote Desktop Connection

Remote Desktop Connection was opened from another Windows machine and pointed at the workstation hostname.

![RDP Connection Target](/screenshots/lab/07-remote-desktop/05-rdp-connection-target.png)

The connection target was:

```text
AD-WIN10-01
```

Windows then requested credentials for the remote session.

![RDP Credentials Entered](/screenshots/lab/07-remote-desktop/06-rdp-credentials-entered.png)

The test connection used:

```text
ADBOX\Administrator
```

Using the hostname confirmed that the connection could reach the workstation by name inside the lab environment.

## Session Validation

After connection, Command Prompt was used inside the RDP session to confirm the account, workstation, session type, and domain network context.

```cmd
whoami
hostname
query user
ipconfig /all
```

![RDP Session Identity](/screenshots/lab/07-remote-desktop/07-rdp-session-identity.png)

Validation Check | Confirmed Result
--- | ---
Signed-In Account | `whoami` returned `adbox\administrator`.
Remote Workstation | `hostname` returned `AD-WIN10-01`.
Session Type | `query user` showed an `rdp-tcp` session.
Domain Context | `ipconfig /all` showed the workstation and `adbox.local` DNS context.

This confirmed that the connection reached the domain-joined workstation and opened an interactive Remote Desktop session.

>`query user` is useful here because it shows the session name. An `rdp-tcp` session confirms that the user is connected through Remote Desktop, not only using the local VirtualBox console.

## Session Behaviour

During testing, the VirtualBox console for `AD-WIN10-01` locked while the RDP session was active.

This was expected because Remote Desktop creates an interactive remote logon session. It does not simply mirror the local VirtualBox screen.

The session was ended cleanly by signing out from inside Windows. Closing the RDP window would disconnect the session, while signing out ends the remote user session properly.

## Result

Remote Desktop was enabled, the access group was added, and a remote session was opened successfully using domain credentials.

Check | Outcome
--- | ---
RDP Enabled On `AD-WIN10-01` | Passed
Remote Desktop Confirmation Accepted | Passed
`GG_RDP_Allowed` Added To Local RDP Access | Passed
Connection Opened To `AD-WIN10-01` | Passed
Domain Credentials Accepted | Passed
Remote Session Validated With Commands | Passed
`rdp-tcp` Session Confirmed | Passed

This stage proves that a domain-joined Windows workstation can be prepared for remote support and validated from inside the remote session.

## Navigation

Previous | Current | Next
--- | --- | ---
[06 Group Policy](06-group-policy.md) | Remote Desktop | [08 File Sharing](08-file-sharing.md)
