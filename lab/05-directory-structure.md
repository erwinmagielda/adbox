# Directory Structure

The joined clients, users, and groups are organised inside a dedicated `ADBox-Lab` Organisational Unit (OU).

This stage moves the lab away from the default Active Directory containers and into a structure that can support real administration: workstation placement, department users, security groups, group membership, Group Policy (GP), Remote Desktop Protocol (RDP), printer mapping, account recovery, and PowerShell administration.

The main point of this stage is to separate **where objects are stored** from **how access is controlled**. OUs organise objects and support policy targeting. Groups are used for permissions, access control, and membership-based administration.

## Default Containers

After the domain was created, Active Directory created several default containers and OUs under `adbox.local`.

![Default Containers Listed](/screenshots/lab/05-directory-structure/01-default-containers-listed.png)

| Default Location            | Purpose                                                           |
|-----------------------------|-------------------------------------------------------------------|
| `Builtin`                   | Stores built-in administrative and security groups.               |
| `Computers`                 | Default container where newly joined computer objects are placed. |
| `Domain Controllers`        | Default OU for Domain Controllers.                                |
| `ForeignSecurityPrincipals` | Used for identities from trusted external domains or forests.     |
| `Managed Service Accounts`  | Stores managed service account objects.                           |
| `Users`                     | Default container for built-in users and groups.                  |

The default `Computers` and `Users` locations are containers, not the main working structure for this lab. A separate `ADBox-Lab` OU was created so lab users, computers, and groups can be organised clearly and targeted later through policies and administration tasks.

`AD-SRV01` was not moved from the default `Domain Controllers` OU. Domain Controllers are placed there automatically and receive Domain Controller-specific policies, so the server object was left in its default administrative location.

## OU Structure

A dedicated `ADBox-Lab` OU was created under `adbox.local`.

![OU Structure Created](/screenshots/lab/05-directory-structure/02-ou-structure-created.png)

The final OU structure was:

```text
adbox.local
└── ADBox-Lab
    ├── Users
    │   ├── IT
    │   ├── Sales
    │   └── Warehouse
    ├── Computers
    │   ├── Workstations
    │   └── Servers
    ├── Groups
    │   ├── Security
    │   └── Distribution
    └── Service-Accounts
```

The `Users` OU stores human user accounts, separated by department. The `Computers` OU stores workstation and server objects. The `Groups` OU stores group objects used for access control and later administration scenarios.

The `Distribution` OU was created to show where mail-related distribution groups could be separated in a larger environment. No distribution groups were created at this stage because ADBox is focused on access control, workstation administration, GP, RDP, and support workflows.

The `Service-Accounts` OU was created as a reserved location for future service or automation accounts, such as accounts used by scripts, scheduled tasks, monitoring tools, or managed services. No service accounts were created at this stage.

## Computer Placement

When `AD-WIN10-01` and `AD-WIN10-02` joined the domain, their computer objects were created in the default `Computers` container.

The joined workstation objects were moved into:

```text
ADBox-Lab → Computers → Workstations
```

![Computers Move Dialog](/screenshots/lab/05-directory-structure/03-computers-move-dialog.png)

After the move, both Windows 10 client objects were visible inside the `Workstations` OU.

![Workstations Objects Moved](/screenshots/lab/05-directory-structure/04-workstations-objects-moved.png)

This prepares the clients for workstation-specific GP settings later in the lab. The Domain Controller object was left in the default `Domain Controllers` OU because it should receive Domain Controller-specific policies, not workstation policies.

## User Accounts

Three department users were created in the `Users` OU structure.

| Department OU | User         | User Principal Name (UPN)  |
|---------------|--------------|----------------------------|
| `IT`          | Alex Morgan  | `alex.morgan@adbox.local`  |
| `Sales`       | Jamie Carter | `jamie.carter@adbox.local` |
| `Warehouse`   | Sam Taylor   | `sam.taylor@adbox.local`   |

The example below shows the creation of the IT user account for Alex Morgan.

![User Creation Dialog](/screenshots/lab/05-directory-structure/05-user-creation-dialog.png)

During account creation, the password option **User must change password at next logon** was selected.

![Password Policy Selected](/screenshots/lab/05-directory-structure/06-password-policy-selected.png)

This supports a normal onboarding pattern: the administrator creates the account with a temporary password, and the user is forced to set their own password at first sign-in.

The created user object was confirmed in Active Directory Users and Computers (ADUC).

![User Object Created](/screenshots/lab/05-directory-structure/07-user-object-created.png)

The ADUC Find tool was also used to confirm that all three user accounts could be located under the lab structure.

![Users Search Results](/screenshots/lab/05-directory-structure/08-users-search-results.png)

## Group Type And Scope

Active Directory groups have a type and a scope. The type controls what the group is used for, while the scope controls where the group can be used.

| Option | Meaning | Lab Decision |
|---|---|---|
| Security | Used to assign permissions and access to resources. | Used because the lab groups support access-control and administration scenarios. |
| Distribution | Used for email distribution lists, not normal access control. | Not used at this stage because mail handling is outside the current ADBox scope. |
| Global | Used to group users from the same domain. | Used because all ADBox users are in the single `adbox.local` domain. |
| Domain Local | Often used to assign permissions to resources inside the domain. | Not used at this stage because resource permission mapping will come later. |
| Universal | Used across multiple domains in a forest. | Not used because ADBox is a single-domain forest. |

The lab groups were created as **Global Security** groups. This fits the current design because the users and resources are all inside `adbox.local`, and the groups are intended for administration and access-control scenarios.

## Security Groups

Security groups were created inside:

```text
ADBox-Lab → Groups → Security
```

The group creation dialog shows `GG_IT_Users` being created as a Global Security group.

![Security Group Created](/screenshots/lab/05-directory-structure/09-security-group-created.png)

The following security groups were created:

| Group                | Purpose                                                 |
|----------------------|---------------------------------------------------------|
| `GG_IT_Users`        | Groups IT department users.                             |
| `GG_Sales_Users`     | Groups Sales department users.                          |
| `GG_Warehouse_Users` | Groups Warehouse department users.                      |
| `GG_RDP_Allowed`     | Prepares an access-control group for later RDP testing. |

![Security Groups Listed](/screenshots/lab/05-directory-structure/10-security-groups-listed.png)

The `GG_` prefix identifies these as Global Groups and keeps the group purpose readable.

## Group Membership

Users were added to security groups to connect department users with access-control objects.

| Group                | Final Member |
|----------------------|--------------|
| `GG_IT_Users`        | Alex Morgan  |
| `GG_Sales_Users`     | Jamie Carter |
| `GG_Warehouse_Users` | Sam Taylor   |
| `GG_RDP_Allowed`     | Alex Morgan  |

During testing, two users were added to `GG_IT_Users` at the same time to confirm that multiple objects can be selected and resolved together.

![Group Members Added](/screenshots/lab/05-directory-structure/11-group-members-added.png)

One incorrect member was then removed from `GG_IT_Users` to confirm that group membership can be corrected through ADUC.

![Group Member Removed](/screenshots/lab/05-directory-structure/12-group-member-removed.png)

This demonstrates a common administration task: add users to a group, identify incorrect membership, remove the incorrect user, and confirm the final state.

The ADUC Find tool was also used to validate the created security groups.

![Groups Search Results](/screenshots/lab/05-directory-structure/13-groups-search-results.png)

## Summary

The ADBox directory now separates users, computers, groups, and future service accounts inside a dedicated `ADBox-Lab` OU.

The Windows 10 clients were moved from the default `Computers` container into the `Workstations` OU. Department users were created under separate department OUs, and Global Security groups were created to support later access-control scenarios.

This structure provides the foundation for applying GP to users and computers in a controlled way.

## Next Stage

[Stage 6: Group Policy](06-group-policy.md), which applies and validates policy settings against the organised users, computers, and OUs.
