# API Configuration Guide - ERPNext N8N Automation

This guide provides detailed information on configuring ERPNext API access for the N8N automation workflows.

## 🔐 ERPNext API Overview

ERPNext provides RESTful APIs that allow external applications to interact with your system data. The N8N automation uses these APIs to:

- Retrieve task information
- Update task status
- Create new tasks
- Manage timesheet entries

## 🚀 Initial API Setup

### Step 1: Enable API Access

1. **Login as Administrator**
2. **Navigate to:** `Settings > System Settings`
3. **Find:** "API Access" section
4. **Enable:** "Allow API Access"
5. **Save:** Click Save

### Step 2: Create Dedicated API User

**Why create a dedicated user?**
- Better security and access control
- Easier to track API usage
- Separate permissions from regular users

**Create the user:**
```
User Details:
- Email: n8n-automation@yourdomain.com
- First Name: N8N
- Last Name: Automation Bot
- Username: n8n_automation
- Language: English
- Time Zone: Asia/Kolkata (or your timezone)

Security:
- Send Welcome Email: No
- Enabled: Yes
- User Type: System User
```

### Step 3: Generate API Credentials

1. **Open** the newly created user record
2. **Scroll down** to "API Access" section
3. **Click** "Generate Keys" button
4. **Copy and securely store:**
   - **API Key** (public identifier)
   - **API Secret** (private key)

**⚠️ Security Note:** Treat API credentials like passwords - never commit them to code repositories or share them publicly.

## 🔑 Authentication Methods

### Method 1: Token-based Authentication (Recommended)

```http
Headers:
Authorization: token {api_key}:{api_secret}
Content-Type: application/json
```

**Example:**
```bash
curl -X GET \
  'http://erp.airlabs.com:8000/api/resource/Task' \
  -H 'Authorization: token abc123:def456' \
  -H 'Content-Type: application/json'
```

### Method 2: Session-based Authentication

```bash
# Login to get session
curl -X POST \
  'http://erp.airlabs.com:8000/api/method/login' \
  -H 'Content-Type: application/json' \
  -d '{
    "usr": "username",
    "pwd": "password"
  }'

# Use session for subsequent requests
curl -X GET \
  'http://erp.airlabs.com:8000/api/resource/Task' \
  -H 'Cookie: sid=session_id_here'
```

## 📊 Required Permissions

### Custom Role Creation

Create a custom role with specific permissions:

1. **Navigate to:** `User and Permissions > Role`
2. **Create new role:** "N8N Automation"
3. **Set permissions** as detailed below

### Doctype Permissions

#### Task Doctype
```
Permissions for "Task":
- Read: ✅
- Write: ✅  
- Create: ✅
- Delete: ❌ (Not required)
- Print: ❌
- Email: ❌
- Report: ✅
- Import: ❌
- Export: ❌
- Set User Permissions: ❌
- Cancel: ✅ (if you want to cancel tasks)
- Amend: ❌

Conditions:
- User Permission: None
- Apply User Permissions: No
```

#### Timesheet Doctype
```
Permissions for "Timesheet":
- Read: ✅
- Write: ✅
- Create: ✅
- Delete: ❌
- Print: ❌
- Email: ❌
- Report: ✅
- Submit: ✅ (if using submitted timesheets)
- Cancel: ❌
- Amend: ❌
```

#### Employee Doctype (Read-only)
```
Permissions for "Employee":
- Read: ✅
- Write: ❌
- Create: ❌
(All others disabled)
```

#### User Doctype (Read-only)
```
Permissions for "User":
- Read: ✅
- Write: ❌
- Create: ❌
(All others disabled)
```

### Assign Role to API User

1. **Open** the N8N automation user record
2. **Go to** "Roles" section
3. **Add roles:**
   - N8N Automation (your custom role)
   - Desk User (for basic system access)

## 🛠️ API Endpoints Used

### 1. Get Tasks

**Endpoint:** `GET /api/resource/Task`

**Parameters:**
```json
{
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
  ],
  "filters": [
    ["status", "!=", "Completed"],
    ["status", "!=", "Cancelled"]
  ],
  "limit": 100
}
```

**N8N Configuration:**
```json
{
  "method": "GET",
  "url": "{{$env.ERPNEXT_URL}}/api/resource/Task",
  "authentication": "predefinedCredentialType",
  "nodeCredentialType": "httpHeaderAuth",
  "sendQuery": true,
  "queryParameters": {
    "fields": "[\"name\",\"subject\",\"description\",\"status\",\"priority\",\"project\",\"exp_end_date\",\"custom_assigned_to\",\"custom_assigned_name\"]",
    "filters": "[[\"status\",\"!=\",\"Completed\"],[\"status\",\"!=\",\"Cancelled\"]]"
  }
}
```

### 2. Update Task Status

**Endpoint:** `PUT /api/resource/Task/{task_id}`

**Body:**
```json
{
  "status": "Completed"
}
```

**N8N Configuration:**
```json
{
  "method": "PUT",
  "url": "{{$env.ERPNEXT_URL}}/api/resource/Task/{{$json.task_id}}",
  "authentication": "predefinedCredentialType", 
  "nodeCredentialType": "httpHeaderAuth",
  "sendBody": true,
  "bodyContentType": "json",
  "jsonBody": {
    "status": "{{$json.new_status}}"
  }
}
```

### 3. Create New Task

**Endpoint:** `POST /api/resource/Task`

**Body:**
```json
{
  "subject": "Task title",
  "description": "Task description",
  "custom_assigned_to": "user@domain.com",
  "custom_assigned_name": "User Name",
  "priority": "Medium",
  "status": "Open",
  "project": "PROJECT-001"
}
```

**N8N Configuration:**
```json
{
  "method": "POST",
  "url": "{{$env.ERPNEXT_URL}}/api/resource/Task",
  "authentication": "predefinedCredentialType",
  "nodeCredentialType": "httpHeaderAuth", 
  "sendBody": true,
  "bodyContentType": "json",
  "jsonBody": {
    "subject": "{{$json.task_description}}",
    "custom_assigned_to": "{{$json.user_email}}",
    "priority": "{{$json.priority}}",
    "status": "Open",
    "description": "Created via email: {{$json.original_content}}"
  }
}
```

### 4. Create Timesheet Entry

**Endpoint:** `POST /api/resource/Timesheet`

**Body:**
```json
{
  "employee": "EMP-001",
  "posting_date": "2025-01-15",
  "time_logs": [
    {
      "task": "TASK-2025-00001",
      "hours": 3.0,
      "description": "Fixed database issues",
      "from_time": "2025-01-15 09:00:00",
      "to_time": "2025-01-15 12:00:00",
      "activity_type": "Development"
    }
  ]
}
```

## 🔧 Custom Field Configuration

### Adding Custom Fields to Task Doctype

1. **Navigate to:** `Customization > Customize Form`
2. **Select DocType:** Task
3. **Add fields:**

#### Custom Assigned To Field
```
Label: Custom Assigned To
Type: Link
Options: User
Fieldname: custom_assigned_to
In List View: Yes
In Standard Filter: Yes
Mandatory: No
```

#### Custom Assigned Name Field
```
Label: Custom Assigned Name
Type: Data
Fieldname: custom_assigned_name
In List View: Yes
Fetch From: custom_assigned_to.full_name
Fetch If Empty: Yes
Read Only: Yes
```

#### Expected End Date Field
```
Label: Expected End Date
Type: Date
Fieldname: exp_end_date
In List View: Yes
In Standard Filter: Yes
Default: Today + 7 days
```

### Field Validation Rules

Add custom validation in Task doctype:

```python
# In Custom Script for Task doctype
def validate(self):
    # Auto-populate assigned name from user
    if self.custom_assigned_to and not self.custom_assigned_name:
        user = frappe.get_doc("User", self.custom_assigned_to)
        self.custom_assigned_name = user.full_name
    
    # Set default expected end date
    if not self.exp_end_date:
        self.exp_end_date = frappe.utils.add_days(frappe.utils.today(), 7)
```

## 🚦 Rate Limiting and Performance

### API Rate Limits

ERPNext doesn't have built-in rate limiting, but consider:

- **Concurrent requests:** Keep below 10 simultaneous requests
- **Request frequency:** Max 1 request per second for bulk operations
- **Timeout settings:** Set 30-second timeout in N8N

### Performance Optimization

```json
// Use specific fields instead of fetching all
"fields": ["name", "subject", "status"]

// Use filters to limit data
"filters": [
  ["modified", ">", "2025-01-01"],
  ["status", "!=", "Completed"]
]

// Limit results
"limit": 50

// Order by modified date
"order_by": "modified desc"
```

## 🔍 Testing API Configuration

### Test Script for Validation

```bash
#!/bin/bash

# Configuration
ERPNEXT_URL="http://erp.airlabs.com:8000"
API_KEY="your_api_key"
API_SECRET="your_api_secret"
AUTH_HEADER="Authorization: token ${API_KEY}:${API_SECRET}"

echo "Testing ERPNext API Configuration..."

# Test 1: Basic connectivity
echo "1. Testing basic connectivity..."
curl -s -o /dev/null -w "%{http_code}" \
  -H "$AUTH_HEADER" \
  "$ERPNEXT_URL/api/resource/Task?limit=1"

# Test 2: Get tasks with custom fields
echo "2. Testing task retrieval..."
curl -H "$AUTH_HEADER" \
  -H "Content-Type: application/json" \
  "$ERPNEXT_URL/api/resource/Task?fields=[\"name\",\"subject\",\"custom_assigned_to\"]&limit=5"

# Test 3: Test permissions
echo "3. Testing create permissions..."
curl -X POST \
  -H "$AUTH_HEADER" \
  -H "Content-Type: application/json" \
  -d '{"subject":"API Test Task","status":"Open"}' \
  "$ERPNEXT_URL/api/resource/Task"
```

### N8N Test Workflow

Create a simple test workflow:

```json
{
  "nodes": [
    {
      "name": "Test API Connection",
      "type": "n8n-nodes-base.httpRequest",
      "parameters": {
        "method": "GET",
        "url": "={{$env.ERPNEXT_URL}}/api/resource/Task",
        "authentication": "predefinedCredentialType",
        "nodeCredentialType": "httpHeaderAuth",
        "sendQuery": true,
        "queryParameters": {
          "limit": "5"
        }
      }
    }
  ]
}
```

## 🔒 Security Best Practices

### API Key Management

1. **Use environment variables** for credentials
2. **Rotate API keys** every 90 days
3. **Monitor API usage** in ERPNext logs
4. **Restrict IP access** if possible
5. **Use HTTPS** for all API calls

### User Permissions

1. **Principle of least privilege** - only grant necessary permissions
2. **Regular permission audits** - review and update quarterly
3. **Separate users** for different integrations
4. **Monitor user activity** through ERPNext activity logs

### Network Security

```nginx
# Nginx configuration to restrict API access
location /api/ {
    # Allow only specific IPs
    allow 10.0.0.100;  # N8N server IP
    deny all;
    
    # Rate limiting
    limit_req zone=api burst=10 nodelay;
    
    proxy_pass http://erpnext_backend;
}
```

## 📋 Troubleshooting API Issues

### Common Error Codes

#### 401 Unauthorized
```json
{
  "message": "Authentication failed"
}
```
**Solution:** Check API key and secret

#### 403 Forbidden  
```json
{
  "message": "Insufficient Permission for Task"
}
```
**Solution:** Review user permissions

#### 404 Not Found
```json
{
  "message": "Task not found"
}
```
**Solution:** Verify task ID exists

#### 417 Expectation Failed
```json
{
  "message": "Mandatory field missing"
}
```
**Solution:** Check required fields in request

### Debug Mode

Enable debug logging in ERPNext:

```python
# In site_config.json
{
    "developer_mode": 1,
    "log_level": "DEBUG",
    "logging": 2
}
```

Monitor logs:
```bash
tail -f logs/web.log
```

---

This completes the API configuration guide. The next step is to implement these configurations in your N8N workflows and test the integration thoroughly.
