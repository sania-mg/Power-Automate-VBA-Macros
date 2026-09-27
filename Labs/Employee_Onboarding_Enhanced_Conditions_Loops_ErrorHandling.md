# Enhancing the Employee Onboarding Flow: Conditions, Loops & Error Handling

## Advanced Tutorial — Building on the Core Onboarding Automation

---

## 📘 Overview

This tutorial extends the base **Employee Onboarding Automation** flow (Form → Folder → Email → Teams → Planner) with three production-grade capabilities:

1. **Conditions & Switch** — branch behavior based on department, employment type, or missing data.
2. **Loops (Apply to Each)** — handle a variable number of checklist tasks and notification recipients instead of just one of each.
3. **Error Handling (Scope, Configure Run After, Terminate)** — ensure a single failed step doesn't silently break the entire onboarding process, and that failures are visible to HR instead of disappearing.

> 📌 **Prerequisite:** This tutorial assumes you've already built the base flow from the *Employee Onboarding Automation Tutorial*. Every enhancement below references the exact action names from that flow. If you're starting fresh, build the base four actions first, then return here.

### Updated Flow Architecture

```
Trigger: When a new response is submitted (Forms)
            ↓
   Get response details (Forms)
            ↓
   ┌─────────────────────────────────────────┐
   │  SCOPE: Try_Onboarding_Actions           │
   │                                          │
   │  Create new folder (SharePoint)          │
   │            ↓                             │
   │  Switch: Department                      │
   │    ├─ IT           → Create IT-specific tasks (loop)
   │    ├─ Sales        → Create Sales-specific tasks (loop)
   │    └─ Default      → Create standard tasks (loop)
   │            ↓                             │
   │  Condition: Is Manager Email provided?   │
   │    ├─ Yes → Notify manager (Teams)       │
   │    └─ No  → Notify HR fallback channel   │
   │            ↓                             │
   │  Apply to Each: Additional Recipients    │
   │    └─ Post Teams message to each         │
   │            ↓                             │
   │  Send Welcome Email (Outlook)            │
   │                                          │
   └─────────────────────────────────────────┘
            ↓ (Configure Run After: has failed)
   ┌─────────────────────────────────────────┐
   │  SCOPE: Catch_Handle_Failure             │
   │  Send failure alert email to HR          │
   │  Terminate (Status: Failed)              │
   └─────────────────────────────────────────┘
```

---

## 🎯 What You Will Add

By the end of this tutorial, your flow will be able to:

1. Route new hires through **department-specific onboarding checklists** using a **Switch** control.
2. Create **multiple Planner tasks automatically** from a list, using **Apply to Each**, instead of one hardcoded task.
3. Notify a **variable number of additional stakeholders** (e.g., onboarding buddy, IT liaison) using a second **Apply to Each** loop.
4. Gracefully handle **missing data** (e.g., no manager email) using a **Condition** with a fallback path.
5. Catch and report failures using a **Scope**-based try/catch pattern with **Configure Run After**, ending in a controlled **Terminate** action rather than a silent failure.

---

## 🧠 New Concepts Primer

| Concept | Plain-English Meaning |
|---|---|
| **Switch** | Like a Condition, but for more than two outcomes — routes the flow down one of several named paths based on a single value (e.g., Department = "IT", "Sales", or "Other"). |
| **Apply to Each** | Repeats a block of actions once for every item in a list or array — used here to handle a variable number of tasks or recipients. |
| **Scope** | A container that groups several actions together so they can be treated as one unit — primarily used to build try/catch error handling. |
| **Configure Run After** | A setting on any action or Scope that controls *when* it runs, based on whether the previous step succeeded, failed, timed out, or was skipped. This is how you build a "catch" block. |
| **Terminate** | An action that deliberately stops the flow and marks the run with a specific status (Succeeded, Failed, or Cancelled) — useful for making failures visible in run history rather than having the flow just stop ambiguously. |

---

## 🏗️ Part 1 — Wrap the Core Actions in a Try Scope

Error handling in Power Automate is built using **Scope** containers, not traditional try/catch syntax — but the concept is identical.

### Step 1.1 — Add a Scope Around Existing Actions
1. Open your existing `Employee Onboarding Automation` flow.
2. Click **+ New step** just after the "Get response details" action, and search for and add:
   > **Action:** `Scope`
3. Rename this Scope (click the three dots → **Rename**) to: `Try_Onboarding_Actions`

### Step 1.2 — Move Existing Actions Inside the Scope
1. Drag your existing four actions — **Create new folder**, **Send an email (V2)**, **Post message in a chat or channel**, **Create a task** — so they sit **inside** the `Try_Onboarding_Actions` scope box.

> 💡 **Tip:** In the designer, you can drag actions by their card header directly into a Scope container. If drag-and-drop feels fiddly, it's often easier to delete and re-add each action from inside the Scope's own "+" button, then re-map dynamic content each time.

✅ **Verification checkpoint:** All four original actions now appear visually nested inside a single `Try_Onboarding_Actions` box.

---

## 🏗️ Part 2 — Add a Switch for Department-Specific Tasks

Instead of creating one generic Planner task for every hire, route the flow based on department so IT and Sales hires get relevant checklists.

### Step 2.1 — Add a Switch Action
1. Inside the `Try_Onboarding_Actions` scope, click **+** below "Create new folder" (before the existing "Create a task" action).
2. Search for and add:
   > **Control:** `Switch`
3. In the **On** field, insert dynamic content **Department** (from "Get response details").

### Step 2.2 — Configure Each Case
1. In the first **Case** box, set the value to `IT`.
2. In the second, click **+ Add a case**, set the value to `Sales`.
3. Leave the **Default** branch to catch every other department.

✅ **Verification checkpoint:** The Switch control shows three branches: `IT`, `Sales`, and `Default`, each an empty container ready for actions.

---

## 🏗️ Part 3 — Add a Loop to Create Multiple Planner Tasks per Case

Rather than a single "Create a task" action, each Switch branch will loop through a **predefined checklist array** and create one Planner task per item.

### Step 3.1 — Build the Checklist Array with a Compose Action
1. Inside the **IT** case, click **+ Add an action** and search for and add:
   > **Action:** `Compose` (Data Operation connector)
2. Rename it: `IT_Checklist_Items`
3. In the **Inputs** field, switch to **Text mode** (click the small icon on the right of the field) and enter the following array:

   ```json
   ["Set up laptop and accounts", "Assign IT security badge", "Add to internal dev tools", "Schedule IT orientation session"]
   ```

4. Repeat this pattern inside the **Sales** case with a Compose action named `Sales_Checklist_Items`:

   ```json
   ["Set up CRM access", "Assign sales territory", "Schedule product training", "Add to sales team distribution list"]
   ```

5. And inside **Default** with `Standard_Checklist_Items`:

   ```json
   ["Set up general system access", "Schedule new hire orientation", "Assign onboarding buddy"]
   ```

> 🧠 **Why hardcode these lists with Compose?** For a beginner-to-intermediate flow, a Compose action with a static array is far easier to read and maintain than a complex expression. If checklist items change over time, a more advanced version could pull this list from a SharePoint list instead — but Compose keeps this tutorial approachable.

### Step 3.2 — Add an Apply to Each Loop in Each Case
1. Immediately after each Compose action, click **+ Add an action** and search for and add:
   > **Control:** `Apply to Each`
2. In the **Select an output from previous steps** field, insert the output of the matching Compose action (e.g., `Outputs` from `IT_Checklist_Items`).

### Step 3.3 — Add "Create a Task" Inside Each Loop
1. Inside the Apply to Each loop, click **Add an action** and search for and add:
   > **Action:** `Create a task` (Planner connector)
2. Configure:
   - **Plan Id:** `[PLANNER_PLAN_ID]`
   - **Bucket Id:** `[PLANNER_BUCKET_NAME]`
   - **Title:** Combine dynamic content for the current loop item with the employee's name:

     ```
     [Employee Full Name] — Current item
     ```

     In the designer: insert **Employee Full Name** dynamic content, type ` — `, then insert the **Current item** dynamic token that appears automatically once you're working inside an Apply to Each loop.

3. Repeat Steps 3.2–3.3 identically inside the **Sales** and **Default** cases, referencing their respective Compose outputs.

✅ **Verification checkpoint:** Testing the flow with a "Department" value of `IT` should create **four separate Planner tasks**, one per checklist item, each titled with the employee's name.

> ⚠️ **Common pitfall:** If "Apply to Each" only seems to run once instead of once-per-item, confirm the loop's input is pointing to the Compose action's **array output**, not a single string value.

---

## 🏗️ Part 4 — Add a Condition for Missing Manager Data

Not every form submission will have a Manager Email filled in correctly. Instead of letting the Teams action fail silently, branch the logic explicitly.

### Step 4.1 — Add a Condition Action
1. After the Switch control (but still inside `Try_Onboarding_Actions`), click **+ Add an action** and search for and add:
   > **Control:** `Condition`
2. In the left box, insert dynamic content **Manager Email**.
3. Set the operator to **is not equal to**.
4. In the right box, leave it blank (this checks whether the field is empty).

### Step 4.2 — Configure the "Yes" Branch (Manager Email Present)
Inside **If yes**, add the existing **Post message in a chat or channel** action (move it here if it currently sits outside the Condition), configured exactly as in the base tutorial — notifying the manager's channel or direct chat.

### Step 4.3 — Configure the "No" Branch (Manager Email Missing)
Inside **If no**, add a new **Post message in a chat or channel** action:
- **Post in:** `Channel`
- **Channel:** an HR fallback channel (e.g., `HR Operations`)
- **Message:**

  ```
  ⚠️ Manager Email was missing for [Employee Full Name]'s onboarding. Please assign a manager manually and notify them.
  ```

✅ **Verification checkpoint:** Submitting a test response with a blank Manager Email field routes the flow to the fallback HR notification instead of failing.

---

## 🏗️ Part 5 — Loop Through Additional Notification Recipients

Some onboarding processes need to notify more than one person — an onboarding buddy, a department lead, IT support, etc. Rather than adding a fixed number of individual actions, use a loop over a dynamic list.

### Step 5.1 — Update the Form to Collect Multiple Recipients
1. Return to your `Employee Onboarding Request` form in Microsoft Forms.
2. Add a new question: `Additional Notification Emails (comma-separated)` — Text type.
3. Instruct HR to enter multiple addresses separated by commas, e.g., `buddy@[YOUR_DOMAIN]; it-support@[YOUR_DOMAIN]`.

### Step 5.2 — Split the Text Field into an Array
1. Inside `Try_Onboarding_Actions`, after the Condition from Part 4, add a **Compose** action named `Split_Additional_Recipients`.
2. In **Inputs**, use the `split()` expression to convert the comma/semicolon-separated text into an array:

   ```
   split(triggerBody()?['r_additionalnotificationemails'], ';')
   ```

   > 🧠 Adjust the delimiter character (`;` in this example) to match whatever separator you instructed HR to use.

### Step 5.3 — Add an Apply to Each Loop
1. Click **+ Add an action** and add:
   > **Control:** `Apply to Each`
2. Set its input to the **Outputs** of `Split_Additional_Recipients`.

### Step 5.4 — Add the Notification Action Inside the Loop
1. Inside the loop, add:
   > **Action:** `Post message in a chat or channel` (Microsoft Teams connector)
2. Configure:
   - **Post in:** `Chat with Flow bot`
   - **Recipient:** insert the **Current item** dynamic token (the current email address in the loop)
   - **Message:**

     ```
     You've been listed as an onboarding contact for [Employee Full Name], starting [Start Date].
     ```

✅ **Verification checkpoint:** Submitting a test response with two or three additional emails results in that same number of individual Teams messages, one per recipient.

> ⚠️ **Common pitfall:** If the additional emails field is left blank, `split()` on an empty string still returns an array with one empty item, which can cause the Teams action to fail with an invalid recipient error. Add a **Condition** before Step 5.3 checking `length(outputs('Split_Additional_Recipients'))` is greater than 0, and skip the loop entirely if not.

---

## 🏗️ Part 6 — Build the Catch: Error Handling with Scope and Configure Run After

This is the most important reliability upgrade: making sure a failure inside `Try_Onboarding_Actions` is caught and reported, rather than silently stopping the flow.

### Step 6.1 — Add a Second Scope for Error Handling
1. Below the entire `Try_Onboarding_Actions` scope (outside of it, as a sibling step), click **+ New step** and add:
   > **Action:** `Scope`
2. Rename it: `Catch_Handle_Failure`

### Step 6.2 — Configure "Configure Run After"
1. Click the three dots on the `Catch_Handle_Failure` scope → **Configure run after**.
2. Uncheck **is successful**.
3. Check **has failed**, **is skipped**, and **has timed out**.
4. Click **Done**.

> 🧠 **What this does:** By default, every step only runs if the previous one succeeded. This setting flips that — `Catch_Handle_Failure` will now run **only** if `Try_Onboarding_Actions` failed, was skipped, or timed out — exactly mirroring a traditional try/catch block.

### Step 6.3 — Add a Failure Notification Email
1. Inside `Catch_Handle_Failure`, add:
   > **Action:** `Send an email (V2)` (Office 365 Outlook connector)
2. Configure:
   - **To:** `[HR_ALERT_DISTRIBUTION_EMAIL]`
   - **Subject:**

     ```
     ⚠️ Onboarding Flow Failed for [Employee Full Name]
     ```

   - **Body:** Include the employee name and a link to the run history so HR/IT can investigate:

     ```
     The onboarding automation failed for [Employee Full Name] (submitted [Start Date]).

     Please review the flow run history in Power Automate and complete any missed onboarding steps manually.
     ```

### Step 6.4 — Add a Terminate Action
1. Still inside `Catch_Handle_Failure`, add:
   > **Action:** `Terminate`
2. Configure:
   - **Status:** `Failed`
   - **Code (optional):** `OnboardingFlowError`
   - **Message (optional):** A short internal note, e.g., `One or more onboarding actions failed. See run history for details.`

> 💡 **Why add Terminate at all?** Without it, a flow that recovers inside a Catch scope shows as "Succeeded" in run history — even though something went wrong upstream. Explicitly terminating with a `Failed` status keeps your run history honest and makes failures easy to spot in monitoring dashboards (Module 8 territory).

✅ **Verification checkpoint:** Temporarily break a mapped field (e.g., point the SharePoint "Create new folder" action at a non-existent library) and run a test. Confirm:
- The `Try_Onboarding_Actions` scope shows a failure.
- The `Catch_Handle_Failure` scope runs and sends the alert email.
- The overall flow run status shows **Failed** (not ambiguously "Succeeded").

Remember to revert your deliberate break before resuming normal testing.

---

## 🔗 Updated Dynamic Content & Expressions Reference

| New Element | Field | Value / Expression |
|---|---|---|
| Switch | On | `Department` (dynamic content) |
| Compose (`IT_Checklist_Items`, etc.) | Inputs | Static JSON array, e.g. `["Set up laptop and accounts", ...]` |
| Apply to Each (checklist) | Select an output | `Outputs` of the matching Compose action |
| Create a task (inside loop) | Title | `[Employee Full Name] — Current item` |
| Condition (Manager Email) | Left side | `Manager Email` — operator **is not equal to** — right side blank |
| Compose (`Split_Additional_Recipients`) | Inputs | `split(triggerBody()?['r_additionalnotificationemails'], ';')` |
| Apply to Each (recipients) | Select an output | `Outputs` of `Split_Additional_Recipients` |
| Post message (inside recipient loop) | Recipient | `Current item` (dynamic token from the loop) |
| Catch_Handle_Failure (Scope) | Configure run after | `has failed`, `is skipped`, `has timed out` |
| Terminate | Status | `Failed` |

---

## ✅ Testing & Validation

### Expanded Test Scenario Matrix

| # | Scenario | Setup | Expected Result |
|---|---|---|---|
| 1 | Happy path — IT department, manager email present, two additional recipients | Submit form with all fields valid, Department = IT | 4 IT Planner tasks created; manager notified via Teams; 2 additional recipients notified individually; welcome email sent; flow succeeds |
| 2 | Sales department routing | Submit form with Department = Sales | 4 Sales-specific Planner tasks created (different titles than IT) |
| 3 | Unlisted department | Submit form with Department = "Marketing" (not an explicit case) | Flow routes to the **Default** branch and creates the standard 3-item checklist |
| 4 | Missing manager email | Submit form with Manager Email left blank | Condition routes to the "No" branch; HR fallback channel receives the alert instead of a failed Teams action |
| 5 | Blank additional recipients field | Submit form with the additional recipients question left empty | No recipient-loop Teams messages are sent (assuming the length-check guard from Part 5 is implemented); flow continues normally |
| 6 | Forced failure (e.g., invalid SharePoint library reference) | Temporarily misconfigure the "Create new folder" action | `Try_Onboarding_Actions` fails; `Catch_Handle_Failure` runs; HR receives a failure alert email; overall run status is **Failed** |

---

## 🧯 Troubleshooting Guide (Advanced Additions)

| Symptom | Likely Cause | Fix |
|---|---|---|
| Switch always falls into Default, even for "IT" | Case value has a typo, trailing space, or doesn't exactly match the Forms dropdown option text | Confirm the Switch case value matches the Forms choice option **character-for-character** |
| Apply to Each creates zero tasks | The Compose action's array output is empty or malformed JSON | Check the Compose action's raw input for valid JSON syntax (use double quotes, not single quotes, around each string) |
| "Create a task" inside a loop fails partway through | One checklist item's resulting Title exceeds Planner's character limit, or contains unsupported characters | Shorten checklist item text; test each array item individually if unsure |
| Catch scope never runs, even when Try clearly failed | "Configure run after" wasn't updated — it's still using the default "is successful" setting | Re-check Step 6.2; the checkbox for "is successful" must be unchecked |
| Flow shows "Succeeded" even though a Catch alert email was sent | The Terminate action is missing, or was placed inside the Try scope instead of the Catch scope | Confirm Terminate exists inside `Catch_Handle_Failure` and is configured with Status = Failed |
| `split()` expression errors out | The referenced field name doesn't match the actual internal Forms field identifier | Re-insert the field via the Dynamic Content picker first, then wrap it with `split()` afterward rather than typing the field reference by hand |
| Recipient loop fails on an empty additional-recipients submission | No guard condition was added before the loop | Implement the `length()` check described in Part 5's pitfall note |

---

## 🎓 Summary

Your onboarding flow now includes the three core flow-control patterns every intermediate Power Automate builder should know:

- ✅ **Switch** — department-specific routing instead of one-size-fits-all logic
- ✅ **Apply to Each** — handling variable-length checklists and recipient lists instead of hardcoded single items
- ✅ **Condition** — graceful handling of missing data (Manager Email) instead of a hard failure
- ✅ **Scope + Configure Run After + Terminate** — a proper try/catch pattern that makes failures visible to HR instead of silently disappearing

**Recommended next steps:**
- Move the static checklist arrays (currently in Compose actions) into a SharePoint list, so HR can update onboarding checklists without editing the flow itself.
- Add a **Do Until** loop with a short delay if any downstream system (e.g., Planner) is prone to transient throttling errors, to add basic retry resilience beyond Power Automate's built-in retry policies.
- Package this flow into a **Solution** for proper Application Lifecycle Management before deploying the enhanced version to production.

---

*End of Tutorial.*
