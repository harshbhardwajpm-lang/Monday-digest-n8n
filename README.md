# Monday-digest-n8n
n8n workflow for weekly enquiries and Leh weather digest

**Farah asks:** Every Monday at 8 AM, one email: what came in last week, and whether the Leh weather is about to turn.

## Build This Workflow

`Schedule → HTTP → Sheets → Filter → Aggregate → AI → Gmail`

- **Given to you:** Schedule trigger and weather HTTP request.
- **You add these five:** Google Sheets, Filter, Aggregate, Basic LLM Chain, and Gmail.

## What You Get

- **Monday Digest Starter (n8n).json:** The trigger and weather call, already working.
- **Wanderline Enquiries:** A Google Sheet of enquiries, old and new.

## Done Means

One email in your inbox covering **only the last 7 days**, containing:
- A count by enquiry type, with spam excluded.
- One line per lead.
- A line about the Leh weather.

**Last week only. One email.**

---

# 1. Set Up

*Ten minutes before you add a single node.*

## Step 1: Copy the Sheet

Open **Wanderline Enquiries** in the Class 11 Drive folder, then select **File → Make a copy**.

Work on your own copy of the spreadsheet.

## Step 2: Import the Starter Workflow

1. Download `Monday Digest Starter (n8n).json`.
2. In n8n, create a new workflow.
3. Open the three-dot (`...`) menu.
4. Choose **Import from File**.
5. Select the downloaded JSON file.

## Step 3: Check the Timezone

Open the workflow menu and navigate to **Settings → Timezone**.

Make sure the timezone is set to:

`Asia/Kolkata`

Otherwise, 8 AM may not be your local 8 AM.

## Step 4: Run the Weather Request

1. Open the **HTTP Request** node.
2. Click **Execute step**.
3. Find today's Leh temperature in the output.
4. Check the temperature and understand what it tells you about the weather.

**Run what you have before you add more.**

---

# 2. Hints 1 & 2: Read and Filter the Sheet

*Try first. Read a hint only when you are stuck.*

## Step 1: Read the Sheet

Add a **Google Sheets → Get row(s) in sheet** node after the HTTP Request node.

1. Connect your Google account.
2. Select your copy of the spreadsheet.
3. Choose the `Enquiries` tab.
4. Execute the node.
5. Check how many items came out.
6. Determine whether all the items are from last week.

## Step 2: Keep Only the Last 7 Days

Add a **Filter** node.

Configure the condition:

- **Condition type:** Date & Time
- **Operation:** Is After
- **Convert types where required:** Turn on if needed

Use these expressions:

**Left:**

```javascript
{{ $json.Date }}
```

**Right:**

```javascript
{{ $now.minus({days: 7}) }}
```

Count last week's rows in the spreadsheet yourself. Does the number of items kept by the Filter node match?

**Check the count, not just the colour.**

---

# 3. Hints 3 & 4: Aggregate and Write the Digest

*Many rows in, one email out. Then two sources in one prompt.*

## Step 3: Many Items Is a Problem

Every node runs once per item. Ten items entering Gmail can mean ten emails.

Add an **Aggregate** node after the Filter node.

Configure it as:

`All Item Data (Into a Single List)`

Execute the node and verify that many input items become one output item containing a list of the enquiry data.

This step ensures that the workflow can produce one digest rather than one email per enquiry.

## Step 4: Write the Digest

Add a **Basic LLM Chain** node with a chat model connected underneath it.

Configure the prompt to request:

1. A count by enquiry type, excluding spam.
2. One line per lead.
3. A weather line that starts with `WEATHER WARNING:` if rain or a sharp temperature drop is coming.

Use the aggregated enquiry data and weather data in the prompt.

**Enquiries:**

```javascript
{{ JSON.stringify($json.data) }}
```

**Current Leh temperature:**

```javascript
{{ $('HTTP Request').item.json.current.temperature_2m }} C
```

The weather is a reach-back: retrieve it from the HTTP Request node rather than typing it manually.
