# Group Policy

Group Policy is the first central management test in ADBox: configure something once on the server, then prove the workstation receives it.

The visible result is a logon notice on `AD-WIN10-01`. The notice is simple on purpose. It proves that the workstation object is in the right Organisational Unit (OU), the Group Policy Object (GPO) is linked correctly, policy refresh works, and the result can be checked from the client.

## Policy Target

The policy was targeted at the workstation OU created during the directory structure stage.

```text
ADBox-Lab -> Computers -> Workstations
```

This OU contains the joined Windows 10 clients:

```text
AD-WIN10-01
AD-WIN10-02
```

Linking the GPO to the `Workstations` OU means the setting applies to the computer objects inside that OU.

Group Policy targeting depends on where the GPO is linked and where the user or computer object is located. Since this test uses a computer-side setting, the important object is the workstation computer account, not the signed-in user account.

## GPO Purpose

A workstation logon notice was used as the first Group Policy test because it gives a clear client-side result.

| Policy Area    | Lab Choice                               |
| -------------- | ---------------------------------------- |
| GPO Name       | `GPO_Workstations_Logon_Notice`          |
| Target OU      | `ADBox-Lab -> Computers -> Workstations` |
| Policy Side    | Computer Configuration                   |
| Visible Result | Logon notice shown before sign-in        |
| Test Client    | `AD-WIN10-01`                            |

The goal was to prove that a setting configured on the Domain Controller could reach a domain-joined workstation.

## GPO Creation and Link

The GPO was created and linked to the Workstations OU.

### Work Path

```text
Server Manager -> Tools -> Group Policy Management -> Forest -> Domains -> adbox.local -> ADBox-Lab -> Computers -> Workstations
```

### Shortcut

```text
Win + R -> gpmc.msc
```

### Action

```text
Context menu on Workstations -> Create a GPO in this domain, and Link it here
```

The GPO was linked to the OU that contains the Windows 10 computer objects. This keeps the test focused on workstations instead of applying the setting to the whole domain.

## Policy Location

The logon notice setting sits under Security Options in Group Policy Management Editor.

### Work Path

```text
Group Policy Management -> GPO_Workstations_Logon_Notice -> Edit -> Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Local Policies -> Security Options
```

The configured settings were:

```text
Interactive logon: Message title for users attempting to log on
Interactive logon: Message text for users attempting to log on
```

The full policy path is:

```text
Computer Configuration
-> Policies
-> Windows Settings
-> Security Settings
-> Local Policies
-> Security Options
```

Computer Configuration settings apply to the machine. User Configuration settings apply to the signed-in user. This distinction matters during troubleshooting because a policy can fail to appear if the setting is configured on the wrong side or linked to the wrong OU.

## Logon Notice Configuration

The GPO was configured with a short workstation logon notice.

| Setting       | Value                          |
| ------------- | ------------------------------ |
| Message Title | `ADBox Lab Workstation`        |
| Message Text  | `Authorised ADBox users only.` |

Figure 6.1 shows the configured logon notice settings inside the GPO.

![Figure 6.1 - Logon policy configured](../screenshots/lab/06-group-policy/01-logon-policy-configured.png)

*Figure 6.1 - Group Policy Management Editor showing the configured logon notice title and message text for workstation sign-in.*

This confirms that the logon notice title and message text were set inside the workstation GPO.

## Client Policy Update

On `AD-WIN10-01`, Group Policy was refreshed manually from Command Prompt.

### Work Path

```text
Win + R -> cmd
```

### Run On

```text
AD-WIN10-01
```

```cmd
gpupdate /force
```

Figure 6.2 shows the successful policy refresh on the client.

![Figure 6.2 - Client GPUpdate success](../screenshots/lab/06-group-policy/02-client-gpupdate-success.png)

*Figure 6.2 - Command Prompt output on `AD-WIN10-01` showing `gpupdate /force` completed successfully for Computer Policy and User Policy.*

`gpupdate /force` tells the client to refresh policy immediately. This is useful in a lab because it avoids waiting for the normal background refresh interval before checking whether a new GPO applies.

## GPResult Summary

After the policy refresh, `gpresult /r` was used to review the policy result from the client side.

### Work Path

```text
Win + R -> cmd
```

### Run On

```text
AD-WIN10-01
```

```cmd
gpresult /r
```

Figure 6.3 shows the summary section of the client-side Group Policy result.

![Figure 6.3 - GPResult client summary](../screenshots/lab/06-group-policy/03-gpresult-client-summary.png)

*Figure 6.3 - `gpresult /r` output showing the user context, client computer, domain, and policy source for `AD-WIN10-01`.*

The summary confirmed the client and domain context.

| Check            | Result                 |
| ---------------- | ---------------------- |
| User Context     | `ADBOX\Administrator`  |
| Client Computer  | `AD-WIN10-01`          |
| Operating System | Windows 10             |
| Domain           | `ADBOX`                |
| Policy Source    | `AD-SRV01.adbox.local` |

This confirmed that the policy result was being generated on the domain-joined workstation and sourced from the ADBox Domain Controller.

## Computer Policy Validation

The computer policy section of `gpresult /r` showed that the workstation logon notice GPO was applied to `AD-WIN10-01`.

Figure 6.4 is the main validation screenshot for this stage because the logon notice is a Computer Configuration setting.

![Figure 6.4 - GPResult computer policy applied](../screenshots/lab/06-group-policy/04-gpresult-computer-applied.png)

*Figure 6.4 - `gpresult /r` computer settings showing `GPO_Workstations_Logon_Notice` applied to `AD-WIN10-01`.*

The result also confirmed that `AD-WIN10-01` was located under the expected workstation OU.

```text
ADBox-Lab -> Computers -> Workstations
```

This links the result back to the directory structure stage: the workstation object was moved into the OU that the GPO targets.

## User Policy Check

The user settings section of `gpresult /r` was also checked.

Figure 6.5 shows the user-side policy result.

![Figure 6.5 - GPResult user settings](../screenshots/lab/06-group-policy/05-gpresult-user-settings.png)

*Figure 6.5 - `gpresult /r` user settings showing the user-side policy result for the signed-in account.*

No user-side GPO was needed for this test because the logon notice was configured as a computer-side policy.

This check is useful because Group Policy results need to be read in two parts: Computer Settings and User Settings. A missing user-side policy does not mean the computer-side policy failed.

## Workstation Result

After the policy refresh and validation checks, the configured logon notice appeared on the Windows 10 client.

### Work Path

```text
AD-WIN10-01 -> Restart or sign out -> Windows sign-in flow
```

### Check

```text
Confirm that the configured logon notice appears before sign-in.
```

Figure 6.6 shows the visible logon notice on the workstation.

![Figure 6.6 - Logon message shown](../screenshots/lab/06-group-policy/06-logon-message-shown.png)

*Figure 6.6 - Windows 10 client showing the configured `ADBox Lab Workstation` logon notice before sign-in.*

This confirmed that the GPO reached the workstation and produced the expected visible result.

## Result

The workstation GPO was created, linked, applied, and validated from the client side.

| Check                             | Outcome |
| --------------------------------- | ------- |
| GPO linked to Workstations OU     | Passed  |
| Logon notice configured           | Passed  |
| Client policy refresh completed   | Passed  |
| `gpresult /r` generated on client | Passed  |
| Computer-side GPO applied         | Passed  |
| User-side GPO check reviewed      | Passed  |
| Visible logon notice displayed    | Passed  |

This stage proves that `AD-SRV01` can centrally apply a workstation configuration to a domain-joined Windows 10 client through Group Policy.

## Navigation

| Previous                                            | Current         | Next                                      |
| --------------------------------------------------- | --------------- | ----------------------------------------- |
| [05 Directory Structure](05-directory-structure.md) | 06 Group Policy | [07 Remote Desktop](07-remote-desktop.md) |
