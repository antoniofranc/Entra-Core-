# 📓 Lab Journal: Microsoft Entra ID Security Hardening & Break-Glass Architecture

## 📌 Project Overview

This laboratory journal documents the step-by-step implementation, architectural decisions, and real-time troubleshooting involved in securing a Microsoft Entra ID tenant using Conditional Access and Identity Protection (P2 features). A primary objective of this project was enforcing strict security controls (e.g., Mandatory MFA, Blocking Legacy Auth) while implementing a bulletproof Break-Glass Architecture to guarantee zero risk of tenant lockout.

---

## 🛠 Phase 1: Break-Glass Account Architecture

### 1. Implementation Steps & Architecture

A dedicated, cloud-only emergency access account was engineered to bypass all active security policies.

- **Menu Path**: Navigation via `Identity > Users > All Users > New User > Create New User`
- **User Principal Name (UPN)**: `emergency-admin@://onmicrosoft.com`
- **Domain Selection**: Default `*.onmicrosoft.com` domain. Custom vanity/federated domains were intentionally avoided to eliminate external DNS or identity provider (IdP) dependencies during an outage.
- **Directory Role Assignment**: Permanently assigned the Global Administrator role via the Assigned Roles interface. Privileged Identity Management (PIM) and Just-In-Time (JIT) access were bypassed because dependency on the PIM engine introduces a single point of failure during an identity platform outage.
- **Password Policy Modification**: Configured via Microsoft Graph PowerShell to ensure the cryptographic 16+ character password explicitly overrides tenant-wide aging policies:

```powershell
Connect-MgGraph -Scopes "User.ReadWrite.All"
Update-MgUser -UserId "emergency-admin@://onmicrosoft.com" -PasswordPolicies "DisablePasswordExpiration"
```
-----
### 2. Critical Architectural Corrections (Troubleshooting Log)
The Guest Account Misconception:

- Initial thought: Invite an external/existing Azure account as a guest user.

- Correction: ❌ Rejected. Guest identities create an operational dependency on external authentication systems. If the external domain or Microsoft's consumer login mesh fails, the guest account cannot authenticate, leading to a permanent lockout. The account must be native to the host tenant.

The Shared Daily Account Risk:

- Initial thought: Utilize the primary administrator account (AntonioFrancisco@...) as the break-glass wrapper.

- Correction: ❌ Rejected. Daily operational accounts face high exposure to phishing, session hijacking, and malicious token theft. Dedicated, dormant infrastructure accounts isolation is required.

----

## 🔒 Phase 2: Tenant Baseline Hardening & Conditional Access
 
### 1. Troubleshooting Log: Deactivating Security Defaults
 
Upon navigating to Protection > Conditional Access > Policies, the policy engine displayed 0 active policies along with a generic introductory splash page.
 
**Root Cause Analysis:** The tenant was protected by Microsoft's automated Security Defaults. Security Defaults enforce blanket legacy MFA prompts across all accounts globally and prohibit the exclusion of specific users. This directly blocks the creation of an isolated break-glass account.
 
**Resolution Path:**
1. Navigated directly to Identity > Overview.
2. Selected the Properties tab from the middle menu column under the Manage header.
3. Scrolled to the absolute bottom of the frame and clicked the Manage security defaults link.
4. Toggled the status from Enabled to Disabled.
5. Selected the mandatory justification: "My organization is using Conditional Access."
---

### 2. Policy Deployments
 
#### Policy A: Enforce MFA for All Users
 
**Objective:** Prevent credential-stuffing attacks by applying mandatory MFA across the enterprise.
 
**Configuration Parameters:**
- **Users:** Include: All Users | Exclude: Users and groups → `emergency-admin@://<tenant>.onmicrosoft.com`
- **Target resources:** Include: All cloud apps.
- **Access controls (Grant):** Select Grant access → Check Require multifactor authentication.
- **Enable policy:** Switch toggle to On.
> **Troubleshooting Note:** A standard warning popup ("Don't lock yourself out!") was presented by the portal. Because the break-glass account was verified under the Exclude tab, the warning was safely bypassed by checking "I understand that my account will be impacted by this policy. Proceed anyway" and saving.
 
---

#### Policy B: Block Legacy Authentication
 
**Objective:** Neutralize automated credential-stuffing backdoors targeting older email protocols that lack modern authentication support.
 
**Configuration Parameters:**
- **Users:** Include: All Users | Exclude: Users and groups → `emergency-admin@://<tenant>.onmicrosoft.com`
- **Target resources:** Include: All cloud apps.
- **Conditions:** Click Client apps → Set Configure to Yes → Check Exchange ActiveSync clients and Other clients (POP, IMAP, SMTP, MAPI, etc.). Leave Browser and Mobile/Desktop unchecked.
- **Access controls (Grant):** Select Block access.
- **Enable policy:** Switch toggle to On.
---

 ## 🔬 Phase 3: Validation, Testing & Real-Time Log Auditing
 
### 1. Break-Glass Validation Check (Pass)
 
**Methodology:** Initialized a sterile browser environment (InPrivate/Incognito). Executed a login sequence using the newly created BreakGlass account credentials.
 
**Observed Behavior:** The account authenticated directly into the Entra Admin Center dashboard, successfully bypassing all MFA onboarding prompts.
<img width="832" height="392" alt="image" src="https://github.com/user-attachments/assets/b60b3827-a907-4f3c-8f7e-cbc93a5b4ecb" />

----
### 2. Regular User Validation Check & Behavioral Nuances
 
**Test Execution:** Authenticated an Entra ID P2 licensed user via an Incognito browser session to verify MFA policy enforcement.
 
**Anomaly Observed:** The user was logged in without an immediate MFA prompt; instead, they encountered a forced password change screen (Update Password).
 
**Technical Root Cause:** Entra ID architecture serializes administrative identity workflows. A forced credential modification takes cryptographic precedence over Conditional Access policies.
 
**Outcome:** Upon completion of the password update sequence, the Entra token pipeline evaluated the active ruleset, refreshed the session, and successfully presented the mandatory "More information required" MFA setup wizard.
 
---

### 3. Log Analysis of Modern Auth vs. Legacy Block Policies
 
**Observed Log State:** Browser sign-in details evaluated under the Block Legacy Authentication policy returned a status of Not Applied.
 
**Log Verification:** This is the architecturally correct state. Because a web browser utilizes modern OpenID Connect/OAuth 2.0 flows, it completely falls outside the policy's target condition (Legacy Client Apps). The policy remains dormant until an unauthenticated protocol request (e.g., IMAP via Thunderbird) attempts traversal.

<img width="971" height="352" alt="Screenshot 2026-05-16 182008" src="https://github.com/user-attachments/assets/173e4c42-db49-4a7c-af56-f622f364d5b4" />

 ---
 ## 💡 Phase 4: Advanced License Integrations (Entra ID P2)
 
### 1. Identity Protection Risk-Based Adaptations
 
Leveraging the active Entra ID P2 license, an advanced behavioral risk layer was deployed inside Conditional Access to supplant depreciated Identity Protection blade modules:
 
**Policy C (High User Risk):**
- **Parameters:** Include All Users | Exclude Break-Glass | Condition: User Risk = High | Grant Control: Require password change.
- **Purpose:** Automatically intercepts and remediates leaked dark-web credentials via automated Self-Service Password Reset.
**Policy D (Medium/High Sign-In Risk):**
- **Parameters:** Include All Users | Exclude Break-Glass | Condition: Sign-in Risk = Medium & High | Grant Control: Require multifactor authentication.
- **Purpose:** Triggers an immediate step-up challenge for abnormal location tracking anomalies (e.g., anonymous proxies or impossible travel).
**Operational Status:** Kept in Report-only mode to establish a baseline before production enforcement.

<img width="1497" height="637" alt="image" src="https://github.com/user-attachments/assets/bcb6526d-2f93-4c3f-86e2-bd02969be56f" />

----

### 2. Policy Postponements & Architectural Deferrals
 
**Policy: Require Compliant Device for Admins:** Explicitly deferred. Because Microsoft Intune or an active Active Directory Hybrid cloud sync anchor is not yet deployed, moving this policy to Report-only or On would trigger a 100% operational failure rate and cause an immediate administrator lockout.

----

## 📋 Final Production Policy Matrix
 
| Policy Name | Purpose / Target | Inclusion Scope | Exclusions | Control Mechanism | Production Status |
|---|---|---|---|---|---|
| Enforce MFA for All Users | Mitigate unauthorized entry across all cloud apps | All Users | Break-Glass Account | Require MFA | 🟢 On |
| Block Legacy Authentication | Disable obsolete protocol backdoors (POP/IMAP/SMTP) | All Users | Break-Glass Account | Block Access | 🟢 On |
| Enforce Password Reset for High User Risk | Force SSPR upon automated compromise detection | All Users | Break-Glass Account | Require Password Change | 🟡 Report-only |
| Enforce MFA for Med/High Sign-in Risk | Step-up authentication challenge for abnormal paths | All Users | Break-Glass Account | Require MFA | 🟡 Report-only |
| Require Compliant Device for Admins | Restrict admin operations to managed endpoints | All Admin Roles | Break-Glass Account | Intune Device Compliance | ❌ Deferred |

--
 
## 🚨 Emergency Recovery: Breaking the Glass (Emergency Operations)
 
This section outlines the operational playbook for utilizing the break-glass account and the recovery steps required if a misconfiguration inadvertently overrides these rules and locks out standard administrators.
 
### 1. How to Activate the Break-Glass Account
 
When a primary authentication mechanism fails:
 
1. Retrieve the physical credential escrow document from the office vault.
2. Open a completely isolated browser session using Incognito or InPrivate mode.
3. Navigate directly to the native login path: `https://microsoft.com`
4. Enter the `emergency-admin@://<tenant>.onmicrosoft.com` username and the full string password.
5. Once inside, immediately navigate to Protection > Conditional Access > Policies to disable or modify the misconfigured policy causing the incident.
---
 
### 2. Troubleshooting Matrix: What to Do If Locked Out
 
| Symptom / Error | Probable Root Cause | Emergency Remediation Action |
|---|---|---|
| "More information required" prompt appears on the Break-Glass account. | The break-glass account was accidentally omitted from the Exclude tab of an active Conditional Access policy, or Security Defaults was re-enabled. | Log into the portal with the account anyway. Complete the registration of a temporary MFA method (like an authenticator app). Once authenticated, fix the policy exclusions immediately, then remove the temporary MFA method from the account. |
| "Your account has been blocked due to suspicious activity" on the Break-Glass account. | The account was caught by an automated Identity Protection Risk Policy because it logged in from an unverified location or IP address. | 1. Navigate to Protection > Identity Protection > Risky Users using a secondary admin account (if accessible). 2. Select the break-glass user and click Dismiss user risk. 3. If completely locked out of all admin accounts, you must initiate a standard Microsoft Support Data Protection request. |
| The primary admin account is blocked, and the Break-Glass account password cannot be found. | The physical document has been lost, corrupted, or incorrectly written down. | 1. Attempt login from a previously verified, compliant device that might hold an active, valid token cache. 2. If unsuccessful, a global admin must contact the Microsoft Data Protection Team via phone to initiate a tenant recovery process. Prepare your corporate domain verification records (DNS TXT/MX keys) to prove legal ownership of the tenant. |
 
---
 
### 3. Disaster Recovery Scenario: Fixing an Accidental Global Lockdown
 
If an administrator accidentally creates a policy targeting "All Users" with a "Block" or "Require Compliant Device" mechanism without excluding themselves or the break-glass account, the tenant enters a state of total administrative lockout.
 
#### Phase 1: Leverage Surviving Session Tokens
 
- Do not close any open browser tabs on any administrator machine.
- Check every workstation used by the IT team to see if someone still has an active session inside the Microsoft Entra admin center or Azure Portal.
- If an active session exists, the policy change might not have propagated to their active token yet. Use that open window to immediately delete or disable the rogue policy.
#### Phase 2: Utilizing PowerShell Backdoors
 
If the web portal is completely blocked, Conditional Access rules occasionally take slightly longer to sync to the Graph API endpoints. Attempt an emergency backend connection via PowerShell:
 
```powershell
# Attempt a direct connection bypass
Connect-MgGraph -Scopes "Policy.ReadWrite.ConditionalAccess"
 
# If connection succeeds, list all policies to find the rogue ID
Get-MgIdentityConditionalAccessPolicy | Select-Object Id, DisplayName, State
 
# Force-disable the misconfigured policy (Replace with your actual rogue Policy ID)
Update-MgIdentityConditionalAccessPolicy -ConditionalAccessPolicyId "YOUR-ROUGUE-POLICY-ID" -State "disabled"
```
 
#### Phase 3: The Last Resort (Microsoft Data Protection Escalation)
 
If all local recovery steps fail, a global admin must contact the Microsoft Data Protection Team via phone to initiate an enterprise lockout override. This process typically takes between 24 to 72 hours and requires rigorous off-band DNS validation ownership proofs.










