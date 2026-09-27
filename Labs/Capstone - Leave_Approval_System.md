# Microsoft Power Automate Lab Guide
## Building an Automated Employee Leave Approval System
**A Hands-On Lab Based on: "Microsoft Power Automate: From Fundamentals to Intelligent Automation"**

---

## 📘 About This Lab Guide

Welcome to your first real-world automation project! This guide walks you through building a **complete, working Employee Leave Approval System** using Microsoft Power Automate — the same kind of workflow used every day inside real companies to replace manual, email-based HR processes.

By the end of this lab, you won't just understand Power Automate theory — you will have **built, tested, and troubleshot** a multi-connector business automation from scratch.

| | |
|---|---|
| **Difficulty** | Beginner (no prior Power Automate experience required) |
| **Estimated Time** | 2.5 – 3 hours |
| **Connectors Used** | Microsoft Forms, Outlook, SharePoint (or Excel Online), Approvals, Microsoft Teams |
| **Maps to Course Module** | Module 3 (Building Flows), Module 5 (Approvals), Module 6 (M365 Connectors), Module 12 (Capstone) |

---

## 🎯 Learning Objectives

By completing this lab, you will be able to:

1. Explain the difference between a **trigger** and an **action** in a cloud flow.
2. Build an **automated cloud flow** that starts when a form is submitted.
3. Use **dynamic content** to pass data between steps.
4. Implement an **Approval** step and branch logic using a **Condition**.
5. Use an **Apply to Each** loop to process multiple items safely.
6. Integrate at least **three Microsoft 365 connectors** (Outlook, SharePoint/Excel, Teams) into a single business process.
7. Test a flow using realistic scenarios and verify expected outputs.
8. Diagnose and fix the most common beginner errors in Power Automate.

---

## 🧩 Core Concepts Primer (Read Before You Start)

If you're new to Power Automate, here are the five terms you'll see repeatedly in this lab. Keep this table handy.

| Term | Plain-English Meaning | Example in This Lab |
|---|---|---|
| **Trigger** | The event that "wakes up" the flow and starts it running. Every flow has exactly one. | "When a new response is submitted" (Microsoft Forms) |
| **Action** | A single step the flow performs after it starts. A flow is just a sequence of actions. | "Send an email" (Outlook) |
| **Dynamic Content** | Data captured in one step (like a form answer) that you can insert into a later step, instead of typing it manually. | Inserting the employee's name into the approval email |
| **Condition** | An IF/ELSE branch — the flow checks something and does one thing if true, another if false. | IF Approved → update status; ELSE → notify rejection |
| **Apply to Each (Loop)** | Repeats a block of actions once for every item in a list. Power Automate often adds this automatically when it detects multiple items (e.g., multiple form questions or list rows). | Not required for the core lab, but used in the Bonus Challenge |

> 💡 **Analogy:** Think of a flow like a recipe. The *trigger* is "when a customer places an order." Each *action* is one cooking step. *Dynamic content* is using the actual ingredient the customer picked instead of a generic placeholder. A *condition* is "if the customer wants it spicy, add chili; otherwise don't."

---

## 🏢 Business Scenario

> Riya, an engineer in the IT team, has been asked to automate the company's paper-based leave request process. Currently, employees email their manager, wait for a reply, and someone manually updates a spreadsheet. This is slow and error-prone.
>
> **Your task:** Build a flow where an employee submits a leave request via a form. The request is automatically routed to their manager for approval. Once a decision is made, the tracker is updated, and the employee is notified by email — with the HR team also alerted in a Teams channel.

### Process Flow Diagram

```
Microsoft Forms (Leave Request Submitted)
            ↓
   Start and Wait for an Approval (Manager)
            ↓
        Condition: Approved?
        ↓                    ↓
   [YES branch]          [NO branch]
   Update SharePoint      Update SharePoint
   list → "Approved"      list → "Rejected"
            ↓                    ↓
   Send Outlook email    Send Outlook email
   to employee            to employee
            ↓                    ↓
        Post message in Microsoft Teams (HR channel)
```

---

## 🛠️ Prerequisite Setup (Do This First)

Complete Step 0.1 only if it is your first time using Power Automate or if you haven't set up the required Microsoft 365 services yet.

### Step 0.1 — Confirm Your Access
You need:
- A Microsoft 365 **work or school account** with access to Power Automate (a trial developer environment works fine if your organization hasn't assigned licenses yet).
- Access to **Microsoft Forms**, **SharePoint Online** (or Excel Online on OneDrive if SharePoint is unavailable), **Outlook**, and **Microsoft Teams**.

> 🔎 **No enterprise environment yet?** Sign up for a free [Power Apps Developer Plan](https://powerapps.microsoft.com/en-us/developerplan/) using your college or personal Microsoft account. This gives you a personal sandbox environment safe for experimentation.

### Step 0.2 — Create the Microsoft Form
1. Go to **forms.office.com** and sign in.
2. Click **New Form**. Title it: `Employee Leave Request`.
3. Add the following questions:
   | Question | Type |
   |---|---|
   | Employee Name | Text |
   | Employee Email | Text |
   | Leave Start Date | Date |
   | Leave End Date | Date |
   | Reason for Leave | Text (long answer) |
4. Click **Preview**, submit one dummy test response (e.g., "Test User"), and note the form's title exactly — you'll need it in Step 1.

✅ **Expected Output:** A published form with 5 questions and at least one test response visible under the **Responses** tab.

### Step 0.3 — Create the SharePoint Tracking List
1. Go to your SharePoint team site (or create one via **sharepoint.com** → **+ Create site**).
2. Select **New → List → Blank list**. Name it `Leave Requests Tracker`.
3. Add these columns:
   | Column Name | Type |
   |---|---|
   | EmployeeName | Single line of text |
   | EmployeeEmail | Single line of text |
   | StartDate | Date |
   | EndDate | Date |
   | Reason | Multiple lines of text |
   | Status | Choice (values: `Pending`, `Approved`, `Rejected`) |

> ⚠️ **No SharePoint access?** Substitute an Excel Online workbook saved in OneDrive for Business, formatted as a **Table** with the same column headers. The lab steps below note the equivalent Excel actions in brackets.

✅ **Expected Output:** An empty SharePoint list (or Excel table) with the six columns above, ready to receive rows.

### Step 0.4 — Identify a "Manager" and a Teams Channel
- Pick a colleague, classmate, or your trainer's account to act as the **Manager** approver (you'll need their email).
- Create or identify a Microsoft Teams channel called `HR Notifications` where status updates will be posted.

---

## 🏗️ Part 1 — Build the Flow Skeleton (Trigger + First Action)

### Step 1.1 — Create a New Automated Cloud Flow
1. Go to **make.powerautomate.com**.
2. Select **+ Create** → **Automated cloud flow**.
3. Name it: `Employee Leave Approval Flow`.
4. In the trigger search box, type `Forms` and select **When a new response is submitted**.
5. Click **Create**.

### Step 1.2 — Configure the Trigger
1. In the trigger card, set **Form Id** to `Employee Leave Request` (the form you built in Step 0.2).
2. This trigger alone doesn't give you the actual answers yet — it only tells the flow *that* a response arrived.

### Step 1.3 — Retrieve the Response Details
1. Click **+ New step**.
2. Search for and add **Get response details** (Forms connector).
3. Set **Form Id** to the same form.
4. Set **Response Id** to the dynamic content token `Response Id` from the trigger step.

> 🧠 **Why two Forms actions?** This trips up almost every beginner. The trigger only detects *that* someone submitted a response; **Get response details** is a separate action that actually *fetches* the answers (name, dates, reason) so you can use them later. Skipping this step is the #1 cause of "blank data" errors in Forms-based flows.

✅ **Expected Output:** Running a test now (see Part 5) should show the trigger firing and the "Get response details" action returning your test submission's data in its output.

---

## 🏗️ Part 2 — Add the Manager Approval Step

### Step 2.1 — Add the Approval Action
1. Click **+ New step**.
2. Search for `Approvals` and select **Start and wait for an approval**.
3. Set:
   - **Approval type:** `Approve/Reject – First to respond`
   - **Title:** `Leave Request from` + (insert dynamic content: **Employee Name**)
   - **Assigned to:** the manager's email you identified in Step 0.4
   - **Details:** Combine dynamic content for **Reason**, **Leave Start Date**, and **Leave End Date** into one readable message, e.g.:
     ```
     Employee: [Employee Name]
     From: [Leave Start Date]  To: [Leave End Date]
     Reason: [Reason for Leave]
     ```

> 💡 **Tip on Dynamic Content:** When you click inside a field, a **Dynamic content** panel opens showing all data available from previous steps. Always pick fields from this panel rather than typing them manually — this is what makes the flow "smart" and reusable for every future submission, not just your test one.

✅ **Expected Output:** When tested, the assigned manager receives an email/Teams approval card with the leave details clearly visible, and two buttons: **Approve** / **Reject**.

---

## 🏗️ Part 3 — Branch the Flow with a Condition

### Step 3.1 — Add a Condition Action
1. Click **+ New step** → search `Condition` → select the **Condition** control.
2. In the left box, insert dynamic content **Outcome** (from the Approval step).
3. Set the operator to **is equal to**.
4. In the right box, type: `Approve`

> 🧠 **Beginner tip:** Power Automate's Approval action always returns the word `Approve` or `Reject` (capital A/R, no punctuation) in the **Outcome** field — not "Approved"/"Yes"/"True". Typing the wrong exact word here is one of the most common reasons a condition silently fails.

### Step 3.2 — Build the "Yes" (Approved) Branch
Inside the **If yes** box, add these two actions in order:

1. **Update item** (SharePoint) *[or Update a row (Excel) if using Excel]*
   - Site/List: `Leave Requests Tracker`
   - Id: dynamic content **ID** from the "Create item" step *(see note below — you'll add this in Part 4)*
   - Status: type `Approved`

2. **Send an email (V2)** (Outlook)
   - To: dynamic content **Employee Email**
   - Subject: `Your Leave Request has been Approved`
   - Body: A short, friendly confirmation message using dynamic content for the dates.

### Step 3.3 — Build the "No" (Rejected) Branch
Inside the **If no** box, mirror the same two actions, but:
   - Status: `Rejected`
   - Email subject: `Your Leave Request has been Reviewed`
   - Body: Politely explain the request was not approved, and include the **Comments** dynamic content from the Approval step (managers can leave a reason).

✅ **Expected Output:** Testing with an "Approve" response should visibly execute only the left branch; testing with "Reject" should execute only the right branch.

---

## 🏗️ Part 4 — Track the Request in SharePoint (Retro-Fit Step)

You referenced a SharePoint "Id" in Part 3 that doesn't exist yet — this is intentional, to teach you a **real debugging habit**: building flows is rarely linear. Now let's add the missing piece.

### Step 4.1 — Create the Tracker Row *Before* the Approval Step
1. Go back and click **between** the "Get response details" action (Part 1) and the "Start and wait for an approval" action (Part 2).
2. Click the **+** icon that appears → **Add an action**.
3. Search for **Create item** (SharePoint) *[or Add a row (Excel)]*.
4. Map each SharePoint column to its matching dynamic content field from the form response:
   - EmployeeName ← Employee Name
   - EmployeeEmail ← Employee Email
   - StartDate ← Leave Start Date
   - EndDate ← Leave End Date
   - Reason ← Reason for Leave
   - Status ← type the literal text `Pending`

### Step 4.2 — Fix the Broken References in Part 3
1. Go to both **Update item** actions from Part 3.
2. In the **Id** field, delete any leftover text and insert dynamic content **ID** — it will now correctly appear, sourced from the **Create item** action you just added.

✅ **Expected Output:** After this fix, a new row appears in your SharePoint list the moment a form is submitted, with Status = "Pending" — later updated to "Approved" or "Rejected" once the manager responds.

---

## 🏗️ Part 5 — Notify the Team via Microsoft Teams

### Step 5.1 — Add a Teams Notification (Outside Both Branches)
1. Scroll below the entire **Condition** block (not inside either branch) and click **+ New step**.
2. Search for **Post message in a chat or channel** (Microsoft Teams).
3. Configure:
   - Post as: `Flow bot`
   - Post in: `Channel`
   - Team: your team
   - Channel: `HR Notifications`
   - Message: `Leave request for` + Employee Name + `has been processed. Status:` + dynamic content **Outcome**

> 🧠 **Why place this step *after* the condition, not inside it?** Both branches need to trigger this same notification. Placing it after the Condition block means it runs regardless of which branch executed — a cleaner design than duplicating the same action twice inside both branches.

✅ **Expected Output:** After every test run (approved or rejected), a message appears in the `HR Notifications` Teams channel summarizing the outcome.

---

## ✅ Part 6 — Save, Test, and Verify

### Step 6.1 — Save the Flow
Click **Save** in the top-right corner. Power Automate will flag any missing required fields — fix them before proceeding.

### Step 6.2 — Run a Manual Test
1. Click **Test** (top right) → **Manually** → **Test**.
2. Go to your Microsoft Form and submit a real response.
3. Return to Power Automate and watch the flow run in real time on the **run history / flow checker** screen.

### Step 6.3 — Approve or Reject as the "Manager"
Check the manager's email or Teams **Approvals** app, and click **Approve** on the test request.

### Test Scenario Checklist

| # | Scenario | Steps | Expected Result |
|---|---|---|---|
| 1 | Happy path – Approved | Submit form → Manager clicks Approve | SharePoint row = "Approved"; employee receives approval email; Teams message posted |
| 2 | Rejected path | Submit form → Manager clicks Reject with a comment | SharePoint row = "Rejected"; employee's email includes the manager's comment; Teams message posted |
| 3 | Missing/invalid email | Submit form with a malformed employee email | Flow run fails at the "Send an email" step — use this to practice reading error messages |
| 4 | Duplicate submission | Submit the same form twice | Two independent SharePoint rows are created and processed separately (each run is isolated) |

---

## 🧯 Troubleshooting Guide (Common Beginner Errors)

| Symptom | Likely Cause | Fix |
|---|---|---|
| Flow run shows a **red X** immediately after the trigger | You forgot the **Get response details** action, so later steps reference empty data | Re-check Part 1, Step 1.3 |
| Condition always goes to the "No" branch, even for approvals | Typed `"Approved"` instead of the exact value `Approve` | Re-check Part 3, Step 3.1's spelling guidance |
| **Update item** action fails with "Item not found" | The **Id** field is empty or pointing to the wrong dynamic content source | Confirm it points to **ID** from **Create item**, not from the trigger |
| Manager never receives the approval request | Wrong or inactive account entered in "Assigned to," or the account has no license | Verify the exact email address; ask the manager to check both email and the Teams Approvals app |
| Flow saves but "Dynamic content" panel is empty in a later step | You're trying to reference a field from an action that comes *after* the current one in the flow order | Dynamic content only flows "downward" — reorder your actions if needed |
| Teams message step fails with a permissions error | The Flow doesn't have access to post in that channel/team | Confirm you're a member of the team, or use "Post message in a chat" to yourself for testing |
| Flow works once but fails on the second test | Manually deleted the test SharePoint row while the flow's connection reference expected it to still exist | Never manually delete rows the flow is actively referencing mid-run; let the flow manage its own data |

---

## 🌟 Bonus Challenge (Optional, Advanced)

Ready to push further? Try extending your flow with an **Apply to Each** loop:

> **Scenario:** HR wants a *daily digest* email listing all leave requests still marked "Pending" for more than 2 days.
>
> **Hint:** Use a **Scheduled cloud flow** with a **Get items** (SharePoint) action filtered by `Status eq 'Pending'`, then an **Apply to Each** loop around a "Compose" action to build a summary list, finishing with a single **Send an email** action containing the full digest.

This mirrors the "Daily Task Reminder System" pattern used later in the full course (Module 6) and is excellent practice for loops and filter queries.

---

## 📝 Review Quiz

Test your understanding before moving on. Answers are provided at the end — try to answer first without peeking!

1. **What is the difference between a trigger and an action?**
2. **In this lab, why did we need both "When a new response is submitted" AND "Get response details" as separate steps?**
3. **What exact text value does the Outcome field return when a manager approves a request?**
4. **Why was the Teams notification step placed *after* the Condition block instead of inside each branch?**
5. **True or False:** Dynamic content from a later step in the flow can be used in an earlier step.
6. **Name the three Microsoft 365 connectors used in this lab's core build (not counting the Bonus Challenge).**
7. **What would happen if the "Id" field in the Update item action was left blank?**
8. **What's the purpose of an Apply to Each loop, and where would you use one in a scaled-up version of this flow?**

<details>
<summary>📋 Click to reveal Answers</summary>

1. A trigger is the single event that starts a flow (there's always exactly one). An action is any step the flow performs afterward; a flow can have many actions.
2. The trigger only detects that a response was submitted — it does not carry the actual answers. "Get response details" retrieves the real data (name, dates, reason) needed by later steps.
3. `Approve` (exact capitalization, no extra punctuation).
4. Because both the Approved and Rejected branches need the same notification to run; placing it after the Condition avoids duplicating the action in two places.
5. **False** — dynamic content only flows forward (downward) through the flow; a step can only use data from steps that ran before it.
6. Microsoft Forms, Outlook, and SharePoint (Teams and Approvals are also used, but the "core three" from the constraint are Forms/Outlook/SharePoint working together with Approvals and Teams as additional integrations).
7. The **Update item** action would fail, because Power Automate wouldn't know which SharePoint row to update.
8. An Apply to Each loop repeats a block of actions once per item in a list — useful when processing multiple pending requests at once, such as in the Bonus Challenge's daily digest flow.

</details>

---

## 🎓 Course Outcome Alignment

Completing this lab directly builds the following skills from the full course:

- ✅ Building cloud flows from scratch (Module 3)
- ✅ Creating approval workflows (Module 5)
- ✅ Integrating multiple Microsoft 365 connectors (Module 6)
- ✅ Applying conditions and error-handling fundamentals (Module 4)
- ✅ Practicing the exact "Employee Leave Management System" capstone use case (Module 12)

**Next steps in your learning path:** Once comfortable with this lab, revisit it and add error handling with a **Scope + Configure Run After** (Module 4) around the email steps, so the flow gracefully handles a case where an employee's email address is invalid.

---

*End of Lab Guide.*
