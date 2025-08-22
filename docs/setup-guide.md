# Setup Guide - ERPNext N8N Automation

This guide will walk you through the complete setup process for the ERPNext Task Management Automation system.

## 📋 Prerequisites Checklist

Before starting, ensure you have:

- [ ] **N8N instance** (self-hosted or cloud)
- [ ] **Frappe ERPNext** with administrator access
- [ ] **Gmail account** with 2FA enabled (or other email provider)
- [ ] **OpenAI API account** with available credits
- [ ] **Basic understanding** of JSON and workflow concepts

## 🏗️ Phase 1: ERPNext Preparation

### Step 1: Enable API Access

1. **Login to ERPNext** as Administrator
2. **Go to:** `Settings > System Settings`
3. **Enable:** "Allow API Access" checkbox
4. **Save** the settings

### Step 2: Create API User

1. **Navigate to:** `User and Permissions > User`
2. **Create new user:**
   ```
   Email: automation@yourdomain.com
   First Name: N8N
   Last Name: Automation
   Role Profile: (Create custom or use existing)
   ```
3. **Generate API Keys:**
   - Go to the user record
   - Scroll to "API Access" section
   - Click **"Generate Keys"**
   - **Save the API Key and API Secret** (you'll need these later)

### Step 3: Configure Task Doctype

1. **Navigate to:** `Customization > Customize Form`
2. **Select DocType:** Task
3. **Add custom fields** (if not already present):

   ```
   Field 1:
   Label: Custom Assigned To
   Type: Link
   Options: User
   Fieldname: custom_assigned_to
   
   Field 2:
   Label: Custom Assigned Name  
   Type: Data
   Fieldname: custom_assigned_name
   ```

4. **Save** the customization

### Step 4: Set Permissions

1. **Create Custom Role:**
   ```
   Role Name: N8N Automation
   ```

2. **Set Permissions for Role:**
   - **Task**: Read, Write, Create, Delete
   - **Timesheet**: Read, Write, Create
   - **Employee**: Read
   - **User**: Read

3. **Assign role to API user**

## 🏗️ Phase 2: Email Configuration

### Step 1: Gmail App Password Setup

1. **Enable 2-Factor Authentication** on your Gmail account
2. **Generate App Password:**
   - Go to Google Account settings
   - Security > 2-Step Verification > App passwords
   - Select app: Mail
   - Select device: Other (Custom name)
   - Name it: "N8N Automation"
   - **Copy the generated password**

### Step 2: Test Email Connectivity

Test SMTP and IMAP connectivity:

```bash
# Test SMTP (sending emails)
telnet smtp.gmail.com 587

# Test IMAP (receiving emails)  
telnet imap.gmail.com 993
```

## 🏗️ Phase 3: OpenAI API Setup

### Step 1: Create OpenAI Account

1. **Go to:** [platform.openai.com](https://platform.openai.com)
2. **Sign up** or log in
3. **Add payment method** (required for API access)
4. **Set usage limits** to control costs

### Step 2: Generate API Key

1. **Navigate to:** API Keys section
2. **Create new secret key**
3. **Copy and save** the key securely
4. **Set usage quotas** (recommended: $10-20/month for testing)

## 🏗️ Phase 4: N8N Environment Setup

### Step 1: Environment Variables

Set up these environment variables in your N8N instance:

```bash
# ERPNext Configuration
ERPNEXT_URL=http://erp.airlabs.com:8000
ERPNEXT_API_KEY=your_api_key_here
ERPNEXT_API_SECRET=your_api_secret_here

# Email Configuration
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_SECURE=false
SMTP_USER=your-email@gmail.com
SMTP_PASS=your_app_password_here

IMAP_HOST=imap.gmail.com
IMAP_PORT=993
IMAP_SECURE=true
IMAP_USER=your-email@gmail.com
IMAP_PASS=your_app_password_here

# AI Configuration
OPENAI_API_KEY=your_openai_key_here
OPENAI_MODEL=gpt-4
OPENAI_MAX_TOKENS=1000

# Scheduling Configuration
REMINDER_SCHEDULE=0 9 * * 1-5
TIMEZONE=Asia/Kolkata

# Debugging
DEBUG_MODE=false
LOG_LEVEL=info
```

### Step 2: Docker Environment (if using Docker)

If running N8N in Docker, add to your `docker-compose.yml`:

```yaml
version: '3.8'
services:
  n8n:
    image: n8nio/n8n
    restart: always
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=your_password
      # Add all environment variables here
      - ERPNEXT_URL=${ERPNEXT_URL}
      - ERPNEXT_API_KEY=${ERPNEXT_API_KEY}
      # ... (all other variables)
    volumes:
      - n8n_data:/home/node/.n8n
      - ./workflows:/home/node/workflows

volumes:
  n8n_data:
```

## 🏗️ Phase 5: Workflow Import and Configuration

### Step 1: Import Workflows

1. **Download workflow files** from this repository
2. **Open N8N interface**
3. **Go to:** Workflows
4. **Click:** Import from file
5. **Import both workflows:**
   - `daily-task-reminder.json`
   - `email-reply-processor.json`

### Step 2: Configure Credentials

For each workflow, set up credentials:

#### ERPNext API Credentials
```json
{
  "name": "ERPNext API",
  "type": "httpHeaderAuth",
  "data": {
    "name": "Authorization",
    "value": "token {{ERPNEXT_API_KEY}}:{{ERPNEXT_API_SECRET}}"
  }
}
```

#### Gmail Credentials
```json
{
  "name": "Gmail SMTP",
  "type": "smtp",
  "data": {
    "host": "smtp.gmail.com",
    "port": 587,
    "secure": false,
    "user": "{{SMTP_USER}}",
    "password": "{{SMTP_PASS}}"
  }
}

{
  "name": "Gmail IMAP", 
  "type": "imap",
  "data": {
    "host": "imap.gmail.com",
    "port": 993,
    "secure": true,
    "user": "{{IMAP_USER}}",
    "password": "{{IMAP_PASS}}"
  }
}
```

#### OpenAI Credentials
```json
{
  "name": "OpenAI",
  "type": "openAi",
  "data": {
    "apiKey": "{{OPENAI_API_KEY}}"
  }
}
```

### Step 3: Update Field Mappings

In both workflows, update field mappings to match your ERPNext setup:

```javascript
// In HTTP Request nodes, update fields array:
"fields": [
  "name",
  "subject", 
  "description",
  "status",
  "priority",
  "project",
  "exp_end_date",           // Your custom due date field
  "custom_assigned_to",     // Your custom assignment field
  "custom_assigned_name"    // Your custom name field
]
```

## 🏗️ Phase 6: Initial Testing

### Step 1: Test ERPNext Connection

1. **Open:** Daily Task Reminder workflow
2. **Click:** Execute Workflow
3. **Check:** HTTP Request node output
4. **Verify:** Task data is retrieved correctly

### Step 2: Test Email Sending

1. **Complete:** Task retrieval test
2. **Check:** Gmail node execution
3. **Verify:** Email received in target inboxes
4. **Review:** Email formatting and content

### Step 3: Test AI Processing

1. **Send test email** reply to the reminder
2. **Monitor:** Email Reply Processor workflow
3. **Check:** OpenAI node output for correct parsing
4. **Verify:** Actions are processed correctly

### Step 4: Test ERPNext Updates

1. **Send completion email:** "TASK-XXX is done"
2. **Check:** Task status updated in ERPNext
3. **Verify:** Confirmation email received

## 🏗️ Phase 7: Production Deployment

### Step 1: Schedule Configuration

1. **Open:** Daily Task Reminder workflow
2. **Configure Schedule Trigger:**
   ```
   Cron Expression: 0 9 * * 1-5  
   Timezone: Asia/Kolkata
   ```
3. **Activate** the workflow

### Step 2: Error Handling Setup

1. **Add error workflows** for failed executions
2. **Set up monitoring** alerts
3. **Configure backup** email notifications

### Step 3: User Training

1. **Create user documentation** with email reply formats
2. **Conduct training sessions** for team members
3. **Provide example emails** and expected responses
4. **Set up support channel** for user questions

## 🔧 Customization Options

### Custom Task Fields

To add support for additional ERPNext fields:

```javascript
// In the grouping code node:
userTasks[assignedTo].tasks.push({
  id: task.name,
  subject: task.subject,
  // Add your custom fields:
  department: task.department,
  customer: task.customer,
  estimated_hours: task.estimated_hours
});
```

### Custom Email Templates

Modify email templates in Gmail nodes:

```html
<!-- Add your company branding -->
<div style="header-style">
  <img src="your-logo-url" alt="Company Logo">
  <h1>Daily Task Report</h1>
</div>

<!-- Customize task display -->
<div class="task-card">
  <!-- Your custom task formatting -->
</div>
```

### AI Prompt Customization

Enhance AI understanding by modifying the system prompt:

```javascript
{
  "role": "system", 
  "content": `You are an AI assistant for [Your Company]. 
             Process task management emails and extract actions.
             
             Additional rules:
             - Recognize department-specific terminology
             - Handle project codes in format: PROJ-YEAR-NUMBER
             - Default new task priority based on keywords
             
             [Your custom instructions...]`
}
```

## 🎯 Next Steps

After successful setup:

1. **Monitor workflow executions** for the first week
2. **Collect user feedback** on email formats and responses
3. **Fine-tune AI prompts** based on actual usage patterns
4. **Add advanced features** as needed
5. **Scale to additional ERPNext doctypes** (Projects, Issues, etc.)

## 🆘 Getting Help

If you encounter issues:

1. **Check:** `troubleshooting.md` for common problems
2. **Review:** N8N execution logs for detailed errors
3. **Test:** Individual components (API, email, AI) separately
4. **Consult:** ERPNext and N8N community forums
5. **Create:** GitHub issue with detailed error information

---

**Congratulations!** You now have a fully automated task management system integrated with ERPNext and N8N. The system will help streamline your project management and improve team communication through intelligent email automation.
