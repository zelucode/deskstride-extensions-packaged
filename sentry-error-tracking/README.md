# Sentry Error Tracking Extension

**Version:** 1.0.1

A Sentry integration for DeskStride that enables error tracking, monitoring, and debugging for workflows.

## Features

- **Capture Exception**: Send exceptions with stack traces to Sentry
- **Capture Message**: Send log messages to Sentry for monitoring
- **Add Breadcrumb**: Track sequence of events leading to errors
- **Set User**: Associate errors with specific users
- **Set Context**: Add custom context information to errors

## Setup

### Sentry Project Setup

1. Create a Sentry account at https://sentry.io/
2. Create a new project in Sentry
3. Select the appropriate platform (Python or Generic)
4. Copy the DSN (Data Source Name) from Sentry project settings

### Extension Settings

Configure in the Extensions page:

- **Sentry DSN**: Data Source Name from your Sentry project (required, marked as secret)
- **Environment**: Environment name (e.g., production, staging, development)
- **Release**: Release version or identifier
- **Sample Rate**: Error sampling rate (0.0 to 1.0, default: 1.0)

## Nodes

### Capture Exception
Capture an exception and send it to Sentry for error tracking and monitoring.

**Inputs:**
- Error Message (required): The error message to capture
- Error Type: Type of error (e.g., ValueError, ConnectionError)
- Stack Trace: Stack trace or additional error details
- Level: Severity level (debug, info, warning, error, fatal)
- Tags: Tags as JSON object or comma-separated key=value pairs
- Extra Data: Additional context data as JSON object

**Outputs:**
- success: Boolean indicating if exception was captured
- errorMessage: The error message that was captured
- errorType: The error type
- level: The severity level
- message: Status message

### Capture Message
Capture a message and send it to Sentry for logging and monitoring.

**Inputs:**
- Message (required): The message to capture
- Level: Severity level (debug, info, warning, error, fatal)
- Tags: Tags as JSON object or comma-separated key=value pairs
- Extra Data: Additional context data as JSON object

**Outputs:**
- success: Boolean indicating if message was captured
- capturedMessage: The message that was captured
- level: The severity level
- message: Status message

### Add Breadcrumb
Add a breadcrumb to Sentry for tracking the sequence of events leading to an error.

**Inputs:**
- Message (required): Breadcrumb message
- Category: Breadcrumb category (e.g., user, http, navigation)
- Level: Breadcrumb level (debug, info, warning, error)
- Type: Breadcrumb type (e.g., default, http, navigation)
- Data: Additional data as JSON object

**Outputs:**
- success: Boolean indicating if breadcrumb was added
- breadcrumbMessage: The breadcrumb message
- category: The breadcrumb category
- level: The breadcrumb level
- message: Status message

### Set User
Set user context for Sentry error tracking. Associates errors with specific users.

**Inputs:**
- User ID: Unique user identifier
- Username: User's username
- Email: User's email address
- Additional User Data: Additional user data as JSON object

**Outputs:**
- success: Boolean indicating if user context was set
- userId: The user ID
- username: The username
- email: The email address
- result: Status message

### Set Context
Set custom context for Sentry error tracking. Adds additional context information to errors.

**Inputs:**
- Context Name (required): Name of the context (e.g., deskstride, workflow)
- Context Data (required): Context data as JSON object

**Outputs:**
- success: Boolean indicating if context was set
- contextName: The context name
- result: Status message

## Use Cases

- **Error Monitoring**: Track and debug workflow errors in production
- **User Impact Analysis**: See which users are affected by errors
- **Performance Tracking**: Monitor workflow performance and identify bottlenecks
- **Debugging**: Get detailed stack traces and context for errors
- **Alerting**: Set up Sentry alerts for critical errors
- **Trend Analysis**: Track error rates over time

## Example Workflows

### Error Handling with Sentry
```
1. Start Workflow
2. Try:
   - Execute main workflow logic
3. Except:
   - Capture Exception: errorMessage={{error}}, level=error
   - Add Breadcrumb: message="Error occurred in main workflow", category=workflow
4. End
```

### User Context Tracking
```
1. Get User ID from previous step
2. Set User: userId={{1.userId}}, username={{1.username}}, email={{1.email}}
3. Execute workflow logic
4. If error occurs, it will be associated with the user
```

### Context-Rich Error Tracking
```
1. Set Context: contextName=workflow, contextData={"workflow_id": "123", "version": "1.0"}
2. Add Breadcrumb: message="Starting data processing", category=workflow
3. Process Data
4. Add Breadcrumb: message="Data processing completed", category=workflow
5. If error occurs, full context is available in Sentry
```

## Best Practices

- **Use Breadcrumbs**: Add breadcrumbs at key workflow steps for better debugging
- **Set User Context**: Always set user context when workflows are user-specific
- **Context-Rich Errors**: Use custom context to provide relevant debugging information
- **Appropriate Levels**: Use appropriate severity levels (debug, info, warning, error, fatal)
- **Tags**: Use tags to filter and group errors in Sentry
- **Sample Rate**: Use sample rate for high-volume workflows to reduce Sentry costs

## Error Levels

- **debug**: Detailed debugging information
- **info**: General informational messages
- **warning**: Warning messages for potential issues
- **error**: Error messages for failures
- **fatal**: Critical errors that require immediate attention

## Context and Tags

### Context
Context provides structured information about the environment or state:
```json
{
  "workflow_id": "abc123",
  "node_id": "process_data",
  "environment": "production",
  "version": "1.2.0"
}
```

### Tags
Tags are key-value pairs for filtering and grouping:
```json
{
  "workflow_type": "data_processing",
  "priority": "high",
  "region": "us-east"
}
```

Or as comma-separated: `workflow_type=data_processing,priority=high,region=us-east`

## Requirements

- sentry-sdk==1.39.1 (included in extension)
- Sentry account and project
- Sentry DSN configured in extension settings

## Security

- Sentry DSN is stored as a secret in extension settings (encrypted)
- No sensitive data should be included in error messages or context
- Use Sentry's data scrubbing features to filter sensitive information
- Consider using sample rate to reduce data sent to Sentry

## Troubleshooting

### "Sentry DSN is required"
- Configure the DSN in extension settings
- Get the DSN from your Sentry project settings
- Ensure the DSN is valid and the project is active

### "Failed to capture exception"
- Check your Sentry DSN is correct
- Verify your Sentry project is accessible
- Check network connectivity to Sentry servers
- Review Sentry logs for specific error details

### Events not appearing in Sentry
- Check Sentry project filters and issues
- Verify the environment matches your Sentry project settings
- Check if sample rate is filtering events
- Review Sentry ingestion logs

## Notes

- Sentry client is initialized on first use with extension settings
- All nodes automatically add DeskStride context information
- Breadcrumbs are included in subsequent error events
- User context persists until changed or cleared
- Context and tags provide powerful filtering in Sentry dashboard
- Extension supports all Sentry Python SDK features for error tracking

## Contract (v1.0.1)

**Install:** Extensions → **Install from file** → choose `sentry-error-tracking.dsext`.

**Permissions:**
- **network**
- **secrets**

No example workflow: needs Sentry DSN.

**Lint / scan / pack:**
```bash
python tools/deskstride_ext_cli.py check extensions/sentry-error-tracking
python tools/deskstride_ext_cli.py pack extensions/sentry-error-tracking -o sentry-error-tracking.dsext
```
