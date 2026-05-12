# Lab Journal: Implementing Just-In-Time (JIT) Privileged Access Management

**Date:** May 11, 2026  
**Author:** [AF]  
**Technology Stack:** Microsoft Entra ID, Privileged Identity Management (PIM), Microsoft 365

---

## 1. Project Overview

The goal of this lab was to move away from "Permanent Admin" accounts (standing access) and implement **Just-In-Time (JIT) access** using Microsoft Entra Privileged Identity Management (PIM). This ensures users only have administrative rights when needed, for a fixed duration, and with proper auditing.

# Microsoft Entra PIM Activation Flow

```text
[User signs in]
       ↓
[Goes to PIM > My roles]
       ↓
[Clicks Activate on eligible role]
       ↓
[Provides justification + duration]
       ↓
[Completes MFA if required]
       ↓
[Request submitted]
       ↓
    ┌───────────────┴───────────────┐
    ↓                               ↓
[Approval required]         [No approval required]
    ↓                               ↓
[Admin reviews request]     [Role activates immediately]
    ↓
[Admin approves]
    ↓
    └───────────────┬───────────────┘
                    ↓
      [Role active for limited time]
                    ↓
       [User performs admin tasks]
                    ↓
       [Role expires automatically]
```


## 2. Environment Setup & User Provisioning

I began by bulk-importing a test user to act as our "Helpdesk Analyst."

- **User:** Taylor Smith  
- **UPN:** `taylor.s@Luvekika.onmicrosoft.com`  
- **Initial Role:** Assigned as an active Helpdesk Administrator (to be converted).

<img width="1620" height="263" alt="Screenshot 2026-05-11 212703" src="https://github.com/user-attachments/assets/df784f5b-7741-4df0-90df-5c2a0c9e7da6" />

---

## 3. Configuration Steps

### Step 1: Transitioning to Eligible Assignments

Instead of Taylor having the Helpdesk Administrator role active 24/7, I converted the assignment to **Eligible**.

1. Navigated to **PIM > Microsoft Entra roles > Assignments**  
2. Located the existing Active assignment for Taylor Smith  
3. Updated the assignment type from **Active** to **Eligible**

<img width="308" height="506" alt="Screenshot 2026-05-11 213330" src="https://github.com/user-attachments/assets/abdb7302-4bdf-42de-9cdb-32eda9ab5c9e" />

---

### Step 2: Configuring Role Activation Settings (JIT Policy)

To harden the Helpdesk Administrator role, I modified the activation requirements:

| Setting | Configuration |
|---------|--------------|
| Max Activation Duration | 2 Hours (Reduces exposure window) |
| MFA Requirement | Enabled (Ensures identity verification) |
| Justification | Required (Ensures an audit trail) |
| Approval Workflow | Enabled (Requires Global Admin sign-off) |

<img width="307" height="381" alt="Screenshot 2026-05-11 213759" src="https://github.com/user-attachments/assets/641a3642-d4f4-460b-828b-40306b6aeba7" />

---

## 4. Troubleshooting: The "Missing Role" Issue

During testing, I encountered a critical issue where Taylor Smith could sign in but could not see any roles under **PIM > My Roles**.

### Issue Diagnosis

- **Symptom:** User was visible as "Eligible" in the Admin portal, but "My Roles" was blank for the user.  
- **Root Cause:** Licensing. While the Global Admin had a Microsoft Entra ID P2 license, **the end-user (Taylor Smith) did not**.  
- **Requirement:** Microsoft PIM requires every user who interacts with an eligible assignment to be licensed for Entra ID P2 or Microsoft 365 E5.

### Resolution

1. Accessed **Active Users** in the M365 Admin Center  
2. Assigned an **Entra ID P2 license** to Taylor Smith  
3. Performed a forced logout and logged back in via an Incognito window to refresh the security token

**Result:** The "Helpdesk Administrator" role appeared correctly in the "Eligible Assignments" tab.

---

## 5. Testing the Activation Workflow

### 5a. User Request

1. Logged in as `taylor.s@Luvekika.onmicrosoft.com`  
2. Navigated to **PIM > My Roles > Microsoft Entra roles**  
3. Click **Activate** on the Helpdesk Administrator role  
4. Entered justification: *"Processing password resets for Ticket #12345"*

<img width="1042" height="542" alt="Screenshot 2026-05-11 215457" src="https://github.com/user-attachments/assets/6f5d00ce-19b3-4c1e-ae6b-e5f137ae9bda" />

### 5b. Admin Approval

1. Switched to Global Admin account  
2. Navigated to **PIM > Approve Requests**  
3. Reviewed Taylor's justification and approved the request

---

## 6. Validation of Permissions

Once activated, I verified Taylor's permissions:

- **Target:** Resetting a password for a non-admin user (Alex Johnson)  
- **Result:** Success. The "Reset Password" button became available **only after activation**  
- **Expiration:** Verified that after the 2-hour window, the role automatically reverted to "Inactive," and permissions were stripped

## 7. Audit & Review

Using the PIM Audit History, I confirmed that every step of this process was logged:

| Action | Details |
|--------|---------|
| Who requested | Taylor Smith |
| Who approved | Global Admin |
| Justification | "Processing password resets for Ticket #12345" |
| Timestamp | Logged for compliance |

<img width="1531" height="398" alt="Screenshot 2026-05-11 221514" src="https://github.com/user-attachments/assets/e31fd4be-f6c8-464e-811b-34ac5c1d704b" />

## 8. Final Summary Checklist

- [x] Verify P2 license assigned to both Admin and User  
- [x] Convert permanent active roles to Eligible  
- [x] Configure JIT settings (MFA, Justification, Duration)  
- [x] Successfully activate role via PIM portal  
- [x] Validate permissions expire automatically  


---

## References

- [Microsoft Entra Privileged Identity Management Documentation](https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/pim-configure)
- [Just-In-Time Access Best Practices](https://learn.microsoft.com/en-us/security/privileged-access-workstations/privileged-access-deployment)









