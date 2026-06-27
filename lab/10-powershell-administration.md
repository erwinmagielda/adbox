# PowerShell Support Administration

This stage used Windows PowerShell on `AD-SRV01` to inspect Active Directory users, groups, computers, account status, DNS resolution, client connectivity, shared folder access, and a basic exported user report.

The purpose was to practise practical support checks that can be used before escalation or handover. Earlier stages used graphical tools such as Active Directory Users and Computers. This stage confirmed that the same environment could also be reviewed from the command line.

| Item          | Value                                    |
| ------------- | ---------------------------------------- |
| Server        | `AD-SRV01`                               |
| Domain        | `adbox.local`                            |
| Tool          | Windows PowerShell                       |
| Module        | Active Directory                         |
| Client Checks | `AD-WIN10-01`, `AD-WIN10-02`             |
| Share Checked | `\\AD-SRV01\Sales`                       |
| Report Path   | `C:\ADBox-Shares\adbox-users-report.csv` |

## Support Command Scope

This stage focused on simple PowerShell checks that are useful in a Windows support environment.

```text
Active Directory module -> Users -> Account details -> Group membership -> Computers -> Disabled users -> DNS -> Connectivity -> Share path -> CSV report
```

The aim was not to create a large script. The commands were kept as short checks that can be read, repeated, and understood during support work.

## Active Directory Module

The Active Directory module was imported on `AD-SRV01`.

```powershell
Import-Module ActiveDirectory

Get-Command -Module ActiveDirectory | Select-Object -First 10
```

### Command Breakdown

| Part                            | Purpose                                                                |
| ------------------------------- | ---------------------------------------------------------------------- |
| `Import-Module ActiveDirectory` | Loads the Active Directory PowerShell module into the current session. |
| `Get-Command`                   | Lists commands available in PowerShell.                                |
| `-Module ActiveDirectory`       | Limits the command list to Active Directory commands.                  |
| `Select-Object -First 10`       | Shows only the first 10 results to keep the output readable.           |

This confirmed that Active Directory PowerShell commands were available on the server.

![PowerShell AD Module](../screenshots/lab/10-powershell-support-administration/01-powershell-ad-module.png)

## ADBox User List

The ADBox lab users were queried from the lab Users OU.

```powershell
Get-ADUser -Filter * -SearchBase "OU=Users,OU=ADBox-Lab,DC=adbox,DC=local" |
Select-Object Name, SamAccountName, Enabled
```

### Command Breakdown

| Part                                                    | Purpose                                               |                                                              |
| ------------------------------------------------------- | ----------------------------------------------------- | ------------------------------------------------------------ |
| `Get-ADUser`                                            | Queries Active Directory user accounts.               |                                                              |
| `-Filter *`                                             | Returns all users that match the search location.     |                                                              |
| `-SearchBase "OU=Users,OU=ADBox-Lab,DC=adbox,DC=local"` | Limits the search to the ADBox lab Users OU.          |                                                              |
| `                                                       | `                                                     | Sends the output of the first command into the next command. |
| `Select-Object Name, SamAccountName, Enabled`           | Shows only the selected fields needed for this check. |                                                              |

The output listed the lab-created user accounts and showed that they were enabled.

![Get AD Users](../screenshots/lab/10-powershell-support-administration/02-get-ad-users.png)

| User         | Account        | Enabled |
| ------------ | -------------- | ------- |
| Alex Morgan  | `alex.morgan`  | True    |
| Jamie Carter | `jamie.carter` | True    |
| Sam Taylor   | `sam.taylor`   | True    |

## User Account Details

Sam Taylor was checked directly to review account status after the previous account recovery stage.

```powershell
Get-ADUser sam.taylor -Properties Enabled, PasswordLastSet, PasswordExpired, CannotChangePassword |
Select-Object Name, SamAccountName, Enabled, PasswordLastSet, PasswordExpired, CannotChangePassword
```

### Command Breakdown

| Part                                                                          | Purpose                                                                 |
| ----------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| `Get-ADUser sam.taylor`                                                       | Queries the Sam Taylor user account by logon name.                      |
| `-Properties Enabled, PasswordLastSet, PasswordExpired, CannotChangePassword` | Requests extra account properties that are not always shown by default. |
| `PasswordLastSet`                                                             | Shows when the password was last changed or reset.                      |
| `PasswordExpired`                                                             | Shows whether the password is currently expired.                        |
| `CannotChangePassword`                                                        | Shows whether the user is blocked from changing their own password.     |
| `Select-Object ...`                                                           | Displays only the fields relevant to the account recovery check.        |

The output confirmed that the account was enabled, the password had been set, the password was not expired, and the user was allowed to change password.

![Get AD User](../screenshots/lab/10-powershell-support-administration/03-get-ad-user.png)

## User Group Membership

Sam Taylor’s group membership was checked from PowerShell.

```powershell
Get-ADPrincipalGroupMembership sam.taylor |
Select-Object Name, GroupCategory, GroupScope
```

### Command Breakdown

| Part                                        | Purpose                                                            |
| ------------------------------------------- | ------------------------------------------------------------------ |
| `Get-ADPrincipalGroupMembership sam.taylor` | Lists the Active Directory groups that Sam Taylor belongs to.      |
| `Name`                                      | Shows the group name.                                              |
| `GroupCategory`                             | Shows whether the group is a Security group or Distribution group. |
| `GroupScope`                                | Shows whether the group is Global, Domain Local, or Universal.     |

The output confirmed that Sam Taylor belonged to `Domain Users` and `GG_Warehouse_Users`.

![User Group Membership](../screenshots/lab/10-powershell-support-administration/04-user-group-membership.png)

| Group              | Category | Scope  |
| ------------------ | -------- | ------ |
| Domain Users       | Security | Global |
| GG_Warehouse_Users | Security | Global |

## Domain Computer Objects

The domain computer objects were listed from Active Directory.

```powershell
Get-ADComputer -Filter * |
Select-Object Name, Enabled, DistinguishedName
```

### Command Breakdown

| Part                | Purpose                                                       |
| ------------------- | ------------------------------------------------------------- |
| `Get-ADComputer`    | Queries computer objects in Active Directory.                 |
| `-Filter *`         | Returns all computer objects in the domain.                   |
| `Name`              | Shows the computer name.                                      |
| `Enabled`           | Shows whether the computer account is enabled.                |
| `DistinguishedName` | Shows where the computer object sits inside Active Directory. |

The output showed the server and both Windows 10 clients.

![Get Domain Computers](../screenshots/lab/10-powershell-support-administration/05-get-domain-computers.png)

| Computer      | Purpose                         |
| ------------- | ------------------------------- |
| `AD-SRV01`    | Domain Controller               |
| `AD-WIN10-01` | Domain-joined Windows 10 client |
| `AD-WIN10-02` | Domain-joined Windows 10 client |

## Disabled Lab User Check

Disabled user accounts were checked inside the ADBox lab Users OU.

```powershell
$disabledLabUsers = Search-ADAccount -AccountDisabled -UsersOnly -SearchBase "OU=Users,OU=ADBox-Lab,DC=adbox,DC=local"

$disabledLabUsers |
Select-Object Name, SamAccountName, Enabled

"Disabled ADBox lab users: $($disabledLabUsers.Count)"
```

### Command Breakdown

| Part                      | Purpose                                                      |
| ------------------------- | ------------------------------------------------------------ |
| `$disabledLabUsers = ...` | Stores the command result in a variable so it can be reused. |
| `Search-ADAccount`        | Searches Active Directory accounts by account condition.     |
| `-AccountDisabled`        | Searches for disabled accounts.                              |
| `-UsersOnly`              | Limits the search to user accounts.                          |
| `-SearchBase ...`         | Limits the search to the ADBox lab Users OU.                 |
| `$disabledLabUsers.Count` | Counts how many disabled lab users were found.               |

The check was scoped to the ADBox lab Users OU so the result focused on lab-created accounts and excluded built-in domain accounts such as `Guest` and `krbtgt`.

![Get Disabled Users](../screenshots/lab/10-powershell-support-administration/06-get-disabled-users.png)

## DNS Resolution Check

Domain Name System (DNS) resolution was checked for `adbox.local`.

```powershell
Resolve-DnsName adbox.local -Type A
```

### Command Breakdown

| Part              | Purpose                           |
| ----------------- | --------------------------------- |
| `Resolve-DnsName` | Performs a DNS lookup.            |
| `adbox.local`     | The domain name being checked.    |
| `-Type A`         | Requests the IPv4 address record. |

The output confirmed that `adbox.local` resolved to `192.168.1.50`, the static IPv4 address used by `AD-SRV01`.

![Resolve DNS Name](../screenshots/lab/10-powershell-support-administration/07-resolve-dns-name.png)

## Client Connectivity Check

Connectivity from `AD-SRV01` to both domain clients was tested with `Test-Connection`.

```powershell
Test-Connection AD-WIN10-01 -Count 2

Test-Connection AD-WIN10-02 -Count 2
```

### Command Breakdown

| Part              | Purpose                                                    |
| ----------------- | ---------------------------------------------------------- |
| `Test-Connection` | Sends network test packets to a target, similar to `ping`. |
| `AD-WIN10-01`     | First domain-joined Windows 10 client.                     |
| `AD-WIN10-02`     | Second domain-joined Windows 10 client.                    |
| `-Count 2`        | Sends two test packets to keep the output short.           |

The output confirmed that both clients responded over the network.

![Test Computer Connection](../screenshots/lab/10-powershell-support-administration/08-test-computer-connection.png)

| Client        | IPv4 Address    |
| ------------- | --------------- |
| `AD-WIN10-01` | `192.168.1.204` |
| `AD-WIN10-02` | `192.168.1.102` |

## Share Path Check

The Sales share from the file sharing stage was checked from PowerShell.

```powershell
Test-Path "\\AD-SRV01\Sales"

Get-ChildItem "\\AD-SRV01\Sales"
```

### Command Breakdown

| Part                               | Purpose                                                        |
| ---------------------------------- | -------------------------------------------------------------- |
| `Test-Path "\\AD-SRV01\Sales"`     | Checks whether the network share path exists and is reachable. |
| `Get-ChildItem "\\AD-SRV01\Sales"` | Lists the files and folders inside the share.                  |
| `\\AD-SRV01\Sales`                 | Universal Naming Convention path to the Sales share.           |

`Test-Path` returned `True`, confirming that the network share path existed. `Get-ChildItem` listed the contents of the share.

![Share Path Check](../screenshots/lab/10-powershell-support-administration/09-share-path-check.png)

The share contained the original test file and the client-created file from the file sharing stage.

| File                          | Purpose                                            |
| ----------------------------- | -------------------------------------------------- |
| `sales-access-test.txt`       | Original file created during share setup           |
| `client-created-document.txt` | File created from the client during access testing |

## User Report Export

A basic user report was exported from the ADBox lab Users OU.

```powershell
Get-ADUser -Filter * -SearchBase "OU=Users,OU=ADBox-Lab,DC=adbox,DC=local" |
Select-Object Name, SamAccountName, Enabled |
Export-Csv "C:\ADBox-Shares\adbox-users-report.csv" -NoTypeInformation
```

### Command Breakdown

| Part                                          | Purpose                                                        |
| --------------------------------------------- | -------------------------------------------------------------- |
| `Get-ADUser -Filter *`                        | Gets all user accounts from the selected search location.      |
| `-SearchBase ...`                             | Limits the export to the ADBox lab Users OU.                   |
| `Select-Object Name, SamAccountName, Enabled` | Keeps only the fields needed for the report.                   |
| `Export-Csv`                                  | Writes the selected output to a CSV file.                      |
| `C:\ADBox-Shares\adbox-users-report.csv`      | Destination path for the report file.                          |
| `-NoTypeInformation`                          | Removes PowerShell type metadata from the top of the CSV file. |

This created a CSV report containing the lab user names, logon names, and enabled status.

![Export User Report](../screenshots/lab/10-powershell-support-administration/10-export-user-report.png)

## User Report Validation

The exported CSV file was then checked from PowerShell.

```powershell
Get-ChildItem "C:\ADBox-Shares\adbox-users-report.csv"

Get-Content "C:\ADBox-Shares\adbox-users-report.csv"
```

### Command Breakdown

| Part                                                     | Purpose                                                |
| -------------------------------------------------------- | ------------------------------------------------------ |
| `Get-ChildItem "C:\ADBox-Shares\adbox-users-report.csv"` | Confirms that the CSV report file exists.              |
| `Get-Content "C:\ADBox-Shares\adbox-users-report.csv"`   | Displays the contents of the CSV report in PowerShell. |

The output confirmed that the report file existed and contained the expected ADBox lab users.

![Validate User Report](../screenshots/lab/10-powershell-support-administration/11-validate-user-report.png)

| Name         | SamAccountName | Enabled |
| ------------ | -------------- | ------- |
| Alex Morgan  | `alex.morgan`  | True    |
| Jamie Carter | `jamie.carter` | True    |
| Sam Taylor   | `sam.taylor`   | True    |

## Validation Summary

| Check                                        | Result |
| -------------------------------------------- | ------ |
| Active Directory PowerShell module imported  | Passed |
| Active Directory commands listed             | Passed |
| ADBox lab users queried                      | Passed |
| Sam Taylor account details checked           | Passed |
| Sam Taylor group membership checked          | Passed |
| Domain computer objects listed               | Passed |
| Disabled lab users checked                   | Passed |
| `adbox.local` resolved through DNS           | Passed |
| `AD-WIN10-01` responded to connectivity test | Passed |
| `AD-WIN10-02` responded to connectivity test | Passed |
| `\\AD-SRV01\Sales` path checked              | Passed |
| Share contents listed                        | Passed |
| ADBox user report exported to CSV            | Passed |
| CSV report validated from PowerShell         | Passed |

## Support Notes

PowerShell is useful in support work because it can quickly confirm account state, group membership, computer objects, DNS resolution, connectivity, and file share access.

The checks in this stage were kept practical. They show how an administrator can inspect the environment without relying only on graphical tools.

This stage also links earlier ADBox work together. The users came from the directory structure stage, Sam Taylor was used in the account recovery stage, the computers came from the domain join stage, DNS came from the domain controller setup, and the Sales share came from the file sharing stage.

The exported CSV report provides simple handover evidence that the lab user accounts existed and were enabled at the time of review.
