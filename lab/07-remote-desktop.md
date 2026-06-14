# Remote Desktop

Remote Desktop Protocol (RDP) allows a user to connect to and control a Windows computer remotely over the network.

This stage tests RDP access to `AD-WIN10-01` after the workstation has joined the `adbox.local` domain. The aim is to prove that the client can accept remote connections, that an Active Directory security group can be added to the local RDP access list, and that a remote session can be validated from inside the workstation.

## RDP Target

The RDP target for this stage was the domain-joined Windows 10 client.

| Item            | Value                 |
| --------------- | --------------------- |
| Target computer | `AD-WIN10-01`         |
| Domain          | `adbox.local`         |
| NetBIOS domain  | `ADBOX`               |
| Access group    | `GG_RDP_Allowed`      |
| Test account    | `ADBOX\Administrator` |

The `GG_RDP_Allowed` group was created during the directory structure stage and added to the workstation's allowed RDP users list. The final connection test used the domain administrator account, which already has remote access by default. The group configuration still documents how delegated RDP access would be assigned to normal users in the lab.

## Remote Desktop Enabled

Remote Desktop was enabled on `AD-WIN10-01` through Windows Settings.

![RDP Settings Enabled](/screenshots/lab/07-remote-desktop/01-rdp-settings-enabled.png)

Windows displayed a confirmation prompt explaining that selected users would be able to connect to the PC remotely.

![RDP Enable Confirmation](/screenshots/lab/07-remote-desktop/02-rdp-enable-confirmation.png)

This confirmed that RDP was being enabled on the workstation before any remote connection was attempted.

## RDP Access Group

The `GG_RDP_Allowed` security group was selected as the access-control group for remote access.

![RDP Group Selected](/screenshots/lab/07-remote-desktop/03-rdp-group-selected.png)

The group was then visible in the Remote Desktop Users list on `AD-WIN10-01`.

![RDP Group Added](/screenshots/lab/07-remote-desktop/04-rdp-group-added.png)

This means members of `ADBOX\GG_RDP_Allowed` are allowed to connect to the workstation through RDP. The dialog also shows that `ADBOX\Administrator` already has access, which is expected for an administrator account.

## Remote Desktop Connection

Remote Desktop Connection was opened from another Windows machine and pointed at the workstation hostname.

![RDP Connection Target](/screenshots/lab/07-remote-desktop/05-rdp-connection-target.png)

The connection target was:

```text
AD-WIN10-01
```

Windows then requested credentials for the remote connection.

![RDP Credentials Entered](/screenshots/lab/07-remote-desktop/06-rdp-credentials-entered.png)

The test connection used:

```text
ADBOX\Administrator
```

## Session Validation

After connection, Command Prompt (CMD) was used inside the RDP session to confirm the user, workstation, session type, and network context.

The validation commands used were:

```text
whoami
hostname
query user
ipconfig /all
```

![RDP Session Identity](/screenshots/lab/07-remote-desktop/07-rdp-session-identity.png)

The command output confirmed:

| Evidence                                               | Meaning                                                             |
| ------------------------------------------------------ | ------------------------------------------------------------------- |
| `whoami` returned `adbox\administrator`                | The session was authenticated with a domain account.                |
| `hostname` returned `AD-WIN10-01`                      | The session was running on the remote workstation.                  |
| `query user` showed `rdp-tcp#1`                        | The session was an RDP session, not a normal local console session. |
| `ipconfig /all` showed `AD-WIN10-01` and `adbox.local` | The machine was operating in the ADBox domain network context.      |

This proves that the remote connection reached the domain-joined workstation and opened an interactive RDP session.

## RDP Session Behaviour

During testing, the VirtualBox console for `AD-WIN10-01` locked while the RDP session was active. This was expected behaviour because RDP creates an interactive remote logon session rather than mirroring the local VirtualBox display.

The remote session was ended cleanly by signing out from inside Windows. Closing the RDP window would disconnect the session, while signing out fully ends the remote user session.

## Summary

RDP was enabled on `AD-WIN10-01`, the `GG_RDP_Allowed` security group was added to the local Remote Desktop Users list, and a remote session was opened successfully using domain credentials.

The session was validated from inside the workstation using CMD. The output confirmed the signed-in domain account, the remote workstation hostname, and the `rdp-tcp` session type.

This stage adds a practical support workflow to ADBox: remotely accessing a domain-joined workstation for administration or troubleshooting.

## Next Stage

[Stage 8: File Sharing](08-file-sharing.md), which tests shared folder access, security group permissions, and domain user validation.
