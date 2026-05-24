# 📌 Manage Accounts in Active Directory

---

## 📘 Overview

This repository documents my completion of a Pluralsight hands-on lab focused on managing a Windows Server Active Directory environment using both GUI administration tools and PowerShell automation.

The lab simulated enterprise administrative tasks, including:

- Creating and managing Organizational Units (OUs)
- Managing users and groups in Active Directory
- Automating account management with PowerShell
- Delegating permissions for domain joins
- Joining and removing systems from an Active Directory domain

The exercises reinforced foundational Windows administration and identity management concepts commonly used in enterprise IT and cybersecurity operations.


![Platform](https://img.shields.io/badge/platform-Pluralsight-39ff7a?style=flat-square)
![Status](https://img.shields.io/badge/status-complete-39ff7a?style=flat-square)
![Date](https://img.shields.io/badge/date-2026-05-23-blue?style=flat-square)
---

## 🎯 Objectives
- Manage Active Directory users and groups using GUI tools
- Create Organizational Units(OU) and security groups
- Use PowerShell cmdlets to automate account creation
- Delegate domain join permissions to non-administrative users.
- Remove and rejoin systems to an Active Directory domain
- Validate domain authentication and delegation functionality
- Document administrative processes and outcomes

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

## 🧠 Security Concepts Demonstrated
Identity and Access Management (IAM)
Active Directory Administration
Least Privilege Delegation
Role-Based Access Control
Windows Domain Management
PowerShell Automation
Account Provisioning
Privileged Group Management


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

**Organizational Structure (OUs)**  
Active Directory environments rely on proper organizational structure and controlled administrative access. Organizational Units (OUs) provide a way to logically separate users, groups, and systems while also defining the scope for Group Policy application and delegated administration. Proper OU design reduces unnecessary exposure of privileges and improves administrative efficiency.

**Delegation & Least Privilege**  
Delegated permissions provide a more secure alternative to assigning broad Domain Admin rights. Limiting access to only required tasks, such as joining and removing systems from the domain, supports the principle of least privilege while maintaining operational efficiency.

**PowerShell Automation**  
PowerShell improves consistency and reduces manual error through repeatable account provisioning workflows. Cmdlets such as `New-ADUser` and `Add-ADGroupMember` allow administrative tasks to be scripted, reviewed, and reused across environments.

**Lifecycle & Cleanup**  
Computer objects may remain in Active Directory after systems are removed from the domain, creating stale entries. Regular auditing and cleanup of unused objects is important for maintaining a secure and accurate directory environment.


---

## 🧠 MITRE ATT&CK Mapping

| ID | Tactic / Technique | How it Appears in This Lab |
|---|---|---|
| TA0001 | Initial Access | Domain authentication and access workflows |
| TA0003 | Persistence | Active Directory account creation and management |
| TA0004 | Privilege Escalation | Delegated administrative permissions |
| TA0006 | Credential Access | User credential and password management |
| TA0007 | Discovery | Enumeration of domain users using PowerShell |

---


🛠️ Skills Gained
Active Directory Administration
Windows Server Management
PowerShell Automation
User and Group Management
Domain Join Troubleshooting
Role-Based Access Control
Identity & Access Management
Organizational Unit Design
Delegated Administration

💼 Resume Bullets
Completed hands-on Active Directory administration lab using Windows Server and PowerShell
Managed users, groups, Organizational Units, and delegated domain permissions
Automated Active Directory account provisioning using PowerShell cmdlets
Performed domain join and unjoin operations using delegated administrative access
Documented enterprise-style Windows administration procedures and outcomes


## 📎 References

- [Pluralsight — Manage Accounts in Active Directory (Lab)](https://app.pluralsight.com/hands-on/labs/e199a271-5764-4256-b654-c9dfda2638c7)
- [Microsoft Docs — ActiveDirectory PowerShell Module](https://learn.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2022-ps)
- [Microsoft Docs — Delegation of Control Wizard](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/delegating-administration-by-using-ou-objects)
- [Microsoft Docs — New-ADUser Cmdlet](https://learn.microsoft.com/en-us/powershell/module/activedirectory/new-aduser)
- [MITRE ATT&CK — T1136.002 Create Account: Domain Account](https://attack.mitre.org/techniques/T1136/002/)
- [MITRE ATT&CK — T1087.002 Account Discovery: Domain Account](https://attack.mitre.org/techniques/T1087/002/)

👤 About Me

I'm Shawn, an IT and cybersecurity student building hands-on technical experience through lab environments, home labs, and documented security projects.

🌐 [GitHub](https://github.com/Shawn-Nichol)
🔗 [Linkedin](https://www.linkedin.com/in/shawn-nichol/?skipRedirect=true)
