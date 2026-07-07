# Email Automation Workflows (n8n)

This repository contains a comprehensive email automation system built with n8n. The system manages the entire lifecycle of email campaigns, from sending initial outreach to handling replies and follow-ups.

## 📋 Workflow Sequence

The automation follows this sequence:

1. **Email Sending** → Sends personalized outreach emails
2. **Email Receiving** → Monitors and processes email replies
3. **Email To Manager** → Sends daily summary reports to management
4. **Email To Interested** → Sends follow-up emails to interested leads

## 🔄 Workflow Details

### 1. Email Sending Automation
**File:** `Email Sending Automation.json`

**Purpose:** Sends personalized marketing emails to contacts from a Google Sheet.

**Features:**
- Fetches contacts and email templates from Google Sheets
- Filters eligible contacts (up to 125 per run)
- Randomly assigns workspaces (W1-W10) and templates (T01-T10)
- Personalizes email content with contact names
- Optionally includes product images from Shopify CDN
- Routes emails through 10 different workspace accounts for better deliverability
- Updates contact status in Google Sheets after sending
- Removes uninterested contacts automatically

**Schedule:** Runs 4 times daily at 9 AM, 12 PM, 3 PM, and 6 PM

**Key Nodes:**
- Fetch Contacts from Sheet
- Fetch Email Templates
- Filter Eligible Contacts
- Prepare Personalized Email
- Select Workspace (routes to W1-W10)
- Send Email via Workspace (10 parallel paths)
- Update Contact Sheet

---

### 2. Email Receiving Automation
**File:** `Email Receiving Automation.json`

**Purpose:** Monitors email replies and categorizes respondents as interested or not.

**Features:**
- Checks for replies across all 10 workspaces
- Groups emails by workspace for efficient processing
- Extracts reply information (sender name, email, message)
- Detects interest level based on reply content
- Updates Google Sheets with reply status
- Appends interested leads to a separate leads sheet
- Removes uninterested contacts from the main list

**Schedule:** Runs every minute

**Key Nodes:**
- Get records to check for replied
- Group by Workspace
- Switch (routes to W1-W10)
- Get Gmail Thread for each workspace
- Extract Reply Info
- Check Interest
- Update Google Sheet / Append Leads Sheet

---

### 3. Email To Manager
**File:** `Email To Manager.json`

**Purpose:** Sends a daily summary email to management with pending email statistics.

**Features:**
- Fetches pending emails from the leads sheet
- Filters up to 500 pending/unscheduled emails
- Counts emails by type/category
- Generates HTML email with summary table
- Schedules emails for next day (24 hours ahead)
- Updates sheet with "Scheduled" status
- Sends formatted report to manager

**Schedule:** Runs daily at 9 AM

**Key Nodes:**
- Read Pending Emails from Sheet
- Filter Pending & Mark Updated
- Get Manager Template
- Prepare Summary Email
- Send Mail to Manager
- Update Status in Sheet

---

### 4. Email To Interested
**File:** `Email To Interested.json`

**Purpose:** Sends personalized follow-up emails to interested leads at their scheduled time.

**Features:**
- Fetches emails with "Scheduled" status
- Filters emails where scheduled_at time has passed
- Limits to 125 emails per run
- Fetches appropriate email template by type
- Personalizes content with contact name and custom data
- Sends email via Gmail
- Verifies successful sending
- Updates status to "SENT" in Google Sheets

**Schedule:** Runs 4 times daily at 9 AM, 12 PM, 3 PM, and 6 PM

**Key Nodes:**
- Fetch Scheduled Emails
- Filter Due Emails
- Fetch Template by Type
- Prepare Email Payload
- Send Email
- Check If Email Sent
- Update Sheet Status to SENT

---

### 5. Global Error Handler
**File:** `Global Error Handler.json`

**Purpose:** Centralized error handling for all workflows.

**Features:**
- Captures errors from any workflow
- Formats detailed error report with:
  - Workflow name and ID
  - Failed node information
  - Error message and full stack trace
  - Execution ID and URL
  - Timestamp
- Logs errors to Google Sheets for tracking
- Sends HTML-formatted email notification to admin

**Trigger:** Activates on any workflow error

---

## 🔧 Setup Instructions

### Prerequisites
- n8n installation (self-hosted or cloud)
- Google Workspace account with Gmail and Sheets access
- OAuth credentials for Gmail and Google Sheets

### Installation

1. **Import Workflows**
   - Open n8n
   - Import each JSON file as a new workflow
   - Activate each workflow

2. **Configure Credentials**
   - Add Gmail OAuth2 credentials for each workspace (W1-W10)
   - Add Google Sheets OAuth2 credentials
   - Update credential IDs in each workflow

3. **Setup Google Sheets**
   - Create the following sheets:
     - Contacts Sheet (for main contact list)
     - Templates Sheet (for email templates)
     - Images Sheet (for product images)
     - Leads Sheet (for interested contacts)
     - Error Log Sheet (for error tracking)
   - Update sheet IDs in workflow configurations

4. **Configure Email Templates**
   - Add templates T01-T10 to the Templates Sheet
   - Templates support variables: `{{name}}`, `${url}`, `${name}`

5. **Set Manager Email**
   - Update the manager email address in "Email To Manager" workflow

### Required Google Sheets Structure

**Contacts Sheet:**
- `id` - Unique contact ID
- `email` - Contact email address
- `name` - Contact name
- `last_sent_date` - Last email sent date
- `templates_sent` - Comma-separated list of sent template IDs
- `workspace_used` - Workspace used for sending
- `replied` - Reply status (yes/no)
- `status` - Current status (SENT/REPLIED)
- `thread_id` - Gmail thread ID

**Templates Sheet:**
- `template_id` - Template identifier (T01-T10)
- `subject` - Email subject line
- `body` - Email body (HTML)
- `image` - Boolean flag for image inclusion

**Leads Sheet:**
- `email` - Lead email
- `name` - Lead name
- `type` - Lead category/type
- `status` - Processing status (Pending/Scheduled/SENT)
- `scheduled_at` - Scheduled send time

---

## ⚙️ Configuration

### Workspace Accounts
The system uses 10 different Gmail accounts (W1-W10) to:
- Distribute email sending load
- Improve deliverability
- Avoid rate limits

### Rate Limits
- **Email Sending:** Max 125 emails per run (4x daily = 500/day)
- **Email To Interested:** Max 125 emails per run
- **Reply Checking:** Continuous (every minute)

### Scheduling
All schedules are in the server's timezone. Adjust as needed:
- Email Sending: 9:00, 12:00, 15:00, 18:00
- Email Receiving: Every 1 minute
- Email To Manager: 9:00
- Email To Interested: 9:00, 12:00, 15:00, 18:00

---

## 🔒 Security Notes

⚠️ **IMPORTANT:** The following files contain sensitive credentials and are excluded from git:
- `client_secret_*.json` - OAuth client secrets
- `*.apps.googleusercontent.com.json` - Google API credentials

Never commit credential files to version control!

---

## 🛠️ Troubleshooting

### Common Issues

1. **Emails not sending**
   - Check Gmail credentials are valid
   - Verify OAuth consent screen is configured
   - Check daily sending limits

2. **Replies not detected**
   - Ensure thread_id is correctly stored
   - Verify Gmail API scope includes read access
   - Check workspace routing logic

3. **Sheet updates failing**
   - Verify Google Sheets API credentials
   - Check sheet IDs are correct
   - Ensure column names match exactly

4. **Error notifications not received**
   - Verify Error Workflow is active
   - Check admin email address
   - Ensure workflows reference correct error handler ID

---

## 📊 Data Flow

```
Contact Sheet → Email Sending → Gmail (W1-W10) → Update Sheet
                                      ↓
Email Receiving ← Gmail Thread ← Check Replies
        ↓
   Interested? 
        ↓
    Yes → Leads Sheet → Email To Manager → Schedule
                              ↓
                    Email To Interested → Send & Update
        ↓
    No → Remove from Sheet
```

---

## 🤝 Contributing

When modifying workflows:
1. Test in n8n development environment
2. Export updated JSON files
3. Update this README with changes
4. Never commit credential files

---

## 📝 License

This project is proprietary. All rights reserved.

---

## 📧 Support

For questions or issues, contact the system administrator.

---

**Last Updated:** October 16, 2025

