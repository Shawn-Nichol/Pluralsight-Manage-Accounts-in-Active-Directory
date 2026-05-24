# 📌 Manage Accounts in Active Directory
<p align="center">
  <img src="Lab%20cover%20photo.png" width="700"/>
</p>

## 📘 Overview / Objectives

This project documents a hands-on Active Directory administration lab completed in a Windows Server environment using both graphical tools and PowerShell automation. The lab focused on simulating common enterprise identity and access management tasks, including user provisioning, group management, and delegated administration.

The primary objectives of this lab were to gain practical experience with Active Directory structure, automate administrative tasks using PowerShell, and apply the principle of least privilege through delegated control.

- Created and managed Organizational Units (OUs) to structure Active Directory environments
- Administered user and group accounts using Active Directory tools and PowerShell
- Automated account provisioning and group membership with PowerShell cmdlets
- Implemented delegated administration using role-based access control and least privilege principles
- Performed domain join and unjoin operations and validated authentication behavior

![Platform](https://img.shields.io/badge/platform-Pluralsight-39ff7a?style=flat-square)
![Status](https://img.shields.io/badge/status-complete-39ff7a?style=flat-square)
![Date](https://img.shields.io/badge/date-2026-05-23-blue?style=flat-square)
---


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

## 🚶 Walkthrough

### ⚙️ 1. Active Directory Administration (GUI)
- Created a new Organizational Unit (OU) to structure domain resources
- Created security groups for role-based access control (NetAdmins, AllUsers)
- Provisioned a new user account using Active Directory Users and Computers (ADUC)
- Moved existing user objects into the newly created OU
- Assigned users to security groups to enforce access control policies

### 💻 2. Active Directory Automation (PowerShell)
- Enumerated all domain users using Get-ADUser to validate directory visibility
- Created a new user account using New-ADUser with secure password handling
- Modified user properties using Set-ADUser to control authentication behavior
- Added users to security groups using Add-ADGroupMember

### 🔐 3. Domain Management & Delegation
- Performed domain unjoin of a Windows Server to simulate machine removal
- Cleaned up stale computer objects from Active Directory
- Configured delegated permissions using the Delegation of Control Wizard
- Granted scoped administrative rights for computer object management (least privilege model)
- Successfully rejoined the server to the domain using delegated credentials


---

## 🧠 Security Concepts Demonstrated

- Identity and Access Management (IAM) within Active Directory environments  
- Least privilege access control through delegated administration  
- Role-Based Access Control (RBAC) using security groups and Organizational Unit (OU) structure  
- Active Directory account lifecycle management (creation, modification, and removal)  
- Privileged access governance and administrative boundary enforcement  
- Secure authentication and domain-based identity validation  

---

## 🛠️ Skills Gained

- Managing Active Directory users, groups, and Organizational Units (OUs)  
- Automating account provisioning using `PowerShell` and the `ActiveDirectory` module  
- Executing directory queries using `Get-ADUser` for account enumeration  
- Creating and modifying user objects using `New-ADUser` and `Set-ADUser`  
- Managing group membership using `Add-ADGroupMember`  
- Performing domain join and unjoin operations in Windows Server environments  
- Implementing delegated administration using the `Delegation of Control Wizard`  

---

## 🔍 Results
- Successfully created and structured the `Globomantics` Organizational Unit within Active Directory, establishing a dedicated environment for user and group management

- Security groups `NetAdmins` and `AllUsers` were created and validated for role-based access control implementation

- User accounts were successfully provisioned using both `Active Directory Users and Computers (ADUC)` and `PowerShell`, confirming multi-method administrative capability

- PowerShell-based user enumeration using `Get-ADUser` and account creation using `New-ADUser` confirmed proper configuration and functionality of the `ActiveDirectory` module

- Group membership assignments were successfully applied using `Add-ADGroupMember`, validating the enforcement of access control through security groups

- A Windows Server was successfully unjoined from the domain, and stale computer objects were removed from the `Active Directory Computers` container to maintain directory hygiene

- Delegation of Control was successfully configured using the `Delegation of Control Wizard`, granting scoped administrative permissions to NetAdmins following the principle of least privilege

- The server was successfully rejoined to the `globomantics.co` domain using delegated credentials, validating correct permission scoping and authentication workflow

---

## 🛡️ Security Analysis / Key Takeaways

- Organizational Units (OUs) define administrative boundaries within Active Directory, enabling structured delegation and controlled policy application across identity resources  

- Delegation of Control and least privilege principles reduce reliance on Domain Admin accounts by limiting permissions to only required administrative actions, such as computer object management  

- PowerShell automation improves consistency and reduces manual error in identity and access management workflows through repeatable cmdlets such as `New-ADUser`, `Set-ADUser`, and `Add-ADGroupMember`  

- Active Directory account and computer lifecycle management is essential for maintaining directory hygiene, reducing stale objects, and minimizing unnecessary attack surface exposure after system changes or removals  

---
## 📸 Evidence
| Description | Evidence |
|------------|----------|
| Active Directory Users and Computers (ADUC) showing `Globomantics` OU with user groups | ![ADUC Globomantics OU](images/1.11%20ADUC%20Globomantics%20OU%20with%20User%20group.png) |
| User creation for `Marina Ortega` via ADUC | ![Marina Ortega User Creation](images/1.12%20User%20creation%20Marina%20Ortega.png) |
| `Globomantics` OU showing user successfully added | ![User Added to OU](images/1.16%20Globomantics%20OU%20user%20added.png) |
| PowerShell output for `Get-ADUser` enumeration | ![Get-ADUser Output](images/2.3%20PowerShell%20Get-ADUSER.png) |
| PowerShell execution for user creation of `Casey McCann` | ![New AD User Casey McCann](images/2.4%20PowerShell%20Add%20user%20Casey%20McCann.png) |
| PowerShell user configuration steps (`New-ADUser`, `Set-ADUser`, `Add-ADGroupMember`) | ![PowerShell User Configuration](images/2.5%20to%202.7%20PowerShell%20User%20configurations.png) |
| File Server `FS01` confirmed as part of the domain | ![FS01 Domain Join](images/3.4%20FS01%20part%20of%20the%20domain.png) |
| File Server `FS01` successfully moved to workgroup (domain removal) | ![FS01 Workgroup](images/3.6%20FS01%20moved%20to%20workgroup.png) |

---

## 🧠 MITRE ATT&CK Mapping

| Technique ID | Technique Name | Relevance |
|-------------|---------------|-----------|
| `T1087.002` | Account Discovery: Domain Account | Enumeration of domain user accounts can be used to identify valid identities within Active Directory for reconnaissance purposes |
| `T1136.002` | Create Account: Domain Account | Domain account creation represents lifecycle management activity that can be abused if performed by unauthorized or compromised privileged users |
| `T1069.002` | Permission Groups Discovery: Domain Groups | Discovery and management of security groups provides visibility into privileged roles and access structure within the domain |
| `T1098` | Account Manipulation | Modification of user attributes and group membership reflects administrative control over identity and access, which is a key area for privilege abuse monitoring |

---

## 💼 Resume Bullets

Completed hands-on Active Directory administration lab using Windows Server and PowerShell
Managed users, groups, Organizational Units, and delegated domain permissions
Automated Active Directory account provisioning using PowerShell cmdlets
Performed domain join and unjoin operations using delegated administrative access
Documented enterprise-style Windows administration procedures and outcomes

---
## 📎 References

- [Pluralsight — Manage Accounts in Active Directory (Lab)](https://app.pluralsight.com/hands-on/labs/e199a271-5764-4256-b654-c9dfda2638c7)
- [Microsoft Docs — ActiveDirectory PowerShell Module](https://learn.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2022-ps)
- [Microsoft Docs — Delegation of Control Wizard](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/delegating-administration-by-using-ou-objects)
- [Microsoft Docs — New-ADUser Cmdlet](https://learn.microsoft.com/en-us/powershell/module/activedirectory/new-aduser)
- [MITRE ATT&CK — T1136.002 Create Account: Domain Account](https://attack.mitre.org/techniques/T1136/002/)
- [MITRE ATT&CK — T1087.002 Account Discovery: Domain Account](https://attack.mitre.org/techniques/T1087/002/)

---

👤 About Me

I'm Shawn, an IT and cybersecurity student building hands-on technical experience through lab environments, home labs, and documented security projects.

🌐 [GitHub](https://github.com/Shawn-Nichol)
🔗 [Linkedin](https://www.linkedin.com/in/shawn-nichol/?skipRedirect=true)
