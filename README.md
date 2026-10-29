# Joiner–Mover–Leaver (JML) Identity Lifecycle Lab — Microsoft Entra ID

![Platform](https://img.shields.io/badge/Platform-Microsoft%20Entra%20ID-0078D4) ![Tier](https://img.shields.io/badge/License-Entra%20ID%20Free-success) ![Time](https://img.shields.io/badge/Time-~2%20hours-lightgrey)

## Overview

This lab simulates how an IT/identity team manages a user's access across their entire time at an organization — from the day they're hired, through a role change, to the day they leave. This process is called **Joiner–Mover–Leaver (JML)**, and it's one of the core responsibilities of Identity and Access Management (IAM).

I built a small fictional organization, **Northbridge University IT**, inside a Microsoft Entra ID tenant and walked three users through each lifecycle stage, documenting every step with screenshots and audit-log evidence.

| Stage | What it means | Who in this lab |
|---|---|---|
| **Joiner** | A new hire gets an account and *only* the access their role needs | Maya Carter — new Finance Analyst |
| **Mover** | An existing employee changes roles; they gain new access **and lose old access** | Jordan Lee — Help Desk → Security Analyst |
| **Leaver** | An employee departs; their access is cut off quickly and safely | Chris Patel — Finance Analyst leaving |

### Skills demonstrated
- User provisioning and attribute management (job title, department, manager, employee ID)
- Security group design and group-based access
- Least privilege and preventing **privilege creep**
- Microsoft Entra admin role assignment and removal
- Secure offboarding: block sign-in, revoke sessions, remove access, soft-delete and restore
- Auditing and evidence collection using Entra ID audit logs

---

## Table of Contents
1. [Before You Start](#before-you-start)
2. [Part 1 — Build the Environment](#part-1--build-the-environment-25-min)
3. [Part 2 — Joiner](#part-2--joiner-maya-carter-20-min)
4. [Part 3 — Mover](#part-3--mover-jordan-lee-20-min)
5. [Part 4 — Leaver](#part-4--leaver-chris-patel-25-min)
6. [Part 5 — Audit and Evidence](#part-5--audit-and-evidence-15-min)
7. [How This Works in the Real World](#how-this-works-in-the-real-world)

---

## Part 1 — Build the Environment (~25 min)

Before we can run joiners, movers, and leavers, we need an "organization" to work inside: departments (groups) and a few existing employees.

### Why groups?
In a real organization you never hand out access one person at a time. You give access to a **group** (e.g., "Finance department") and then put people in that group. When someone joins, you add them to the group; when they leave the team, you remove them. This is called **group-based access control**, and it's what makes JML manageable and auditable.

### Naming convention
We'll use a consistent naming pattern so anyone can tell what a group is for at a glance:
- `SG-` = Security Group
- `Dept-` = based on department

### Step 1.1 — Create the security groups
1. Go to **Groups → All groups → New group**.
2. Fill in:
   - **Group type:** Security
   - **Group name:** `SG-AllStaff`
   - **Group description:** `All active Northbridge University IT employees`
   - **Membership type:** Assigned
3. Leave Owners and Members empty for now. Click **Create**.
4. Repeat for these groups:

| Group name | Description |
|---|---|
| `SG-AllStaff` | All active employees |
| `SG-Dept-Finance` | Finance department — access to finance systems and shares |
| `SG-Dept-IT-HelpDesk` | IT Help Desk team — ticketing and support tools |
| `SG-Dept-Security` | Security Operations team — security tooling and logs |

> **What "Assigned" means:** Membership is controlled manually by an admin. The other option, **Dynamic**, automatically adds users based on attributes like Department — that requires a P1 license. See [How This Works in the Real World](#how-this-works-in-the-real-world).

📸 **Screenshot `01-groups-created.png`** — The **All groups** list showing all four `SG-` groups. (Type `SG-` in the search box to filter the list.)

![Groups created](screenshots/01-groups-created.png)

### Step 1.2 — Create the existing employees
These are people who already "work" at Northbridge before the lab starts.

1. Go to **Users → All users → New user → Create new user**.
2. **Basics tab:**
   - **User principal name:** `dana.brooks` @yourdomain
   - **Display name:** `Dana Brooks`
   - **Password:** leave **Auto-generate password** checked. Click the copy icon and save it somewhere temporarily (a Notepad file you'll delete after the lab).
   - **Account enabled:** checked
3. **Properties tab** (fill in what's listed below, leave the rest blank):
   - **First name / Last name**
   - **Job title**
   - **Department**
   - **Company name:** `Northbridge University`
   - **Employee ID**
4. **Review + create → Create.**

Create these four users:

| Display name | Username | Job title | Department | Employee ID | Manager (set in Step 1.3) |
|---|---|---|---|---|---|
| Dana Brooks | dana.brooks | Finance Manager | Finance | NB1001 | — |
| Sam Rivera | sam.rivera | Director of IT | IT | NB1002 | — |
| Jordan Lee | jordan.lee | Help Desk Analyst | IT Help Desk | NB1003 | Sam Rivera |
| Chris Patel | chris.patel | Finance Analyst | Finance | NB1004 | Dana Brooks |

> **Save Chris Patel's temporary password.** We'll sign in as Chris later to prove the leaver process actually blocks access.

### Step 1.3 — Set managers
Managers matter because in real organizations they approve access requests and are notified during offboarding.

1. Open **Jordan Lee → Edit properties → Job Information** tab.
2. Under **Manager**, click **Change** (or **Add manager**), select **Sam Rivera**, and **Save**.
3. Do the same for **Chris Patel** with **Dana Brooks** as manager.

### Step 1.4 — Give existing employees their current access
1. Open **SG-AllStaff → Members → Add members** and add all four users.
2. Open **SG-Dept-Finance → Members → Add members** and add **Dana Brooks** and **Chris Patel**.
3. Open **SG-Dept-IT-HelpDesk → Members → Add members** and add **Sam Rivera** and **Jordan Lee**.

### Step 1.5 — Give Jordan a help desk admin role
Help desk staff usually need to reset users' passwords. In Entra ID, that permission comes from the **Helpdesk Administrator** role.

1. Open **Jordan Lee → Assigned roles → Add assignments**.
2. Search for and select **Helpdesk Administrator**. Click **Add** (or **Next → Assign**, depending on your portal version).

> **What an admin role is:** Group membership controls access to *resources* (files, apps). Entra **admin roles** control what someone can do *to the directory itself* — reset passwords, create users, read security reports. Admin roles are high-risk, so they should be removed the moment someone doesn't need them anymore. This will matter in the Mover step.

📸 **Screenshot `02-baseline-users.png`** — **All users** list showing the four employees. Use **Manage view → Edit columns** (or the **Columns** button) to add **Job title** and **Department** so the screenshot shows them.

![Baseline users](screenshots/02-baseline-users.png)

📸 **Screenshot `03-jordan-baseline-role.png`** — Jordan Lee's **Assigned roles** page showing **Helpdesk Administrator**.

![Jordan baseline role](screenshots/03-jordan-baseline-role.png)

---

## Part 2 — Joiner: Maya Carter (~20 min)

**Scenario:** HR notifies IT that **Maya Carter** has been hired as a **Finance Analyst**, reporting to **Dana Brooks**, starting Monday. IT needs to create her account with exactly the access a Finance Analyst needs — nothing more.

### Step 2.1 — Create the account
1. **Users → All users → New user → Create new user.**
2. **Basics:**
   - **User principal name:** `maya.carter`
   - **Display name:** `Maya Carter`
   - **Auto-generate password:** checked — copy and save it.
   - **Account enabled:** checked
3. **Properties:**
   - First/Last name: `Maya` / `Carter`
   - Job title: `Finance Analyst`
   - Department: `Finance`
   - Company name: `Northbridge University`
   - Employee ID: `NB1005`
   - Manager: `Dana Brooks` (you can set it here on the Properties tab, or afterward through Edit properties)
4. **Assignments tab:** click **Add group**, select **SG-AllStaff** and **SG-Dept-Finance**.
   - *If your portal version doesn't show an Assignments tab, create the user, then add her to both groups from her profile → **Groups → Add memberships**.*
5. **Review + create → Create.**

> **Why the auto-generated password?** IT should never know a user's real password. An auto-generated temporary password is handed to the new hire through a secure channel (usually their manager or HR), and Entra forces them to set their own password at first sign-in.

> **Why these groups and nothing else?** This is **least privilege**: give people the minimum access needed to do their job. Maya is in Finance, so she gets the Finance group and the all-staff group. She does not get Help Desk or Security access, and she gets **no admin roles**.

📸 **Screenshot `04-joiner-review-properties.png`** — The **Review + create** page before you click Create, showing Maya's details: Job title, Department, Employee ID, and Manager (Dana Brooks).

![Joiner review properties](screenshots/04-joiner-review-properties.png)

📸 **Screenshot `05-joiner-groups.png`** — Maya's **Groups** page showing **SG-AllStaff** and **SG-Dept-Finance** only.

![Joiner groups](screenshots/05-joiner-groups.png)

### Step 2.2 — Verify the new hire can sign in (first-day test)
1. Open a **private/InPrivate browser window**.
2. Go to <https://myaccount.microsoft.com> and sign in as `maya.carter@yourdomain` with the temporary password.
3. You'll be forced to **update the password**. Set a new one (save it for the lab).
   - If you're prompted to set up MFA, you can complete it with the Microsoft Authenticator app or choose **Skip for now** if offered.
4. Once signed in, click **Groups** (or **Organizations**) on the My Account page to see what she belongs to.
5. Close the private window.

> **Why test?** A joiner process isn't done until the person can actually work on day one. Verifying sign-in catches issues like a typo in the username before the new hire is stuck on their first morning.

📸 **Screenshot `06-joiner-first-signin.png`** — Maya's **My Account** page after signing in (crop out the URL bar if it shows anything sensitive).

![Joiner first signin](screenshots/06-joiner-first-signin.png)

---

## Part 3 — Mover: Jordan Lee (~20 min)

**Scenario:** **Jordan Lee** has been promoted from **Help Desk Analyst** to **Security Analyst** on the Security Operations team. Jordan still reports to Sam Rivera.

### The key concept: privilege creep
The most common mistake in a mover process is **only adding** the new access. Over years, someone who moves through four teams ends up with access to all four. This is called **privilege creep** (or access accumulation), and it's a real security risk: if that account is ever compromised, the attacker gets everything the person ever had.

A correct mover process does **both**:
1. **Add** access for the new role
2. **Remove** access from the old role

### Step 3.1 — Document "before" access
Before changing anything, record what Jordan currently has. This is your audit trail.

📸 **Screenshot `07-mover-before-groups.png`** — Jordan's **Groups** page showing **SG-AllStaff** and **SG-Dept-IT-HelpDesk**.

![Mover before groups](screenshots/07-mover-before-groups.png)

*(You already captured Jordan's role in `03-jordan-baseline-role.png` — that's your "before" for roles.)*

### Step 3.2 — Update job attributes
1. Open **Jordan Lee → Edit properties → Job Information**.
2. Change:
   - **Job title:** `Security Analyst`
   - **Department:** `Security`
3. **Save.**

> **Why update attributes?** Job title and department are the "source of truth" many systems read from — the company directory, email signatures, org charts, and (in larger tenants) dynamic groups and automated workflows. Stale attributes cause wrong access later.

### Step 3.3 — Add new access
1. Open **SG-Dept-Security → Members → Add members**, add **Jordan Lee**.
2. Open **Jordan Lee → Assigned roles → Add assignments**, select **Security Reader**, add it.

> **Why Security Reader and not Security Administrator?** A Security Analyst needs to *view* security reports, alerts, and settings. They don't need to *change* security policy. Security Reader is read-only. Least privilege again.

### Step 3.4 — Remove old access
1. Open **SG-Dept-IT-HelpDesk → Members**, check **Jordan Lee**, click **Remove**, confirm.
2. Open **Jordan Lee → Assigned roles**, check **Helpdesk Administrator**, click **Remove assignments**, confirm.

> **Leave SG-AllStaff alone** — Jordan is still an employee.

📸 **Screenshot `08-mover-after-properties.png`** — Jordan's profile showing the new Job title (**Security Analyst**) and Department (**Security**).

![Mover after properties](screenshots/08-mover-after-properties.png)

📸 **Screenshot `09-mover-after-groups.png`** — Jordan's **Groups** page now showing **SG-AllStaff** and **SG-Dept-Security** (Help Desk is gone).

![Mover after groups](screenshots/09-mover-after-groups.png)

📸 **Screenshot `10-mover-after-roles.png`** — Jordan's **Assigned roles** page showing only **Security Reader** (Helpdesk Administrator is gone).

![Mover after roles](screenshots/10-mover-after-roles.png)

> **Tip for your write-up:** Put screenshots 07 and 09 side by side in your README as a "Before / After" table. Recruiters love a clear before/after.

---

## Part 4 — Leaver: Chris Patel (~25 min)

**Scenario:** **Chris Patel** (Finance Analyst) has resigned. Today is the last day. HR asks IT to remove access at 5:00 PM.

### The order matters
Offboarding has a specific order:

| # | Action | Why this order |
|---|---|---|
| 1 | **Block sign-in** | Stops any *new* sign-ins immediately. This is the "lock the door" step. |
| 2 | **Revoke sessions** | Blocking sign-in doesn't kick out someone who's *already* signed in. Their apps hold tokens that can stay valid for a while. Revoking sessions invalidates their refresh tokens so they're forced to re-authenticate — and they can't, because sign-in is blocked. |
| 3 | **Reset the password** | Prevents use of any password the person may have saved or shared. *(In Entra ID this must be done **before** blocking sign-in, because disabled accounts can't have their password reset. See Step 4.3.)* |
| 4 | **Remove groups and roles** | Removes access to resources. Done *after* the lockout so there's no gap where they could still act. |
| 5 | **Update attributes** | Records the departure for audit purposes. |
| 6 | **Soft-delete** (after a retention period) | In most organizations the account sits disabled for a set time (e.g., 30 days) so data can be handed off. We'll delete now to demonstrate the recovery feature. |

### Step 4.0 — Create an active session first (so we can prove the lockout works)
1. Open a **private/InPrivate window**, go to <https://myaccount.microsoft.com>, and sign in as `chris.patel@yourdomain` with the temporary password you saved. Set a new password when prompted (save it).
2. Leave this window open — this represents Chris still being "logged in" at their desk.

### Step 4.1 — Block sign-in
1. In your admin window, open **Chris Patel → Edit properties → Settings** tab.
2. **Uncheck Account enabled.**
3. **Save.**

### Step 4.2 — Revoke sessions
1. On Chris Patel's **Overview** page, click **Revoke sessions** (it's in the toolbar at the top of the user's page).
2. Confirm **Yes**.

### Step 4.3 — Reset the password (troubleshooting note)
1. On Chris's page, click **Reset password**.
2. You'll get a message saying the password can't be reset because you lack permissions **or the account has been disabled**.

> **What I found:** Entra ID won't reset the password on a disabled account. Since Chris is already blocked and his sessions are revoked, the account is still fully locked out. But it showed me the right order for a real runbook: **reset the password first, then block sign-in immediately after, then revoke sessions**, all done back to back so there's no gap.

📸 **Screenshot `11a-leaver-reset-blocked.png`** — The Reset password panel showing the error. This documents a real troubleshooting finding.

![Leaver reset blocked](screenshots/11a-leaver-reset-blocked.png)

### Step 4.4 — Prove the lockout works
1. **Close** the private window from Step 4.0, open a **new** private window, and try to sign in as Chris at <https://myaccount.microsoft.com> using the password you set in Step 4.0.
2. You should get an error saying the account has been **locked or disabled**.

📸 **Screenshot `11-leaver-signin-blocked.png`** — The sign-in error page showing Chris cannot sign in. **This is one of the most impressive screenshots in the lab** — it proves the control actually works, not just that you clicked a button.

![Leaver signin blocked](screenshots/11-leaver-signin-blocked.png)

### Step 4.5 — Remove access
1. Open **Chris Patel → Groups**. Take the "before" screenshot first:

📸 **Screenshot `12-leaver-before-groups.png`** — Chris's **Groups** page showing **SG-AllStaff** and **SG-Dept-Finance**.

![Leaver before groups](screenshots/12-leaver-before-groups.png)

2. Remove Chris from **SG-Dept-Finance** and **SG-AllStaff** (open each group → **Members** → select Chris → **Remove**; or from Chris's **Groups** page, select the groups → **Remove memberships**).
3. Check **Chris Patel → Assigned roles**. There should be none — confirm it's empty.

### Step 4.6 — Update attributes
1. **Edit properties → Job Information**:
   - **Job title:** `Former Employee - Finance Analyst`
   - **Department:** `Terminated`
2. **Save.**

> **Why not delete right away?** In production, accounts are usually *disabled* and kept for a retention period so the manager can recover files, emails can be forwarded, and any legal hold requirements are met. Deleting immediately can destroy data the business still needs.

📸 **Screenshot `13-leaver-disabled-state.png`** — Chris's profile showing **Account enabled: No**.

![Leaver disabled state](screenshots/13-leaver-disabled-state.png)

📸 **Screenshot `13a-leaver-updated-job-info.png`** — Chris's job title and department updated to **Former Employee - Finance Analyst** / **Terminated**.

![Leaver updated job info](screenshots/13a-leaver-updated-job-info.png)

📸 **Screenshot `13b-leaver-no-groups.png`** — Chris's **Groups** page, now empty.

![Leaver no groups](screenshots/13b-leaver-no-groups.png)

### Step 4.7 — Soft-delete and restore (demonstrating recovery)
1. Go to **Users → All users**, check **Chris Patel**, click **Delete**, confirm.
2. Go to **Users → Deleted users**. Chris appears here.

> **Soft delete:** Deleted users stay in a recycle-bin state for **30 days**, then are permanently deleted. This protects against mistakes — for example, if the wrong person was offboarded.

📸 **Screenshot `14-leaver-deleted-users.png`** — The **Deleted users** page showing Chris Patel and the permanent deletion date.

![Leaver deleted users](screenshots/14-leaver-deleted-users.png)

3. *Recovery demo:* Imagine HR says Chris was offboarded by mistake. Check **Chris Patel → Restore users → OK.**
4. Back in **All users**, confirm Chris is back — but notice the account is **still disabled** and **not in any groups**. Restoring an account brings back the object, not the access you removed.
5. Since Chris really did leave, **delete Chris again** to finish the lab cleanly.

---

## Part 5 — Audit and Evidence (~15 min)

Every change you made was logged. In real IT, this is how you prove to auditors (or your manager) that a process was followed — and how you investigate when something goes wrong.

### Step 5.1 — Review the audit log
1. Go to **Monitoring & health → Audit logs**.
2. Set **Date** to the last 24 hours.
3. Look for these activities (use **Add filters → Activity**, or just scroll):
   - `Add user` — Maya's account created
   - `Add member to group` — group assignments
   - `Update user` — Jordan's title/department change, Chris being disabled
   - `Remove member from group` — Jordan leaving Help Desk, Chris removed from all groups
   - `Add member to role` / `Remove member from role` — Jordan's role swap
   - `Delete user` / `Restore user` — Chris's deletion and recovery
4. Click any entry to see **who** made the change, **when**, **what** changed (the **Modified Properties** tab shows old vs new values).

📸 **Screenshot `15-audit-log-overview.png`** — The audit log list showing a mix of the activities above.

![Audit log overview](screenshots/15-audit-log-overview.png)

📸 **Screenshot `16-audit-log-detail.png`** — Open the **Update user** entry for Jordan's promotion → **Modified Properties** tab, showing Job title changing from **Help Desk Analyst** to **Security Analyst**. This is great proof of a mover event.

![Audit log detail](screenshots/16-audit-log-detail.png)

> **Note:** Entra ID Free keeps audit logs for **7 days**. Do this part on the same day as the rest of the lab. In production, organizations export logs to a SIEM (like Microsoft Sentinel) to keep them longer.

### Step 5.2 — (If available) Check the failed sign-in
1. Go to **Monitoring & health → Sign-in logs**.
2. Filter by user **Chris Patel**. Look for the failed sign-in from Step 4.4. Open it — the **Basic info** tab shows the failure reason (error code **50057**, which means the account is disabled).

📸 **Screenshot `17-signin-log-failure.png`** — The failed sign-in entry for Chris with the failure reason visible.

![Signin log failure](screenshots/17-signin-log-failure.png)

*If sign-in logs aren't visible in your tenant, skip this — the screenshot from Step 4.4 already proves the lockout.*

---

## How This Works in the Real World

This lab is done manually to show the underlying steps. In a real organization, most of this is automated.

| Manual step in this lab | How it's automated in production |
|---|---|
| Creating the user by hand | **HR-driven provisioning**: the HR system (e.g., Workday, SuccessFactors) is the source of truth and automatically creates the account in Entra ID. On-prem environments sync from Active Directory using **Entra Connect**. |
| Adding users to department groups | **Dynamic groups** (Entra ID P1): a rule like `user.department -eq "Finance"` adds/removes members automatically when the Department attribute changes. Jordan would have been moved between groups just by updating their department. |
| Mover access review | **Access reviews** (Entra ID Governance): managers periodically confirm whether their team still needs access. |
| Leaver checklist | **Lifecycle Workflows** (Entra ID Governance): pre-built joiner/mover/leaver tasks that run automatically based on hire and leave dates. |
| Permanent admin roles | **Privileged Identity Management (PIM)** (Entra ID P2): admins get roles *just-in-time*, for a few hours, with approval — instead of having them all the time. |
| Leaver's mailbox and files | In Microsoft 365, the mailbox is often converted to a **shared mailbox** and OneDrive access is handed to the manager before the license is removed. |
| Keeping logs | Logs are streamed to a **SIEM** (e.g., Microsoft Sentinel) for long-term retention and alerting. |
