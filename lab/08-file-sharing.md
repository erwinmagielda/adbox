# File Sharing

File sharing is where the lab moves from directory objects into a normal support problem: can the right user access the right folder from a domain client?

This stage creates a Sales department share on `AD-SRV01`, applies group-based NTFS permissions, signs in as the Sales user on `AD-WIN10-01`, and confirms both read and write access.

## Share Target

The first share test uses the Sales department.

| Area           | Value                   |
| -------------- | ----------------------- |
| Server         | `AD-SRV01`              |
| Client         | `AD-WIN10-01`           |
| Share Name     | `Sales`                 |
| Share Path     | `\\AD-SRV01\Sales`      |
| Local Folder   | `C:\ADBox-Shares\Sales` |
| Security Group | `GG_Sales_Users`        |
| Test User      | Jamie Carter            |
| Test Account   | `ADBOX\jamie.carter`    |

The share is hosted on the server and tested from the client while signed in as the Sales user.

`\\AD-SRV01\Sales` is a Universal Naming Convention (UNC) path. It points to a network share by using the server name and share name instead of a local drive path.

## Access Chain

The access model uses a simple chain from the user account to the shared folder.

```text
User Account -> Security Group -> Shared Folder -> Share Permissions -> NTFS Permissions -> Client Test
```

`Jamie Carter` is a member of `GG_Sales_Users`. The `Sales` folder is shared from `AD-SRV01`, and the Sales group is given folder access through NTFS permissions.

This stage checks both permission layers used by Windows file sharing:

| Layer             | Purpose                                                      |
| ----------------- | ------------------------------------------------------------ |
| Share Permissions | Controls access when the folder is reached over the network. |
| NTFS Permissions  | Controls access to the actual folder and files on disk.      |

Windows checks both layers. The effective access is limited by the most restrictive result.

## Folder Creation

A new folder was created on `AD-SRV01` for the Sales share.

### Work Path

```text
AD-SRV01 -> File Explorer -> C:\ADBox-Shares
```

### Action

```text
Create Sales folder.
Add sales-access-test.txt.
```

The local folder path was:

```text
C:\ADBox-Shares\Sales
```

A test file was added so the client could later prove that the share contents were visible.

```text
sales-access-test.txt
```

Figure 8.1 shows the Sales folder and test file created on the server.

![Figure 8.1 - Sales folder created](../screenshots/lab/08-file-sharing/01-sales-folder-created.png)

*Figure 8.1 - File Explorer on `AD-SRV01` showing the `Sales` folder under `C:\ADBox-Shares` with the initial test file created.*

## Advanced Sharing

The `Sales` folder was shared using Advanced Sharing.

### Work Path

```text
File Explorer -> C:\ADBox-Shares -> Sales -> Properties -> Sharing -> Advanced Sharing
```

### Action

```text
Tick Share this folder.
Set share name to Sales.
Add a share comment.
Leave the simultaneous user limit at 10.
```

Advanced Sharing was used because it exposes the share name, user limit, comment, and share permissions in one place.

| Setting           | Value                                              |
| ----------------- | -------------------------------------------------- |
| Share This Folder | Enabled                                            |
| Share Name        | `Sales`                                            |
| User Limit        | `10`                                               |
| Comment           | `Sales department share for ADBox access testing.` |

Figure 8.2 shows the Advanced Sharing settings for the Sales share.

![Figure 8.2 - Sales advanced sharing](../screenshots/lab/08-file-sharing/02-sales-advanced-sharing.png)

*Figure 8.2 - Advanced Sharing settings showing the `Sales` share name, share comment, and simultaneous user limit.*

The user limit controls how many network users can connect to the share at the same time. It does not decide who is allowed to access the folder.

## Share Permissions

Share permissions control access when the folder is reached through the network share path.

### Work Path

```text
Sales Properties -> Sharing -> Advanced Sharing -> Permissions
```

### Action

```text
Set share-level permissions for Everyone.
Allow Change and Read.
Leave Full Control unticked.
```

The network share path is:

```text
\\AD-SRV01\Sales
```

For this lab, `Everyone` was given `Change` and `Read` at the share permission layer.

| Principal  | Full Control | Change | Read |
| ---------- | ------------ | ------ | ---- |
| `Everyone` | No           | Yes    | Yes  |

Figure 8.3 shows the share permissions set for `Everyone`.

![Figure 8.3 - Share permissions set](../screenshots/lab/08-file-sharing/03-share-permissions-set.png)

*Figure 8.3 - Share permissions showing `Everyone` allowed `Change` and `Read`, with `Full Control` left unticked.*

`Full Control` was left unticked because it gives more access than needed for this department share test. `Change` allows normal file work through the share, while permission management stays restricted.

In this lab, the share permission is broad enough to allow network access, while NTFS permissions decide which domain group can actually work with the folder.

## NTFS Permissions

New Technology File System (NTFS) permissions control access to the actual folder and files on disk.

### Work Path

```text
File Explorer -> C:\ADBox-Shares -> Sales -> Properties -> Security -> Edit -> Add
```

### Action

```text
Add GG_Sales_Users.
Assign Modify permission.
Leave Full Control unselected for the Sales group.
```

For the `Sales` folder, the Sales security group was given `Modify` access.

| Principal        | Permission   |
| ---------------- | ------------ |
| `GG_Sales_Users` | Modify       |
| `Administrators` | Full Control |
| `SYSTEM`         | Full Control |

Figure 8.4 shows the NTFS permissions applied to the Sales folder.

![Figure 8.4 - NTFS group permissions](../screenshots/lab/08-file-sharing/04-ntfs-group-permissions.png)

*Figure 8.4 - Security tab showing `GG_Sales_Users` added to the Sales folder with Modify access.*

`Modify` was used because it allows normal department file work: read, create, edit, and delete. `Full Control` was not needed for the Sales group because department users do not need to change folder permissions or ownership.

## Group Membership Check

Before testing from the client, `Jamie Carter` was confirmed as a member of `GG_Sales_Users`.

### Work Path

```text
Server Manager -> Tools -> Active Directory Users and Computers -> ADBox-Lab -> Groups -> Security -> GG_Sales_Users -> Members
```

### Shortcut

```text
Win + R -> dsa.msc
```

### Check

```text
Confirm Jamie Carter is listed as a member of GG_Sales_Users.
```

Figure 8.5 shows the Sales group membership.

![Figure 8.5 - Sales group membership](../screenshots/lab/08-file-sharing/05-sales-group-membership.png)

*Figure 8.5 - Active Directory Users and Computers showing `Jamie Carter` as a member of `GG_Sales_Users`.*

This confirms that the file-share test uses the intended department user and group membership.

## Client Sign-In

The access test was performed from `AD-WIN10-01` using the Sales user account.

### Work Path

```text
AD-WIN10-01 -> Windows sign-in screen -> Other user
```

### Sign-In Account

```text
ADBOX\jamie.carter
```

Signing in as the department user confirms that access is being tested through the intended Active Directory account and group membership.

## Client Access Test

The share was tested from `AD-WIN10-01` using Command Prompt.

### Work Path

```text
Win + R -> cmd
```

### Run On

```text
AD-WIN10-01 while signed in as ADBOX\jamie.carter
```

```cmd
whoami
hostname
dir \\AD-SRV01\Sales
```

Figure 8.6 shows the share access test from the client.

![Figure 8.6 - Client share access](../screenshots/lab/08-file-sharing/06-client-share-access.png)

*Figure 8.6 - Command Prompt on `AD-WIN10-01` showing `adbox\jamie.carter`, the client hostname, and the contents of `\\AD-SRV01\Sales`.*

| Validation Check   | Confirmed Result                                        |
| ------------------ | ------------------------------------------------------- |
| Signed-In Account  | `whoami` returned `adbox\jamie.carter`.                 |
| Client Workstation | `hostname` returned `AD-WIN10-01`.                      |
| Share Listing      | `dir \\AD-SRV01\Sales` listed the Sales share contents. |

This confirmed that the Sales user could reach and list the shared folder from the domain-joined client.

## Client Write Test

A file was then created in the shared folder from the client using PowerShell.

### Work Path

```text
AD-WIN10-01 -> PowerShell
```

### Run As

```text
ADBOX\jamie.carter
```

```powershell
New-Item \\AD-SRV01\Sales\client-created-document.txt -ItemType File
dir \\AD-SRV01\Sales
```

Figure 8.7 shows the client-created file inside the share.

![Figure 8.7 - Client created document](../screenshots/lab/08-file-sharing/07-client-created-document.png)

*Figure 8.7 - PowerShell on `AD-WIN10-01` showing a file created in `\\AD-SRV01\Sales` by the Sales user.*

The output confirmed that the Sales user could create a file inside the share and then list it from the client.

This proves that the user had more than read-only access. The `Modify` permission allowed the normal department file action being tested.

## Result

The Sales department share was created, secured with group-based NTFS permissions, and tested from a domain-joined Windows 10 client.

| Check                                      | Outcome |
| ------------------------------------------ | ------- |
| Sales folder created on `AD-SRV01`         | Passed  |
| Folder shared as `Sales`                   | Passed  |
| Share permissions configured               | Passed  |
| `GG_Sales_Users` added to NTFS permissions | Passed  |
| Jamie Carter confirmed in `GG_Sales_Users` | Passed  |
| Jamie Carter signed in on `AD-WIN10-01`    | Passed  |
| Client listed share contents               | Passed  |
| Client created file in share               | Passed  |

This stage proves that Active Directory group membership can be used to control access to a shared folder in the lab domain.

## Navigation

| Previous                                  | Current         | Next                                          |
| ----------------------------------------- | --------------- | --------------------------------------------- |
| [07 Remote Desktop](07-remote-desktop.md) | 08 File Sharing | [09 Account Recovery](09-account-recovery.md) |
