# File Sharing

This stage configured a department file share on `AD-SRV01` and tested access from a domain-joined Windows 10 client.

The purpose was to connect the Active Directory structure from the previous stages to a practical support task: giving a department user access to a shared folder through group membership and Windows permissions.

The test used the Sales department as the first example.

| Item           | Value                   |
| -------------- | ----------------------- |
| Server         | `AD-SRV01`              |
| Client         | `AD-WIN10-01`           |
| Share          | `Sales`                 |
| Share Path     | `\\AD-SRV01\Sales`      |
| Local Folder   | `C:\ADBox-Shares\Sales` |
| Security Group | `GG_Sales_Users`        |
| Test User      | `Jamie Carter`          |
| Test Account   | `ADBOX\jamie.carter`    |

## Access Model

The file share uses a simple access chain.

```text
User Account -> Security Group -> Shared Folder -> Share Permissions -> NTFS Permissions -> Client Test
```

`Jamie Carter` is a member of `GG_Sales_Users`. The `Sales` folder was shared from `AD-SRV01`, and `GG_Sales_Users` was given folder access through NTFS permissions.

![Sales Group Membership](../screenshots/lab/08-file-sharing/05-sales-group-membership.png)

## Folder Creation

A new folder path was created on `AD-SRV01` for ADBox shared resources.

```text
C:\ADBox-Shares\Sales
```

A test file was created inside the folder to confirm that the share contents could be listed from a client later.

```text
sales-access-test.txt
```

![Sales Folder Created](../screenshots/lab/08-file-sharing/01-sales-folder-created.png)

## Advanced Sharing Configuration

The `Sales` folder was shared using Advanced Sharing.

Advanced Sharing was used because it exposes the share name, simultaneous user limit, caching options, and share permissions in one place. This makes the configuration easier to document and validate.

The simpler Sharing wizard is useful for quick folder sharing, but Advanced Sharing gives clearer control over the settings being tested in this lab.

| Setting           | Value                                              |
| ----------------- | -------------------------------------------------- |
| Share This Folder | Enabled                                            |
| Share Name        | `Sales`                                            |
| User Limit        | `10`                                               |
| Comment           | `Sales department share for ADBox access testing.` |

The simultaneous user limit controls how many network users can connect to the share at the same time. It does not grant or deny access by itself. Access is controlled through share permissions and NTFS permissions.

For this lab, the limit was left at `10`, which is enough for the small ADBox test environment and keeps the share configuration easy to review.

![Sales Advanced Sharing](../screenshots/lab/08-file-sharing/02-sales-advanced-sharing.png)

## Share Permissions

Share permissions control access to the folder when it is reached through the network path.

```text
\\AD-SRV01\Sales
```

For this lab, `Everyone` was given `Change` and `Read` at the share-permission layer.

| Principal  | Full Control | Change | Read |
| ---------- | ------------ | ------ | ---- |
| `Everyone` | No           | Yes    | Yes  |

`Full Control` was left unticked because it gives more access than needed for this test. `Change` allows users to read, create, edit, and delete files through the share. `Full Control` also allows permission-level control at the share layer, which is unnecessary for a normal department share test.

This keeps the share permission open enough for network access while leaving the real access decision to NTFS permissions.

![Share Permissions Set](../screenshots/lab/08-file-sharing/03-share-permissions-set.png)

## NTFS Permissions

New Technology File System (NTFS) permissions are the file-system permissions stored on the folder itself. They apply to the folder on disk, whether the folder is accessed locally on the server or remotely through a share.

For shared folders, Windows checks both permission layers.

| Permission Layer  | Purpose                                                 |
| ----------------- | ------------------------------------------------------- |
| Share Permissions | Controls access through the network share path.         |
| NTFS Permissions  | Controls access to the actual folder and files on disk. |

The effective access is based on the most restrictive result between the two layers.

For the `Sales` folder, the Sales security group was given `Modify` access through NTFS permissions.

| Principal        | Permission   |
| ---------------- | ------------ |
| `GG_Sales_Users` | Modify       |
| `Administrators` | Full Control |
| `SYSTEM`         | Full Control |

`Modify` was used because it is suitable for normal department file work. It allows users to read, create, edit, and delete files without giving them control over folder ownership or permission management.

`Full Control` was not given to `GG_Sales_Users` because department users do not need to change permissions or take ownership of the folder.

![NTFS Group Permissions](../screenshots/lab/08-file-sharing/04-ntfs-group-permissions.png)

## Client Sign-In

The access test was performed from `AD-WIN10-01` using the Sales user account.

```text
ADBOX\jamie.carter
```

Signing in as the department user proves that access is being tested through the intended Active Directory account and group membership.

## Client Access Test

The share was tested from `AD-WIN10-01` using the network path.

```cmd
whoami
hostname
dir \\AD-SRV01\Sales
```

The command output confirmed that `ADBOX\jamie.carter` was signed in on `AD-WIN10-01` and could list the contents of the `Sales` share.

![Client Share Access](../screenshots/lab/08-file-sharing/06-client-share-access.png)

## Client Write Test

A file was then created in the shared folder from the client using PowerShell.

```text
New-Item \\AD-SRV01\Sales\client-created-document.txt -ItemType File
dir \\AD-SRV01\Sales
```

The output confirmed that the Sales user could create a file inside the share and then list it from the client.

![Client Created Document](../screenshots/lab/08-file-sharing/07-client-created-document.png)

## Validation Summary

| Check                                                | Result |
| ---------------------------------------------------- | ------ |
| Sales folder created on `AD-SRV01`                   | Passed |
| Folder shared as `Sales`                             | Passed |
| Share comment added                                  | Passed |
| Share user limit left at `10`                        | Passed |
| Share permissions configured                         | Passed |
| `GG_Sales_Users` added to NTFS permissions           | Passed |
| Jamie Carter confirmed as member of `GG_Sales_Users` | Passed |
| Jamie Carter signed in on `AD-WIN10-01`              | Passed |
| Client listed share contents as `ADBOX\jamie.carter` | Passed |
| Client created file in share as `ADBOX\jamie.carter` | Passed |

## Support Notes

This stage shows how a basic department share can be prepared and tested in a Windows domain.

The key point is that sharing a folder and securing a folder are related but separate tasks. Advanced Sharing creates the network share and controls share-level access. NTFS permissions control what users and groups can actually do with the folder and files.

For normal department access, `Modify` is usually enough. It allows everyday file work without giving users permission-management control over the folder.

Testing with `ADBOX\jamie.carter` confirms that access works through the Sales user account and `GG_Sales_Users` group membership.
