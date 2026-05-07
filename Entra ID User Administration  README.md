# Lab Journal: Microsoft Entra ID User Administration & Automation

**Date:** May 6, 2026  
**Administrator:** [AF]  
**Tenant:** Luvekika.onmicrosoft.com  
**License Level:** Microsoft Entra ID P2

---

## 1. Objective

The goal of this lab was to master user lifecycle management within Microsoft Entra ID, transitioning from manual entry to automated bulk operations and exploring administrative task management (blocking, role assignment, and deletion).

## 2. Task 1: Manual User Creation (Internal Members)

**Action:** Created the first set of cloud-only users manually to understand the basic attribute requirements (UPN, Display Name, Job Title, and Department).

**Process:**
- Navigated to **Entra ID > Users > New user**.
- Configured identity details and auto-generated temporary passwords.
- Assigned metadata (Department/Job Title) in the **Properties** tab.

<p align="center">
  <img width="50%" alt="image" src="https://github.com/user-attachments/assets/c718fc6a-a5c5-459c-99aa-f6b6fce08c0e"" />
</p>

## 3. Task 2: Automation via Bulk Operations (CSV Upload)

**Action:** Scaled the operation by creating 10 users simultaneously using the **Bulk Create** tool to save time and ensure consistency.

### The CSV Formatting Strategy
To satisfy the Entra portal requirements, I used the mandatory three-row header format:

| Row | Content |
| :-- | :------ |
| 1 | `version:v1.0` |
| 2 | System headers (`Name`, `UPN`, `Password`, etc.) |
| 3 | Placeholder example |

 ` ` `
version:v1.0
Name [displayName] Required,User name [userPrincipalName] Required,Initial password [passwordProfile] Required,Block sign in (Yes/No) [accountEnabled] Required,Job title [jobTitle],Department [department]
Example User,user@example.com,Password123,No,Title,Department
Alex Johnson,alex.j@Luvekika.onmicrosoft.com,Welcome2026!,No,Sales Rep,Sales
Jamie Lee,jamie.l@Luvekika.onmicrosoft.com,Welcome2026!,No,Marketing Coordinator,Marketing
Taylor Smith,taylor.s@Luvekika.onmicrosoft.com,Welcome2026!,No,Helpdesk Analyst,IT
Jordan Brown,jordan.b@Luvekika.onmicrosoft.com,Welcome2026!,No,HR Generalist,HR
Casey White,casey.w@Luvekika.onmicrosoft.com,Welcome2026!,No,Accountant,Finance
Riley Green,riley.g@Luvekika.onmicrosoft.com,Welcome2026!,No,Sales Manager,Sales
Morgan King,morgan.k@Luvekika.onmicrosoft.com,Welcome2026!,No,Sys Admin,IT
Sam Parker,sam.p@Luvekika.onmicrosoft.com,Welcome2026!,No,Content Specialist,Marketing
Quinn Hughes,quinn.h@Luvekika.onmicrosoft.com,Welcome2026!,No,Ops Analyst,Operations
Blake Rivera,blake.r@Luvekika.onmicrosoft.com,Welcome2026!,No,Paralegal,Legal
 ` ` `

 ### Troubleshooting & Resolution

| Issue | Cause | Resolution |
| :---- | :---- | :---------- |
| Initial upload failed with *"Selected file has no version number"* error | The file lacked the specific `version:v1.0` header | Re-formatted the CSV to include the mandatory headers |
| Bulk create reported *"Completed with errors"* (10 Success, 1 Failure) | The *"Example User"* in Row 3 used `@example.com` (domain not verified in tenant) | Accepted the 10 successful creations; the rejected line was expected |

## 4. Task 3: User Administration & Security

**Action:** Performed post-creation management tasks to simulate real-world IT helpdesk tickets.

- **A. Property Updates:** Modified Job Titles and assigned Entra roles (e.g., *Helpdesk Administrator*).
- **B. Account Security:** Tested the **Block Sign-in** feature to disable access without deleting the user object.
- **C. Password Management:** Performed administrative password resets with the *"Force change on next sign-in"* requirement enabled.

<img width="421" height="221" alt="Screenshot 2026-05-06 193617" src="https://github.com/user-attachments/assets/cbf7b9e7-fd6c-44da-bb6d-9448048d3b70" />

<img width="1631" height="241" alt="Screenshot 2026-05-06 194102" src="https://github.com/user-attachments/assets/218d58d2-8c1f-4b48-ad6f-dc7b36ecaf3e" />

## 5. Task 4: Cleanup & Deletion

**Action:** Practiced the decommissioning process for users no longer requiring access.

**Step:** Selected specific users and used the **Delete User** function.

**Observation:** Noted that deleted users move to **Deleted Users** for 30 days before permanent removal.


<p align="center">
  <img width="50%" alt="image" src="https://github.com/user-attachments/assets/005710df-3cdd-447f-9b03-de1e2c193c10"" />
</p>

## 6. Final Summary & Key Learnings

- ✅ **Automation is King:** Moving from manual creation to CSV bulk operations reduced the setup time for 10 users by approximately **80%**.
- ✅ **Strict Syntax:** The Microsoft Entra Bulk tool is highly sensitive to CSV headers; using the official template is mandatory to avoid `BadRequest` errors.
- ✅ **Tenant Boundaries:** You cannot create users with a domain suffix (e.g., `@example.com`) that has not been verified in your Entra ID tenant.

---




 
