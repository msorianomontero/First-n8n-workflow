# Build an n8n AI Agent live

# Live Workshop: Build an AI Website Monitoring Agent

## Workshop Overview

In this live workshop, we'll build an AI-powered website monitoring system that automatically tracks changes across multiple websites and provides intelligent analysis of what changed. You'll leave with a fully functional automation that runs 3x per week!

### What You'll Get

- Automated website monitoring system
- AI-powered change detection and analysis
- Organized tracking in Airtable
- Scheduled monitoring (Monday, Wednesday, Friday)

---

## Workshop Steps (We'll Do Together)

### Step 1: Set Up Your Airtable Base

**Duplicate the Airtable Base:**

- **Duplicate our template base**: [**https://airtable.com/app6VjWeXicjLds2Y/shrTEkn6U9GoNdEiH**](https://airtable.com/app6VjWeXicjLds2Y/shrTEkn6U9GoNdEiH)
- Click **Use template** and select your workspace
- The base should already contain our test website: `https://9x.notion.site/Automation-Inc-256edcede7df80c89a4ec076d82cb47c`
- **Verify the Monitoring status is set to "Active"** (if not, change it to Active)

---

### Step 2: Import Workflow and Configure Airtable Node

**Import the Starter Workflow:**

- **Import the provided n8n workflow file** into your n8n instance

<aside>
⬇️

[Website Monitoring Agent - Starter Workflow.json](attachment:beb0d389-ea41-4a4c-9ece-0cf0a9463772:Website_Monitoring_Agent_-_Starter_Workflow.json)

</aside>

- Go to **Workflows** → **Click the “…” in the top right corner** → **Import from File**
- Select the **Website Monitoring Agent - Starter Worfklow.json** file
- Click **Import**
- You should see: Manual Trigger & Schedule Trigger → Airtable Search

**Get Your Airtable API Key:**

- Go to [airtable.com/create/tokens](https://airtable.com/create/tokens)
- Create token with `data.records:read`, `data.records:write`, `schema.bases:read`
- Select your Website Monitoring base
- Copy and save your token

**Configure the Airtable Node in n8n:**

- Click on **Search records** node
- Select ‘Credential to connect with’ and select ‘**Create new credential’**
- **Give it a Connection Name**: "Website Monitoring Airtable"
- **Access Token**: Paste your API token
- Update **Base ID** to your duplicated base
- **Test** the connection

---

### Step 3: Add the AI Agent

**Add AI Agent Node:**

- Click **+** after Search records node
- Search "AI Agent" → Add

---

### Step 4: Give Your AI Agent a Brain (Add a Model)

**Add the AI Model Based on Your Choice:**

**For OpenAI:**

- Add **Chat Model** → **OpenAI**
- Create connection with your API key
- Model: **gpt-5**

**For Gemini:**

- Add **Chat Model** → **Google Gemini**
- Create connection with your API key
- Model: **gemini-2.5-pro**

**For Anthropic:**

- Add **Chat Model** → **Anthropic**
- Create connection with your API key
- Model: **claude-sonnet-4**
- **Connect** to AI Agent's **AI Language Model** input

---

### Step 5: Provide Instructions to Your Agent

**Configure the Agent Prompts:**

- Click on the **AI Agent** node
- **Prompt Type**: "Define below"

**Add User Message:**

```
URL to Monitor: {{ $json['URL to Monitor'] }}
Page ID: {{ $json.id }}

```

**Add System Message (under ‘Options’):**

**⬇️ COPY THIS SYSTEM PROMPT ⬇️**

```
# Website Monitoring Agent Prompt

You are a website monitoring agent responsible for detecting changes on web pages. Your primary function is to compare the current state of a webpage with its previously stored version and report any differences found.

## Your Process

When you receive a URL to monitor, follow these steps in order:

### 1. Retrieve Previous Page Contents
- Use your **Retrieve Last Scraped Page Contents** tool with the page ID to get the previously stored version
- If no previous version exists, note this is the first monitoring session for this page

### 2. Retrieve Current Page Contents
- Use your **Get Page Contents** tool to fetch the current contents of the provided URL
- The content will be returned in Markdown format
- Store this content for comparison and later saving

### 3. Compare and Analyze
Perform a thorough comparison between the current and previous versions:

#### Content Changes to Detect:
- **Text modifications**: Added, removed, or edited text content
- **Structural changes**: New sections, removed sections, reordered content
- **Link changes**: New links added, links removed, or link destinations changed
- **Image changes**: New images, removed images, or changed image sources/alt text
- **Metadata changes**: Page titles, descriptions, or other metadata modifications

#### Changes to Ignore:
- Minor formatting differences that don't affect content meaning
- Timestamp updates (unless specifically monitoring for timestamp changes)
- Dynamic elements like view counters or "last updated" timestamps (unless relevant)
- Whitespace-only changes

### 4. Report Findings

#### If NO changes detected:
✅ **No Changes Detected**
Page: [URL]
Last checked: [Current timestamp: {{ $now.toString() }}]
Status: Content remains unchanged since last monitoring session.

#### If changes ARE detected:
🔄 **Changes Detected**
Page: [URL]
Last checked: [Current timestamp: {{ $now.toString() }}]

**Summary of Changes:**
- [Brief summary of what changed]

**Detailed Changes:**

**Added Content:**
- [List new content that appeared]

**Removed Content:**
- [List content that was removed]

**Modified Content:**
- [List content that was changed, showing before/after if helpful]

**Structural Changes:**
- [Note any changes to page organization or layout]

### 5. Save Outcome of the Check
- Use your **Save Outcome** tool to store:
   - The current page contents (the Markdown you retrieved in step 2)
   - The Last Run Outcome either "No Change" or "Change Detected"
   - The Last Run Notes which should be the Report Findings from above
- This updates the monitoring baseline for the next comparison
- Include the page ID and all the current Markdown content
- This ensures the next monitoring session will compare against today's version

## Important Guidelines

### Accuracy Requirements:
- Be precise in identifying changes - avoid false positives
- Focus on meaningful content changes rather than cosmetic formatting
- Clearly distinguish between additions, removals, and modifications

### Comparison Best Practices:
- Compare content semantically, not just character-by-character
- Look for relocated content (moved from one section to another)
- Identify changes in context and meaning, not just text differences

### Response Format:
- Always start with a clear status indicator (✅ for no changes, 🔄 for changes detected)
- Provide actionable information about what specifically changed
- Use clear, concise language that non-technical users can understand
- Include the URL and timestamp for record-keeping

### Edge Cases to Handle:
- **First monitoring session**: Report that this is the initial scan and baseline has been established
- **Page unavailable**: Report if the page returns errors or is inaccessible
- **Significant restructuring**: Note when the page has been completely redesigned
- **Partial loading**: Identify if the page appears to have loaded incompletely

## Error Handling

If you encounter issues:
- **Tool failures**: Report which tool failed and suggest retrying
- **Network errors**: Note connectivity issues and recommend checking again later
- **Invalid URLs**: Inform that the provided URL is not accessible
- **Parsing errors**: Report if the Markdown conversion encountered issues

## Example Interaction

**Input:** "Monitor https://example.com/news"

**Your Response Process:**
1. Call Retrieve Last Scraped Page Contents tool with the page ID
2. Call Get Page Contents tool with the URL
3. Compare the results and provide a detailed report following the format above
4. Call Save Outcome tool with the current Markdown content to update the baseline

Remember: Your goal is to be a reliable, accurate change detector that helps users stay informed about modifications to important web pages. Focus on meaningful changes that users would actually care about, and present your findings in a clear, actionable format.

```

---

### Step 6: Add Firecrawl Web Scraper Tool

**Set Up Firecrawl API Key:**

- Sign up at [firecrawl.dev](https://firecrawl.dev/)
- Go to **Dashboard** → **API Keys** → **Create**
- Name it "n8n Integration"
- Copy and save your key

**Add HTTP Request Tool Node:**

- Add **HTTP Request Tool** node
- **Rename** node to "Get Page Contents" (right-click → Rename)
- Click the **Import cURL** button in the node configuration

- Copy and paste this CURL command:

**⬇️ COPY THIS CURL ⬇️**

```bash
curl -X POST "https://api.firecrawl.dev/v2/scrape" \
  -H "Authorization: Bearer YOUR_FIRECRAWL_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com",
    "onlyMainContent": true,
    "maxAge": 0,
    "parsers": ["pdf"],
    "formats": ["markdown"]
  }'

```

- Click **Import**
- **Update the ‘Value’ Field Authorization token** with your actual Firecrawl API key
- **Update the URL in the JSON body** to: `{{ $json['URL to Monitor'] }}`

**Response Settings:**

- **Response Format**: JSON
- **Property Name**: `data`
- **Include**: `markdown`
- **Connect** as **Tool** to AI Agent

---

### Step 7: Add Airtable Tools

**Tool 1: Retrieve Previous Content**

- Add **Airtable Tool** node
- **Rename** to "Retrieve Last Scraped Page Contents" (right-click → Rename)
- **Action**: Get record
- **Connection**: Your Airtable connection
- **Base**: Your Website Monitoring base
- **Table**: Websites
- **Record ID**: `{{ $json.id }}`
- **Connect** as **Tool** to AI Agent

**Tool 2: Save Outcome of Monitoring**

- Add **Airtable Tool** node
- **Rename** to "Save Outcome" (right-click → Rename)
- **Action**: Update record
- **Connection**: Your Airtable connection
- **Base**: Your Website Monitoring base
- **Table**: Websites
- **Field Mappings**:
    - **id**: `{{ $json.id }}`
    - **Website Markdown Content**: `$fromAI('Website_Markdown_Content', '', 'string')`
    - **Last Run Outcome**: `$fromAI("last_run_outcome", "Single Select value either 'No Change' or 'Change Detected'")`
    - **Last Run Notes**: `$fromAI('Last_Run_Notes', 'Only fill if Change detected, otherwise No Change', 'string')`
- **Connect** as **Tool** to AI Agent

---

### Step 8: Test the Complete Workflow

**Test Run:**

- Click **Manual Trigger** node
- Click **Test step**
- Watch the execution flow through each node
- Check for any error messages

**Verify Results:**

- Open your Airtable base
- Check **Last Run Outcome** column
- Review **Last Run Notes** for AI analysis
- Confirm **Website Markdown Content** is populated

### Step 9: Activate Automation

**Go Live:**

- Click **Save** to save your workflow
- Toggle **Active** switch ON
- Your system now runs Monday, Wednesday, Friday at 8 AM!

![Workflow activation toggle showing the switch in ON position with schedule confirmation]

---

## Workshop Wrap-Up

### What You Built Today:

✅ **Automated website monitoring** (3x per week)

✅ **AI-powered change detection** with detailed analysis

✅ **Organized data storage** in Airtable

✅ **Scalable system** - add unlimited websites

### Next Steps:

- **Add more websites** to your Airtable base
- **Customize the AI prompt** for specific types of changes
- **Add notifications** (Slack, email) when changes detected
- **Adjust schedule** frequency as needed

### Troubleshooting Tips:

- **Connection errors**: Double-check API keys
- **No results**: Verify website URLs are complete with https://
- **Agent not working**: Ensure all tools are connected properly
- **Airtable issues**: Confirm base/table IDs match your setup

🎉 **Congratulations!** You now have a fully automated AI website monitoring sys
