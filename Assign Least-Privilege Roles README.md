# Lab Journal: Implementing Entra ID Least-Privilege Architecture

## 📌 Project Overview

The goal of this project was to move away from "Global Admin" dependency and implement a Zero Trust administrative model within a Microsoft Entra ID (formerly Azure AD) tenant. I focused on delegating administrative tasks using scoped roles and Administrative Units to ensure that users have exactly the permissions they need—and nothing more.

---

## 🛠 Phase 2: Role Assignment & Delegation

### 1. The "Privileged Role Administrator" Check

Before assigning roles, I performed a self-audit of my tenant permissions.

> **Troubleshoot Note:** As the Global Admin, I noticed the Privileged Role Administrator (PRA) role was empty.

**Discovery:** I learned that while a Global Admin has inherent rights to manage roles, the PRA role is a more specific "least-privilege" version for role management. I decided to maintain my Global Admin status for the "Break-glass" account but used the PRA logic to delegate further assignments.

---

### 2. Implementation of Scoped Roles

I categorized my 10 test users into different security tiers to demonstrate three levels of scoping:

#### Level A: Tenant-Wide (Limited Function)

**Action:** Assigned Helpdesk Administrator to Taylor Smith.

**Security Control:** Taylor can reset passwords for regular users but cannot modify other admins or access billing.

<img width="1380" height="367" alt="Screenshot 2026-05-10 221201" src="https://github.com/user-attachments/assets/4aa0cfda-5a82-4910-91d3-d5c3e794015b" />

---

#### Level B: Administrative Unit Scoped (The "True" Least-Privilege)

**Action:** Created an Administrative Unit (AU) titled **"West Coast Sales Unit"**.

**The Workflow:**
1. Created the AU
2. Added specific members (Alex Johnson, Riley Green)
3. Assigned Jamie Lee as User Administrator inside the AU

**Security Control:** Jamie is now a "User Admin" but is effectively "blind" to any users outside of the West Coast Sales Unit. This prevents accidental or malicious changes to the wider organization.


<img width="70%" alt="image" src="https://github.com/user-attachments/assets/cc381eb2-e191-45e4-bcec-ff22c625a933" />

<img width="70%" alt="image" src="https://github.com/user-attachments/assets/4643dd95-8972-4a7e-b437-4fbe3edd92f9" />

<img width="70%" alt="image" src="https://github.com/user-attachments/assets/2d68de24-4a92-4560-a8cd-978dab324a26" />

<img width="70%" alt="image" src="https://github.com/user-attachments/assets/bdc4a964-2e96-4167-89f1-5fca4cffd949" />

## 📝 Troubleshooting & Key Insights

### The "Restricted Management" Decision

During the creation of the Administrative Unit, I had to decide whether to enable Restricted Management.

- **The Conflict:** Does "Least Privilege" mean I should lock everyone out?
- **The Resolution:** I chose **"No"** for Restricted Management.
- **Reasoning:** Restricted Management blocks even Global Admins from managing the AU members unless specifically assigned. For this project, a standard AU provided the necessary delegation without creating a management "black hole" for the primary tenant admin.

---

### PIM (Privileged Identity Management) Integration

Using the Entra ID P2 Trial, I moved beyond "Permanent" assignments.

- **Implementation:** Changed roles from *Active* to *Eligible*
- **Result:** Admins now have **Zero Standing Access**. They must "activate" their role, provide a business justification, and operate within a 4-hour window.

## 📊 Deliverable: Role Assignment Matrix

| User | Assigned Role | Scope | Security Justification |
|------|---------------|-------|------------------------|
| Taylor Smith | Helpdesk Admin | Tenant-wide | Limit password resets to non-admin staff only. |
| Jamie Lee | User Admin | AU: West Coast Sales | Cannot impact users in other departments/units. |
| Alex Johnson | Groups Admin | AU: West Coast Sales | Manage sales groups only. |

## ✅ Summary

This project successfully demonstrated:
- Moving away from permanent Global Admin dependencies
- Implementing tiered, scoped role assignments
- Configuring Administrative Units for department-level isolation
- Setting up PIM for Just-In-Time access











