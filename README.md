# Leave Request Automation System
### Built on Microsoft 365 · Power Automate · SharePoint Online

> **When an employee submits a leave request, the line manager is instantly notified by email — automatically, no manual steps.**

---

## ⚡ What It Does

| Without Automation | With This System |
|---|---|
| Employee emails manager manually | Employee fills SharePoint form |
| Manager may miss or forget | Manager gets instant email notification |
| No record of requests | All requests stored in SharePoint list |
| No audit trail | Every submission timestamped and tracked |

---

## 🏗️ Architecture

```
Employee submits leave request
         │
         ▼
SharePoint List  ◄─── Trigger (Power Automate watches 24/7)
         │
         ▼
Get Manager from Azure AD  ◄─── Looks up reporting manager automatically
         │
         ▼
Send Email to Manager  ◄─── Notification with full leave details
```

---

## 🖥️ System Setup

### Microsoft 365 Tenant
Built on a Microsoft 365 Developer Program tenant with real users, real emails and real SharePoint.

### Users & Roles

| User | Role | Reports To |
|------|------|-----------|
| Test Employee | Leave submitter | Bharath Manager |
| Bharath Manager | Line Manager (receives notifications) | Sharath Tatikonda |
| Sharath Tatikonda | Director / Admin | — |
| Finance Lead | Finance team | — |

![M365 Admin Center — Users](screenshots/admin-users.png)

### Manager Hierarchy (Azure Active Directory)
```
Test Employee  →  Bharath Manager  →  Sharath Tatikonda (Director)
```
> Manager relationships configured in Azure AD (Entra). Power Automate reads this automatically — no hardcoded emails in the flow.

![Azure AD Manager Hierarchy](screenshots/azure-ad-hierarchy.png)

---

## 📋 SharePoint List — "Leave Requests"

Stores all employee leave submissions. Acts as the data layer for the entire system.

| Column | Type | Purpose |
|--------|------|---------|
| Title | Text | Leave request name |
| Reason | Text | Why the employee needs leave |
| StartDate | Date | Leave start date |
| EndDate | Date | Leave end date |

![SharePoint Leave Requests List — Live Data](screenshots/sharepoint-list.png)

---

## ⚙️ Power Automate Flow

3 steps. Fully automated. Runs in under 1 second.

### Step 1 — Trigger: When an item is created
Watches the SharePoint "Leave Requests" list 24/7. Fires the moment a new item is added.

### Step 2 — Get Manager (V2)
Calls the Office 365 Users connector to look up the submitter's manager from Azure Active Directory. No hardcoded emails — works for any employee automatically.

```
Input:  triggerBody()?['Author/Email']
Output: outputs('Get_manager_(V2)')?['body/mail']
```

### Step 3 — Send an Email (V2)
Sends a notification to the manager with all leave details pulled dynamically from the SharePoint submission.

![Power Automate Flow — Successful Run](screenshots/flow-success.png)

---

## 📧 Evidence — Email Delivered to Manager

Manager receives an instant email the moment a leave request is submitted.

![Email Received by Manager](screenshots/manager-email.png)

> ⚠️ **Known issue (fixing in Phase 2):** Email body currently renders the concat expression as text instead of formatted output. The notification is delivered correctly — formatting fix in progress.

---

## ✅ Proof It Works

- 4 leave requests submitted and tracked in SharePoint ✅
- Flow triggered on every submission ✅
- Manager email delivered within 1 second ✅
- Manager resolved automatically from Azure AD — no hardcoding ✅

---

## 🔧 Tech Stack

| Tool | Purpose |
|------|---------|
| Microsoft Power Automate | Workflow automation engine |
| SharePoint Online | Leave request data store |
| Office 365 Outlook | Email delivery |
| Office 365 Users connector | Azure AD manager lookup |
| Microsoft 365 Admin Center | User & licence management |

---

## 🗺️ Roadmap

This is **Phase 1** of a full HR Automation System being built progressively.

- [x] **Phase 1 — DONE** · Employee submits → Manager notified instantly
- [ ] **Phase 2** · Manager approves/rejects → Employee notified of decision
- [ ] **Phase 3** · HR team notified of all approved/rejected requests
- [ ] **Phase 4** · SharePoint dashboard showing all pending/approved/rejected requests
- [ ] **Phase 5** · Automated reminders if manager hasn't responded in 48 hours

---

## 🗂️ Repository Structure

```
leave-request-automation/
│
├── screenshots/
│   ├── sharepoint-list.png        # SharePoint list with live data
│   ├── admin-users.png            # M365 users setup
│   ├── azure-ad-hierarchy.png     # Manager hierarchy in AAD
│   ├── flow-success.png           # Power Automate successful run
│   └── manager-email.png          # Email delivered to manager
│
└── README.md
```

---

*Built by Sharath Kumar · March 2026 · [github.com/sharathkumar1996](https://github.com/sharathkumar1996)*
