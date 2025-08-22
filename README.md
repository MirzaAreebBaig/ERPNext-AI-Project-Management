# ERPNext AI Project Management
Connect N8N with ERPNext for Automating your Project Management 
# ERPNext Task Management Automation with N8N

An intelligent task management automation system that integrates Frappe ERPNext with N8N workflows to automate project management through email notifications and AI-powered response processing.

## 🚀 Features

- **📧 Automated Daily Task Reminders**: Send personalized emails to users with their pending tasks
- **🤖 AI-Powered Email Processing**: Automatically interpret user email replies and take appropriate actions
- **✅ Smart Task Updates**: Update task status, create new tasks, and log time entries based on email responses
- **👤 Personalized Communications**: Address users by name and customize content based on their specific tasks
- **🔄 Real-time Synchronization**: Seamlessly sync between email responses and ERPNext data
- **📊 Comprehensive Logging**: Track all automated actions with confirmation emails

## 📋 Prerequisites

### Software Requirements
- **N8N** (Self-hosted or Cloud)
- **Frappe ERPNext** instance with API access
- **Email Server** (Gmail/SMTP for sending, IMAP for receiving)
- **OpenAI API** account (or compatible AI service)

### ERPNext Configuration
- API access enabled
- Custom fields in Task doctype:
  - `custom_assigned_to` (Link to User)
  - `custom_assigned_name` (Data)
- Required permissions for API user:
  - Task: Read, Write, Create
  - Timesheet: Read, Write, Create
  - Employee: Read

### Email Setup
- **SMTP Access** for sending emails
- **IMAP Access** for receiving replies
- **App Passwords** configured (for Gmail)

## 🏗️ Architecture

```
┌─────────────────┐    ┌──────────────┐    ┌─────────────────┐
│   ERPNext       │◄──►│     N8N      │◄──►│   Email Server  │
│   (Tasks)       │    │  Workflows   │    │  (SMTP/IMAP)    │
└─────────────────┘    └──────┬───────┘    └─────────────────┘
                              │
                       ┌──────▼───────┐
                       │   OpenAI     │
                       │   (AI API)   │
                       └──────────────┘
```

## 📁 Repository Structure

```
├── workflows/
│   ├── daily-task-reminder.json          # Main workflow for sending daily reminders
│   ├── email-reply-processor.json        # AI-powered email reply processing
│   └── workflow-settings.json            # Environment variables template
├── docs/
│   ├── setup-guide.md                    # Detailed setup instructions
│   ├── api-configuration.md              # ERPNext API setup guide
│   └── troubleshooting.md                # Common issues and solutions
├── examples/
│   ├── sample-emails.md                  # Example email formats and responses
│   └── test-scenarios.md                 # Testing scenarios and expected outcomes
└── README.md                             # This file
```

## 🛠️ Installation & Setup

### 1. Clone Repository
```bash
git clone https://github.com/mirzaareebbaig/ERPNext-AI-Project-Management.git
cd erpnext-n8n-automation
```

### 2. Import N8N Workflows
1. Open your N8N instance
2. Go to **Workflows** → **Import from file**
3. Import `workflows/daily-task-reminder.json`
4. Import `workflows/email-reply-processor.json`

### 3. Configure Environment Variables
Create the following environment variables in N8N:

```bash
# ERPNext Configuration
ERPNEXT_URL=http://erp.airlabs.com:8000
ERPNEXT_API_KEY=your_api_key
ERPNEXT_API_SECRET=your_api_secret

# Email Configuration
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@domain.com
SMTP_PASS=your_app_password

IMAP_HOST=imap.gmail.com
IMAP_PORT=993
IMAP_USER=your-email@domain.com
IMAP_PASS=your_app_password

# AI Configuration
OPENAI_API_KEY=your_openai_api_key
OPENAI_MODEL=gpt-4

# Scheduling
REMINDER_SCHEDULE=0 9 * * 1-5  # Monday-Friday at 9 AM
TIMEZONE=Asia/Kolkata
```

### 4. Customize Task Fields
Update the field mappings in both workflows to match your ERPNext Task doctype:

```javascript
// In daily-task-reminder.json, update the fields array:
"fields": [
  "name",
  "subject", 
  "description",
  "status",
  "priority",
  "project",
  "exp_end_date",
  "custom_assigned_to",
  "custom_assigned_name"
]
```

## 🎯 Usage

### Daily Task Reminder Workflow

**Automatic Execution:**
- Runs Monday-Friday at 9 AM (configurable)
- Fetches all pending tasks from ERPNext
- Groups tasks by assigned user
- Sends personalized emails with task lists

**Manual Execution:**
1. Open `daily-task-reminder` workflow
2. Click **"Execute Workflow"**
3. Monitor execution in the workflow panel

### Email Reply Processing

Users can reply to task reminder emails with natural language:

**Update Task Status:**
```
"TASK-2025-00001 is completed"
"Finished working on TASK-2025-00002"
"TASK-2025-00003 is still in progress"
```

**Create New Tasks:**
```
"I need to add a task for testing the API"
"Create task: Review documentation for new features"
"New task needed: Fix database performance issues"
```

**Log Time Entries:**
```
"Spent 3 hours on TASK-2025-00001 fixing bugs"
"Logged 5 hours yesterday on TASK-2025-00002"
"Worked 2.5 hours on database optimization"
```

**Request Help:**
```
"Having issues with TASK-2025-00001, server won't start"
"Need help with TASK-2025-00002, unclear requirements"
```

## 🔧 Configuration

### ERPNext API Setup

1. **Enable API Access:**
   ```
   Settings → System Settings → Enable API Access = Yes
   ```

2. **Create API User:**
   ```
   User Management → Add User
   - Generate API Key & Secret
   - Assign custom role with required permissions
   ```

3. **Required Permissions:**
   - Task: Read, Write, Create, Delete
   - Timesheet: Read, Write, Create
   - Employee: Read
   - User: Read

### Email Template Customization

The email templates can be customized in the workflow nodes:

```html
<!-- Located in Gmail/SMTP nodes -->
<h2>📋 Hello {{ $json.user_name }}!</h2>
<h3>Your Pending Tasks ({{ $json.task_count }} tasks)</h3>

<!-- Task details rendered from tasks_html -->
{{ $json.tasks_html }}

<!-- Instructions for users -->
<h3>📧 Reply to this email with updates:</h3>
<!-- ... -->
```

### AI Prompt Customization

Modify the system prompt in the OpenAI node:

```javascript
// Located in email-reply-processor workflow
{
  "role": "system",
  "content": "You are an AI assistant that processes task management email replies..."
}
```

## 📊 Monitoring & Logging

### Workflow Execution Logs
- Access via N8N interface: **Executions** tab
- Filter by workflow name and date range
- Review input/output data for each node

### Email Confirmations
All actions generate confirmation emails:
- ✅ **Success**: Action completed successfully
- ❌ **Error**: Action failed with error details
- 📧 **Help**: User request logged for manual follow-up

### ERPNext Activity Logs
Check ERPNext for automated updates:
- **Task Comments**: Track status changes
- **Timesheet Entries**: Review logged hours
- **Activity Log**: Monitor API calls

## 🧪 Testing

### Test Scenarios

1. **Basic Task Reminder:**
   - Execute daily-task-reminder workflow
   - Verify email sent to users with tasks
   - Check email formatting and content

2. **Task Status Updates:**
   - Reply to reminder email: "TASK-2025-00001 is done"
   - Verify task status updated in ERPNext
   - Confirm confirmation email received

3. **New Task Creation:**
   - Reply: "Create task: Test new feature"
   - Check new task appears in ERPNext
   - Verify correct assignment and details

4. **Time Logging:**
   - Reply: "Logged 3 hours on TASK-2025-00001"
   - Check timesheet entry in ERPNext
   - Verify time calculation and description

### Debugging Tips

1. **Check N8N Execution Logs:**
   ```
   Workflows → [Workflow Name] → Executions
   ```

2. **Verify API Responses:**
   - Monitor HTTP Request node outputs
   - Check ERPNext API error messages

3. **Test AI Processing:**
   - Review OpenAI node inputs/outputs
   - Validate JSON parsing in subsequent nodes

4. **Email Configuration Issues:**
   - Test SMTP/IMAP connections separately
   - Check app password configuration

## 📈 Advanced Features

### Custom Task Fields
Add support for additional ERPNext fields:

```javascript
// In the grouping code node:
userTasks[assignedTo].tasks.push({
  id: task.name,
  subject: task.subject,
  // Add your custom fields here:
  custom_field_1: task.custom_field_1,
  custom_field_2: task.custom_field_2
});
```

### Multi-language Support
Customize email templates and AI prompts for different languages:

```javascript
// Add language detection in email processing:
const userLanguage = getUserLanguage(task.custom_assigned_to);
const template = getEmailTemplate(userLanguage);
```

### Integration with Other Systems
Extend workflows to integrate with:
- Slack notifications
- Microsoft Teams alerts
- Jira synchronization
- Calendar events

### Advanced AI Processing
Enhance AI capabilities:
- Sentiment analysis for user responses
- Priority classification for new tasks
- Automatic task categorization
- Smart time estimation

## 🤝 Contributing

1. **Fork the repository**
2. **Create a feature branch:**
   ```bash
   git checkout -b feature/amazing-feature
   ```
3. **Make your changes and test thoroughly**
4. **Commit your changes:**
   ```bash
   git commit -m 'Add amazing feature'
   ```
5. **Push to the branch:**
   ```bash
   git push origin feature/amazing-feature
   ```
6. **Create a Pull Request**

### Contribution Guidelines
- Follow existing code style and naming conventions
- Add comprehensive documentation for new features
- Include test scenarios for workflow changes
- Update README.md if adding new functionality

## 🐛 Troubleshooting

### Common Issues

**Issue: No emails being sent**
- Check SMTP configuration and credentials
- Verify task assignment fields are populated
- Review N8N execution logs for errors

**Issue: AI not processing emails correctly**
- Verify OpenAI API key and quota
- Check email content extraction in logs
- Review AI prompt for accuracy

**Issue: ERPNext API calls failing**
- Confirm API credentials and permissions
- Check ERPNext server connectivity
- Verify field names match your doctype

**Issue: Workflow not triggering**
- Check cron schedule format
- Verify N8N timezone settings
- Ensure workflow is activated

For detailed troubleshooting, see `docs/troubleshooting.md`.

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- **Frappe Team** for the excellent ERPNext platform
- **N8N Community** for the powerful automation platform  
- **OpenAI** for enabling intelligent email processing
- **Contributors** who helped improve this project

## 📞 Support

- **Issues**: [GitHub Issues](https://github.com/mirzaareebbaig/ERPNext-AI-Project-Management/issues)
- **Discussions**: [GitHub Discussions](https://github.com/mirzaareebbaig/ERPNext-AI-Project-Management/discussions)
- **Documentation**: [Wiki](https://github.com/mirzaareebbaig/ERPNext-AI-Project-Management/wiki)

---

**Made with ❤️ for the ERPNext and N8N community**

*Automate your project management and let AI handle the routine tasks while you focus on what matters most.*
