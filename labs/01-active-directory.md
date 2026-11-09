# Lab 01: Active Directory on Windows Server 2022

## Objective

Build a small Windows domain in VMware with Windows Server 2022, Active Directory Domain Services, DNS, organizational units, users, security groups, a password policy, and an account lockout exercise.

This lab supports help desk work because many requests begin with identity. A technician may need to confirm whether a user exists, whether the account is enabled, which group gives access, whether a password expired, or whether repeated failed sign ins locked the account.

By the end of the guide, a standard test user should be able to sign in to a domain joined Windows client. An administrator should be able to find the user, reset the password, unlock the account, and explain which policy controls each action.

## Concepts explained

### Domain and domain controller

A domain is a security and administration boundary managed by domain controllers. A domain controller hosts Active Directory, stores directory information, and answers authentication requests for users and computers.

### Organizational units, users, and groups

An organizational unit, usually called an OU, organizes directory objects and supports Group Policy targeting. An OU is a management container, not a security boundary. A simple OU design is easier to support than a deep design with no clear owner or policy purpose.

A user account represents a person or service identity. A security group collects accounts or computers so permission can be assigned to the group. Group based access makes access reviews and removal cleaner than many individual assignments.

### Group Policy, password policy, and lockout policy

A Group Policy Object, usually called a GPO, contains settings for users or computers. It can be linked to a domain, site, or OU. Link location, inheritance, security filtering, and precedence determine the final result.

Password policy controls length, history, age, and complexity. Account lockout policy controls what happens after repeated failed sign in attempts. Domain password and lockout settings belong in a GPO linked at the domain level. Fine grained password policies can apply stricter rules to selected users or global security groups.

### DNS

Active Directory depends on DNS. Domain joined computers use DNS records to find domain controllers. If a client points to the wrong DNS server, a name resolution problem can look like a password problem.

## What you need

* VMware installed on the lab host.
* Windows Server 2022 and Windows client installation images.
* One virtual machine for the server and one for the client.
* A private VMware network for the lab LAN.
* A lab domain name, such as corp.local.
* A written plan for server name, domain name, IP address, subnet mask, gateway, and DNS.
* A password manager entry for lab administrator credentials.
* Clean snapshots before major changes.

## Build steps

### Part A: Install and prepare Windows Server 2022

1. Create a virtual machine and attach it to the private lab network.
2. Install Windows Server 2022 with Desktop Experience.
3. Sign in with the local administrator account created during setup.
4. Rename the server to a clear name, such as DC01, and restart if prompted.
5. Assign a static IPv4 address, subnet mask, and gateway from the written plan, and set the server to use itself for DNS once the DNS role is installed.
6. Confirm the date, time, and time zone.
7. Install available Windows updates and restart when required.

### Part B: Install Active Directory Domain Services and DNS

1. Open Server Manager and select Manage, then Add Roles and Features.
2. Select Active Directory Domain Services, accept the required management features, and select DNS Server if it is not already selected.
3. Complete the installation.
4. Select the Server Manager notification flag and choose Promote this server to a domain controller.
5. Select Add a new forest and enter the chosen root domain name.
6. Set the functional levels offered for Windows Server 2022.
7. Confirm that the server will provide DNS and a global catalog.
8. Enter a Directory Services Restore Mode password and store it in the password manager.
9. Review the NetBIOS name and database paths.
10. Complete the prerequisite checks and install the promotion.
11. Allow the server to restart and sign in with the domain administrator account.

[Screenshot to add: Server Manager showing Active Directory Domain Services and DNS installed]

### Part C: Create the OU structure

1. Open Active Directory Users and Computers.
2. Right click the domain, select New, then select Organizational Unit.
3. Create OUs named People, Groups, Workstations, Servers, Service Accounts, and Disabled Accounts, then create IT, Operations, and Support Practice inside People.
4. Protect each new OU from accidental deletion if the option is offered.
5. Review the tree and confirm that each OU has a clear purpose.

[Screenshot to add: Active Directory Users and Computers showing the OU structure]

### Part D: Create users and groups

1. In People and IT, create an administrator practice user.
2. In People and Support Practice, create at least three standard test users.
3. Complete the name, display name, user logon name, and user principal name for each user.
4. Set a temporary password and require a change at next sign on for standard test users.
5. In Groups, create global security groups named Help Desk, Lab Administrators Practice, and Standard Users Practice, then add users to the groups that match their practice roles.
6. Open one user and use the Member Of tab to confirm the assignments.
7. Keep routine practice accounts separate from administrator practice accounts.

[Screenshot to add: A test user account showing the Member Of tab]

### Part E: Create the password policy GPO

1. Open Group Policy Management and expand the forest and domain.
2. Right click the domain and select Create a GPO in this domain, and Link it here.
3. Name the GPO Domain Account Policy Practice and edit it.
4. Open Computer Configuration, Policies, Windows Settings, Security Settings, Account Policies, Password Policy.
5. Set Enforce password history to 5 passwords remembered, Maximum password age to 180 days, Minimum password age to 1 day, and Minimum password length to 12 characters.
6. Enable the setting that requires passwords to meet complexity requirements.
7. Open Account Lockout Policy.
8. Set Account lockout threshold to 5 invalid logon attempts, Account lockout duration to 15 minutes, and Reset account lockout counter after to 15 minutes.
9. Close the editor and confirm the GPO is linked to the domain and enabled.
10. On the client, run `gpupdate /force` if immediate testing is needed.
11. Run `gpresult /r` and confirm that the policy reaches the computer.

The values in this section are lab practice values. An employer environment should follow its security standard and any applicable regulatory requirement.

[Screenshot to add: Group Policy Management showing the account policy GPO linked to the domain]

### Part F: Run the account lockout exercise

1. Choose one standard test user and confirm that the account is enabled and unlocked.
2. At the domain joined client, enter the correct user name and an incorrect password until the threshold is reached.
3. Record the exact message shown by Windows.
4. Open the user in Active Directory Users and Computers and select the Account tab.
5. Confirm that Windows reports the account as locked.
6. Review failed logon events in the domain controller Security log if you want evidence practice.
7. Select Unlock account and apply the change.
8. Reset the password if the scenario requires it, require a change at next logon, and test sign in with the correct or new password.
9. Record the fault, evidence, fix, and lesson in the session log.

[Screenshot to add: A test user Account tab showing the unlock control]

## Reference PowerShell for bulk user creation

Review every value before running this reference script in the lab. The CSV should contain GivenName, Surname, SamAccountName, UserPrincipalName, Department, OUPath, and TemporaryPassword.

```powershell
$Users = Import-Csv "C:\LabFiles\users.csv"
foreach ($User in $Users) {
    $DisplayName = "$($User.GivenName) $($User.Surname)"
    $SecurePassword = ConvertTo-SecureString $User.TemporaryPassword -AsPlainText -Force
    New-ADUser `
        -Name $DisplayName `
        -GivenName $User.GivenName `
        -Surname $User.Surname `
        -SamAccountName $User.SamAccountName `
        -UserPrincipalName $User.UserPrincipalName `
        -Department $User.Department `
        -Path $User.OUPath `
        -AccountPassword $SecurePassword `
        -Enabled $true `
        -ChangePasswordAtLogon $true
}
```

After an import, use Active Directory Users and Computers and `Get-ADUser` to confirm names, enabled state, and OU placement. Do not store real passwords in a CSV file outside a controlled lab exercise.

## Verification checks

| Checkpoint | Action | Expected evidence |
| --- | --- | --- |
| Domain services | Open Server Manager | Active Directory Domain Services and DNS show as installed |
| DNS | Run `nslookup` for the domain from the client | The domain controller answers the query |
| OU design | Open Active Directory Users and Computers | The planned OUs appear in the correct places |
| User access | Sign in to the client as a test user | The user reaches the Windows desktop |
| Group design | Open a user and select Member Of | The intended practice groups are listed |
| Policy application | Run `gpresult /r` on the client | The domain account policy appears in the results |
| Lockout recovery | Unlock and reset the test account | The user can sign in again |

## Troubleshooting

**The client cannot join the domain.** Start with DNS. Confirm that the client uses the domain controller as its DNS server, can reach the server by IP address and name, and uses the correct domain name. Check that the client and server clocks are close enough for authentication.

**Sign in fails with the correct user name.** Open the account and check whether it is disabled, expired, or locked. Confirm that the user selected the domain rather than the local computer. Reset the password only after confirming the account identity and state.

**Group Policy does not apply.** Confirm the GPO link, the object location, and the scope of management. Run `gpupdate /force`, sign out or restart when required, and review `gpresult /r`. Check for a conflicting GPO with higher precedence.

**The password policy does not change current passwords.** Password policy affects new passwords and password changes. It does not rewrite existing passwords. Confirm that the GPO is linked at the domain level and check whether a fine grained password policy applies to the user or a group containing the user.

**The account will not unlock.** Confirm that you are connected to the correct domain and domain controller. Refresh Active Directory Users and Computers and verify that the displayed account is the account used at the client. In a multi controller lab, allow replication time or connect to the controller that recorded the lockout.

## Security+ tie in

This lab mainly supports Domain 4, Security Operations, through identity and access management, account management, authentication, and operational evidence.

It also supports Domain 1, General Security Concepts, through password controls, least privilege, account states, and administrative separation. It supports Domain 5, Security Program Management and Oversight, because account policy turns a governance standard into technical settings that a technician can verify.

Useful exam language includes authentication, authorization, accounting, identity store, security group, policy enforcement, account disablement, password reset, and account lockout.

## Portfolio evidence checklist

* [Screenshot to add: Server Manager dashboard after role installation]
* [Screenshot to add: Group membership for a test user]
* [Screenshot to add: Password policy settings in the Group Policy editor]
* [Screenshot to add: Applied policy result from the client]

## My session log

| Date | What I did | What broke | How I fixed it | What I learned |
| --- | --- | --- | --- | --- |
|  |  |  |  |  |
|  |  |  |  |  |
|  |  |  |  |  |
|  |  |  |  |  |
|  |  |  |  |  |
