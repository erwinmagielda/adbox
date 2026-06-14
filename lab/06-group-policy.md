# Group Policy

Group Policy is used to centrally apply Windows settings to domain-joined computers and users.

After the clients were joined to `adbox.local` and moved into the `Workstations` Organisational Unit (OU), a Group Policy Object (GPO) was created to test centralised workstation configuration. The policy applies a logon notice to the Windows 10 client, proving that `AD-SRV01` can push configuration to a domain workstation through Active Directory.

The logon notice itself is a simple visible test. The main purpose of this stage is to prove the full policy chain:

```text
Workstation OU → Linked GPO → Configured Computer Policy → Client Policy Update → GP Result Validation → Visible Workstation Result
```

## Policy Target

The policy was created for the workstation computers organised in the previous lab stage.

```text
ADBox-Lab → Computers → Workstations
```

This OU contains the joined Windows 10 clients:

```text
AD-WIN10-01
AD-WIN10-02
```

Linking the GPO to the `Workstations` OU means the setting applies to the computer objects inside that OU, rather than the whole domain.

## Policy Location

Group Policy settings are split by where the setting applies and what part of Windows it controls.

| Area                     | Purpose                                                                                                  |
| ------------------------ | -------------------------------------------------------------------------------------------------------- |
| Computer Configuration   | Applies machine-wide settings to the computer.                                                           |
| User Configuration       | Applies settings to the signed-in user.                                                                  |
| Security Settings        | Controls logon rules, security options, audit settings, and rights.                                      |
| Administrative Templates | Controls Windows behaviour, interface restrictions, and registry-based settings.                         |
| Preferences              | Handles practical support items such as mapped drives, printers, files, shortcuts, and registry changes. |

The logon notice is configured under **Security Options** because Windows treats the sign-in screen as a security-related area, not a normal desktop preference.

The policy path used was:

```text
Computer Configuration → Policies → Windows Settings → Security Settings → Local Policies → Security Options
```

The configured settings were:

```text
Interactive logon: Message title for users attempting to log on
```

```text
Interactive logon: Message text for users attempting to log on
```

## Logon Policy Configuration

The GPO was configured with a workstation logon notice.

| Setting       | Value                          |
| ------------- | ------------------------------ |
| Message Title | `ADBox Lab Workstation`        |
| Message Text  | `Authorised ADBox users only.` |

![Logon Policy Configured](/screenshots/lab/06-group-policy/01-logon-policy-configured.png)

This confirms that the logon notice title and text were defined in the GPO.

## Client Policy Update

On `AD-WIN10-01`, Group Policy was updated manually from Command Prompt (CMD):

```text
gpupdate /force
```

![Client GPUpdate Success](/screenshots/lab/06-group-policy/02-client-gpupdate-success.png)

The command completed successfully for both Computer Policy and User Policy.

## GPResult Summary

The `gpresult /r` command was then used to check which policies applied to the client.

```text
gpresult /r
```

![GPResult Client Summary](/screenshots/lab/06-group-policy/03-gpresult-client-summary.png)

The summary confirmed that the result was generated for:

| Item             | Value                  |
| ---------------- | ---------------------- |
| User context     | `ADBOX\Administrator`  |
| Client computer  | `AD-WIN10-01`          |
| Operating system | Windows 10             |
| Domain           | `ADBOX`                |
| Policy source    | `AD-SRV01.adbox.local` |

## Computer Policy Validation

The applied computer policies confirmed that `GPO_Workstations_Logon_Notice` was applied to `AD-WIN10-01`.

![GPResult Computer Applied](/screenshots/lab/06-group-policy/04-gpresult-computer-applied.png)

This is the main validation point for the lab because the logon notice is a **Computer Configuration** policy. It applies to the workstation before any user signs in.

The result also confirms that `AD-WIN10-01` is located under:

```text
ADBox-Lab > Computers > Workstations
```

This proves that the OU placement from the previous lab stage was used as the policy target.

## User Policy Check

The user settings section of `gpresult /r` was also checked.

![GPResult User Settings](/screenshots/lab/06-group-policy/05-gpresult-user-settings.png)

No user-side GPO was applied for this test. This is expected because the logon notice was configured as a computer-side policy, not a user-side policy.

This screenshot is useful because it shows the difference between **Computer Settings** and **User Settings** in Group Policy troubleshooting.

## Workstation Result

After the policy update and validation, the workstation displayed the configured logon notice.

![Logon Message Shown](/screenshots/lab/06-group-policy/06-logon-message-shown.png)

This confirms that the GPO setting reached the Windows 10 client and produced the expected visible result.

## Summary

A workstation GPO was created, configured, applied, and validated.

The lab confirmed that `AD-SRV01` can centrally manage a domain-joined Windows 10 client through Group Policy. The `Workstations` OU was used as the policy target, `gpupdate /force` refreshed the client policy, `gpresult /r` confirmed the applied computer-side GPO, and the logon screen showed the configured notice.

This completes the first Group Policy test in ADBox and provides the foundation for later policy-based support tasks such as Remote Desktop access, mapped drives, printer deployment, desktop restrictions, and workstation configuration.

## Next Stage

[Stage 7: Remote Desktop Access](07-remote-desktop.md), which tests domain-based Remote Desktop access using security groups and workstation validation.
