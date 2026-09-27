# Power Automate Lab Guide: Using a Pre-Built AI Builder Model

## Automating Customer Feedback Analysis with Sentiment Analysis

---

## 📘 About This Lab

AI Builder lets you add artificial intelligence to your flows **without writing any code or training any model yourself**. Microsoft has already built and trained a set of ready-to-use AI models — covering tasks like reading text sentiment, extracting key phrases, scanning receipts, and recognizing text in images.

In this lab, you'll connect one of these **pre-built models** to a Power Automate flow so that whenever new customer feedback arrives, the flow automatically detects whether the sentiment is positive, negative, neutral, or mixed — and takes action based on the result.

> 🔒 **Scope of this lab:** This guide uses **only** a pre-built AI Builder model that is ready to use out of the box. You will **not** train, retrain, or publish any custom model — that is a separate, more advanced skill outside today's lab.

---

## 🎯 Lab Objectives

By the end of this lab, you will be able to:

1. Explain what a **pre-built AI Builder model** is and how it differs from a custom-trained model.
2. Add an **AI Builder action** to a Power Automate cloud flow.
3. Configure the inputs and outputs of a pre-built model without any machine learning knowledge.
4. Use a **Condition** to route a flow based on the AI model's result.
5. Test the flow with sample data and verify the AI Builder output is accurate.

---

## 🧠 What Is a Pre-Built AI Builder Model? (Quick Primer)

| Concept | Plain-English Explanation |
|---|---|
| **Pre-built model** | An AI model Microsoft has already trained on millions of examples. You just plug in your data — no setup, no training required. |
| **Custom model** | A model *you* train yourself using your own sample data. **Not covered in this lab.** |
| **Sentiment Analysis** | A pre-built model that reads a piece of text and tells you whether it sounds Positive, Negative, Neutral, or Mixed. |
| **Confidence Score** | A number (0–1) showing how sure the AI model is about its answer. Higher = more confident. |
| **AI Builder credits** | The "fuel" pre-built AI models consume each time they run. Your organization's license determines how many you have per month. |

> 💡 **Why start with a pre-built model?** Pre-built models require zero training data and zero setup time — making them the fastest way to add real AI capability to a beginner's first flow. Once you're comfortable here, custom models (trained on your own company's data) are a natural next step.

---

## ✅ Prerequisites

Before starting, confirm you have the following. Replace bracketed placeholders with values specific to your organization or trainer's instructions.

| Requirement | Details |
|---|---|
| **License** | A Microsoft 365 or Power Automate license that includes **AI Builder** (a trial or Power Apps Developer Plan environment is sufficient for this lab) |
| **Environment** | Access to the Power Automate/Power Apps environment named **[ENVIRONMENT_NAME]** |
| **Permissions** | Ability to create flows and use AI Builder within **[ENVIRONMENT_NAME]** (ask your Environment Admin if you see a "license required" message) |
| **Data source** | A source of text data to analyze — this lab uses **[TRIGGER_SOURCE]**, which can be a SharePoint list, Microsoft Forms, an Excel table, or manual test input |
| **Region availability** | AI Builder is available only in supported regions — confirm **[ENVIRONMENT_NAME]** is in a supported region if you see an "AI Builder not available" error |

> 🔎 **No environment set up yet?** Sign up for a free trial at **[ENVIRONMENT_NAME]** via your organization's Power Platform Admin Center, or ask your instructor for a shared lab environment.

---

## 🏗️ Part 1 — Set Up Your Data Source

For this lab, we'll simulate a simple **Customer Feedback** intake using **[TRIGGER_SOURCE]**. If your trainer has already provisioned this for you, skip to Part 2.

### Step 1.1 — Create the Feedback List
1. Open **[TRIGGER_SOURCE]** (e.g., a SharePoint list or Excel table).
2. Create a table/list named `Customer Feedback` with these columns:

   | Column Name | Type |
   |---|---|
   | CustomerName | Single line of text |
   | FeedbackText | Multiple lines of text |
   | Sentiment | Single line of text *(this will be filled in automatically by the flow)* |

3. Add one test row manually, e.g.:
   - CustomerName: `Test Customer`
   - FeedbackText: `The delivery was late and the product arrived damaged.`

✅ **Expected Output:** A list/table with at least one row of sample feedback text ready to be analyzed.

---

## 🏗️ Part 2 — Create the Cloud Flow and Trigger

### Step 2.1 — Start a New Flow
1. Go to **make.powerautomate.com** and confirm you're working inside **[ENVIRONMENT_NAME]** (check the environment selector in the top-right corner).
2. Select **+ Create** → **Automated cloud flow**.
3. Name the flow: `Customer Feedback Sentiment Analysis`.

### Step 2.2 — Add the Trigger
1. In the trigger search box, search for a trigger matching your **[TRIGGER_SOURCE]** — for example:
   - SharePoint: **When an item is created**
   - Excel/OneDrive: **When a new row is added**
   - Manual testing: **Manually trigger a flow**
2. Configure the trigger to point at your `Customer Feedback` list/table from Part 1.
3. Click **Create**.

✅ **Expected Output:** A new flow is created with a single trigger step configured against **[TRIGGER_SOURCE]**.

---

## 🏗️ Part 3 — Add the Pre-Built AI Builder Sentiment Analysis Action

This is the core of the lab — adding real AI capability with a single action, no coding or training required.

### Step 3.1 — Search for the AI Builder Action
1. Click **+ New step**.
2. In the search box, type `AI Builder`.
3. From the list of pre-built model actions, select:
   > **[SELECTED_PREBUILT_MODEL]** *(for this lab, choose "Analyze positive or negative sentiment in text")*

> 🧠 **Beginner tip:** AI Builder actions are always labeled by *what they do*, not by a model version number — this makes it easy to find the right one just by describing your task (e.g., "analyze sentiment," "extract key phrases," "extract text from a receipt").

### Step 3.2 — Configure the Action's Input
1. In the **Text** field of the action, click inside it to open the **Dynamic content** panel.
2. Select the **FeedbackText** field from your trigger step.

```
Action: [SELECTED_PREBUILT_MODEL]
Input → Text: [FeedbackText] (dynamic content from trigger)
```

> 💡 **Why dynamic content?** Typing feedback text manually would only ever analyze one hardcoded sentence. Using dynamic content means the AI model analyzes *whatever* feedback comes in next — this is what makes the flow reusable and automated.

### Step 3.3 — Review the Action's Output
The pre-built Sentiment Analysis action returns a small set of fields you can use in later steps:

| Output Field | Description |
|---|---|
| `Sentiment label` | The overall result: Positive, Negative, Neutral, or Mixed |
| `Sentiment score` | A confidence score between 0 and 1 |

You do **not** need to configure anything further here — the model runs automatically the moment the flow reaches this step.

✅ **Expected Output:** The action is added to your flow with **Text** correctly mapped to dynamic content, and no red error icons on the card.

---

## 🏗️ Part 4 — Route the Flow Based on the AI Result

Now let's make the flow *do something* with the AI's answer.

### Step 4.1 — Add a Condition
1. Click **+ New step** → search `Condition` → select the **Condition** control.
2. In the left box, insert dynamic content **Sentiment label** (from the AI Builder action).
3. Set the operator to **is equal to**.
4. In the right box, type: `Negative`

### Step 4.2 — Build the "Yes" Branch (Negative Feedback Alert)
Inside **If yes**, add:
1. **Post message in a chat or channel** (Microsoft Teams)
   - Message: `⚠️ Negative feedback received from` + dynamic content **CustomerName** + `. Please review.`

### Step 4.3 — Build the "No" Branch (Update the Record)
Inside **If no**, add:
1. **Update item** (SharePoint) *or* **Update a row** (Excel) — matching your **[TRIGGER_SOURCE]**
   - Set the **Sentiment** column to dynamic content **Sentiment label**

### Step 4.4 — Also Update the Record on the "Yes" Branch
Go back to the **If yes** branch and add the same **Update item/row** action there too, so the Sentiment column is filled in regardless of outcome.

✅ **Expected Output:** Every feedback entry gets its Sentiment column filled in automatically, and negative feedback additionally triggers a Teams alert.

---

## ✅ Part 5 — Testing & Validation

### Step 5.1 — Save the Flow
Click **Save**. Fix any fields Power Automate flags as incomplete.

### Step 5.2 — Run a Manual Test
1. Click **Test** → **Manually** → **Test**.
2. Add a new row to your **[TRIGGER_SOURCE]** list/table with clearly negative text, e.g.:
   > `"This is the worst experience I have had. I want a refund."`
3. Return to Power Automate and watch the run execute in the **run history**.

### Step 5.3 — Verify the AI Builder Output
1. Click on the **[SELECTED_PREBUILT_MODEL]** action in the run history to expand it.
2. Confirm the **Sentiment label** output correctly shows `Negative` and review the **Sentiment score** — a high score (e.g., above 0.8) means the model is highly confident.

### Step 5.4 — Verify the Downstream Actions
Confirm both of the following happened:
- ✅ A message appeared in your Teams channel about the negative feedback.
- ✅ The Sentiment column in **[TRIGGER_SOURCE]** was updated to `Negative`.

### Validation Test Matrix

| # | Input Text | Expected Sentiment Label | Expected Flow Behavior |
|---|---|---|---|
| 1 | "The team was amazing and delivery was super fast!" | Positive | Sentiment column updated; no Teams alert |
| 2 | "It was okay, nothing special." | Neutral | Sentiment column updated; no Teams alert |
| 3 | "Terrible service, I'm never ordering again." | Negative | Teams alert sent; Sentiment column updated |
| 4 | "Great product but the packaging was awful." | Mixed | Sentiment column updated; no Teams alert *(since it isn't exactly "Negative")* |

> 🔎 **Try this:** Test scenario #4 above and observe that "Mixed" does **not** trigger the Teams alert, since our Condition only checks for the exact word `Negative`. This is a good discussion point for extending the lab — ask yourself how you'd also alert on "Mixed" feedback.

---

## 🧯 Troubleshooting Guide

| Symptom | Likely Cause | Fix |
|---|---|---|
| AI Builder action isn't in the search results | Your license or **[ENVIRONMENT_NAME]** doesn't include AI Builder | Confirm with your Environment Admin that AI Builder capacity is assigned |
| "Not enough AI Builder credits" error | Your organization's monthly AI Builder credit allowance has been used up | Check credit usage in the Power Platform Admin Center, or wait for the next billing cycle |
| Sentiment label output is blank | The **Text** input field wasn't mapped to dynamic content, or the source field was empty | Re-check Step 3.2 and confirm dynamic content is used, not typed text |
| Condition never matches "Negative" | Extra spaces or incorrect capitalization typed manually | Always select the operator value carefully — sentiment labels are case-sensitive |
| Flow works in test but not on real submissions | Trigger condition doesn't match how new items are actually being added to **[TRIGGER_SOURCE]** | Confirm the trigger type (e.g., "item created" vs. "item modified") matches your real data flow |
| "AI Builder not available in this region" | **[ENVIRONMENT_NAME]** is hosted in a region without AI Builder support | Use or request an environment in a supported region |

---

## 📝 Review Quiz

1. **What is the key difference between a pre-built AI Builder model and a custom model?**
2. **Which output field tells you how confident the Sentiment Analysis model is in its result?**
3. **True or False:** You need training data to use a pre-built AI Builder model.
4. **Why did we use dynamic content for the AI Builder action's Text input instead of typing sample text directly?**
5. **In the lab's Condition step, what exact value does the flow check for to trigger a Teams alert?**

<details>
<summary>📋 Click to reveal Answers</summary>

1. A pre-built model is already trained by Microsoft and ready to use immediately; a custom model must be trained by you using your own sample data (not covered in this lab).
2. The **Sentiment score** field (a value between 0 and 1).
3. **False** — pre-built models require no training data at all; that's their main advantage for beginners.
4. So the model analyzes whatever feedback text actually arrives with each new submission, making the flow reusable instead of only ever analyzing one hardcoded sentence.
5. The exact text `Negative`.

</details>

---

## 🎓 What You've Learned

You've now built a working, AI-powered business flow using only a **pre-built AI Builder model** — no training data, no machine learning expertise required. You practiced:

- ✅ Adding a pre-built AI Builder action to a cloud flow
- ✅ Mapping dynamic content into an AI model's input
- ✅ Reading and using AI-generated outputs (label + confidence score)
- ✅ Branching flow logic based on an AI result
- ✅ Testing and validating AI-driven automation with a structured test matrix

**Next steps:** Once comfortable, explore other pre-built models such as **Extract key phrases from text**, **Extract information from receipts**, or **Extract text from images (OCR)** — all follow the same pattern you just learned: pick a pre-built model, map your input, use the output.

---

*End of Lab Guide.*
