# 🚀 Power Automate Learning Journey

> **Learning Power Automate by building real-world projects from scratch**  
> Tenant: Microsoft 365 Developer Program  
> Author: Sharath Tatikonda  
> Started: March 2026

---

## 📚 Table of Contents

- [About This Repository](#about-this-repository)
- [Environment Setup](#environment-setup)
- [Project 1 – Leave Request Notifier](#project-1--leave-request-notifier)
- [Project 2 – Multi-Level Expense Approval (In Progress)](#project-2--multi-level-expense-approval-in-progress)
- [Key Concepts Learned](#key-concepts-learned)
- [Interview Preparation Notes](#interview-preparation-notes)
- [What's Next](#whats-next)

---

## About This Repository

This repository documents my hands-on learning journey with **Microsoft Power Automate**.  
Instead of just watching tutorials, I learn by **building real projects** that solve actual business problems.

Each project includes:
- ✅ What problem it solves
- ✅ Step-by-step setup instructions
- ✅ Flow architecture explanation
- ✅ Key concepts learned
- ✅ Screenshots of the working flow
- ✅ Troubleshooting notes

---

## Environment Setup

### Microsoft 365 Developer Tenant
| Detail | Value |
|--------|-------|
| Tenant | m365devprogramspx.onmicrosoft.com |
| SharePoint Site | Expense Management |
| Site URL | https://m365devprogramspx.sharepoint.com/sites/ExpenseManagement |

### Users Created

| Name | Email | Role in Flows |
|------|-------|---------------|
| Sharath Tatikonda | SharathTatikonda@m365devprogramspx.onmicrosoft.com | Admin / Director (Tier 3 Approver) |
| Bharath Manager | bharathmanager@m365devprogramspx.onmicrosoft.com | Line Manager (Tier 1 Approver) |
| Finance Lead | financelead@m365devprogramspx.onmicrosoft.com | Finance Approver (Tier 2) |
| Test Employee | testemployee@m365devprogramspx.onmicrosoft.com | Expense/Leave submitter |

### Manager Hierarchy (Azure AD)
```
Test Employee
    └── reports to → Bharath Manager
                         └── reports to → Sharath Tatikonda (Director)

Finance Lead ← receives Tier 2 approval emails directly
```

> 💡 **Why this matters:** Power Automate's "Get Manager (V2)" action reads this hierarchy
> directly from Azure Active Directory. Without setting up manager relationships in AAD,
> the flow cannot automatically route approvals to the correct person.

---

## Project 1 – Leave Request Notifier

### 🎯 Problem Statement
When an employee submits a leave request, the manager has no idea it happened.
They need to be notified automatically without any manual process.

### ✅ Solution
A Power Automate flow that:
1. Watches the SharePoint "Leave Requests" list
2. Automatically finds the employee's manager from Azure AD
3. Sends a formatted email to the manager instantly

### 📐 Flow Architecture
```
TRIGGER
"When an item is created" in SharePoint Leave Requests list
        ↓
ACTION 1 — Get Manager (V2)
Looks up the submitter's manager from Azure Active Directory
        ↓
ACTION 2 — Send an email (V2)
Sends notification email to manager with all leave details
```

### 🗂️ SharePoint List: "Leave Requests"

| Column Name | Type | Purpose |
|-------------|------|---------|
| Title | Single line of text | Name of the leave request |
| Reason | Single line of text | Why the employee needs leave |
| StartDate | Date and Time | When leave begins |
| EndDate | Date and Time | When leave ends |

### 🔧 Flow Configuration

**Trigger: When an item is created**
- Site Address: Expense Management site
- List Name: Leave Requests

**Action 1: Get manager (V2)**
- Connector: Office 365 Users
- User (UPN): `triggerBody()?['Author/Email']`
- Purpose: Fetches the manager's email from Azure AD automatically

**Action 2: Send an email (V2)**
- Connector: Office 365 Outlook
- To: `outputs('Get_manager_(V2)')?['body/mail']`
- Subject: `concat('Leave Request from ', triggerBody()?['Author/DisplayName'])`
- Body:
```
Hi,

{EmployeeName} has submitted a leave request.

Reason    : {Reason}
Start Date: {StartDate}
End Date  : {EndDate}

Please review and respond to the employee.

Regards,
Leave Management System
```

### 📧 Email Received by Manager
When Test Employee submits a leave request, Bharath Manager receives:
- Subject: `Leave Request from Test Employee`
- Body contains all leave details dynamically populated

### 🧪 How to Test
1. Log in as `testemployee@m365devprogramspx.onmicrosoft.com`
2. Go to SharePoint → Leave Requests list
3. Click **+ Add new item**
4. Fill in Title, Reason, StartDate, EndDate
5. Click Save
6. Check `bharathmanager@m365devprogramspx.onmicrosoft.com` inbox
7. Email should arrive within 30 seconds ✅

### ⚠️ Troubleshooting Encountered

| Error | Cause | Fix |
|-------|-------|-----|
| "No manager found" | Submitting as Director (no manager above them) | Always test as Test Employee |
| "Unauthorized" on Get Manager | Connection not authenticated | Change connection → Add new → Sign in again |
| "Invalid connection" on trigger | SharePoint connection expired | Change connection → Reconnect |
| Employee can't see SharePoint site | Site is Private | Add employee as Member in Site Permissions |

### ✅ Status: **WORKING** ✅

---

## Project 2 – Multi-Level Expense Approval (In Progress)

### 🎯 Problem Statement
Expense reports need to go through 3 levels of approval:
1. Line Manager must approve first
2. Finance team reviews second
3. Director approves only if amount > $500

Manual routing is slow and error-prone. Approvers forget. There's no audit trail.

### ✅ Solution
A Power Automate flow that:
1. Automatically routes to the correct approver at each tier
2. Sends approval emails with full expense details
3. Updates SharePoint with decisions at each stage
4. Notifies the employee of the final outcome with reasons

### 📐 Flow Architecture
```
TRIGGER: New item in "Expense Reports" SharePoint list
        ↓
Get Manager from Azure AD
        ↓
Update SharePoint item (SubmitterEmail, ManagerEmail)
        ↓
TIER 1: Send approval to Line Manager
        ├── REJECTED → Update status, Email employee with reason, END
        └── APPROVED ↓
TIER 2: Send approval to Finance Lead
        ├── REJECTED → Update status, Email employee with reason, END
        └── APPROVED ↓
                ├── Amount ≤ $500 → Status = Approved, Email employee, END
                └── Amount > $500 ↓
TIER 3: Send approval to Director
        ├── REJECTED → Update status, Email employee with reason, END
        └── APPROVED → Status = Approved, Email employee, END
```

### 🗂️ SharePoint List: "Expense Reports"

| Column | Type | Purpose |
|--------|------|---------|
| Title | Single line of text | Report name |
| SubmitterEmail | Single line of text | Auto-filled by flow |
| SubmitterName | Single line of text | Auto-filled by flow |
| Department | Choice | HR, Finance, Engineering, Sales, Other |
| ExpenseDate | Date and Time | When expense occurred |
| Amount | Currency | Expense amount in USD |
| Category | Choice | Travel, Meals, Software, Hardware, Training |
| Description | Multiple lines | Expense description |
| ManagerEmail | Single line of text | Auto-filled by flow from AAD |
| ApprovalStatus | Choice | Pending, Approved, Rejected |
| ManagerDecision | Choice | Approved, Rejected |
| FinanceDecision | Choice | Approved, Rejected |
| DirectorDecision | Choice | Approved, Rejected, N/A |
| RejectionReason | Multiple lines | Filled by approver |

### 🔧 Key Expressions Used

| Purpose | Expression |
|---------|-----------|
| Submitter email | `triggerBody()?['Author/Email']` |
| Submitter name | `triggerBody()?['Author/DisplayName']` |
| Manager email from AAD | `outputs('Get_manager_(V2)')?['body/mail']` |
| Expense amount | `triggerBody()?['Amount']` |
| Approval outcome | `outputs('Start_and_wait_for_an_approval')?['body/outcome']` |
| Rejection reason | `outputs('Start_and_wait_for_an_approval')?['body/responses'][0]?['comments']` |
| SharePoint item link | `concat('https://m365devprogramspx.sharepoint.com/sites/ExpenseManagement/Lists/ExpenseReports/DispForm.aspx?ID=',triggerBody()?['ID'])` |

### ✅ Status: **IN PROGRESS** 🔧
- [x] SharePoint list created with all columns
- [x] Users and manager hierarchy set up
- [x] Trigger configured
- [x] Get Manager action added
- [x] Update Item action added
- [x] Tier 1 Manager approval added
- [x] Tier 2 Finance approval added
- [x] Tier 3 Director approval added
- [x] All rejection email notifications added
- [x] All approval email notifications added
- [ ] Condition expression bug fix (in progress)
- [ ] End-to-end testing

---

## Key Concepts Learned

### 1. Triggers
> **"What wakes up the flow?"**

| Trigger Type | When to Use | Example |
|--------------|-------------|---------|
| Automated | When something happens in an app | New SharePoint item |
| Instant | When you manually press a button | Button in mobile app |
| Scheduled | At a specific time | Every morning at 9AM |

### 2. Actions
> **"What does the flow DO after it wakes up?"**

Actions are the tasks Power Automate performs. Examples:
- Send an email
- Update a SharePoint item
- Get data from Azure AD
- Post a Teams message

### 3. Connectors
> **"Which app does the action use?"**

Connectors are pre-built integrations to apps and services:
- **SharePoint connector** — read/write SharePoint lists
- **Office 365 Outlook connector** — send emails
- **Office 365 Users connector** — get user/manager info from Azure AD
- **Approvals connector** — send approval requests and wait for response

### 4. Dynamic Content
> **"Use real data from previous steps"**

Instead of hardcoding values, we use dynamic content to pull real data:
- `triggerBody()?['Title']` → the actual title the user typed
- `outputs('Get_manager_(V2)')?['body/mail']` → the actual manager email

### 5. Expressions (fx)
> **"Perform operations on data"**

```
concat('Hello ', triggerBody()?['Author/DisplayName'])
→ "Hello Sharath Tatikonda"

outputs('Start_and_wait_for_an_approval')?['body/outcome']
→ "Approve" or "Reject"
```

### 6. Conditions
> **"Make decisions based on data"**

```
IF outcome equals "Approve"
    THEN → proceed to next approval tier
    ELSE → send rejection email, end flow
```

### 7. Connections
> **"Permission for Power Automate to act on your behalf"**

Every connector needs a connection — like signing in to an app.
If a connection expires or breaks, the flow fails with "Unauthorized".
Fix: Change connection → Add new → Sign in again.

---

## Interview Preparation Notes

### Q: What is Power Automate?
**A:** Power Automate (formerly Microsoft Flow) is a cloud-based automation service by Microsoft. It allows you to create automated workflows between apps and services. You can automate repetitive tasks, route approvals, sync data between systems, and send notifications — all without writing traditional code.

### Q: What is the difference between a Trigger and an Action?
**A:** A **Trigger** is the event that starts the flow — like "when a new item is created in SharePoint." An **Action** is what the flow does after it starts — like "send an email" or "update a record." Every flow has exactly one trigger but can have many actions.

### Q: What are connectors in Power Automate?
**A:** Connectors are pre-built integrations to external apps and services. For example, the SharePoint connector lets you read and write SharePoint data, the Outlook connector lets you send emails, and the Office 365 Users connector lets you look up user information from Azure AD. There are 400+ connectors available.

### Q: What is Dynamic Content?
**A:** Dynamic content is data from previous steps in the flow used in later steps. For example, when a user submits a leave request, the trigger captures their name, email, and request details. In the email action, instead of hardcoding "Dear Employee," I use dynamic content to say "Dear [actual name]." This makes flows work for any user automatically.

### Q: What is the Approvals connector used for?
**A:** The Approvals connector sends an approval request to a specified person and pauses the flow until they respond. The approver receives an email (and Teams notification) with Approve/Reject buttons. When they respond, the flow continues based on their decision. This is used in multi-level approval workflows.

### Q: How do you handle errors in Power Automate?
**A:** You can configure "Run After" settings on each action to handle failures. For example, you can add an error notification action that only runs if a previous action fails. You can also use Try/Catch patterns using Scope actions. The Run History in Power Automate shows detailed logs of each step's inputs and outputs, which helps diagnose issues.

### Q: What is the difference between Standard and Premium connectors?
**A:** Standard connectors (like SharePoint, Outlook, Teams) are included in most Microsoft 365 licenses. Premium connectors (like Salesforce, SAP, HTTP) require a Power Automate Premium license. In this project, only Standard connectors were used — making it cost-effective.

### Q: Can you explain the flow you built?
**A:** I built a multi-level expense approval flow with 3 tiers:
1. When an employee submits an expense report in SharePoint, the flow triggers automatically
2. It fetches the employee's manager from Azure Active Directory using the Office 365 Users connector
3. It sends an approval email to the manager. If rejected, the employee is notified immediately
4. If approved, it goes to the Finance team for review
5. If the amount exceeds $500, it escalates to the Director for final approval
6. At every step, SharePoint is updated with the decision and the employee is notified of the outcome

---

## What's Next

### Planned Improvements to Leave Request Notifier:
- [ ] Manager can Approve/Reject directly from email (Approvals connector)
- [ ] Employee gets notified when manager decides
- [ ] HR team gets notified of all approved/rejected requests
- [ ] Status column added to SharePoint list

### Future Projects:
- [ ] Project 3: Daily Leave Summary (Scheduled flows + loops)
- [ ] Project 4: New Employee Welcome (Multiple actions)
- [ ] Project 5: Expense Report with full approval (fix and complete)

---

## Tech Stack

| Technology | Purpose |
|-----------|---------|
| Microsoft Power Automate | Flow automation |
| SharePoint Online | Data storage (lists) |
| Office 365 Outlook | Email notifications |
| Office 365 Users | Azure AD user/manager lookup |
| Microsoft Approvals | Approval request management |
| Microsoft 365 Developer Tenant | Free dev/test environment |

---

*Documentation updated: March 2026*  
*All flows built on Microsoft 365 Developer Program tenant*
