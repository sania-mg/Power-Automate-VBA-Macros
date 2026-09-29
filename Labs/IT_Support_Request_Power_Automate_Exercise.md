# Exercise: Create an IT Support Request Automation

## Objective

Create a simple Power Automate cloud flow that connects **Microsoft Forms, SharePoint, Conditions, and Microsoft Teams**.

The automation will:

1. Capture an IT support request submitted through Microsoft Forms.
2. Save the request in a SharePoint List.
3. Check whether the request has **High** priority.
4. Send an appropriate notification to Microsoft Teams.

---

## Business Scenario

Employees use a Microsoft Form to submit IT support requests.

When a request is submitted:

- The request is stored automatically in a SharePoint List.
- If the priority is **High**, an urgent Teams notification is posted.
- If the priority is **Medium** or **Low**, a normal Teams notification is posted.

### Automation

**Microsoft Forms → Get Response Details → SharePoint → Condition → Microsoft Teams**

---


# Part 1: Create the SharePoint List from Form

Create a SharePoint List from Form named:

**IT Support Requests**

Add the following questions:

| Question | Type | Options |
|---|---|---|
| Employee Name | Text | — |
| Employee Email | Text | — |
| Issue Category | Choice | Hardware, Software, Network, Access Request |
| Issue Description | Long text | — |
| Priority | Choice | Low, Medium, High |

Save the form.


# Part 2: Add new column to SharePoint List

Once the SharePoint List is created, add another column for Status fo type choice with the following options:

- New
- In Progress
- Resolved

---

# Part 3: Create the Power Automate Flow

Go to Power Automate and create an:

**Automated cloud flow**

Name the flow:

**IT Support Request Automation**

---

## Step 1: Add the Trigger

Select:

**Microsoft Forms – When a new response is submitted**

Configure:

- **Form Id:** IT Support Request

This trigger starts the flow whenever someone submits the form.

---

## Step 2: Get the Form Response

Add the action:

**Microsoft Forms – Get response details**

Configure:

| Field | Value |
|---|---|
| Form Id | IT Support Request |
| Response Id | Response Id |

Use the **Response Id** dynamic value from the trigger.

---

# Part 4: Create the SharePoint Item

Add the action:

**SharePoint – Create item**

Configure:

- **Site Address:** Select your SharePoint site.
- **List Name:** IT Support Requests

Map the Microsoft Forms responses to the SharePoint columns.

| SharePoint Column | Form Response |
|---|---|
| Title | Employee Name |
| EmployeeName | Employee Name |
| EmployeeEmail | Employee Email |
| IssueCategory | Issue Category |
| IssueDescription | Issue Description |
| Priority | Priority |
| Status | New |

The flow will now save every submitted request in SharePoint.

---

# Part 5: Add a Condition

Add:

**Control → Condition**

Configure the condition as:

**Priority is equal to High**

Conceptually:

```text
Priority
   ↓
is equal to
   ↓
High
```

The Condition creates two branches:

- **Yes**
- **No**

---

# Part 6: Configure the YES Branch

The **YES** branch means that the request has **High** priority.

Add:

**Microsoft Teams – Post a message in a chat or channel**

Select the appropriate:

- Team
- Channel

Use a message similar to:

```text
🚨 HIGH PRIORITY IT SUPPORT REQUEST

Employee: [Employee Name]
Category: [Issue Category]
Description: [Issue Description]
Priority: High

Please review this request urgently.
```

Use dynamic content from the Forms response wherever appropriate.

---

# Part 7: Configure the NO Branch

The **NO** branch means that the priority is either:

- Medium
- Low

Add:

**Microsoft Teams – Post a message in a chat or channel**

Use a message similar to:

```text
📢 New IT Support Request

Employee: [Employee Name]
Category: [Issue Category]
Priority: [Priority]

Description:
[Issue Description]

A new support request has been added to the SharePoint List.
```

Again, insert the appropriate dynamic content from the Forms response.

---

# Final Flow Structure

The completed flow should look conceptually like this:

```text
┌────────────────────────────────────┐
│ Microsoft Forms                    │
│ When a new response is submitted   │
└──────────────────┬─────────────────┘
                   ↓
┌────────────────────────────────────┐
│ Microsoft Forms                    │
│ Get response details               │
└──────────────────┬─────────────────┘
                   ↓
┌────────────────────────────────────┐
│ SharePoint                         │
│ Create item                        │
│ IT Support Requests                │
└──────────────────┬─────────────────┘
                   ↓
             ┌─────────────┐
             │  Condition  │
             │ Priority =  │
             │    High?    │
             └──────┬──────┘
                    │
             ┌──────┴──────┐
             │             │
            YES            NO
             │             │
             ↓             ↓
     ┌──────────────┐ ┌──────────────┐
     │ Microsoft    │ │ Microsoft    │
     │ Teams        │ │ Teams        │
     │ Urgent Alert │ │ Normal Alert │
     └──────────────┘ └──────────────┘
```

---

# Part 8: Test the Flow

Select **Save** and then test the flow.

Submit the Microsoft Form with:

### Test 1 – High Priority

Example:

```text
Employee Name: John Smith
Employee Email: john.smith@contoso.com
Issue Category: Network
Issue Description: Unable to connect to the corporate network.
Priority: High
```

Expected result:

1. The response is captured by Power Automate.
2. A new item is created in the SharePoint List.
3. The Condition evaluates to **Yes**.
4. An urgent notification is posted in Teams.

---

### Test 2 – Medium Priority

Submit another response with:

```text
Priority: Medium
```

Expected result:

1. A SharePoint item is created.
2. The Condition evaluates to **No**.
3. A normal Teams notification is posted.

---

### Test 3 – Low Priority

Submit another response with:

```text
Priority: Low
```

Expected result:

1. A SharePoint item is created.
2. The Condition evaluates to **No**.
3. A normal Teams notification is posted.

---

# Expected Outcome

After completing the exercise, participants should have a working automation that:

- Captures information using **Microsoft Forms**.
- Retrieves form responses using **Get response details**.
- Creates records in a **SharePoint List**.
- Uses a **Condition** to evaluate priority.
- Sends notifications through **Microsoft Teams**.
- Uses **Dynamic Content** to transfer information between actions.

---

# Skills Practiced

By completing this exercise, participants practice:

1. Creating an automated cloud flow.
2. Working with Microsoft Forms triggers and actions.
3. Using dynamic content.
4. Creating SharePoint List items.
5. Configuring Conditions.
6. Creating Yes/No branches.
7. Posting messages to Microsoft Teams.
8. Testing and troubleshooting a Power Automate flow.

---

# Challenge Exercise

Extend the flow with the following requirements:

### Challenge 1 – Update Status

After creating the SharePoint item, ensure that the **Status** is set to:

**New**

### Challenge 2 – Different Teams Messages

Modify the flow so that:

- High → 🚨 Urgent notification
- Medium → ⚠️ Attention required
- Low → ℹ️ Normal notification

### Challenge 3 – Include the SharePoint Item ID

Add the SharePoint **ID** to the Teams notification so that the support team can identify the request.

Example:

```text
Ticket ID: 105
Employee: John Smith
Priority: High
Issue: Unable to connect to the corporate network.
```

---

# Flow Summary

| Component | Purpose |
|---|---|
| Microsoft Forms | Capture the support request |
| Get response details | Retrieve submitted form information |
| SharePoint – Create item | Store the request |
| Condition | Check request priority |
| Microsoft Teams | Notify the support team |

## End Result

**Employee submits Form → Request is stored in SharePoint → Priority is evaluated → Teams notification is sent**
