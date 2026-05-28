# 🔐 JML Lifecycle Management with Microsoft Entra ID, Dynamic Groups & SCIM Provisioning

## 📋 Project Overview
 
This project demonstrates a complete **Joiner–Mover–Leaver (JML)** identity lifecycle using **Microsoft Entra ID P2**, dynamic groups, group-based licensing, and **SCIM provisioning** integrated with Salesforce.
 
The objective was to automate user onboarding, role changes, and offboarding while reducing manual administrative work and improving identity governance.
 
---

## 🛠️ Technologies Used
 
| Technology | Purpose |
|---|---|
| Microsoft Entra ID P2 | Identity platform and licensing |
| Dynamic Security Groups | Automated group membership |
| Group-Based Licensing | Automated license assignment |
| SCIM 2.0 | Application provisioning protocol |
| Salesforce Enterprise Application | Target provisioning application |
| SAML Single Sign-On | Federated authentication |
| Microsoft Entra Provisioning Service | Sync engine |
 
---

## 📖 What is JML Lifecycle Management?
 
| Phase | Meaning | Purpose |
|---|---|---|
| **Joiner** | New employee onboarding | Create accounts, assign access, provision applications |
| **Mover** | Employee role/department change | Update permissions and synchronize access |
| **Leaver** | Employee termination/offboarding | Remove access, disable accounts, deprovision applications |
 
---

## ✅ Prerequisites Checklist
 
- [ ] Microsoft Entra ID P2 license
- [ ] Signed in as **Groups Administrator** or **Global Administrator**
- [ ] Test users created (Alex Johnson, Jamie Lee, Taylor Smith, etc.)
- [ ] Salesforce app configured as an Enterprise Application
---

## 📚 Table of Contents
 
- [Step 1: Configure Group-Based Licensing](#step-1-configure-group-based-licensing)
- [Step 2: Create Dynamic Groups](#step-2-create-dynamic-groups)
- [Step 3: Configure SCIM Provisioning for Salesforce](#step-3-configure-scim-provisioning-for-salesforce)
- [Step 4: Simulate Joiner Scenario](#step-4-simulate-joiner-scenario-onboarding)
- [Step 5: Simulate Mover Scenario](#step-5-simulate-mover-scenario-role-change)
- [Step 6: Simulate Leaver Scenario](#step-6-simulate-leaver-scenario-termination)
- [Dynamic Group Rule Reference](#dynamic-group-rule-reference)
- [SCIM Attribute Mapping Reference](#scim-attribute-mapping-reference)
- [Troubleshooting Guide](#troubleshooting-guide)
- [Security Best Practices](#security-best-practices)

---

### 1a. Create a License Group
 
1. Sign in to **Microsoft Entra admin center**
2. Browse to **Identity > Groups > All groups**
3. Click **+ New group**

| Setting | Value |
|---|---|
| Group type | Security |
| Group name | `Licensed Users - P2` |
| Membership type | Assigned |
| Description | Users who receive Microsoft Entra ID P2 license |

<img width="459" height="359" alt="Screenshot 2026-05-21 204121" src="https://github.com/user-attachments/assets/0edbfa7d-e09d-4a74-9335-61d4dd7f1410" />

### 1b. Add Members to the License Group
 
1. Open the group → **Members** → **+ Add members**
2. Add your test users (Alex Johnson, Jamie Lee, Taylor Smith, etc.)

<img src="https://github.com/user-attachments/assets/3354bf28-dd7d-4976-9c6b-0d60bf8ed4cf" width="60%">

### 1c. Assign License to the Group
 
1. In the group page, click **Licenses** (left menu)
2. Click **+ Assignments**
3. Select **Microsoft Entra ID P2** (and any other licenses needed)
4. Click **Save**

<img src="https://github.com/user-attachments/assets/68bd63cd-ce4c-4195-a678-7ffc02550500" width="60%">


### 1d. Verify License Assignment
 
1. Go to **Identity > Users** → Select a member user
2. Click **Licenses** — should show P2 license assigned via the group

<img src="https://github.com/user-attachments/assets/be82a780-c0ae-4643-a083-507de2e1fb81" width="60%">

---
 
## Step 2: Create Dynamic Groups
 
Dynamic groups automatically add/remove members based on attribute rules.

### 2a. Create a Dynamic Group for "Sales Department"
 
1. Go to **Groups > All groups > + New group**

| Setting | Value |
|---|---|
| Group type | Security |
| Group name | `Dynamic Group - Sales` |
| Membership type | Dynamic User |
| Description | Automatically includes all Sales department users |

<img src="https://github.com/user-attachments/assets/29d96a78-7023-4387-8f27-463c6ab55d25" width="50%">

### 2b. Configure the Dynamic Membership Rule
 
Using the **Rule Builder**:
 
| And/Or | Property | Operator | Value |
|---|---|---|---|
| | `department` | Equals | `Sales` |
 
**Rule Syntax (Advanced):**
 
```
(user.department -eq "Sales")
```
<img src="https://github.com/user-attachments/assets/8a18f45c-aa3f-4cc6-b19b-877d38cb8049" width="60%">

### 2c. Create a Second Dynamic Group — "All Users with P2 License"
 
Create another dynamic group using this rule syntax:
 
```
user.assignedPlans -any (assignedPlan.servicePlanId -eq "41781fb2-bc02-4b7c-bd55-b576c07bb09d" and assignedPlan.capabilityStatus -eq "Enabled")
```

 <img src="https://github.com/user-attachments/assets/af9d69df-e3dd-4b17-bd8f-dcc1dfaffb90"  width="60%">

### 2d. Check Processing Status
 
1. Open the dynamic group → **Overview**
2. View **Dynamic rule processing status**
   
| Status | Meaning |
|---|---|
| Evaluating | Rule validation started |
| Processing | Membership calculations running |
| Update Complete | Group membership updated successfully |

<img src="https://github.com/user-attachments/assets/8f21962c-18ad-4afc-ad19-5074030342dd"  width="50%">
 
 ---

## Step 3: Configure SCIM Provisioning for Salesforce
 
### 3a. Open Provisioning Configuration
 
1. Go to **Identity > Applications > Enterprise applications**
2. Select **Salesforce**
3. Click **Provisioning** (left menu)

### 3b. Configure Admin Credentials
 
| Setting | Value |
|---|---|
| Provisioning Mode | Automatic |
| Admin Username | `your-admin@yourdomain.com` |
| Secret Token | *(Salesforce SCIM token)* |

<img src="https://github.com/user-attachments/assets/1da7bbb4-9bb8-40de-b593-4ad44f6e6d30"  width="50%">

### 3c. Configure Attribute Mappings
 
1. Click **Provisioning > Edit attribute mapping**
2. Review and configure mappings
**Key Mappings:**
 
| Microsoft Entra Attribute | Target SCIM Attribute |
|---|---|
| `userPrincipalName` | `userName` |
| `givenName` | `name.givenName` |
| `surname` | `name.familyName` |
| `mail` | `emails[type="work"].value` |
| `department` | `Department` |

<img src="https://github.com/user-attachments/assets/3fe7d09b-b7df-4881-867b-a50e16dd3d2c"  width="60%">

### 3d. Set Provisioning Scope
 
| Setting | Recommended Value |
|---|---|
| Scope | Sync only assigned users and groups |
| Prevent accidental deletion | Yes |
| Provisioning Status | On |

<img src="https://github.com/user-attachments/assets/93131a3d-bfed-4ca0-9d35-a6fe9826cc9a"  width="60%">

### 3e. Assign Users and Groups to Salesforce
 
1. In the Salesforce application, click **Users and groups**
2. Click **+ Add user/group**
3. Select your dynamic groups or users
4. Choose the appropriate Salesforce role (e.g., Standard User)

<img src="https://github.com/user-attachments/assets/a7bd1573-3a5a-48a1-a8a2-499d2127521a"  width="60%">

---

## Step 4: Simulate Joiner Scenario (Onboarding)
 
### Scenario
 
> New employee **Morgan Chen** joins as a Sales Associate. They need a Microsoft Entra user account, P2 license, and access to Salesforce.
 
### 4a. Create New User
 
| Field | Value |
|---|---|
| User principal name | `morgan.chen@tenant.onmicrosoft.com` |
| Display name | Morgan Chen |
| Department | Sales |
| Job title | Sales Associate |
 
### 4b. Verify Dynamic Group Membership
 
Since `department = Sales`, the user is automatically added to `Dynamic Group - Sales`.
 
### 4c. Verify Provisioning to Salesforce
 
Check **Provisioning logs** for Salesforce to confirm the user was created.
 
### Joiner Summary
 
| Step | Action | System | Status |
|---|---|---|---|
| 1 | Create user in Entra ID | Microsoft Entra | ✅ |
| 2 | User added to dynamic group | Dynamic Group - Sales | ✅ |
| 3 | License assigned via group | Group-based licensing | ✅ |
| 4 | User provisioned to Salesforce | SCIM provisioning | ✅ |
 
---
 
## Step 5: Simulate Mover Scenario (Role Change)
 
### Scenario
 
> **Morgan Chen** is promoted from Sales Associate to Sales Manager.
 
### 5a. Update User Attributes
 
1. Go to **Identity > Users** → Select Morgan Chen
2. Click **Properties**
3. Update **Job title** from `Sales Associate` to `Sales Manager`
4. Click **Save**
### 5b. Verify SCIM Sync
 
Check **Provisioning logs** for Salesforce to confirm the job title was updated.
 
### Mover Summary
 
| Step | Action | Automation | Status |
|---|---|---|---|
| 1 | Update job title in Entra ID | Manual | ✅ |
| 2 | Dynamic groups recalculate | Automatic | ✅ |
| 3 | License reassessed | Automatic | ✅ |
| 4 | SCIM updates target app | Automatic | ✅ |
 
---
 
## Step 6: Simulate Leaver Scenario (Termination)
 
### Scenario
 
> Employee **Taylor Smith** is leaving the organization.
 
### 6a. Block User Sign-In
 
1. Go to **Identity > Users** → Select Taylor Smith
2. Click **Settings**
3. Set **Account enabled** = `No`
### 6b. Verify Deprovisioning
 
Check **Provisioning logs** for Salesforce. The user should be set to `active = false`.
 
### Leaver Summary
 
| Step | Action | System | Status |
|---|---|---|---|
| 1 | Block sign-in (Account enabled = No) | Microsoft Entra | ✅ |
| 2 | Remove from dynamic groups | Automatic | ✅ |
| 3 | License released | Automatic | ✅ |
| 4 | Deprovision from Salesforce | SCIM (set active=false) | ✅ |
 
---
 
## Dynamic Group Rule Reference
 
| Use Case | Rule Expression |
|---|---|
| All users in Sales department | `user.department -eq "Sales"` |
| All managers | `user.jobTitle -contains "Manager"` |
| All US-based users | `user.country -eq "US"` |
| All users with P2 license | `user.assignedPlans -any (assignedPlan.servicePlanId -eq "41781fb2-bc02-4b7c-bd55-b576c07bb09d" and assignedPlan.capabilityStatus -eq "Enabled")` |
| External guests only | `user.userType -eq "Guest"` |
| Combined rule | `(user.department -eq "Sales" -or user.department -eq "Marketing") -and user.country -eq "US"` |
 
---
 
## SCIM Attribute Mapping Reference
 
| Microsoft Entra | SCIM (Salesforce) | Required |
|---|---|---|
| `userPrincipalName` | `userName` | ✅ |
| `mail` | `emails[type="work"].value` | ✅ |
| `givenName` | `name.givenName` | ✅ |
| `surname` | `name.familyName` | ✅ |
| `active` | `active` | ✅ |
| `department` | `Department` | Optional |
| `manager` | `Manager` | Optional |
 
---
 
## Troubleshooting Guide
 
| Issue | Resolution |
|---|---|
| Dynamic group not updating | Check processing status; wait 5–10 minutes; verify rule syntax |
| SCIM sync failing | Test connection; verify secret token; check attribute mappings |
| License not applying | Verify group membership; check available licenses |
| User not in Salesforce | Verify app assignment and provisioning scope |
| Attribute mismatch error | Review SCIM attribute mappings (e.g., invalid Salesforce fields) |
| Credential validation failed | Regenerate SCIM token in Salesforce; test connection again |
 
---
 
## 📊 Project Statistics
 
| Metric | Value |
|---|---|
| Dynamic Groups Created | 2 |
| SCIM-Enabled Applications | 1 (Salesforce) |
| Provisioning Method | SCIM 2.0 |
| License Type | Microsoft Entra ID P2 |
| Average Sync Time | ~40 minutes |
| Lifecycle Scenarios Tested | Joiner, Mover, Leaver |
| Automation Level | 90–100% |
 
---
 
## 🔒 Security Best Practices
 
| Best Practice | Reason |
|---|---|
| Use dynamic groups | Reduces manual errors and improves consistency |
| Use group-based licensing | Simplifies license management at scale |
| Enable accidental deletion threshold | Prevents mass deprovisioning disasters |
| Monitor provisioning logs daily | Detect sync failures early |
| Test SCIM with pilot users first | Avoid large-scale configuration issues |
| Use least privilege access | Reduces security risks |
 
---
 
## 🏁 Conclusion
 
This project demonstrates a practical implementation of **Identity and Access Management (IAM)** automation using Microsoft Entra ID and SCIM provisioning.
 
By combining:
 
- **Dynamic groups** for automatic membership management
- **Group-based licensing** for simplified license management
- **SCIM automation** for application provisioning
- **JML lifecycle management** for complete identity governance
The organization can significantly reduce manual administrative tasks, improve security posture, and ensure users always have the correct access throughout their employment lifecycle.
 
> This project reflects real-world IAM operations commonly used in enterprise cloud environments.
 
---
 
 
*Built with ❤️ using Microsoft Entra ID P2 and SCIM 2.0*
 





