# Directory Structure

The directory structure stage turns a working domain into something easier to administer.

After both clients joined `adbox.local`, the default Active Directory locations were no longer enough for the rest of the lab. This stage creates a dedicated OU structure, moves the workstations, creates department users, builds security groups, and validates membership.

## Directory Target

The structure is built under a dedicated lab OU.

Area | Value
--- | ---
Domain | `adbox.local`
Top-Level Lab OU | `ADBox-Lab`
Workstations | `AD-WIN10-01`, `AD-WIN10-02`
Department Users | IT, Sales, Warehouse
Main Tool | Active Directory Users and Computers
Shortcut | `dsa.msc`

## Build Steps

The directory structure was built after both Windows 10 clients joined `adbox.local`.

Step | Action | Covers
--- | --- | ---
01 | Review Default Containers | Check the starting Active Directory locations created with the domain.
02 | Create Lab OU Structure | Add `ADBox-Lab` with separate areas for users, computers, groups, and service accounts.
03 | Move Workstations | Move `AD-WIN10-01` and `AD-WIN10-02` into the `Workstations` OU.
04 | Create Department Users | Add example IT, Sales, and Warehouse users.
05 | Create Security Groups | Add Global Security groups for department membership and RDP access.
06 | Set Group Membership | Add users to the correct groups and correct an intentional membership mistake.
07 | Validate Objects | Confirm users and groups can be found in Active Directory Users and Computers.

## Default Containers

After the domain was created, Active Directory created several default containers and OUs under `adbox.local`.

> Open: Server Manager → Tools → Active Directory Users and Computers
> Shortcut: Win + R → `dsa.msc`
> Path: `adbox.local`

![Default Containers Listed](../screenshots/lab/05-directory-structure/01-default-containers-listed.png)

Default Location | Purpose
--- | ---
`Builtin` | Stores built-in administrative and security groups.
`Computers` | Default container where newly joined computer objects are placed.
`Domain Controllers` | Default OU for Domain Controllers.
`ForeignSecurityPrincipals` | Used for identities from trusted external domains or forests.
`Managed Service Accounts` | Stores managed service account objects.
`Users` | Default container for built-in users and groups.

Containers can store objects, but normal Group Policy targeting is based around OUs. For this lab, the working users, groups, and workstations were placed under a dedicated OU structure so later stages can target them cleanly.

`AD-SRV01` was left in the default `Domain Controllers` OU. Domain Controllers are placed there automatically and receive Domain Controller-specific policies, so the server object was not moved into the lab workstation structure.

## OU Structure

A dedicated `ADBox-Lab` OU was created under `adbox.local`.

> Open: Active Directory Users and Computers
> Path: Right-click `adbox.local` → New → Organizational Unit
> Action: Create `ADBox-Lab`, then create the child OUs inside it.

![OU Structure Created](../screenshots/lab/05-directory-structure/02-ou-structure-created.png)

The final OU structure was:

```text
adbox.local
│
└── ADBox-Lab
    │
    ├── Users
    │   ├── IT
    │   ├── Sales
    │   └── Warehouse
    │
    ├── Computers
    │   ├── Workstations
    │   └── Servers
    │
    ├── Groups
    │   ├── Security
    │   └── Distribution
    └── Service-Accounts
```

Area | Purpose
--- | ---
`Users` | Stores human user accounts separated by department.
`Computers` | Stores workstation and server computer objects.
`Groups` | Stores group objects used for access control and administration.
`Service-Accounts` | Reserved for future service, automation, or scheduled task accounts.

The `Distribution` OU was included as a separate location for mail-related distribution groups. No distribution groups were created in this stage because the lab is focused on access control and workstation administration.

No service accounts were created in this stage. The OU was added so the structure has a clear place for future service or automation accounts.

## Computer Placement

When `AD-WIN10-01` and `AD-WIN10-02` joined the domain, their computer objects were created in the default `Computers` container.

> Open: Active Directory Users and Computers
> Path: `adbox.local` → Computers
> Action: Select `AD-WIN10-01` and `AD-WIN10-02` → Right-click → Move → ADBox-Lab → Computers → Workstations

The joined workstation objects were moved into:

```text
ADBox-Lab → Computers → Workstations
```

![Computers Move Dialog](../screenshots/lab/05-directory-structure/03-computers-move-dialog.png)

After the move, both Windows 10 client objects were visible inside the `Workstations` OU.

![Workstations Objects Moved](../screenshots/lab/05-directory-structure/04-workstations-objects-moved.png)

This prepares the clients for workstation-specific Group Policy settings later in the lab.

## User Accounts

Three department users were created in the `Users` OU structure.

> Open: Active Directory Users and Computers
> Path: ADBox-Lab → Users → Department OU
> Action: Right-click department OU → New → User

Department OU | User | User Principal Name
--- | --- | ---
`IT` | Alex Morgan | `alex.morgan@adbox.local`
`Sales` | Jamie Carter | `jamie.carter@adbox.local`
`Warehouse` | Sam Taylor | `sam.taylor@adbox.local`

A User Principal Name is the email-style sign-in name for a domain user. In this lab, the UPN format is `username@adbox.local`.

The example below shows the creation of the IT user account for Alex Morgan.

![User Creation Dialog](../screenshots/lab/05-directory-structure/05-user-creation-dialog.png)

During account creation, **User must change password at next logon** was selected.

![Password Policy Selected](../screenshots/lab/05-directory-structure/06-password-policy-selected.png)

This matches a normal onboarding pattern: the administrator sets a temporary password, and the user is forced to create their own password at first sign-in.

The created user object was confirmed in Active Directory Users and Computers.

![User Object Created](../screenshots/lab/05-directory-structure/07-user-object-created.png)

The ADUC Find tool was also used to confirm that all three user accounts could be located under the lab structure.

> Open: Active Directory Users and Computers
> Action: Right-click domain or OU → Find

![Users Search Results](../screenshots/lab/05-directory-structure/08-users-search-results.png)

## Group Type and Scope

Active Directory groups have a type and a scope. The type controls what the group is used for. The scope controls where the group can be used.

Option | Meaning | Lab Decision
--- | --- | ---
Security | Used to assign permissions and access to resources. | Used for access-control and administration scenarios.
Distribution | Used for email distribution lists. | Not used because mail handling is outside this stage.
Global | Used to group users from the same domain. | Used because all ADBox users are in `adbox.local`.
Domain Local | Often used to assign permissions to resources inside the domain. | Not used at this stage because resource permissions are tested later.
Universal | Used across multiple domains in a forest. | Not used because ADBox is a single-domain forest.

The lab groups were created as Global Security groups. That fits the current design because the users and resources are all inside one domain.

## Security Groups

Security groups were created inside:

```text
ADBox-Lab → Groups → Security
```

> Open: Active Directory Users and Computers
> Path: ADBox-Lab → Groups → Security
> Action: Right-click Security OU → New → Group

The group creation dialog shows `GG_IT_Users` being created as a Global Security group.

![Security Group Created](../screenshots/lab/05-directory-structure/09-security-group-created.png)

The following security groups were created:

Group | Purpose
--- | ---
`GG_IT_Users` | Groups IT department users.
`GG_Sales_Users` | Groups Sales department users.
`GG_Warehouse_Users` | Groups Warehouse department users.
`GG_RDP_Allowed` | Prepares an access-control group for later Remote Desktop testing.

![Security Groups Listed](../screenshots/lab/05-directory-structure/10-security-groups-listed.png)

The `GG_` prefix is used here to mark these as Global Groups. It keeps the group type visible in the object name during later access-control testing.

## Group Membership

Users were added to security groups to connect department users with access-control objects.

> Open: Active Directory Users and Computers
> Path: ADBox-Lab → Groups → Security → Group Properties → Members
> Action: Add the correct user account, then confirm the final membership.

Group | Final Member
--- | ---
`GG_IT_Users` | Alex Morgan
`GG_Sales_Users` | Jamie Carter
`GG_Warehouse_Users` | Sam Taylor
`GG_RDP_Allowed` | Alex Morgan

During testing, two users were added to `GG_IT_Users` at the same time to confirm that multiple objects can be selected and resolved together.

![Group Members Added](../screenshots/lab/05-directory-structure/11-group-members-added.png)

One incorrect member was then removed from `GG_IT_Users` to confirm that group membership can be corrected through ADUC.

> Action: Group Properties → Members → Select incorrect user → Remove

![Group Member Removed](../screenshots/lab/05-directory-structure/12-group-member-removed.png)

This is a normal administration task: add users to a group, identify incorrect membership, remove the wrong user, and confirm the final state.

The ADUC Find tool was also used to validate the created security groups.

![Groups Search Results](../screenshots/lab/05-directory-structure/13-groups-search-results.png)

## Result

The ADBox directory structure now separates users, computers, groups, and reserved service-account space inside a dedicated `ADBox-Lab` OU.

Directory Area | Result
--- | ---
Workstations | `AD-WIN10-01` and `AD-WIN10-02` moved into the `Workstations` OU.
Department Users | IT, Sales, and Warehouse users created under separate OUs.
Security Groups | Global Security groups created for department membership and RDP preparation.
Group Membership | Users added to their intended groups and incorrect membership corrected.
Policy Readiness | Workstations are now placed where a workstation GPO can target them.

This structure provides the foundation for applying Group Policy and testing access-control tasks in later stages.

## Navigation

Previous | Current | Next
--- | --- | ---
[04 Domain Join](04-domain-join.md) | Directory Structure | [06 Group Policy](06-group-policy.md)
