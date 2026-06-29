# Account Recovery

Account recovery tests the kind of user issue that appears constantly in support work: a user cannot get in, a password needs resetting, or access needs to be blocked without deleting the account.

This stage uses the Warehouse user `Sam Taylor` to test password reset, forced password change, account disablement, blocked sign-in, account re-enable, and restored access from a Windows 10 client.

## Recovery Target

The test account was `Sam Taylor`, a Warehouse user created earlier in the lab.

Item | Value
--- | ---
Domain | `adbox.local`
User | Sam Taylor
Account | `ADBOX\sam.taylor`
User Location | `ADBox-Lab → Users → Warehouse`
Client | `AD-WIN10-01`
Admin System | `AD-SRV01`
Admin Tool | Active Directory Users and Computers

The recovery actions were performed from `AD-SRV01`. The sign-in tests were performed from `AD-WIN10-01` to confirm the result from the workstation side.

## Recovery Steps

The account recovery test followed a simple support workflow.

Step | Action | Covers
--- | --- | ---
01 | Review Account | Confirm the user exists before making changes.
02 | Reset Password | Set a temporary password from Active Directory Users and Computers.
03 | Force Password Change | Require the user to create a new password at next sign-in.
04 | Test User Sign-In | Confirm Windows asks the user to change the temporary password.
05 | Confirm Access | Validate the signed-in user and workstation with Command Prompt.
06 | Disable Account | Block new sign-ins without deleting the account object.
07 | Test Blocked Sign-In | Confirm Windows denies access for the disabled account.
08 | Enable Account | Restore the user account in Active Directory.
09 | Confirm Restored Access | Validate that the user can sign in again.

## Account State Before Reset

The account was checked in Active Directory Users and Computers before the recovery actions were applied.

> Open: Server Manager → Tools → Active Directory Users and Computers
> Shortcut: Win + R → `dsa.msc`
> Path: ADBox-Lab → Users → Warehouse → Sam Taylor

![Account Before Reset](../screenshots/lab/09-account-recovery/01-account-before-reset.png)

This confirmed that `sam.taylor` existed in the domain and was available for the recovery test.

## Password Reset

The password was reset from Active Directory Users and Computers using the **Reset Password** action.

> Open: Active Directory Users and Computers
> Path: ADBox-Lab → Users → Warehouse
> Action: Right-click Sam Taylor → Reset Password

The reset used a temporary password and kept **User must change password at next logon** enabled.

Setting | Value
--- | ---
Temporary Password | Set by administrator
User Must Change Password At Next Logon | Enabled
Unlock User Account | Not selected
Account Lockout Status | Account was not locked

![Password Reset Dialog](../screenshots/lab/09-account-recovery/02-password-reset-dialog.png)

A password reset lets the administrator set a temporary password. Forcing a password change at next sign-in means the user cannot keep using that temporary password as their normal password.

## Password Change Required

On `AD-WIN10-01`, the user attempted to sign in as:

> Open: Windows sign-in screen on `AD-WIN10-01`
> Action: Other user → sign in with the temporary password

```text
ADBOX\sam.taylor
```

Windows required the password to be changed before the user could finish signing in.

![Password Change Required](../screenshots/lab/09-account-recovery/03-password-change-required.png)

This confirmed that the password reset and forced password change setting were being enforced from Active Directory.

## Client Password Update

The password was changed from the Windows sign-in screen.

> Open: Password change prompt on `AD-WIN10-01`
> Action: Enter the temporary password, then set a new password.

![Client Password Changed](../screenshots/lab/09-account-recovery/04-client-password-changed.png)

This completed the user-side part of the reset workflow: the temporary password was accepted, then replaced before desktop access was allowed.

## Post-Change Sign-In

After the password change, Sam Taylor signed in successfully on `AD-WIN10-01`.

The session was checked with Command Prompt.

> Open: Win + R → `cmd`
> Run on: `AD-WIN10-01` after Sam Taylor signs in

```cmd
whoami
hostname
```

Expected result:

```text
adbox\sam.taylor
AD-WIN10-01
```

![Post Change Login](../screenshots/lab/09-account-recovery/05-post-change-login.png)

This confirmed that the recovered user account could authenticate to the domain from a Windows 10 client.

## Account Disabled

The account was then disabled in Active Directory Users and Computers.

> Open: Active Directory Users and Computers
> Path: ADBox-Lab → Users → Warehouse
> Action: Right-click Sam Taylor → Disable Account

![Account Disabled](../screenshots/lab/09-account-recovery/06-client-account-disabled.png)

Disabling an account blocks new sign-ins while keeping the account object, group membership, and administrative history in Active Directory. This is useful when access needs to be removed without deleting the user.

## Disabled Account Test

After the account was disabled, sign-in was tested again from `AD-WIN10-01`.

> Open: Windows sign-in screen on `AD-WIN10-01`
> Action: Try to sign in as `ADBOX\sam.taylor`

Windows blocked the login and showed that the account had been disabled.

![Disabled Account Notification](../screenshots/lab/09-account-recovery/07-account-disabled-notification.png)

This proved that the disabled account state was enforced during domain sign-in.

## Account Enabled Again

The account was enabled again in Active Directory Users and Computers.

> Open: Active Directory Users and Computers
> Path: ADBox-Lab → Users → Warehouse
> Action: Right-click Sam Taylor → Enable Account

![Account Enabled](../screenshots/lab/09-account-recovery/08-client-account-enabled.png)

This restored the account for normal authentication.

## Access Restored

After the account was enabled again, Sam Taylor signed in successfully on `AD-WIN10-01`.

The restored session was checked with Command Prompt.

> Open: Win + R → `cmd`
> Run on: `AD-WIN10-01` after Sam Taylor signs in again

```cmd
whoami
hostname
```

Expected result:

```text
adbox\sam.taylor
AD-WIN10-01
```

![Access Restored](../screenshots/lab/09-account-recovery/09-client-access-restored.png)

This confirmed that the account was usable again after being re-enabled.

## Validation Summary

Check | Result
--- | ---
Sam Taylor account reviewed before reset | Passed
Password reset completed in ADUC | Passed
Forced password change enabled | Passed
Client required password change before sign-in | Passed
User completed password change | Passed
Sign-in confirmed as `ADBOX\sam.taylor` | Passed
Account disabled in ADUC | Passed
Disabled account blocked from signing in | Passed
Account enabled again in ADUC | Passed
Sign-in restored as `ADBOX\sam.taylor` | Passed

## Result

The account recovery workflow was completed successfully.

Outcome | Meaning
--- | ---
Password Reset | The user could be given a temporary password.
Forced Password Change | The user had to replace the temporary password before desktop access.
Account Disable | New sign-ins were blocked without deleting the account.
Account Enable | The account was restored for normal authentication.
Client Validation | The result was confirmed from a domain-joined Windows 10 workstation.

This stage confirms that basic account recovery actions can be performed from Active Directory and validated from the user side.

## Navigation

Previous | Current | Next
--- | --- | ---
[08 File Sharing](08-file-sharing.md) | Account Recovery | [10 PowerShell Administration](10-powershell-administration.md)
