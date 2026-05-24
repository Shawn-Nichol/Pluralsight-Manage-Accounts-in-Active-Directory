# 📌 Manage Accounts in Active Directory

---

## 📘 Overview

This lab covers end-to-end Active Directory account management within the `globomantics.co` domain using a live Windows Server domain controller. Across three objectives, the lab simulates real-world identity administration tasks: creating Organizational Units, user accounts, and security groups via the GUI; automating user provisioning and reporting with PowerShell 7; and delegating domain join permissions to a non-admin group, which is then used to join a file server to the domain. These tasks directly reflect the responsibilities of a systems administrator or identity-focused SOC analyst in an enterprise Windows environment.


![Platform](https://img.shields.io/badge/platform-Pluralsight-39ff7a?style=flat-square)
![Status](https://img.shields.io/badge/status-complete-39ff7a?style=flat-square)
![Date](https://img.shields.io/badge/date-2026-05-23-blue?style=flat-square)
---

## 🎯 Objectives

- Create an Organizational Unit (OU) and security groups within an existing AD domain using ADUC
- Create and move user accounts between containers and OUs
- Manage group membership by adding users to security groups via the GUI
- Use PowerShell to enumerate domain users and automate account creation
- Delegate domain join permissions to a security group using the Delegation of Control Wizard
- Remove a server from the domain (unjoin) and rejoin it using a non-Domain Admin account

---

## 🧰 Tools & Technologies

| Tool | Purpose |
|------|---------|
| Windows Server 2022 (Domain Controller) | Hosts Active Directory Domain Services for the domain |
| Active Directory Users and Computers (ADUC) | GUI-based OU, user, and group management |
| PowerShell 7 (Run as Administrator) | Scripted user enumeration, account creation, and group membership management |
| ActiveDirectory PowerShell Module | Provides AD cmdlets: |
| Server Manager | Local Server configuration and domain join/unjoin on the File Server |
| Delegation of Control Wizard | Assigns scoped AD permissions to the group |

---

## 🧪 Walkthrough

### Objective 1 — Manage Windows Users Using GUI Tools

#### Step 1 — Open Active Directory Users and Computers

From the Domain Controller desktop, navigate to:

```
Start → Windows Administrative Tools → Active Directory Users and Computers
```

---

#### Step 2 — Create an Organizational Unit
In the left-hand pane, right-click the `globomantics.co` domain and select **New → Organizational Unit**.

---

#### Step 3 — Create Security Groups

Right-click the **Globomantics** OU and select **New → Group**. Create two groups:

| Group Name | Group Scope | Group Type |
|------------|-------------|------------|
| `NetAdmins` | Global | Security |
| `AllUsers` | Global | Security |

---

#### Step 4 — Create a New User Account

Right-click the **Globomantics** OU and select **New → User**. Complete the wizard with the following values:

| Field | Value |
|-------|-------|
| First Name | Marina |
| Last Name | Ortega |
| User Logon Name | `M.Ortega` |
| Password | `Password1` |
| User must change password at next logon | Unchecked |
| Password never expires | Checked |

---

#### Step 5 — Move an Existing User to the Globomantics OU

In the left-hand pane, click the **Users** container. In the right-hand pane, right-click the user `mcauthon` and select **Move...**. In the Move dialog, select the **Globomantics** OU and click **OK**.

Click the **Globomantics** OU to confirm both `mcauthon` and `Marina Ortega` are now present.

---

#### Step 6 — Add User to a Security Group

Double-click the **AllUsers** group → **Members tab → Add...**

In the object name field, enter `M.Ortega` and click **Check Names**. The name resolves to:


---

### Objective 2 — Manage Windows Users Using PowerShell

#### Step 1 — Launch PowerShell 7 as Administrator

Run **PowerShell 7** as admin

---

#### Step 2 — Enumerate All Domain Users

Run the following command to generate a report of all user accounts in the domain:

```powershell
Get-ADUser -Filter * | Select-Object samAccountName, givenName, surname
```

This returns a list of all AD users, including their logon names and display names, a quick way to inventory domain accounts or spot unauthorized users.

---

#### Step 3 — Create a New User Account via PowerShell

Run the following command to create user **Casey McCann** in the Globomantics OU:

```powershell
New-ADUser -Name 'Casey McCann' `
  -Path 'OU=Globomantics,DC=Globomantics,DC=co' `
  -GivenName 'Casey' `
  -Surname 'McCann' `
  -SamAccountName 'C.McCann' `
  -AccountPassword (Read-Host -AsSecureString "Input User Password") `
  -PasswordNeverExpires $true `
  -Enabled $True
```

**Command parameter breakdown:**

| Parameter | Purpose |
|-----------|---------|
| `-Name` | Display name of the AD object |
| `-Path` | Distinguished name of the target OU |
| `-GivenName / -Surname` | First and last name attributes |
| `-SamAccountName` | Pre-Windows 2000 logon name (used for authentication) |
| `-AccountPassword` | Prompts securely at runtime — avoids plaintext in script |
| `-PasswordNeverExpires $true` | Exempts account from domain password expiration policy |
| `-Enabled $True` | Creates account in enabled state (default is disabled) |

---

#### Step 4 — Disable Forced Password Change at Logon

```powershell
Set-ADUser C.McCann -ChangePasswordAtLogon $false
```

#### Step 5 — Add User to the NetAdmins Group

```powershell
Add-ADGroupMember NetAdmins -Members C.McCann
```

---

### Objective 3 — Perform Domain Join and Unjoin

#### Step 1 — Unjoin the File Server from the Domain
From the File server
```
Start → Server Manager → Local Server → Domain field (globomantics.co) → Change...
```

In the **Computer Name/Domain Changes** window, select **Workgroup** and enter:

```
WORKGROUP
```
Agree to all the confirmation prompts. Once completed, the server will need to be restarted. Upon restart, the file server will no longer be part of the domain.

---

#### Step 2 — Delete the Stale Computer Object from AD

Return to the **Active Directory Domain Controller** tab. In ADUC, navigate to `globomantics.co → Computers`. Right-click **FS01** and select **Delete**. Click **Yes** to confirm.

> **Why this matters:** After a domain unjoin, the computer object remains in AD and should be removed to keep the directory clean and avoid authentication conflicts if the machine rejoins with a new SID.

---

#### Step 3 — Delegate Domain Join Permissions to NetAdmins

Right-click `globomantics.co` and select **Delegate Control...**

Follow the Delegation of Control Wizard:

| Wizard Step | Selection |
|-------------|-----------|
| Users or Groups | Add `NetAdmins` |
| Tasks to Delegate | Create a custom task to delegate |
| AD Object Type | Only the following objects: **Computer objects** |
| Object Scope | Check **Create selected objects in this folder** and **Delete selected objects in this folder** |
| Permissions | Read, Write, Create All Child Objects, Delete All Child Objects |

Click **Next** and **Finish** to apply the delegation.

> This grants members of `NetAdmins` the ability to join computers to the domain without requiring Domain Admin credentials — a principle of least privilege applied to a common administrative task.

---

#### Step 4 — Rejoin the File Server to the Domain

Return to the **File Server** tab. Navigate to:

```
Start → Server Manager → Local Server → WORKGROUP → Change...
```

Select **Domain** and enter:

```
globomantics.co
```

Click **OK**. When prompted for credentials


The **Welcome to the globomantics.co domain** confirmation message confirms the join succeeded using the non-admin `C.McCann` account — validating the delegation configured in the previous step.

Click **OK**, then **Restart Now** to complete the domain join.

---

## 🔍 Results

- Organizational Unit `Globomantics` created under `globomantics.co` with accidental deletion protection disabled for lab flexibility
- Security groups `NetAdmins` and `AllUsers` created within the Globomantics OU
- User `Marina Ortega` (`M.Ortega`) created via ADUC and added as a member of `AllUsers`; existing user `mcauthon` moved from the default Users container to the Globomantics OU
- `Get-ADUser -Filter *` returned a full enumeration of all domain accounts, confirming PowerShell AD module availability and domain connectivity
- User `Casey McCann` (`C.McCann`) created via `New-ADUser` in the Globomantics OU and added to `NetAdmins` via `Add-ADGroupMember`
- File Server (FS01) successfully unjoined from `globomantics.co`, stale computer object removed from AD Computers container
- Delegation of Control Wizard applied to `globomantics.co` — `NetAdmins` granted scoped Computer object permissions (Read, Write, Create, Delete)
- File Server successfully rejoined `globomantics.co` using `C.McCann` credentials, confirming the delegation granted the correct permissions without Domain Admin rights

---

## 📸 Evidence

| Description | Screenshot |
|------------|------------|
| ADUC showing Globomantics OU with users and groups | null |
| New User wizard — Marina Ortega account creation | null |
| AllUsers group Members tab showing Marina Ortega | null |
| PowerShell — `Get-ADUser` enumeration output | null |
| PowerShell — `New-ADUser` command for Casey McCann | null |
| PowerShell — `Add-ADGroupMember` command output | null |
| ADUC Computers container — FS01 object before deletion | null |
| Delegation of Control Wizard — permissions summary screen | null |
| File Server domain join success dialog (globomantics.co) | null |

---

## 🛡️ Security Analysis / Key Takeaways

**Organizational Units as an Access Control Boundary**

OUs are not just an organizational convenience; they define the scope of Group Policy application and delegation. Creating a dedicated OU for domain objects allows administrators to apply targeted GPOs and delegate management without granting broad domain-level permissions. Poor OU design leads to either overly broad policy application or excessive admin access.

**Delegation of Control vs. Domain Admin Sprawl**

One of the most common misconfigurations in enterprise AD environments is granting Domain Admin rights to users who only need a subset of that access. This lab demonstrates a key alternative: using the Delegation of Control Wizard to grant `NetAdmins` members only the permission needed to join and remove computer objects, nothing more. 

**PowerShell for Repeatable, Auditable Provisioning**

Manual GUI-based account creation is error-prone at scale. PowerShell cmdlets like `New-ADUser` and `Add-ADGroupMember` enable repeatable, scriptable provisioning that can be peer-reviewed, version-controlled, and integrated into onboarding workflows. From a SOC perspective, scripted provisioning also makes deviations (unexpected account creation) more detectable against a baseline.

**Stale Computer Objects as a Security Risk**

After unjoining FS01 from the domain, its computer object remained in AD until manually deleted. Stale computer objects are common in environments without automated lifecycle management and can be leveraged by attackers to pass-the-hash or perform relay attacks against accounts that previously authenticated on that machine. Regular auditing of the computer container is a recommended hygiene practice.


---

## 🧠 MITRE ATT&CK Mapping

| Technique ID | Technique Name | Relevance |
|-------------|---------------|-----------|
| T1136.002 | Create Account: Domain Account | Lab covers domain user creation via ADUC and PowerShell; Event ID 4720 provides detection visibility |
| T1078.002 | Valid Accounts: Domain Accounts | Demonstrates how delegated non-admin accounts can perform privileged actions (domain join), a common attacker pivot path if such accounts are compromised |
| T1087.002 | Account Discovery: Domain Account | `Get-ADUser -Filter *` mirrors the enumeration technique used by attackers to map domain users post-compromise |
| T1098 | Account Manipulation | Adding users to security groups (`Add-ADGroupMember`) changes effective permissions, monitored via Event ID 4728 (member added to global security group) |
| T1484.001 | Domain Policy Modification: Group Policy | Delegation of Control modifies AD object permissions, changes are auditable via Event ID 5136 (directory service object modified) |

---

## 📎 References

- [Pluralsight — Manage Accounts in Active Directory (Lab)](https://app.pluralsight.com/hands-on/labs/e199a271-5764-4256-b654-c9dfda2638c7)
- [Microsoft Docs — ActiveDirectory PowerShell Module](https://learn.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2022-ps)
- [Microsoft Docs — Delegation of Control Wizard](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/delegating-administration-by-using-ou-objects)
- [Microsoft Docs — New-ADUser Cmdlet](https://learn.microsoft.com/en-us/powershell/module/activedirectory/new-aduser)
- [MITRE ATT&CK — T1136.002 Create Account: Domain Account](https://attack.mitre.org/techniques/T1136/002/)
- [MITRE ATT&CK — T1087.002 Account Discovery: Domain Account](https://attack.mitre.org/techniques/T1087/002/)
