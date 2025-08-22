# Troubleshooting Guide - ERPNext N8N Automation

This guide helps you diagnose and fix common issues with the ERPNext N8N automation system.

## 🚨 Quick Diagnosis Checklist

Before diving into detailed troubleshooting, run through this quick checklist:

- [ ] **N8N workflow is activated** and shows "Active" status
- [ ] **Environment variables** are set correctly
- [ ] **ERPNext API** responds to test calls
- [ ] **Email credentials** are valid and working
- [ ] **OpenAI API key** has available credits
- [ ] **Task assignments** use correct field names (`custom_assigned_to`)

## 🔧 N8N Workflow Issues

### Issue: Workflow Not Executing Automatically

**Symptoms:**
- Schedule trigger doesn't fire
- No automatic email sends
- Manual execution works fine

**Diagnosis Steps:**
```bash
# Check N8N logs
docker logs n8n-container

# Verify cron expression
node -e "console.log(require('cron-parser').parseExpression('0 9 * * 1-5').next().toString())"
```

**Solutions:**

1. **Check Timezone Settings:**
```json
// In workflow settings
{
  "timezone": "Asia/Kolkata",
  "settings": {
    "executionOrder": "v1"
  }
}
```

2. **Verify Cron Expression:**
```javascript
// Common cron patterns
"0 9 * * 1-5"    // 9 AM, Monday-Friday
"0 9 * * *"      // 9 AM, every day
"*/30 * * * *"   // Every 30 minutes
```

3. **Check N8N System Status:**
- Ensure N8N has enough memory (minimum 512MB)
- Verify disk space availability
- Check system time synchronization

4. **Workflow Activation:**
- Deactivate and reactivate the workflow
- Save workflow after making any changes
- Check for any node errors (red indicators)

### Issue: Workflow Executions Failing

**Symptoms:**
- Red error indicators on nodes
- Partial execution completion
- Error messages in execution logs

**Common Error Messages and Solutions:**

#### "Cannot read property of undefined"
```javascript
// Problem: Accessing non-existent data
$json.tasks.forEach() // tasks might be undefined

// Solution: Add null checks
const tasks = $json.tasks || [];
tasks.forEach(task => {
  // Process task
});
```

#### "Network Error" or "Connection Timeout"
```javascript
// In HTTP Request nodes, add retry logic:
{
  "retry": {
    "times": 3,
    "interval": 5000
  },
  "timeout": 30000
}
```

#### "JSON Parse Error"
```javascript
// Problem: Malformed JSON from AI
try {
  const aiResponse = JSON.parse($json.choices[0].message.content);
} catch (error) {
  // Fallback handling
  return [{
    type: 'request_help',
    help_message: 'Could not parse AI response',
    confidence: 0.1
  }];
}
```

## 📡 ERPNext API Issues

### Issue: 401 Authentication Failed

**Symptoms:**
```json
{
  "message": "Authentication failed",
  "exc_type": "AuthenticationError"
}
```

**Diagnosis:**
```bash
# Test API credentials manually
curl -H "Authorization: token YOUR_KEY:YOUR_SECRET" \
     "http://erp.airlabs.com:8000/api/resource/Task?limit=1"
```

**Solutions:**

1. **Regenerate API Keys:**
   - Go to User record in ERPNext
   - Delete existing API keys
   - Generate new keys
   - Update N8N environment variables

2. **Check User Status:**
   - Ensure user is enabled
   - Verify user has "System User" type
   - Check user role assignments

3. **Verify API Access:**
   - System Settings > Allow API Access = Yes
   - User has API access enabled

### Issue: 403 Permission Denied

**Symptoms:**
```json
{
  "message": "Insufficient Permission for Task",
  "exc_type": "PermissionError"
}
```

**Solutions:**

1. **Review User Permissions:**
```python
# In ERPNext console
frappe.get_roles('your-api-user@domain.com')
frappe.permissions.get_doc_permissions(frappe.get_doc('Task', 'TASK-001'), 'your-api-user@domain.com')
```

2. **Check Doctype Permissions:**
   - User and Permissions > Doctype > Task
   - Verify Read/Write/Create permissions for user role
   - Check permission conditions

3. **User Permission Conflicts:**
   - Check if user
