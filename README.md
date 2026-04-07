# Active-Directory
Active Directory security policy lab with Domain Controller (Windows Server 2022) and domain-joined workstation (Windows 11). Implements user accounts, security groups, OUs, and Group Policy Objects (account lockout, employee restrictions, guest access controls).

## 📃 Purpose
I understood what Active Directory is and the role it plays with Group Policies and setting up user accounts. However, my only hands-on experience with Active Directory came from bits and pieces on TryHackMe.

Building this project from beginning to end gave me the complete picture of how Active Directory works, the entire process, the types of restrictions you can place on an Organizational Unit (OU), the granular control you have over each OU, and how much setup is required before you can truly enforce restrictions.

## 📋 Overview
This lab simulates a small corporate Active Directory environment with a Domain Controller and a domain-joined workstation. I built it to gain hands-on experience with core AD administration tasks such as creating Organizational Units, managing user accounts, configuring security groups, and applying Group Policy Objects.

To reflect real workplace scenarios, I implemented foundational policies including account lockout rules, employee access restrictions, and guest access controls. Working through these helped me understand how policy interactions can create unexpected behavior and why structured procedures are essential in enterprise environments.

This project strengthened my ability to design clean AD structures, troubleshoot misconfigurations, and think through the impact of security policies, all in a safe, self-built lab environment.

## 🚀 Skills Demonstrated
- Active Directory Users and Computers (ADUC)
- Group Policy Management
- Role-Based Access Control (RBAC)
- User account creation and management
- Security policy enforcement
- Windows Server administration
- VirtualBox networking

## 🛠️ Tools & Technologies
- VirtualBox 7.x
- Windows Server 2022 (Domain Controller)
- Windows 11 IoT Enterprise LTSC (Workstation)
- Group Policy Management Console
- Active Directory Users and Computers

## 🗺️ Lab Architecture
- **DC-01:** Windows Server 2022 (Domain Controller) — `192.168.100.10`
- **WS-01:** Windows 11 IoT Enterprise LTSC (Workstation) — `192.168.100.50`
- **Network:** Isolated VirtualBox Internal Network

## 👥 User Accounts & Structure
| OU | User | Group Membership | Purpose |
|----|------|------------------|---------|
| Admins | admin_jsmith | Domain Admins | Full domain administration |
| Employees | e_sarah | GG_Employees | Standard employee with restrictions |
| Employees | e_marcus | GG_Employees, GG_Special_Project | Employee with extra folder access |
| Guests | g_public | GG_Guests | Highly restricted public access |

## 📝 Step-by-Step Implementation

### Phase 1: VM Setup
Created two VMs in VirtualBox:
- DC-01 (Windows Server 2022) — 4GB RAM, 2 CPU, 50GB disk
- WS-01 (Windows 11 IoT Enterprise) — 4GB RAM, 2 CPU, 50GB disk

Set both to Internal Network mode for isolation. Assigned static IP addresses and renamed the server to DC-01 [1]. Renamed the workstation to WS-01.
Verified communication by pinging between both VMs.

### Phase 2: Domain Controller Configuration
Installed Active Directory Domain Services (AD DS) role, then promoted the server to a Domain Controller as a new forest: CYBERLAB.local. [2]


### Phase 3: OUs, Users, and Groups
Created three Organizational Units: Admins, Employees, Guests.
**Users created:**
- admin_jsmith (Admins OU)
- e_sarah, e_marcus (Employees OU)
- g_public (Guests OU)

**Security groups created:** [3]
- GG_Employees
- GG_Special_Project
- GG_Guests

**Group memberships assigned:** [4]
- e_sarah, e_marcus → GG_Employees
- e_marcus → GG_Special_Project
- g_public → GG_Guests
- admin_jsmith → Domain Admins

### Phase 4: File Permissions
Created C:\Shares\SpecialProject folder. Granted GG_Special_Project Full Control access. [5]

### Phase 5: Group Policy Configuration
**Default Domain Policy** (applies to all users)[6]:
- Account lockout threshold: 5 invalid attempts
- Account lockout duration: 15 minutes

**Employee_Restrictions** (linked to Employees OU) [7],[8]:
- No Control Panel access
- No command prompt
- No Run menu
- Removed Shut Down/Restart options

**Guest_Restrictions** (linked to Guests OU) [9]:
- Hide C: drive
- Disable USB storage
- Remove password change option
- Logon hours: Monday-Friday, 8 AM – 6 PM

### Phase 6: Joining the Domain
On WS-01, set DNS to 192.168.100.10. Joined CYBERLAB.local domain and rebooted.

### Phase 7: Validation Testing
Logged in as each user to verify restrictions:

**admin_jsmith**  Full access. [10]
**e_sarah**  Control Panel blocked, no shutdown option, CMD blocked. [11]
**e_marcus**  Same restrictions as e_sarah, plus access to SpecialProject folder.
**g_public**  Logon hours enforced [12], C: drive hidden [13], E: drive access denied [14], password change missing [15], Run dialog blocked [16], Start Menu access blocked [17]


## 🐛 Challenges & Lessons Learned
**VM Instability**
The Windows Server VM experienced intermittent freezing, leading to critical logging errors. Multiple reboots and snapshot reverts were required. Lesson: Take snapshots before major changes, this saved hours of rework.

**Lab vs. Production Configuration**
For this lab environment, user accounts were configured with password never expires and user cannot change password. This is for testing convenience only. In a real production environment, passwords would expire after 90 days, users would change their own passwords, and new accounts would require password change at next logon.

**Domain Naming**
Always append .local or .internal to internal domain names. These TLDs are reserved for private networks and prevent DNS conflicts with public domains.

**Testing Domain User Restrictions**
Standard domain users cannot log into Domain Controllers by default, this is a security feature, not a limitation. Use the domain-joined workstation (WS-01) to test user restrictions. In production, regular users never log into Domain Controllers.

**Group Policy Notes**
- The command `gpupdate /force` forces an immediate Group Policy update on a client machine. Without it, policy changes can take up to 90 minutes to apply automatically.
- GPO links can be removed without deleting the underlying policy. Re-linking the GPO reapplies the restrictions.
- When users attempt unapproved actions, they receive a clear "cancelled due to restrictions" error message, confirming the policy is actively enforced.

**The Real Lesson: Process Over Technology**
- The biggest challenge in this lab wasn't technical, it was process. I had a plan, but I didn't realize how easy it was to create contradictions between policies, which caused more problems to troubleshoot.
- This experience gave me a real appreciation for structural operating procedures, how following a clear, documented process helps avoid errors and makes everything more efficient.

## 📸 Screenshots
| # | Screenshot | Description |
|---|------------|-------------|
| [1] | ![re-namedDC-01](https://github.com/user-attachments/assets/5b8653f6-e32d-4fc5-bd94-daafa2df759b) | Renaming server to DC-01 |
| [2] | ![ServerManger_dashboard](https://github.com/user-attachments/assets/d98ffbef-04c9-4ce0-9c0b-52293b6b8fdf) | AD DS installed, promoted to Domain Controller |
| [3] | ![ADUC_Group_to_user](https://github.com/user-attachments/assets/7ab20bd5-9273-41fd-873b-0c1cfb84dc24) | Security groups created (GG_Employees, GG_Special_Project) |
| [4] | ![e_marcus_member_of_group](https://github.com/user-attachments/assets/d7be9c17-f7b8-45fd-a7dc-a80eccc7cd61) | Group memberships for e_marcus |
| [5] | ![Premission_of_SpecialProject](https://github.com/user-attachments/assets/f2eb4618-c6bd-400a-8716-dafa086061a8) | GG_Special_Project folder permissions |
| [6] | ![DefaultDomainPolicy_changes](https://github.com/user-attachments/assets/a246d222-d145-48a1-be27-ccd947c420fc) | Account lockout policy settings |
| [7] | ![Employee_restriction_in_EmployeesOU](https://github.com/user-attachments/assets/fff66513-e388-4f83-aa7e-f236a37ad9fa) | Employee_Restrictions linked to Employees OU |
| [8] | ![Removing_powerbutton_default_setup](https://github.com/user-attachments/assets/8343f2a2-cc35-42d3-8a0a-6ae2910111d6) | Power options removed for employees |
| [9] | ![Logon_hours_for_public_visitor](https://github.com/user-attachments/assets/e49c578e-dd0e-42b3-8309-61c425c9f6fa) | Guest logon hours configuration |
| [10] | ![Login_Admin_jsmith](https://github.com/user-attachments/assets/755f9389-5973-4bb8-8ea6-07cfbbc2dfa5) | Admin login successful |
| [11] | ![login_to_employee_Account_via_windows11](https://github.com/user-attachments/assets/73c79ba4-24c6-4920-9701-a74fc21b79d9) | Employee login successful |
| [12] | ![Logon_restriction_in_action](https://github.com/user-attachments/assets/d454a6a7-2406-411d-ae0c-4cc39f4295a0) | Guest denied outside logon hours |
| [13] | ![Policy_to_remove_Cdrive_in_action](https://github.com/user-attachments/assets/087fd883-c530-437d-a400-931f53b21c98) | C: drive hidden for guest |
| [14] | ![shows_EDrive_not_accessible](https://github.com/user-attachments/assets/30d6dda4-8718-49ae-bb72-c3b528a268fe) | E: drive access denied |
| [15] | ![Change_password_removed](https://github.com/user-attachments/assets/992a0ef6-8f24-411f-b3e4-716bde3d464f) | Password change option missing |
| [16] | ![Restriction_prompt](https://github.com/user-attachments/assets/b261c143-02c8-4285-86d9-e2ecb9dba1d4) | Run dialog blocked by policy |
| [17] | ![Shows_restriction_via_trying_to_access_application_through_searchmenu](https://github.com/user-attachments/assets/7abfc40e-6b18-4de0-a37f-c122499c92ec) | Start Menu access blocked |


## 📚 References
- [Microsoft Learn: Active Directory Domain Services Overview](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview)
- [Microsoft Learn: Group Policy Overview](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/group-policy/group-policy-overview)
- [Windows Server 2022 Evaluation Center](https://www.microsoft.com/en-us/evalcenter/download-windows-server-2022)
- [VirtualBox Documentation: Internal Networking](https://www.virtualbox.org/manual/ch06.html#network_internal)
